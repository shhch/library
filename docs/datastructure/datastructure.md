# 数据结构与算法

## 数据结构

+ 树（你怕了吗？）
  ```
    二叉树：每个节点有最多两个子节点的树；
    二叉查找树：按照中序遍历进行插入的树，思路和二分查找相似；
    二叉平衡树（AVL树）：二叉查找树存在歪的情况，既一直插入越来越大（小）的值，则树会往对应方向倾斜；二叉平衡树的出现就是为了解决这个问题，它规定所有节点的左右子树的高度差必须小于1；当插入节点时，需要进行平衡因子检测，不平衡则需要进行旋转，旋转太复杂，不讲了；
    红黑树：平衡二叉树是绝对的平衡，但是因为追求绝对的平衡，所以实现复杂，同时性能不佳，红黑树则是折中的实现，平衡二叉树在红黑树之后出现的；红黑树有以下规则：
    1、给每个节点是红色或黑色；
    2、根节点始终为黑色；
    3、没有两个相邻的红色节点，可以有相邻的黑色节点；
    4、每个叶子节点（NULL）是黑色；
    5、从节点（包括根）到其任何后代NULL节点的每条路径都具有相同数量的黑色节点；
    它有三种操作，变色，左旋，右旋；新插入的节点永远是红色，如果是黑色的话，插入前树是平衡的，你加个黑的，肯定得旋转，如果是根节点，再变成黑的就行了；
    BTree，B+Tree:这两个在mysql讲

  ```

## 用数组存储完全二叉树
