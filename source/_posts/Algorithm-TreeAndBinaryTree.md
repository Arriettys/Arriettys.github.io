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

+ 二叉树在i层上的结点数目最多 $2^{i - 1}$ (i >= 1)

+ 深度为k的二叉树至多有 $2^{k} - 1$ 个结点 (k >= 1)

+ 由以上两点可推出叶子结点（度为0）数为m，度为2的结点数为n，则m = 1 + n

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

对于完全二叉树的结点数n，我们根据从上到下、从左到右的顺序排序，从0开始，可轻松地算出它的父结点、孩子结点的位置。比如编号为i的结点：

+ 其父结点编号为$\frac{i - 1}{2}$
+ 如果$2 * i + 1 < n$，其左子树的根结点编号为$2 * i + 1$，否则没有左子树
+ 如果$2 * i + 2 < n$，其左子树的根结点编号为$2 * i + 2$，否则没有左子树



### 完满二叉树

换句话说，所有非叶子结点的度都是2（只要你有孩子，你就必然是有两个孩子） 

![](https://s2.ax1x.com/2019/11/10/Muydqe.png)

### 二叉查找树

**二叉查找树也称为有序二叉查找树,满足二叉查找树的一般性质,是指一棵空树具有如下性质：**

+ **任意结点左子树不为空,则左子树的值均小于根结点的值**
+ **任意结点右子树不为空,则右子树的值均大于于根结点的值**
+ **任意结点的左右子树也分别是二叉查找树**
+ **没有键值相等的结点**

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

##### 增加结点

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
     * 获取要插入的结点的父结点，该父结点满足以下几种状态之一
     *  1、父结点为子结点
     *  2、插入结点值比父结点小，但父结点没有左子结点
     *  3、插入结点值比父结点大，但父结点没有右子结点
     *  4、插入结点值和父结点相等。
     *  5、父结点为空
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
    // 该结点是叶子结点，则返回
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
    {// 初始化根结点
        node = new Node(val);
        root = node;
        size++;
        return true;
    }
    Node temp = node;
    Node p;
    int t;
    /**
         * 通过do while循环迭代获取最佳结点，
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

原理其实和递归一样，都是获取最佳结点，在该结点上进行操作。

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

可以说在二叉搜索树的操作中，删除是最复杂的，要考虑的情况也相对多，在常规思路中，删除二叉搜索树的某一个结点，肯定会想到以下四种情况

![](https://s2.ax1x.com/2019/11/16/MB86OA.png)



+ 要删除的结点没有左右子结点，如上图的D、E、G结点
+ 要删除的结点只有左子结点，如B结点
+ 要删除的结点只有右子结点，如F结点
+ 要删除的结点既有左子结点，又有右子结点，如 A、C结点

先来分析第一种比如 删除D结点，则可以将B结点的左子结点设置为null，若删除G结点，则可将F结点的右子结点设置为null

第二种，删除B结点，则只需将A结点的左结点设置成D结点，将D结点的父结点设置成A即可。具体设置哪一边，也是看删除的结点位于父结点的哪一边

第三种，同第二种

第四种，比如要删除C结点，将F结点的父结点设置成A结点，F结点左结点设置成E结点，将A的右结点设置成F，E的父结点设置F结点(也就是将F结点替换C结点)，还有一种，直接将E结点替换C结点。那采用哪一种呢，如果删除结点为根结点，又该怎么删除？对于第四种情况，可以这样想，找到C或者A结点的后继结点，删除后继结点，且将后继结点的值设置为C或A结点的值。先来补充下后继结点的概念。一个结点在整棵树中的后继结点必满足，大于该结点值得所有结点集合中值最小的那个结点，即为后继结点，当然，也有可能不存在后继结点。但是对于第四种情况，后继结点一定存在，且一定在其右子树中，而且还满足，只有一个子结点或者没有子结点两者情况之一。具体原因可以这样想，因为后继结点要比C结点大，又因为C结点左右子节一定存在，所以一定存在右子树中的左子结点中。就比如C的后继结点是F，A的后继结点是E。

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
    //以下所有父结点为空的情况，则表明删除的结点是根结点
    if(leftChild == null && rightChild == null)
    {//没有子结点
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
            //不存在父结点，则表明删除结点为根结点
            root = null;
        }
        node = null;
        return true;
    } else if(leftChild == null && rightChild != null)
    {
        // 只有右结点
        if(parent != null && parent.val > val)
        {
            // 存在父结点，且node位置为父结点的左边
            parent.leftChild = rightChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父结点，且node位置为父结点的右边
            parent.rightChild = rightChild;
        } else
        {
            root = rightChild;
        }
        node = null;
        return true;
    } else if(leftChild != null && rightChild == null)
    {
        // 只有左结点
        if(parent != null && parent.val > val)
        {
            // 存在父结点，且node位置为父结点的左边
            parent.leftChild = leftChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父结点，且node位置为父结点的右边
            parent.rightChild = leftChild;
        } else
        {
            root = leftChild;
        }
        return true;
    } else if(leftChild != null && rightChild != null)
    {
        // 两个子结点都存在
        Node successor = getSuccessor(node);// 这种情况，一定存在后继结点
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
     * 找到node结点的后继结点
     * 1、先判断该结点有没有右子树，如果有，则从右结点的左子树中寻找后继结点，没有则进行下一步
     * 2、查找该结点的父结点，若该父结点的右结点等于该结点，则继续寻找父结点，
     *   直至父结点为Null或找到不等于该结点的右结点。
     * 理由，后继结点一定比该结点大，若存在右子树，则后继结点一定存在右子树中，这是第一步的理由
     *      若不存在右子树，则也可能存在该结点的某个祖父结点(即该结点的父结点，或更上层父结点)的右子树中，
     *      对其迭代查找，若有，则返回该结点，没有则返回null
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
        // 1、左结点不存在，右结点可能存在，包含两种情况  ，两个结点都不存在和只存在右结点
        transplant(node, node.rightChild);
    } else if(node.rightChild == null)
    {
        //2、左孩子存在，右结点不存在
        transplant(node, node.leftChild);
    } else
    {
        // 3、两个结点都存在
        Node successor = getSuccessor(node);// 得到node后继结点 
        if(successor.parent != node)
        {
            // 后继结点存在node的右子树中。
            transplant(successor, successor.rightChild);// 用后继结点的右子结点替换该后继结点
            successor.rightChild = node.rightChild;// 将node结点的右子树赋给后继结点的右结点，即类似后继与node结点调换位置
            successor.rightChild.parent = successor;// 接着上一步  给接过来的右结点的父引用复制
        }
        transplant(node, successor);
        successor.leftChild = node.leftChild;
        successor.leftChild.parent = successor;
    }
    return true;
}
/**
     * 将child结点替换node结点
     * @param root    根结点
     * @param node    要删除的结点
     * @param child   node结点的子结点
     */
private void transplant(Node node,Node child)
{
    /**
         * 1、先判断 node是否存在父结点
         *    1、不存在，则child替换为根结点
         *    2、存在，则继续下一步
         * 2、判断node结点是父结点的那个孩子(即判断出 node是右结点还是左结点)，
         *    得出结果后，将child结点替换node结点 ，即若node结点是左结点 则child替换后 也为左结点，否则为右结点
         * 3、将node结点的父结点置为child结点的父结点
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

**一个二叉查找树是由n个结点随机构成,所以，对于某些情况,二叉查找树会退化成一个有n个结点的线性链.如下图:**

![](https://s2.ax1x.com/2019/11/10/MuWoi4.png)

### 平衡二叉树（AVL树）

AVL树是带有平衡条件的二叉查找树，和红黑树相比,它是严格的平衡二叉树,平衡条件必须满足(所有结点的左右子树高度差不超过1).不管我们是执行插入还是删除操作,只要不满足上面的条件,就要通过旋转来保持平衡,而旋转是非常耗时的

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

这里截取打破平衡后左子树的高度比右子树高度高2的所有可能情况(若右子树高，情况一样，这里只选取一种分析)，下面的图，只是代表着那个被打破平衡的子树(被打破平衡的点就是这个结点的左右子树高度差的绝对值大于或等于2，当然，这里只能等于2)，不代表整棵树。



这是第一种情况，其中A结点和B结点只是平衡二叉树的某一个子集合，要想打破这个平衡，那么插入的结点C必然在B的子结点上，即左右子结点

![](https://s2.ax1x.com/2019/11/12/M8ANBF.png)

要使A结点的左右子树差的绝对值小于2，此时只需将B结点来替换A结点，A结点成为B结点的右孩子。若A结点有父结点，则A的父结点的子结点要去指向B结点，而A结点的父结点要去指向B结点，这段操作也就是右旋操作

![](https://s2.ax1x.com/2019/11/12/M8eBVO.png)

```Java
public Node leftRotation(Node aNode)
{
    if(aNode != null)
    {
        Node bNode = aNode.leftChild;// 先用一个变量来存储B结点
        bNode.parent = aNode.parent;// 重新分配A结点的父结点指向
        //判断A结点的父结点是否存在
        if(aNode.parent != null)
        {
            // A结点不是根结点
            /**
                 * 分两种情况
                 *   1、A结点位于其父结点左边，则B结点也要位于左边
                 *   2、A结点位于其父结点右边，则B结点也要位于右边
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
            // 说明A结点是根结点，直接将B结点置为根结点
            this.root = bNode;
        }
        bNode.rightChild = aNode;// 将B结点的右孩子置为A结点
        aNode.parent = bNode;// 将A结点的父结点置为B结点
        return bNode;// 返回旋转的结点
    }
    return null;
}
```

以上代码无法应对另一种情况：

![](https://s2.ax1x.com/2019/11/12/M8ucRJ.png)

如果将C结点替换B结点位置，而B结点成为C结点的左结点，这样就成为了上一段代码的那种情况。这段B结点替换成为C结点的代码如下，这里操作也就是，先左旋后右旋

```Java
public Node rightRotation(Node bNode)
{
    if(bNode != null)
    {
        Node cNode = bNode.rightChild;// 用临时变量存储C结点
        cNode.parent = bNode.parent;
        // 这里因为bNode结点父结点存在，所以不需要判断。加判断也行,
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
        Node leftChild = node.leftChild;// 用变量存储node结点的左子结点
        node.leftChild = leftChild.rightChild;// 将leftChild结点的右子结点赋值给node结点的左结点
        if(leftChild.rightChild != null)
        {
            // 如果leftChild的右结点存在，则需将该右结点的父结点指给node结点
            leftChild.rightChild.parent = node;
        }
        leftChild.parent = node.parent;
        if(node.parent == null)
        {
            // 即表明node结点为根结点
            this.root = leftChild;
        } else if(node.parent.rightChild == node)
        {
            // 即node结点在它原父结点的右子树中
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







这是第二种情况，其中A、B、C、D、E五个结点也是该平衡树的某个子集合，同样要打破这个平衡，那么，插入的结点F必然在D结点和E结点上。

![](https://s2.ax1x.com/2019/11/12/M8VBy6.png)



![](https://s2.ax1x.com/2019/11/17/Mrn5h4.png)



![](https://s2.ax1x.com/2019/11/17/MrMo9g.png)



至此，打破平衡后，经过一系列操作达到平衡，由以上可知，大致有以下四种操作情况:

+ 只需要经过一次右旋即可达到平衡
+ 只需要经过一次左旋即可达到平衡
+ 需先经过左旋，再经过右旋也可达到平衡
+ 需先经过右旋，再经过左旋也可达到平衡

那问题就来了，怎么判断被打破的平衡要经历哪种操作才能达到平衡呢？

经过了解，这四种情况，还可大致分为两大类，如下(以下的A结点就是被打破平衡的那个结点)

第一大类，A结点的左子树高度比右子树高度高2，最终需要经过右旋操作(可能需要先左后右)

第二大类，A结点的左子树高度比右子树高度低2，最终需要经过左旋操作(可能需要先右后左)

所以很容易想到，在插入结点后，判断插入的结点是在A结点的左子树还是右子树（因为插入之前已经是平衡二叉树）再决定采用哪个大类操作，在大类操作里再去细分要不要经历两步操作。

```Java
public boolean put(int val)
{
    return putVal(root,val);
}
private boolean putVal(Node node,int val)
{
    if(node == null)
    {
        // 初始化根结点
        node = new Node(val);
        root = node;
        size++;
        return true;
    }
    Node temp = node;
    Node p;
    int t;
    /**
         * 通过do while循环迭代获取最佳结点，
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
// 计算node结点的高度
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
    //以下所有父结点为空的情况，则表明删除的结点是根结点
    if(leftChild == null && rightChild == null)
    {
        //没有子结点
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
            //不存在父结点，则表明删除结点为根结点
            root = null;
        }
        p = parent;
        node = null;
        flag =  true;
    } else if(leftChild == null && rightChild != null)
    {
        // 只有右结点
        if(parent != null && parent.val > val)
        {
            // 存在父结点，且node位置为父结点的左边
            parent.leftChild = rightChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父结点，且node位置为父结点的右边
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
        // 只有左结点
        if(parent != null && parent.val > val)
        {
            // 存在父结点，且node位置为父结点的左边
            parent.leftChild = leftChild;
        } else if(parent != null && parent.val < val)
        {
            // 存在父结点，且node位置为父结点的右边
            parent.rightChild = leftChild;
        } else
        {
            root = leftChild;
        }
        p = parent;
        flag =  true;
    } else if(leftChild != null && rightChild != null)
    {
        // 两个子结点都存在
        Node successor = getSuccessor(node);// 这种情况，一定存在后继结点
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



### 红黑树

红黑树(Red-Black Tree，简称R-B Tree)，它一种特殊的二叉查找树。
红黑树是一棵二叉搜索树，它在每个结点增加了一个存储位记录结点的颜色，可以是RED,也可以是BLACK；通过任意一条从根到叶子简单路径上颜色的约束，<font color=red>红黑树保证最长路径不超过最短路径的二倍，因而近似平衡</font>。

![](https://s2.ax1x.com/2019/11/22/M7hagP.jpg)

#### 特性:

+ 每个结点或者是黑色，或者是红色。

+ 根结点是黑色。
+ 每个叶子结点是黑色。 [注意：这里叶子结点，是指为空的叶子结点！]
+ 如果一个结点是红色的，则它的子结点必须是黑色的 <font color=red>（没有连续的红结点）</font>
+ 从一个结点到该结点的子孙结点的所有路径上包含相同数目的黑结点。

那么为什么当满足以上性质时，就能保证最长路径不超过最短路径的二倍了呢？我们分析一下：

![](https://s2.ax1x.com/2019/11/23/MHg77j.png)

你的最短路径就是全黑结点，最长路径就是一个红结点一个黑结点，最后黑色结点相同时，最长路径刚好是最短路径的两倍

#### 证明

**红黑树的时间复杂度为：O($\log_2 n$)**

定理：**一颗含有n个结点的红黑树的高度之多为$2\log_2 (n + 1)$**

证明：

 "一棵含有n个结点的红黑树的高度至多为$2\log_2(n + 1)$" 的**逆否命题**是 "高度为h的红黑树，它的包含的内结点个数至少为$2^{\frac{h}{2}} - 1$个"。
我们只需要证明逆否命题，即可证明原命题为真；即只需证明 "高度为h的红黑树，它的包含的内结点个数至少为 $2^{\frac{h}{2}} - 1$个"。

  从某个结点x出发（不包括该结点）到达一个叶结点的任意一条路径上，黑色结点的个数称为该结点的黑高度(x's black height)，记为**bh(x)**。关于bh(x)有两点需要说明： 
    第1点：根据红黑树的"**特性(5)** ，即*从一个结点到该结点的子孙结点的所有路径上包含相同数目的黑结点*"可知，从结点x出发到达的所有的叶结点具有相同数目的黑结点。**这也就意味着，bh(x)的值是唯一的**！
    第2点：根据红黑色的"特性(4)，即*如果一个结点是红色的，则它的子结点必须是黑色的*"可知，从结点x出发达到叶结点"所经历的黑结点数目">= "所经历的红结点的数目"。假设x是根结点，则可以得出结论"**bh(x) >= $\frac{h}{2}$**"。进而，我们只需证明 "高度为h的红黑树，它的包含的黑结点个数至少为 $2^{bh(x)} - 1$个"即可。

 到这里，我们将需要证明的定理已经由
**"一棵含有n个结点的红黑树的高度至多为$2\log_2(n + 1)$"** 
    转变成只需要证明
**"高度为h的红黑树，它的包含的内结点个数至少为 $2^{bh(x)} - 1$个"。**



归纳假设证明：
    归纳奠基：当高度bh(x)=0时，黑结点数n(b) = 0 = $2^{bh(x)}-1$ = $2^{0}-1$，原命题成立；
    归纳假设：当高度bh(x)=k时，假设该树至少有为 $2^{bh(x)}-1$ = $2^{k}-1$个结点；
    归纳递推：当高度bh(x)=k+1时，根结点的两棵子树的高度肯定为k(特性4)，则两棵子树上的结点个数为$2(2^{bh(x)} - 1)$ = $2^{k+1}-2$(归纳假设),那么该树共有结点$2^{k+1}-2+1$（根结点） = $2^{k+1}-1$个结点，原命题成立。



红黑树的基本操作是**添加**、**删除**和**旋转**。在对红黑树进行添加或删除后，会用到旋转方法。为什么呢？道理很简单，添加或删除红黑树中的结点之后，红黑树就发生了变化，可能不满足红黑树的5条性质，也就不再是一颗红黑树了，而是一颗普通的树。而通过旋转，可以使这颗树重新成为红黑树。简单点说，旋转的目的是让树保持红黑树的特性。

左旋与右旋的代码和AVL相同，这里就不再列出



#### 插入

**第一步: 将红黑树当作一颗二叉查找树，将结点插入**

红黑树本身就是一颗二叉查找树，将结点插入后，该树仍然是一颗二叉查找树。也就意味着，树的键值仍然是有序的。此外，无论是左旋还是右旋，若旋转之前这棵树是二叉查找树，旋转之后它一定还是二叉查找树。这也就意味着，任何的旋转和重新着色操作，都不会改变它仍然是一颗二叉查找树的事实。
好吧？那接下来，我们就来想方设法的旋转以及重新着色，使这颗树重新成为红黑树！

**第二步：将插入的结点着色为"红色"。**

为什么着色成红色，而不是黑色呢？为什么呢？将插入的结点着色为红色，不会违背"特性(5)"！少违背一条特性，就意味着我们需要处理的情况越少。接下来，就要努力的让这棵树满足其它性质即可；满足了的话，它就又是一颗红黑树了

**第三步: 通过一系列的旋转或着色等操作，使之重新成为一颗红黑树。**

  第二步中，将插入结点着色为"红色"之后，不会违背"特性(5)"。那它到底会违背哪些特性呢？
       对于"特性(1)"，显然不会违背了。因为我们已经将它涂成红色了。
       对于"特性(2)"，显然也不会违背。在第一步中，我们是将红黑树当作二叉查找树，然后执行的插入操作。而根据二叉查找树的特点，插入操作不会改变根结点。所以，根结点仍然是黑色。
       对于"特性(3)"，显然不会违背了。这里的叶子结点是指的空叶子结点，插入非空结点并不会对它们造成影响。
       对于"特性(4)"，是有可能违背的！
       那接下来，想办法使之"满足特性(4)"，就可以将树重新构造成红黑树了

所有插入情况如下图所示

![](https://s2.ax1x.com/2019/11/24/MON1PO.png)

[转载自](https://www.jianshu.com/p/e136ec79235c)

在开始每个情景的讲解前，我们还是先来约定下

![](https://s2.ax1x.com/2019/11/24/MLqyGV.png)

##### 插入情景1：红黑树为空树

最简单的一种情景，直接把插入结点作为根结点就行，但注意，根据红黑树性质2：根结点是黑色。还需要把插入结点设为黑色。

**处理：把插入结点作为根结点，并把结点设置为黑色**。

##### 插入情景2：插入结点的val已存在

插入结点的val已存在，既然红黑树总保持平衡，在插入前红黑树已经是平衡的，那么把插入结点设置为将要替代结点的颜色，再把结点的值更新就完成插入。

**处理：**

- **把I设为当前结点的颜色**
- **更新当前结点的值为插入结点的值**

##### 插入情景3：插入结点的父结点为黑结点

由于插入的结点是红色的，当插入结点的黑色时，并不会影响红黑树的平衡，直接插入即可，无需做自平衡。

**处理：直接插入**。

##### 插入情景4：插入结点的父结点为红结点

再次回想下红黑树的性质2：根结点是黑色。**如果插入的父结点为红结点，那么该父结点不可能为根结点，所以插入结点总是存在祖父结点**。这点很重要，因为后续的旋转操作肯定需要祖父结点的参与。

情景4又分为很多子情景，下面将进入重点部分

##### **插入情景4.1：叔叔结点存在并且为红结点**

从红黑树性质4可以，祖父结点肯定为黑结点，因为不可以同时存在两个相连的红结点。那么此时该插入子树的红黑层数的情况是：黑红红。显然最简单的处理方式是把其改为：红黑红

**处理：**

- **将P和S设置为黑色**
- **将PP设置为红色**
- **把PP设置为当前插入结点**

![](https://s2.ax1x.com/2019/11/24/MLOCkR.png)

可以看到，我们把PP结点设为红色了，如果PP的父结点是黑色，那么无需再做任何处理；但如果PP的父结点是红色，根据性质4，此时红黑树已不平衡了，所以还需要把PP当作新的插入结点，继续做插入操作自平衡处理，直到平衡为止。

试想下PP刚好为根结点时，那么根据性质2，我们必须把PP重新设为黑色，那么树的红黑结构变为：黑黑红。换句话说，从根结点到叶子结点的路径中，黑色结点增加了。**这也是唯一一种会增加红黑树黑色结点层数的插入情景**。

我们还可以总结出另外一个经验：**红黑树的生长是自底向上的**。这点不同于普通的二叉查找树，普通的二叉查找树的生长是自顶向下的。

##### **插入情景4.2：叔叔结点不存在或为黑结点，并且插入结点的父亲结点是祖父结点的左子结点**

单纯从插入前来看，也即不算情景4.1自底向上处理时的情况，叔叔结点非红即为叶子结点(Nil)。因为如果叔叔结点为黑结点，而父结点为红结点，那么叔叔结点所在的子树的黑色结点就比父结点所在子树的多了，这不满足红黑树的性质5。后续情景同样如此，不再多做说明了。

前文说了，需要旋转操作时，肯定一边子树的结点多了或少了，需要租或借给另一边。插入显然是多的情况，那么把多的结点租给另一边子树就可以了。

##### **插入情景4.2.1：插入结点是其父结点的左子结点**

**处理：**

- **将P设为黑色**
- **将PP设为红色**
- **对PP进行右旋**

![](https://s2.ax1x.com/2019/11/24/MLX5Is.png)

##### **插入情景4.2.2：插入结点是其父结点的右子结点**

这种情景显然可以转换为情景4.2.1

**处理：**

- **对P进行左旋**
- **把P设置为插入结点，得到情景4.2.1**
- **进行情景4.2.1的处理**

![](https://s2.ax1x.com/2019/11/24/MLjZod.png)

##### **插入情景4.3：叔叔结点不存在或为黑结点，并且插入结点的父亲结点是祖父结点的右子结点**

该情景对应情景4.2，只是方向反转，不做过多说明了，直接看图

**插入情景4.3.1：插入结点是其父结点的右子结点**
**处理：**

- **将P设为黑色**
- **将PP设为红色**
- **对PP进行左旋**

![](https://s2.ax1x.com/2019/11/24/MLj2f1.png)

##### **插入情景4.3.2：插入结点是其父结点的右子结点**

**处理：**

- **对P进行右旋**
- **把P设置为插入结点，得到情景4.3.1**
- **进行情景4.3.1的处理**

![](https://s2.ax1x.com/2019/11/24/MLjImD.png)

代码

```Java
public boolean put(T val)
{
    return putVal(this.root, val);
}

private boolean putVal(Node<T> node, T val)
{
    if (node == null)
    {
        node = new Node<T>(BLACK, val, null, null, null);
        this.root = node;
        size++;
        return true;
    }
    Node<T> cur = node;
    Node<T> prev = null;
    int cmp;
    do
    {
        prev = cur;
        cmp = node.val.compareTo(val);
        if (cmp > 0)
            cur = cur.left;
        else if (cmp < 0)
            cur = cur.right;
        else
            return false;	
    } while (cur != null);
    Node<T> newNode = new Node<T>(RED, val, null, null, prev);
    if (cmp > 0)
        prev.left = newNode;
    else  
        prev.right = newNode;
    rebuild(newNode);
    size++;
    return true;
}

private void rebuild(Node<T> node)
{ 
    Node<T> parent, gparent;
    // 若“父结点存在，并且父结点的颜色是红色”
    while ((parent = parentOf(node)) != null && isRed(parent))
    {
        gparent = parentOf(parent);
        //若“父结点”是“祖父结点的左孩子”
        if (parent == gparent.left)
        {
            // Case 1条件：叔叔结点是红色
            Node<T> uncle = gparent.right;
            if (uncle != null && isRed(uncle))
            {
                setBlack(uncle);
                setBlack(node);
                setRed(gparent);
                node = gparent;
                continue;
            }

            // Case 2条件：叔叔是黑色或null，且当前结点是右孩子
            if (parent.right == node)
            {
                leftRotation(parent);
                Node<T> temp;
                temp = parent;
                parent = node;
                node = temp; 
            }

            // Case 3条件：叔叔是黑色或bull，且当前结点是左孩子。
            setBlack(parent);
            setRed(gparent);
            rightRotation(gparent);
        } else
        {
            //若“z的父结点”是“z的祖父结点的右孩子”
            // Case 1条件：叔叔结点是红色
            Node<T> uncle = gparent.right;
            if (uncle != null && isRed(uncle))
            {
                setBlack(uncle);
                setBlack(parent);
                setRed(gparent);
                node = gparent;
                continue;
            }

            // Case 2条件：叔叔是黑色或null，且当前结点是右孩子
            if (parent.left == node)
            {
                rightRotation(parent);
                Node<T> temp;
                temp = parent;
                parent = node;
                node = temp; 
            }

            // Case 3条件：叔叔是黑色或bull，且当前结点是左孩子。
            setBlack(parent);
            setRed(gparent);
            leftRotation(gparent);
        }
    }
    setBlack(this.root);
}
```

#### 删除

二叉树删除结点找替代结点有3种情情景：

+ 情景1：若删除结点无子结点，直接删除
+ 情景2：若删除结点只有一个子结点用子结点替换删除结点
+ 情景3：若删除结点有两个子结点，用后继结点替换删除结点