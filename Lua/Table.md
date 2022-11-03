asdasdad[^1]
```lua

typedef struct Table {

  CommonHeader;

  lu_byte flags;  /* 1<<p means tagmethod(p) is not present */

  lu_byte lsizenode;  /* log2 of size of 'node' array */

  unsigned int alimit;  /* "limit" of 'array' array */

  TValue *array;  /* array part */

  Node *node;

  Node *lastfree;  /* any free position is before this position */

  struct Table *metatable;

  GCObject *gclist;

} Table;
```

[^1]:     asdasd
