---
title: blender 学习章
date: 2019-07-10 23:05:54
tags: [游戏美术,建模]
categories: 游戏栈
---

![](/start.jpg)

由于自己是做独立游戏开发的，缄默是少不了的。
开源免费的Blender近年来大热，做一般游戏建模肯定是没问题的，B站这个教程很不错：

#### [【跟顺子老师学3D】blender零基础入门教程【全】](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D1)

一晚上一口气就看了十几课，速度快一点的话两三天就可以学完，Blender的快捷键非常多而且使用频繁，对于一个初学者来说很容易就忘光了，然后回头在视频里查找又是比较没效率的事情，所以必须写点笔记。

### 一、结合Unity

Unity游戏开发用了5年多了，一开始使用blender时很困惑，因为坐标系跟Unity完全不一样， Z轴居然向上，在以前的Unity老版本里好像还要自己进行手动转换坐标，而在当前Unity 2018版本可以直接拖动blender的文件到项目里，会自动转换坐标，转换后的视角相当于blender里的后视图的视角。

> **2018.7.3：今天特意试了一下把blender工程文件直接保存到unity目录下，然后加入到场景中，发现Cycles渲染模式下创建的材质全部都不能正常显示，又试了一下Blender渲染模式可以正常显示，而且在Blender中所作的修改可以在Unity中自动刷新载入显示。其实想想也正常，毕竟Cycles渲染是离线渲染而Unity是实时渲染，无法直接使用是很正常的。**
> **因为我主要应用于Unity，所以平常还是多学习Blender渲染模式，除非一些高质量场景使用烘焙的情况才使用Cycles。**

> **2018.7.6：尝试导出包含骨骼动画的fbx文件然后在Unity中载入。出现了很多奇怪的问题，比如人物部分位置偏移（好像是使用了IK骨的原因）。最终还是使用blender源文件直接载入到Unity中，然后一点问题都没有了，WTF！真是折腾死我了。**

### 二、视频学习笔记

目前Blender 2.8版本还没正式推出，视频里使用的是2.79版，我使用2.79b版本好像没什么区别。

------

#### [01. 简介](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D1)

下载安装软件，没啥可说的。

------

#### [02. 界面及基本操作](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D2)

中文界面设置："File" -> "User Preference"->“System"，将滚动条拉到下面，勾选“International Fonts”在下面，选择Language 为“simplified Chinese(简体中文)”。
勾选下面的“Interface”和“Tooltips”，不要选择“New Data”（因为可能一些插件不支持中文路径。）

同页面的Cycles Compute Device选择CUDA，再选中下面的显卡，这样在主窗口可以选择Cycles渲染。

勾选“界面”页中的“自动透视模式”。

关闭设置窗口前注意要点击左下角“保存用户设置”，否则重启后丢失修改的设置。

滚动鼠标中键：视图缩放。
按住鼠标中键：视图旋转。
SHIFT+鼠标中键：视图平移。
鼠标右键：选中目标。

多窗口分割：拖动窗口边缘的三角形。
多窗口关闭：拖动窗口边缘的三角形到反方向。
每个分割窗口左下角或者右上角按钮可以切换不同视图。
“T键”：切换工具栏显示。
“N键”：切换属性栏显示。
小键盘区：
“5”：透视/正交视图切换。
其他键懒得写了，各种视图切换。

无小键盘（比如笔记本）：启用自带插件“3D View:3D Navigation，然后在工具栏的“显示”页里点击按钮代替小键盘。

选择物体后：
“G键”：（grab)移动物体。然后按X,Y,Z可以根据轴移动。按SHIFT+X/Y/Z可以锁定X/Y/Z，然后根据其他轴移动（此功能还不如直接在正交视图中移动）。
“S键”：（scale)缩放物体。然后按X,Y,Z可以根据轴缩放。
“R键”：（rotate)旋转物体。然后按X,Y,Z可以根据轴旋转。双击“R"实现360度立体旋转。
GSR操作时按住“SHIFT”键可以精细操作。
切换GSR操作的参照坐标系：切换窗口下面的菜单栏里的“全局”和“自身”。

“SHIFT+C”：3D游标恢复到视图中心。
“SHIFT+A”：（add）（在3D游标处）添加物体。
“X键”：删除物体。

------

#### [03. 主题安装及其他补充](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D3)

