# Real-Time Rendering 第16章 多边形技术 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [16.1 三维数据的来源](<Real-Time_Rendering_4th_中文/第16章/16.01.md>)

- [16.2 曲面细分与三角剖分](<Real-Time_Rendering_4th_中文/第16章/16.02.md>)

- [16.3 整合](<Real-Time_Rendering_4th_中文/第16章/16.03.md>)

- [16.4 三角形扇、三角形带与网格](<Real-Time_Rendering_4th_中文/第16章/16.04.md>)

- [16.5 简化](<Real-Time_Rendering_4th_中文/第16章/16.05.md>)

- [16.6 压缩与精度](<Real-Time_Rendering_4th_中文/第16章/16.06.md>)


## 章首导言

来源：原书第681—682页（PDF第702—703页）。本文件包含章标题、题辞、章首导言及脚注，到16.1节标题前止。

> “三角形这样简单的图形，竟然蕴含着如此无穷无尽的奥妙，实在令人惊叹。”
>
> ——利奥波德·克雷勒（Leopold Crelle）

到目前为止，我们一直假定，要渲染的模型恰好采用我们所需的格式，而且细节量也恰到好处。现实中，我们很少有这样的好运。建模器和数据采集设备各有其特有的习性与局限，这会导致数据集中出现歧义和错误，进而反映在渲染结果中。我们常常需要在存储大小、渲染效率和结果质量之间作出权衡。本章将讨论多边形数据集中会遇到的各种问题，以及针对这些问题的一些修复方法和变通办法。随后，我们将介绍高效渲染和存储多边形模型的技术。

在交互式计算机图形学中，多边形表示的总体目标是视觉准确性和速度。“准确性”的含义取决于具体情境。例如，工程师希望以交互速率检查和修改机器零件，并要求物体上的每一处斜面和倒角在任何时刻都清晰可见。相比之下，在游戏中，只要帧率足够高，某一帧出现轻微的错误或不准确之处是可以接受的，因为这些问题可能不在注意力集中的区域，也可能在下一帧就消失。在交互式图形工作中，明确待解决问题的边界非常重要，因为这些边界决定了可以采用哪些技术。

本章涉及的领域包括曲面细分、整合、优化、简化和压缩。输入的多边形可能具有许多不同形式，通常必须将它们分割为更易处理的图元，例如三角形或四边形。这个过程称为三角剖分，或者更一般地称为曲面细分（tessellation）。注1 我们用“整合”一词来指代这样一个过程：将独立的多边形合并成网格结构，并推导出用于表面着色的新数据，例如法线。“优化”是指对网格中的多边形数据进行排序，以使其渲染得更快。“简化”是指去除网格中无关紧要的特征。“压缩”则关注如何尽可能减少描述网格的各种元素所需的存储空间。

三角剖分确保给定的网格描述能够正确显示。整合通过共享计算并减少内存占用，进一步改善数据的显示，而且通常还能提高速度。优化技术可以进一步加快速度。简化通过移除不必要的三角形，还可以带来更高的速度。压缩可用于进一步降低总体内存占用，继而通过降低内存和总线的带宽需求来提高速度。

**注1：** “Tessellation”中的l要写两个；这个词大概是计算机图形学中最常被拼错的词，“frustum”（视锥体）则紧随其后。


## 16.1 三维数据的来源

来源：原书第682—683页（PDF第703—704页）。范围从16.1节标题起，到16.2节标题前止。

创建或生成多边形模型有以下几种方式：

- 直接输入几何描述。
- 编写生成这类数据的程序。这称为程序化建模。
- 将其他形式的数据转换为表面或体积，例如，将蛋白质数据转换成一组球体和圆柱体。
- 使用建模程序构建或雕刻物体。
- 根据同一物体的一张或多张照片重建其表面，这称为摄影测量。
- 使用三维扫描仪、数字化仪或其他传感设备，在真实模型的不同位置进行采样。
- 生成等值面，用于表示某一空间体积内数值相同的位置，例如医学计算机轴向断层扫描（CAT）或磁共振成像（MRI）的数据，或在大气中测得的气压、温度样本。
- 组合使用上述技术。

在建模领域，建模器主要分为两类：基于实体的建模器和基于表面的建模器。基于实体的建模器通常用于计算机辅助设计（CAD）领域，往往侧重于提供与实际机械加工过程相对应的建模工具，例如切削、钻孔和刨削。在内部，它们具有一个计算引擎，能够严密地处理物体底层的拓扑边界。为了进行显示和分析，这类建模器配备了面片生成器（faceter）。面片生成器是一种软件，它将模型的内部表示转换为可供显示的三角形。例如，数据库中可能使用一个中心点和一个半径来表示球体，而面片生成器可以将其转换为任意数量的三角形或四边形，以表示该球体。有时候，最好的渲染加速方法恰恰是最简单的方法：使用面片生成器时降低所要求的视觉精度，就能通过生成更少的三角形来提高速度并节省存储空间。

在CAD工作中，一个重要的考虑因素是：所使用的面片生成器是否专为图形渲染而设计。例如，用于有限元法（FEM）的面片生成器，其目标是将表面划分为面积近乎相等的三角形。这类曲面细分结果非常适合进行简化，因为其中包含大量对图形显示没有用处的数据。同样，一些面片生成器生成的三角形集合非常适合通过3D打印制造现实物体，但它们缺少顶点法线，而且往往不适合快速图形显示。

Blender或Maya等建模器并不以一种内置的实体性概念为基础，而是通过表面来定义物体。与实体建模器一样，这些基于表面的系统可能使用内部表示和面片生成器来显示样条曲面或细分曲面等物体（第17章）。它们也可能允许直接操作表面，例如添加或删除三角形或顶点。这样，用户便可以手动减少模型的三角形数量。

还有其他类型的建模器，例如隐式曲面创建系统（包括具有“团块状”外观的元球系统）[67, 558]，它们使用混合、权重和场等概念。这些建模器通过生成由某个函数方程 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_01_0a05d480e57cac.png) 的解所定义的曲面，来创建有机形态。然后，使用移动立方体（marching cubes）等多边形化技术，生成用于显示的三角形集合（第17.3节）。

点云非常适合应用简化技术。这类数据通常按规则间隔采样，因此，许多样本对所形成表面的视觉感知影响微乎其微。研究人员已经花费数十年时间研究过滤缺陷数据以及从点云重建网格的技术[137]。有关这一领域的更多内容，请参见第13.9节。

对于由扫描数据生成的网格，可以执行各种清理操作或更高层次的操作。例如，分割技术会分析多边形模型，并尝试识别其中不同的组成部分[1612]。这样做有助于创建动画、应用纹理贴图、匹配形状以及进行其他操作。

还有许多其他方式可以生成用于表面表示的多边形数据。关键在于理解这些数据是如何创建的，以及创建它们的目的。很多时候，生成数据并不是专门为了高效的图形显示。此外，三维数据文件格式种类繁多，任意两种格式之间的转换往往不是无损操作。了解输入数据可能具有哪些局限、可能出现哪些问题，是本章的一个主要主题。


## 16.2 曲面细分与三角剖分

来源：《Real-Time Rendering》第4版，书页 683—690（PDF 物理页 704—711）。本节从 16.2 标题起，至 16.3 标题前止，包含 16.2.1、16.2.2 及图 16.1—16.9。

曲面细分是把一个表面分割成一组多边形的过程。这里我们着重讨论多边形表面的细分；曲面本身的细分将在第 17.6 节讨论。进行多边形细分可以有多种原因。最常见的原因是，所有图形 API 和硬件都针对三角形进行了优化。三角形几乎就像原子：任何表面都可以由它们构成并加以渲染。把复杂多边形转换成三角形的过程称为三角剖分。


![图16.1 各种曲面细分方式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.1.png)

**图 16.1** 各种类型的细分。最左边的多边形没有细分；接下来一个被划分为凸区域；再下一个进行了三角剖分；最右边的则进行了均匀网格划分。

对多边形进行细分时，可能有几个不同的目标。例如，即将使用的算法可能只能处理凸多边形。这种细分称为凸划分。也可能需要把表面细分为网格，以便使用全局光照技术，在每个顶点处存储阴影或相互反射的效果 [400]。图 16.1 展示了这些不同细分方式的例子。与图形显示无关的细分原因包括：要求任何三角形都不能大于某个给定面积，或者要求三角形各顶点处的角都大于某个最小角度。Delaunay 三角剖分要求，每个三角形的三个顶点所确定的圆都不包含其余任何顶点，从而使最小角最大化。虽然这类限制通常属于有限元分析等非图形应用的要求，但它们也可以改善表面的外观。细长三角形往往值得避免，因为在相距很远的顶点之间进行插值可能产生伪影。它们的光栅化效率也可能较低 [530]。

大多数细分算法在二维空间中工作。它们假设多边形的所有点都位于同一平面上。然而，有些模型创建系统可能生成严重翘曲、并不共面的多边形面片。这个问题的一种常见情形是，几乎沿边缘方向观察一个翘曲四边形；它可能形成所谓的沙漏形或蝴蝶结形四边形。见图 16.2。虽然对这个特定多边形只需创建一条对角边就能进行三角剖分，但更复杂的翘曲多边形并不那么容易处理。


![图16.2 翘曲四边形及两种三角剖分](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.2.png)

**图 16.2** 沿边缘方向观察翘曲四边形，会形成一个定义不明确的蝴蝶结形或沙漏形图形；图中还显示了两种可能的三角剖分。

如果可能出现翘曲多边形，一种快速的修正措施是，把顶点投影到一个垂直于多边形近似法线的平面上。通常通过计算多边形在三个相互正交的 xy、xz 和 yz 平面上的投影面积来求出这个平面的法线。也就是说，舍去 x 坐标后得到的多边形在 yz 平面上的面积，就是 x 分量的值；在 xz 平面上的面积给出 y 分量，在 xy 平面上的面积给出 z 分量。这种计算平均法线的方法称为 Newell 公式 [1505, 1738]。

