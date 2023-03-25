#CSharp 
GC.SuppressFinalize(this);
请求 CLR 不要调用指定对象的终结器。
就是说当你手动调用Dispose或者Close方法释放了非托管资源后，通过此方法强制告诉CLR不要再触发我的析构函数了，否则再执行析构函数相当于又做了一次清理非托管资源的操作，造成未知风险。
https://www.cnblogs.com/huangxincheng/p/12811291.html