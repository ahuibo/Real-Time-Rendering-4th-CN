# Real-Time Rendering 第22章 相交测试方法 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [22.1 GPU加速拾取](<Real-Time_Rendering_4th_中文/第22章/22.01.md>)

- [22.2 定义与工具](<Real-Time_Rendering_4th_中文/第22章/22.02.md>)

- [22.3 包围体的创建](<Real-Time_Rendering_4th_中文/第22章/22.03.md>)

- [22.4 几何概率](<Real-Time_Rendering_4th_中文/第22章/22.04.md>)

- [22.5 经验法则](<Real-Time_Rendering_4th_中文/第22章/22.05.md>)

- [22.6 射线与球体相交](<Real-Time_Rendering_4th_中文/第22章/22.06.md>)

- [22.7 射线与盒相交](<Real-Time_Rendering_4th_中文/第22章/22.07.md>)

- [22.8 射线与三角形求交](<Real-Time_Rendering_4th_中文/第22章/22.08.md>)

- [22.9 射线与多边形求交](<Real-Time_Rendering_4th_中文/第22章/22.09.md>)

- [22.10 平面与盒体相交测试](<Real-Time_Rendering_4th_中文/第22章/22.10.md>)

- [22.11 三角形／三角形相交](<Real-Time_Rendering_4th_中文/第22章/22.11.md>)

- [22.12 三角形与盒的相交](<Real-Time_Rendering_4th_中文/第22章/22.12.md>)

- [22.13 包围体／包围体相交](<Real-Time_Rendering_4th_中文/第22章/22.13.md>)

- [22.14 视锥体相交](<Real-Time_Rendering_4th_中文/第22章/22.14.md>)

- [22.15 直线与直线相交](<Real-Time_Rendering_4th_中文/第22章/22.15.md>)

- [22.16 三个平面的交点](<Real-Time_Rendering_4th_中文/第22章/22.16.md>)


## 章首导言

原书来源：第941—942页（PDF第962—963页）。本文件包含章标题、题辞及22.1节之前的完整导言。

> “我且坐看那朵飘行的小云，
> 会碰上月亮，还是与它擦身而过。”
>
> ——罗伯特·弗罗斯特（Robert Frost）

相交测试在计算机图形学中经常使用。我们可能希望确定两个物体是否碰撞，或者求出到地面的距离，以便让相机保持恒定的高度。另一个重要用途是判断一个物体究竟是否应当被送入流水线。所有这些操作都可以通过相交测试来完成。本章将介绍最常见的射线与物体、物体与物体之间的相交测试。

在同样建立于层次结构之上的碰撞检测算法中，系统必须判断两个基本几何物体是否发生碰撞。这些物体包括三角形、球体、轴对齐包围盒（AABB）、定向包围盒（OBB）和离散定向多面体（k-DOP）。

正如我们在19.4节中所见，视锥体剔除是一种高效舍弃视锥体之外几何体的方法。为了使用这种方法，需要通过测试判断一个包围体（BV）是完全位于视锥体之外、完全位于其中，还是部分位于其中。

上述情形都涉及某一类需要进行相交测试的问题。相交测试判断两个物体 A 和 B 是否相交；可能出现的情况包括 A 完全位于 B 内部（或反过来）、A 与 B 的边界相交，或者二者互不相交。不过，有时还需要更多信息，例如距离某个位置最近的交点，或者相互穿入的深度和方向。

本章重点讨论快速相交测试方法。我们不仅介绍基本算法，还就如何构造新的高效相交测试方法给出建议。当然，本章介绍的方法也适用于离线计算机图形学应用。例如，22.6节至22.9节介绍的射线相交算法就用于光线追踪程序。

简要介绍硬件加速的拾取方法之后，本章将继续介绍一些有用的定义，然后给出为图元构造包围体的算法。接下来介绍构造高效相交测试方法的经验法则。最后，本章的大部分篇幅将以实用方法集的形式介绍各种相交测试方法。


## 22.1 GPU加速拾取

原书来源：第942—943页（PDF第963—964页），从22.1节标题起至22.2节标题之前。

我们常常希望让用户使用鼠标或其他输入设备，通过在某个物体上进行拾取（单击）来选中它。当然，这种操作需要具有很高的性能。

如果需要获得屏幕上某一点或某个较大区域内的所有物体，而不考虑它们是否可见，那么采用 CPU 端的拾取方案可能是合适的。这种拾取有时见于建模或 CAD 软件包中。使用包围体层次结构（19.1.1节），可以在 CPU 上高效地解决这个问题。在像素位置处构造一条射线，使其从视锥体的近平面穿行至远平面。随后根据需要对这条射线与包围体层次结构进行相交测试，这与全局光照算法中用于加速光线追踪的做法类似。对于用户在屏幕上定义的矩形区域，则构造一个视锥体来代替射线，并将其与层次结构进行测试。

根据具体需求，在 CPU 上进行相交测试有若干缺点。对于含有数千个三角形的网格，逐个三角形测试可能代价高昂，除非在网格本身上建立层次结构或网格划分等加速结构。如果精度很重要，那么 CPU 就需要生成与位移映射或 GPU 曲面细分所生成的几何体相匹配的几何体。对于树叶等使用 alpha 映射的物体，用户应当无法选中完全透明的纹素。为了模拟纹理访问，以及任何因各种原因而丢弃纹素的其他着色器行为，CPU 都需要完成大量工作。

通常，我们只需要获得某个像素处或屏幕某一区域内可见的物体。对于这种选择，可以直接使用 GPU 流水线。一种方法最早由 Hanrahan 和 Haeberli [661] 提出。为了支持拾取，在渲染场景时，为每个三角形、多边形或网格物体赋予一个唯一的标识符值，可以将它视为一种颜色。这一思路的目的与可见性缓冲区类似，所形成的图像与第906页的图20.12相似。生成的图像保存在离屏缓冲区中，随后即可用于极快的拾取。当用户单击某个像素时，在这幅图像中查询该像素的颜色标识符，便能立即识别出物体。这些标识符值可以在执行常规渲染的同时，使用简单的着色器渲染到单独的渲染目标中，因此成本相对较低。主要开销可能来自将像素从 GPU 回读到 CPU。

像素着色器接收或计算的任何其他类型的信息，也都可以存储到离屏目标中。例如，法线或纹理坐标就是显而易见的候选信息。借助插值，还可以使用这样的系统求出某个点在三角形内部的相对位置 [971]。在一个单独的渲染目标中渲染每个三角形，并将三角形的三个顶点颜色分别设为红色（255, 0, 0）、绿色（0, 255, 0）和蓝色（0, 0, 255）。假设选中像素的插值颜色为（23, 192, 40），这意味着红色顶点的贡献系数为23/255，绿色顶点为192/255，红色顶点为40/255。这些数值就是重心坐标，22.8.1节将进一步讨论。

> 译注：原文在最后一个贡献系数40/255处写的是“red”（红色），但根据前文的 RGB 顶点赋色以及蓝色通道的数值40，此处应当指蓝色顶点。上文保留原书用词，并在此说明这一排印疑点。

使用 GPU 进行拾取最初是作为三维绘画系统的一部分提出的。这种拾取尤其适合此类系统，因为相机与物体都不移动，因此整个拾取缓冲区只需生成一次，之后便可反复使用。当相机移动时，另一种拾取方法是使用聚焦于屏幕极小区域的离轴相机，将场景再次渲染到一个很小的目标中，例如3 × 3大小的目标。CPU 端的视锥体剔除应当能够消除几乎所有几何体，而且只需对少量像素着色，因此这个渲染遍相对较快。为了拾取所有物体（而不仅是可见物体），可以通过深度剥离，或简单地不再渲染此前已选中的物体，多次执行这种微小窗口方法 [298]。


## 22.2 定义与工具

来源：原书第 943—948 页（PDF 物理页 964—969），从 22.2 节标题起至 22.3 节标题前；包括全部正文、图 22.1—22.6、公式（22.1）—（22.4）及原注 1。

本节介绍对本章整体都有用的记号与定义。

射线 **r**(t) 由起点 **o** 和方向向量 **d** 定义（为方便起见，方向向量通常会归一化，因此 ‖**d**‖ = 1）。其数学表达式见式（22.1），示意见图 22.1：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_0fb6aee8ae4f92.png)


标量 t 是用于生成射线上不同点的变量。t 小于零的点称为位于射线起点后方（因此不属于射线），t 为正的点位于起点前方。另外，由于射线方向已经归一化，一个 t 值所生成的射线上的点，与射线起点之间的距离为 t 个距离单位。

实际使用中，我们通常还保存一个当前距离 l，表示希望沿射线搜索的最大距离。例如，拾取时通常希望找到沿射线最近的交点；比这个交点更远的物体可以放心忽略。


![图22.1 射线及其参数](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_2_22.1.png)

**图 22.1** 一条简单的射线及其参数：**o**（射线起点）、**d**（射线方向），以及用来生成射线上不同点的 t；**r**(t) = **o** + t**d**。

距离 l 的初值为 ∞。每当成功求得与物体的交点时，就用交点距离更新 l。一旦设定 l，射线在测试中就成为一条线段。在下面要讨论的射线／物体相交测试中，我们通常不把 l 纳入讨论。如果希望使用 l，只需要先执行普通的射线／物体测试，再将 l 与算出的交点距离比较，并采取适当的操作。

讨论曲面时，我们区分隐式曲面与显式曲面。隐式曲面由式（22.2）定义：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_cbef20b84441eb.png)


这里，**p** 是曲面上的任意一点。这意味着，把曲面上的一点代入 f，结果就是 0；否则，f 的结果就不为零。隐式曲面的一个例子是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_3d947cdb82fce3.png)，它描述了一个位于原点、半径为 r 的球面。容易看出，这可以改写成 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_64f9d549a51799.png)，因此它确实是隐式的。第 17.3 节简要介绍了隐式曲面；Gomes 等人 [558] 和 de Araújo 等人 [67] 则全面讨论了使用多种隐式曲面进行建模与渲染的方法。

另一方面，显式曲面由向量函数 **f** 和一些参数 (ρ, φ) 定义，而不是用曲面上的点来定义。这些参数产生曲面上的点 **p**。下面的式（22.3）给出了基本思路：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_ab4a2047b90268.png)


显式曲面的一个例子仍然是球面，不过这次以球坐标表示，其中 ρ 为纬度，φ 为经度，如式（22.4）所示：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_366268454e067e.png)


> 译注：原文将 ρ 称为 latitude（纬度）。按式（22.4）本身，ρ 实际是从正 z 轴量起的极角（余纬），而不是从赤道平面量起的通常意义的纬度；这里保留原文表述及公式，并指出这一术语疑点。

再举一个例子，三角形 △![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_778795e4d7147b.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_2564edb9e64007.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_2013f131af37b0.png) 可以写成如下显式形式：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_0b0daa6abfe856.png)，其中必须满足 u ≥ 0、v ≥ 0 和 u + v ≤ 1。

最后，我们给出球体以外的一些常见包围体的定义。

**定义。** 轴对齐包围盒（axis-aligned bounding box，也称矩形盒），简称 AABB，是各面法线与标准基的坐标轴重合的盒体。例如，一个 AABB A 可由两个对角点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_ffbdba56c7dcf7.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_18ef7cbf229fa7.png) 描述，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_3c415c0134c9bb.png)。

图 22.2 给出了三维 AABB 及其记号的示意图。


![图22.2 三维轴对齐包围盒](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_2_22.2.png)

**图 22.2** 三维 AABB A，图中给出了它的极值点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_ffbdba56c7dcf7.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_18ef7cbf229fa7.png)，以及标准基的坐标轴。

**定义。** 有向包围盒（oriented bounding box），简称 OBB，是各面法线两两正交的盒体，也就是经过任意旋转的 AABB。一个 OBB B 可以用盒体中心点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_248c97f9a38908.png) 和三个归一化向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_5317fc2318848a.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_0c9de2a732285e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_7c27ab6d61b60c.png) 描述，这些向量给出盒体各边的方向。它们各自为正值的半长度记作 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_f924e853720df3.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_c5c60e99823c2c.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_c3dc9993cc856a.png)，即从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_248c97f9a38908.png) 到对应面中心的距离。

> 译注：原文“各面法线两两正交”表述不严谨；盒体相对面的法线彼此平行或反向，三组相对面的法线方向之间才两两正交。其后给出的三个轴向向量明确了定义意图。

图 22.3 给出了三维 OBB 及其记号。


![图22.3 三维有向包围盒](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_2_22.3.png)

**图 22.3** 三维 OBB B，图中给出了中心点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_248c97f9a38908.png)，以及归一化、指向各轴正向的边向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_5317fc2318848a.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_0c9de2a732285e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_7c27ab6d61b60c.png)。图中所示的边半长度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_f924e853720df3.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_c5c60e99823c2c.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_c3dc9993cc856a.png)，是从盒体中心到各面中心的距离。

**定义。** k-DOP（离散有向多面体，discrete oriented polytope）由 k/2 个归一化法线（方向）![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_a85255f83fc76e.png) 定义，其中 k 为偶数，1 ≤ i ≤ k/2。每个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_a85255f83fc76e.png) 关联两个标量值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_6c657f2e46ebd8.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_ff6fddb95a4040.png)，满足 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_91a120d1b71407.png)。每个三元组 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_aa23b939ff6f06.png) 描述一个厚平板 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_67481a5c727ed2.png)，即两个平面 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_0fa8d7890a222c.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_11ac044fe3141f.png) 之间的体积；所有厚平板的交集 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_4c21972d91c37b.png) 就是实际的 k-DOP 体积。k-DOP 定义为包围物体的最紧密的一组厚平板 [435]。AABB 和 OBB 均可表示为 6-DOP，因为它们各自由三个厚平板定义六个平面。图 22.4 展示了二维情形下的一个 8-DOP。


![图22.4 茶杯的二维8-DOP](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_2_22.4.png)

**图 22.4** 茶杯的二维 8-DOP 示例，图中显示了所有法线 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_a85255f83fc76e.png)，以及第一个厚平板 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_200cadf1bca082.png) 和它的“尺寸”：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_51d049f8991e8d.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_02_949b120025f40b.png)。

为了定义凸多面体，引入平面的半空间概念很有帮助。正半空间包括所有满足 **n** · **x** + d ≥ 0 的点 **x**，负半空间则为 **n** · **x** + d ≤ 0。

**定义。** 凸多面体是由 p 个平面的负半空间的交集定义的有限体积，其中每个平面的法线均指向多面体外部。

AABB、OBB、k-DOP，以及任意视锥体，都是凸多面体的特殊形式。更复杂的 k-DOP 和凸多面体主要用于碰撞检测算法，因为计算底层网格的精确相交可能代价高昂。构成这些包围体的额外平面能够从包围物体的体积中进一步削去多余部分，因此可能值得付出额外的开销。

另外两种值得关注的包围体是线段扫掠球体和矩形扫掠球体。它们也分别更常被称为胶囊体（capsule）和扁圆体（lozenge），示例见图 22.5。