投影到这个平面上的多边形仍可能存在自相交问题，即两条或更多条边互相交叉。此时就需要更精细、计算代价更高的方法。Zou 等人 [1978] 讨论了以最小化所得细分的表面积或二面角为目标的既有工作，并提出了将一组中的几个非平面多边形一起优化的算法。

Schneider 和 Eberly [1574]、Held [714]、O’Rourke [1339] 以及 de Berg 等人 [135] 分别概述了多种三角剖分方法。最基本的三角剖分算法会检查多边形上任意给定两点之间的每一条线段，看它是否与多边形的任意一条边相交或重叠。如果是，就不能用这条线段分割多边形，于是继续检查下一对可能的点。否则，就用这条线段把多边形分成两部分，再以同样的方法对这些新多边形进行三角剖分。这种方法极其缓慢，复杂度为 O(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_584fde46074e95.png))。


![图16.3 切耳法](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.3.png)

**图 16.3** 切耳法。图中多边形在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_37d45ef245b1bd.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_25a7dc3c5b57aa.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_30543a5ab7b772.png) 处有潜在的耳。右图移除了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_25a7dc3c5b57aa.png) 处的耳。随后重新检查相邻顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_239e8d38e2bb18.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_30543a5ab7b772.png)，看它们现在是否构成耳；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_30543a5ab7b772.png) 构成了耳。

更高效的方法是切耳法（ear clipping）；把它分成两个过程实施时，其复杂度为 O(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_02_09c317452584b9.png))。首先遍历一次多边形以找出耳：考察所有顶点索引为 i、(i + 1)、(i + 2) 的三角形（索引对 n 取模），检查连接 i 与 (i + 2) 的线段是否不与任何多边形边相交。如果不相交，则 (i + 1) 处的三角形构成一个耳。见图 16.3。依次从多边形中移除每个可用的耳，并重新检查顶点 i 和 (i + 2) 处的三角形，看它们现在是否成为耳。最终所有耳都被移除，多边形也就完成了三角剖分。其他更复杂的三角剖分方法可以达到 O(n log n)，其中一些在典型情况下实际上可以达到 O(n)。Schneider 和 Eberly [1574] 给出了切耳法以及其他更快三角剖分方法的伪代码。

与三角剖分相比，把多边形划分成凸区域在存储开销和后续计算代价两方面都可能更有效率。Schorn 和 Fisher [1576] 给出了稳健的凸性测试代码。凸多边形很容易表示为三角形扇或三角形带，第 16.4 节会对此展开讨论。有些凹多边形也可以作为三角形扇处理（这类多边形称为星形多边形），但检测它们需要更多工作 [1339, 1444]。Schneider 和 Eberly [1574] 给出了两种凸划分方法：一种快速但粗糙的方法，以及一种最优方法。


![图16.4 多轮廓转单轮廓](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.4.png)

**图 16.4** 将具有三个轮廓的多边形转换成单轮廓多边形。连接边以红色显示。多边形内部的蓝色箭头表示遍历顶点以形成单一环路的顺序。

多边形并不总是由单一轮廓组成。图 16.4 显示了一个由三个轮廓组成的多边形，也称为环路（loops）或轮廓线（contours）。通过在环路之间谨慎地生成连接边（也称锁孔边或桥接边），总能把这样的表示转换成单轮廓多边形。Eberly [403] 讨论了如何寻找用来定义这类边的互相可见顶点。这个转换过程也可以反向进行，以恢复各个独立环路。

编写稳健而通用的三角剖分器是一项艰巨工作。各种细微错误、病态情形和精度问题，会使编写万无一失的代码变得出乎意料地棘手。巧妙避开三角剖分问题的一种方法，是使用图形加速器本身直接渲染复杂多边形。把多边形作为三角形扇渲染到模板缓冲中。这样，应当填充的区域会被绘制奇数次，而凹陷和孔洞会被绘制偶数次。对模板缓冲使用反转模式，第一遍结束时就只有应填充区域被标记。见图 16.5。在第二遍中再次渲染三角形扇，使用模板缓冲只允许绘制应填充的区域。通过绘制每个环路形成的三角形，这种方法甚至可以渲染具有多个轮廓的多边形。主要缺点是，每个多边形都必须用两遍进行渲染，模板缓冲每帧都要清除，而且无法直接使用深度缓冲。这项技术可用于显示某些用户交互，例如显示即时绘制的复杂选择区域的内部。


![图16.5 模板缓冲奇偶填充](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.5.png)

**图 16.5** 通过光栅化进行三角剖分，利用奇偶性判断哪些区域可见。左侧多边形以顶点 0 为起点、由三个三角形组成的扇形绘制到模板缓冲中。第一个三角形 [0, 1, 2]（中左）填充其覆盖区域，其中包括多边形外部的空间。三角形 [0, 2, 3]（中右）填充其覆盖区域，使 A 和 B 区域的绘制次数变成偶数，从而将它们清空。三角形 [0, 3, 4]（右）填充多边形的其余部分。

### 16.2.1 着色问题

有时收到的数据是四边形网格，必须转换成三角形才能显示。极少数情况下，四边形是凹的，此时只有一种三角剖分方式。否则，可以选择两条对角线中的任意一条将它分割。花一点时间选择更好的对角线，有时能够显著改善视觉效果。

有几种不同的方法可以决定如何分割四边形。关键思想是尽量减小新边两端顶点之间的差异。对于顶点不带附加数据的平面四边形，通常最好选择最短的对角线。对于每个顶点存储一种颜色的简单烘焙全局光照结果，应选择两端颜色差异较小的对角线 [17]。见图 16.6。根据某种启发式规则连接差异最小的两个角，这个想法通常有助于尽量减少伪影。


![图16.6 对角线选择与着色差异](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.6.png)

**图 16.6** 左图作为四边形进行渲染；中图是连接右上角与左下角形成的两个三角形；右图显示使用另一条对角线时的结果。中图的视觉效果优于右图。

有时三角形无法恰当地表达设计者的意图。如果给一个扭曲四边形应用纹理，沿任何一条对角线分割都无法保留设计意图。不过，对未经三角剖分的四边形做简单的水平插值，也就是从左边到右边插值，同样行不通。图 16.7 展示了这个问题。问题产生的原因在于，应用到表面上的图像在显示时需要发生扭曲。一个三角形只有三个纹理坐标，因此它能确定一个仿射变换，却不能确定这种扭曲。三角形上的基本 (u, v) 纹理至多只能发生剪切，而不能发生扭曲。Woo 等人 [1901] 进一步讨论了这个问题。有几种可能的解决办法：

- 预先扭曲纹理，再使用新的纹理坐标重新应用这张新图像。
- 将表面细分成更精细的网格。这只能减轻问题。
- 使用投影纹理，在运行时即时扭曲纹理 [691, 1470]。这样会产生一个不理想的效果：纹理在表面上的间距不均匀。
- 使用双线性映射方案 [691]。通过为每个顶点增加数据即可实现。


![图16.7 扭曲四边形纹理映射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.7.png)

**图 16.7** 左上图显示设计者的意图：将字母“R”的正方形纹理贴图应用到一个变形四边形上。右侧两图显示两种三角剖分及其差异。下排将所有多边形旋转；未经三角剖分的四边形改变了外观。

虽然纹理畸变听起来像是一种病态情形，但只要应用的纹理数据与底层四边形的比例不匹配，它就会在某种程度上发生，也就是说，几乎任何曲面上都会出现。一个极端情形出现在一种常见图元上：圆锥。为圆锥应用纹理并将其面片化时，圆锥尖端处的三角形顶点具有不同的法线。这些顶点法线不被相邻三角形共享，因此会出现着色不连续 [647]。

### 16.2.2 边缘裂缝与 T 形顶点

第 17 章将详细讨论的曲面，通常会细分成网格以供渲染。这种细分沿着定义表面的样条曲线逐步采样，从而计算顶点位置和法线。使用简单的步进方法时，样条曲面相接之处可能出现问题。在共享边上，两个表面的点必须重合。由于模型本身的特点，有时它们会恰好重合；但往往如果不够谨慎，为一条样条曲线生成的点就会与相邻曲线生成的点不匹配。这种现象称为边缘裂缝，观察者能够从缝隙中看到表面的另一侧，因此可能产生令人不适的视觉伪影。即使观察者无法透过裂缝看到后方，着色插值方式的差异通常也会使接缝可见。

修复这些裂缝的过程称为边缘缝合。目标是确保沿共享（曲）边的所有顶点都由两个样条曲面共享，从而不出现裂缝。见图 16.8。第 17.6.2 节讨论如何使用自适应曲面细分避免样条曲面出现裂缝。


![图16.8 边缘裂缝与缝合](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.8.png)

**图 16.8** 左图显示两个表面相接之处的裂缝。中图通过匹配边缘上的点修复了裂缝。右图显示修正后的网格。

连接平面表面时会遇到一个相关问题：T 形顶点。只要两个模型的边相接，却没有共享这些边上的所有顶点，就可能出现这类问题。虽然理论上这些边应该完美相接，但如果渲染器表示屏幕顶点位置的精度不够，就可能出现裂缝。现代图形硬件使用亚像素寻址 [985] 来帮助避免这个问题。

更明显、且并非由精度引起的问题，是可能出现的着色伪影 [114]。图 16.9 展示了这个问题；可以通过找出这样的边，并确保与毗邻面共享公共顶点来修复。另一个问题是，使用简单的三角形扇算法存在生成退化（零面积）三角形的风险。例如，在图中，假设把右上方的四边形 **abcd** 三角剖分为三角形 **abc** 和 **acd**。三角形 **abc** 是退化三角形，因此点 **b** 是一个 T 形顶点。Lengyel [1023] 讨论了如何寻找这样的顶点，并提供了正确地对凸多边形重新进行三角剖分的代码。Cignoni 等人 [267] 描述了一种在已知 T 形顶点位置时，避免创建退化（零面积）三角形的方法。他们的算法复杂度为 O(n)，并保证最多生成一个三角形带和一个三角形扇。


