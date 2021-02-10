---
title: HotSpot JVM 浮点比较
date: 2021-02-10 21:55:52
updated: 2021-02-10 22:31:45
category: JVM
tags: [HotSpot, JVM]
comments: true
---

在 JVM 中，比较主要有两类，一类是整型比较，还有一类就是本文要讲的浮点比较(Float Compare)。

<!--more-->

## JVM 对浮点比较的定义

浮点比较时，会出现操作数为非数 (NaN, not an number) 的情况。对于比较的操作数是否包含 NaN，分为 ordered 和 unordered 两种。其中，unordered 表示包含 NaN 的情况。

浮点比较的字节码有四个：

- fcmpl (字节码序: 149, 0x95)
- fcmpg (字节码序: 150, 0x96)
- dcmpl (字节码序: 151, 0x97)
- dcmpg (字节码序: 152, 0x98)

其中 `fcmpl/fcmpg` 用作单精度浮点 (Float 类型数据) 的比较，而 `dcmpl/dcmpg` 则用作双精度浮点 (Double 类型数据) 的比较。除单/双进度的区别意外，`fcmp` 和 `dcmp` 的行为是一致的。

对于两个比较数 value1 和 value2，value1 和 value2 从操作数栈中弹出并作了值转换，成为了 value1' 和 value2'，然后进行比较。`fcmp/dcmp` 的比较结果如下：

- 如果 `value1' > value2'`，那么将会把 1 作为比较结果，并 push 到操作数栈上
- 否则，如果 `value1' == value2'`，那么将把 0 作为比较结果，并 push 到操作数栈上
- 否则，如果 `value1' < value2'`，那么将把 -1 作为比较结果，并 push 到操作数栈上
- 否则，在 `value1'` 和 `value2'` 中至少有一个数为 NaN。则 `fcmpg/dcmpg` 将会把 1 作为比较结果，`fcmpl/dcmpl` 则会把 -1 作为比较结果，push 到操作数栈上

浮点比较遵循的是 IEEE 745 标准。除 NaN 以外的浮点比较均为 ordered，其中负无穷 (negative infinity) 小于任意的有穷值 (finite values)；正无穷 (positive infinity) 则大于任意有穷值；正零 (positive zero) 和负零 (negative zero) 则相等。

Oracle 的文档中对于 `fcmpg/dcmpg` 和 `fcmpl/dcmpl` 的区别是这样描述的：

> The fcmpg/dcmpg and fcmpl/dcmpl instructions differ only in their treatment of a comparison involving NaN. NaN is unordered, so any float/double comparison fails if either or both of its operands are NaN. With both fcmpg/dcmpg and fcmpl/dcmpl available, any float/double comparison may be compiled to push the same result onto the operand stack whether the comparison fails on non-NaN values or fails because it encountered a NaN.

也就是说，对于 `fcmpg/dcmpg` 和 `fcmpl/dcmpl` 的区别仅在于比较 NaN 时返回的值不同。NaN 表示 unordered 比较，当一个或多个数为 NaN 时，任何浮点比较都不应该成立。对于任意浮点比较指令，无论比较熟是否包含 NaN，比较不成立时所产生的值应该保持一致。

下面举个例子来说明一下：

```java
int lessThan100(double d) {
    if (d < 100.0) {
        return 1;
    } else {
        return -1;
    }
}
```

这段代码对应的字节码为：

```
int lessThan100(double);
  Code:
     0: dload_1
     1: ldc2_w        #2                  // double 100.0d
     4: dcmpg
     5: ifge          10
     8: iconst_1
     9: ireturn
    10: iconst_m1
    11: ireturn
```

如果 d 不是 NaN 且 d < 100.0，则 dcmpg 将会将 -1 push 到操作数栈上，由于结果为 -1，`ifge` 并不会跳转，而是继续执行 `iconst_1` (1) 。而如果 `d > 100.0` 或者为 NaN，则会将 1 push 到操作数栈上，`ifge` 会跳转到 `iconst_m1` (-1)，如果 `d == 100.0`，则会将 0 push 到操作数栈上，`ifge` 跳转到 `iconst_m1` (-1)。那么最终方法的返回值为：

