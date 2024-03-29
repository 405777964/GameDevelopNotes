#Lua 
Lua和C之间通信的主要组件是`虚拟栈`（stack）几乎所有的API调用都是在操作这个栈中的值，Lua与C之间所有的数据交换都是通过这个栈完成的。此外还可以通过栈保存中间结果。

Lua与C交换数据有两个问题：
1. 动态类型和静态类型体系之间的不匹配。
2. 自动内存管理和手动内存管理之间的不匹配。

## C调用Lua
栈中每个元素都能保存Lua中任意类型的值。当我们想从Lua中获取一个值时（如一个全局变量），只需调用Lua，Lua就会将指定的值压入栈中。当想要将一个值传给Lua，首先将这个值压入栈，然后调用Lua将其从栈中弹出即可。

## Lua调用C
当Lua调用C函数时，我们必须注册该函数，即必须以一种`恰当的方式`为Lua提供该C函数的地址。
Lua调用C函数时，也使用了一个与C调用Lua函数时相同类型的栈，C函数从栈中获取参数，并将结果压入栈中。（区别是这个栈不是全局结构，而是每个函数都有其私有的局部栈）

所有在Lua中注册的函数都必须使用**一个相同的原型**，该原型是定义在``lua.h``的``lua_CFunction`` 
``typedef int (*lua_CFunction) (lua_State *L);``
这个函数只有一个指向Lua状态类型的指针作为参数，返回值为一个整型数，代表压入栈中的返回值的个数。因此，该函数在压入结果前无需清空栈。在该函数返回后，Lua会自动保存返回值并清空整个栈。
```c
static int l_sin(lua_State *L){
	double d = lua_tonumber(L,1);   /*获取Lua传过来的参数*/
	lua_pushnumber(L, sin(d));      /*压入返回值*/
	return 1;                       /*返回值的个数*/	
}
```
在Lua中，调用这个函数前，必须通过``lua_pushcfunction``**注册**该函数。``lua_pushcfunction``会获得一个指向C函数的指针，然后在Lua中创建一个`"function"`类型，代表**待注册**的函数。一旦完成注册C函数就可以像其他Lua函数一样行事了。
```c
lua_pushcfunction(L, l_sin);    /*压入一个函数类型的值*/
lua_setglobal(L, "mysin");      /*将这个值赋给全局变量mysin*/
```

https://zhuanlan.zhihu.com/p/76250674


### Tips
这里使用3种方式向表增加值：
```lua
local tinsert = table.insert
tm = os.clock()
for i = 1, 3000 do
	local a = {} 
	for i = 1, 10000 do
		-- table.insert(a, i)
		-- tinsert(a, i) 
		a[#a+1] = i 
	end 
end 
print(">>>>>>", os.clock() - tm)
```
1.  直接使用table.insert，消耗的时间是：3.032s
2.  把table.insert保存为一个本地变量再调用，消耗的时间是：2.688s
3.  直接用a[#a+1] = i 表达式，消耗的时间是：1.994s

直接用Lua语句的效率最高，因为减少了C函数的调用开销；用tinsert次之；第1种最慢，原因是table.insert是**先从_ENV取出table，再从table取出insert**。