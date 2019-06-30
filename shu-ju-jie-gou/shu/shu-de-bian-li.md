# 树的遍历

## 前序遍历算法

```

void PreOderTraverse(BiTree T)
{
    if(T == NULL)
        return;
    printf("%c",T->data);  //显示结点数据，可以更改为其他对结点操作
    PreOderTraverse(T->lchild);   //先遍历左子树
    PreOderTraverse(T->rchild);    //最后遍历右子树 
} 
```

 ## 中序遍历递归算法

```
 /*中序遍历递归算法*/
void InOderTraverse(BiTree T)
{
    if(T == NULL)
        return ;
    InOderTraverse(T->lchild);   //中序遍历左子树
    printf("%c",T->data);   //显示结点数据，可以更改为其他对结点的操作
    InOderTraverse(T->rchild);  //最后中序遍历右子树 
} 
```



 ## 后序遍历递归算法
```

void PostOderTraverse(T)
{
    if(T==NULL)
        return;
    PostOderTraverse(T->lchild);   //先遍历左子树 
    PostsOderTraverse(T->rchild);  //再遍历右子树 
    printf("%c",T->data);    //显示结点数可以更改为其他对结点数据 
} 
```







  通过代码可以看出，二叉树的遍历过程使用递归方式实现既有助于理解，又简化了代码量。