- `d < 100.0`，返回 1
- `d >= 100.0`，返回 -1
- d 为 NaN，返回 -1

可以看到 d 为 NaN 时与 `d >= 100.0` 时，`lessThan100` 都返回了 -1，即比较数包含 NaN 时 与比较不成立时返回的结果保持一致。

而对于 dcmpl，则可以通过下面的例子说明：

```java
int greaterThan100(double d) {
    if (d > 100.0) {
        return 1;
    } else {
        return -1;
    }
}
```

其对应的字节码为：

```
int greaterThan100(double);
  Code:
       0: dload_1
       1: ldc2_w        #2                  // double 100.0d
       4: dcmpl
       5: ifle          10
       8: iconst_1
       9: ireturn
      10: iconst_m1
      11: ireturn
```

同样的，最终方法的返回结果如下：

- `d <= 100.0`，返回 -1
- `d > 100`，返回 1
- d 为 NaN，返回 -1

对于 `greaterThan100` 方法，d 为 NaN 时返回的结果与 `d > 100.0` 比较不成立时返回的结果一致。

## Interpreter 中 Float Compare 的实现

> 下面所涉及的代码均基于 OpenJDK 11u 分支的代码：https://github.com/openjdk/jdk11u

在 `share/interpreter/bytecodeInterpreter.hpp` 中有定义 VMfloatCompare 和 VMdoubleCompare：

```cpp
/*
 * Comparisons (returning an int value: 0, 1, or -1)
 *
 * Between operands:
 *
 * Compare "op1" and "op2" according to the semantics of the
 * "fcmpl" (direction is -1) or "fcmpg" (direction is 1) bytecodes.
 */

static int32_t VMfloatCompare(jfloat op1, jfloat op2,
                              int32_t direction);

/*
 * Comparisons (returning an int32_t value: 0, 1, or -1)
 *
 * Between operands:
 *
 * Compare "op1" and "op2" according to the semantics of the
 * "dcmpl" (direction is -1) or "dcmpg" (direction is 1) bytecodes.
 */

static int32_t VMdoubleCompare(jdouble op1, jdouble op2, int32_t direction);
```

这里的 direction 用于区分 `fcmpl/dcmpl` 和 `dcmpg/dcmpg`。下面就以 double 类型为例 (float 与 double 一致)。

在 `share/interpreter/templateTable.cpp` 中定义了 `Bytecodes::_dcmpl` 和 `Bytecodes::_dcmpg`：

```cpp
def(Bytecodes::_dcmpl, ____|____|____|____, dtos, itos, double_cmp, -1);
def(Bytecodes::_dcmpg, ____|____|____|____, dtos, itos, double_cmp,  1);
```

在同一文件中，`double_cmp` 的定义如下：

```cpp
void TemplateTable::double_cmp(int unordered_result) {
  transition(dtos, itos);
  float_cmp(false, unordered_result);
}
```

从这两处代码，可以看到在定义字节码是传入的 -1 和 1 是给 `double_cmp` 中的 `unordered_result` 赋值用的。然后在 `float_cmp` 方法中做了平台相关的比较。这里的 false 表示的是比较的类型是 float (true) 还是 double (false)。

以 AArch64 的 `float_cmp` 方法为例，其定义位于 `cpu/aarch64/templateTable_aarch64.cpp` 中：

