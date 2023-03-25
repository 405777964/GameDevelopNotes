#CSharp 
    `public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);`
    EventHandler提供了一个包装器，可确保方法只能以**类型安全**的方式调用。
    `button.Click += new EventHandler(button1_Click);`

## 类型安全
