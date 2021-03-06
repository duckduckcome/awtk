# FAQ

#### 1.return\_value\_if\_fail作为AWTK中使用率排第一的宏，它的功能、优点和注意事项都有哪些？

**功能**

* 主要用于对函数的参数或函数的返回值进行检查(这是防御性编程的手段之一)。

> return\_value\_if\_fail这个宏并非是AWTK原创，而是从GTK+(或者说glib)里拿来的。

**优点**

* 以简洁的方式对函数的参数或函数的返回值进行检查。

* Release模式和Debug模式可以做不同的处理。

> 在参数出现错误时，悄无声息的返回一个错误码，其实是对调用者的纵容，很容易把错误隐藏起来。所以在Debug模式我们可以打出一条警告信息，甚至直接assert掉，这对于定位BUG非常有效。

**注意事项**

* 内部函数(static)一般不需要对参数进行检查。
* 只对异常的情况进行判断，对于正常的失败或无效参数，请不要使用本宏。
* 如果在返回之前，有资源需要释放，请不要用本宏。可以用goto\_error\_if\_fail跳到error出，释放资源后再返回。

---

#### 2.每次在绘制图片前，都要调用image\_manager\_load去加载图片，这样做会不会很慢？有什么优点？

* 不会慢。因为image\_manager中有缓存，不会每次都去解码。

**优点**

* 缓存有助于多个控件共享同一张图片。

* 外面不保存对bitmap的引用，缓存管理更加灵活。比如，可以清除最近没有被渲染的图片(即使某个隐藏的窗口还在使用该图片)。

---

#### 3.使用矢量字体，速度会慢吗？

* 几乎没有影响。因为有缓存，所以只需要渲染一次，之后和位图字体的并无不同。

---

#### 4.在16位LCD上显示PNG图片效果很差，有什么办法吗？

* 如果是不透明的图片，可以将PNG转换成JPG文件，转换过程中启用dithering算法做平滑处理。

可以用imagemagic转换：
```
convert bg.png  -ordered-dither o8x8,32,64,32 bg.jpg
```
> 参考：http://www.imagemagick.org/Usage/quantize/


#### 5.在Windows平台，SVG显示不正常，如何解决？

原因是Windows下OpenGL不支持非凸多边形，所以要解决这个问题，需要使用不同的NANOVG\_BACKEND。这可以在SConstruct文件中修改。如：

```
#NANOVG_BACKEND='GL3'
#NANOVG_BACKEND='GLES2'
#NANOVG_BACKEND='GLES3'
#NANOVG_BACKEND='AGG'
#NANOVG_BACKEND='AGGE'
NANOVG_BACKEND='BGFX'
```

在PC上，BGFX是推荐的NANOVG\_BACKEND，但是需要进入3rd/bgfx目录，手工下载bgfx相关源码：

```
cd 3rd/bgfx
git clone https://github.com/bkaradzic/bx.git 
git clone https://github.com/bkaradzic/bimg.git 
git clone https://github.com/bkaradzic/bgfx.git
```

> 如果您不需要使用SVG，仍然可以使用GL3作为NANOVG\_BACKEND。