energy(能量)主题：
原作者：[https://studiollb.wordpress.com/download/](https://links.jianshu.com/go?to=https%3A%2F%2Fstudiollb.wordpress.com%2Fdownload%2F)

------

#### [04. 保存和备份](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D4)

保存时会自动产生一个备份文件。
保存窗口里的“+”、“-”按钮可以自动递增递减文件名。

------

#### [05. 编辑模式基本操作](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D5)

“TAB键”：切换编辑模式/物体模式。
编辑模式：
“CTRL+TAB键”：切换顶点/边/面。
“A键”：全选或者取消全选。
“SHIFT+鼠标右键”：多选点线面。

------

#### [06. 编辑模式下的多选](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D6)

编辑模式：
“CTRL+鼠标左键”：自动选择邻近点线面。
“ALT+鼠标左键”：选中整行或者整列点线面。
“C键”：自由刷选。可以滚动鼠标中键来控制刷选区大小。左键选中，中键按下取消选择。
“B键”：（box)框选。
“CTRL+鼠标左键拖动”：在按下后可以自由拖选。
“L键”：全选鼠标所在点线面的物体。
“CTRL+小键盘加号”：扩展选区。
“CTRL+小键盘减号”：缩减选区。

------

#### [07. 环切](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D7)

编辑模式：
“CTRL+R”：环切。环切时滚动滚轮可以切多刀。

------

#### [08. 细分](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D8)

编辑模式：
细分：选中物体后按“W键”弹出菜单，选择“细分”。细分时工具栏的细分选项页可以控制切割次数。
“SHIFT+R”：(repeat)重做上一次操作。

细分主要对面操作，对边也可以细分但是细分为点。顶点无法细分。

“CTRL+E”：反细分。（不在课程内，自己查的）。

------

#### [09. 挤出](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D9)

编辑模式：
“E键”：选中点线面都可以挤出，但一般主要对面操作。

挤出时右键取消实际上还是挤出了，只不过重叠在原有位置。

环切左键确定后可以按“S键”然后移动坐标轴进行缩放环切线。缩放时可以直接输入缩放倍数，在左下角有显示缩放当前倍数。

------

#### [10. 倒角](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D10)

编辑模式：
选中边，按“CTRL+B”进行倒角，此时可以滚动滚轮增加倒角边。

环切确认后也可以按“CTRL+B”和滚轮增加环切边。

------

#### [11. 合并与分离](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D11)

**物体的合并与分离：**
物体模式：
“CTRL+J”：“join”。合并选中的多个物体。合并后的轴心在最后选中物体的轴心。
编辑模式：
“P键”：选中物体后按P键弹跳出菜单可以进行多种分离模式。没选中任何物体时也可以按P键进行按松散块分离。

**顶点的合并与分离：**
“V键”：选中顶点后进行按键拖动分离。此模式比较坑的一点是就算按鼠标右键取消分离，其实也还是分离了，如果不想分离只能按CTRL+Z撤消操作。
“ALT+M键”：选中两个点按键后弹出菜单合并。

------

#### [12. 创建边与面](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D12)

编辑模式：
“F键”：(fill)选中点或者边，然后按F键进行填充边或者面。

------

#### [13. 衰减编辑](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D13)

编辑模式：
“O键”：启用衰减编辑。底下菜单栏也可以选择，还可以切换衰减模式。
衰减编辑模式：
“G键”：此时滚动鼠标滚轮控制衰减范围。
“ALT+O键”：“衰减编辑模式-相连项”：开启后不会影响不相连的其他物体。

------

#### [14. 轴心点](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D14)

选中多个物体可以通过下面的菜单栏切换轴心点，默认轴心点为质心点。缩放旋转都是根据轴心点进行的。

合并多个物体后，轴心点会被设置到最后一个选中的物体的轴心点上。分离物体后各个物体的轴心点不会恢复的原有的位置。

“CTRL+ALT+SHIFT+C键”：弹出设置原点菜单，可以修改原点。原点即轴心点。

编辑模式：
“SHIFT+S键”：（snap）吸附菜单。可以修改游标位置等。

------

#### [15. 内插面&应用缩放比例](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D15)

实体显示状态：属性栏->着色方式->Mapcap修改编辑时的着色方式，不影响渲染。

编辑模式：
内插面方式一：选择面按E挤出时右键取消实际上还是挤出了，只不过重叠在原有位置。然后按S修改面大小，然后再按E挤出。

内插面方式二：
“I键”：（insert）插入面。然后再按E挤出。

物体模式：
“CTRL+A键”：（apply）应用菜单。缩放物体后比例变化，可以应用比例把比例值变为1，其实意思就是应用保存修改。

------

#### [16. 表面细分](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D16)

选中物体添加修改器，添加表面细分修改器。
表面细分可以配合工具栏->工具中的着色方式（光滑、平直）使用。

快捷键：选中物体按“CTRL+数字键”直接添加表面细分器，数字代表了细分的次数。

------

#### [17. 镜射](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D17)

编辑模式：
切分后线框模式下按B框选可以选择背后的边。按X删除顶点。添加镜射修改器，根据XYZ轴选择镜射。

勾选镜射修改器的范围限制可以防止交界处分离或者越界。

------