分离轴指定这样一条直线：两个不重叠（不相交）的物体在这条直线上的投影也不重叠。同样，如果可以在两个三维物体之间插入一个平面，该平面的法线就定义了一条分离轴。下面给出一种重要的相交测试工具 [576, 592]，它适用于 AABB、OBB 和 k-DOP 等凸多面体。这是分离超平面定理 [189] 的一种情形。注1

**注1：** 在计算机图形学中，这个测试有时被称为“分离轴定理”；我们在本书前几版中也助长了这种误称的传播。它本身并不是一个定理，而是分离超平面定理的一个特例。


![图22.5 线段扫掠球体与矩形扫掠球体](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_2_22.5.png)

**图 22.5** 线段扫掠球体与矩形扫掠球体，也称胶囊体与扁圆体。

**分离轴测试（SAT）。** 对任意两个互不相交的凸多面体 A 和 B，至少存在一条分离轴，使这两个多面体在该轴上的投影（形成轴上的区间）也互不相交。如果其中一个物体是凹的，这一结论就不成立。例如，井壁与井内的水桶可能并不接触，但没有任何平面能将它们分开。此外，如果 A 和 B 互不相交，那么它们就可以由一条与下列某项正交的轴分开（也就是说，可以由一个与该项平行的平面分开）[577]：

1. A 的一个面。
2. B 的一个面。
3. 从两个多面体各取的一条边（例如，利用叉积）。

前两项测试表明：如果一个物体完全位于另一个物体的某个面的外侧，那么它们就不可能重叠。在前两项测试处理各个面之后，最后一项测试以物体的边为依据。为了通过第三项测试把物体分开，我们希望插入一个尽可能靠近两个物体的平面（其法线就是分离轴）；这个平面最靠近一个物体时，也只能贴着它的一条边。因此，要测试的每一条分离轴，都是由分别来自这两个物体的一条边的叉积形成的。图 22.6 以两个盒体说明了这个测试。

注意，这里对凸多面体的定义较为宽泛。线段和三角形这样的凸多边形也算凸多面体（不过是退化的，因为它们不围成任何体积）。线段 A 没有面，因此第一项测试就不再需要。第 22.12 节推导三角形／盒体重叠测试，以及第 22.13.5 节推导 OBB／OBB 重叠测试时，都使用了这个测试。Gregorius [597] 指出，对任何使用分离轴的相交测试，都可以采用一项重要优化：时间连贯性。如果本帧找到了一条分离轴，就存储该轴，并在下一帧对这一对物体首先测试它。


![图22.6 盒体的分离轴](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_2_22.6.png)

**图 22.6** 分离轴。将蓝色盒体称为 A，黄色盒体称为 B。第一幅图中，B 完全位于 A 的右侧面的右方；第二幅图中，A 完全位于 B 的左下面的下方。第三幅图中，没有哪个面所在的平面能将另一个盒体完全排除在外，因此，用 A 的右上边和 B 的左下边的叉积形成一条轴，便可确定分隔两个物体的平面的法线。

回到可用方法的讨论，一种常见的相交测试优化技巧，是在一开始就做一些简单计算，以确定射线或物体是否会错过另一个物体。这种测试称为排除测试（rejection test）；如果测试成功，就称相交被排除。

本章经常使用的另一种方法，是将三维物体投影到“最佳”的正交平面（xy、xz 或 yz）上，改为在二维空间中求解问题。

最后，由于数值计算不精确，相交测试中经常使用一个极小的数。这个数记为 ε（epsilon），其取值因测试而异。不过，人们往往只是选择一个能解决程序员手头问题案例的 ε（Press 等人 [1446] 称之为“方便的虚构”），而没有仔细分析舍入误差并相应调整 ε。把这样的代码用于另一种场景时，很可能因条件不同而失效。Ericson 的著作 [435] 在几何计算的背景下深入讨论了数值鲁棒性。在充分强调这一注意事项之后，我们有时仍会尝试给出一些 ε，使它们至少能够作为“正常”数据的合理初值：这些数据尺度较小（例如小于 100、大于 0.1），并且接近原点。


## 22.3 包围体的创建

来源：原书书页 948—953（PDF 物理页 969—974）。本节包含 22.3.1—22.3.4，正文从 22.3 标题开始，止于 22.4 标题之前。

给定一组物体，找到紧密贴合它们的包围体，对于尽量降低相交测试的开销十分重要。任意一条射线击中任意凸物体的概率，与该物体的表面积成正比（第 22.4 节）。尽量减小这一面积，可以提高任何相交算法的效率，因为计算一次排除结果所需的时间，绝不会比计算一次相交结果更长。相比之下，对于碰撞检测算法，通常尽量减小每个包围体（BV）的体积会更好。本节简要介绍：给定一组多边形，如何寻找最优或近似最优的包围体。


![图 22.7 包围球](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_3_22.7.png)

**图 22.7.** 包围球。左图展示最简单的情形：可以在物体的包围盒外面再包一个球。如果物体没有延伸到包围盒的任何角点，就可以改进这个球：使用盒子的中心，遍历所有顶点，找到距离中心最远的顶点，以此设定球的半径，如中图所示。通过移动球心，还可能获得更小的半径，如右图所示。

### 22.3.1 AABB 和 k-DOP 的创建

最容易创建的包围体是轴对齐包围盒（AABB）。沿各个轴，取这组多边形顶点坐标的最小值和最大值，就形成了 AABB。k-DOP 是 AABB 的推广：将顶点投影到 k-DOP 的每一个法线 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_03_7d5a28e25b3fcd.png) 上，把这些投影的极值（min、max）存入 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_03_d3856f297146e0.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_03_00be8e96008a6a.png)。这两个值定义了该方向上最紧的平行平面夹层。所有这些值共同定义一个最小的 k-DOP。

### 22.3.2 球的创建

包围球的构建不像确定平行平面夹层的范围那样直接。有多种算法能够完成这项任务，它们在速度与质量之间有不同的取舍。一种快速、常数时间的单遍算法是：先为这组多边形构造一个 AABB，然后利用盒子的中心和对角线构造球。译注1 这样得到的球有时贴合得很差，但再遍历一遍可能有所改善：以 AABB 的中心作为球形包围体的球心，再次遍历全部顶点，找到离这个中心最远的顶点（比较距离的平方，以避免开平方）。这一距离就是新的半径。见图 22.7。

如果要把子球嵌套在一个父球内，只需稍微修改这两种技术。如果所有子球的半径相同，就可以将它们的球心当作顶点，并在任一种过程结束时，把这个子球半径加到父球的半径上。如果各个半径不同，则可以在边界计算中计入这些半径，求出 AABB 的边界，从而找到一个合理的中心。如果执行第二遍遍历，就把每个子球的半径加到对应点与父球球心之间的距离上。

Ritter [1500] 提出了一种简单算法，用于创建近似最优的包围球。其思路是：分别找到沿 x、y、z 轴处于最小值和最大值位置的顶点。在这三对顶点中，找出相距最远的一对。利用这对顶点构造一个球，球心位于两点的中点，半径等于中点到它们的距离。遍历所有其他顶点，检查它们到球心的距离 d。如果顶点位于半径为 r 的球外，就把球心朝该顶点移动 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_03_795d48ac93a496.png)，把半径设为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_03_452cec38aaf904.png)，然后继续。这一步的作用，是用一个新的球同时包住该顶点和现有的球。第二次遍历完列表之后，就能保证包围球包含全部顶点。

Welzl [1867] 提出了一种更复杂的算法；Eberly [404, 1574]、Ericson [435] 等人对其作了实现，网上提供了代码。其思路是找到一组能够定义球的支撑点。一个球可以由球面上的两个、三个或四个点组成的集合来定义。发现某个顶点处于当前球外时，就将其位置加入支撑集（也可能从集合中移除旧的支撑顶点），计算新球，然后重新遍历整个列表。重复这一过程，直到球包含所有顶点。虽然比前述方法更复杂，但该算法保证找到最优包围球。

Ohlarik [1315] 比较了 Ritter 和 Welzl 两种算法的变体的速度。Ritter 算法的一种简化形式，其开销可能仅比基本版本高 20%；不过它有时会给出更差的结果，因此两种都运行一下是值得的。对于顺序随机化的点列表，Eberly 实现的 Welzl 算法预期具有线性时间复杂度，但运行速度要慢大约一个数量级。

### 22.3.3 凸多面体的创建

凸多面体是一种通用的包围体形式。凸物体可以使用分离轴测试。AABB、k-DOP 和 OBB 都是凸多面体，但还可以找到更紧的边界。我们可以把 k-DOP 理解为通过增加更多成对的平面，进一步削去物体外的体积；类似地，凸多面体可以由任意一组平面定义。削去更多多余体积，就能避免对被包围的多边形物体的整个网格进行开销更大的测试。我们希望像“热缩包装”那样紧裹多边形物体，找到构成其凸包的这组平面。图 22.8 给出了一个例子。例如，可以使用 Quickhull 算法 [100, 596] 求出凸包。尽管名字里有“快速”，这个过程的时间复杂度仍高于线性，因此对于复杂模型，通常将其作为离线预处理执行。

可以看出，这一过程可能产生大量平面，每个平面都由凸包上的一个多边形定义。实际应用中，我们可能不需要这么高的精度。先创建原始网格的简化版本，并可能将其向外扩张，以完全包含原始网格，就会得到精度较低但更简单的凸包。还要注意，对于 k-DOP，随着 k 增大，包围体会越来越接近凸包。


![图 22.8 茶壶的凸包](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_3_22.8.png)

**图 22.8.** 使用 Quickhull [596] 计算出的茶壶凸包。（图片由 Valve 公司的 Dirk Gregorius 提供。）

### 22.3.4 OBB 的创建

一个物体可能天然就有一个有向包围盒（OBB）：它最初有一个 AABB，随后经过旋转，于是这个 AABB 就变成了 OBB。然而，这时使用的 OBB 未必最优。设想建模时让一根旗杆从建筑物上倾斜伸出。围住它的 AABB，就不如沿其长度方向延伸的 OBB 紧密。对于没有明显最佳轴向的模型，由于 OBB 的基底可以任意定向，构建 OBB 比寻找一个合理的包围球还要复杂。

针对这个问题的算法，已有相当多的研究。O’Rourke [1338] 在 1985 年给出的一种精确解法，运行时间为 O(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_03_584fde46074e95.png))。Gottschalk [577] 提出了一种更快、更简单的方法，能够近似求得最佳 OBB。它先计算多边形网格的凸包，避免位于该体积内部的模型顶点使结果产生偏向。随后使用在线性时间内运行的主成分分析（PCA），找出合理的 OBB 轴向。译注2 这种方法的缺点是，所得盒子有时包得不够紧 [984]。Eberly 描述了一种采用最小化技术计算最小体积 OBB 的方法。他对盒子的一组可能方向进行采样，选取其中 OBB 最小的一组轴向，作为数值最小化算法的起点。然后使用 Powell 方向集法 [1446] 寻找最小体积的盒子。Eberly 在网上提供了执行这一操作的代码 [404]。此外还有其他算法；Chang 等人 [254] 对以往工作作了较为全面的概述，并提出了自己的最小化技术，使用遗传算法辅助搜索解空间。


![图 22.9 近似最优 OBB 的构建](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_3_22.9.png)

**图 22.9.** 近似最优 OBB 的构建；请记住，所有点都位于三维空间中。对于 k-DOP 的每个平行平面夹层（用一对彩色线表示），其边界上都有一对点，以黑色标出；底部的两个顶点各自同时是两个夹层平面的极值点。以灰色标出的其他顶点，在后续步骤中不会使用。从这四对点中，取相距最远的两个顶点构成一条边。取距离这条边所在直线最远的极值点，与这条边一起组成三角形。构造三个盒子，每个盒子都使用三角形的一条边来定义其轴向，并使用其余极值点来定义其边界。保存三个盒子中最佳的一个。

这里介绍 Larsson 和 Källberg [984] 的一种算法。这是一种不需要凸包、在线性时间内执行的近似最优方法。它通常比 Gottschalk 基于 PCA 的方法质量更好，执行速度快得多，适合 SIMD 并行化，而且作者提供了代码。首先为物体构造一个 k-DOP，并为 k-DOP 的每个平行平面夹层保存一对接触其相对两侧的顶点（任取一对即可）。所有这些顶点对合在一起，称为物体的极值点。例如，26-DOP 会产生 13 对点，其中一些点可能指向同一个顶点，因此整体点集可能更小。将包围物体的 AABB 初始化为“最佳 OBB”。接着，算法寻找可能贴合得更好的 OBB 朝向。先构造一个较大的底三角形，再从其面向外扩展出两个四面体。由此形成一组共七个三角形，它们能够产生可能接近最优的 OBB。

彼此相距最远的一对点构成底三角形的一条边。在剩余极值点中，取距离这条边所在直线最远的顶点，作为三角形的第三个点。对三角形的每条边，使用该边以及三角形平面内垂直于该边的法线，构成一个候选新 OBB 的两条轴。将其余极值点投影到这些轴上，分别求出三个 OBB 在该平面内的二维边界。见图 22.9。用最小的二维包围矩形，从三者中选出最佳 OBB。由于这三个 OBB 的高度，也就是沿三角形法线方向的距离，完全相同，因此每个 OBB 对应的二维包围盒，就足以判定哪一个最好。

然后将其余极值点向三角形法线方向投影，求出这个 OBB 在三维空间中的范围。把这样完整构造出来的 OBB 与初始 AABB 比较，判断哪个更好。在这一过程中找到的两个极值点，一个位于最大高度，另一个位于最小高度，随后用来构造两个四面体，每个四面体都以原来的大三角形为底。每个四面体又形成三个额外的三角形；对其中每一个，都像处理原始三角形一样，评估由该三角形产生的三个候选 OBB。与之前一样，将每个三角形的最佳二维 OBB 沿高度方向扩展，不过这次只是为了得到候选 OBB 的最终尺寸，不再用它构造更多三角形。总共形成七个三角形，每个三角形各生成一个完整的 OBB，并参与比较。

找到最佳 OBB 后，将原始物体中的所有点投影到它的轴上，根据需要增大盒子的尺寸。最后再与原始 AABB 比较一次，检查这个 OBB 是否确实贴合得更好。整个过程比以前的技术更快，这得益于大多数步骤都只使用一小组极值点。值得注意的是，作者倾向于根据包围盒的表面积而非体积进行优化，原因将在下一节说明。

**译注1：** 原文在此写作“constant-time single pass algorithm”（常数时间的单遍算法）。若包括为任意数量顶点构建 AABB 的遍历，其总耗时随顶点数增长；在已经得到 AABB 后，由其中心与对角线确定球才是常数时间操作。正文保留原书表述；另须注意，上述对角线长度是球的直径，半径为其一半。

**译注2：** 原文写作“Principle component analysis”，标准术语应为“Principal component analysis”，此处按主成分分析翻译。


## 22.4 几何概率

原书页码：953—954（PDF 第 974—975 页）。本节从“22.4 Geometric Probability”开始，至“22.5 Rules of Thumb”标题之前结束。

常见的几何运算包括判断一个平面或一条射线是否与物体相交，以及一个点是否位于物体内部。与此相关的一个问题是：一个点、一条射线或一个平面与物体相交的相对概率是多少？空间中一个随机点位于物体内部的相对概率相当直观：它与物体的体积成正比。因此，一个 1 × 2 × 3 的盒子包含某个随机选取点的可能性，是一个 1 × 1 × 1 盒子的 6 倍。

