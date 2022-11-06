### 函数原型
每个Lua函数都有一个原型，这是由GC管理的对象，它**挂靠**在函数上，为函数提供必要的信息。比如操作码（opcodes），常量信息，本地变量信息，upvalue信息，和调试信息等等。
```lua
/*

** Function Prototypes

*/

typedef struct Proto {

  CommonHeader;

  lu_byte numparams;  /* number of fixed (named) parameters 固定参数的数量*/

  lu_byte is_vararg;  /*是否有可变参数*/

  lu_byte maxstacksize;  /* number of registers needed by this function 该函数需要的栈大小*/

  int sizeupvalues;  /* size of 'upvalues' */

  int sizek;  /* size of 'k' */

  int sizecode;

  int sizelineinfo;

  int sizep;  /* size of 'p' */

  int sizelocvars;

  int sizeabslineinfo;  /* size of 'abslineinfo' */

  int linedefined;  /* debug information  */

  int lastlinedefined;  /* debug information  */

  TValue *k;  /* constants used by the function */

  Instruction *code;  /* opcodes */

  struct Proto **p;  /* functions defined inside the function */

  Upvaldesc *upvalues;  /* upvalue information */

  ls_byte *lineinfo;  /* information about source lines (debug information) */

  AbsLineInfo *abslineinfo;  /* idem */

  LocVar *locvars;  /* information about local variables (debug information) */

  TString  *source;  /* used for debug information */

  GCObject *gclist;

} Proto;

```