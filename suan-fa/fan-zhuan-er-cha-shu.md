# 翻转二叉树

输入一个二叉树，输出其镜像（交换所有节点的左右子树）。


  1                1
 / \                / \
2   3   =>    3   2
   /                \
  4                4

定义二叉树结点：

```c
struct NodeBinaryTree
{
    int data;
    NodeBinaryTree * pLeft;
    NodeBinaryTree * pRight;
};
```



递归算法：

```c
NodeBinaryTree * invertBinaryTreeRecursive( NodeBinaryTree * root )
{
    if( NULL == root )
        return root;

    NodeBinaryTree * pTmp = root->pLeft;
    root->pLeft = invertBinaryTreeRecursive( root->pRight );
    root->pRight = invertBinaryTreeRecursive( pTmp );

    return root;
}
```



非递归算法：

```c
NodeBinaryTree * invertBinaryTree( NodeBinaryTree * root )
{
    if( NULL == root )
        return root;

    queue<NodeBinaryTree *> tree_queue;

    tree_queue.push( root );
    while( tree_queue.size() > 0 )
    {
        NodeBinaryTree * pNode = tree_queue.front(); // 取队列中第一个元素
        tree_queue.pop(); // 弹出要处理的元素

        // 翻转左右子树
        NodeBinaryTree * pTmp = pNode->pLeft; 
        pNode->pLeft = pNode->pRight;
        pNode->pRight = pTmp;

        // 判断左右节点是否存在，存在则入队列。
        if( NULL != pNode->pLeft )
            tree_queue.push( pNode->pLeft );
        if( NULL != pNode->pRight )
            tree_queue.push( pNode->pRight );
    }

    return root;
}
```