对于空间中的任意一条射线，它与一个物体相交的可能性，相对于与另一个物体相交的可能性是多少？这个问题与另一个问题有关：采用正交投影时，一个任意朝向的物体平均会覆盖多少个像素？正交投影可以看作视景体内的一组平行射线，每个像素都有一条射线穿过。给定一个朝向随机的物体，它覆盖的像素数就等于与它相交的射线数。

答案出人意料地简单：任何凸实体的平均投影面积，都是其表面积的四分之一。对于屏幕上的球体，这一点显然成立：它的正交投影始终是面积为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_04_e4686f81040cde.png) 的圆，而它的表面积为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_04_5a92c4f6f1b041.png)。对于其他任意朝向的凸物体，例如盒子或 k-DOP，其平均投影面积也满足相同的比例关系。非形式化的证明可参见 Nienhuys 的文章 [1278]。

球体、盒子或其他凸物体，在其覆盖的每个像素处总有一个正面和一个背面，因此深度复杂度为二。这种概率度量可以推广到任意多边形，因为一个（双面）多边形的深度复杂度始终为一。因此，任意多边形的平均投影面积都是其表面积的一半。

在光线追踪文献中，这种度量称为**表面积启发式**（surface area heuristic，SAH）[71, 1096, 1828]，它对于为数据集构建高效的可见性结构十分重要。其用途之一是比较包围体的效率。例如，与一个内接于球体的立方体（即各顶点都与球面接触的立方体）相比，球体被射线击中的相对概率为 1.57（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_04_f86dd03e3d1ffe.png)）。同样，与内接于立方体的球体相比，立方体被击中的相对概率为 1.91（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_04_c87b35c7083a37.png)）。这类概率度量可以用于细节层次计算等领域。例如，设想一个细长物体，它覆盖的像素远少于一个形状较圆的物体，但两者的包围球大小相同。通过包围盒的表面积预先得知命中比例后，就可以认为这个细长物体在视觉影响方面的相对重要性较低。

现在我们知道，点被包含的概率与体积有关，而射线相交的概率与表面积有关。平面与盒子相交的可能性，和盒子在三个维度上的尺寸之和成正比 [1580]。这个和称为物体的**平均宽度**（mean width）。例如，边长为 1 的立方体，其平均宽度为 1 + 1 + 1 = 3。盒子的平均宽度与它被平面击中的可能性成正比。因此，1 × 1 × 1 的盒子对应度量值 3，而 1 × 2 × 3 的盒子对应度量值 6，这意味着后者被任意平面相交的可能性是前者的两倍。

不过，这个和大于真正的**几何平均宽度**（geometric mean width）；后者是指遍历物体所有可能的朝向时，它沿某一固定轴的投影长度的平均值。在不同类型的凸物体之间，并不存在类似表面积那样简单的关系，可供计算平均宽度。直径为 d 的球体，其几何平均宽度就是 d，因为无论朝向如何，球体沿该轴跨越的长度都相同。关于这个话题，我们最后只说明一点：将盒子的各维尺寸之和（即这里所说的平均宽度）乘以 0.5，就得到它的几何平均宽度；这个值可以直接与球体的直径比较。因此，度量值为 3 的 1 × 1 × 1 盒子，其几何平均宽度为 3 × 0.5 = 1.5。包围这个盒子的球体，其直径为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_04_ed7e018f25e852.png)。因此，包围立方体的球体被任意平面相交的可能性，是该立方体的 1.732/1.5 = 1.155 倍。

这些关系有助于判断各种算法能带来多大的收益。视锥体剔除尤其适合应用这些关系，因为它涉及平面与包围体的相交测试。另一个用途是确定：对于一个包含物体的 BSP 节点，是否应当划分，以及在哪里划分最佳，从而改善视锥体剔除的性能（第 19.1.2 节）。


## 22.5 经验法则

来源：原书第 954—955 页（PDF 第 975—976 页）。本节从“22.5 Rules of Thumb”标题起，至“22.6 Ray/Sphere Intersection”标题前止。

在开始研究具体的相交方法之前，先介绍一些经验法则，它们有助于让相交测试更快、更稳健，也更精确。在设计、构思和实现相交测试例程时，应当牢记这些法则：

- 尽早执行那些可能轻易排除或确认各种相交情况的计算与比较，以便提前退出，避免进一步计算。
- 如果可能，利用先前测试的结果。
- 如果使用了不止一项排除或确认测试，那么在可能的情况下，尝试改变它们的内部执行顺序，因为这样可能得到效率更高的测试。不要以为看起来微不足道的改动就不会产生影响。
- 将代价高昂的计算（尤其是三角函数、平方根和除法）推迟到真正需要时再执行（第 22.8 节给出了推迟一次高开销除法的示例）。
- 降低问题的维度，往往可以显著简化相交问题，例如从三维降至二维，甚至降至一维。示例见第 22.9 节。
- 如果一次需要将同一条射线或同一个物体与许多其他物体进行比较，应当寻找能够在测试开始前仅执行一次的预计算。
- 当相交测试代价高昂时，通常适合先用包围物体的球体或其他简单包围体（BV）进行测试，作为第一级快速排除手段。
- 养成始终在自己的计算机上进行计时比较的习惯，并在计时时使用真实的数据和测试情境。
- 利用前一帧的结果。例如，如果在前一帧中发现某条轴可以将两个物体分离，那么在下一帧中先尝试这条轴，可能是个好办法。
- 最后，尽量使代码稳健。这意味着它应当能够处理所有特殊情况，并尽可能不受各种浮点精度误差的影响。同时，要清楚代码可能存在的任何局限。关于数值稳健性和几何稳健性的更多信息，请参阅 Ericson 的著作 [435]。

最后，我们强调，对于某一种具体测试，很难确定是否存在“最佳”算法。评估时，人们常常使用具有一组不同预设命中率的随机数据，但这只能反映部分实际情况。算法最终会用于真实场景，例如游戏，因此最好在这样的环境中进行评估。使用的测试场景越多，对性能问题的理解就越深入。在某些架构上，例如 GPU 和宽 SIMD 实现，由于需要执行多个排除分支，性能可能会下降。最好避免想当然，而是制定扎实的测试计划。


## 22.6 射线与球体相交

来源：原书第955—959页（PDF物理页976—980），从22.6节标题起至22.7节标题前，包含22.6.1与22.6.2全部内容。

我们先从一个数学上简单的相交测试开始，即射线与球体的相交测试。后面将会看到，只要开始从相关几何关系着眼，就能让直接的数学解法变得更快 [640]。


![图22.10 射线与球体的三种相交情况](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_6_22.10.png)

**图22.10** 左图中的射线未命中球体，因此![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_f7e47c1bd9f48e.png) − c < 0。中图中的射线与球体相交于两点（b² − c > 0），这两个点由标量![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_52e22163357e7d.png)与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_4ba997395871ae.png)确定。右图表示![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_f7e47c1bd9f48e.png) − c = 0的情况，此时两个交点重合。

### 22.6.1 数学解法

球体可由球心 **c** 和半径r定义。因此，与前面介绍的表达式相比，球体有一个更紧凑的隐式公式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_9cce840808ad0a.png)


其中，**p** 是球面上的任意一点。为求射线与球体的交点，只需用射线 **r**(t) 替换式（22.5）中的 **p**，得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_0fe450acbfb10b.png)


利用式（22.1），即 **r**(t) = **o** + t**d**，可将式（22.6）化简如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_d9d7ba49913430.png)


最后一步利用了 **d** 已归一化这一假设，即 **d** · **d** = ‖**d**‖² = 1。所得方程是二次多项式，这并不令人意外；这意味着，如果射线与球体相交，最多会有两个交点，见图22.10。如果方程的解为虚数，那么射线未命中球体；否则，可以把两个解![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_52e22163357e7d.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_4ba997395871ae.png)代入射线方程，计算球面上的交点。

得到的式（22.7）可写成二次方程：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_1016d71a4a3d99.png)


其中b = **d** · (**o** − **c**)，c = (**o** − **c**) · (**o** − **c**) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_730f558bb648d5.png)。这个二次方程的解如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_fee3aef16e832a.png)


注意，如果![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_f7e47c1bd9f48e.png) − c < 0，射线就未命中球体，此时可以判定不相交，并省去后续计算（例如开平方以及一些加法）。若通过此测试，就可以计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_b9388f1a9164b0.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_fa101f5dfe8de0.png)。还需要再做一次比较，找出![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_9d5212cdbc9a74.png)与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_52e22163357e7d.png)中最小的正值。关于另一种数值上更稳定的二次方程求解方法，可参阅realtimerendering.com上的碰撞检测章节 [1446]。

如果改从几何角度审视这些计算，就能发现更好的排除测试。下一小节介绍这样的一个例程。

### 22.6.2 优化解法

对于射线与球体相交的问题，我们首先注意到，射线起点后方的交点并不是所需的。例如，拾取通常就是如此。为了及早检查这一情况，先计算向量 **l** = **c** − **o**，它是从射线起点指向球心的向量。所用的全部记号见图22.11。同时计算该向量的长度平方，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_85818320b7a2e8.png) = **l** · **l**。如果![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_85818320b7a2e8.png) < ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_730f558bb648d5.png)，就表明射线起点位于球体内部，这又意味着射线必定命中球体；若只想检测射线是否命中球体，此时即可退出；否则继续。接下来，计算 **l** 在射线方向 **d** 上的投影：s = **l** · **d**。

现在进行第一个排除测试：如果s < 0，并且射线起点在球体外部，那么球体位于射线起点后方，可以判定不相交。否则，利用勾股定理计算球心到投影点的距离平方：m² = l² − s²。第二个排除测试比第一个更简单：若m² > ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_730f558bb648d5.png)，射线肯定未命中球体，可以放心省去其余计算。如果球体和射线通过了这最后一个测试，那么射线必定命中球体；如果只关心是否命中，此时便可退出。

为了求出实际交点，还需要做一点工作。首先计算距离平方![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_9a78a21dbbc024.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_730f558bb648d5.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_ffbbe823ae3e15.png)。注2 见图22.11。由于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_ffbbe823ae3e15.png) ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_730f558bb648d5.png)，因此![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_9a78a21dbbc024.png)大于或等于零，这意味着可以计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_32ae5363510719.png)。最后，到交点的距离为t = s ± q，其解的形式与前面数学解法一节中得到的二次方程解十分相似。

**注2：** 可以只计算一次标量![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_730f558bb648d5.png)，并将它存储在球体的数据结构中，以求进一步提高效率。实际上，这样的“优化”也可能更慢，因为它需要访问更多内存，而内存访问是影响算法性能的主要因素之一。


![图22.11 优化的射线与球体相交测试的几何记号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_6_22.11.png)

**图22.11** 优化的射线与球体相交测试所用的几何记号。左图中，射线与球体相交于两点，沿射线到这两点的距离为t = s ± q。中图展示了球体位于射线起点后方时作出的排除判定。最后，右图中的射线起点位于球体内部，此时射线总会命中球体。

如果只关心第一个正向交点，那么，射线起点位于球体外部时，应采用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_52e22163357e7d.png) = s − q；起点位于内部时，应采用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_4ba997395871ae.png) = s + q。将相应的t值代入射线方程（式（22.1）），即可得到实际交点。

优化版本的伪代码如下。该例程返回一个布尔值：射线未命中球体时为REJECT，否则为INTERSECT。如果射线与球体相交，还会返回从射线起点到交点的距离t，以及交点 **p**。

```text
RaySphereIntersect(o, d, c, r)
返回 ({REJECT, INTERSECT}, t, p)
 1: l = c − o
 2: s = l · d
 3: l² = l · l
 4: 如果 (s < 0 且 l² > r²)，返回 (REJECT, 0, 0);
 5: m² = l² − s²
 6: 如果 (m² > r²)，返回 (REJECT, 0, 0);
 7: q = √(r² − m²)
 8: 如果 (l² > r²)，t = s − q
 9: 否则 t = s + q
10: 返回 (INTERSECT, t, o + td);
```

注意，在第3行之后，可以测试 **p** 是否位于球体内部；如果我们只想知道射线与球体是否相交，那么在相交时，例程即可终止。译注1 此外，第6行之后，射线就保证会命中球体。如果进行运算次数统计（计算加法、乘法、比较及类似操作的次数），会发现，在完整执行至结束的情况下，几何解法与前面给出的代数解法大致相当。重要的差别在于，排除测试在计算过程中的执行时机提前了许多，因此，该算法的平均总成本更低。

对于射线与其他一些二次曲面及混合物体之间的求交，也有优化过的几何算法。例如，已有针对圆柱 [318, 713, 1621]、圆锥 [713, 1622]、椭球、胶囊体以及圆角矩形体（lozenge）[404] 的方法。

**译注1：** 原文此处写作“测试p是否在球体内部”，但依前文定义，p是尚未求出的交点；第3行计算的是射线起点o到球心的距离平方。因此这里疑应指测试起点o是否在球体内部。译文保留原文点名，不暗改。另，原文前段及图22.10用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_52e22163357e7d.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_4ba997395871ae.png)标记两个交点，而式（22.9）后的正文改用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_9d5212cdbc9a74.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_06_52e22163357e7d.png)；本译文也保留该编号变化。二次方程中的标量c与粗体球心向量c同名，按原书字体区分。


## 22.7 射线与盒相交

来源：原书第 959—962 页（PDF 第 980—983 页）。范围从 22.7 节标题开始，到 22.8 节标题之前，包含 22.7.1 和 22.7.2。

下面给出三种判断射线是否与实心盒相交的方法。第一种同时处理轴对齐包围盒（AABB）和有向包围盒（OBB）。第二种是一种通常更快的变体，但只能处理较简单的 AABB。第三种基于第 947 页的分离轴测试，只处理线段与 AABB 的相交问题。这里使用第 22.2 节中包围体（BV）的定义和记号。

> 译注：原文此处写“三种方法”，但本节实际只设置了“平板法”和“射线斜率法”两个下级小节；分离轴测试在平板法的优化讨论中被提及。此处保留原文陈述，不另补写第三种算法。

### 22.7.1 平板法

一种射线与 AABB 相交的方案基于 Kay 和 Kajiya 的平板法（slab method）[640, 877]，该方法又受到 Cyrus–Beck 线裁剪算法的启发 [319]。

我们将这一方案扩展到更一般的 OBB 包围体。它返回最近的正 t 值，也就是从射线原点 **o** 到交点的距离（如果存在交点）。在介绍一般情形之后，我们再讨论针对 AABB 的优化。解决问题的途径是：计算射线与 OBB 各个面所在平面相交时的所有 t 值。把盒看成三个平板的集合；图 22.12 左侧用二维情况说明了这一点。对每个平板，都有最小和最大的 t 值，分别称为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_16a767040260cd.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_a62fda5d413c65.png)，其中 i ∈ {u, v, w}。下一步是计算式（22.10）中的变量：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_1ea6bad3387e1a.png)


![图22.12 平板法的二维示意](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_7_22.12.png)

