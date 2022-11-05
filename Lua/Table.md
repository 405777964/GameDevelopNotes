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

散列表部分的数组组织是，先计算key所在的散列桶位置，这个位置称为mainposition，该散列桶以链表的形式组织。
当找不到对应的key时会调用newkey方法分配一个新的key来返回。

以上操作涉及对表空间的重新分配（remash）
1. 分配一个位图nums，将它所有位置设为0。该位图的意义是：nums数组中第i个元素存放的是key在2^(i-1)和2^i之间的元素数量。
2. 