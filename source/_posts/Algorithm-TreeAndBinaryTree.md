---
title: 树和二叉树
date: 2019-11-10 12:55:51
tags: Java
categories: 数据结构与算法
---

树(Tree)的基本概念
树是由结点或顶点和边组成的(可能是非线性的)且不存在着任何环的一种数据结构。没有结点的树称为空(null或empty)树。一棵非空的树包括一个根结点，还(很可能)有多个附加结点，所有结点构成一个多级分层结构

<!--more-->

## 二叉树

**每个结点至多拥有两棵子树(即二叉树中不存在度大于2的结点)，并且，二叉树的子树有左右之分，其次序不能任意颠倒**，二叉树具有一下性质

+ 二叉树在i层上的节点数目最多 $2^{i - 1}$ (i >= 1)

+ 深度为k的二叉树至多有 $2^{k} - 1$ 个节点 (k >= 1)

+ 由以上两点可推出叶子节点（度为0）数为m，度为2的节点数为n，则m = 1 + n

+ 包含n个结点的二叉树的高度至少为 $\log_2 (n + 1)$   

  

二叉树的遍历可根据根所在的位置顺序命名：

+ 前序遍历：根->左子树->右子树
+ 中序遍历：左子树->根->右子树
+ 后序遍历：左子树->右子树->根



### 完美二叉树

一个深度为k(>=1)且有 $2^{k} - 1$ 个结点的二叉树称为完美二叉树