**图 22.12。** 左图展示由两个平板构成的二维 OBB，右图展示两条接受 OBB 相交测试的射线。图中标出了所有 t 值，绿色平板使用下标 u，橙色平板使用下标 v。极端的 t 值用方框标出。左边的射线击中 OBB，因为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_895ea9b45ab9c1.png) < ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_21db721743681f.png)；右边的射线未击中，因为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_21db721743681f.png) < ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_895ea9b45ab9c1.png)。

现在来看这个巧妙的测试：如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_895ea9b45ab9c1.png) ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_21db721743681f.png)，那么射线所定义的直线与盒相交；否则不相交。换句话说，我们求出每个平板的近交点距离和远交点距离。如果所得“近”距离中最远的一个，小于或等于“远”距离中最近的一个，那么射线所定义的直线就击中了盒。仔细观察图 22.12 右侧的示意图，便可以理解这一点。这两个距离定义了直线上的交点，所以，如果最近的“远”距离非负，那么射线本身就击中了盒，也就是说，盒没有位于射线后方。

下面给出 OBB（A）与射线（由式（22.1）描述）之间的射线/OBB 相交测试伪代码。代码返回一个布尔值，表示射线是否与 OBB 相交（INTERSECT 或 REJECT），还返回到交点的距离（如果存在交点）。回顾一下：对于 OBB A，中心记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_02e2955f87ff25.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_296c7d5f7e84fb.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_db493932eacf8c.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_22a45635262ad8.png) 是盒各边的归一化方向；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_e19f63929be1ad.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_a7199a17e538ce.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_0c1dccbb7e236e.png) 是正的半长度，即从中心到盒面的距离。

```text
RayOBBIntersect(o, d, A)
返回 ({REJECT, INTERSECT}, t);
 1: tᵐⁱⁿ = −∞
 2: tᵐᵃˣ = ∞
 3: p = aᶜ − o
 4: 对每个 i ∈ {u, v, w}
 5:     e = aⁱ · p
 6:     f = aⁱ · d
 7:     if (|f| > ϵ)
 8:         t₁ = (e + hᵢ)/f
 9:         t₂ = (e − hᵢ)/f
10:         if (t₁ > t₂) swap(t₁, t₂);
11:         if (t₁ > tᵐⁱⁿ) tᵐⁱⁿ = t₁
12:         if (t₂ < tᵐᵃˣ) tᵐᵃˣ = t₂
13:         if (tᵐⁱⁿ > tᵐᵃˣ) return (REJECT, 0);
14:         if (tᵐᵃˣ < 0) return (REJECT, 0);
15:     else if (−e − hᵢ > 0 or −e + hᵢ < 0) return (REJECT, 0);
16: if (tᵐⁱⁿ > 0) return (INTERSECT, tᵐⁱⁿ);
17: else return (INTERSECT, tᵐᵃˣ);
```

第 7 行检查射线方向是否不垂直于当前受测平板的法线方向。换句话说，它测试射线是否不平行于平板的两个平面，从而能够与它们相交。注意，这里的 ϵ 是一个极小的数，数量级约为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_05afafc20f4b2d.png)，其作用仅仅是避免除法发生溢出。第 8 行和第 9 行都要除以 f；在实际实现中，通常先计算一次 1/f，再乘以这个值会更快，因为除法往往开销很大。第 10 行确保 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_52e22163357e7d.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_4ba997395871ae.png) 中的较小值存储在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_52e22163357e7d.png) 中，因此较大值存储在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_4ba997395871ae.png) 中。实际实现并不一定要进行交换；可以在该分支中重复第 11 行和第 12 行，并在那里调换 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_52e22163357e7d.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_4ba997395871ae.png) 的位置。如果第 13 行返回，那么射线没有击中盒；类似地，如果第 14 行返回，那么盒位于射线原点后方。射线平行于平板、因而不能与其相交时，会执行第 15 行；这一行测试射线是否位于平板之外。若是，射线便没有击中盒，测试结束。为了进一步加快代码，Haines 讨论了一种展开循环的方法，可以借此省去一些代码 [640]。

还有一项测试没有写进伪代码，但值得加入实际代码中。如定义射线时所述，我们通常希望找到最近的物体。因此，在第 15 行之后还可以测试 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_895ea9b45ab9c1.png) ≥ l 是否成立，其中 l 是当前射线长度。这实际上把射线当作线段处理。如果新交点并不更近，就拒绝这一相交结果。这项测试可以推迟到整个射线/OBB 测试完成之后，但在循环内部尝试提前拒绝通常效率更高。

对于 OBB 恰好是 AABB 的特殊情况，还有其他优化。第 5 行和第 6 行变为 e = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_453340eada2895.png) 和 f = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_3e7a00b9b5a749.png)，这会使测试更快。通常在第 8 行和第 9 行使用 AABB 的角点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_ffbdba56c7dcf7.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_07_18ef7cbf229fa7.png)，从而省去加法和减法。Kay 和 Kajiya [877] 以及 Smits [1668] 指出，通过允许除以 0 并正确解释处理器的结果，可以省去第 7 行。Kensler [1629] 给出了这一测试的精简版本代码。Williams 等人 [1887] 提供了正确处理除以 0 的实现细节，以及其他优化。Aila 等人 [16] 展示了在某些 NVIDIA 架构上，如何用单次 GPU 操作完成“最小值中的最大值”测试，或者反过来的测试。也可以使用分离轴测试（SAT）推导射线与盒的测试，但这样得到的结果不包含相交距离，而相交距离往往很有用。

平板法的推广形式可用于计算射线与 k-DOP、视锥体或任意凸多面体的相交；网上提供了代码 [641]。

### 22.7.2 射线斜率法

2007 年，Eisemann 等人 [410] 提出了一种与盒求交的方法，看起来比此前的方法更快。它不进行三维测试，而是将射线与盒在二维中的三个投影分别进行测试。关键思想是：对每个二维测试，都有两个盒角点界定射线所“看到”的极端范围，类似于模型的轮廓边。要与盒的这一投影相交，射线斜率必须位于由射线原点与这两个点分别定义的两条直线的斜率之间。如果三个投影全部通过这一测试，射线就一定击中盒。这种方法极快，因为一些用于比较的项完全取决于射线自身的数值。只计算一次这些项，随后就能高效地将该射线与大量盒进行比较。此方法可以仅返回是否击中盒，也可以付出少量额外开销，同时返回相交距离。


## 22.8 射线与三角形求交

来源：原书第 962—966 页，对应 PDF 第 983—987 页；范围从 22.8 节标题起，至 22.9 节标题前，包含 22.8.1 与 22.8.2。

在实时图形库和 API 中，三角形几何通常存储为一组带有相应着色法线的顶点，每个三角形由其中三个顶点定义。三角形所在平面的法线往往并未存储，此时若需要使用，就必须计算它。射线与三角形的相交测试有许多种，其中很多先计算射线与三角形所在平面的交点。然后，将交点和三角形顶点投影到使三角形面积最大的轴对齐平面（xy、yz 或 xz）上。这样就把问题降到了二维，只需判断这个二维点是否位于二维三角形内部。已有多种此类方法，Haines [642] 对它们进行了评述和比较，网上也提供了代码。第 22.9 节介绍了一种采用该技术的常用算法。研究者针对不同的 CPU 架构、编译器和命中率评估了大量算法 [1065]，但无法得出某一种测试在所有情况下都最优的结论。

这里重点介绍一种不假定法线已经预计算的算法。对于三角形网格，这可以节省相当可观的内存。对于动态几何，也不必每帧重新计算三角形的平面方程。该算法直接依据三角形的顶点进行测试，而不是先测试射线与三角形所在平面的相交，再检查交点是否落在三角形的二维表示之内。Möller 和 Trumbore [1231] 讨论了这一算法及其优化，这里采用他们的讲解方式。Kensler 和 Shirley [882] 指出，大多数直接在三维中进行的射线与三角形相交测试在计算上是等价的。他们开发了利用 SSE 将四条射线与一个三角形进行测试的新方法，并使用遗传算法，为这种等价测试寻找最佳运算顺序。他们的论文给出了性能最好的测试代码。需要注意，这方面存在大量不同方法。例如，Baldwin 和 Weber [96] 提供了一种采用不同空间与速度权衡的方法。这类测试的一个潜在问题是：恰好与三角形边或顶点相交的射线，可能被判定为未命中三角形。这意味着，射线有可能击中两个三角形的公共边而穿过网格。Woop 等人 [1906] 提出了一种在边和顶点处均保持水密性的射线与三角形相交测试。其性能会稍低一些，具体取决于所用的遍历方式。

使用式（22.1）中的射线，测试它与由三个顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_19aa1dc33bda1e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_3a55303a18d915.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_6aa5478e61d94d.png) 定义的三角形，即 △![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_6aa5478e61d94d.png)，是否相交。

> 译注：原书本段将顶点记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_19aa1dc33bda1e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_3a55303a18d915.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_6aa5478e61d94d.png)，而从式（22.11）起及后续推导、图 22.14、伪代码均使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c11de44d831b47.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_19aa1dc33bda1e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_3a55303a18d915.png)。这里保留原书各处的编号；它们均指三角形的三个顶点。


![图22.13 三角形的重心坐标](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_8_22.13.png)

**图 22.13** 三角形的重心坐标，以及若干示例点的取值。在三角形内部，u、v、w 的取值均介于 0 和 1 之间；在整个平面上，这三个值之和始终为 1。这些值可以作为权重，表示三个顶点各自的数据对三角形上任意一点的影响。注意，在每个顶点处，一个值为 1，其余两个为 0；而在各条边上，总有一个值为 0。

### 22.8.1 求交算法

三角形上的一点 f(u, v) 可由以下显式公式给出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_34489706f78a06.png)


其中，(u, v) 是两个重心坐标，必须满足 u ≥ 0、v ≥ 0，以及 u + v ≤ 1。注意，(u, v) 可以用于纹理映射、法线插值或颜色插值等操作。也就是说，u 和 v 是各顶点对某一特定位置的贡献所使用的权重，第三个权重为 w = (1 − u − v)。在其他文献中，这些坐标常记为 α、β 和 γ。为了便于阅读并保持记号一致，这里使用 u、v 和 w。见图 22.13。

计算射线 r(t) 与三角形 f(u, v) 的交点，等价于求解 r(t) = f(u, v)，由此得到：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_22002b96147ee7.png)


将各项重新排列，得到：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c78566814716b3.png)


这意味着，通过求解这一线性方程组，就能求得重心坐标 (u, v) 以及从射线原点到交点的距离 t。


![图22.14 射线原点的平移与换基](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_8_22.14.png)

**图 22.14** 射线原点的平移与换基。

从几何角度看，上述运算可以理解为：将三角形平移到原点，再将其变换成 y、z 方向上的单位三角形，同时使射线方向与 x 方向对齐。图 22.14 展示了这一过程。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c13b49e2c5ce4e.png) 是式（22.13）中的矩阵，那么将式（22.13）左乘 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_81a6d74be42f21.png) 即可得到解。

记 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_392783cc92c86c.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_19aa1dc33bda1e.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c11de44d831b47.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_f92bdb6d943d5e.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_3a55303a18d915.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c11de44d831b47.png)、s = o − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c11de44d831b47.png)，利用克拉默法则可得到式（22.13）的解：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_1f5862f88186d9.png)


根据线性代数，我们知道 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c0d108d41db77d.png)。因此，式（22.14）可以改写为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_05518ca126caf2.png)


其中，q = d × ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_f92bdb6d943d5e.png)，r = s × ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_392783cc92c86c.png)。利用这些因子可以加快计算。

如果能够承担一些额外的存储开销，就可以重新表述这一测试，以减少运算次数。式（22.15）可改写为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c70f87fa734778.png)


其中，n = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_392783cc92c86c.png) × ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_f92bdb6d943d5e.png) 是三角形未经归一化的法线，因此对于静态几何而言是常量；m = s × d。如果为每个三角形存储 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_c11de44d831b47.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_392783cc92c86c.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_f92bdb6d943d5e.png) 和 n，就可以省去射线与三角形求交中的许多运算。收益主要来自省去一次叉积。应当指出，这违背了该算法最初的思想，即只为三角形存储尽可能少的信息。不过，如果速度是首要考虑因素，这可能是一个合理的替代方案。需要权衡的是，额外的内存访问是否会抵消所节省的计算。最终只有仔细测试，才能确定哪种方式最快。

### 22.8.2 实现

下面的伪代码概括了该算法。除了返回射线是否与三角形相交之外，算法还返回前面介绍的三元组 (u, v, t)。这段代码不剔除背向三角形，也会返回 t 为负值的交点；如果需要，也可以将这些情况剔除。

```text
RayTriIntersect(o, d, p₀, p₁, p₂)
返回 ({REJECT, INTERSECT}, u, v, t)；
 1：e₁ = p₁ − p₀
 2：e₂ = p₂ − p₀
 3：q = d × e₂
 4：a = e₁ · q
 5：如果 (a > −ε 且 a < ε)，返回 (REJECT, 0, 0, 0)；
 6：f = 1/a
 7：s = o − p₀
 8：u = f(s · q)
 9：如果 (u < 0.0)，返回 (REJECT, 0, 0, 0)；
10：r = s × e₁
11：v = f(d · r)
12：如果 (v < 0.0 或 u + v > 1.0)，返回 (REJECT, 0, 0, 0)；
13：t = f(e₂ · r)
14：返回 (INTERSECT, u, v, t)；
```

其中，REJECT 表示拒绝交点，INTERSECT 表示相交。

有几行代码可能需要解释。第 4 行计算 a，它是矩阵 M 的行列式。随后进行测试，以避开接近零的行列式。只要适当调整 ε 的值，这一算法就非常稳健。在浮点精度和“正常”条件下，ε = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_08_98c233e2dae60d.png) 就能很好地工作。第 9 行将 u 的值与三角形的一条边（u = 0）进行比较。

网上提供了该算法的 C 代码，包括剔除与不剔除背面的两个版本 [1231]。C 代码有两个分支：一个高效地剔除所有背向三角形，另一个对双面三角形进行相交测试。所有计算都会推迟到确有需要时才执行。例如，只有确定 u 的值处于允许范围内后，才会计算 v 的值（这一点在伪代码中也可以看到）。

单面求交例程会排除所有行列式为负值的三角形。这样一来，就能将例程中唯一一次除法运算推迟到确认相交之后才执行。


## 22.9 射线与多边形求交

来源：原书第 966—970 页（PDF 第 987—991 页）。本节从 22.9 标题开始，到 22.10 标题之前结束，包含 22.9.1 的全部内容。

尽管三角形是最常见的渲染图元，但拥有一个计算射线与多边形交点的例程仍然很有用。一个具有 n 个顶点的多边形由有序顶点列表 {![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_08896f686baac3.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_5607dc78935cb8.png), …, ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_5011da0bd967ae.png)} 定义：当 0 ≤ i < n − 1 时，顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_3ad693c18421f2.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_9847367b31f8d8.png) 构成一条边，再用从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_5011da0bd967ae.png) 到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_08896f686baac3.png) 的边将多边形闭合。多边形所在的平面记作 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_e803fc15d80aed.png)。

首先计算射线（式 22.1）与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_8639a1c672797a.png) 的交点，只需用射线表达式替换 x，就可以轻松完成。解如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_826b21329db91c.png)