![图16.9 T形顶点造成的着色不连续](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_2_16.9.png)

**图 16.9** 上排展示了一个表面的底层网格及其着色不连续。顶点 **b** 是一个 T 形顶点，因为它属于左侧的三角形，却不是三角形 **acd** 的组成部分。一种解决办法是把这个 T 形顶点加入该三角形，创建三角形 **abd** 和 **bcd**（未示出）。细长三角形更容易引起其他着色问题，因此重新进行三角剖分通常是更好的办法，下排显示了这种做法。


## 16.3 整合

来源：*Real-Time Rendering, Fourth Edition*，书页 690—696（PDF 物理页 711—717）。本节从 16.3 标题开始，到 16.4 标题之前结束，包含全部下级小节；图内英文标签保留，图注译为中文。

模型经过所需的曲面细分算法处理之后，我们得到的是一组表示该模型的多边形。有几种操作可能有助于显示这些数据。最简单的一种，是检查多边形本身是否构造正确，即是否至少具有三个位置互不相同且不共线的顶点。例如，如果一个三角形中的两个顶点重合，那么它就没有面积，可以将其丢弃。注意，本节确实是在讨论多边形，而不仅仅是三角形。根据你的目标，保留每个多边形，而不是立即将其转化为三角形来显示，可能会更高效。三角剖分会产生更多边，进而增加后续操作的工作量。

经常对多边形执行的一种处理是**合并**（merging），即找出各个面之间共享的顶点。另一种操作称为**定向**（orientation），使构成一个表面的所有多边形都朝向相同方向。对背面剔除、折痕边检测，以及正确的碰撞检测与响应等多种算法来说，网格定向都很重要。与定向相关的还有**顶点法线生成**（vertex normal generation），它使表面看起来平滑。我们将这些类型的技术统称为**整合算法**（consolidation algorithms）。

### 16.3.1 合并

有些数据以彼此不相连的多边形形式提供，通常称为**多边形汤**（polygon soup）或**三角形汤**（triangle soup）。分开存储多边形会浪费内存，逐个显示这些独立多边形也极其低效。出于这些以及其他原因，通常会将各个多边形合并成一个**多边形网格**（polygon mesh）。最简单的网格由一个顶点列表和一组轮廓组成。每个顶点包含一个位置，以及其他可选数据，例如着色法线、纹理坐标、切向量和颜色。每个多边形轮廓都有一个整数索引列表。每个索引都是从 0 到 n − 1 的一个数，其中 n 为顶点数量，因此索引指向列表中的一个顶点。这样，每个顶点只需存储一次，就可以由任意多个多边形共享。**三角形网格**（triangle mesh）是只包含三角形的多边形网格。第 16.4.5 节深入讨论网格的存储方案。

给定一组彼此不相连的多边形，可以用多种方式进行合并。一种方法是使用散列 [542, 1135]。将顶点计数器初始化为零。对于每个多边形，依次尝试将其每个顶点加入散列表，散列依据是顶点的各个值。如果表中尚不存在该顶点，就将其连同顶点计数器的当前值一起存入表中，然后递增计数器；同时，将该顶点存入最终的顶点列表。反之，如果找到匹配的顶点，则取出其已存储的索引。保存多边形时，使用指向这些顶点的索引。处理完所有多边形之后，顶点列表和索引列表便构建完成。

输入的模型数据有时会出现这样的情况：不同多边形的顶点位置极为接近，但并不完全相同。合并这类顶点的过程称为**焊接**（welding）。通过排序，并对位置采用更宽松的相等判定函数，可以高效地完成顶点焊接 [1135]。

### 16.3.2 定向

模型数据中一个与质量相关的问题是面的朝向。有些模型数据已经正确定向，其表面法线以显式或隐式方式指向正确方向。例如，在 CAD 工作中，标准约定是：从正面观察多边形时，沿其轮廓排列的顶点按逆时针方向行进。这称为**绕序方向**（winding direction），三角形遵循**右手定则**。想象右手四指按逆时针顺序沿着多边形的顶点弯曲，拇指所指的方向就是多边形法线的方向。这种定向与所用的观察空间或世界坐标系采用左手系还是右手系无关，因为它完全取决于从三角形正面观察时，顶点在世界中的排列顺序。不过，如果对已定向的网格应用反射矩阵，每个三角形的法线相对于其绕序方向都会反转。

对于一个合理的模型，下面是一种为多边形网格定向的方法：

1. 为所有多边形建立边—面结构。
2. 对边进行排序或散列，找出相匹配的边。
3. 找出彼此接触的多边形组。
4. 对每个组，根据需要翻转面，使朝向保持一致。

第一步是创建一组**半边**（half-edge）对象。半边就是多边形的一条边，并带有一个指向其所属面（多边形）的指针。由于一条边通常由两个多边形共享，因此这种数据结构称为半边。创建每条半边时，按照排序顺序存储两个顶点，使第一个顶点排在第二个顶点之前。如果一个顶点的 x 坐标值较小，那么它在排序顺序中就位于另一个顶点之前。如果 x 坐标相等，就使用 y 值；如果 y 值也相等，就使用 z 值。例如，顶点 (−3, 5, 2) 排在顶点 (−3, 6, −8) 之前：两者的 −3 相同，但 5 < 6。

目标是找出哪些边完全相同。因为每条边在存储时都保证第一个顶点小于第二个顶点，所以比较边只需比较两条边各自的第一个顶点，以及各自的第二个顶点。不必交换顺序，例如不必将一条边的第一个顶点与另一条边的第二个顶点比较。可以使用散列表查找匹配的边 [19, 542]。如果此前已合并所有顶点，使半边使用相同的顶点索引，那么可以将每条半边放入与其第一个顶点索引关联的临时列表中，借此进行匹配。一个顶点平均连接 6 条边，因此分组之后，边的匹配速度极快 [1487]。

边匹配完成后，相邻多边形之间的连接关系便已知，从而形成一个**邻接图**（adjacency graph）。对于三角形网格，可以为每个三角形保存一个列表，列出其最多三个相邻三角形面，以此表示邻接图。任何没有两个相邻多边形的边都是边界边。通过边连接起来的一组多边形构成一个连续组。例如，一个茶壶模型包含两个组：壶身与壶盖。

下一步是使网格的朝向一致，例如我们通常希望所有多边形都具有逆时针轮廓。对于每个连续的多边形组，任意选择一个起始多边形。检查它的每个相邻多边形，并判断朝向是否一致。如果两个多边形沿公共边的遍历方向相同，就必须翻转相邻多边形。参见图 16.10。递归检查这些相邻多边形的相邻多边形，直到连续组中的所有多边形都恰好检查一次。


![图 16.10：根据共享边的遍历顺序统一多边形朝向](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_3_16.10.png)

**图 16.10** 选取起始多边形 S，并检查其相邻多边形。由于 S 与 B 的共享边上各顶点的遍历顺序相同（从 x 到 y），因此需要反转 B 的轮廓，使其遵循右手定则。

虽然此时所有面的朝向已经一致，但它们可能全都朝向内部。在大多数情况下，我们希望它们朝外。快速判断是否应翻转所有面的一种方法，是计算该组的有符号体积并检查其正负号。如果体积为负，就反转所有轮廓环和法线。计算体积时，对每个三角形计算用于求有符号体积的标量三重积，然后求和。有关体积计算，请参阅本书网站 realtimerendering.com 上的在线线性代数附录。

这种方法对实体对象很有效，但并非万无一失。例如，如果对象是一个构成房间的盒子，用户希望它的法线朝内、指向摄像机。如果对象不是实体，而只是一个表面描述，那么自动确定每个表面的朝向就可能变得棘手。比如，同一网格中的两个立方体沿一条边接触，那么该边就会由四个多边形共享，使定向更加困难。莫比乌斯带这样的单侧对象永远无法完全定向，因为它没有内外之分。即使对于性质良好的表面网格，也可能难以确定哪一侧应该朝外。Takayama 等人 [1736] 讨论了此前的工作，并提出了他们自己的解决方案：从每个小面发射随机射线，确定哪一种朝向从外部更容易被看见。

### 16.3.3 实体性

非正式地说，如果一个网格已经定向，并且从外部可见的所有多边形都具有相同朝向，那么这个网格就构成一个实体。换句话说，网格只有一侧可见。这样的多边形网格称为**封闭的**（closed）或**水密的**（watertight）。

知道对象是实体，就意味着可以使用背面剔除来提高显示效率，详见第 19.2 节。对于投射阴影体的对象（第 7.3 节）以及其他若干算法，实体性也是关键性质。例如，3D 打印机要求待打印的网格是实体。

最简单的实体性检测，是检查网格中每条多边形边是否都恰好由两个多边形共享。这个检测对大多数数据集已经足够。这样的表面通常被宽泛地称为**流形**（manifold），具体而言是**二维流形**（two-manifold）。严格说来，流形表面是没有任何拓扑不一致的表面，例如不能出现三个或更多多边形共享同一条边，或者两个或更多角彼此接触的情形。构成实体的连续表面是一个无边界流形。

### 16.3.4 法线平滑与折痕边

有些多边形网格构成曲面，但多边形顶点没有法向量，因此无法在渲染时营造曲面的视觉效果。参见图 16.11。


![图 16.11：没有逐顶点法线与具有逐顶点法线的对象](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_3_16.11.png)

**图 16.11** 左侧对象没有逐顶点法线，右侧对象则有。

