表包含**散列表**和**数组**这两部分数据。

```lua

typedef struct Table {

  CommonHeader;

  lu_byte flags;  /* 1<<p means tagmethod(p) is not present */

  lu_byte lsizenode;  /* log2 of size of 'node' array */

  unsigned int alimit;  /* "limit" of 'array' array */

  TValue *array;  /* array part 存放数组类型*/

  Node *node;    /* 存放散列表类型*/

  Node *lastfree;  /* any free position is before this position */

  struct Table *metatable;

  GCObject *gclist;

} Table;

/*
** Common Header for all collectable objects (in macro form, to be
** included in other objects)
*/
#define CommonHeader  struct GCObject *next; lu_byte tt; lu_byte marked
```


## 查找
```
如果输入的key是一个正整数，并且它的值 > 0 && <= 数组大小
	尝试在数组部分查找
否则尝试在散列表部分查找：
	计算出该key的散列值，根据此散列值访问Node数组得到散列桶所在的位置
	遍历该散列桶下的所有链表元素找到对应的key为止
```

## 新增元素
<p>疑问 ： 新增元素是不是先在数组范围里找对应的key，替换value值。如果不在数组范围再去散列表部分寻找</p>

散列表部分的数组组织是，先计算key所在的散列桶位置，这个位置称为mainposition，该散列桶以**链表**的形式组织。
当找不到对应的key时会调用newkey方法分配一个新的key来返回。

以上操作涉及对表空间的重新分配（remash）
1. 分配一个位图nums，将它所有位置设为0。该位图的意义是：nums数组中第i个元素存放的是key在2^(i-1)和2^i之间的元素数量。
2. 遍历Lua表中的数组部分，计算其中的元素数量，更新对应的nums数组中的元素数量(numusearray函数)。
3. 遍历Lua表中的散列桶部分，因为其中也可能存放了正整数，根据这里的正整数数量更新对应的nums数组元素数量（numusehash）
4. 此时nums已经有当前Table中所有正整数的分配统计，逐个遍历nums数组，获得**其范围区间内所包含的整数数量大于50%的最大索引**，作为重新散列后的数组大小。**超过这个范围的正整数就分配到散列桶部分了**。
5. 根据上面计算得出调整后的数组和散列桶大小调整表（resize整数）。
```lua
--例子  table存在key为1,2,3,20，nums中各个key在2^(i-1)和2^i之间的元素数量
nums[0] = 1 --（1落在数组中）
nums[1] = 1 --（2落在数组中）
nums[2] = 1 --（3落在数组中）
nums[3] = 0
nums[4] = 0
nums[5] = 1 --（20落在散列桶中）
```

一个整数的key在同个表的不同阶段可能会被分配到数组或者散列桶部分。
同时重新散列的代价是挺大的。
```lua
local a = {}
for i=1,3 do
	a[i] = true
end
```
该代码执行了三次重新散列，有点类似List表Add新元素以2次幂扩容的操作。
然而有100万个元素的表只会执行20次重新散列操作，因此如果创建很多长度很小的表，可以进行**预先填充**避免重新散列，提高代码运行速度。
```lua
local a = {false,false,false}
for i=1,3 do
	a[i] = true
end
```

## 迭代
一般算法库的设计中，针对容器类的迭代，会提供一个迭代器的数据，这个数据主要用于维护当前迭代到容器的哪部分数据了，下次根据这个位置查找下一个部分的数据。
而Lua的表迭代传入的不是一个迭代器，而是key。迭代对外的API是``LuaH_next``函数。
```luaH_next函数
在数组部分查找数据
	查找成功，则返回该key的下一个数据
否则在散列桶部分查找数据
	查找成功，则返回该key的下一个数据
```
这个函数一开始就进入``findindex``中进行查询,并区分数组和散列桶的部分，``findindex``函数返回的结果是一个整数索引，如果索引在sizearray内则落入数组部分，如果在``sizearray``外则落入散列桶部分。而在返回散列桶部分时，这个索引值为”``sizearray + 对应散列桶索引的值``”。