如果分母满足 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_c0018f21b6199e.png)，其中 ε 是一个极小的数，那么便认为射线与多边形平面平行，不发生相交。在这项计算中，ε 取 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_05afafc20f4b2d.png) 或更小的值也可以，因为这里的目的是避免除法溢出。我们忽略射线位于多边形平面内的情况。

否则，计算射线与多边形平面的交点 p：p = o + td，其中 t 的值取自式 22.17。随后，将判断 p 是否位于多边形内部的问题从三维降为二维。具体做法是将所有顶点和 p 投影到 xy、xz 或 yz 平面之一，并选择使投影后多边形面积最大的那个平面。换言之，可以舍弃与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_8665a8659242ed.png) 对应的坐标分量，将另外两个分量保留下来作为二维坐标。例如，给定法线 (0.6, −0.692, 0.4)，y 分量的绝对值最大，因此忽略所有 y 坐标。选择绝对值最大的分量，是为了避免投影到可能产生退化、零面积三角形的平面上。注意，为了提高效率，可以预先计算一次这一分量信息，并将其存储在多边形中。投影过程中，多边形与交点的拓扑关系保持不变（前提是多边形确实是平面的；有关这一问题的更多内容见 16.2 节）。投影过程如图 22.15 所示。


![图 22.15 多边形与交点的正交投影](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_9_22.15.png)

**图 22.15** 将多边形的顶点和交点 p 正交投影到 xy 平面，使投影后多边形的面积最大。这是一个利用降维来简化计算的例子。

剩下的问题是：二维的射线／平面交点 p 是否包含在二维多边形中。这里，我们只介绍其中一种较为实用的算法——“穿越”测试。Haines [642] 以及 Schneider 和 Eberly [1574] 对二维点在多边形内的判定策略进行了广泛综述。更形式化的论述可参阅计算几何文献 [135, 1339, 1444]。Lagae 和 Dutré [955] 基于 Möller 和 Trumbore 的射线／三角形测试，给出了一种快速的射线／四边形求交方法。Walker [1830] 给出了一种对顶点数超过 10 的多边形进行快速测试的方法。Nishita 等人 [1284] 讨论了具有曲线边界的形状的点包含测试。

### 22.9.1 穿越测试

穿越测试基于拓扑学中的若尔当曲线定理。由该定理可知，如果从一个点出发、沿平面内任意方向发出的射线穿越了奇数条多边形边，那么这个点就在多边形内部。若尔当曲线定理实际上仅适用于不自交的闭合曲线。对于自交的闭合曲线，这种射线测试会将一些视觉上位于多边形内部的区域判为外部，如图 22.16 所示。这种测试也称为奇偶性测试或偶奇测试。

穿越算法从点 p 的投影出发，沿 x 轴正方向发射一条射线（也可以选择任意方向；选择 x 方向只是因为代码实现更高效）。随后计算多边形各边与这条射线的穿越次数。正如若尔当曲线定理所证明的，奇数次穿越表明该点位于多边形内部。

也可以将测试点 p 视为位于原点，然后改为测试平移后的各条边与 x 轴正半轴的关系。图 22.17 展示了这种做法。如果一条多边形边的两个端点的 y 坐标同号，那么这条边就不可能穿越 x 轴。否则，它有可能穿越 x 轴，此时再检查 x 坐标。如果两个 x 坐标都为正，就将穿越次数加一，因为测试射线必然与这条边相交。如果两个 x 坐标异号，就必须计算该边与 x 轴交点的 x 坐标；若它为正，就将穿越次数加一。


![图 22.16 自交多边形的奇偶性判定](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_9_22.16.png)

**图 22.16** 一个既自交又非凸的一般多边形，但它围成的区域并非全都被视为内部（只有棕色区域属于内部）。顶点用大的黑点标出。图中显示了三个待测试的点及其测试射线。按照若尔当曲线定理，如果与多边形各边的穿越次数为奇数，那么该点位于内部。因此，最上方和最下方的点位于内部（分别穿越一次和三次）。中间的两个点各穿越两条边，因此被视为位于多边形外部。

> 译注：原图注写作“三个”测试点，但图中实际画有四个测试点及四条测试射线，原图注后文也分别讨论了最上方、最下方和中间两个点。此处保留原文并说明这一点数不一致。

在图 22.17 中，也可以将所有被围住的区域都归类为内部。这种变体测试求出的是绕数，即多边形的闭合曲线绕测试点旋转的圈数。有关论述见 Haines 的文章 [642]。

> 译注：此段原文引用的是图 22.17。结合前文对自交多边形的讨论及“所有被围住的区域”这一表述，它可能意指图 22.16；这里保留原图号，不作暗改。


![图 22.17 将测试点平移到原点的穿越测试](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_9_22.17.png)

**图 22.17** 多边形已平移 −p（p 是需要测试是否被多边形包含的点），因此，与 x 轴正半轴的穿越次数决定了 p 是否位于多边形内部。边 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_43270b023aa067.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_f92bdb6d943d5e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_2cf0929b20f3a8.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_a5b9178eff7d9e.png) 不穿越 x 轴。必须计算边 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_392783cc92c86c.png) 与 x 轴的交点，但由于交点的 x 分量为负，它不会产生一次穿越。边 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_795bc454ddf698.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_d2e8d2e9ec7efe.png) 各使穿越次数增加一次，因为每条边的两个顶点的 x 分量都为正，而 y 分量一正一负。最后，边 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_b06908a29a287a.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_443381eb8d7cbc.png) 共享一个满足 y = 0 且 x > 0 的顶点，它们合起来使穿越次数增加一次。将 x 轴上的顶点视为位于射线上方，就会把 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_b06908a29a287a.png) 归类为穿越射线，而将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_443381eb8d7cbc.png) 归类为位于射线上方。

当测试射线与某个顶点相交时，可能会出现问题，因为这时可能检测到两次穿越。解决方法是将该顶点视为位于射线上方无穷小的距离处；实际实现时，把 y ≥ 0 的顶点也解释为位于 x 轴（射线）上方即可。这样就不会再与任何顶点相交，代码也会变得更简单、更快 [640]。

下面给出一种高效穿越测试的伪代码。它受 Joseph Samosky [1537] 和 Mark Haigh-Hutchinson 工作的启发，代码可在网上获得 [642]。该算法比较二维测试点 t 与多边形 P，后者的顶点为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_08896f686baac3.png) 到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_5011da0bd967ae.png)。

```text
bool PointInPolygon(t, P)
返回 ({TRUE, FALSE});
 1: bool inside = FALSE
 2: e₀ = vₙ₋₁
 3: bool y₀ = (e₀.y ≥ t.y)
 4: for i = 0 to n − 1
 5:     e₁ = vᵢ
 6:     bool y₁ = (e₁.y ≥ t.y)
 7:     if (y₀ ≠ y₁)
 8:         if (((e₁.y − t.y)(e₀.x − e₁.x) ≥ (e₁.x − t.x)(e₀.y − e₁.y)) == y₁)
 9:             inside = ¬inside
10:     y₀ = y₁
11:     e₀ = e₁
12: return inside;
```

这里用 `.x` 和 `.y` 表示原伪代码中写在下标位置的坐标分量；相邻括号相乘，¬ 表示逻辑非。TRUE、FALSE 分别表示真、假，inside 记录点是否位于内部。

第 3 行检查多边形最后一个顶点的 y 值是否大于或等于测试点 t 的 y 值，并将结果存入布尔变量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_4aa89317c95466.png)。换言之，它判断我们将要测试的第一条边的第一个端点位于 x 轴上方还是下方。第 7 行测试端点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_43270b023aa067.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_09_392783cc92c86c.png) 是否位于经过测试点的 x 轴的不同侧。如果是，第 8 行就测试 x 轴截距是否为正。实际上，这里的做法还要更快一点：为避免计算截距通常需要的除法，这里执行了一项消除符号影响的运算。第 9 行通过反转 inside 的值来记录发生了一次穿越。第 10—12 行转向下一个顶点。

> 译注：上一句保留了原文的“第 10—12 行”。对照伪代码，更新当前边端点和符号并继续遍历的是第 10—11 行；第 12 行实际返回最终结果。

在伪代码中，我们没有在第 7 行之后再进行一次测试，以检查两个端点的 x 坐标相对于测试点是否都更大或都更小。尽管介绍算法时，我们使用了对这类边快速接受或拒绝的办法，但基于上述伪代码的程序在省去这一测试后，往往运行得更快。一个主要影响因素是被测试多边形的顶点数：顶点越多，先检查 x 坐标的差值就可能越高效。

穿越测试的优点是相对快速、稳健，而且不要求为多边形保存额外信息或进行预处理。其缺点是，除了指明一个点位于多边形内部还是外部以外，它不提供其他信息。其他方法，例如 22.8.1 节中的射线／三角形测试，还能计算重心坐标，用来对测试点的附加信息进行插值 [642]。注意，重心坐标可以扩展到具有三个以上顶点的凸多边形和凹多边形 [474, 773]。Jiménez 等人 [826] 给出了一种基于重心坐标的优化算法，目标是将多边形各边上的所有点都纳入内部判定，其性能可与穿越测试相媲美。

更一般的问题是，判断一个点是否位于由线段和 Bézier 曲线构成的闭合轮廓内部；同样可以采用类似方式，通过统计射线穿越次数来完成。Lengyel [1028] 给出了用于这一过程的稳健算法，并将它用于渲染文本的像素着色器中。

> 译注：原书此处在“Bézier curves”后混入了“curves!Bézier”字样，页面实图亦如此，疑为未清理的索引排印标记；正文按其正常语义译为“Bézier 曲线”，并在此记录原文异常。


## 22.10 平面与盒体相交测试

本节来源：《Real-Time Rendering, 4th Edition》书页 970—972（PDF 物理页 991—993），范围从 22.10 标题开始，到 22.11 标题之前，包含 22.10.1 与 22.10.2。

把一个点代入平面方程 π：**n**·**x** + d = 0，就可以知道该点到平面的距离。所得结果的绝对值就是到平面的距离。因此，平面与球体的测试很简单：把球心代入平面方程，检查结果的绝对值是否小于或等于球体半径。

判断盒体是否与平面相交的一种方法，是把盒体的所有顶点代入平面方程。如果得到的结果既有正值又有负值（或者有零值），就说明顶点位于平面的两侧（或者位于平面上），因而检测到了相交。还有一些更巧妙、更快的测试方法，接下来的两小节将分别介绍针对 AABB 和 OBB 的方法。

这两种方法的共同思想是：八个角点中，只需把两个代入平面方程。对于朝向任意的盒体，无论它是否与平面相交，沿平面法线方向测量时，总有两个互为对角的角点之间距离最大。每个盒体的角点构成四条对角线。把各条对角线的方向与平面法线做点积，最大值就确定了包含这两个最远点的那条对角线。只测试这两个角点，就能完成整个盒体与平面的测试。

### 22.10.1 AABB

假设有一个轴对齐包围盒（AABB）B，由中心点 **c** 和各分量为正的半对角线向量 **h** 定义。注意，利用 B 的最小角点与最大角点，可以很容易地求出 **c** 和 **h**，即

![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_97f8dee0d99db1.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_75b14a0e57e5c8.png)。

现在，要测试 B 与平面 **n**·**x** + d = 0 的关系。有一种快得令人惊讶的方法可以完成这一测试。其思想是计算盒体投影到平面法线 **n** 上的“延伸范围”，这里记作 e。理论上，可以把盒体八条不同的半对角线全部投影到法线上，再选出最长的一条。不过，实际实现时可以用下面的式子快速求得：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_a1d3366ab4b062.png)


![图22.18 平面与轴对齐盒体的相交测试](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_10_22.18.png)

**图 22.18** 对一个中心为 **c**、正半对角线为 **h** 的轴对齐盒体进行与平面 π 的测试。其思想是计算盒体中心到平面的有符号距离 s，并将其与盒体的“延伸范围” e 比较。向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_2fac333bd9bc7d.png) 是二维盒体中各条可能的对角线，本例中 **h** 等于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_63d06f29cad89d.png)。还要注意，有符号距离 s 为负，而且其绝对值大于 e，这表明盒体位于平面内侧（s + e < 0）。图中的 positive half-space 表示“正半空间”。

为什么这等价于求八条不同半对角线投影的最大值？这八条半对角线是以下组合：

![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_9e784174649b57.png)，

我们要对全部八个 i 计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_2fac333bd9bc7d.png)·**n**。当点积的每一项都为正时，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_2fac333bd9bc7d.png)·**n** 取得最大值。对于 x 项，当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_a7b7f94e7daf67.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_2d89e8b5b640d5.png) 同号时就会如此；不过，既然已知 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_bbc58392ab7d2f.png) 为正，就可以把这一项的最大值计算为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_bbc58392ab7d2f.png)|![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_a7b7f94e7daf67.png)|。对 y 和 z 也这样做，就得到了公式（22.18）。

接下来，计算中心点 **c** 到平面的有符号距离 s，计算式为 s = **c**·**n** + d。图 22.18 展示了 s 和 e。假设平面的“外侧”是正半空间，那么只需测试 s − e > 0；若成立，就说明盒体完全位于平面外侧。类似地，s + e < 0 表示盒体完全位于内侧。否则，盒体与平面相交。这项技术基于 Ville Miettinen 的思想及其巧妙实现。伪代码如下：

```text
PlaneAABBIntersect(B, π)
returns({OUTSIDE, INSIDE, INTERSECTING});
1 : c = (bᵐᵃˣ + bᵐⁱⁿ)/2
2 : h = (bᵐᵃˣ − bᵐⁱⁿ)/2
3 : e = hₓ|nₓ| + hᵧ|nᵧ| + h[z]|n[z]|
4 : s = c·n + d
5 : if(s − e > 0) return (OUTSIDE);
9 : if(s + e < 0) return (INSIDE);
10 : return (INTERSECTING);
```

其中 OUTSIDE、INSIDE、INTERSECTING 分别表示外侧、内侧和相交。伪代码中的 c、h、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_9fe3d56980215f.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_3321bf6399a38b.png) 和 n 均为向量；h[z] 与 n[z] 表示相应向量的 z 分量。

### 22.10.2 OBB

对有向包围盒（OBB）进行平面测试，与上一小节的 AABB／平面测试只有细微差别。只需改变盒体“延伸范围”的计算方式，改为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_fd4c06b5347ef2.png)


回顾一下，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_655c99d03b8bc3.png) 是 OBB 的坐标系轴（参见 22.2 节对 OBB 的定义），而 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_41bbbcde6070bf.png) 是盒体沿这些轴的长度。

> 译注：本节将 **n**·**x** + d 直接称为有符号距离，隐含了 **n** 为单位法线的条件；若法线未归一化，实际距离还须除以其模长。对盒体分类而言，只要 s 和 e 使用同一法线尺度，文中的比较仍成立。
>
> 译注：原文推导中使用的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_4e98e9284b6322.png)，按上下文表示第 i 条半对角线的 x 分量；前文将这条向量记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_10_2fac333bd9bc7d.png)，原文此处符号不统一，译文保留原式。原书伪代码行号从 5 跳到 9，亦原样保留。
>
> 译注：OBB 末段原文写作沿轴的“长度”；依公式（22.19）中从中心向两侧延伸的含义，这些量应按半边长理解，不能代入完整边长。