许多模型格式不提供表面边的信息。各种边的类型参见第 15.2 节。这些边的重要性体现在多个方面。它们可以突出模型中由一组多边形构成的区域，也可以帮助实现非真实感渲染。由于这些边提供了重要的视觉线索，渐进网格算法（第 16.5 节）通常会优先保留它们，避免将其简化掉。

通常，可以从已经定向的网格中相当成功地推导出合理的折痕边与顶点法线。一旦朝向一致并且邻接图已经建立，就可以用**平滑技术**生成顶点法线。模型格式可能会通过为多边形网格指定平滑组来提供帮助。平滑组的取值用于明确指定组中的哪些多边形应归在一起，构成一个曲面。不同平滑组之间的边被视为锐边。

平滑多边形网格的另一种方法是指定一个**折痕角**（crease angle）。将这个值与**二面角**（dihedral angle），即两个多边形所在平面的法线之间的夹角进行比较。折痕角的取值通常为 20° 到 50°。如果两个相邻多边形之间的二面角小于指定的折痕角，就认为这两个多边形属于同一个平滑组。这种技术有时称为**边保持**（edge preservation）。

使用折痕角有时会造成不恰当的平滑程度，将本应形成折痕的边变圆，或者反过来。通常需要进行试验，而且可能不存在一个能对整个网格都完美适用的角度。即使平滑组也存在局限。例如，在一张纸的中间捏出折痕时，可以将整张纸视为一个平滑组，但它内部又有折痕，而平滑组会将这些折痕平滑掉。此时，建模者需要使用多个互相重叠的平滑组，或者直接在网格上定义折痕边。另一个例子是用三角形构成的圆锥。对整个圆锥表面进行平滑，会产生一种奇怪的结果：锥尖只有一条沿圆锥轴线直接向外的法线。锥尖是一个奇点。为了完美表示插值法线，每个三角形实际上需要更像一个四边形，在锥尖位置具有两条法线 [647]。

所幸，这类有问题的情况通常很少见。一旦找到了平滑组，就可以为组内共享的顶点计算顶点法线。教科书中求顶点法线的标准方法，是对共享该顶点的多边形表面法线取平均 [541, 542]。然而，这种方法可能导致不一致且权重分配不合理的结果。Thürmer 和 Wüthrich [1770] 提出了另一种方法：根据每个多边形在该顶点处形成的角度，对其法线的贡献进行加权。这种方法有一个理想性质：无论共享某顶点的多边形是否经过三角剖分，都会得到相同结果。例如，如果一个多边形细分成两个共享该顶点的三角形，那么法线平均法会错误地使这两个三角形的影响力达到原多边形的两倍。参见图 16.12。


![图 16.12：法线平均与按顶点角加权的比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_3_16.12.png)

**图 16.12** 左图对一个四边形和两个三角形的表面法线取平均，得到顶点法线。中图对该四边形进行了三角剖分。由于每个多边形的法线权重相等，这导致平均法线发生偏移。右图使用 Thürmer 和 Wüthrich 的方法，按照构成相应顶点角的两条边之间的夹角，对每条法线的贡献加权，因此三角剖分不会使法线偏移。

Max [1146] 给出了另一种加权方法，其假设是：长边构成的多边形对法线的影响应该更小。当使用简化技术时，这种平滑方式可能更优，因为简化形成的较大多边形更不容易贴合表面的曲率。

Jin 等人 [837] 全面综述了这些方法以及其他方法，得出的结论是，在各种条件下，按角度加权的方法要么是最好的，要么是最好的方法之一。Cignoni [268] 在 Meshlab 中实现了几种方法，并作出了大致相同的观察。他还警告，不要按照每条法线所属三角形的面积对该法线的贡献进行加权。

对于高度场，Shankel [1614] 展示了如何通过求各坐标轴方向上邻点的高度差，快速近似按角度加权方法得到的平滑结果。给定一个点 **p** 和四个相邻点：高度场 x 轴方向上的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_03_48ceb6deba93d3.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_03_719058da8efa3c.png)，以及 y 轴方向上的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_03_dc55d2590aab74.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_03_f2bf45adb780d3.png)，**p** 处的（未归一化）法线可以较好地近似为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_03_c552684f626288.png)


> 译注：式（16.1）的上下标、相减顺序和第三分量 2 均按书页 696 原式保留。原文先描述“邻点的高度差”，但式中写作邻点的 x、y 分量差，记号与文字说明存在疑点；此处不擅自更改原式。


## 16.4 三角形扇、三角形带与网格

来源：《Real-Time Rendering, Fourth Edition》，书页 696—705（PDF 物理页 717—726）；正文从 16.4 标题起，至 16.5 标题前。图 16.12 和式（16.1）属于前节，提前排在书页 705 的图 16.16 属于第 16.5 节，均不纳入本节。

三角形列表是存储和显示一组三角形最简单、通常也是效率最低的方式。每个三角形的顶点数据依次放入列表。每个三角形都有自己独立的一组三个顶点，因此三角形之间不共享顶点数据。提高图形性能的一种标准方法，是把共享顶点的成组三角形送入图形流水线。共享意味着减少顶点着色器的调用，因此需要变换的点和法线也更少。这里介绍多种共享顶点信息的数据结构，从三角形扇和三角形带开始，逐步讨论更复杂、也更高效的曲面渲染表示形式。

### 16.4.1 三角形扇

图 16.13 展示了一个三角形扇。这种数据结构说明，构造三角形时，存储开销可以低于每个三角形三个顶点。所有三角形共享的顶点称为中心顶点，即图中的顶点 0。对于起始三角形 0，依次发送顶点 0、1、2。对于后续三角形，始终使用中心顶点、上一次发送的顶点和当前发送的顶点。发送顶点 3，就构成三角形 1，其三个顶点为 0（始终包含）、2（上一次发送的顶点）和 3。发送顶点 4 构成三角形 2，依此类推。注意，一般的凸多边形很容易表示成三角形扇，因为它的任意一个顶点都可以作为起始的中心顶点。


![图16.13 三角形扇与凸多边形](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_4_16.13.png)

图 16.13：左图说明三角形扇的概念。三角形 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_4b1d7908f5e266.png) 发送顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_08896f686baac3.png)（中心顶点）、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_5607dc78935cb8.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_37d45ef245b1bd.png)。后续三角形 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_130f49865f8afc.png)（i > 0）只发送顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_146f26fb6d871d.png)。右图为凸多边形，它总能转换成一个三角形扇。

由 n 个顶点组成的三角形扇定义为一个有序顶点列表：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_80ba40f085776f.png)


其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_08896f686baac3.png) 是中心顶点；在该列表上规定一种结构，使第 i 个三角形为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_4929a99be8de11.png)


其中 0 ≤ i < n − 2。

如果三角形扇包含 m 个三角形，那么第一个三角形要发送三个顶点，剩余 m − 1 个三角形每个再发送一个顶点。这意味着，对于长度为 m 的顺序三角形扇，平均每个三角形发送的顶点数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1840d2964599e4.png) 可表示为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_355f1fe750d93f.png)


很容易看出，当 m → ∞ 时，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1840d2964599e4.png) → 1。这看起来或许与实际情况关系不大，但可以考虑一个更现实的数值。如果 m = 5，那么 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1840d2964599e4.png) = 1.4，也就是说，平均每个三角形只发送 1.4 个顶点。

### 16.4.2 三角形带

三角形带与三角形扇相似，都会重用先前三角形中的顶点。不过，它重用的不是一个固定的中心点和前一个顶点，而是前一个三角形的两个顶点，用它们帮助构成下一个三角形。考虑图 16.14。如果把这些三角形当作一个带，就能以更紧凑的方式将它们发送到渲染流水线。对于第一个三角形（记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_4b1d7908f5e266.png)），依次发送全部三个顶点（记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_08896f686baac3.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_5607dc78935cb8.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_37d45ef245b1bd.png)）。对于带中的后续三角形，只须发送一个顶点，因为另外两个顶点已经随前一个三角形发送过。例如，发送三角形 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_47714cb824af95.png) 时，只发送顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_239e8d38e2bb18.png)，并利用三角形 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_4b1d7908f5e266.png) 的顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_5607dc78935cb8.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_37d45ef245b1bd.png) 来构成 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_47714cb824af95.png)。对于三角形 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_85e3550cabd641.png)，只发送顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_25a7dc3c5b57aa.png)，带中的其余部分依此类推。


![图16.14 三角形带](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_4_16.14.png)

图 16.14：可以表示成一个三角形带的三角形序列。注意，带内各三角形的朝向交替变化，而带中的第一个三角形决定所有三角形的朝向。内部通过按 [0, 1, 2]、[1, 3, 2]、[2, 3, 4]、[3, 5, 4] 等顺序遍历顶点，保持一致的逆时针次序。

由 n 个顶点组成的顺序三角形带定义为一个有序顶点列表：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_74d74a6e49eab0.png)


在该列表上规定一种结构，使第 i 个三角形为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d1fd3b3d727b57.png)


其中 0 ≤ i < n − 2。这种带称为顺序三角形带，是因为顶点按给定次序发送。这一定义意味着，具有 n 个顶点的顺序三角形带包含 n − 2 个三角形。

对于长度为 m（即包含 m 个三角形）的三角形带，其平均顶点数也记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1840d2964599e4.png)，分析与三角形扇相同（见式 16.4），因为二者的启动阶段相同，之后每个新三角形都只发送一个顶点。同样，当 m → ∞ 时，三角形带的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1840d2964599e4.png) 自然也趋于每个三角形一个顶点。当 m = 20 时，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1840d2964599e4.png) = 1.1，远好于 3，并且已接近极限值 1.0。与三角形扇一样，第一个三角形总要付出三个顶点的启动开销，这一开销由后续三角形分摊。