![](https://s2.ax1x.com/2019/11/10/MuylVJ.png)

### 完全二叉树

换句话说，完全二叉树从根结点到倒数第二层满足完美二叉树，最后一层可以不完全填充，其叶子结点都靠左对齐

![](https://s2.ax1x.com/2019/11/10/MuyJ8x.png)

完全二叉树的应用：堆排序

对于完全二叉树的节点数n，我们根据从上到下、从左到右的顺序排序，从0开始，可轻松地算出它的父节点、孩子节点的位置。比如编号为i的节点：

+ 其父节点编号为$\frac{i - 1}{2}$
+ 如果$2 * i + 1 < n$，其左子树的根节点编号为$2 * i + 1$，否则没有左子树
+ 如果$2 * i + 2 < n$，其左子树的根节点编号为$2 * i + 2$，否则没有左子树



### 完满二叉树

换句话说，所有非叶子结点的度都是2（只要你有孩子，你就必然是有两个孩子） 

![](https://s2.ax1x.com/2019/11/10/Muydqe.png)

### 二叉查找树

**二叉查找树也称为有序二叉查找树,满足二叉查找树的一般性质,是指一棵空树具有如下性质：**

+ **任意节点左子树不为空,则左子树的值均小于根节点的值**
+ **任意节点右子树不为空,则右子树的值均大于于根节点的值**
+ **任意节点的左右子树也分别是二叉查找树**
+ **没有键值相等的节点**

#### 实现：

##### 构建

```Java
class Node
{
        Node parent;
        Node leftChild;
        Node rightChild;
        int val;
        public Node(Node parent, Node leftChild, Node rightChild,int val) 
        {
            super();
            this.parent = parent;
            this.leftChild = leftChild;
            this.rightChild = rightChild;
            this.val = val;
        }
    
        public Node(int val)
        {
            this(null,null,null,val);
        }
        
        public Node(Node node,int val)
        {
            this(node,null,null,val);
        }
}
```

##### 增加节点

递归

```Java
public boolean add(int val)
{
        if(root == null)
        {
            root = new Node(val);
            size++;
            return true;
        }
        Node node = getAdapterNode(root, val);
        Node newNode = new Node(val);
        if(node.val > val)
        {
            node.leftChild = newNode;
            newNode.parent = node;
        } else if(node.val < val)
        {
            node.rightChild = newNode;
            newNode.parent = node;
        } else
        {
            // 暂不做处理
        }
        size++;
        return true;
}
    
    /**
     * 获取要插入的节点的父节点，该父节点满足以下几种状态之一
     *  1、父节点为子节点
     *  2、插入节点值比父节点小，但父节点没有左子节点
     *  3、插入节点值比父节点大，但父节点没有右子节点
     *  4、插入节点值和父节点相等。
     *  5、父节点为空
     *  如果满足以上5种情况之一，则递归停止。
     * @param node
     * @param val
     * @return
     */
private Node getAdapterNode(Node node,int val)
{
    if(node == null)
    {
        return node;
    }
    // 往左子树中插入，但没左子树，则返回
    if(node.val > val && node.leftChild == null)
    {
        return node;
    }
    // 往右子树中插入，但没右子树，也返回
    if(node.val < val && node.rightChild == null)
    {
        return node;
    }
    // 该节点是叶子节点，则返回
    if(node.leftChild == null && node.rightChild == null)
    {
        return node;
    }

    if(node.val > val && node.leftChild != null)
    {
        return getAdaptarNode(node.leftChild, val);
    } else if(node.val < val && node.rightChild != null)
    {
        return getAdaptarNode(node.rightChild, val);
    } else
    {
        return node;
    }
}
```

迭代

```Java
public boolean put(int val)
{
        return putVal(root,val);
}
private boolean putVal(Node node,int val)
{
    if(node == null)
    {// 初始化根节点
        node = new Node(val);
        root = node;
        size++;
        return true;
    }
    Node temp = node;
    Node p;
    int t;
    /**
         * 通过do while循环迭代获取最佳节点，
         */
    do
    { 
        p = temp;
        t = temp.val-val;
        if(t > 0)
        {
            temp = temp.leftChild;
        } else if(t < 0)
        {
            temp = temp.rightChild;
        } else
        {
            temp.val = val;
            return false;
        }
    }while(temp != null);
    Node newNode = new Node(p, val);
    if(t > 0)
    {
        p.leftChild = newNode;
    } else if(t < 0)
    {
        p.rightChild = newNode;
    }
    size++;
    return true;
}
```

原理其实和递归一样，都是获取最佳节点，在该节点上进行操作。

论起性能，肯定迭代版本最佳，所以一般情况下，都是选择迭代版本进行操作数据。

#### 查找

查找比较简单

```Java
public Node getNode(int val)
{
        Node temp = root;
        int t;
        do
        {
            t = temp.val-val;
            if(t > 0)
            {
                temp = temp.leftChild;
            } else if(t < 0)
            {
                temp = temp.rightChild;
            } else
            {
                return temp;
            }
        } while(temp != null);
        return null;
}
```



#### 删除

可以说在二叉搜索树的操作中，删除是最复杂的，要考虑的情况也相对多，在常规思路中，删除二叉搜索树的某一个节点，肯定会想到以下四种情况

![](https://s2.ax1x.com/2019/11/16/MB86OA.png)



+ 要删除的节点没有左右子节点，如上图的D、E、G节点
+ 要删除的节点只有左子节点，如B节点
+ 要删除的节点只有右子节点，如F节点
+ 要删除的节点既有左子节点，又有右子节点，如 A、C节点

先来分析第一种比如 删除D节点，则可以将B节点的左子节点设置为null，若删除G节点，则可将F节点的右子节点设置为null

第二种，删除B节点，则只需将A节点的左节点设置成D节点，将D节点的父节点设置成A即可。具体设置哪一边，也是看删除的节点位于父节点的哪一边

第三种，同第二种

第四种，比如要删除C节点，将F节点的父节点设置成A节点，F节点左节点设置成E节点，将A的右节点设置成F，E的父节点设置F节点(也就是将F节点替换C节点)，还有一种，直接将E节点替换C节点。那采用哪一种呢，如果删除节点为根节点，又该怎么删除？对于第四种情况，可以这样想，找到C或者A节点的后继节点，删除后继节点，且将后继节点的值设置为C或A节点的值。先来补充下后继节点的概念。一个节点在整棵树中的后继节点必满足，大于该节点值得所有节点集合中值最小的那个节点，即为后继节点，当然，也有可能不存在后继节点。但是对于第四种情况，后继节点一定存在，且一定在其右子树中，而且还满足，只有一个子节点或者没有子节点两者情况之一。具体原因可以这样想，因为后继节点要比C节点大，又因为C节点左右子节一定存在，所以一定存在右子树中的左子节点中。就比如C的后继节点是F，A的后继节点是E。

```Java
public boolean delete(int val)
{
    Node node = getNode(val);
    if(node == null)
    {
        return false;
    }
    Node parent = node.parent;
    Node leftChild = node.leftChild;
    Node rightChild = node.rightChild;
    //以下所有父节点为空的情况，则表明删除的节点是根节点
    if(leftChild == null && rightChild == null)
    {//没有子节点
        if(parent != null)
        {
            if(parent.leftChild == node)
            {
                parent.leftChild = null;
            } else if(parent.rightChild == node)
            {
                parent.rightChild = null;
            }
        } else
        {
            //不存在父节点，则表明删除节点为根节点
            root = null;
        }
        node = null;
        return true;
    } else if(leftChild == null && rightChild != null)
    {
        // 只有右节点
        if(parent != null && parent.val > val)
        {
            // 存在父节点，且node位置为父节点的左边
            parent.leftChild = rightChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父节点，且node位置为父节点的右边
            parent.rightChild = rightChild;
        } else
        {
            root = rightChild;
        }
        node = null;
        return true;
    } else if(leftChild != null && rightChild == null)
    {
        // 只有左节点
        if(parent != null && parent.val > val)
        {
            // 存在父节点，且node位置为父节点的左边
            parent.leftChild = leftChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父节点，且node位置为父节点的右边
            parent.rightChild = leftChild;
        } else
        {
            root = leftChild;
        }
        return true;
    } else if(leftChild != null && rightChild != null)
    {
        // 两个子节点都存在
        Node successor = getSuccessor(node);// 这种情况，一定存在后继节点
        int temp = successor.val;
        boolean delete = delete(temp);
        if(delete)
        {
            node.val = temp;
        }
        successor = null;
        return true;
    }
    return false;
}
    
/**
     * 找到node节点的后继节点
     * 1、先判断该节点有没有右子树，如果有，则从右节点的左子树中寻找后继节点，没有则进行下一步
     * 2、查找该节点的父节点，若该父节点的右节点等于该节点，则继续寻找父节点，
     *   直至父节点为Null或找到不等于该节点的右节点。
     * 理由，后继节点一定比该节点大，若存在右子树，则后继节点一定存在右子树中，这是第一步的理由
     *      若不存在右子树，则也可能存在该节点的某个祖父节点(即该节点的父节点，或更上层父节点)的右子树中，
     *      对其迭代查找，若有，则返回该节点，没有则返回null
     * @param node
     * @return
     */
private Node getSuccessor(Node node)
{
    if(node.rightChild != null)
    {
        Node rightChild = node.rightChild;
        while(rightChild.leftChild != null)
        {
            rightChild = rightChild.leftChild;
        }
        return rightChild;
    }
    Node parent = node.parent;
    while(parent != null && (node == parent.rightChild))
    {
        node = parent;
        parent = parent.parent;
    }
    return parent;
}
```

除了上述实现，在算法导论书中，提供了另外一种实现

```Java
public boolean remove(int val)
{
    Node node = getNode(val);
    if(node == null)
    {
        return false;
    }
    if(node.leftChild == null)
    {
        // 1、左节点不存在，右节点可能存在，包含两种情况  ，两个节点都不存在和只存在右节点
        transplant(node, node.rightChild);
    } else if(node.rightChild == null)
    {
        //2、左孩子存在，右节点不存在
        transplant(node, node.leftChild);
    } else
    {
        // 3、两个节点都存在
        Node successor = getSuccessor(node);// 得到node后继节点 
        if(successor.parent != node)
        {
            // 后继节点存在node的右子树中。
            transplant(successor, successor.rightChild);// 用后继节点的右子节点替换该后继节点
            successor.rightChild = node.rightChild;// 将node节点的右子树赋给后继节点的右节点，即类似后继与node节点调换位置
            successor.rightChild.parent = successor;// 接着上一步  给接过来的右节点的父引用复制
        }
        transplant(node, successor);
        successor.leftChild = node.leftChild;
        successor.leftChild.parent = successor;
    }
    return true;
}
/**
     * 将child节点替换node节点
     * @param root    根节点
     * @param node    要删除的节点
     * @param child   node节点的子节点
     */
private void transplant(Node node,Node child)
{
    /**
         * 1、先判断 node是否存在父节点
         *    1、不存在，则child替换为根节点
         *    2、存在，则继续下一步
         * 2、判断node节点是父节点的那个孩子(即判断出 node是右节点还是左节点)，
         *    得出结果后，将child节点替换node节点 ，即若node节点是左节点 则child替换后 也为左节点，否则为右节点
         * 3、将node节点的父节点置为child节点的父节点
         */

    if(node.parent == null)
    {
        this.root = child;
    } else if(node.parent.leftChild == node)
    {
        node.parent.leftChild = child;
    } else if(node.parent.rightChild == node)
    {
        node.parent.rightChild = child;
    }
    if(child != null)
    {
        child.parent = node.parent;
    }
}
```



#### 局限性及应用：

**一个二叉查找树是由n个节点随机构成,所以，对于某些情况,二叉查找树会退化成一个有n个节点的线性链.如下图:**

![](https://s2.ax1x.com/2019/11/10/MuWoi4.png)

### 平衡二叉树（AVL树）

AVL树是带有平衡条件的二叉查找树，和红黑树相比,它是严格的平衡二叉树,平衡条件必须满足(所有节点的左右子树高度差不超过1).不管我们是执行插入还是删除操作,只要不满足上面的条件,就要通过旋转来保持平衡,而旋转是非常耗时的

#### 使用场景

**AVL树适合用于插入删除次数比较少，但查找多的情况，也在Windows进程地址空间管理中得到了使用，旋转的目的是为了降低树的高度，使其平衡**

```Java
class Node
{
        Node parent;
        Node leftChild;
        Node rightChild;
        int val;
        public Node(Node parent, Node leftChild, Node rightChild,int val) 
        {
            super();
            this.parent = parent;
            this.leftChild = leftChild;
            this.rightChild = rightChild;
            this.val = val;
        }
        
        public Node(int val)
        {
            this(null,null,null,val);
        }
        
        public Node(Node node,int val)
        {
            this(node,null,null,val);
        }
}
```



#### 插入

在谈及平衡二叉树的增加，先来考虑什么样的情况会打破这个平衡，假如A树已经是一颗平衡二叉树，但现在要往里面插入一个元素，有这两种结果，一、平衡未打破，这种肯定皆大欢喜，二、平衡被打破了。那一般要考虑三个问题

+ 平衡被打破之前是什么状态？
+ 被打破之后又是一个什么样的状态？
+ 平衡被打破了，该怎么调整，使它又重新成为一个平衡二叉树呢？

这里截取打破平衡后左子树的高度比右子树高度高2的所有可能情况(若右子树高，情况一样，这里只选取一种分析)，下面的图，只是代表着那个被打破平衡的子树(被打破平衡的点就是这个节点的左右子树高度差的绝对值大于或等于2，当然，这里只能等于2)，不代表整棵树。



这是第一种情况，其中A节点和B节点只是平衡二叉树的某一个子集合，要想打破这个平衡，那么插入的节点C必然在B的子节点上，即左右子节点

![](https://s2.ax1x.com/2019/11/12/M8ANBF.png)

要使A节点的左右子树差的绝对值小于2，此时只需将B节点来替换A节点，A节点成为B节点的右孩子。若A节点有父节点，则A的父节点的子节点要去指向B节点，而A节点的父节点要去指向B节点，这段操作也就是右旋操作

![](https://s2.ax1x.com/2019/11/12/M8eBVO.png)

```Java
public Node leftRotation(Node aNode)
{
    if(aNode != null)
    {
        Node bNode = aNode.leftChild;// 先用一个变量来存储B节点
        bNode.parent = aNode.parent;// 重新分配A节点的父节点指向
        //判断A节点的父节点是否存在
        if(aNode.parent != null)
        {
            // A节点不是根节点
            /**
                 * 分两种情况
                 *   1、A节点位于其父节点左边，则B节点也要位于左边
                 *   2、A节点位于其父节点右边，则B节点也要位于右边
                 */
            if(aNode.parent.leftChild == aNode)
            {
                aNode.parent.leftChild = bNode;
            } else
            {
                aNode.parent.rightChild = bNode;
            }
        }else
        {
            // 说明A节点是根节点，直接将B节点置为根节点
            this.root = bNode;
        }
        bNode.rightChild = aNode;// 将B节点的右孩子置为A节点
        aNode.parent = bNode;// 将A节点的父节点置为B节点
        return bNode;// 返回旋转的节点
    }
    return null;
}
```

以上代码无法应对另一种情况：

![](https://s2.ax1x.com/2019/11/12/M8ucRJ.png)

如果将C节点替换B节点位置，而B节点成为C节点的左节点，这样就成为了上一段代码的那种情况。这段B节点替换成为C节点的代码如下，这里操作也就是，先左旋后右旋

```Java
public Node rightRotation(Node bNode)
{
    if(bNode != null)
    {
        Node cNode = bNode.rightChild;// 用临时变量存储C节点
        cNode.parent = bNode.parent;
        // 这里因为bNode节点父节点存在，所以不需要判断。加判断也行,
        if(bNode.parent.rightChild == bNode)
        {
            bNode.parent.rightChild = cNode;
        } else
        {
            bNode.parent.leftChild = cNode;
        }
        cNode.leftChild = bNode;
        bNode.parent = cNode;
        return cNode;
    }
    return null;
}
```

将两种情况综合一下

```Java
public Node rightRotation(Node node)
{
    if(node != null)
    {
        Node leftChild = node.leftChild;// 用变量存储node节点的左子节点
        node.leftChild = leftChild.rightChild;// 将leftChild节点的右子节点赋值给node节点的左节点
        if(leftChild.rightChild != null)
        {
            // 如果leftChild的右节点存在，则需将该右节点的父节点指给node节点
            leftChild.rightChild.parent = node;
        }
        leftChild.parent = node.parent;
        if(node.parent == null)
        {
            // 即表明node节点为根节点
            this.root = leftChild;
        } else if(node.parent.rightChild == node)
        {
            // 即node节点在它原父节点的右子树中
            node.parent.rightChild = leftChild;
        } else if(node.parent.leftChild == node)
         
            node.parent.leftChild = leftChild;
        }
        leftChild.rightChild = node;
        node.parent = leftChild;
        return leftChild;
    }
    return null;
}

public Node leftRotation(Node node)
{
    if(node != null)
    {
        Node rightChild = node.rightChild;
        node.rightChild = rightChild.leftChild;
        if(rightChild.leftChild != null)
        {
            rightChild.leftChild.parent = node;
        }
        rightChild.parent = node.parent;
        if(node.parent == null)
        {
            this.root = rightChild;
        } else if(node.parent.rightChild == node)
        {
            node.parent.rightChild = rightChild;
        } else if(node.parent.leftChild == node)
        {
            node.parent.leftChild = rightChild;
        }
        rightChild.leftChild = node;
        node.parent = rightChild;
        return rightChild;
    }
    return null;
}
```



![](https://s2.ax1x.com/2019/11/12/M8l3Os.png)







这是第二种情况，其中A、B、C、D、E五个节点也是该平衡树的某个子集合，同样要打破这个平衡，那么，插入的节点F必然在D节点和E节点上。

![](https://s2.ax1x.com/2019/11/12/M8VBy6.png)



![](https://s2.ax1x.com/2019/11/17/Mrn5h4.png)



![](https://s2.ax1x.com/2019/11/17/MrMo9g.png)



至此，打破平衡后，经过一系列操作达到平衡，由以上可知，大致有以下四种操作情况:

+ 只需要经过一次右旋即可达到平衡
+ 只需要经过一次左旋即可达到平衡
+ 需先经过左旋，再经过右旋也可达到平衡
+ 需先经过右旋，再经过左旋也可达到平衡

那问题就来了，怎么判断被打破的平衡要经历哪种操作才能达到平衡呢？

经过了解，这四种情况，还可大致分为两大类，如下(以下的A节点就是被打破平衡的那个节点)

第一大类，A节点的左子树高度比右子树高度高2，最终需要经过右旋操作(可能需要先左后右)

第二大类，A节点的左子树高度比右子树高度低2，最终需要经过左旋操作(可能需要先右后左)

所以很容易想到，在插入节点后，判断插入的节点是在A节点的左子树还是右子树（因为插入之前已经是平衡二叉树）再决定采用哪个大类操作，在大类操作里再去细分要不要经历两步操作。

```Java
public boolean put(int val)
{
    return putVal(root,val);
}
private boolean putVal(Node node,int val)
{
    if(node == null)
    {
        // 初始化根节点
        node = new Node(val);
        root = node;
        size++;
        return true;
    }
    Node temp = node;
    Node p;
    int t;
    /**
         * 通过do while循环迭代获取最佳节点，
         */
    do
    { 
        p = temp;
        t = temp.val-val;
        if(t > 0)
        {
            temp = temp.leftChild;
        } else if(t < 0)
        {
            temp = temp.rightChild;
        } else
        {
            temp.val = val;
            return false;
        }
    } while(temp != null);
    Node newNode = new Node(p, val);
    if(t > 0)
    {
        p.leftChild = newNode;
    } else if(t < 0)
    {
        p.rightChild = newNode;
    }
    rebuild(p);// 使二叉树平衡的方法
    size++;
    return true;
}
private void rebuild(Node p)
{
    while(p != null)
    {
        if(calcNodeBalanceValue(p) == 2)
        {
            // 说明左子树高，需要右旋或者 先左旋后右旋
            fixAfterInsertion(p,LEFT);// 调整操作
        } else if(calcNodeBalanceValue(p) == -2)
        {
            fixAfterInsertion(p,RIGHT);
        }
        p = p.parent;
    }
}
private int calcNodeBalanceValue(Node node)
{
    if(node != null)
    {
        return getHeightByNode(node);
    }
    return 0;
}
// 计算node节点的高度
public int getChildDepth(Node node)
{
    if(node == null)
    {
        return 0;
    }
    return 1+Math.max(getChildDepth(node.leftChild),getChildDepth(node.rightChild));
}
public int getHeightByNode(Node node)
{
    if(node == null)
    {
        return 0;
    }
    return getChildDepth(node.leftChild)-getChildDepth(node.rightChild);
}
private void fixAfterInsertion(Node p, int type) 
{
    if(type == 0)
    {
        //需要右旋或者 先左旋后右旋
        final Node leftChild = p.leftChild;
        if(leftChild.rightChild != null && leftChild.leftChild == null)
        {
            // 先左旋后右旋
            leftRotation(leftChild);
            rightRotation(p);
        } else if(leftChild.leftChild != null && leftChild.rightChild == null)
        {
            //右旋
            rightRotation(p);
        } else // 第二种情况下
        {
            if (getChildDepth(leftChild.leftChild) > getChildDepth(leftChild.rightChild))
                rightRotation(p);
            else
            {
                leftRotation(leftChild);
            	rightRotation(p);
            }
        }
    } else
    {
        final Node rightChild = p.rightChild;
        if(rightChild.leftChild != null && rightChild.rightChild == null)
        {
            // 先右旋，后左旋
            rightRotation(rightChild);
            leftRotation(p);
        } else if(rightChild.rightChild != null && rightChild.leftChild == null)
        { 
            // 左旋
            leftRotation(p);
        } else
        {
            if (getChildDepth(rightChild.leftChild) < getChildDepth(rightChild.rightChild))
                leftRotation(p);
            else
            {
                rightRotation(leftChild);
            	leftRotation(p);
            }
        }
    }
}

```

#### 删除

```Java
public boolean delete(int val)
{
    Node node = getNode(val);
    if(node == null){
        return false;
    }
    boolean flag = false;
    Node p = null;
    Node parent = node.parent;
    Node leftChild = node.leftChild;
    Node rightChild = node.rightChild;
    //以下所有父节点为空的情况，则表明删除的节点是根节点
    if(leftChild == null && rightChild == null)
    {
        //没有子节点
        if(parent != null)
        {
            if(parent.leftChild == node)
            {
                parent.leftChild = null;
            } else if(parent.rightChild == node)
            {
                parent.rightChild = null;
            }
        } else
        {
            //不存在父节点，则表明删除节点为根节点
            root = null;
        }
        p = parent;
        node = null;
        flag =  true;
    } else if(leftChild == null && rightChild != null)
    {
        // 只有右节点
        if(parent != null && parent.val > val)
        {
            // 存在父节点，且node位置为父节点的左边
            parent.leftChild = rightChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父节点，且node位置为父节点的右边
            parent.rightChild = rightChild;
        } else
        {
            root = rightChild;
        }
        p = parent;
        node = null;
        flag =  true;
    } else if(leftChild != null && rightChild == null)
    {
        // 只有左节点
        if(parent != null && parent.val > val)
        {
            // 存在父节点，且node位置为父节点的左边
            parent.leftChild = leftChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父节点，且node位置为父节点的右边
            parent.rightChild = leftChild;
        } else
        {
            root = leftChild;
        }
        p = parent;
        flag =  true;
    } else if(leftChild != null && rightChild != null)
    {
        // 两个子节点都存在
        Node successor = getSuccessor(node);// 这种情况，一定存在后继节点
        int temp = successor.val;
        boolean delete = delete(temp);
        if(delete)
        {
            node.val = temp;
        }
        p = successor;
        successor = null;
        flag =  true;
    }
    if(flag)
    {
        rebuild(p);
    }
    return flag;
}
```

