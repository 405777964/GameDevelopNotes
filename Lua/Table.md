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