三角形带的吸引力正来自这一点。视渲染流水线的瓶颈位置而定，相对于简单三角形列表，渲染时间最多可能节省三分之二。加速源于避免了冗余操作，例如将每个顶点向图形硬件发送两次，以及随后对各顶点执行矩阵变换、裁剪等操作。三角形带适合草叶等对象，以及边界顶点不会被其他带重用的对象。由于形式简单，几何着色器在输出多个三角形时会使用三角形带。

三角形带有若干变体，例如不对三角形施加严格的顺序限制，或者使用重复顶点或重新开始索引值，将多个不相连的带存入同一个缓冲区。过去，如何以最佳方式将任意三角形网格分解成带，曾是研究热点 [1076]。这类研究后来逐渐消退，因为索引三角形网格的引入允许更充分地重用顶点数据，使显示更快，而且通常减少总体内存需求。

### 16.4.3 三角形网格

三角形扇和三角形带仍有用途，但在所有现代 GPU 上，复杂模型通常采用具有单一索引列表的三角形网格（第 16.3.1 节）[1135]。带和扇允许一定程度的数据共享，而网格存储允许更充分的共享。在网格中，额外的索引数组记录哪些顶点组成各个三角形。这样，一个顶点就可以关联多个三角形。

连通平面图的欧拉–庞加莱公式 [135] 有助于确定构成闭合网格的平均顶点数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_02313e0ee2f88a.png)


这里，v 是顶点数，e 是边数，f 是面数，g 是亏格。亏格是物体中孔洞的数量。例如，球面的亏格为 0，环面的亏格为 1。这里假定每个面只有一个边界环。如果面可以有多个边界环，公式变为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_1eed0314cdf870.png)


其中 l 是边界环的数量。

对于闭合（实体）模型，每条边关联两个面，每个面至少有三条边，所以 2e ≥ 3f。如果网格完全由三角形组成，正如 GPU 所要求的那样，那么 2e = 3f。假设亏格为 0，并在公式中用 1.5f 代替 e，可得 f ≤ 2v − 4。如果所有面都是三角形，则 f = 2v − 4。

因此，对于大型闭合三角形网格，经验法则是三角形数量大约等于顶点数量的两倍。同样可以发现，每个顶点平均连接近六个三角形（因此也连接六条边）。与一个顶点相连的边数称为该顶点的价。注意，网格的连接结构不会影响这个结果，只有三角形数量会影响它。由于带中每个三角形的平均顶点数趋近于一，而顶点数量是三角形数量的两倍，因此，如果用三角形带表示大型网格，每个顶点平均必须发送两次。在极限情况下，三角形网格每个三角形可以只发送 0.5 个顶点。

> 译注：原书此处写作“顶点数量是三角形数量的两倍”，与本段开头及其后推论相反，疑为倒置；应理解为“三角形数量约为顶点数量的两倍”。此外，上段从 2e ≥ 3f 推得一般多边形情形的 f ≤ 2v − 4 时，应使用不等式；用 e = 1.5f 直接代入对应的是全三角形情形。此处保留原文陈述并指出疑点。

注意，这一分析仅适用于光滑、闭合的网格。一旦存在边界边（没有被两个多边形共享的边），顶点数与三角形数之比就会上升。欧拉–庞加莱公式仍然成立，但必须把网格的外边界看作一个单独的（未使用的）面，它邻接所有外部边。同样，任意模型中的每个平滑组实际上都是一个独立网格，因为在两个组相接的锐边处，GPU 需要使用具有不同法线的独立顶点记录。例如，立方体的一个角在同一位置有三条法线，因此存储三条顶点记录。纹理或其他顶点数据的变化也可能增加不同顶点记录的数量。

理论预言，每个三角形大约需要处理 0.5 个顶点。实践中，GPU 变换顶点后，会将它们放入先进先出（FIFO）缓存，或近似最近最少使用（LRU）策略的系统 [858]。该缓存保存每个经过顶点着色器处理的顶点的变换后结果。如果传入的顶点已经在缓存中，就可以直接使用缓存的变换后结果，无须调用顶点着色器，从而显著提高性能。反之，如果三角形网格中的三角形以随机顺序发送，缓存就不太可能发挥作用。三角形带算法相当于针对大小为二的缓存进行优化，即只保留最近使用的两个顶点。Deering 和 Nelson [340] 最早探索了在更大的 FIFO 缓存中保存顶点数据的思路，利用算法确定顶点加入缓存的顺序。

FIFO 缓存的容量有限。例如，PLAYSTATION 3 系统能容纳大约 24 个顶点，具体取决于每个顶点的字节数。较新的 GPU 并未显著增加这一缓存，典型的最大容量为 32 个顶点。

Hoppe [771] 引入了衡量缓存重用的重要指标：平均缓存未命中率（ACMR）。它表示每个三角形平均需要处理的顶点数。其范围可以从 3（每个三角形的每个顶点每次都必须重新处理）到 0.5（大型闭合网格上的完美重用，没有顶点被重复处理）。如果缓存足以容纳整个网格，ACMR 就等于理论上的顶点数与三角形数之比。对于给定的缓存大小和网格顺序，可以精确计算 ACMR，从而描述某种方法在该缓存大小下的效率。

### 16.4.4 缓存无关的网格布局

网格中理想的三角形顺序应最大限度地利用顶点缓存。Hoppe [771] 提出了一种使网格 ACMR 最小化的算法，但必须预先知道缓存大小。如果假定的缓存大于实际缓存，得到的网格可能获益明显减少。针对不同大小的缓存求解，可能得到不同的最优顺序。在目标缓存大小未知时，可以使用缓存无关的网格布局算法；这类算法生成的顺序，无论缓存大小如何，都能表现良好。这种顺序有时称为通用索引序列。

Forsyth [485] 以及 Lin 和 Yu [1047] 给出了采用相似原理的快速贪心算法。根据顶点在缓存中的位置，以及与其相连但尚未处理的三角形数量，为顶点评分。接下来处理顶点总分最高的三角形。通过给最近使用的三个顶点略低的分数，算法避免简单地生成三角形带，而是形成类似希尔伯特曲线的模式。通过给剩余相连三角形较少的顶点更高分数，算法倾向于避免留下孤立三角形。获得的平均缓存未命中率可与开销更大、更复杂的算法媲美。Lin 和 Yu 的方法稍复杂一些，但采用相关思路。对于大小为 12 的缓存，一组 30 个未经优化的模型的平均 ACMR 为 1.522；优化后，根据缓存大小，平均值降至 0.664 或更低。

Sander 等人 [1544] 概述了已有工作，并提出自己的更快方法 Tipsify，不过它并不是缓存大小无关的方法。它增加的一项考虑，是尽量将最外层的三角形提前放入列表，以尽量减少过度绘制（第 18.4.5 节）。例如，想象一个咖啡杯。先渲染组成杯子外部的三角形，后面渲染的内部三角形就很可能被遮挡。

Storsjö [1708] 对比了 Forsyth 和 Sander 的方法，并提供了二者的实现。他认为，这些方法产生的布局已接近理论极限。Kapoulkine [858] 的一项较新研究在三家硬件厂商的 GPU 上比较了四种感知缓存的顶点排序算法。他的结论之一是，Intel 使用一个具有 128 个条目的 FIFO，每个顶点占用三个或更多条目；而 AMD 和 NVIDIA 的系统近似于具有 16 个条目的 LRU 缓存。这种架构差异显著影响算法行为。他发现，Tipsify [1544] 在这些平台上都表现较好，Forsyth 算法 [485] 也有类似表现，但程度稍弱。

总的来说，离线预处理三角形网格能够明显改善顶点缓存性能；当顶点阶段是瓶颈时，还能提高总体帧率。这种处理很快，在实践中实际达到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3657599716cc3d.png)。已有若干开源版本可用 [485]。由于这类算法可以自动应用于网格，优化不增加存储开销，也不影响工具链中的其他工具，因此这些方法通常是成熟开发系统的一部分。例如，Forsyth 算法似乎就是 PLAYSTATION 网格处理工具链的一部分。虽然现代 GPU 采用统一着色器架构后，顶点变换后缓存已经发生演变，但避免缓存未命中仍是重要问题 [530]。

### 16.4.5 顶点与索引缓冲区／数组

向现代图形加速器提供模型数据的一种方式，是使用 DirectX 所称的顶点缓冲区，以及 OpenGL 所称的顶点缓冲区对象（VBO）。本节采用 DirectX 术语，所介绍的概念在 OpenGL 中都有对应形式。

顶点缓冲区的思想，是在一块连续内存中保存模型数据。顶点缓冲区是采用特定格式的顶点数据数组。格式规定顶点是否包含法线、纹理坐标、颜色或其他特定信息。每个顶点的数据集中成组，各顶点依次排列。一个顶点占用的字节数称为其步长。这种存储方式称为交错缓冲区。另一种方式是使用一组顶点流。例如，一个流保存位置数组 {![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) …}，另一个流单独保存法线数组 {![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png) …}。实践中，将每个顶点的全部数据放在单一缓冲区中，通常在 GPU 上更高效，但优势还没有大到必须避免多流的程度 [66, 1494]。多流的主要开销是额外的 API 调用；当应用程序受 CPU 限制时，这一开销可能值得避免，否则并不显著 [443]。

Wihlidal [1884] 讨论了多流改善渲染系统性能的不同方式，包括 API、缓存和 CPU 处理方面的优势。例如，CPU 上用于向量处理的 SSE 和 AVX 更容易应用于独立的数据流。使用多流的另一个原因，是可以更高效地更新网格。比如，若只有顶点位置流随时间变化，那么只更新这一个属性缓冲区，要比构造并发送整个交错数据流的开销更低 [1609]。

如何访问顶点缓冲区，由设备的 DrawPrimitive 方法决定。数据可以被解释为：