```cpp
void TemplateTable::float_cmp(bool is_float, int unordered_result)
{
  Label done;
  if (is_float) {
    // XXX get rid of pop here, use ... reg, mem32
    __ pop_f(v1);
    __ fcmps(v1, v0);
  } else {
    // XXX get rid of pop here, use ... reg, mem64
    __ pop_d(v1);
    __ fcmpd(v1, v0);
  }
  if (unordered_result < 0) {
    // we want -1 for unordered or less than, 0 for equal and 1 for
    // greater than.
    __ mov(r0, (u_int64_t)-1L);
    // for FP LT tests less than or unordered
    __ br(Assembler::LT, done);
    // install 0 for EQ otherwise 1
    __ csinc(r0, zr, zr, Assembler::EQ);
  } else {
    // we want -1 for less than, 0 for equal and 1 for unordered or
    // greater than.
    __ mov(r0, 1L);
    // for FP HI tests greater than or unordered
    __ br(Assembler::HI, done);
    // install 0 for EQ otherwise ~0
    __ csinv(r0, zr, zr, Assembler::EQ);

  }
  __ bind(done);
}
```

由于 AArch64 中存在 flag 寄存器，因此比较一般都是先通过 `cmp` 指令来改变 flag 寄存器，然后通过读取 flag 寄存器的值来确定比较结果。我们只需关注源码中 unordered_result 的那个 if/else 分支。可以看到，对于 unordered_result 的取值不同，比较返回的结果是不一样的，总结如下：

- unordered_result < 0 (Bytecodes::_dcmpl)
  - -1: lt or unordered (NaN)
  - 0: eq
  - 1: gt
- unordered_result > 0 (Bytecodes::_dcmpg)
  - -1: lt
  - 0: eq
  - 1: gt or unordered (NaN)

从对字节码的定义来看，`unordered_result < 0` 对应的是 `Bytecodes::_dcmpl`，`unordered_result > 0` 对应的则是 `Bytecodes::_dcmpg`，这与 Oracle 文档中的描述一致。此外，由于 `dcmpl/dcmpg` 会产生 -1，0，1 三种结果，并不表示最终的比较结果，因此后面往往会跟随一条 `ifxx` (`_ifeq`/`_ifne`/`_iflt`/`_ifle`/`_ifgt`/`_ifge`，均与 0 进行比较) 字节码，也就是说实际 Java 代码中的浮点 Double 的比较结果，是由 `dcmpl/dcmpg + ifxx` 组合产生的。

## C1 中 Float Compare 的实现

C1 `LIRGenerator::do_CompareOp` 方法用于生成 `_dcmpl` 和 `_dcmpg` 对应的汇编指令 (位于 `cpu/aarch64/c1_LIRGenerator_aarch64.cpp`)：

```cpp
// _lcmp, _fcmpl, _fcmpg, _dcmpl, _dcmpg
void LIRGenerator::do_CompareOp(CompareOp* x) {
  LIRItem left(x->x(), this);
  LIRItem right(x->y(), this);
  ValueTag tag = x->x()->type()->tag();
  if (tag == longTag) {
    left.set_destroys_register();
  }
  left.load_item();
  right.load_item();
  LIR_Opr reg = rlock_result(x);

  if (x->x()->type()->is_float_kind()) {
    Bytecodes::Code code = x->op();
    __ fcmp2int(left.result(), right.result(), reg, (code == Bytecodes::_fcmpl || code == Bytecodes::_dcmpl));
  } else if (x->x()->type()->tag() == longTag) {
    __ lcmp2int(left.result(), right.result(), reg);
  } else {
    Unimplemented();
  }
}
```

对于浮点比较，只需的是 `fcmp2int` 这个方法，其定义位于 `share/c1/c1_LIR.cpp` 中：

```cpp
void LIR_List::fcmp2int(LIR_Opr left, LIR_Opr right, LIR_Opr dst, bool is_unordered_less) {
  append(new LIR_Op2(is_unordered_less ? lir_ucmp_fd2i : lir_cmp_fd2i,
                     left,
                     right,
                     dst));
}
```

这里根据 `is_unordered_less` 的取值会选择 `lir_ucmp_fd2i` 或者 `lir_cmp_fd2i`。而 `is_unordered_lsee` 为 true 的情况为`Bytecodes::_fcmpl` 或者 `Bytecodes::_dcmpl`。也就是说，这里的 `is_unordered_less` 的含义与解释器中的 unordered_result 一致：

