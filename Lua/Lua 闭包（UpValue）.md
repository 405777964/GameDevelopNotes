
```lua
/*

** Function Prototypes

*/

typedef struct Proto {

  CommonHeader;

  lu_byte numparams;  /* number of fixed (named) parameters */

  lu_byte is_vararg;

  lu_byte maxstacksize;  /* number of registers needed by this function */

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