1. 独立点的列表。
2. 不相连的线段列表，即顶点对。
3. 一条折线。
4. 三角形列表，每三个顶点构成一个三角形，例如 [0, 1, 2] 构成一个，[3, 4, 5] 构成下一个，依此类推。
5. 三角形扇，第一个顶点与后续每对顶点构成三角形，例如 [0, 1, 2]、[0, 2, 3]、[0, 3, 4]。
6. 三角形带，每三个连续顶点构成一个三角形，例如 [0, 1, 2]、[1, 2, 3]、[2, 3, 4]。

从 DirectX 10 开始，三角形和三角形带还可以包含相邻三角形的顶点，供几何着色器使用（第 3.7 节）。

顶点缓冲区既可以直接使用，也可以由索引缓冲区引用。索引缓冲区中的索引保存顶点在顶点缓冲区中的位置。索引存储为 16 位无符号整数；如果网格很大，而且 GPU 和 API 支持，也可以使用 32 位（第 16.6 节）。索引缓冲区与顶点缓冲区的组合，可以显示与“原始”顶点缓冲区相同类型的绘制图元。区别在于，索引／顶点缓冲区组合中的每个顶点，在顶点缓冲区里只需存储一次；没有索引的顶点缓冲区则可能重复存储顶点。

三角形网格的结构由索引缓冲区表示。索引缓冲区中最先存储的三个索引指定第一个三角形，接下来三个指定第二个，依此类推。这种组织称为索引三角形列表，即索引本身构成三角形列表。OpenGL 使用顶点数组对象（VAO），把索引缓冲区、一个或多个顶点缓冲区以及顶点格式信息绑定在一起。索引也可以按三角形带顺序排列，从而节省索引缓冲区空间。这种格式称为索引三角形带，实践中很少使用，因为为大型网格创建这样一组带需要付出一定工作，而且处理几何的所有工具也必须支持这种格式。图 16.15 给出了顶点与索引缓冲区结构的例子。


![图16.15 顶点和索引缓冲区的组织方式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_4_16.15.png)

图 16.15：定义图元的不同方式，从上至下大致按内存使用量由多到少排列：独立三角形、顶点三角形列表、使用两个或一个数据流的三角形带，以及列出独立三角形或按三角形带顺序排列的索引缓冲区。

图内文字与数据完整译录：

- 三个三角形，由顶点位置 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) 至 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png)、法线 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png) 至 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png) 组成。
- 可以通过一系列独立调用渲染这些三角形：begin, ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png), end, begin, ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png), end, begin, ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png), end。
- 位置和法线可以放入两个独立列表。将这两个数组视为三角形列表，使数组中每个互不重叠的三元组构成一个三角形。位置数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png)]；法线数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png)]。
- 位置和法线也可以放入数组，让每个连续三元组定义一个三角形。位置数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png)]；法线数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png)]。
- 每个顶点可以放入一个交错数组，使每个互不重叠的三元组，或每个连续三元组（即三角形带）构成一个三角形。图示三角形带的顶点数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png)]。
- 每个顶点可以放入单一数组，并使用索引列表指定独立三角形。顶点数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png)]；索引数组为 [0 1 2　1 3 2　2 3 0]。
- 每个顶点可以放入单一数组，并使用索引列表定义三角形带。顶点数组为 [![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_8794b6a3c050bf.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_19aa1dc33bda1e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a00723a6f5471.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_3a55303a18d915.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_d13080baeee9f5.png)　![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_6aa5478e61d94d.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_04_f2a16a55b07b30.png)]；索引数组为 [0 1 2 3 0]。

采用哪种结构，取决于图元和程序。显示简单矩形时，只用一个顶点缓冲区，将四个顶点表示为由两个三角形组成的带或扇，就很容易实现。如前所述，索引缓冲区的一个优势是共享数据。另一个优势是简单：三角形可以采用任意顺序和配置，没有三角形带那种严格的前后衔接要求。最后，使用索引缓冲区时，需要传输并存入 GPU 的数据量通常更小。共享顶点节省的内存，远远超过引入索引数组所需的少量开销。

一个索引缓冲区加上一个或多个顶点缓冲区，可以描述多边形网格。不过，数据通常以 GPU 渲染效率为目标来存储，不一定采用最紧凑的形式。例如，存储立方体的一种方法，是将八个角的位置保存在一个数组中，将六个不同的法线保存在另一个数组中，再保存定义六个面的六个四索引边界环。此时每个顶点位置用两个索引描述，一个指向顶点列表，另一个指向法线列表。纹理坐标再使用另一个数组和第三个索引。许多模型文件格式采用这种紧凑表示，例如 Wavefront OBJ。在 GPU 上，只能使用一个索引缓冲区。单一顶点缓冲区会存储 24 个不同的顶点，因为每个角位置具有三条独立法线，分别对应相邻的三个面。索引缓冲区保存定义表面 12 个三角形的索引。Masserann [1135] 讨论了如何把这类文件描述高效地转换成紧凑、高效的索引／顶点缓冲区，而不是不共享顶点的非索引三角形列表。还可以采用更紧凑的方案，例如将网格存储在纹理图或缓冲区纹理中，并使用顶点着色器的纹理读取或拉取机制，但这样无法利用顶点变换后缓存，需要付出性能代价 [223, 1457]。

为了达到最高效率，顶点缓冲区中的顶点顺序应与索引缓冲区访问它们的顺序一致。也就是说，索引缓冲区中第一个三角形引用的前三个顶点，应位于顶点缓冲区最前面。每当索引缓冲区遇到一个新顶点，它就应当是顶点缓冲区中的下一个顶点。这种排序能最大限度地减少顶点变换前缓存的未命中；该缓存与第 16.4.4 节讨论的变换后缓存是分开的。重新排列顶点缓冲区中的数据是一项简单操作，但它对性能的重要性，可能与为顶点变换后缓存寻找高效的三角形顺序同样大 [485]。

还有更高层次的方法，可以通过分配和使用顶点与索引缓冲区获得更高效率。例如，不发生变化的缓冲区可以存放在 GPU 上供每帧使用；同一缓冲区也可以生成某个对象的多个实例和变体。第 18.4.2 节将深入讨论这些技术。

利用流水线的流输出功能（第 3.7.1 节），可以将处理后的顶点发送到新缓冲区，从而在 GPU 上处理顶点缓冲区而不进行渲染。例如，描述三角形网格的顶点缓冲区，可以在初始一遍处理中被当作简单的点集。顶点着色器可以按需执行逐顶点计算，并通过流输出把结果发送到新的顶点缓冲区。在后续一遍处理中，可以将这个新的顶点缓冲区与描述网格连接关系的原始索引缓冲区配对，进一步处理并显示得到的网格。


## 16.5 简化

来源：原书第 706—712 页（PDF 第 727—733 页）；图 16.16 及其图注位于原书第 705 页（PDF 第 726 页）。正文从 16.5 标题开始，到 16.6 标题之前结束。

网格简化（mesh simplification），也称数据缩减（data reduction）或减面（decimation），是对一个精细模型减少其三角形数量，同时尽量保持其外观的过程。在实时应用中，这样做是为了减少需要存储并沿流水线传送的顶点数量。这对于使应用程序能够适应不同性能的硬件可能十分重要，因为性能较弱的机器可能需要显示更少的三角形。接收到的模型数据也可能经过了过度的曲面细分，超出了合理表达模型所需的程度。图 16.16 展示了数据缩减技术如何减少需要存储的三角形数量。


![图16.16 火山口湖高度场及简化网格](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.16.png)

**图 16.16** 左上图是使用 200,000 个三角形渲染的火山口湖（Crater Lake）高度场。右上图显示该模型被简化为由 1,000 个三角形构成的不规则三角网（triangulated irregular network，TIN）。下图显示其底层的简化网格。（图片由 Michael Garland 提供。）

Luebke [1091, 1092] 将网格简化分为三种：静态简化、动态简化和视点相关简化。静态简化的思路是在开始渲染之前创建彼此独立的细节层次（level of detail，LOD）模型，然后由渲染器从中选择。这种形式将在第 19.9 节介绍。离线简化也可以用于其他任务，例如提供供细分曲面进一步细化的粗网格 [1006, 1007]。动态简化提供的是连续的一系列 LOD 模型，而不是少数几个离散模型，因此这类方法被称为连续细节层次（continuous level of detail，CLOD）算法。视点相关技术适用于模型内部不同部分采用不同细节层次的情况。具体而言，地形渲染就是这样的例子：视野中附近的区域需要精细表示，而远处的区域则使用较低的细节层次。本节讨论后两种简化方式。

### 16.5.1 动态简化

减少三角形数量的一种方法是使用边折叠（edge collapse）操作：通过移动一条边的两个顶点，使它们重合，从而删除这条边。图 16.17 给出了这一操作的示例。对于实体模型，一次边折叠总共会删除两个三角形、三条边和一个顶点。因此，一个包含 3,000 个三角形的封闭模型，需要执行 1,500 次边折叠才能使面数减少到零。一个经验法则是：具有 v 个顶点的封闭三角网格大约有 2v 个面和 3v 条边。这个法则可以由实体表面的欧拉—庞加莱公式 f − e + v = 2 推导出来（第 16.4.3 节）。

> 译注：此处保留原书“1,500 次折叠减至零面”的计数说明。上述欧拉关系适用于拓扑上等同于球面的闭合曲面；实际要求保持有效封闭网格拓扑的边折叠，不能不加限制地一直执行到零面。


![图16.17 边折叠操作](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.17.png)

**图 16.17** 左图表示 uv 边折叠发生之前的情况；右图表示点 u 折叠到点 v，从而删除三角形 A、B 以及边 uv。

边折叠过程是可逆的。按顺序存储边折叠操作后，我们就能从简化模型开始，重建复杂模型。这一特性对通过网络传输模型很有用：经过边折叠的数据可以以高效压缩的形式发送，并在接收过程中逐步构建和显示模型 [768, 1751]。由于具有这一特性，这种简化过程通常称为视点无关渐进网格（view-independent progressive meshing，VIPM）。

