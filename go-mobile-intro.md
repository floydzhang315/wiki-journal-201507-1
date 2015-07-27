# 用Go语言来看Android! 出发, Android. 出发!     -- 王韬懿
来源：[http://www.codingvelocity.com/2015/07/23/go-mobile-intro.html](http://www.codingvelocity.com/2015/07/23/go-mobile-intro.html)

时间：2015年7月23日

作者：Chester

## 关于本文
如今移动开发虽然三分天下，但主要市场还是 Android 和 IOS ,但是对于一些中小型公司来说单独开发的成本明显比较大。2014 年苹果推出了全新的语言 Swift ,而作为对头的 Google 也有一门自己独特的语言，那就是 Go 语言，Go 语言不仅能用来进行 Android 开发，而且也正在适配 IOS 平台，下面让我们来入个门吧。

## 文章内容

随着 Go 1.5 的即将发布,在 Android 和 IOS 上编译 Go 语言的代码正在被引进。你既可以完全用 Go 语言来写你的应用并用 opengl 来做 UI 界面,也可以写一个可以和原生的 Android 和 IOS 交互的 Go 类库。这为跨平台类库打开了大门，这让我激动不已。Google 为了确保用 Go 语言写出来的 apps 不会被 apple store 拒之门外而做了很多跑腿儿活，比如[Ivy](https://itunes.apple.com/us/app/ivy-big-number-calculator/id1012116478?mt=8) 。

### 入门指南

让我们开始吧，首先你必须安装一个可运行的 go 1.5. 你可以从[源代码](https://golang.org/doc/install/source)编译或者使用一个之前[编译好的版本](https://golang.org/dl/)。我在写这篇博客的时候在一台 Linux 机器上使用的是 的 go 1.5 beta2. 我注意到在 Windows 上面使用的时候在 Go 语言源代码里面会有一些警告信息，因此如果你正在使用 Windows 开发，这可能不会起作用。

一旦你要安装 Go 语言，你可以通过下面的命令行来安装：

```
go get golang.org/x/mobile/cmd/gomobile
gomobile init
```

Android 平台：你需要安装 Android [sdk](https://developer.android.com/sdk/installing/index.html?pkg=tools) 并且确保你的 adb 在你的环境变量里面，并且你的设备要能够使用 adb 调试。

IOS 平台：很不幸的是 IOS 并不是 100% 适配，因此可能不会有作用。通常在 IOS 上编译你需要安装 xcode 并写使用 OSX 系统。

### 安装一个示例

让我们看看可不可以编译并且安装一些 go 语言的代码。Google 已经提供了一些我们可以使用的[例子](https://godoc.org/golang.org/x/mobile/example) 。为了简便起见我这里只安装 android 版本（主要原因是因为我没有 IOS 设备）。

接下来的命令就会安装示例应用：

```
gomobile install golang.org/x/mobile/example/basic
gomobile install golang.org/x/mobile/example/audio
gomobile install golang.org/x/mobile/example/sprite 
```
虽然看起来不多，但是我觉得这相当酷。上面的应用使用纯 go 语言写的，并且使用 opengl 来做渲染。目前还有一些 api 限制，但是我相信不久就会改善。

### 分析跨平台开发的应用

好，我们可以编译他人的代码，但是它在干什么？让我们看看在这个基本的示例应用里面发生了什么。

```
//excerpt from golang.org/x/mobile/example/basic
func main() {
    app.Main(func(a app.App) {
        var c config.Event
        for e := range a.Events() {
            switch e := app.Filter(e).(type) {
            case lifecycle.Event:
                switch e.Crosses(lifecycle.StageVisible) {
                case lifecycle.CrossOn:
                    onStart()
                case lifecycle.CrossOff:
                    onStop()
                }
            case config.Event:
                c = e
                touchLoc = geom.Point{c.Width / 2, c.Height / 2}
            case paint.Event:
                onPaint(c)
                a.EndPaint()
            case touch.Event:
                touchLoc = e.Loc
            }
        }
    })
}
```
用 Go 语言写的应用会从 app 包里面调用主函数。在这里你可以定义应该发生什事件，你可以在[事件文档](https://godoc.org/golang.org/x/mobile/event)了解更多细节。这些事件基于注册在应用里面的接口。

上面的代码遍历在事件频道里面所有的事件。配置事件定义了屏幕的大小，绘图事件正在绘制我们长方形的颜色。点击事件改变了长方形的位置，生命周期事件构造或者析构这个基于应用焦点的项目。

### 更多阅读

我希望我已经激起了你的兴趣，研究 Go 语言可以参考这些文章。

所有的示例都可以在这里找到 [http://golang.org/x/mobile/example](http://golang.org/x/mobile/example)

在这里可以找到文档[https://godoc.org/golang.org/x/mobile](https://godoc.org/golang.org/x/mobile)

源代码在这里 [https://github.com/golang/mobile](https://github.com/golang/mobile)