- is_unordered_less == true ===> unordered_result < 0 ===> `dcmpl`
- is_unordered_less == false ===> unordered_result > 0 ===> `dcmpg`

而 C1 的浮点比较是通过 `LIR_Assembler::comp_fl2i` 来完成的 (位于 `cpu/aarch64/c1_LIRAssembler_aarch64.cpp`)：

```cpp
void LIR_Assembler::comp_fl2i(LIR_Code code, LIR_Opr left, LIR_Opr right, LIR_Opr dst, LIR_Op2* op){
  if (code == lir_cmp_fd2i || code == lir_ucmp_fd2i) {
    bool is_unordered_less = (code == lir_ucmp_fd2i);
    if (left->is_single_fpu()) {
      __ float_cmp(true, is_unordered_less ? -1 : 1, left->as_float_reg(), right->as_float_reg(), dst->as_register());
    } else if (left->is_double_fpu()) {
      __ float_cmp(false, is_unordered_less ? -1 : 1, left->as_double_reg(), right->as_double_reg(), dst->as_register());
    } else {
      ShouldNotReachHere();
    }
  } else if (code == lir_cmp_l2i) {
    Label done;
    __ cmp(left->as_register_lo(), right->as_register_lo());
    __ mov(dst->as_register(), (u_int64_t)-1L);
    __ br(Assembler::LT, done);
    __ csinc(dst->as_register(), zr, zr, Assembler::EQ);
    __ bind(done);
  } else {
    ShouldNotReachHere();
  }
}
```

对于浮点数 (fpu)，最终的比较在 `float_cmp` 中 (位于 `cpu/aarch64/c1_MacroAssembler_aarch64.cpp`)：

```cpp
void C1_MacroAssembler::float_cmp(bool is_float, int unordered_result,
                                  FloatRegister f0, FloatRegister f1,
                                  Register result)
{
  Label done;
  if (is_float) {
    fcmps(f0, f1);
  } else {
    fcmpd(f0, f1);
  }
  if (unordered_result < 0) {
    // we want -1 for unordered or less than, 0 for equal and 1 for
    // greater than.
    cset(result, NE);  // Not equal or unordered
    cneg(result, result, LT);  // Less than or unordered
  } else {
    // we want -1 for less than, 0 for equal and 1 for unordered or
    // greater than.
    cset(result, NE);  // Not equal or unordered
    cneg(result, result, LO);  // Less than
  }
}
```

可以看到，`C1_MacroAssembler::float_cmp` 的实现逻辑基本与解释器中的 `fcmp` 一致：

- unordered_result < 0 (is_unordered_less == -1) ===> `Bytecodes::_dcmpl`
  - -1: lt or unordered (NaN)
  - 0: eq
  - 1: gt
- unordered_result > 0 (is_unordered_less == 1) ===> `Bytecodes::_dcmpg`
  - -1: lt
  - 0: eq
  - 1: gt or unordered (NaN)

对于浮点比较的逻辑，解释器与 C1 是保持一致的。

## C2 中 Float Compare 的实现

C2 中对于 `Bytecodes::_dcmpl` 和 `Bytecodes::_dcmpg` 的处理与解释器/C1 稍有区别 (位于 `share/opto/parse2.cpp`)，

```cpp
case Bytecodes::_dcmpl:
  b = pop_pair();
  a = pop_pair();
  c = _gvn.transform( new CmpD3Node( a, b));
  push(c);
  break;

case Bytecodes::_dcmpg:
  b = pop_pair();
  a = pop_pair();
  // Same as dcmpl but need to flip the unordered case.
  // Commute the inputs, which negates the result sign except for unordered.
  // Flip the unordered as well by using CmpD3 which implements
  // unordered-lesser instead of unordered-greater semantics.
  // Finally, negate the result bits.  Result is same as using a
  // CmpD3Greater except we did it with CmpD3 alone.
  c = _gvn.transform( new CmpD3Node( b, a));
  c = _gvn.transform( new SubINode(_gvn.intcon(0),c) );
  push(c);
  break;
```

