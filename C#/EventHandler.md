#CSharp 
    `public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);`
    EventHandler提供了一个包装器，可确保方法只能以**类型安全**的方式调用。
    `button.Click += new EventHandler(button1_Click);`

## 类型安全
如果这个语言的类型系统能够保证所有程序都不会出现未定义行为，那么这个语言就是类型安全的。