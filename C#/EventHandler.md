#CSharp 
    public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);
    EventHandler提供了一个包装器，可确保方法只能以类型安全的方式调用。
    `button.Click += new EventHandler(button1_Click)`