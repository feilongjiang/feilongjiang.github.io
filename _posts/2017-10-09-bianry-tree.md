---
title: 【转载】二叉树相关问题
date: 2017-10-09 19:33:35
category: [Data Structure]
tags: [binary tree]
comments: true
---

文章转自 <http://blog.csdn.net/walkinginthewind/article/details/7518888>

# 二叉树

树是一种比较重要的数据结构，尤其是二叉树。二叉树是一种特殊的树，在二叉树中每个节点最多有两个子节点，一般称为左子节点和右子节点（或左孩子和右孩子），并且二叉树的子树有左右之分，其次序不能任意颠倒。二叉树是递归定义的，因此，与二叉树有关的题目基本都可以用递归思想解决，当然有些题目非递归解法也应该掌握，如非递归遍历节点等等。

# 二叉树节点定义

```c++
/* Definition for a binary tree node*/
struct TreeNode {
    int val;
    TreeNode* left;    // left tree node
    TreeNode* right;   // right tree node
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}    // ctor
};
```

# 问题列表

二叉树的大部分题目都应用了递归，时刻往递归结构去思考。

[1. 求二叉树中的节点个数](#nodeNum)  
[2. 求二叉树的深度](#nodeDepth)  
[3. 前序、中序、后序遍历](#nodeTraverse)  
[4. 其他遍历二叉树方法（深度、广度优先）](#nodeDFSBFS)  
[5. 将二叉查找树变为有序的双向链表](#nodeList)  
[6. 求二叉树第K层的节点个数](#nodeKth)  
[7. 求二叉树中叶子节点的个数](#nodeLeaf)  
[8. 判断两棵二叉树结构是否相同](#nodeStructure)  
[9. 判断二叉树是不是平衡二叉树](#nodeAVL)  
[10. 求二叉树的镜像](#nodeMirror)  
[11. 求二叉树中两个节点的最低公共祖先节点](#nodeAncestor)  
[12. 求二叉树中节点的最大距离](#nodeDistance)  
[13. 由前序遍历序列和中序遍历序列重建二叉树](#nodeRebuild)  
[14. 判断二叉树是不是完全二叉树](#nodeComplete)

# 详细解答

## <span id="nodeNum">**求二叉树中的节点个数**</span>

递归解法：
1. 如果二叉树为空，节点个数为
2. 如果二叉树不为空，二叉树节点个数=左子树节点个数+右子树节点个数+1

参考代码：
```c++
int getNodeNum(TreeNode* root)
{
    if (!root) return 0;    // 递归出口
    return getNodeNum(root->left) + getNodeNum(root->right) + 1;
}
```

## <span id="nodeDepth">**求二叉树的深度**</span>

递归解法：
1. 如果二叉树为空，二叉树的深度为0
2. 如果二叉树不为空，二叉树的深度=max(左子树深度， 右子树深度)+1

参考代码：
```c++
int getDepth(TreeNode* root)
{
    if (!root) return 0;    // 递归出口
    int depthLeft = getDepth(root->left);
    int depthRight = getDepth(root->right);
    return depthLeft > depthRight ? depthLeft + 1 : depthRight + 1;
}
```

## <span id="nodeTraverse">**前序、中序、后序遍历**</span>

### 前序遍历递归解法：

1. 如果二叉树为空，空操作
2. 如果二叉树不为空，访问根节点，前序遍历左子树，前序遍历右子树

参考代码：
```c++
void preOrderTraverse(TreeNode* root)
{
    if (!root) return;
    visit(root);                     // 访问根结点
    preOrderTraverse(root->left);    // 前序遍历左子树
    preOrderTraverse(root->right);   // 前序遍历右子树
}
```

### 中序遍历递归解法：

1. 如果二叉树为空，空操作
2. 如果二叉树不为空，中序遍历左子树，访问根结点，中序遍历右子树

参考代码：
```c++
void inOrderTraverse(TreeNode* root)
{
    if (!root) return;
    inOrderTraverse(root->left);    // 中序遍历左子树
    visit(root)；                   // 访问根结点
    inOrderTraverse(root->right);   // 中序遍历右子树
}
```

### 后序遍历递归解法：

1. 如果二叉树为空，空操作
2. 如果二叉树不为空，后序遍历左子树，后序遍历右子树，访问根结点

参考代码：
```c++
void postOrderTraverse(TreeNode* root)
{
    if (!root) return;
    postOrderTraverse(root->left);    // 后序遍历左子树
    postOrderTraverse(root->right);   // 后序遍历右子树
    visit(root);                      // 访问根结点
}
```

## <span id="nodeDFSBFS">**其他遍历二叉树方法(深度、广度优先)**</span>

### 深度优先遍历解法：
1. 借助一个栈（后进先出）来实现深度遍历
2. 先访问根结点
3. 遍历左子树接着遍历右子树

参考代码：
```c++
void DFS(TreeNode* root)
{
    if (!root) return;
    stack<TreeNode*> nodeStack;
    nodeStack.push(root);
    while (!nodeStack.empty()) {
        TreeNode *tmp = nodeStack.top();
        visit(tmp);
        nodeStack.pop();
        if (tmp->right)
            nodeStack.push(tmp->right);
        if (tmp->left)
            nodeStack.push(tmp->left);
    }
}
```

### 广度优先遍历解法：
1. 借助队列（先进先出）来实现广度优先遍历
2. 将根节点入队
3. 当队列不为空，弹出一个节点，访问，若左子节点或右子节点不为空，将其入队

参考代码：
```c++
void BFS(TreeNode* root)
{
    if (!root) return;
    queue<TreeNode*> nodeQueue;
    q.push(root);
    while (!nodeQueue.empty()) {
        TreeNode *tmp = q.front();
        q.pop();
        visit(tmp);
        if (tmp->left)
            nodeQueue.push(tmp->left);
        if (tmp->right)
            nodeQueue.push(tmp->right);
}
```

## <span id="nodeList">**将二叉查找树变为有序的双向链表**</span>

要求不创建新节点，只调整指针。
递归解法：
1. 如果二叉查找树为空，不需要转换，对应双向链表的第一个节点是NULL，最后一个节点也是NULL
2. 如果二叉查找树不为空：
    如果左子树为空，对应双向有序链表的第一个节点是根节点，左边不需要其他操作；
    如果左子树不为空，转换左子树，二叉查找树对应双向有序链表的第一个节点就是左子树转换后双向有序链表的第一个节点，同时将根节点与左子树转换后的双向有序链表的最后一个节点连接；
    如果右子树为空，对应双向有序链表的最后一个节点是根节点，右边不需要其他操作；
    如果右子树不为空，对应双向有序链表的最后一个节点就是右子树转换后双向有序链表的最后一个节点，同时将根节点与右子树转换后的双向有序链表的第一个节点连接。

参考代码：
```c++
/**
  * root: 二叉查找树的根结点指针
  * pFirstNode: 转换后双向有序链表的第一个节点指针
  * pLastNode: 转换后双向有序链表的最后一个节点指针
  **/
 void convert(TreeNode* root, 
              TreeNode* &pFirstNode, 
              TreeNode* &pLastNode) {
    TreeNode *pFirstLeft, *pLastLeft, *pFirstRight, *pLastRight;
    if (!root) {
        pFirstNode = NULL;
        pLastNode = NULL;
        return;
    }

    if (!root->left)
        // 如果左子树为空，对应双向有序链表的第一个节点是根节点
        pFirstNode = root;
    else {
        convert(root->left, pFirstLeft, pLastLeft);
        // 二叉查找树对应双向有序链表的第一个节点就是
        // 左子树转换后双向有序链表的第一个节点
        pFristNode = pFirstLeft;
        // 将根节点与左子树转换后的双向有序链表的最后一个节点连接
        root->left = pLastLeft;
        pLastLeft->right = root;
    }

    if (!root->right)
        // 对应双向有序链表的最后一个节点是根节点
        pLastNode = root;
    else {
        convert(root->right, pFirstRight, pLastRight);
        // 对应双向有序链表的最后一个节点就是
        // 右子树转换后双向有序链表的最后一个节点
        pLastNode = pLastRight;
        // 将根节点和右子树转换后的双向有序链表的第一个节点连接
        root->right = pFirstRight;
        pFirstRight->left = root;
    }
}
```

## <span id="nodeKth">**求二叉树第K层的节点个数**</span>

递归解法:
1. 如果二叉树为空或者k < 1，返回0
2. 如果二叉树不为空且k = 1，返回1
3. 如果二叉树不为空且k > 1，返回左子树中k-1层节点个数与右子树k-1层节点个数之和

参考代码：
```c++
int getKthLevelNodeNum(TreeNode* root)
{
    if (!roo || k < 1) return 0;
    if (k == 1) return 1;
    
    int leftNum = getKthLevelNodeNum(root->left);    // 左子树中k-1层节点个数
    int rightNum = getKthLevelNodeNum(root->right);  // 右子树中k-1层节点个数
    return (leftNum + rightNum);
}
```

## <span id="nodeLeaf">**求二叉树中叶子节点的个数**</span>

递归解法：
1. 如果二叉树为空，返回0
2. 如果二叉树不为空且左右子树为空，返回1
3. 如果二叉树不为空，且左右子树不同时为空，返回左子树中叶子节点个数加上右子树中叶子结点个数

参考代码：
```c++
int getLeafNodeNum(TreeNode* root)
{
    if (!root) return 0;
    if(!root->left && !root->right) return 1;
    
    int numLeft = getLeafNodeNum(root->left);    // 左子树中叶节点个数
    int numRight = getLeafNodeNum(root->right);  // 右子树中叶节点个数
    return (numLeft + numRight);
}
```

## <span id="nodeStructure">**判断两棵二叉树结构是否相同**</span>

不考虑数据内容。结构相同意味着对应左子树和右子树结构都相同。
递归解法:
1. 如果两棵二叉树都为空，返回真
2. 如果两颗二叉树一棵为空，另一个不为空，返回假
3. 如果两棵二叉树都不为空，如果对应的左右子树都同构则返回真，其他返回假

参考代码：
```c++
bool structureCmp(TreeNode* lhs, TreeNode* rhs)
{
    if (!lhs && !rhs) return true;           // 都为空树，返回真
    else if (!lhs || !rhs) return false;     // 一个为空而另一个不为空，返回假
bool resultLeft = structureCmp(lhs->left, rhs->left);    // 比较对应左子树
bool resultRight = structureCmp(lhs->right, rhs->right); // 比较对应右子树
return (resultLeft && resultRight);
}
```

## <span id="nodeAVL">**判断二叉树是不是平衡二叉树**</span>

递归解法：
1. 如果二叉树为空，返回真
2. 如果二叉树不为空，如果左子树和右子树都是AVL树且左子树和右子树高度差不大于1，返回真，其他返回假

参考代码：
```c++
boo isAVL(TreeNode* root, int &height)
{
    if (!root) {    // 空树，返回真
        height = 0;
        return true;
    }
    
    int heightLeft;
    bool resultLeft = isAVL(root->left, heightLeft);
    int heightRight;
    bool resultRight = isAVL(root->right, heightRight);
    if (resultLeft && resultRight && abs(heightLeft - heightRight) <= 1){    
        // 左右子树都是AVL树，并且高度差不大于1，返回真
        height = max(heightLeft, heightRight) + 1;
        return true;
    }
    else {
        height = max(heightLeft, heightRight) + 1;
        return false;
    }
}
```

## <span id="nodeMirror">**求二叉树的镜像**</span>

递归解法：
1. 如果二叉树为空，返回空
2. 如果二叉树不为空，求左子树和右子树的镜像，然后交换左右子树

参考代码：
```c++
TreeNode* mirrorTree(TreeNode* root)
{
    if (!root) return NULL;
    
    TreeNode *leftTree = mirrorTree(root->left);    // 求左子树镜像
    TreeNode *rightTree = mirrorTree(root->right);  // 求右子树镜像
    // 交换左右子树
    root->left = leftTree;
    root->right = rightTree;
    return root;
}
```

## <span id="nodeAncestor">**求二叉树中两个节点的最低公共祖先节点**</span>

### 递归解法：

1. 如果两个节点分别在根结点的左子树和右子树，则返回根结点
2. 如果两个节点都在左子树，则递归处理左子树；如果两个节点都在右子树，则递归处理右子树

参考代码：
```c++
bool findNode(TreeNode* root, TreeNode *pNode)
{
    if (!root || !pNode) return false;
    
    if (root == pNode) return true;
    bool found = findNode(root->left, pNode);
    if (!found)
        found = findNode(root->right, pNode);
    return found;
}

TreeNode* getLowestCommonAncestor(TreeNode* root,
                                  TreeNode* pNode1,
                                  TreeNode* pNode2)
{
    if (findNode(root->left, pNode1)) {
        if (findNode(root->right, pNode2))
            return root;
        else
            return getLowestCommonAncestor(root->left, pNode1, pNode2);
    }
    else {
        if (find(root->left, pNode2))
            return root;
        else
            return getLowestCommonAncestor(root->right, pNode1, pNode2);
    }
}
```

递归解法效率较低，有很多重复遍历，下面看一下非递归解法。

### 非递归解法：

先从根结点到两个节点的路径，然后再比较对应路径的节点，最后一个相同的节点就是他们在二叉树中的最低公共祖先节点

参考代码：
```c++
bool getNodePath(TreeNode* root, 
                 TreeNode* pNode, 
                 list<TreeNode*> &path)
{
    if (root == pNode) {
        path.push_back(root);
        return true;
    }
    if (!root) return false;
    path.push_back(root);
    bool found = false;
    found = getNodePath(root->left, pNode, path);
    if (!found)
        found = getNodePath(root->right, pNode, path);
    if (!found)
        path.pop_back();
    return found;
}

TreeNode* getLowestCommonAncestor(TreeNode *root, 
                                  TreeNode* pNode1, 
                                  TreeNode* pNode2)
{
    if (!root || !pNode1 || !pNode2) return NULL;
    
    list<TreeNode*> path1;
    bool result1 = getNodePath(root, pNode1, path1);
    list<TreeNode*> path2;
    bool result2 = getNodePath(root, pNode2, path2);
    
    if (!result1 || !result2) return NULL;
    TreeNode* pLast = NULL;
    list<TreeNode*>::iterator iter1 = path1.begin();
    list<TreeNode*>::iterator iter2 = path2.begin();
    while (iter != path1.end() && iter2 != path2.end()) {
        if (*iter1 == *iter2)
            pLast = *iter1;
        else
            break;
        ++iter1;
        ++iter2;
    }
    return pLast;
}
```

在上述算法的基础上稍加变化即可求二叉树中任意两个节点的距离了。

## <span id="nodeDistance">**求二叉树中节点的最大距离**</span>

即二叉树中相距最远的两个节点之间的距离。
递归解法：
1. 如果二叉树为空，则返回0，同时记录左子树和右子树的深度都为0
2. 如果二叉树不为空，最大距离要么是左子树中的最大距离，要么是右子树中的最大距离，要么是左子树节点中到根节点的最大距离+右子树节点到根节点的最大距离，同时记录左子树和右子树节点到根节点的最大距离

参考代码：
```c++
int getMaxDistance(TreeNode* root, int &maxLeft, int &maxRight)
{
    // maxLeft: 左子树中的节点离根节点的最远距离
    // maxRight: 右子树中的节点离根节点的最远距离
    if (!root) {
        maxLeft = 0;
        maxRight = 0;
        return 0;
    }
    
    int maxLL, maxLR, maxRL, maxRR;
    int maxDistLeft, maxDistRight;
    if (root->left) {
        maxDistLeft = getMaxDistance(root->left, maxLL, maxLR);
        maxLeft = max(maxLL, maxLR) + 1;
    }
    else {
        maxDistLeft = 0;
        maxLeft = 0;
    }
    if (root->right) {
        maxDistRight = getMaxDistance(root->right, maxRL, maxRR);
        maxRight = max(maxRL, maxRR) + 1;
    }
    else {
        maxDistRight = 0;
        maxRight = 0;
    }
    return max(max(maxDistLeft, maxDistRight), maxLeft + maxRight);
}
```

## <span id="nodeRebuild">**由前序遍历序列和中序遍历序列重建二叉树**</span>

二叉树前序遍历序列中，第一个元素总是树的根结点。中序遍历序列中，左子树的节点位于根结点的左边，右子树的节点位于根结点的右边。
递归解法：
1. 如果前序遍历为空或者节点个数小于等于0，返回NULL。
2. 创建根节点。前序遍历的第一个节点就是根节点，在中序遍历中找到根节点的位置，可分别得知左子树和右子树的前序和中序遍历序列，重建左右子树

参考代码：
```c++
/**
  * pPreOrder: 前序遍历序列
  * pInOrder: 中序遍历序列
  * nodeNum: 二叉树节点数
  **/
TreeNode* rebuildTree(int* pPreOrder, int* pInOrder, int nodeNum)
{
    if (!pPreOrder || !pInOrder || nodeNum <= 0) return NULL;
    TreeNode* root = new TreeNode;
    // 前序遍历的第一个节点就是根节点
    root->val = pPreOrder[0];
    root->left = NULL;
    root->right = NULL;
    // 查找根节点在中序遍历中的位置，中序遍历中，根节点左边为左子树，右边为右子树
    int rootPoistionInOrder = -1;
    for (int i = 0; i < nodeNum; ++i) {
        if (pInOrder[i] == root->val) {
            rootPositionInOrder = i;
            break;
        }
    }
    if (rootPositionInOrder == -1)
        throw std::exception("Invalid Input.");
    
    // 重建左子树
    int nodeNumLeft = rootPositionInOrder;
    int* pPreOrderLeft = pPreOrder + 1;
    int* pInOrderLeft = pInOrder;
    root->left = rebuildTree(pPreOrderLeft, pInOrderLeft, nodeNumLeft);
    // 重建右子树
    int nodeNumRight = nodeNum - nodeNumLeft -1;
    int* pPreOrderRight = pPreOrder + 1 + nodeNumLeft;
    int* pInOrderRight = pInOrder + nodeNumLeft + 1;
    root->right = rebuildTree(pPreOrderRight, pInOrderRight, nodeNumRight);
    return root;
}
```

同样，有中序遍历序列和后序遍历序列，类似的方法可重建二叉树，但前序遍历序列和后序遍历序列不同恢复一棵二叉树

## <span id="nodeComplete">**判断二叉树是不是完全二叉树**</span>

若设二叉树的深度为h，除第h层外，其他各层(1~h-1)的节点数都达到最大个数，第h层所有的节点都连续集中在最左边，这就是完全二叉树。
如何判断是否为完全二叉树：
按层次(从上之下，从左至右)遍历二叉树，当遇到一个节点的左子树为空时，则该节点的右子树必须为空，且后面遍历的节点左右子树都必须为空，否则不是完全二叉树

参考代码：
```c++
bool isCompleteTree(TreeNode* root)
{
    if (!root) return false;
    queue<TreeNode*> nodeQueue;
    nodeQueue.push(root);
    bool mustHaveNoChild = false;
    bool result = true;
    while (!nodeQueue.empty()) {
        TreeNode* pNode = nodeQueue.front();
        nodeQueue.pop();
        if (mustHaveNoChild) {   // 已经出现了有空子树的节点了，后面的节点必须为叶子结点(没有孩子节点)
            if (pNode->left || pNode->right) {
                result = false;
                break;
            }
        }
        else {
            if (pNode->left && pNode->right) {
                nodeQueue.push(pNode->left);
                nodeQueue.push(pNode->right);
            }
            else if (pNode->left && !pNode->right) {
                mustHaveNoChild = true;
                nodeQueue.push(pNode->left);
            }
            else if (!pNode->left && pNode->right) {
                result = false;
                break;
            }
            else
                mastHaveNoChild = true;
        }
    }
    return result;
}
```
