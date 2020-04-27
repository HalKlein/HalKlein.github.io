# 数据库索引之 B+ Tree、Hash、BitMap


<!--more-->

## B+ Tree

由于，二叉查看树在插入修改过程中，很可能出现只有左右子树的情况，虽然可以用平衡调整二叉树，但是每次IO依然很费时间，每访问一层就要对磁盘进行IO，总体来说二叉树还不够优。由此找到了更好的替代方法-B树，又由于B+ Tree具有更好的特性，所以B+ Tree就成了常见的索引结构之一。

![B+ Tree](https://i.loli.net/2019/12/24/xkZWtEguQIAoRXK.png)

**B+ Tree相对于B树，具有的新特点：**

- 非叶子节点的子树指针与关键字个数相同
- 非叶子节点的子树指针P[]，指向关键字值（[Ki]，K[i+1]）的子树
- 非叶子节点仅用来索引，数据都保存在叶子节点中
- 所有叶子节点均有一个链指针指向下一个叶子结点



## Hash

通过Hash索引，能够通过Hash函数快速定位到索引目标，理论上比B+ Tree更快。

![Hash索引](https://i.loli.net/2019/12/24/hci2ueoaKS4klxf.png)

**但是Hash索引也有也有严重的缺陷：**

- 仅仅能满足“="，“IN"，不能使用范围查询
- 无法被用来避免数据的排序操作
- 不能利用部分索引键查询
- 不能避免表扫描
- 遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高

所以，索引没有最优，只有更适合某种情况的索引结构，他们常常组合使用。



## BitMap

位图索引，适用于某个字段的值只有固定的几种为了实现高效的统计的情况，如性别（男、女）。不过支持位图索引的数据库比较少，典型的有Oracle数据库。

![BitMap](https://i.loli.net/2019/12/24/4uxK3ZdCTknmj9w.png)



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