#### [18. 布尔](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D18)

添加布尔修改器，指定参照物体。

布尔插件：Bool Tool。添加插件后，在工具栏->工具下有Bool Tool，选中两个物体可以进行布尔操作。

有时按中键移动视图很慢，按小键盘小数点键恢复一下视图即可。

------

#### [19. 修改器顺序](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D19)

修改器不同的顺序产生不同的效果。

实体修改器增加厚度，比如平面、衣服等厚度。

------

#### [20. 相机设置及渲染](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D20)

“SHIFT+Z键”：切换渲染模式（预览）。

开启快捷切换插件：Pie Menu 3D ->View Numpad Pie: Q。 可以快速切换视图。
前视图移动物体时按住CTRL可以按网格移动。
“F12键”：开始渲染。
“M键”：移动选中物体到不同图层。
“小键盘0键”：显示相机视图。

选中相机按N打开属性面板->锁定到视图，可以锁定相机视图，此时移动视图然后渲染所见即所得。

渲染窗口：切换底部“槽”可以保存每次的渲染结果。可以使用大键盘数字键来切换槽。

编辑窗口的大键盘数字键可以切换图层（自己试出来的）。

最右属性栏->世界环境->背景->颜色旁的小点弹出菜单可以选择天空纹理。

------

#### [21. 灯光设置](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D21)

灯光没啥好说的，自己试验修改各种参数看效果。日光只有旋转才会有不同效果，移动位置不影响。

使用给平面增加材质来模拟光源：创建平面，增加纹理，修改漫射BSDF为自发光。
选中这个发光的面，设置物体属性->Cycles 设定->Ray Visibilty->关闭摄像机可以隐藏这个面，但发出的光还是有效。
可以对这个发光的面进行各种编辑（缩放旋转修改形状等）来产生不同的效果。

基本打光技巧，3个光源（3点光）:
主光源放在摄像机位置附近、辅助光源小一点放在摄像机侧面使主光源阴影更少一点，再增加一个光源放在摄像机所对物体后面。两个辅助光源都去掉投射阴影（只有灯光才有投射阴影，如果使用材质模拟光则没有。）。

“ALT+S键”：还原缩放。
“ALT+R键”：还原旋转。

------

#### [22. 提高Cycles渲染速度](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D22)

最右属性栏->渲染->采样：影响渲染效果和速度。
最右属性栏->渲染层->Denoising：降噪功能。此功能在我的烂显卡上不支持GPU渲染，只能CPU渲染。
最右属性栏->渲染->分块大小：影响渲染速度。插件：Auto Tile Size，自动根据机器配置设置分块大小。

最右属性栏->渲染->光程：适当调节减小值可以提高速度，我不想调就用默认的。

------

#### [23. 材质](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D23)

各种材质没啥好说的，自己试效果就可以了。

节点编辑器：各种快捷键跟3D视图编辑器差不多。可以新增着色器进行连线等操作。

------

#### [24. Principled着色器](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D24)

世界环境->使用节点->背景->颜色（点按钮）->环境纹理->添加下载好的HDR文件。

Principled着色器可以调出金属、塑料、透明等效果。

------

#### [25. blender渲染](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D25)

主要是材质和灯光跟Cycle的不同。
优势是速度快，但已经不再更新。

------

#### [26. blender卡通渲染](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D26)

主要讲了Blender渲染模式下的卡通效果，具体的操作还是得参考视频。
freestyle描边。

------

#### [27. principled着色器补充](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D27)

表面细分：光穿透物体的程度。
Subsurface Radius：表面细分范围（RGB三色）。