## 22.11 三角形／三角形相交

来源：原书书页 972—974（PDF 物理页 993—995）；从 22.11 节标题开始，至 22.12 节标题之前。

由于图形硬件将三角形作为最重要的绘制图元，并针对它进行了优化，因此也对这种数据进行碰撞检测，是很自然的做法。所以，碰撞检测算法的最深层通常包含一个用于判定两个三角形是否相交的例程。给定两个三角形 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) = △![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_166fb0f2b267c9.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) = △![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_d1e0e0565819e6.png)（它们分别位于平面 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_da60106891227f.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_eb593e06ecb03d.png) 上），我们希望确定它们是否相交。

从总体思路来看，通常先检查 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) 是否与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_eb593e06ecb03d.png) 相交，以及 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 是否与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_da60106891227f.png) 相交 [1232]。只要其中任意一项测试失败，两个三角形就不可能相交。假设两个三角形不共面，我们知道平面 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_da60106891227f.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_eb593e06ecb03d.png) 的交集是一条直线 L，如图 22.19 所示。从图中可以得出：如果两个三角形相交，那么它们各自与 L 相交所得的区间也必须重叠；否则，两个三角形就不相交。这一思路有不同的实现方式，接下来介绍 Guigue 和 Devillers [622] 的方法。


![图 22.19 三角形与其所在平面，以及交线上的区间](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_11_22.19.png)

**图 22.19**　三角形及其所在平面。两幅图中的相交区间都以红色标出。左：沿直线 L 的区间重叠，两个三角形也相交。右：不存在相交；两个区间不重叠。


![图 22.20 螺旋方向示意](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_11_22.20.png)

**图 22.20**　沿 **d − c** 方向观察螺旋向量 **b − a** 的示意图。

在这一实现中，会大量使用由四个三维向量 **a**、**b**、**c** 和 **d** 构成的 4 × 4 行列式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_05d2e866f8277a.png)


从几何上看，公式（22.20）有一种直观解释。叉积 (**b − a**) × (**c − a**) 可以看作是在计算三角形 △**abc** 的法线。将这条法线与从 **a** 指向 **d** 的向量做点积，就得到一个数值；如果 **d** 位于三角形 △**abc** 所在平面的正半空间中，这个数值就是正的。另一种解释是：行列式的符号告诉我们，沿 **b − a** 方向的螺旋是否按 **d − c** 所指示的方向转动。图 22.20 对此进行了说明。

首先测试 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) 是否与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_eb593e06ecb03d.png) 相交，并反过来测试 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 是否与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_da60106891227f.png) 相交。这可以通过计算公式（22.20）所定义的特殊行列式 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_98a7412aadb892.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_be5ab28b79fa3b.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_6cbf0e04c89660.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_2cf71c68d27bf4.png)]、[![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_98a7412aadb892.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_be5ab28b79fa3b.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_6cbf0e04c89660.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_3b128402d9cea8.png)] 和 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_98a7412aadb892.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_be5ab28b79fa3b.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_6cbf0e04c89660.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_7de97f50afa9fa.png)] 来完成。第一项测试相当于先计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 的法线，再测试点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_2cf71c68d27bf4.png) 位于哪个半空间。如果这些行列式的符号相同且都不为零，两个三角形就不可能相交，测试随即结束。如果它们全为零，两个三角形就共面，需要执行一项单独的测试来处理这种情况。否则，继续用同一类型的测试来检查 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 是否与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_da60106891227f.png) 相交。

此时，需要计算 L 上的两个区间 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_b5be89bc7e1811.png) = [i, j] 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_003d004c7738c4.png) = [k, l]，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_b5be89bc7e1811.png) 由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) 得到，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_003d004c7738c4.png) 由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 得到。为此，要重新排列每个三角形的顶点，使第一个顶点独自位于另一个三角形所在平面的一侧。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_b5be89bc7e1811.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_003d004c7738c4.png) 重叠，两个三角形就相交，而这种情况只会在 k ≤ j 且 i ≤ l 时发生。为了实现 k ≤ j 的测试，可以利用行列式的符号测试（公式（22.20）），并注意到 j 来自 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_d2accb1191aafd.png)，k 来自 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_221c2c0ddf70ab.png)。借助行列式计算的“螺旋测试”解释，可以得出：如果 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_2cf71c68d27bf4.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_3b128402d9cea8.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_98a7412aadb892.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_be5ab28b79fa3b.png)] ≤ 0，就有 k ≤ j。因此，最终测试变为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_c7def4cf483d4a.png)


整个测试从六次行列式测试开始，而前三次的前几个参数相同，因此可以共享许多计算。原则上，可以利用许多较小的 2 × 2 子行列式来计算该行列式；如果这些子行列式出现在多个 4 × 4 行列式中，就可以共享它们的计算。网上提供了这一测试的代码 [622]，还可以扩充代码，以计算实际的相交线段。

如果两个三角形共面，就将它们投影到使三角形面积最大的轴对齐平面上（第 22.9 节）。随后执行一个简单的二维三角形重叠测试。首先，测试 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) 的所有闭合边（即包含端点）是否与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 的闭合边相交。只要发现任意相交，两个三角形就相交。否则，必须测试 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) 是否完全包含于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 中，或者反过来。这可以通过从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_47714cb824af95.png) 中取一个顶点，对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_11_85e3550cabd641.png) 执行点在三角形内测试（第 22.8 节），并反过来再测试一次来完成。

注意，可以利用分离轴测试（参见第 947 页）推导出三角形／三角形重叠测试。这里介绍的是 Guigue 和 Devillers [622] 的测试方法，它比使用 SAT 更快。还存在其他用于三角形／三角形相交测试的算法 [713, 1619, 1787]。体系结构和编译器的差异，以及预期命中率的变化，意味着我们无法推荐一种在所有情况下性能都最好的算法。还应注意，与任何几何测试一样，这里也可能出现精度问题。Robbins 和 Whitesides [1501] 使用 Shewchuk [1624] 的精确算术来避免这一问题。


## 22.12 三角形与盒的相交

来源：《Real-Time Rendering, Fourth Edition》，书页 974—975（PDF 物理页 995—996）；并核对 PDF 第 997 页以确认节边界及插图归属。

本节介绍一种判定三角形是否与轴对齐盒相交的算法。这种测试可用于体素化和碰撞检测。

Green 和 Hatch [581] 提出了一种算法，可以判定任意多边形是否与盒重叠。Akenine-Möller [21] 基于分离轴测试（第 947 页）开发了一种更快的方法，这就是我们在这里介绍的方法。三角形与球的测试也可以利用这一测试来完成，详情参见 Ericson 的文章 [440]。

我们重点考虑由中心 **c** 和半边长向量 **h** 定义的轴对齐包围盒（AABB），与三角形 Δ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_2c30c7680293f1.png) 之间的测试。为简化测试，首先平移盒和三角形，使盒的中心位于原点，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_6174fdaf578643.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_8997bdde7ab1ec.png) − **c**，i ∈ {0, 1, 2}。图 22.21 展示了这一平移以及所用记号。若要针对有向盒进行测试，则先利用盒变换的逆变换来旋转三角形顶点，再使用这里的测试。根据分离轴测试（SAT），我们测试以下 13 条轴：

1. ［3 次测试］![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_69f2e5c0d09163.png) = (1, 0, 0)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_6ccdf61733397a.png) = (0, 1, 0)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_1fa9060ca16e84.png) = (0, 0, 1)，即 AABB 的面法线。换句话说，测试该 AABB 与包围三角形的最小 AABB 是否重叠。

2. ［1 次测试］**n**，即 Δ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_2c30c7680293f1.png) 的法线。我们使用一种快速的平面与 AABB 重叠测试（第 22.10.1 节），它只测试盒的一条体对角线的两个端点；这条体对角线的方向与三角形法线最为接近。


![图 22.21 三角形与盒重叠测试使用的记号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_12_22.21.png)

**图 22.21**　三角形与盒重叠测试使用的记号。左图显示盒和三角形的初始位置；右图中，盒和三角形都经过了平移，使盒的中心与原点重合。

3. ［9 次测试］![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_14bf6300c72bc7.png)，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_d6d338908d05ee.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_ff23f9a0248d04.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_0b099269428b7d.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_6b9fa8172c6d2c.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c17c2507a83155.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_ff23f9a0248d04.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_a2929b3bcc776e.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_0b099269428b7d.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c17c2507a83155.png)，也就是边向量。这些测试的形式相似，我们只展示 i = 0 且 j = 0 这一情形的推导（见下文）。

一旦找到分离轴，算法就终止并返回“不重叠”。如果所有测试都通过，也就是说不存在分离轴，那么三角形就与盒重叠。

下面推导步骤 3 中九次测试之一，即 i = 0 且 j = 0 的情形。这意味着 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_f86cd722ca8190.png)。因此，现在需要将三角形顶点投影到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_d04e26e861417d.png)（以下称为 **a**）上：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_bcc4b8a3c51cf8.png)


通常，我们需要求 min(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c11de44d831b47.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_19aa1dc33bda1e.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_3a55303a18d915.png)) 和 max(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c11de44d831b47.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_19aa1dc33bda1e.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_3a55303a18d915.png))，但幸运的是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c11de44d831b47.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_19aa1dc33bda1e.png)，这简化了计算。现在只需求 min(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c11de44d831b47.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_3a55303a18d915.png)) 和 max(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_c11de44d831b47.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_3a55303a18d915.png))，速度会快得多，因为条件语句在现代 CPU 上的开销很大。

将三角形投影到 **a** 上之后，还需要把盒也投影到 **a** 上。盒在 **a** 上投影的“半径”r 按下式计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_a7e4502e8605d6.png)


其中，最后一步成立是因为对于这条特定的轴，有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_7c6c64eb80e1ee.png) = 0。于是，这条轴上的测试变为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_12_8b00b8d8de70a1.png)


代码可在网上获取 [21]。


## 22.13 包围体／包围体相交

来源：《Real-Time Rendering, Fourth Edition》，书页 976—981（PDF 物理页 997—1002）。本节包含 22.13.1—22.13.5 的全部内容；图 22.22 位于本节标题之前，但属于本节，亦收录于此。正文止于 22.14 标题之前。

包围体的目的是提供更简单的相交测试，并更高效地排除不相交的情况。例如，要测试两辆汽车是否碰撞，可以先求出它们的包围体（BV），再测试这些包围体是否重叠。如果不重叠，就能保证汽车不会碰撞（这里假定这是最常见的情况）。这样就不必把一辆汽车的每个图元与另一辆汽车的每个图元逐一进行测试，从而节省计算。

测试两个包围体是否重叠是一项基本操作。下面各小节将介绍 AABB、k-DOP 和 OBB 的重叠测试方法。为图元构造包围体的算法见第 22.3 节。

之所以使用比球体和 AABB 更复杂的包围体，是因为更复杂的包围体通常能够更紧密地贴合物体。图 22.22 展示了这一点。当然，也可以使用其他包围体。例如，有时会使用圆柱体和椭球体作为物体的包围体；也可以布置多个球体来包围单个物体 [782, 1582]。


![图 22.22：球体、AABB、OBB 与 k-DOP](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_13_22.22.png)

图 22.22．图中展示了同一物体的球体包围体（左）、AABB（中左）、OBB（中右）和 k-DOP（右）；OBB 和 k-DOP 内部的空余空间明显少于另外两种。

对于胶囊体和圆角矩形体（lozenge）包围体，计算最小距离是一项相对快速的操作。因此，它们常用于间距容差验证应用；这类应用希望验证两个或更多物体之间的距离至少达到某个指定值。Eberly [404] 和 Larsen 等人 [979] 为这些类型的包围体推导了公式和高效算法。

### 22.13.1 球体／球体相交

对于球体，相交测试简单而快速：计算两个球心之间的距离；如果该距离大于两个球体的半径之和，就判定不相交，否则它们相交。实现这一算法时，最好比较这两个量的平方，因为所需的只是比较结果。这样就能避免计算平方根这一开销较大的操作。Ericson [435] 给出了同时测试四组独立球体对的 SSE 代码。

### 22.13.2 球体／包围盒相交

Arvo [70] 最早提出了测试球体与 AABB 是否相交的算法，该算法出人意料地简单。其思路是找出 AABB 上距离球心 **c** 最近的点。对 AABB 的三个轴分别进行一次一维测试：将球心在某个轴上的坐标与 AABB 在该轴上的边界比较。如果坐标位于边界之外，就计算球心到包围盒沿该轴的距离（一次减法），并将其平方。对三个轴完成这些操作后，将距离平方之和与球体半径的平方 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_730f558bb648d5.png) 比较。如果前者小于半径平方，则最近点位于球体内部，包围盒与球体重叠。正如 Arvo 所示，可以修改此算法，使其处理空心包围盒、空心球体以及轴对齐椭球体。

Larsson 等人 [982] 提出了该算法的一些变体，其中包括速度快得多的 SSE 向量化版本。他们的关键想法是尽早使用简单的排除测试：可以逐轴进行，也可以一开始就全部进行。排除测试检查球心到包围盒沿某个轴的距离是否大于半径。如果是，就可以提前结束测试，因为球体此时不可能与包围盒重叠。当重叠的可能性较低时，这种提前排除的方法明显更快。下面给出他们的 QRI（quick rejections intertwined，交错快速排除）版本。第 4 行和第 7 行是提前退出测试，必要时可以去掉。

以下伪代码保留原算法的函数名、变量名和行号；OVERLAP 表示“重叠”，DISJOINT 表示“不相交”。


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_45545598f6fa63.png)


为了实现快速的向量化版本（使用 SSE），Larsson 等人建议消除大部分分支。思路是用下面的表达式同时计算第 3 行与第 6 行：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_a939a27f05361f.png)


通常，接下来会按 d = d + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_8732ed5c8c7938.png) 更新 d。不过，使用 SSE 可以针对 x、y、z 并行计算式（22.25）。完整测试的伪代码如下。


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_836bfdabf8051d.png)


注意，第 1 行和第 2 行可以用并行的 SSE max 函数实现。尽管这一测试没有提前退出，它仍然比其他技术更快。这是因为它消除了分支，并使用了并行计算。另一种 SSE 实现方式是对物体对进行向量化。Ericson [435] 给出了同时将四个球体与四个 AABB 进行比较的 SIMD 代码。

对于球体／OBB 相交测试，先将球心变换到 OBB 的空间中。也就是说，以 OBB 的归一化轴为基，变换球心。此时球心已经相对于 OBB 的各轴表示，因此可以把 OBB 当成 AABB，然后使用球体／AABB 算法测试相交。

Larsson [983] 给出了一种高效的椭球体／OBB 相交测试方法。首先缩放两个物体，使椭球体变成球体，OBB 变成平行六面体。可以利用球体／平板相交测试快速接受或排除。最后，只测试球体与那些朝向它的平行四边形是否相交。

### 22.13.3 AABB／AABB 相交

顾名思义，AABB 是各面与主坐标轴方向对齐的包围盒。因此，只需两个点就足以描述这样的包围体。这里采用第 22.2 节给出的 AABB 定义。

