# DrrrClient For PC

## fork的PyQt5项目

### 简述
近来想找个PyQt项目好好学习学习，毕竟正在学PyQt5，刚去官网下了PyQt5.6,刚好PyQt5.6支持python3.5,电脑上又有python3.5.1，然而关于PyQt5，网上的资料实在是太少了，就连官网的文档大多都是转向Qt的c++文档，我也是醉了。

找到一个用python2.7+PyQt5(低于5.6)的版本写的一个针对[Drrr chat room ](http://drrr.com/)网站写的客户端，界面如下：

![](http://i.imgur.com/ji5KPOJ.png)

<!-- more -->

想好好研读下人家的代码，并向python3.5.1+PyQt5.6移植，对于Python2到3的移植，由于源代码比较简单，就1000+行代码，而且就一个py文件，界面和功能都在这个文件中，基本上对于python的移植就是将：

```python
print "something"
```

改为：

```python
print("something")
```

令人头疼的是PyQt的移植，只想说版本之间兼容性无话可说，有些内容差距真是太大了，尤其是QtWebKit 到 QtWebEngine的移植，表示官网文档都快被翻烂了，也就是根据函数名猜其功能，实在是受不了了。附上官网的这篇文章，供PyQt移植用：

[Porting from QtWebKit to QtWebEngine](https://wiki.qt.io/Porting_from_QtWebKit_to_QtWebEngine)

原作者[github](https://github.com/harry159821/DrrrClient)

看的出来，是作者的练手作品，作者还做了PyQt4到PyQt5的移植，主要也就是信号与槽函数建立连接的方式。大体来说整个程序先自绘窗口，然后中间窗口是个QWebView，确实像作者所言大体是个浏览器，但是依然还是有很多值得学习的地方。下面开始源码解读

### 大体描述
目录结构比较简单：

![](http://i.imgur.com/e6d0XHJ.png)

Shot目录是几个截图，不重要，img目录是项目需要的图片和几个音频文件，ini配置文件存放用户名和语言等配置信息，不重要，Makefile.py是py2exe用于打包整个项目所写的配置文件(目前python3.5的项目不能用py2exe打包，可以用pyInstaller)，不重要，bat是执行打包，不重要。唯一重要的也就是DrrrChatRoom.py这个文件，代码总共1000+行，也不是特别麻烦，程序有不少注释，结构比较清晰，如果把界面和功能代码分离那就更好了。

### 源码阅读
labelBtn类：

```python
class labelBtn(QtWidgets.QLabel):
    clicked = QtCore.pyqtSignal(str)
    Entered = QtCore.pyqtSignal(str)
    Leaved = QtCore.pyqtSignal(str)
    Moved = QtCore.pyqtSignal(str,int,int)

    def __init__(self,name,timeoutset=None,parent=None):
        super(labelBtn,self).__init__()
        self.setMouseTracking(True)
        self.name = name
            
    def mouseReleaseEvent(self,event):
        self.clicked.emit(self.name)
        
    def mouseMoveEvent(self,event):
        self.Moved.emit(self.name,event.globalPos().x(),event.globalPos().y())
    
    def enterEvent(self,event):
        self.Entered.emit(self.name)
   
    def leaveEvent(self,event):
        self.Leaved.emit(self.name)
```

这个类是作者用来自定义窗口右上角最大化，最小化，关闭三个按钮，类中定义了四个信号，并重写四个事件，分别emit这四个信号。

FrameLessTransparentWindow类：

```python
class FrameLessTransparentWindow(QtWidgets.QMainWindow):
    def __init__(self):...

    def minFunc(self,name):...

    def showMinimized2(self):...

    def maxFunc(self,name):...

    def closeFunc(self,name):...

    def exitFunc(self):...

    def buttonEnterFunc(self,name):...

    def buttonLeavedFunc(self,name):...

    def mousePressEvent(self, event):...

    def mouseMoveEvent(self, event):...

    def leaveEvent(self,event):...
```

这个类是主要窗口的基类，里面都是些通用函数，顾名思义就知道他们是干什么用的。

ShadowsWindow类

```python
class ShadowsWindow(FrameLessTransparentWindow):
    def __init__(self):
        super(ShadowsWindow, self).__init__()
        self.setWindowFlags(Qt.FramelessWindowHint)  #去掉窗口标题栏
        self.setAttribute(Qt.WA_TranslucentBackground)   #窗体标题栏不透明,背景透明
        self.SHADOW_WIDTH=15

    def drawShadow(self,painter):
        self.pixmaps=[]
        self.pixmaps.append("./img/left_top.png")
        self.pixmaps.append("./img/left_bottom.png")
        self.pixmaps.append("./img/right_top.png")
        self.pixmaps.append("./img/right_bottom.png")
        self.pixmaps.append("./img/top_mid.png")
        self.pixmaps.append("./img/bottom_mid.png")
        self.pixmaps.append("./img/left_mid.png")
        self.pixmaps.append("./img/right_mid.png")
        painter.drawPixmap(0, 0, 
            self.SHADOW_WIDTH, self.SHADOW_WIDTH, QPixmap(self.pixmaps[0]))   # 左上角
        painter.drawPixmap(self.width()-self.SHADOW_WIDTH, 0, 
            self.SHADOW_WIDTH, self.SHADOW_WIDTH, QPixmap(self.pixmaps[2]))   # 右上角
        painter.drawPixmap(0,self.height()-self.SHADOW_WIDTH, 
            self.SHADOW_WIDTH, self.SHADOW_WIDTH, QPixmap(self.pixmaps[1]))   # 左下角
        painter.drawPixmap(self.width()-self.SHADOW_WIDTH, self.height()-self.SHADOW_WIDTH, 
            self.SHADOW_WIDTH, self.SHADOW_WIDTH, QPixmap(self.pixmaps[3]))  # 右下角
        painter.drawPixmap(0, self.SHADOW_WIDTH, self.SHADOW_WIDTH, 
            self.height()-2*self.SHADOW_WIDTH, 
            QPixmap(self.pixmaps[6]).scaled(self.SHADOW_WIDTH, self.height()-2*self.SHADOW_WIDTH)) # 左
        painter.drawPixmap(self.width()-self.SHADOW_WIDTH, self.SHADOW_WIDTH, 
            self.SHADOW_WIDTH, self.height()-2*self.SHADOW_WIDTH, 
            QPixmap(self.pixmaps[7]).scaled(self.SHADOW_WIDTH, self.height()- 2*self.SHADOW_WIDTH)) # 右
        painter.drawPixmap(self.SHADOW_WIDTH, 0, self.width()-2*self.SHADOW_WIDTH, self.SHADOW_WIDTH, 
            QPixmap(self.pixmaps[4]).scaled(self.width()-2*self.SHADOW_WIDTH, self.SHADOW_WIDTH)) # 上
        painter.drawPixmap(self.SHADOW_WIDTH, 
            self.height()-self.SHADOW_WIDTH, 
            self.width()-2*self.SHADOW_WIDTH, 
            self.SHADOW_WIDTH, QPixmap(self.pixmaps[5]).scaled(self.width()-2*self.SHADOW_WIDTH, 
            self.SHADOW_WIDTH))   # 下        
        
    def paintEvent(self, event):
        painter = QPainter(self)
        self.drawShadow(painter)
        painter.setPen(Qt.NoPen)
        painter.setBrush(Qt.white)
```

ShadowsWindow窗口类继承自FrameLessTransparentWindow，没有标题栏，客户区背景透明，重写了绘图事件，手动绘制了窗口的四个角，和上下左右的阴影，所以这个类取名为shadow

titleBar类

```python
class titleBar(QWidget):
	def __init__(self,parent=None):...
	def drawShadow(self,painter):...
	def paintEvent(self, event):...
	def enterEvent(self,event):...
```

这个类画的是窗口标题栏，重写绘图事件，并在标题栏窗口边上画阴影就不说了，具体来看下构造函数：

首先设置好长宽等属性后，通过：

```python
self.setStyleSheet(...)
```

设置一些样式。

```python
self.title_label = QLabel()
self.title_label.setText(u"    DRRR Chat Room")
self.font = QtGui.QFont()
self.font.setPixelSize(22)   # 设置字号32,以像素为单位
self.font.setFamily("SimSun")# 设置字体，宋体
self.font.setBold(True)
self.font.setItalic(False)   # 设置字型,不倾斜
self.font.setUnderline(False)# 设置字型,无下划线
self.title_label.setFont(self.font)
```

设置标题栏文字，并设置字体。

```python
self.close_button = labelBtn('x')
self.min_button = labelBtn('-')
self.max_button = labelBtn('口')

self.close_button.setPixmap(QPixmap("./img/orange.png"))
self.min_button.setPixmap(QPixmap("./img/green.png"))
self.max_button.setPixmap(QPixmap("./img/blue.png"))
```

新建最大化，最小化，关闭三个按钮，并设置按钮图片，之后设置样式和一些其他属性

```python
self.title_layout = QHBoxLayout()
self.title_layout.setContentsMargins(0, 0, 20, 0)
self.title_layout.addWidget(self.title_label,1,Qt.AlignCenter)
self.title_layout.addStretch()
self.title_layout.addWidget(self.min_button  ,0,Qt.AlignVCenter)
self.title_layout.addWidget(self.max_button  ,0,Qt.AlignVCenter)
self.title_layout.addWidget(self.close_button,0,Qt.AlignVCenter)

self.setLayout(self.title_layout)
```

将文字和三个按钮分别放在布局中，并设置titleBar窗口内的布局

StatusWindow窗口类：

用来画窗口的状态栏，基本和titleBar类一样。

DrrrWindow类：

继承自ShadowsWindow类，是程序的主要窗口。来看下其构造方法：

```python
self.getSetting()
...
def getSetting(self):
    '''获取应用设置'''
    self.settings = QtCore.QSettings("DrrrChatRoom.ini", QtCore.QSettings.IniFormat)
```

首先获取ini中的配置

```python
self.WebView = QWebEngineView()
```

new 一个QWebEngineView，然后

```python
# 设置加载网页，和网页加载完成以及加载过程信号与槽函数关联
self.WebView.loadStarted.connect(self.loadStarted)
self.WebView.loadFinished.connect(self.loadFinished)
self.WebView.loadProgress.connect(self.loading)

# 重定义QWebEnginePage中javaScriptAlert等函数
self.WebView.page().javaScriptAlert = self._javascript_alert                
self.WebView.page().javaScriptConsoleMessage = self._javascript_console_message
self.WebView.page().javaScriptConfirm = self._javascript_confirm
self.WebView.page().javaScriptPrompt = self._javascript_prompt
```

然后：

```python
# new一个标题栏和状态栏
self.titlebar = titleBar()
self.statusBar = StatusWindow()

# 中心窗口布局，并添加标题栏和状态栏
self.contentLayout = QVBoxLayout()
self.contentWidget = QWidget()
self.contentWidget.gridLayout = QtWidgets.QGridLayout(self.contentWidget)
self.contentWidget.gridLayout.addLayout(self.contentLayout, 0, 0, 1, 1)
self.contentLayout.addWidget(self.WebView)
self.contentWidget.gridLayout.setContentsMargins(0,0,0,0)
self.contentLayout.setContentsMargins(1,0,1,0)
self.contentWidget.setStyleSheet("""
    border-left:    1px solid black;
    border-right:   1px solid black;
    """)

self.main_layout = QVBoxLayout()
self.main_layout.addWidget(self.titlebar)
self.main_layout.addWidget(self.contentWidget)
self.main_layout.addWidget(self.statusBar)
self.main_layout.setSpacing(0)

# 窗口属性
self.setWindowFlags(Qt.Widget | QtCore.Qt.FramelessWindowHint)
self.setAttribute(Qt.WA_NoSystemBackground, True)
self.setAttribute(QtCore.Qt.WA_TranslucentBackground,True)

self.widget = QWidget()
self.setCentralWidget(self.widget)
self.widget.setLayout(self.main_layout)
self.widget.setMouseTracking(True)        
self.resize(650,650)
self.center()

# 将三个按钮点击信号与相关槽函数相关联
self.titlebar.min_button.clicked.connect(self.hideIt)
self.titlebar.max_button.clicked.connect(self.MaxAndNormal)
self.titlebar.close_button.clicked.connect(self.closeIt)

# 状态栏进度条：将LoadProgress信号与loading槽函数相关联
self.WebView.loadProgress.connect(self.loading)
```

最后：

```python
# 设置加载时的网页，显示主窗口，并加载drrr.com
self.WebView.setHtml(WaitingHTML)
self.show()
self.WebView.setStyleSheet("""
    QWebView {
        background-color:black
    }        
    QWebView::QScrollBar:Vertical {
        background-color:black
    }
    """)
self.WebView.load(QUrl("http://drrr.com/"))
```

该类中重写了loadStarted()，loadFinished()和方法，在其中获取网页中用户名等信息，并存储在ini配置文件中，或根据ini配置文件填写网页表单中的内容，由于PyQt5.6去除了mainFrame()方法，所以删除了这部分的操作。至此，整个程序基本结束。

### 移植过后存在的问题
- 程序能运行，但是进入room播放不了音乐，可能是QtWebEngine不支持吧
- 右上角三个按钮的功能没有实现，因为源程序用了animation动画，这个某些设置与PyQt5.6不兼容，也不想再弄了，麻烦。

### 结语
整个项目用来学习PyQt5还算不错。