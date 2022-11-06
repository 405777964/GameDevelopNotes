闭包就是一个函数外加能够使该函数正确访问非局部变量（``UpValue``）所需的其他机制。
从技术上讲，Lua中只有闭包没有函数，函数本身只是闭包的一种原型。

### 函数原型
每个Lua函数都有一个原型，这是由GC管理的对象，它**挂靠**在函数上，为函数提供必要的信息。比如操作码（``opcodes``），常量信息，本地变量信息，``upvalue``信息，和调试信息等等。
```lua
/*

** Function Prototypes

*/

typedef struct Proto {

  CommonHeader;

  lu_byte numparams;  /* number of fixed (named) parameters 固定参数的数量*/

  lu_byte is_vararg;  /*是否有可变参数*/

  lu_byte maxstacksize;  /* number of registers needed by this function 该函数需要的栈大小*/

  int sizeupvalues;  /* size of 'upvalues' upvalues数量*/

  int sizek;  /* size of 'k' * 常量数量*/

  int sizecode;   /*指令数量*/

  int sizelineinfo;   /*行信息数量*/

  int sizep;  /* size of 'p' 内嵌原型数量*/

  int sizelocvars;   /*本地变量的数量*/

  int sizeabslineinfo;  /* size of 'abslineinfo' */

  int linedefined;  /* debug information  函数进入的行*/

  int lastlinedefined;  /* debug information  函数返回的行*/

  TValue *k;  /* constants used by the function 常量数量*/

  Instruction *code;  /* opcodes 指令数组*/

  struct Proto **p;  /* functions defined inside the function 内嵌函数原型*/

  Upvaldesc *upvalues;  /* upvalue information UpVulue信息*/

  ls_byte *lineinfo;  /* information about source lines (debug information) */

  AbsLineInfo *abslineinfo;  /* idem */

  LocVar *locvars;  /* information about local variables (debug information) */

  TString  *source;  /* used for debug information */

  GCObject *gclist;

} Proto;

```