由于简单，AABB 既常用于碰撞检测算法，也常作为场景图中节点的包围体。两个 AABB（A 和 B）的相交测试非常简单，概括如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_95d9071351a436.png)


第 1 行和第 2 行遍历全部三个标准坐标轴方向 x、y、z。Ericson [435] 提供了同时测试四组独立 AABB 对的 SSE 代码。

### 22.13.4 k-DOP／k-DOP 相交

一个 k-DOP 与另一个 k-DOP 的相交测试只包含 k/2 次区间重叠测试。Klosowski 等人 [910] 表明，当 k 取适中数值时，两个 k-DOP 的重叠测试比两个 OBB 的测试快一个数量级。书页 946 的图 22.4 展示了一个简单的二维 k-DOP。注意，AABB 是 6-DOP 的一种特例，其法线方向为主坐标轴的正向和负向。OBB 也是 6-DOP 的一种形式，但是只有两个 OBB 具有相同的轴时，才能采用这一快速测试。

下面的相交测试简单而且极快，虽然不精确，但具有保守性。若要测试两个 k-DOP，即 A 和 B（以上标 A 和 B 区分），是否相交，则测试所有相互平行的平板对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_f4c04243d603f1.png) 是否重叠；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_861dbb53b5d3ac.png) 是一维区间重叠测试，很容易求解。这正是第 22.5 节经验法则所建议的降维方法的一个例子：三维平板测试在这里被简化为一维区间重叠测试。

只要出现 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_a5f27d78afc4f9.png) = ∅（即空集），就说明这两个包围体不相交，测试随即终止。否则，继续进行平板重叠测试。当且仅当对所有 1 ≤ i ≤ k/2 均有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_a5f27d78afc4f9.png) ≠ ∅ 时，才认为两个包围体重叠。按照分离轴测试（第 22.2 节），还需要分别从两个 k-DOP 中各取一条边，测试与其叉积平行的轴。但是，这些测试的开销通常大于它们带来的性能收益，所以常被省略。因此，如果下面的测试返回 k-DOP 重叠，它们实际上仍有可能不相交。k-DOP／k-DOP 重叠测试的伪代码如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_53c9aef26fd175.png)


注意，每个 k-DOP 实例只需存储 k 个标量值（法线 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_a85255f83fc76e.png) 是固定的，因此所有 k-DOP 共享一份法线数据）。如果两个 k-DOP 分别平移 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_58d9d71a623d9b.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_e2534940aa6708.png)，测试只会稍微复杂一点。将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_58d9d71a623d9b.png) 投影到法线 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_a85255f83fc76e.png) 上，例如 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_b8de1aef919e0a.png)（注意，这与任何具体 k-DOP 无关，因此每个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_58d9d71a623d9b.png) 或 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_e2534940aa6708.png) 只需计算一次），然后在 if 语句中，分别将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_e343241df02888.png) 加到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_3563fcd0a35b05.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_ab70df643d56ac.png) 上。对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_e2534940aa6708.png) 也做同样的处理。换句话说，平移会改变 k-DOP 沿各法线方向的距离。

Laine 和 Karras [965] 提出了一种称为顶点映射（apex point map）的 k-DOP 扩展。其思路是将一组平面法线映射到 k-DOP 上的各个点，使存储的每个点表示沿对应方向最远的位置。这个点与该方向共同确定一个平面，使模型完全包含在该平面的某个半空间内；也就是说，该点位于模型 k-DOP 的最外端。在测试期间，针对给定方向检索到的顶点可以用于更精确地测试 k-DOP 之间的相交、改善视锥体剔除，以及在旋转之后求得更紧密的 AABB 等。

### 22.13.5 OBB／OBB 相交

本小节简要介绍一种快速测试两个 OBB（A 和 B）是否相交的方法 [436, 576, 577]。该算法采用分离轴测试，比此前使用最近特征或线性规划的方法快大约一个数量级。OBB 的定义见第 22.2 节。

测试在由 A 的中心和各轴构成的坐标系中进行。这意味着原点为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_8262b31c2dc839.png)，该坐标系的主轴为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_0e0b588d7f60ef.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_794af91080b37f.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_e4ad80ba147af0.png)。另外，假定 B 相对于 A 的位置由平移 **t** 和旋转矩阵 **R** 给定。

根据分离轴测试，只需找到一条能够将 A 和 B 分开的轴，就能确定它们不相交（不重叠）。需要测试十五条轴：三条来自 A 的面，三条来自 B 的面，还有 3 · 3 = 9 条来自 A 与 B 的边的组合。图 22.23 用二维情形说明了这一过程。


![图 22.23：OBB 的分离轴测试](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_13_22.23.png)

图 22.23．可以使用分离轴测试来判断两个 OBB 是否重叠。这里展示的是二维情形。四条分离轴与两个 OBB 的面正交，每个包围盒对应两条轴。随后将两个 OBB 投影到这些轴上。如果在所有轴上，两者的投影都重叠，那么 OBB 就重叠；否则不重叠。因此，只要找到一条将投影分开的轴，就足以确定 OBB 不重叠。在本例中，左下方的轴是唯一一条能够将投影分开的轴。（图据 Ericson [436] 绘制。）图内“no overlap”意为“不重叠”。

由于矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_cbb254fd63f640.png) 的正交归一性，与 A 的各面正交的候选分离轴就是轴 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_b4f752291d7e72.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_6585b58e3fe886.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_40dd78f553b8f6.png)。B 也同样如此。其余九条候选轴各由 A、B 的一条边构成，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_33a73e29f76f7b.png)。好在网上已有这部分的优化代码 [1574]。

> 译注：原文在球体／AABB 的文字说明中使用“平方和小于半径平方”来描述重叠，而伪代码只在 d > ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_730f558bb648d5.png) 时返回不相交，因此将 d = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_13_730f558bb648d5.png) 的相切情况归入重叠。此处保留原文的两种表述，不擅自改写其边界条件。


## 22.14 视锥体相交

来源：《Real-Time Rendering, Fourth Edition》，书页 981—987（PDF 物理页 1002—1008）。本节从 22.14 标题开始，至 22.15 标题之前结束，包含全部下级小节及跨页续文。

如第 19.4 节所述，要快速渲染复杂场景，层次化视锥体剔除至关重要。在遍历包围体层次结构以进行剔除的过程中，调用的少数几种操作之一就是视锥体与包围体之间的相交测试。因此，这些操作对于快速执行极为关键。理想情况下，它们应当确定包围体（BV）是完全位于视锥体内部（包含）、完全位于外部（排除），还是与视锥体相交。

先回顾一下：视锥体是一个被近、远两个相互平行的平面截断的棱锥，由此成为有限的体积。实际上，它变成了一个多面体。图 22.24 展示了这一点，并标出了六个平面的名称：近、远、左、右、上和下。视锥体的体积界定了场景中应当可见、因而应当被渲染的部分（对于棱锥形视锥体，采用透视方式渲染）。

用于层次结构（例如场景图）内部节点以及包围几何体的最常见包围体，是球体、轴对齐包围盒（AABB）和有向包围盒（OBB）。因此，这里将讨论并推导视锥体/球体以及视锥体/AABB/OBB 测试。

为了理解为什么需要外部、内部、相交这三种返回结果，我们来看遍历包围体层次结构时会发生什么。如果发现某个 BV 完全位于视锥体外部，就不再继续遍历该 BV 的子树，其任何几何体都不会被渲染。另一方面，如果 BV 完全位于内部，该子树就不必再计算任何视锥体/BV 测试，所有可渲染的叶节点都会被绘制。对于部分可见的 BV，也就是与视锥体相交的 BV，则递归地对其子树进行视锥体测试。如果该 BV 对应一个叶节点，就必须渲染这个叶节点。


![图22.24 由无限棱锥截取视锥体](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_14_22.24.png)

图 22.24：左图是一个无限延伸的棱锥，用相互平行的近、远平面截取它，就构成了视锥体。图中也标出了其余平面的名称；摄像机位于棱锥的顶点。

完整的测试称为排除/包含/相交测试。有时，人们可能认为第三种状态——相交——的计算代价过高。这时，BV 被归类为“可能在内部”。我们把这种简化算法称为排除/包含测试。如果无法成功排除某个 BV，就有两种选择。一种是把“可能在内部”状态当作包含，即渲染 BV 内的所有内容。这通常效率不高，因为不会再进行进一步的剔除。另一种选择是依次对该子树中的每个节点进行排除测试。这种测试也常常没有收益，因为子树中的很大一部分确实可能位于视锥体内部。由于这两种选择都不特别理想，尝试快速区分相交与包含通常是值得的，即使测试并不完美。

必须认识到，对场景图进行剔除时，快速分类测试不必精确，只需保守即可。在区分排除与包含时，唯一的要求是：测试即使出错，也必须偏向包含。也就是说，实际上应当排除的对象，可以被错误地包含进来。这样的错误只会多花一些时间。反过来，应当包含的对象绝不能被快速测试归类为排除，否则就会产生渲染错误。至于包含与相交之间的区分，两种误分类通常都是允许的。如果把完全包含的 BV 归类为相交，就会浪费时间对其子树进行相交测试。如果把相交的 BV 视为完全位于内部，就会因渲染全部对象而浪费时间，其中一些对象原本可以被剔除。

在介绍视锥体与球体、AABB 或 OBB 之间的测试之前，我们先描述视锥体与一般对象之间的一种相交测试方法。图 22.25 展示了这种测试。其思路是把 BV/视锥体测试转化为点/体积测试。首先，选取一个相对于 BV 位置固定的点。然后，让 BV 沿着视锥体外侧移动，在不重叠的前提下尽量靠近视锥体。在移动过程中，跟踪这个相对于 BV 固定的点，其轨迹会形成一个新体积（图 22.25 中以粗边界表示的多边形）。由于 BV 已经尽可能靠近视锥体移动，因此，如果该点在 BV 原始位置所对应的位置落在轨迹形成的体积内，BV 就与视锥体相交，或者位于视锥体内部。因此，我们不再测试 BV 与视锥体的相交，而是测试这个相对于 BV 固定的点是否位于由该点轨迹形成的新体积内。同样，也可以让 BV 沿着视锥体内侧移动，并尽可能靠近视锥体边界。这会描出一个新的、更小的视锥体，其各平面与原视锥体平行 [83]。如果相对于对象固定的点位于这个新体积内，BV 就完全位于视锥体内部。后续小节将利用这种技术推导测试方法。注意，新体积的构造与实际 BV 的位置无关，只取决于该点相对于 BV 的位置以及 BV 的形状。这意味着，可以用相同的这些体积测试处于任意位置的 BV。


![图22.25 将视锥体与一般包围体的相交转化为点与体积测试](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_14_22.25.png)

图 22.25：左上图显示了视锥体（蓝色）和一般包围体（绿色），并选取了一个相对于对象位置固定的点 p。让对象沿视锥体的外侧（右上）和内侧（左下）移动，并尽量靠近视锥体，同时跟踪点 p，就可以把视锥体/BV 测试改写为点 p 与外、内两个体积的测试，如右下图所示。如果点 p 位于橙色体积之外，则 BV 位于视锥体之外。如果 p 位于橙色区域内，BV 就与视锥体相交；如果 p 位于紫色区域内，BV 就完全位于视锥体内部。

只需在各个子节点处保存其父 BV 的相交状态，就是一种有用的优化。如果已知父节点完全位于视锥体内部，那么其所有后代都不必再进行视锥体测试。第 19.4 节讨论的平面掩码和时间连贯性技术，也能显著改善对包围体层次结构的测试，不过在 SIMD 实现中，它们的作用较小 [529]。

首先，我们推导视锥体的平面方程，因为这些测试需要用到它们。接着介绍视锥体/球体相交，再解释视锥体/盒体相交。

### 22.14.1 视锥体平面提取

进行视锥体剔除，需要视锥体六个不同侧面的平面方程。这里介绍一种巧妙而快速的推导方法。假设观察矩阵为 **V**，投影矩阵为 **P**，则复合变换为 **M** = **PV**。点 **s**（其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_19a6940bbf6aba.png) = 1）按 **t** = **Ms** 变换为 **t**。此时，由于透视投影等原因，可能有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_1dbdd5e91b168a.png) ≠ 1。因此，将 **t** 的所有分量除以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_1dbdd5e91b168a.png)，得到满足 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_e9dfa681b49f6e.png) = 1 的点 **u**。对于位于视锥体内部的点，满足 −1 ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_b9eca245191763.png) ≤ 1，其中 i 取 x、y、z；也就是说，点 **u** 位于一个单位立方体内。这适用于 OpenGL 类型的投影矩阵（第 4.7 节）。DirectX 的情况相同，唯一区别是 0 ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_042aa5f04d58a0.png) ≤ 1。视锥体的平面可以直接由复合变换矩阵的各行推导出来。

先来看单位立方体左平面右侧的体积，该体积满足 −1 ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_b1b488b1be96cc.png)。展开如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_3aa8675aa54d61.png)


在推导中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_f3e6f29afdfb50.png) 表示 **M** 的第 i 行。最后一步 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_31052e1c193c56.png) 实际上表示视锥体左平面的一个（半）平面方程。这是因为单位立方体中的左平面已被变换回世界坐标。此外，注意 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_19a6940bbf6aba.png) = 1，这使该方程成为一个平面方程。为了使平面的法线指向视锥体外部，必须将方程取反，因为原方程描述的是单位立方体内部。这样就得到视锥体左平面的方程 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_efefd8b6af0cd9.png)。这里改用 (x, y, z, 1)，以采用 ax + by + cz + d = 0 这种形式的平面方程。归纳起来，所有平面为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_8cdf34d3c33eff.png)


网上提供了在 OpenGL 和 DirectX 中完成这项操作的代码 [600]。

> 译注：式（22.27）按原书保留，其中近平面的表达式对应前文 OpenGL 的深度范围。若采用前文 DirectX 的 0 ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_042aa5f04d58a0.png) ≤ 1，近平面应由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_14_5e3757ace7a09c.png) 提取。另，后文通过将点代入平面方程获得有符号距离时，平面法线应为单位长度；提取后须相应归一化全部平面系数。

### 22.14.2 视锥体/球体相交

正交视图的视锥体是一个盒体，因此，这种情况下的重叠测试变成了球体/OBB 相交，可以使用第 22.13.2 节介绍的算法求解。要进一步测试球体是否完全位于盒体内部，首先检查球心是否位于盒体沿各坐标轴的边界之间，并且到边界的距离大于球体半径。如果在三个维度上都满足这一条件，球体就被完全包含。关于这一修改算法的高效实现及代码，参见 Arvo 的文章 [70]。

按照推导视锥体/BV 测试的方法，对于任意视锥体，我们选取球心作为要跟踪的点 p，如图 22.26 所示。让半径为 r 的球体沿视锥体内侧和外侧移动，并尽量靠近视锥体，那么点 p 的轨迹就给出了重新表述视锥体/球体测试所需的体积。实际体积见图 22.26 中间部分。与之前一样，如果 p 位于橙色体积之外，球体就在视锥体之外。如果 p 位于紫色区域内，球体就完全位于视锥体内部。如果该点位于橙色区域内，球体就与视锥体的侧面平面相交。通过这种方式，可以进行精确测试。不过，为了提高效率，我们使用图 22.26 右侧所示的近似。这里把橙色体积向外扩展，以避免处理圆角所需的更复杂计算。注意，外部体积由视锥体各平面沿其法线方向向外移动 r 个距离单位构成；内部体积则可以通过将各平面沿法线方向向内移动 r 个距离单位来构造。