C2 中 `dcmpl` 和 `dcmpg` 的比较是通过 `CmpD3Node` 来实现的，其比较对应的 C2 instruct 为 `compD3_reg_reg` (定义在 `cpu/aarch64/aarch64.ad` 中)：

```cpp
instruct compD3_reg_reg(iRegINoSp dst, vRegD src1, vRegD src2, rFlagsReg cr)
%{
  match(Set dst (CmpD3 src1 src2));
  effect(KILL cr);

  ins_cost(5 * INSN_COST);
  format %{ "fcmpd $src1, $src2\n\t"
            "csinvw($dst, zr, zr, eq\n\t"
            "csnegw($dst, $dst, $dst, lt)"
  %}

  ins_encode %{
    Label done;
    FloatRegister s1 = as_FloatRegister($src1$$reg);
    FloatRegister s2 = as_FloatRegister($src2$$reg);
    Register d = as_Register($dst$$reg);
    __ fcmpd(s1, s2);
    // installs 0 if EQ else -1
    __ csinvw(d, zr, zr, Assembler::EQ);
    // keeps -1 if less or unordered else installs 1
    __ csnegw(d, d, d, Assembler::LT);
    __ bind(done);
  %}
  ins_pipe(pipe_class_default);

%}
```

这里返回的结果为：

- -1: lt or unordered (NaN)
- 0: eq
- 1: gt

即 C2 中的 `compD3_reg_reg` 的结果与 `dcmpl` 期望的结果一致。

但是 `compD3_reg_reg` 只有 `is_unordered < 0` (也就是 `unordered_less`) 的情况，并没有考虑 `is_unordered >0`。而在解释器/C1 中，两种情况均有考虑。我们再回到 C2 中对应 `Bytecodes::_dcmpg` 处，可以发现相比 `dcmpl`，`dcmpg` 把两个比较的操作数位置互换了，并且比较完以后，并没有直接将结果返回，而是通过一个 `SubINode` 做了一个减法，将 0 减去 compare 所产生的结果，这样就会让结果完全反过来，也即原本的 -1，0，1经过 `SubINode` 后变成了 1，0，-1。那么对于 NaN，`dcmpg` 的返回值时多少呢？

由于将比较的操作数交换了，那么原来 `dcmpl` 中比较的 a compare b 就变成了 b compare a。对于 `compD3_reg_reg` 来说，并没有任何影响。对于 a compare b 的结果如下：

- -1: a lt b or unordered
- 0: a eq b
- 1: a gt b

那么现在 b compare a 的结果为：

- -1: b lt a or unordered
- 0: b eq a
- 1: b gt a

在通过 `SubINode`：

- 1: b lt a or unordered
- 0: b eq a
- -1: b gt a

那么最终就等于：

- -1: a lt b
- 0: a eq b
- 1: a gt b or unordered

这样就与解释器/C1 中的 `unordered_result < 0` 一致了。也就是说，C2 平台相关的部分只实现了 `unordered_result < 0` 的情况，而通过 `unordered_result < 0` 、交换比较操作数以及将比较结果取反，巧妙地得到了 `unordered_result > 0` 的结果。

## 总结

HotSpot JVM 中对于浮点的比较，可以分为 `fcmpl/dcmpl` 和 `fcmpl/dcmpg` 两种，两个操作数的比较有三个结果：1，0，-1。因此，为了返回两个浮点数的比较结果，一般均会伴随一个 `ifxx` 字节码，用于跳转到正确的比较结果并返回。而对于操作数包含 NaN 的场景，`fcmpl/dcmpl` 和 `fcmpl/dcmpg` 的结果稍有不同，`fcmpl/dcmpl` 返回 -1，`fcmpl/dcmpl` 则返回 1。

## 参考资料

[1]: [jdk11u ](https://github.com/openjdk/jdk11u)
[2]: [Chapter 6. The Java Virtual Machine Instruction Set](https://docs.oracle.com/javase/specs/jvms/se15/html/jvms-6.html)