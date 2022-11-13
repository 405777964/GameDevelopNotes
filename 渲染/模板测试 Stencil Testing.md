当片段着色器处理完一个片段之后，模板测试(Stencil Test)会开始执行。

模板测试是根据又一个缓冲来进行的，它叫做模板缓冲(Stencil Buffer)。

一个模板缓冲中，（通常）每个模板值(Stencil Value)是8位的。所以每个像素/片段一共能有256种不同的模板值。我们可以将这些模板值设置为我们想要的值，然后当某一个片段有某一个模板值的时候，我们就可以选择丢弃或是保留这个片段了。

![Untitled](渲染/渲染管线流程/模板测试%20Stencil%20Testing/Untitled.png)

模板缓冲步骤

- 启用模板缓冲的写入。
- 渲染物体，更新模板缓冲的内容。
- 禁用模板缓冲的写入。
- 渲染（其它）物体，这次根据模板缓冲的内容丢弃特定的片段。

## 模板掩码

和深度测试的glDepthMask函数一样，模板缓冲也有一个类似的函数。glStencilMask允许我们设置一个位掩码(Bitmask)，它会与将要写入缓冲的模板值进行与(AND)运算。默认情况下设置的位掩码所有位都为1，不影响输出，但如果我们将它设置为`0x00`，写入缓冲的所有模板值最后都会变成0.这与深度测试中的glDepthMask(GL_FALSE)是等价的。

```
glStencilMask(0xFF); // 每一位写入模板缓冲时都保持原样
glStencilMask(0x00); // 每一位在写入模板缓冲时都会变成0（禁用写入）

```

大部分情况下你都只会使用`0x00`或者`0xFF`作为模板掩码(Stencil Mask)，但是知道有选项可以设置自定义的位掩码总是好的。

## 模板函数

和深度测试一样，我们对模板缓冲应该通过还是失败，以及它应该如何影响模板缓冲，也是有一定控制的。一共有两个函数能够用来配置模板测试：glStencilFunc和glStencilOp。

glStencilFunc(GLenum func, GLint ref, GLuint mask)一共包含三个参数：

- `func`：设置模板测试函数(Stencil Test Function)。这个测试函数将会应用到已储存的模板值上和glStencilFunc函数的`ref`值上。可用的选项有：
    
    GL_NEVER
    
    GL_LESS
    
    GL_LEQUAL
    
    GL_GREATER
    
    GL_GEQUAL
    
    GL_EQUAL
    
    GL_NOTEQUAL
    
    GL_ALWAYS
    
- `ref`：设置了模板测试的参考值(Reference Value)。模板缓冲的内容将会与这个值进行比较。
- `mask`：设置一个掩码，它将会与参考值和储存的模板值在测试比较它们之前进行与(AND)运算。初始情况下所有位都为1。

在一开始的那个简单的模板例子中，函数被设置为：

```
glStencilFunc(GL_EQUAL, 1, 0xFF)

```

这会告诉OpenGL，只要一个片段的模板值等于(`GL_EQUAL`)参考值1，片段将会通过测试并被绘制，否则会被丢弃。

但是glStencilFunc仅仅描述了OpenGL应该对模板缓冲内容做什么，而不是我们应该如何更新缓冲。这就需要glStencilOp这个函数了。

glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)一共包含三个选项，我们能够设定每个选项应该采取的行为：

- `sfail`：模板测试失败时采取的行为。
- `dpfail`：模板测试通过，但深度测试失败时采取的行为。
- `dppass`：模板测试和深度测试都通过时采取的行为。

用来实现物体轮廓功能

![Untitled](渲染/渲染管线流程/模板测试%20Stencil%20Testing/Untitled%201.png)

先将物体渲染，并写入模板缓冲。

再将物体放大，根据模板缓冲，只对为0的模板进行绘制边框。