![图22.26 视锥体与球体的精确测试及保守近似](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_14_22.26.png)

图 22.26：左图显示了一个视锥体和一个球体。精确的视锥体/球体测试，可以表述为对点 p 与中图橙色、紫色体积的测试。右图是对中间体积的一种合理近似。如果球心位于圆角之外，却位于全部外侧平面之内，那么即使球体实际上位于视锥体外部，也会被错误地归类为相交。

假设视锥体的平面方程使正半空间位于视锥体之外。那么，实际实现时将遍历视锥体的六个平面，针对每个平面计算球心到该平面的有符号距离。这通过把球心代入平面方程来完成。如果距离大于半径 r，则球体位于视锥体之外。如果到所有六个平面的距离都小于 −r，则球体位于视锥体内部；否则，球体与视锥体相交。更准确地说，我们把结果报告为球体与视锥体相交，但球心可能位于图 22.26 所示圆角之外的某个尖角区域。这意味着球体实际上位于视锥体之外，但为了保证保守正确性，我们将其报告为相交。

为提高测试精度，可以增加额外的平面，用来测试球体是否在外部。不过，对于快速剔除场景图节点这一用途，偶尔出现的误命中只会引起不必要的测试，并不会导致算法失败；而增加这些测试会使总体耗时更多。第 20.3 节介绍了另一种更准确、但仍不精确的方法，当这些尖角区域的影响显著时，该方法很有用。

在高效着色技术中，视锥体常常高度不对称；书页 895 的图 20.7 描述了一种针对这种情况的特殊方法。Assarsson 和 Möller [83] 提供的方法把视锥体划分为八个卦限，并确定对象中心位于哪个卦限，从而在每次测试中省去三个平面。

### 22.14.3 视锥体/盒体相交

如果视图采用正交投影，即视锥体呈盒状，就可以使用 OBB/OBB 相交测试（第 22.13.5 节）进行精确测试。对于一般的视锥体/盒体相交测试，有两种常用方法。一种简单方法是使用视锥体的观察矩阵和投影矩阵，将盒体的全部八个角点变换到视锥体坐标系。对沿各轴范围均为 [−1, 1] 的规范视体进行裁剪测试（第 4.7.1 节）。如果所有点都位于某个边界之外，就拒绝该盒体；如果所有点都位于内部，则盒体被完全包含 [529]。由于这种方法模拟了裁剪，因此可以用于任何由一组点界定的对象，例如线段、三角形或 k-DOP。这种方法的一个优点是不需要提取视锥体平面。它简单且自成一体，因此适合在计算着色器中高效使用 [1883, 1884]。

在 CPU 上效率高得多的方法，是使用第 22.10 节介绍的平面/盒体相交测试。与视锥体/球体测试一样，将 OBB 或 AABB 与视锥体的六个平面分别比较。在平面/盒体测试中，我们不必计算全部八个角点到平面的有符号距离，而只检查由平面法线确定的至多两个角点。如果最近的角点位于平面外侧，那么整个盒体都在外部，可以提前结束测试。如果对于每一个平面，最远角点都位于内侧，那么盒体就被包含在视锥体内部。注意，由于近平面和远平面相互平行，它们可以共用点积距离计算。这第二种方法唯一额外的代价，就是必须先推导视锥体的平面；如果要测试几个盒体，这点开销微不足道。

与视锥体/球体算法一样，该测试也会把实际上完全位于外部的盒体归类为相交。图 22.27 显示了这类错误。Quílez [1452] 指出，对于固定大小的地形网格或其他大型对象，这种情况可能更加频繁。他的解决方案是：当报告相交时，再用构成包围盒的各个平面测试视锥体的角点。如果全部点都位于盒体某个平面之外，那么视锥体与盒体就不相交。这项额外测试相当于分离轴测试的第二部分，即测试垂直于第二个对象各面的轴。话虽如此，额外测试的代价可能超过其收益。Eng [425] 在自己的 GIS 渲染器中发现，这项优化每帧花费 2 ms 的 CPU 时间，却只节省了少数几个绘制调用。


![图22.27 盒体测试的保守误分类区域](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_14_22.27.png)

图 22.27：黑色粗线是视锥体的平面。使用所介绍的算法测试盒体（左）与视锥体时，可能把实际位于外部的盒体错误地归类为相交。对于图中的情况，当盒体中心位于红色区域内时，就会发生这种情况。

> 译注：原图注写作“盒体（左）”，但原图中绿色盒体实际位于右侧；这里保留原文表述并指出该处图注疑误。

Wihlidal [1884] 则在视锥体剔除中朝另一个方向改进：只使用视锥体的四个侧平面，不进行近平面和远平面的剔除测试。他指出，这两个平面对电子游戏帮助不大。近平面大多是冗余的，因为侧平面已经裁掉了近平面所裁空间的几乎全部；远平面通常又被设置成能够看到场景中的所有对象。

另一种方法是使用分离轴测试（见第 22.13 节）来推导相交例程。多位作者使用分离轴测试给出了两个凸多面体的一般解法 [595, 1574]。这样，单个经过优化的测试就可以用于线段、三角形、AABB、OBB、k-DOP、视锥体和凸多面体的任意组合。


## 22.15 直线与直线相交

来源：《Real-Time Rendering》第 4 版，书页 987—990（PDF 物理页 1008—1011）。本节从 22.15 标题开始，到 22.16 标题之前结束。

本节推导并研究二维和三维的直线与直线相交测试。我们考察直线、射线和线段之间的相交，并介绍既快速又优雅的方法。

### 22.15.1 二维

#### 第一种方法

从理论的角度来看，第一种计算两条二维直线交点的方法确实十分优美。考虑两条直线，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4dd577f6e9a964.png)(s) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_0b6b03e8fbe6d9.png) + s![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png)(t) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_25431144bcc690.png) + t![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_72023cc3795bc8.png)。由于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4ad5c030aeadbd.png)（第 1.2.1 节中的垂直点积 [735]），![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4dd577f6e9a964.png)(s) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png)(t) 之间的求交计算变得优雅而简单。注意，本小节中的所有向量都是二维向量：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4a0f1275d8595c.png)


如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_0e3463617d7763.png)，则两条直线平行，不会产生交点。对于无限长的直线，s 和 t 的所有取值都有效；但对于方向已归一化的线段，假设长度分别为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f2b89bdf47651.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_800b741f6605bf.png)（起点为 s = 0 和 t = 0，终点为 s = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f2b89bdf47651.png) 和 t = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_800b741f6605bf.png)），则当且仅当 0 ≤ s ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f2b89bdf47651.png) 且 0 ≤ t ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_800b741f6605bf.png) 时，才存在有效的交点。或者，如果令 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_0b6b03e8fbe6d9.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_49788f15959bcd.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png)（即线段从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png) 开始，在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_49788f15959bcd.png) 结束），并对起点和终点分别为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f9266439beb7c.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_8cb898bbde464d.png) 的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png) 作同样处理，那么当且仅当 0 ≤ s ≤ 1 且 0 ≤ t ≤ 1 时，才存在有效的交点。对于具有起点的射线，有效范围为 s ≥ 0 且 t ≥ 0。将 s 代入 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4dd577f6e9a964.png)，或将 t 代入 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png)，都可以得到交点。

> 译注：原文将上述分母为零概括为“平行且不相交”。严格来说，还需单独处理两条直线重合的退化情形；此时不能用式（22.28）的除法求得唯一交点。

#### 第二种方法

Antonio [61] 描述了另一种判断两条线段（即长度有限）是否相交的方法：增加比较和提前排除，并避免前述公式中代价昂贵的计算（除法）。因此，这种方法更快。这里继续使用前面的记号，即第一条线段从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png) 到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_49788f15959bcd.png)，第二条线段从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f9266439beb7c.png) 到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_8cb898bbde464d.png)。这意味着 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4dd577f6e9a964.png)(s) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png) + s(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_49788f15959bcd.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png))，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png)(t) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f9266439beb7c.png) + t(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_8cb898bbde464d.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f9266439beb7c.png))。利用式（22.28）的结果，可以求出 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4dd577f6e9a964.png)(s) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png)(t) 的解：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4e3c5f8d9532a3.png)


在式（22.29）中，**a** = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_8cb898bbde464d.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f9266439beb7c.png)，**b** = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_49788f15959bcd.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png)，**c** = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_6daa83021b6136.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f9266439beb7c.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_b58b5a667c9ec1.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_acfe974bcb4074.png)，以及 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e2e09de1ab85b4.png)。因子 s 的化简步骤利用了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4f7324ade6466e.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_3a4dd70871ef42.png)。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_9748240cb1463b.png)，则两条直线共线。Antonio [61] 注意到，s 和 t 的分母相同，而且由于不需要显式求出 s 和 t，可以省略除法运算。定义 s = d/f，t = e/f。使用以下代码测试是否满足 0 ≤ s ≤ 1：

```text
1: if (f > 0)
2:     if (d < 0 or d > f) return NO_INTERSECTION;
3: else
4:     if (d > 0 or d < f) return NO_INTERSECTION;
```

> 译注：原文这里写作“共线”（collinear）；但该条件只能保证方向平行，不能单独保证两条直线共线。f = 0 的平行、共线及线段重叠情形需另行处理。代码中的 NO_INTERSECTION 表示“不相交”。

通过这项测试后，便可保证 0 ≤ s ≤ 1。接着对 t = e/f 做同样的测试（将代码中的 d 替换为 e）。如果例程在这次测试后仍未返回，那么线段确实相交，因为此时 t 值也有效。

这个例程的整数版本源代码可在网上获得 [61]，并且很容易改为使用浮点数。

### 22.15.2 三维

假设我们希望在三维中计算两条直线的交点（直线用射线来定义，见式（22.1））。仍将这两条直线记作 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_4dd577f6e9a964.png)(s) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_0b6b03e8fbe6d9.png) + s![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_d0d99428b64576.png)(t) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_25431144bcc690.png) + t![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_72023cc3795bc8.png)，t 的取值不受限制。在这里，垂直点积在三维中的对应运算是叉积，因为 **a** × **a** = **0**，因此三维版本的推导与二维版本非常相似。两条直线的求交推导如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_a0a601ec25fdce.png)


第 3 步通过从等式两边减去 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_0b6b03e8fbe6d9.png)（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_25431144bcc690.png)），再与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_72023cc3795bc8.png)（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png)）作叉积得到；第 4 步则通过与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png) × ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_72023cc3795bc8.png)（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_72023cc3795bc8.png) × ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png)）作点积得到。最后，把右边改写为行列式（并改变下面那个等式中的一些符号），再除以位于 s（t）右侧的项，就得到第 5 步，也就是所求的解。

Goldman [548] 指出，如果分母 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_18df69a29356a5.png) 等于 0，那么两条直线平行。他还指出，如果两条直线异面（即不在同一平面内），那么参数 s 和 t 表示两条直线上彼此距离最近的点。

如果要将这两条直线视为长度分别为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f2b89bdf47651.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_800b741f6605bf.png) 的线段（假设方向向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_e1887638149f61.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_72023cc3795bc8.png) 已归一化），就检查 0 ≤ s ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_2f2b89bdf47651.png) 和 0 ≤ t ≤ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_15_800b741f6605bf.png) 是否同时成立。若不成立，则排除相交。

Rhodes [1490] 对两条直线或线段的求交问题给出了深入的解决方案。他给出了能处理特殊情况的稳健解法，并讨论了优化，还提供了源代码。

> 译注：三维中的异面直线没有交点。式（22.30）仍能给出最近点参数，因此参数落在线段范围内本身并不足以证明相交；还需确认两个对应点重合（数值实现中采用合适的容差）。原文推导中的等价符号在按最近点解释时也应结合这一限制理解。


## 22.16 三个平面的交点

来源：《Real-Time Rendering, 4th Edition》书页 990（PDF 第 1011 页），范围从本节标题至“延伸阅读与资源”之前。

给定三个平面，每个平面分别由一个归一化的法向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_a85255f83fc76e.png) 和平面上的任意一点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_557ff8e9d48118.png) 描述，其中 i = 1、2、3。这些平面的唯一交点 **p** 由式（22.31）给出 [549]。注意，分母是三个平面法向量构成的行列式；如果两个或更多平面平行，则该行列式为零：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_f3f784823447c5.png)


这个公式可用于计算由一组平面构成的包围体（BV）的顶点。k-DOP 就是一个例子，它由 k 个平面方程构成。只要将适当的平面代入式（22.31），就能计算出该凸多面体的顶点。

如果像通常那样，平面以隐式形式给出，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_a59d4fe1157fa9.png)，那么为了使用上述公式，我们需要先求出点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_557ff8e9d48118.png)。可以选择平面上的任意一点。我们计算距离原点最近的点，因为这种计算的代价很低。给定一条从原点出发、沿平面法向量方向的射线，求它与平面的交点，即可得到距离原点最近的点：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_aca60d33d51073.png)


这个结果并不令人意外，因为平面方程中的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_22_16_3e7a00b9b5a749.png) 仅仅表示从原点到平面沿垂直方向的负距离（要使这一说法成立，法向量必须为单位长度）。


## 第22章 延伸阅读与资源

来源：《Real-Time Rendering, 4th Edition》书页 990—991（PDF 第 1011—1012 页）；另记录 PDF 第 1013 页的出版方标识。本部分在原书中不编号，与 22.16 节正文分开保存。

Ericson 的《实时碰撞检测》（Real-Time Collision Detection）[435] 和 Eberly 的《3D 游戏引擎设计》（3D Game Engine Design）[404] 涵盖了多种物体与物体的相交测试及层次结构遍历方法，还涉及许多其他内容，并附有源代码。Schneider 和 Eberly 的《计算机图形学几何工具》（Geometric Tools for Computer Graphics）[1574] 提供了许多用于二维和三维几何相交测试的实用算法。开放获取期刊《计算机图形技术期刊》（Journal of Computer Graphics Techniques）刊载相交测试方面的改进算法及代码。较早出版的《实用线性代数》（Practical Linear Algebra）[461] 是一个很好的资料来源，其中介绍了二维相交例程，以及许多对计算机图形学有用的其他几何操作。《图形学精粹》（Graphics Gems）系列 [72, 540, 695, 902, 1344] 收录了许多不同类型的相交例程，其代码可在网上获取。免费的 Maxima [1148] 软件适合用来变换方程和推导公式。本书网站中的 [相交测试资源页面](https://realtimerendering.com/intersections.html) 汇总了许多物体与物体相交测试的相关资源。

## 出版方标识

![出版方标识（原页无图号）](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_22_99_publisher.png)

PDF 第 1013 页没有正文，仅有出版方标识：Taylor & Francis（泰勒与弗朗西斯）；Taylor & Francis Group（泰勒与弗朗西斯集团）；[出版方网址](http://taylorandfrancis.com)。