在图 16.17 中，u 被折叠到 v 的位置，但也可以将 v 折叠到 u。如果一个简化系统仅允许这两种可能，它采用的就是子集放置策略（subset placement strategy）。这种策略的一个优点是，限制可能的选择后，我们或许能够隐式编码所做的选择 [516, 768]。由于需要评估的可能性更少，这种策略速度更快；但由于考察的解空间较小，它也可能得到质量较低的近似结果。

使用最优放置策略（optimal placement strategy）时，我们考察的可能性范围更广。不是将一个顶点折叠到另一个顶点，而是将一条边的两个顶点一起收缩到一个新位置。Hoppe [768] 考察了 u 和 v 都移动到连接二者的边上某个位置的情况。他指出，为提高最终数据表示的压缩率，可以将搜索限制为只检查中点。Garland 和 Heckbert [516] 则更进一步，通过求解一个二次方程来寻找最优位置，而这个位置可能不在原来的边上。最优放置策略的优点在于，它们通常能够生成质量更高的网格。其缺点是需要额外的处理和代码，以及用于记录更广泛放置可能性的内存。

为了确定最佳的点放置位置，我们需要分析局部邻域。这种局部性是一项重要且有用的特性，原因有几个。如果一次边折叠的代价仅取决于少数局部变量，例如边长和边附近各面的法线，那么代价函数就很容易计算，而且每次折叠只影响少量邻居。例如，假设在开始时为某个模型计算了 3,000 种可能的边折叠。首先执行代价函数值最低的边折叠。由于它只影响附近少量三角形及其边，所以只需重新计算那些代价函数受这些变化影响的边折叠候选，例如重新计算 10 个，而不是 3,000 个；候选列表也只需进行少量重新排序。由于一次边折叠只会影响少数其他边折叠的代价值，使用堆或其他优先队列来维护这份代价值列表是很好的选择 [1649]。

有些收缩操作无论代价如何都必须避免。图 16.18 给出了一个例子。检查折叠是否使邻接三角形的法线方向翻转，就能检测出这类情况。


![图16.18 不良折叠造成边交叉](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.18.png)

**图 16.18** 一次不良折叠的例子。左图是顶点 u 折叠到 v 之前的网格。右图是折叠之后的网格，展示了边如何发生交叉。图内的“edge crossing”意为“边交叉”。

折叠操作本身就是对模型数据库的一次编辑。用于存储这些折叠的数据结构已有详细文献介绍 [481, 770, 1196, 1726]。使用代价函数分析每个边折叠，然后执行其中代价值最小的那个。最佳代价函数可以而且确实会随着模型类型及其他因素而变化 [1092]。根据所要解决的问题，代价函数可以在速度、质量、稳健性和简单性之间进行权衡。也可以专门设计代价函数，以保持表面边界、材质位置、光照效果、沿某条轴的对称性、纹理放置、体积或其他约束。

为了说明此类函数如何工作，我们将介绍 Garland 和 Heckbert 的二次误差度量（quadric error metric，QEM）代价函数 [515, 516]。这一函数广泛适用于许多情况。相比之下，Garland 和 Heckbert 在较早的研究 [514] 中发现，使用豪斯多夫距离（Hausdorff distance）进行地形简化效果最好，其他研究者也证实了这一点 [1496]。这个函数就是简化网格中的顶点到原始网格的最大距离。图 16.16 展示了使用这一度量得到的结果。

> 译注：上一句依原书表述翻译。严格的对称豪斯多夫距离需要同时考虑两个集合到对方的距离；原书这里给出的是针对简化网格顶点到原网格的直观单向描述。

对于一个给定顶点，有一组三角形共享该顶点，而每个三角形都对应一个平面方程。移动顶点的 QEM 代价函数，就是新位置到这些平面各自的距离平方之和。更形式化地说，


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_05_88728472822670.png)


是新位置 **v** 相对于 m 个平面的代价函数，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_05_34778776540256.png) 是平面 i 的法线，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_05_8039a811d0b4aa.png) 是该平面相对于原点的偏移量。

图 16.19 展示了同一条边的两种可能收缩。假设立方体的宽度为两个单位。将 e 折叠到 c（e → c）的代价函数值为 0，因为点 e 移动到 c 时，并未离开与它关联的那些平面。c → e 的代价函数值为 1，因为 c 离开了立方体右侧面所在的平面，移动距离的平方为 1。由于 e → c 的代价更低，因此它优于 c → e。


![图16.19 同一条边的两种收缩](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.19.png)

**图 16.19** 左图是一个在某条边上多出一个点的立方体。中图显示将这个点 e 折叠到角点 c 后的情况。右图显示将 c 折叠到 e 后的情况。

这个代价函数可以进行多种修改。设想两个三角形共享一条边，并构成一道锐利的边缘，例如它们是鱼鳍或涡轮叶片的一部分。折叠这条边上一个顶点的代价函数值很低，因为一个点沿着其中一个三角形滑动时，不会远离另一个三角形所在的平面。基本函数的代价值与移除该特征所引起的体积变化有关，但不能很好地反映该特征的视觉重要性。保留锐利折痕边的一种方法是，添加一个包含该边的额外平面，其法线为这两个三角形法线的平均值。这样，远离该边的顶点就会具有较高的代价函数值 [517]。一种变体是用三角形面积的变化来加权代价函数。

另一种扩展是，使用以保持其他表面特征为依据的代价函数。例如，模型的折痕边和边界边对于表现模型十分重要，因此应降低它们被修改的可能性。见图 16.20。其他值得保留的表面特征包括材质发生变化的位置、纹理贴图边界以及逐顶点颜色发生变化的位置 [772]。见图 16.21。


![图16.20 飞机网格简化](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.20.png)

**图 16.20** 网格简化。左上图为包含 13,546 个面的原始网格，右上图简化为 1,000 个面，左下图简化为 500 个面，右下图简化为 150 个面 [770]。（图片 ©1996 Microsoft。保留所有权利。）


![图16.21 保持纹理的网格简化](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.21.png)

**图 16.21** 网格简化。上排：显示网格并使用简单的灰色材质。下排：使用纹理。从左到右，模型分别包含 51,123、6,389 和 1,596 个三角形。模型上的纹理尽可能得到保留，不过随着三角形数量减少，仍然逐渐出现一些失真。（图片 ©2016 Microsoft。保留所有权利。）

大多数简化算法都会遇到一个严重问题：纹理的外观常常会明显偏离原始外观 [1092]。随着边被折叠，纹理到底层表面的映射可能发生扭曲。此外，纹理坐标值在边界处可能相同，却属于应用纹理的不同区域，例如模型沿中央的一条边进行镜像时就是如此。Caillaud 等人 [220] 综述了此前的多种方法，并提出了他们自己的纹理接缝处理算法。

速度也是需要考虑的问题。在由用户创建内容的系统中，例如 CAD 系统，需要即时生成细节层次模型。利用 GPU 执行简化已经取得了一定成功 [1008]。另一种思路是使用更简单的简化算法，例如顶点聚类（vertex clustering）[1088, 1511]。这种方法的核心思想是，用三维体素网格或类似结构覆盖模型。一个体素中的所有顶点都会被移动到该单元的“最佳”顶点位置。这样可能消除一些三角形：当某个三角形的两个或更多顶点落到同一位置时，它就会退化。这种算法很稳健，不需要网格的连接信息，而且很容易将多个独立网格合并为一个。不过，基本的顶点聚类算法很少能得到与完整 QEM 方法同样好的结果。Willmott [1890] 讨论了他的团队如何让这种聚类方法稳健而高效地处理游戏《孢子》（Spore）中由用户创建的内容。

将表面的原始几何转换为用于凹凸映射的法线贴图，是一种与简化相关的思路。纽扣或皱纹等细小特征可以用纹理表示，而保真度损失很小。Sander 等人 [1540] 讨论了这一领域此前的工作，并给出了一种解决方案。这类算法通常用于交互式应用的模型开发，将高质量模型烘焙成带纹理的表示 [59]。

简化技术能够从一个复杂模型生成大量细节层次（LOD）模型。使用 LOD 模型时会遇到一个问题：如果相邻两帧之间一个模型瞬间替换另一个模型，有时就能看出切换过程 [508]。这个问题称为“突跳”（popping）。一种解决方法是使用几何渐变（geomorph）[768] 来增加或减少细节层次。由于我们知道复杂模型中的顶点如何映射到简单模型，因此可以创建平滑过渡。更多细节见第 19.9.1 节。

使用视点无关渐进网格的一个优点是，只需创建一次顶点缓冲区，同一模型处于不同细节层次的各个副本就可以共享它 [1726]。不过，在基本方案中，仍需为每个副本创建独立的索引缓冲区。另一个问题是效率。由于折叠顺序决定了三角形的显示顺序，顶点缓存的相干性较差。Forsyth [481] 讨论了若干实用方案，用于提高建立和共享索引缓冲区时的效率。

网格缩减技术虽然有用，但完全自动化的系统并非万能。图 16.22 展示了保持对称性方面的问题。有经验的模型制作者能够创建三角形数量很少、质量却优于自动流程生成结果的物体。例如，眼睛和嘴是面部最重要的部分，而一个简单粗糙的算法可能把这些部分当成无关紧要的细节平滑掉。重拓扑（retopology）是在模型中添加边的过程，使各种特征在应用建模、平滑或简化技术时仍能保持彼此分离。与简化相关的算法仍在不断发展，并尽可能实现自动化。


![图16.22 自动简化的对称性问题](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_5_16.22.png)

**图 16.22** 对称性问题。左侧圆柱体有 10 个平面面片，包括顶面和底面。中间圆柱体经自动缩减消除一个面后，剩下 9 个平面面片。右侧圆柱体由建模器的曲面面片化工具重新生成，也有 9 个平面面片。