各向异性过滤：配合Metalic(金属）实现类似拉丝金属。
Anisotropic Rotation：各向异性过滤的旋转角度。

> 我自己网上找的，更详细的Principled 知识参考下面链接：
> [2.79新材质节点 Principled 图解](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.blendercn.org%2F1771.html%2Fcomment-page-1%23comment-634)

------

#### [28. 表皮修改器制做小青蛙](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D28)

创建平面，选中四点，ALT+M合并到中心，添加镜射修改器，挤出顶点，添加表皮修改器，添加表面细分修改器。

表皮修改器标记根结点。
对顶点不能直接使用S键缩放，缩放顶点周边使用CTRL+A。

做出大概形状后应用表皮修改器，再进一步进行细化。
需要先应用镜射修改器再应用表皮修改器。
表皮修改器可以直接创建简单的骨架。

------

#### [29. UV展开与纹理](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D29)

“U键”：UV映射菜单。
“CTRL+E键”：(edge)选中边以后，按快捷键弹出边菜单，选择标记缝合边。
标记缝合边以后再按U->展开UV，然后在底部图像菜单新建图像。
底部UV菜单导出UV布局图。
在外部修改UV图像以后，使用新建材质->颜色->图像纹理->载入图像然后即可在材质视图看到效果。

进入底部菜单的“纹理绘制”模式可以直接进行绘制。

------

#### [30. 相机跟踪](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D30)

创建空物体。
选中摄像机添加标准跟随约束，设置目标为空物体。修改约束中的-Z和向上为Y。
添加曲线->圆环，放大后CTRL+A应用缩放。
选中摄像机添加跟随路径约束，设置目标为圆环。
移动跟随路径约束到标准跟随约束上面。
点击跟随路径约束的动画路径。
播放即可实现动画。
可以对圆环进行各种编辑。

------

#### [31. 父级](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D31)

物体模式：
“CTRL+P键”：设定父级。
“ATL+P键”：清空父级。

------

#### [32.1 骨骼之骨骼建立](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D32)

这节课里的lowpoly死侍模型挺好看，可以学着做一个。

骨架的镜像：
选中骨架->编辑模式->工具栏->选项->X轴镜像
然后按“SHIFT+E键”挤出骨骼即可左右自动对称挤出。
对于手臂和腿的挤出使用“ATL+P键”：断开骨骼连接。这样还保持着父级。

头和臂从身体上段挤出，腿从身体下段挤出。

------

#### [32.2 骨骼之权重绘制](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D33)

把骨架做为物体的父级（“CTRL+P键”），附带自动权重。
“CTRL+TAB键”：骨架姿态模式或者物体权重绘制模式。

权重绘制只对顶点绘制，一般使用工具栏->工具里的F Mix笔刷，权重0表示不受骨骼影响，1表示完全受骨骼影响，其他中间值为部分受骨骼影响。按“F键”可以对笔刷大小缩放。

使用选中物体模块在最右工具栏中数据页指定骨骼的方式可以直接设置权重值。

------

#### [32.3 骨骼之IK骨](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D34)

“SHIFT+I键”：添加IK骨。
默认IK骨的链长为0会影响所有骨骼，要修改为想要的链长。
关节处也要增加IK骨。
精细的多顶点模型有益于Blender创建自动权重的计算。

------

#### [33. 动画](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D35)

“I键”：添加关键帧。
曲线编辑器：“T键”：修改关键帧插值模式。

N键属性栏可以直接修改关键帧变换数值。

材质的颜色也可以增加关键帧。

------

#### [34. 作业](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav24292767%2F%3Fp%3D36)

没啥可说的，开始做作业吧。

------

### 三、自己的学习补充

1. 摄像机视角控制
   小键盘快捷键“0”进入摄像机视图，按“SHIFT+F”，此时可以使用WASD和QE来移动视角（不过有一次我突然发现在此模式下居然无法使用WASDQE移动，然后重启软件后又可以了。)

------

### 四、贴吧看到的Rigify骨架插件使用技巧

还未实践。

> 1.检查模型是否在世界中心，移动/旋转/缩放 模型，并Ctrl+A应用。
> 2.灯光、摄像机移至其他图层，并将模型贴图设置为Shadeless（个人建议）。
> 3.分离模型部件（如有需要的话）——分离的目的是为了待会儿更好地分类绑定与权重绘制。
> 4.开启Rigify插件，Shift+A添加。
> 5.大致匹配Rigify骨架于模型，并Ctrl+A应用骨架。
> 6.Tab进入骨架编辑模式，注意开启左侧竖栏Options选项里面的X-Axis Mirror，X轴镜像编辑。
> 7.对应骨点位置。
> 8.Check Unity Doc. 删除手指根骨节。
> 9.Normal Trans Orientation 去查看每个骨节的朝向。
> 10.Save As，存盘。
> 11.Generate生成带有控制器的骨架，并将原有基础骨架移至其他层。
> 12.Check Unity Doc. 选择相应骨架层，重置某些骨节的父子关系。
> 13.添加附加骨架（如果有需要的话），并为其设置对应的父子关系——需要配合项目骨节数要求。
> 注意：有些L.R对应的镜像骨节，可以在Normal Trans Orientation（Edit Mode）下查看其轴向,
> 以确保其在Pose Mode下可以被统一旋转。
> 14.绑定（为模型与骨架建立父子关系）——建议：Ctrl+P → Automatic Weight
> 15.选中模型，打开修改器面板下的Armature，勾选【Preserve Volume】如果有需要的话。
> 16.权重绘制（巧用笔刷工具与顶点遮罩工具）
> 17.眼球：Modifier, [Track to]
> 18.表情：Shape Key
> 19.武器（props）：利用[Child of]修改器与角色骨架建立关系。
> 20.Remove WGT-Bones in The Hierarchy. Using This Way will Affect Smoothly into Unity.
> 21.用极限POSE去检查绑定。

暂时到这吧～

![](/end.jpg)