## 16.6 压缩与精度

来源：《Real-Time Rendering》第4版，书页712—715（PDF物理页733—736）；从16.6节标题起，至“延伸阅读与资源”标题前。本节没有下级小节。

三角形网格数据可以采用多种方式压缩，并由此获得类似的收益。正如PNG和JPEG图像文件格式对纹理采用无损和有损压缩一样，人们也为三角形网格数据的压缩开发了多种算法和格式。

压缩以编码和解码所花费的时间为代价，尽量减少数据存储占用的空间。传输更小的数据表示所节省的时间，必须超过解压数据额外花费的时间。在互联网上传输时，下载速度较慢意味着可以采用更复杂的算法。网格的连接关系可以使用TFAN [1116]进行压缩和高效解码；该方法已被MPEG-4采用。Open3DGC、OpenCTM和Draco等编码器生成的模型文件，其大小可以降至仅采用gzip压缩时的四分之一，甚至更小 [1335]。这些方案中的解压原本就被设计成一次性操作，速度相对较慢，每秒只能处理几百万个三角形，但节省的数据传输时间足以抵偿这项开销，而且可能绰绰有余。Maglo等人 [1099]对相关算法作了全面综述。这里，我们重点讨论直接涉及GPU本身的压缩技术。

本章的很大一部分篇幅都在介绍尽量减少三角形网格存储空间的各种方法。这样做的主要动机是提高渲染效率。在多个三角形之间复用顶点数据，而不是重复存储，可以减少缓存未命中。删除视觉影响很小的三角形，既能减少顶点处理，也能节省内存。内存占用更小，意味着带宽开销更低、缓存利用更好。此外，GPU可在内存中存储的数据量也有限，因此，数据缩减技术可以增加能够显示的三角形数量。

顶点数据可以采用固定码率压缩，其原因与压缩纹理时类似（第6.2.6节）。这里所说的**固定码率压缩**，指最终压缩后存储大小已知的方法。如果每个顶点都采用自包含的压缩表示，就可以在GPU上解码。Calver [221]介绍了多种利用顶点着色器进行解压的方案。Zarge [1961]指出，数据压缩还有助于使顶点格式与缓存行对齐。Purnomo等人 [1448]将简化与顶点量化技术结合起来，利用图像空间度量，针对给定的目标网格大小优化网格。

索引缓冲区的格式中就存在一种简单的压缩方式。索引缓冲区由无符号整数数组组成，这些整数给出顶点在顶点缓冲区数组中的位置。如果顶点缓冲区中的顶点数小于或等于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_06_2beea91ec90c47.png)，那么索引缓冲区就可以使用无符号短整数，而不是无符号长整数。某些API支持对顶点数少于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_06_c04da3533f52b4.png)的网格使用无符号字节，但这样可能造成代价高昂的对齐问题，因此通常会避免使用。值得注意的是，OpenGL ES 2.0、未启用扩展的WebGL 1.0，以及某些较老的台式机和笔记本电脑GPU，都存在不支持无符号长整数索引缓冲区的限制，因此必须使用无符号短整数。

另一个压缩机会在于三角形网格数据本身。举一个基本例子，有些三角形网格会为每个顶点存储一个或多个颜色，用来表示烘焙好的光照、仿真结果或其他信息。在典型显示器上，一个颜色由红、绿、蓝三个各占8位的分量表示，因此可以在顶点记录中用三个无符号字节存储这些数据，而不必使用三个浮点数。GPU的顶点着色器可以把这个字段转换成独立的数值，随后在遍历三角形时对它们进行插值。不过，在许多架构上都需要谨慎处理。例如，Apple建议在iOS上把3字节数据字段填充到4字节，以避免额外处理 [66]。参见图16.23中间的示意图。

另一种压缩方法是完全不存储颜色。例如，如果颜色数据表示温度结果，就可以把温度本身存成一个数值，再将其转换为一维纹理的索引，从中取得颜色。更进一步，如果不需要温度值本身，那么只用一个无符号字节就能引用这张颜色纹理。

即使存储温度本身，也可能只需要精确到小数点后几位。一个浮点数的总精度为24位，略多于7位十进制有效数字。请注意，16位就能提供接近5位十进制有效数字的精度。温度值的范围很可能足够小，以至于不需要浮点格式的指数部分。以最小值作为偏移量，以最大值减去最小值作为缩放量，就能把这些数值均匀分布到一个有限的范围内。例如，如果数值范围是28.51至197.12，那么可以把一个无符号短整数值转换为温度：先将它除以![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_06_81e2c04de949a7.png)，再将结果乘以缩放因子（197.12 − 28.51），最后加上偏移量28.51。只要存储数据集的缩放与偏移因子，并将它们传给顶点着色器程序，数据集本身就只需原来一半的存储空间。这种变换称为**标量量化** [1099]。


![图16.23 顶点数据的固定码率压缩](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_16_6_16.23.png)

**图16.23** 顶点数据的典型固定码率压缩方法。（八分体转换示意图出自Cigolle等人 [269]，由Morgan McGuire惠允提供。）

图内标签译文：左侧为“单精度浮点数据”；上排为位置x、y、z，使用包围盒的尺寸转换为无符号短整数；中排为红、绿、蓝，转换为字节并添加填充；下排为法线x、y、z，采用八分体编码，得到oct.u和oct.v两个分量。

顶点位置数据通常很适合采用这种缩减方法。单个网格在空间中覆盖的区域很小，因此，为整个场景设置缩放和偏移向量（或一个4 × 4矩阵），便可以节省大量空间，而不会明显损失保真度。对于某些场景，可以为每个对象分别生成缩放和偏移，从而提高每个模型的精度。不过，这样做可能在独立网格接触的地方产生裂缝 [1381]。原本处于相同世界位置、但分别属于不同模型的顶点，经过缩放和偏移后，可能落在略有不同的位置。当所有模型相对于整个场景都比较小时，一种解决办法是对所有模型使用相同的缩放量，并使偏移量对齐；这样可以多获得几位精度 [1010]。

有时，即使用浮点数存储顶点数据，也不足以避免精度问题。一个经典例子是在地球上空渲染航天飞机。航天飞机模型本身可能精细到毫米尺度，但它距地表超过100,000米，尺度差异达到8个十进制数量级。当相对于地球计算航天飞机的世界空间位置时，所生成的顶点位置就需要更高的精度。如果不采取修正措施，当观察者在航天飞机附近移动时，航天飞机就会在屏幕上抖动。虽然航天飞机是这种问题的极端例子，但大型多人游戏世界如果始终使用同一个坐标系，也会受到同样的影响。位于边缘区域的对象会损失足够多的精度，使问题变得可见：动画对象会跳动，各个顶点会在不同时间突然跳到另一个位置，而相机只要稍微移动，阴影贴图的纹素就会跳变。一种解决方法是重新组织变换流水线，使每个以原点为中心的对象所用的世界平移和相机平移先合并起来，从而让它们在很大程度上相互抵消 [1379, 1381]。另一种方法是将世界分区，并将原点重新定义到每个分区的中心；这时的难点就变成了如何从一个分区移动到另一个分区。Ohlarik [1316]以及Cozzi和Ring [299]深入讨论了这些问题及其解决方法。

其他顶点数据也可能有专门适用的压缩技术。纹理坐标通常限制在[0.0, 1.0]范围内，因此一般可以安全地缩减为无符号短整数，隐含的偏移量为0，缩放除数为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_16_06_81e2c04de949a7.png)。纹理坐标通常成对出现，恰好可以存进两个无符号短整数 [1381]；根据精度要求，甚至只需要3字节 [88]。

与其他坐标集合不同，法线通常经过归一化，因此所有归一化法线的集合构成一个球面。正因如此，研究者研究了从球面到平面的变换，以便高效压缩法线。Cigolle等人 [269]分析了各种算法的优点与权衡，并提供了代码示例。他们得出的结论是，八分体投影和球面投影最为实用，既能尽量减小误差，又能高效编码和解码。Pranckevičius [1432]和Pesce [1394]讨论了为延迟着色（第20.1节）生成G缓冲区时的法线压缩。

其他数据也可能具有可用来减少存储的性质。例如，法线向量、切线向量和副切线向量通常用于法线映射。当这三个向量两两垂直（没有倾斜），且坐标系的手性一致时，就可以只存储其中两个向量，再用叉积推导出第三个。还有更紧凑的办法：只用一个4字节四元数，其中将一个手性位与7位的w分量一起保存，就能够表示这组基所形成的旋转矩阵 [494, 1114, 1154, 1381, 1639]。为了获得更高精度，可以省略四元数四个分量中最大的那个，将另外三个分量各用10位存储。剩下的2位用于指明四个分量中哪个没有存储。由于四元数各分量的平方和为1，因此可以从另外三个分量推导出第四个 [498]。Doghramachi等人 [363]采用一种存储旋转轴和旋转角度的切线／副切线／法线方案。它同样占用4字节，但与四元数存储相比，解码时所需的着色器指令大约只有一半。

图16.23汇总了一些固定码率压缩方法。


## 延伸阅读与资源

来源：《Real-Time Rendering》第4版，第16章章末，书页716（PDF物理页737）。原书本标题未编号；本文件名中的16.99仅用于章末排序。

Meshlab是一个开源的网格可视化与操作系统，实现了大量算法，包括网格清理、法线推导和简化。Assimp是一个开源库，能够读写种类繁多的三维文件格式。更多软件推荐可参阅本书网站realtimerendering.com。

Schneider和Eberly [1574]介绍了各种有关多边形和三角形的算法，并附有伪代码。

Luebke的实用综述 [1091]虽然年代较早，但仍然是简化算法的一份良好入门资料。《三维图形的细节层次》（Level of Detail for 3D Graphics）[1092]一书深入介绍了简化及相关主题。
