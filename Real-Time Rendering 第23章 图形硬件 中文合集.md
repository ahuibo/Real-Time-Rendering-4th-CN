# Real-Time Rendering 第23章 图形硬件 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [23.1 光栅化](<Real-Time_Rendering_4th_中文/第23章/23.01.md>)

- [23.2 大规模计算与调度](<Real-Time_Rendering_4th_中文/第23章/23.02.md>)

- [23.3 延迟与占用率](<Real-Time_Rendering_4th_中文/第23章/23.03.md>)

- [23.4 内存架构与总线](<Real-Time_Rendering_4th_中文/第23章/23.04.md>)

- [23.5 缓存与压缩](<Real-Time_Rendering_4th_中文/第23章/23.05.md>)

- [23.6 颜色缓冲](<Real-Time_Rendering_4th_中文/第23章/23.06.md>)

- [23.7 深度剔除、测试与缓冲](<Real-Time_Rendering_4th_中文/第23章/23.07.md>)

- [23.8 纹理处理](<Real-Time_Rendering_4th_中文/第23章/23.08.md>)

- [23.9 架构](<Real-Time_Rendering_4th_中文/第23章/23.09.md>)

- [23.10 案例研究](<Real-Time_Rendering_4th_中文/第23章/23.10.md>)

- [23.11 光线追踪架构](<Real-Time_Rendering_4th_中文/第23章/23.11.md>)


## 章首导言

来源：原书第993页（PDF第1014页），第23章章首标题、题辞及23.1节之前的导言。

> “等我们拿到最终版硬件，性能就会一飞冲天。”
>
> ——J. Allard

尽管图形硬件正在飞速发展，其设计仍普遍采用一些通用概念和架构。本章旨在帮助读者理解图形系统中的各种硬件组成部分，以及它们彼此之间的关系。本书其他部分讨论这些硬件如何用于具体算法；这里则从硬件本身出发进行介绍。我们首先说明如何对线段和三角形进行光栅化，随后介绍GPU的大规模计算能力如何运作、任务如何调度，其中包括延迟和占用率的处理。接着，我们讨论内存系统、缓存、压缩、颜色缓冲，以及GPU中与深度系统有关的各个方面。之后介绍纹理系统的细节，再用一节讨论GPU的架构类型。23.10节给出三种不同架构的案例分析，最后简要讨论光线追踪架构。


## 23.1 光栅化

来源：原书第993—1002页（PDF第1014—1023页）；从23.1节标题起，到23.2节标题前止，包含23.1.1与23.1.2。章首题辞及导言另存于23.00_章首导言.md。

任何GPU的一项重要特性，都是绘制三角形和线段的速度。如2.4节所述，光栅化由三角形设置和三角形遍历组成。此外，我们还将介绍如何在三角形上插值属性，这与三角形遍历密切相关。最后介绍保守光栅化，它是标准光栅化的一种扩展。

回顾一下，像素中心为(x + 0.5, y + 0.5)，其中x ∈ [0, W − 1]和y ∈ [0, H − 1]均为整数，W × H是屏幕分辨率，例如3840 × 2160。将未经变换的顶点记为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3ad693c18421f2.png)，i ∈ {0, 1, 2}；将经过变换后的顶点记为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0f9b3811916657.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d362ce63009f47.png)，这些变换包括投影，但不包括除以w。二维屏幕空间坐标于是为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_9b6925c7679a3d.png)，即先用w分量进行透视除法，再通过缩放和平移使数值对应屏幕分辨率。图23.1展示了这一设置。


![图23.1 屏幕空间三角形与辅助像素](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.1.png)

**图23.1** 一个三角形，其三个二维顶点![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_19aa1dc33bda1e.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3a55303a18d915.png)位于屏幕空间中。屏幕大小为16 × 8像素。注意，像素(x, y)的中心是(x + 0.5, y + 0.5)。底边的法向量以红色显示，长度缩放为原来的0.25。只有绿色像素位于三角形内部。黄色的辅助像素属于某个四像素组（2 × 2像素），组中至少有一个像素被判定为在三角形内部，而辅助像素的采样点（中心）位于三角形外部。辅助像素用于通过有限差分计算导数。

如图所示，像素网格被划分为2 × 2像素的小组，称为四像素组（quad）。为能够计算纹理细节层次所需的导数（23.8节），只要四像素组中至少有一个像素位于三角形内部，就要对该组全部像素进行像素着色（3.8节也讨论过这一点）。这是绝大多数GPU——甚至可能是所有GPU——的一项核心设计，会影响后面的许多阶段。三角形越小，辅助像素相对于三角形内部像素的比例就越高。这意味着，在执行像素着色时，小三角形的代价相对于其面积而言很高。最糟糕的情况是三角形仅覆盖一个像素，此时需要三个辅助像素。辅助像素的数量有时称为四像素组过度着色（quad overshading）。

为了判断像素中心或其他任意采样位置是否位于三角形内部，硬件为三角形的每条边使用一个边函数[1417]。这些函数基于直线方程，即


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_282370e4915ed4.png)


其中n是与边正交的向量，有时称为边的法向量；p是直线上的一点。这类方程可以改写为ax + by + c = 0。下面推导经过![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_19aa1dc33bda1e.png)的边函数![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_f92bdb6d943d5e.png)(x, y)。边向量为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_19aa1dc33bda1e.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)，因此法向量就是将该边逆时针旋转90度得到的向量，即![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_89e22e331ef7d0.png)，它指向三角形内部，如图23.1所示。将![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)代入式（23.1），得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_8987757588f5c1.png)


对于恰好位于边上的点(x, y)，有e(x, y) = 0。法向量指向三角形内部，意味着对于位于法向量所指一侧的点，有e(x, y) > 0。这条边把空间分成两部分，e(x, y) > 0有时称为正半空间，e(x, y) < 0称为负半空间。利用这些性质就能判断某一点是否在三角形内。将三角形的各条边记为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_90aed5cf0f1430.png)，i ∈ {0, 1, 2}。若采样点(x, y)位于三角形内部或边上，那么对所有i都必须满足![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_90aed5cf0f1430.png)(x, y) ≥ 0。

图形API规范通常要求，将屏幕空间中浮点表示的顶点坐标转换成定点坐标。强制这样做，是为了以一致的方式定义边界归属规则（稍后介绍），同时也可以提高采样点内部测试的效率。例如，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_453340eada2895.png)的x和y坐标都可以按1.14.8位格式存储，即1个符号位、14个整数坐标位，以及8个表示像素内部小数位置的位。在这种情况下，像素内部沿x和y两个方向分别可以有![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0f0dcd0fa3153b.png)个位置，整数坐标必须落在[−(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_ac21ee6291b151.png) − 1), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_ac21ee6291b151.png) − 1]范围内。实际处理中，这种位置吸附会在计算边方程之前进行。

边函数的另一个重要特性是增量性质。假设已经在某个像素中心(x, y) = (![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_fb3dbd2822545a.png) + 0.5, ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_62bb131a10625f.png) + 0.5)计算了边函数，其中(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_fb3dbd2822545a.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_62bb131a10625f.png))是整数像素坐标，也就是说，已经求得e(x, y) = ax + by + c。例如，要计算右侧像素的值，就需要求e(x + 1, y)，它可改写为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_ee949b3ed2dd25.png)


也就是说，只需把当前像素的边函数值e(x, y)加上a即可。y方向也可以采用类似的推理。利用这些性质，可以快速计算一小块像素（例如8 × 8像素）的三个边方程，从而“盖印”生成覆盖掩码，其中每个像素对应一位，表示它是否位于内部。本节稍后会介绍这种层次遍历。

必须考虑边或顶点恰好穿过像素中心时会发生什么。例如，假设两个三角形共享一条边，而这条边经过某个像素中心。该像素应属于第一个三角形、第二个三角形，还是同时属于两者？从效率角度看，同时属于两者是错误的答案，因为一个三角形会先写入该像素，随后另一个三角形又将其覆盖。为此，通常使用一种边界归属规则；这里介绍DirectX采用的左上规则。对所有i ∈ {0, 1, 2}都满足![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_90aed5cf0f1430.png)(x, y) > 0的像素，始终被视为位于内部。当边穿过像素时，左上规则便开始发挥作用。如果像素中心位于一条顶边或左边上，那么该像素被视为在内部。若一条边是水平的，且其余边都位于它下方，它就是顶边。若一条边不是水平的，且位于三角形左侧，它就是左边；这意味着一个三角形最多可以有两条左边。检测一条边是顶边还是左边很简单：顶边满足a = 0（水平）且b < 0，左边满足a > 0。判断采样点(x, y)是否位于三角形内部的整个测试，有时称为内部测试（inside test）。


![图23.2 瓦片与边的半空间测试](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.2.png)

**图23.2** 边函数的负半空间e(x, y) < 0始终被视为三角形外部。这里将一个4 × 4像素瓦片的各个角点投影到边的法向量上。只需对带有黑色圆点的角进行这条边的测试，因为该角在n上的投影最大。由此便可判定整个瓦片位于三角形外部。

我们还没有说明如何遍历线段。通常，可以把线段绘制为一个细长、宽度为一个像素的矩形；这个矩形既可由两个三角形组成，也可以通过增加一个边方程来表示。这种设计的优点是，处理边方程的同一套硬件也能用于线段。点则绘制为四边形。

为了提高效率，通常以层次方式进行三角形遍历[1162]。硬件一般先计算屏幕空间顶点的包围盒，再确定哪些瓦片位于包围盒内并且与三角形重叠。判断一个瓦片是否位于某条边外侧，可以使用22.10.1节AABB/平面测试的二维版本。图23.2展示了基本原理。要将其用于基于瓦片的三角形遍历，可以在遍历开始之前，先确定对一条边应测试瓦片的哪个角[24]。对于一条给定的边，所有瓦片都使用同一种角，因为最近的瓦片角仅取决于边的法向量。在这些预先选定的角点处求边方程的值；如果选定的角位于相应边的外侧，则整个瓦片都在外部，硬件便不必对其中的像素逐一执行内部测试。移至相邻瓦片时，可以在瓦片级别应用上述增量性质。例如，水平向右移动8个像素，只需加上8a。

有了瓦片/边相交测试，就可以按层次遍历三角形，如图23.3所示。瓦片本身也需要按一定顺序遍历，可以采用之字形顺序，也可以采用某种空间填充曲线[1159]；这两种方法通常都能增强访问的连贯性。如有需要，还可以增加层次遍历的层数。例如，可以先访问16 × 16的瓦片，再对每个与三角形重叠的瓦片测试其4 × 4子瓦片[1599]。


![图23.3 瓦片遍历顺序](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.3.png)

**图23.3** 使用4 × 4像素瓦片进行遍历时的一种可能顺序。本例从左上角开始，向右继续。上方各瓦片均与三角形重叠，尽管右上瓦片中没有位于三角形内部的像素。遍历随后进入正下方的瓦片，它完全位于外部，因此不需要逐像素执行内部测试。然后向左继续遍历，接下来的两个瓦片被发现与三角形重叠，而左下瓦片则不重叠。

与按扫描线顺序遍历三角形相比，瓦片遍历的主要优点是以更连贯的方式处理像素，相应地也以更连贯的方式访问纹素。它还有一个好处：访问颜色缓冲和深度缓冲时，能更好地利用局部性。例如，考虑按扫描线顺序遍历一个大三角形的情况。纹素会被缓存，最近访问的纹素保留在缓存中以供复用。假设纹理映射使用mipmap，这会增加缓存中纹素的复用程度。如果按扫描线顺序访问像素，那么到达扫描线末端时，扫描线起点用到的纹素很可能早已被逐出缓存。复用缓存中的纹素比反复从内存获取更高效，因此三角形通常按瓦片遍历[651, 1162]。这对纹理映射[651]、深度缓冲[679]和颜色缓冲[1463]都有很大益处。事实上，纹理以及深度缓冲和颜色缓冲也采用瓦片形式存储，原因大体相同。23.4节将进一步讨论这一点。

在开始遍历三角形之前，GPU通常有一个三角形设置阶段。这一阶段的目的，是计算整个三角形上保持不变的因子，以便高效遍历。例如，三角形各边方程（式（23.2））中的常数![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_91de03b8b83f7a.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_8fa8d9657c94c7.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_8c71e236159b58.png)，i ∈ {0, 1, 2}，在此只计算一次，随后用于当前三角形的整个遍历过程。三角形设置还负责计算与属性插值相关的常数（23.1.1节）。随着讨论继续，我们还会看到其他能够在三角形设置阶段一次性计算的常数。

裁剪必须在三角形设置之前进行，因为裁剪可能产生更多三角形。在裁剪空间中，针对视景体裁剪三角形的过程代价很高，因此GPU会尽可能避免，除非确有必要。近裁剪平面的裁剪始终是必需的，可能生成一个或两个三角形。对于屏幕边界，多数GPU采用保护带裁剪（guard-band clipping），这是一种更简单的方案，可以避免更复杂的完整裁剪过程。图23.4直观展示了该算法。


![图23.4 保护带裁剪](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.4.png)

**图23.4** 通过保护带尽量避免完整裁剪。假设保护带区域在x和y两个方向上都是±16K像素，那么中间的屏幕大约为6500 × 4900像素，这说明图中的三角形非常巨大。底部两个绿色三角形在三角形设置阶段或更早的步骤中被剔除。最常见的情况是中间的蓝色三角形：它与屏幕区域相交，并完全位于保护带内部。由于只处理可见瓦片，因此不需要执行完整裁剪。红色三角形超出了保护带，又与屏幕区域相交，因而需要裁剪。注意，右侧红色三角形被裁剪成两个三角形。

### 23.1.1 插值

在22.8.1节中，重心坐标是计算射线与三角形交点时得到的副产物。任意逐顶点属性![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_91de03b8b83f7a.png)，i ∈ {0, 1, 2}，都可以使用重心坐标(u, v)进行插值：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_a7631d0bcd2cad.png)


其中a(u, v)是三角形上(u, v)位置处的插值属性。重心坐标的定义为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_48f189f650e1fd.png)


其中![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_db175bc6cb7648.png)是图23.5左图中各子三角形的面积。第三个坐标w = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_1be24fe3c2e8cc.png)/(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_1be24fe3c2e8cc.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_cc5c8879bf5964.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3a2cec4976a7a9.png))也是定义的一部分，因此u + v + w = 1，也就是w = 1 − u − v。这里用1 − u − v代替w。

式（23.2）中的边方程可以用边的法向量![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png) = (![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_706dcf1b6e7fee.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_5e1d8fad66594b.png))表示为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_9c775bbb02b3b8.png)


![图23.5 重心坐标与子三角形面积](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.5.png)

**图23.5** 左：顶点带有标量属性(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0e21fdcc41c3d1.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_a323819885cd5b.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_706dcf1b6e7fee.png))的三角形。点p处的重心坐标与有向面积(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_cc5c8879bf5964.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3a2cec4976a7a9.png), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_1be24fe3c2e8cc.png))成比例。中：重心坐标(u, v)在三角形上的变化示意。右：法向量![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)由边![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_19aa1dc33bda1e.png)逆时针旋转90度得到，其长度与该边相同。因此面积![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3a2cec4976a7a9.png)为bh/2。

其中p = (x, y)。根据点积的定义，可将其改写为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_b830b9b576161e.png)


其中α是![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)与p − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)之间的夹角。注意，b = ‖![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)‖等于边![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_19aa1dc33bda1e.png)的长度，因为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)就是将该边旋转90度得到的。第二项‖p − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)‖cos α的几何意义，是把p − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)投影到![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)上所得向量的长度；该长度恰好等于面积为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3a2cec4976a7a9.png)的子三角形的高h，如图23.5右图所示。于是值得注意的是，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_f92bdb6d943d5e.png)(p) = ‖![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_d13080baeee9f5.png)‖‖p − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c11de44d831b47.png)‖cos α = bh = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_1f9bfc1eb6fdbf.png)。这非常有用，因为计算重心坐标正需要子三角形的面积。这意味着


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_c036c4102adfcf.png)


三角形设置阶段通常会计算1/(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_1be24fe3c2e8cc.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_cc5c8879bf5964.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3a2cec4976a7a9.png))，因为三角形的面积不变，而且这样还可以避免逐像素执行除法。因此，当我们使用边方程遍历三角形时，式（23.8）的所有项都会作为内部测试的副产物得到。正如稍后将看到的，它们可用于深度插值，也适用于正交投影；但对于透视投影，重心坐标并不能产生期望结果，如图23.6所示。

透视校正重心坐标需要对每个像素做一次除法[163, 694]。这里省略推导[26, 1317]，只总结最重要的结果。由于线性插值的成本低，而且已经知道如何计算(u, v)，因此即使在透视校正中，我们也希望尽可能使用屏幕空间中的线性插值。有些令人意外的是，在三角形上对a/w和1/w都可以进行线性插值，其中w是顶点完成所有变换后的第四个分量。恢复插值属性a，只需使用这两个插值结果：


![图23.6 透视投影与透视校正](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.6.png)

**图23.6** 左：在透视投影中，几何体的投影图像随距离增加而缩小。中：从侧面观察三角形的投影。注意，三角形上半部分在投影平面上覆盖的区域小于下半部分。右：带棋盘格纹理的四边形。上图使用重心坐标进行纹理映射，下图使用透视校正重心坐标。


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_98711512ee5e79.png)


这就是前面提到的逐像素除法。

一个具体例子可以展示其效果。假设沿三角形的一条水平边插值，左端![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0e21fdcc41c3d1.png) = 4，右端![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_a323819885cd5b.png) = 6。两端点之间的中点处，属性值是多少？对于正交投影（或端点的w值相同时），答案就是a = 5，即![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0e21fdcc41c3d1.png)与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_a323819885cd5b.png)的中间值。

现在假设端点的w值分别为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) = 1和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) = 3。这时需要插值两次，分别得到a/w和1/w。对于a/w，左端值为4/1 = 4，右端值为6/3 = 2，所以中点值为3。对于1/w，两个端点分别为1/1和1/3，因此中点值为2/3。用3除以2/3，得到透视情况下中点处的属性值a = 4.5。

实践中，通常需要在三角形上对多个属性进行透视校正插值。因此，常见做法是计算透视校正重心坐标，记为(ũ, ṽ)，然后用它们进行所有属性插值。为此，引入以下辅助函数[26]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_7883e0c78b05a4.png)


注意，由于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_43270b023aa067.png)(x, y) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0e21fdcc41c3d1.png)x + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_ad0d7a1ea0a9ba.png)y + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0988a1f5693327.png)，三角形设置阶段可以计算并存储![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_0e21fdcc41c3d1.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png)以及其他类似项，以加快逐像素求值。另一种方法是将所有![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_9d4f0c1b7b3252.png)函数乘以![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png)；例如，存储![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_efa168eaae1490.png)(x, y)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_49f2bf766aba7e.png)(x, y)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_59ff3812b9d05a.png)(x, y)[1159]。

> 译注：上句示例按原书第1001页保留，存在记号疑点。按照式（23.10），若所有![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_9d4f0c1b7b3252.png)确实统一乘以![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png)，结果应分别为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_43270b023aa067.png)(x, y)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_392783cc92c86c.png)(x, y)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_f92bdb6d943d5e.png)(x, y)。原文示例却仍写![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_efa168eaae1490.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_49f2bf766aba7e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_59ff3812b9d05a.png)，和“统一乘以![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e28dbeac91a99.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_50251dbe8a1da7.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_2e3e884a113385.png)”的叙述不一致。

透视校正重心坐标为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_1eb838dd70a763.png)


每个像素只需计算一次这些坐标，之后便可用它们插值任意属性，同时获得正确的透视缩短效果。注意，这些坐标不像(u, v)那样与子三角形面积成比例。另外，分母也不像普通重心坐标的分母那样是常数，这就是必须逐像素执行该除法的原因。

最后，注意深度是z/w。从式（23.10）可知，不应使用那些方程来插值深度，因为深度已经除以w。因此，应对每个顶点计算![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_23b590f1ff977e.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_01_3e929edc96716e.png)，然后使用(u, v)进行线性插值。这有若干优点，例如有利于深度缓冲压缩（23.7节）。

### 23.1.2 保守光栅化

从DirectX 11开始，以及通过OpenGL的扩展，可以使用一种新的三角形遍历方法，称为保守光栅化（conservative rasterization，CR）。CR有两种类型，分别称为过估计CR（OCR）和低估计CR（UCR），有时也称为外保守光栅化和内保守光栅化。图23.7展示了这两种方式。

大致来说，OCR会访问所有与三角形重叠或位于其内部的像素，而UCR只访问完全位于三角形内部的像素。通过把瓦片大小缩小为单个像素，OCR和UCR都可以使用瓦片遍历来实现[24]。如果硬件不提供支持，可以使用几何着色器或三角形扩张来实现OCR[676]。关于CR的更多信息，请参阅各API的规范。CR可用于图像空间碰撞检测、遮挡剔除、阴影计算[1930]、抗锯齿等算法。


![图23.7 保守光栅化](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_1_23.7.png)

**图23.7** 三角形的保守光栅化。使用外保守光栅化时，所有着色像素都属于三角形。使用标准光栅化时，黄色和绿色像素位于三角形内部；使用内保守光栅化时，则只生成绿色像素。

最后需要指出，所有类型的光栅化都是几何处理与像素处理之间的桥梁。为了计算三角形顶点的最终位置和像素的最终颜色，GPU需要大量灵活的计算能力。下一节将对此进行说明。


## 23.2 大规模计算与调度

来源：原书第 1002—1004 页（PDF 第 1023—1025 页），从第 23.2 节标题起，至第 23.3 节标题前。

为了提供可用于任意计算的庞大算力，绝大多数 GPU 架构（即便不是全部）都采用统一着色器架构，使用具有多线程的 SIMD 处理，这有时也称为 SIMT 处理或超线程。有关线程、SIMD 处理、线程束（warp）和线程组等术语，可参阅第 3.10 节复习。请注意，我们采用的是 NVIDIA 的术语 warp，而在 AMD 硬件上，它们称为 wave 或 wavefront。本节首先考察 GPU 中使用的一种典型的统一算术逻辑单元（arithmetic logic unit，ALU）。

在这里，ALU 是一种经过优化、用于为单个实体（例如一个顶点或片元）执行程序的硬件。我们有时会用 SIMD 通道（SIMD lane）这一术语代替 ALU。图 23.8 左侧展示了一个典型的 GPU ALU。其主要计算单元是一个浮点（FP）单元和一个整数单元。浮点单元通常遵循 IEEE 754 浮点标准，并支持融合乘加（fused-multiply and add，FMA）指令，这是其最复杂的指令之一。除了余弦、正弦和指数等超越运算外，ALU 通常还具有移动／比较、加载／存储功能，以及一个分支单元。不过，应当注意，在某些架构中，其中一些功能可能位于独立的硬件单元中。例如，一小组超越函数硬件单元可能为数量更多的 ALU 提供服务。对于执行频率相对较低的运算，就可能采用这种方式。这些单元归入图 23.8 右侧所示的特殊单元（SU）模块。ALU 架构通常由几个硬件流水线阶段构成，也就是说，硅片上实际构建了多个并行执行的模块。例如，在当前指令进行乘法运算时，下一条指令可以读取寄存器。若有 n 个流水线阶段，理想情况下，吞吐量可提高到原来的 n 倍。这通常称为流水线并行。采用流水线的另一个重要原因是：在流水线处理器中，最慢的硬件模块决定了该模块能够运行的最高时钟频率。增加流水线阶段的数量，会减少每个流水线阶段包含的硬件模块数，通常就能够提高时钟频率。不过，为了简化设计，ALU 的流水线阶段一般较少，例如 4—10 级。


![图 23.8 算术逻辑单元与多处理器结构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_2_23.8.png)

图 23.8。左：一种为每次执行一个项目而构建的算术逻辑单元示例。分派端口接收当前待执行指令的信息，操作数收集器读取该指令所需的寄存器。右：这里将 8 × 4 个 ALU 与其他若干硬件单元组装在一起，形成一个称为多处理器的模块。这 32 个 ALU 有时称为 SIMD 通道，它们以锁步方式执行相同的程序，也就是说，它们构成了一个 SIMD 引擎。此外，还有寄存器文件、L1 缓存、局部数据存储、纹理单元，以及用于处理 ALU 无法处理的各种指令的特殊单元。

统一 ALU 与 CPU 核心不同，它没有分支预测、寄存器重命名和深度指令流水线等许多复杂的附加机制。芯片面积主要用于复制 ALU 以提供庞大算力，以及增大寄存器文件，以便在线程束之间切换。例如，NVIDIA GTX 1080 Ti 拥有 3584 个 ALU。为了高效调度提交给 GPU 的工作，大多数 GPU 会将 ALU 按一定数量（例如 32 个）分组。它们以锁步方式执行，这意味着整组 32 个 ALU 就是一个 SIMD 引擎。对于这样一组 ALU 连同附加硬件单元构成的整体，不同厂商使用不同名称；我们采用通用名称多处理器（multiprocessor，MP）。例如，NVIDIA 使用流式多处理器（streaming multiprocessor）这一术语，Intel 使用执行单元（execution unit），AMD 则使用计算单元（compute unit）。图 23.8 右侧展示了一个 MP 示例。MP 通常包含一个向 SIMD 引擎分派工作的调度器，以及 L1 缓存、局部数据存储（LDS）、纹理单元（TX）和一个用于处理不在 ALU 中执行的指令的特殊单元。MP 将指令分派到 ALU 上，指令在那里以锁步方式执行，也就是 SIMD 处理（第 3.10 节）。请注意，MP 的具体组成随厂商和架构代际而异。

SIMD 处理适合图形工作负载，因为其中有大量同类对象（例如顶点和片元）执行相同的程序。在这里，架构利用了线程级并行，也就是说，顶点和片元等对象可以独立于其他顶点和片元执行自己的着色器。此外，任何类型的 SIMD／SIMT 处理都会利用数据级并行，因为一条指令会在 SIMD 机器的所有通道上执行。还有一种指令级并行：如果处理器能够找到彼此独立的指令，并且具有可并行执行的资源，就可以同时执行这些指令。

MP 附近有一个（线程束）调度器，它接收要在该 MP 上执行的大块工作。线程束调度器的任务是以线程束为单位将工作分配给 MP，在寄存器文件（RF）中为线程束内的线程分配寄存器，然后以尽可能好的方式确定工作的优先级。通常，下游工作的优先级高于上游工作。例如，像素着色位于可编程阶段的末端，其优先级高于流水线中较早的顶点着色。这可以避免停顿，因为靠后的阶段就不太容易阻塞前面的阶段。图形流水线示意图可参阅第 34 页的图 3.2。一个 MP 能够处理数百乃至数千个线程，以隐藏内存访问等操作的延迟。调度器可以将当前正在 MP 上执行（或等待）的线程束换出，换入一个已准备好执行的线程束。由于调度器由专用硬件实现，这种切换通常可以做到零开销 [1050]。例如，如果当前线程束执行一条预期延迟较长的纹理加载指令，调度器就可以立即换出当前线程束，换入另一个线程束，并继续执行后者。这样，计算单元便能得到更充分的利用。

请注意，对于像素着色工作，线程束调度器会分派若干完整的四像素块（quad），因为为了计算导数，像素以四像素块为粒度进行着色。第 23.1 节曾提及这一点，第 23.8 节还会进一步讨论。因此，如果一个线程束的大小为 32，就能调度 32 / 4 = 8 个四像素块执行。这里存在一种架构设计上的选择：可以让整个线程束固定属于同一个三角形，也可以允许一个线程束中的每个四像素块属于不同的三角形。前者实现较简单，但对于较小的三角形，效率会降低。后者更复杂，但处理较小三角形时效率更高。

一般来说，为了在芯片上获得更高的计算密度，MP 也会被复制，因此 GPU 通常还具有更高层级的调度器。它的任务是根据提交给 GPU 的工作，将工作分配给不同的线程束调度器。一个线程束中包含许多线程，通常也意味着一个线程的工作需要独立于其他线程的工作。当然，图形处理中常常满足这一条件。例如，对一个顶点进行着色通常不依赖其他顶点，一个片元的颜色一般也不依赖其他片元。

请注意，不同架构之间存在许多差异。第 23.10 节将通过若干不同的案例研究，着重介绍其中一些差异。到这里，我们已经了解了光栅化的实现方式，以及如何利用大量复制的统一 ALU 来进行着色计算。剩下的一大部分内容是内存系统、所有相关缓冲区以及纹理处理。这些是从第 23.4 节开始的后续各节的主题，不过在此之前，我们先进一步介绍延迟与占用率。


## 23.3 延迟与占用率

来源：《Real-Time Rendering》第 4 版，书页 1004—1006（PDF 物理页 1025—1027）。本节从“23.3 Latency and Occupancy”标题开始，到“23.4 Memory Architecture and Buses”标题前结束。

一般来说，**延迟**是从发出查询到收到结果之间的时间。例如，可以请求内存中某个地址处的值，从发出查询到获得结果所需的时间就是延迟。另一个例子是向纹理单元请求经过过滤的颜色；从发出请求到该值可用，可能需要数百个、甚至数千个时钟周期。为了高效利用 GPU 中的计算资源，必须隐藏这种延迟。如果不隐藏这些延迟，内存访问很容易就会占据执行时间的主要部分。

隐藏延迟的一种机制是 SIMD 处理中的多线程部分，书页 33 的图 3.1 对此作了说明。一般来说，一个多处理器（MP）能够处理的线程束（warp）数量有一个上限。**活动线程束**的数量取决于寄存器的使用情况，也可能取决于纹理采样器、L1 缓存、插值量的使用情况以及其他因素。这里，我们将**占用率** o 定义为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_7ce8d8481efcf2.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_b03010fd30ad73.png) 是一个 MP 上允许的最大线程束数，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_ca135ada1a14a5.png) 是当前活动线程束的数量。也就是说，o 衡量的是计算资源保持被利用的程度。例如，假设 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_494c3f11304823.png)，一个着色处理器具有 256 kB 的寄存器存储空间，某个着色器程序的单个线程使用 27 个 32 位浮点寄存器，另一个着色器程序则使用 150 个。此外，我们假设决定活动线程束数量的因素是寄存器使用量。假设 SIMD 宽度为 32，那么这两种情况下的活动线程束数分别可以计算为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_3b4203a23faa23.png)


在第一种情况下，也就是使用 27 个寄存器的短程序，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_30bb84ce01d6a5.png)，因此占用率 o = 1。这是理想情况，因而有利于隐藏延迟。然而，在第二种情况下，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_03_0bb3950e1c8f5d.png)，所以 o ≈ 13.65/32 ≈ 0.43。由于活动线程束较少，占用率就更低，这可能不利于隐藏延迟。因此，在设计架构时，在线程束数量上限、寄存器数量上限以及其他共享资源之间取得良好平衡十分重要。

有时候，过高的占用率可能适得其反：如果着色器进行了大量内存访问，就可能导致缓存抖动 [1914]。另一种隐藏延迟的机制，是在发出内存请求后继续执行同一个线程束；只要存在不依赖该内存访问结果的指令，就可以这样做。虽然这会使用更多寄存器，但有时保持较低占用率反而可能更高效 [1914]。循环展开就是一个例子：它通常会生成更长的独立指令链，为指令级并行提供更多机会，从而能够在切换线程束之前持续执行更长时间。不过，这也会使用更多临时寄存器。一般原则仍然是争取更高的占用率。例如，着色器请求纹理访问时，占用率低就意味着能够切换到另一个线程束的可能性较小。

另一种延迟出现在从 GPU 向 CPU 回读数据时。一个很好的理解方式是，把 GPU 和 CPU 看成两台异步工作的独立计算机，而两者之间的通信需要付出一定代价。改变信息流动方向所产生的延迟可能严重损害性能。从 GPU 回读数据时，可能必须先排空流水线，再进行读取。在此期间，CPU 会等待 GPU 完成工作。对于 Intel 的 GEN 架构 [844] 这类 GPU 与 CPU 位于同一芯片、并采用共享内存模型的架构，这种延迟会大幅降低。较低层级的缓存由 CPU 和 GPU 共享，而较高层级的缓存则不共享。共享缓存带来的延迟降低，使得不同类型的优化及其他种类的算法成为可能。例如，这一特性已被用于加速光线追踪：光线在图形处理器与 CPU 核心之间来回传递，而不产生开销 [110]。

遮挡查询就是一种不会导致 CPU 停顿的回读机制，参见第 19.7.1 节。在遮挡测试中，其机制是先执行查询，然后不时检查 GPU，看看查询结果是否已经可用。在等待结果期间，CPU 和 GPU 都可以执行其他工作。

> 译注：式（23.13）及随后占用率的数值按原书保留。这里将寄存器容量相除所得的小数作为容量估算；实际可同时驻留的完整线程束数必须取整数。在本例仅考虑寄存器容量和 32 束上限的简化假设下，两种情况分别最多容纳 32 束和 13 束，后一种对应 13/32 = 0.40625；真实硬件还可能受到寄存器分配粒度等约束。


## 23.4 内存架构与总线

来源：原书第 1006—1007 页（PDF 第 1027—1028 页），从 23.4 节标题起至 23.5 节标题前。

这里，我们将介绍一些术语，讨论几种不同类型的内存架构，然后介绍压缩与缓存。

**端口**（port）是在两个设备之间传送数据的通道，而**总线**（bus）是在两个以上设备之间传送数据的共享通道。**带宽**（bandwidth）用于描述端口或总线上的数据吞吐量，以每秒字节数（B/s）计量。端口和总线在计算机图形学架构中很重要，简单地说，是因为它们将不同的构建模块连接在一起。同样重要的是，带宽是一种稀缺资源，因此在构建图形系统之前，必须进行仔细的设计和分析。由于端口和总线都提供数据传输能力，人们也经常将端口称为总线，这里我们也沿用这一惯例。

对于许多 GPU，在图形加速器上配备 GPU 专用内存是很常见的，这种内存通常称为**显存**（video memory）。访问这种内存，通常比让 GPU 经由总线访问系统内存快得多，例如经由 PC 中使用的 PCI Express（PCIe）总线。16 通道的 PCIe v3 在两个方向上均可提供 15.75 GB/s 的带宽，PCIe v4 则可提供 31.51 GB/s。然而，Pascal 图形架构（GTX 1080）的显存带宽可达 320 GB/s。

传统上，纹理和渲染目标存储在显存中，但显存也可以用来存储其他数据。场景中的许多物体在相邻帧之间并不会发生明显的形状变化。即使是人物角色，通常也是用一组保持不变的网格来渲染，并在关节处使用 GPU 端的顶点混合。对于这种完全通过建模矩阵和顶点着色器程序实现动画的数据，通常使用放置在显存中的**静态**顶点缓冲区和索引缓冲区。这样可以让 GPU 快速访问数据。对于每帧由 CPU 更新的顶点，则使用**动态**顶点缓冲区和索引缓冲区，并将它们放置在可以经由 PCI Express 等总线访问的系统内存中。PCIe 的一个良好特性是，查询可以采用流水线方式处理，因此能够在结果返回之前发出多个查询请求。


![图 23.9：Intel Gen9 片上系统的内存架构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_4_23.9.png)

**图 23.9** Intel 片上系统（SoC）Gen9 图形架构的内存架构简图，该架构与 CPU 核心相连，并采用共享内存模型。注意，末级缓存（LLC）由图形处理器和 CPU 核心共享。（插图依据 Junkins [844] 绘制。）

大多数游戏主机，例如所有 Xbox 主机和 PLAYSTATION 4，都采用**统一内存架构**（unified memory architecture，UMA），这意味着图形加速器可以使用主机内存中的任何部分来存储纹理和各种缓冲区 [889]。CPU 和图形加速器使用同一块内存，因此也使用同一条总线。这显然不同于使用专用显存的方式。Intel 也采用 UMA，使 CPU 核心与 GEN9 图形架构共享内存 [844]，如图 23.9 所示。不过，并非所有缓存都是共享的。图形处理器拥有自己的一组 L1 缓存、L2 缓存，以及一个 L3 缓存。末级缓存是内存层次结构中第一个共享的资源。对于任何计算机架构或图形架构，拥有缓存层次结构都很重要。如果内存访问具有某种局部性，这样做就能降低访问内存的平均时间。下一节将讨论 GPU 的缓存与压缩。


## 23.5 缓存与压缩

来源：原书第 1007—1009 页（PDF 第 1028—1030 页）；从 23.5 节标题起，至 23.6 节标题前。

每个 GPU 的多个不同部位都设有缓存，但不同架构的缓存有所不同，我们将在第 23.10 节看到这一点。一般来说，为架构增加缓存层次结构，是为了利用内存访问模式的局部性，降低内存访问延迟和带宽使用量。也就是说，如果 GPU 访问了某个数据项，那么它很可能很快就会再次访问同一数据项或其附近的数据项 [715]。大多数缓冲区和纹理格式都采用分块格式存储，这也有助于提高局部性 [651]。假设一条缓存行包含 512 位，即 64 字节，而当前使用的颜色格式中，每个像素占 4 B。那么，一种设计选择就是用 64 B 存储一个 4 × 4 区域内的全部像素，这样的区域也称为一个块（tile）。也就是说，整个颜色缓冲区会被划分为多个 4 × 4 块。一个块也可以跨越多条缓存行。


![图 23.10：GPU 渲染目标的缓存后压缩与缓存前压缩硬件框图](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_5_23.10.png)

**图 23.10** GPU 中用于渲染目标压缩与缓存的硬件技术框图。左：缓存后压缩（post-cache compression），压缩器／解压缩器硬件单元位于缓存之后（下方）。右：缓存前压缩（pre-cache compression），压缩器／解压缩器硬件单元位于缓存之前（上方）。

要获得高效的 GPU 架构，就需要从各个方面着手减少带宽使用量。大多数 GPU 都包含硬件单元，可以在运行过程中即时压缩和解压缩渲染目标，例如在图像渲染期间进行这些操作。必须认识到，这类压缩算法是无损的；也就是说，始终能够精确地还原原始数据。这些算法的核心是我们所说的块表（tile table），其中为每个块存储了附加信息。块表可以存储在芯片上，也可以经由内存层次结构通过缓存访问。图 23.10 给出了这两类系统的框图。通常，同样的配置也可用于深度、颜色和模板压缩，有时需要做一些修改。块表中的每个元素都存储着帧缓冲区内一个像素块的状态。每个块的状态可以是已压缩、未压缩或已清除（接下来讨论）。一般来说，还可以有不同类型的压缩块。例如，一种压缩模式可能将数据压缩到原大小的 25%，另一种则压缩到 50%。必须认识到，压缩程度取决于 GPU 能够处理的内存传输大小。假设某种架构的最小内存传输量为 32 B。如果将块大小选为 64 B，那么就只能压缩到原大小的 50%。但是，若块大小为 128 B，就可以压缩到 75%（96 B）、50%（64 B）和 25%（32 B）。

块表还经常用于实现渲染目标的快速清除。当系统发出清除渲染目标的命令时，表中每个块的状态都会被设置为已清除，而帧缓冲区本身不会被改动。当访问渲染目标的硬件单元需要读取已清除的渲染目标时，解压缩器单元首先检查表中的状态，判断该块是否已清除。如果是，就将该渲染目标块放入缓存，并将其中的所有值设置为清除值，而无需读取和解压缩实际的渲染目标数据。这样就能在清除期间尽可能减少对渲染目标本身的访问，从而节省带宽。如果状态不是已清除，就必须读取该块对应的渲染目标。系统会读取块中存储的数据；如果数据已经压缩，则先经过解压缩器，再继续传送。

当访问渲染目标的硬件单元完成新值的写入，并且该块最终从缓存中被逐出时，就会将它送入压缩器，尝试对其进行压缩。如果有两种压缩模式，就可以分别尝试，然后使用能以最少位数压缩该块的那一种。由于 API 要求渲染目标压缩必须无损，因此，如果所有压缩技术都失败，就需要退回到使用未压缩数据的方式。这也意味着，无损渲染目标压缩绝不可能减少实际渲染目标的内存占用；这类技术只能减少内存带宽使用量。如果压缩成功，就将块的状态设置为已压缩，并以压缩形式发送信息。否则，就以未压缩形式发送，并将状态设置为未压缩。

请注意，压缩器和解压缩器单元既可以位于缓存之后（称为缓存后，post-cache），也可以位于缓存之前（缓存前，pre-cache），如图 23.10 所示。缓存前压缩可以显著增大缓存的有效容量，但通常也会增加系统的复杂性 [681]。有专门用于压缩深度 [679, 1238, 1427] 和颜色 [1427, 1463, 1464, 1716] 的算法。后者包括有关有损压缩的研究，不过，据我们所知，当时还没有任何硬件提供这种有损压缩 [1463]。大多数算法会对一个代表块内全部像素的锚定值进行编码，然后以不同方式对相对于该锚定值的差值进行编码。对于深度，常见的做法是存储一组平面方程 [679]，或者使用差分之差技术 [1238]。由于深度在屏幕空间中是线性的，这两种方法都能取得良好的效果。


## 23.6 颜色缓冲

来源：原书书页 1009—1013（PDF 物理页 1030—1034）；另核查 PDF 第 1035 页以确认下一节边界。以下技术陈述保留原书出版时的语境。

使用 GPU 进行渲染需要访问多种不同的缓冲区，例如颜色、深度和模板缓冲区。注意，尽管称为“颜色”缓冲区，但任何类型的数据都可以渲染并存储到其中。

颜色缓冲区通常提供几种颜色模式，其依据是表示颜色所用的字节数。这些模式包括：

- **高彩色（high color）**：每像素 2 字节，其中 15 或 16 位用于颜色，分别提供 32,768 或 65,536 种颜色。
- **真彩色或 RGB 彩色（true color / RGB color）**：每像素 3 或 4 字节，其中 24 位用于颜色，提供 16,777,216 ≈ 1680 万种不同颜色。
- **深彩色（deep color）**：每像素 30、36 或 48 位，提供至少十亿种不同颜色。

高彩色模式有 16 位可用于颜色分辨率。通常，红、绿、蓝三个通道各分配至少 5 位，因此每个颜色通道有 32 个级别。这样还剩下一位，通常分给绿色通道，形成 5-6-5 的划分。之所以选择绿色通道，是因为它对人眼感知亮度的影响最大，因此需要更高的精度。高彩色相对于真彩色和深彩色具有速度优势，因为访问每像素 2 字节的内存通常比访问每像素 3 字节或更多字节更快。话虽如此，如今高彩色模式已经很少使用，甚至几乎不再使用。当每个通道只有 32 或 64 个颜色级别时，相邻颜色级别的差异很容易被辨认出来。这个问题有时称为**色带（banding）**或**色调分离（posterization）**。人类视觉系统还会因一种称为 **马赫带效应（Mach banding）** 的感知现象而进一步放大这些差异 [543, 653]。见图 23.11。**抖动（dithering）** [102, 539, 1081] 通过混合相邻级别，以空间分辨率换取有效颜色分辨率的提高，从而减轻这种现象。即使在 24 位显示器上，渐变中的色带也可能很明显。向帧缓冲图像添加噪声，可以掩盖这个问题 [1823]。


![图 23.11：32 级灰度色带与马赫带错觉](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_6_23.11.png)

**图 23.11**　当矩形从白到黑进行着色时，会出现色带。尽管这 32 个灰度条各自都具有均一的强度级别，但由于马赫带错觉，每一条看起来都可能左侧较暗、右侧较亮。

真彩色使用 24 位 RGB 颜色，每个颜色通道占 1 字节。在 PC 系统中，排列顺序有时反过来，成为 BGR。在内部，这些颜色通常以每像素 32 位存储，因为大多数内存系统都针对访问 4 字节元素进行了优化。在某些系统中，额外的 8 位还可以用于存储 alpha 通道，使像素具有 RGBA 值。24 位颜色（不含 alpha）的表示也称为**紧凑像素格式（packed pixel format）**，与对应的 32 位非紧凑格式相比，可以节省帧缓冲内存。对于实时渲染，使用 24 位颜色几乎总是可以接受的。虽然仍有可能看到颜色色带，但发生的可能性远低于只有 16 位的情况。

深彩色使用 30、36 或 48 位来表示一个 RGB 颜色，即每通道 10、12 或 16 位。如果加入 alpha，这些数字就增加到 40/48/64。HDMI 从 1.3 版本起支持全部 30/36/48 位模式，DisplayPort 标准也支持每通道最高 16 位。

颜色缓冲区通常会按第 23.5 节所述进行压缩和缓存。此外，第 23.10 节的各个案例研究还会进一步说明如何将传入的片元数据与颜色缓冲区混合。混合由 **光栅操作（raster operation，ROP）** 单元处理，而每个 ROP 通常连接到一个内存分区，例如采用一种广义棋盘格模式进行组织 [1160]。接下来，我们将讨论视频显示控制器，它读取颜色缓冲区并使其显示在屏幕上。随后介绍单缓冲、双缓冲和三缓冲。

### 23.6.1 视频显示控制器

每个 GPU 中都有一个**视频显示控制器（video display controller，VDC）**，也称为**显示引擎（display engine）**或**显示接口（display interface）**，负责把颜色缓冲区显示在显示器上。它是 GPU 中的一个硬件单元，可以支持多种接口，例如高清多媒体接口（HDMI）、DisplayPort、数字视频接口（DVI）和视频图形阵列（VGA）。待显示的颜色缓冲区可能位于 CPU 执行任务所使用的同一内存中，也可能位于专用帧缓冲内存或显存中；其中，显存可以包含任意 GPU 数据，但 CPU 无法直接访问。每种接口都使用其标准规定的协议来传输颜色缓冲区的各部分、时序信息，有时甚至包括音频。VDC 还可能执行图像缩放、降噪、多个图像源的合成以及其他功能。

显示器（例如 LCD）更新图像的频率通常为每秒 60 到 144 次（赫兹）。这也称为**垂直刷新率**。当刷新率低于 72 Hz 时，大多数观看者会注意到闪烁。有关这一主题的更多信息，请参见第 12.5 节。

显示器技术已经在刷新率、每分量位数、色域和同步等多个方面取得进步。刷新率过去通常为 60 Hz，但 120 Hz 正变得越来越常见，甚至可以达到 600 Hz。在高刷新率下，通常会多次显示同一幅图像，有时还会插入黑帧，以尽量减少在一帧显示期间眼睛移动造成的拖影伪影 [7, 646]。显示器也可以采用每通道超过 8 位的精度，HDR 显示器可能成为显示技术的下一个重要发展方向。这些显示器可以使用每通道 10 位或更高的精度。杜比拥有一种 HDR 显示技术，利用分辨率较低的 LED 背光阵列来增强其 LCD 显示器。这样，它们的显示器能够达到普通显示器约 10 倍的亮度和 100 倍的对比度 [1596]。具有更宽色域的显示器也越来越常见。通过使纯光谱色成为可表示的颜色，这些显示器能够显示更广的颜色范围，例如更加鲜艳的绿色。有关色域的更多信息，请参见第 8.1.3 节。

为了减少撕裂现象，各公司开发了自适应同步技术，例如 AMD 的 FreeSync 和 NVIDIA 的 G-sync。其思想是让显示器的更新速率适应 GPU 能够生成图像的速率，而不使用预先确定的固定速率。例如，如果一帧需要 10 ms 渲染，下一帧需要 30 ms，那么每幅图像渲染完成后，就立即开始向显示器更新该图像。采用这类技术后，渲染结果看起来会流畅得多。此外，如果图像没有更新，就不必把颜色缓冲区发送给显示器，从而节省功耗。

### 23.6.2 单缓冲、双缓冲和三缓冲

在第 2.4 节中，我们提到，双缓冲能够确保图像直到渲染完成后才显示在屏幕上。这里，我们将介绍单缓冲、双缓冲，乃至三缓冲。

假设我们只有一个缓冲区。这个缓冲区必然就是当前显示在屏幕上的缓冲区。随着一帧中的三角形逐个绘制，显示器每次刷新时都会显现出越来越多的三角形，这样的效果并不令人信服。即使帧率等于显示器的更新速率，单缓冲仍然存在问题。如果我们决定清除缓冲区或绘制一个大三角形，那么，当视频显示控制器传输颜色缓冲区中正在被绘制的区域时，我们就会短暂地看到颜色缓冲区实际发生的局部变化。这种现象有时称为**撕裂（tearing）**，因为显示的图像看起来仿佛被短暂地撕成了两半；这不是实时图形所希望出现的效果。在某些很早期的系统中，例如 Amiga，可以检测扫描束的位置，从而避免在那里绘制，使单缓冲能够正常工作。如今单缓冲已经很少使用，一个可能的例外是虚拟现实系统，其中“与扫描束赛跑（racing the beam）”可以成为降低延迟的一种方式 [6]。

为了避免撕裂问题，通常使用双缓冲。完整的图像显示在**前缓冲区（front buffer）**中，而一个离屏的**后缓冲区（back buffer）**包含当前正在绘制的图像。然后，图形驱动程序交换后缓冲区和前缓冲区；为避免撕裂，这通常发生在整幅图像刚刚传输给显示器之后。交换往往只需交换两个颜色缓冲区指针。对于 CRT 显示器，这个事件称为**垂直回扫（vertical retrace）**，这段时间内的视频信号称为**垂直同步脉冲（vertical synchronization pulse）**，简称 **vsync**。对于 LCD 显示器，虽然没有扫描束的物理回扫，但我们仍使用同一个术语来表示整幅图像刚刚传输到显示器的时刻。在渲染完成后立即交换前后缓冲区，有利于对渲染系统进行基准测试，也被许多应用采用，因为这样可以使帧率最大化。不在垂直同步时更新也会导致撕裂，但由于此时有两幅完整成形的图像，这种伪影不像单缓冲时那么严重。交换之后，新的后缓冲区立即开始接收图形命令，而新的前缓冲区则显示给用户。图 23.12 展示了这一过程。


![图 23.12：单缓冲、双缓冲和三缓冲的状态轮换](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_6_23.12.png)

**图 23.12**　对于单缓冲（上），始终显示前缓冲区。对于双缓冲（中），起初缓冲区 0 在前，缓冲区 1 在后；随后每帧它们都会交换，前者变为后者，后者变为前者。三缓冲（下）通过额外设置一个待处理缓冲区来工作。首先，清除一个缓冲区并开始向其渲染（pending，待处理）；其次，系统继续使用该缓冲区进行渲染，直到图像完成（back，后缓冲）；最后，显示该缓冲区（front，前缓冲）。图中 buffer 0、buffer 1、buffer 2 分别表示缓冲区 0、1、2。

可以给双缓冲增加第二个后缓冲区，我们把它称为**待处理缓冲区（pending buffer）**。这称为**三缓冲（triple buffering）** [1155]。待处理缓冲区与后缓冲区一样，也位于屏幕之外，而且可以在前缓冲区显示期间进行修改。待处理缓冲区成为三缓冲循环的一部分。在某一帧期间，可以访问待处理缓冲区。到下一次交换时，它成为后缓冲区，并在其中完成渲染。然后，它变为前缓冲区并显示给观看者。再下一次交换时，这个缓冲区又变成待处理缓冲区。图 23.12 底部直观展示了这一过程。

相对于双缓冲，三缓冲有一个主要优势：等待垂直回扫时，系统仍可以访问待处理缓冲区。而对于双缓冲，在等待垂直回扫以便进行交换期间，图像构建只能继续等待。原因在于，前缓冲区必须显示给观看者，而后缓冲区也必须保持不变，因为其中已有一幅完整图像，正等待显示。三缓冲的缺点是，延迟最多会增加整整一帧。这会推迟对用户输入的响应，例如按键，以及鼠标或摇杆移动。控制可能感觉迟钝，因为待处理缓冲区开始渲染后，这些用户事件的处理会被推迟。

理论上，可以使用三个以上的缓冲区。如果计算一帧所需的时间变化很大，更多缓冲区可以带来更均衡的运行和总体更高的显示速率，代价是可能增加延迟。更一般地说，可以把多缓冲看作一个环形结构。其中有一个渲染指针和一个显示指针，分别指向不同的缓冲区。渲染指针领先于显示指针，在当前渲染缓冲区计算完成后移动到下一个缓冲区。唯一的规则是，显示指针绝不能与渲染指针指向同一个缓冲区。

对于 PC 图形加速器，一种相关的进一步加速方法是使用 **SLI 模式**。早在 1998 年，3dfx 就使用 SLI 作为**扫描线交错（scanline interleave）**的缩写：两个图形芯片组并行运行，一个处理奇数扫描线，另一个处理偶数扫描线。NVIDIA（收购了 3dfx 的资产）则把这个缩写用于一种完全不同的双显卡（或多显卡）连接方式，称为**可扩展链接接口（scalable link interface）**。AMD 将其称为 CrossFire X。这种并行形式可以通过以下方式划分工作：把屏幕分成两个或更多水平区域，每张显卡负责一个区域；或者让每张显卡完整渲染自己的一帧，交替输出。还有一种模式，允许各张显卡加速同一帧的抗锯齿。最常见的用法是让每个 GPU 渲染单独的一帧，称为**交替帧渲染（alternate frame rendering，AFR）**。虽然这种方案听起来似乎会增加延迟，但它往往几乎不影响延迟，甚至完全没有影响。假设单 GPU 系统以 10 FPS 渲染。如果 GPU 是瓶颈，那么使用 AFR 的两个 GPU 可能以 20 FPS 渲染，四个 GPU 甚至可以达到 40 FPS。每个 GPU 渲染自己那一帧所花的时间相同，因此延迟不一定会改变。

屏幕分辨率持续提高，给基于逐像素采样的渲染器带来了严峻挑战。维持帧率的一种方法，是在屏幕各处 [687, 1805] 以及表面各处 [271] 自适应地改变像素着色速率。


## 23.7 深度剔除、测试与缓冲

来源：《Real-Time Rendering, Fourth Edition》，书页 1014—1016（PDF 物理页 1035—1037）；已核对 PDF 第 1038 页的 23.8 节标题以确认结束边界。图 23.13 在原书中排于本节标题之前，属于本节。

本节将介绍与深度有关的各个方面，包括分辨率、测试、剔除、压缩、缓存、缓冲以及 early-z。

深度分辨率很重要，因为它有助于避免渲染错误。例如，假设你建立了一张纸的模型，并把它放在桌子上，位置仅比桌面略高一点。由于为桌面和纸张计算的 z 深度具有精度限制，桌面可能会在若干位置穿透纸张。这种问题有时称为 **z-fighting（深度冲突）**。注意，如果纸张放置得与桌面完全等高，即纸张与桌面共面，那么在没有关于两者关系的额外信息时，就不存在正确答案。这种问题源于不良建模，无法通过提高 z 精度来解决。

正如第 2.5.2 节所述，z 缓冲（也称为深度缓冲）可用于判定可见性。这类缓冲通常为每个像素（或采样点）分配 24 位或 32 位，可以采用浮点或定点表示 [1472]。对于正交观察，距离值与 z 值成正比，因此得到均匀的分布。然而，对于透视观察，分布是不均匀的，正如书页 99—102 所述。应用透视变换（式 4.74 或式 4.76）后，需要除以 w 分量（式 4.72）。此时深度分量变为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_c9619252000a31.png)，其中 **q** 是与投影矩阵相乘后得到的点。对于定点表示，数值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_c9619252000a31.png) 从其有效范围（例如 DirectX 中的 [0, 1]）映射到整数范围 [0, ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_ca58d1399ab9c7.png) − 1]，并存入 z 缓冲，其中 b 是位数。有关深度精度的更多信息，参见书页 99—102。


![图 23.13 深度流水线的一种可能实现](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_7_23.13.png)

**图 23.13** 深度流水线的一种可能实现，其中 z-interpolate 只是通过插值计算深度值。（插图据 Andersson 等人 [46] 绘制。）

硬件深度流水线如图 23.13 所示。这条流水线的主要目标，是将光栅化图元时产生的每个输入深度与深度缓冲进行测试；如果片元通过深度测试，则可能将输入深度写入深度缓冲。同时，这条流水线还必须高效运行。图的左侧从粗粒度光栅化开始，也就是在图块层面进行光栅化（第 23.1 节）。此时，只有与图元重叠的图块才会被送往下一阶段，即 HiZ 单元，在那里执行 z 剔除技术。HiZ 单元以一个称为粗粒度深度测试的模块开始，这里通常执行两类测试。我们先介绍 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 剔除，它是第 19.7.2 节所介绍的 Greene 分层 z 缓冲算法 [591] 的简化形式。其思想是存储每个图块内全部深度的最大值，称为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png)。图块尺寸取决于体系结构，但常用尺寸为 8 × 8 像素 [1238]。这些 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 值可以存放在固定的片上存储器中，也可以通过缓存访问。在图 23.13 中，我们将其称为 HiZ 缓存。简言之，我们要测试三角形是否在一个图块内被完全遮挡。为此，需要计算三角形在该图块内的最小 z 值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_6c5f73fc6e4890.png)。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_fa82e2f9011dcf.png)，就能保证该三角形在这个图块内被先前渲染的几何体遮挡。于是可以终止该三角形在这个图块内的处理，从而省去逐像素深度测试。注意，这并不会减少任何像素着色器执行，因为无论如何，流水线后续的逐采样点深度测试都会剔除被遮挡的片元。实际上，我们无法承担精确计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_6c5f73fc6e4890.png) 的开销，因此改为计算一个保守估计值。计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_6c5f73fc6e4890.png) 可以采用几种不同的方法，各有优缺点：

1. 可以使用三角形三个顶点中的最小 z 值。这并不总是准确，但额外开销很小。
2. 利用三角形的平面方程，求出图块四个角点处的 z 值，并取其中的最小值。

将这两种策略结合起来，可以获得最佳剔除性能。做法是取这两个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) 值中较大的一个。

另一类粗粒度深度测试是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) 剔除，其思想是存储图块中所有像素的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) [22]。它有两种用途。首先，可以用它避免读取 z 缓冲。如果正在渲染的三角形确定处于先前渲染的全部几何体前方，就没有必要进行逐像素深度测试。在某些情况下，可以完全避免读取 z 缓冲，从而进一步提高性能。其次，它可以用于支持不同类型的深度测试。对于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 剔除方法，我们假定使用标准的“小于”深度测试。不过，如果其他深度测试也能使用剔除，就会很有益；而当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 都可用时，通过这种剔除过程就能支持所有深度测试。Andersson 的博士论文 [49] 给出了对深度流水线更详细的硬件描述。

图 23.13 中的绿色方框涉及更新图块 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) 值的不同方式。如果一个三角形覆盖整个图块，就可以直接在 HiZ 单元中完成更新。否则，需要读取整个图块的逐采样点深度，将其归约为最小值与最大值，再传回 HiZ 单元，这会引入一定延迟。Andersson 等人 [50] 提出了一种方法，无需代价较高的深度缓存反馈便能完成这一过程，同时仍能保留大部分剔除效率。

对于通过粗粒度深度测试的图块，接下来会确定像素或采样点的覆盖情况（使用第 23.1 节所述的边方程），并计算逐采样点深度（图 23.13 中称为 z-interpolate）。这些值被传送到图右侧所示的深度单元。按照 API 的描述，接下来应当执行像素着色器。不过，在下文将介绍的某些情况下，可以在不改变预期行为的前提下，执行一种额外测试，称为 **early-z** [1220, 1542] 或**提前深度测试**。实际上，early-z 就是将逐采样点深度测试放在像素着色器之前执行，并丢弃被遮挡的片元。因此，这一过程可以避免不必要的像素着色器执行。early-z 测试经常与 z 剔除混淆，但二者由完全独立的硬件执行。任何一种技术都可以在不使用另一种技术的情况下单独使用。

在许多情况下，GPU 会自动使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 剔除、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) 剔除和 early-z。然而，例如当像素着色器写入自定义深度、使用 discard 操作，或向无序访问视图写入数值时，就必须禁用这些技术 [50]。如果不能使用 early-z，则在像素着色器之后进行深度测试（称为**延后深度测试**）。

在较新的硬件上，可能可以在着色器中对图像执行原子的读—改—写操作，以及加载和存储操作。在这些情况下，如果你确定这样做是安全的，就可以显式启用 early-z，并覆盖这些限制。另一项可以在像素着色器输出自定义深度时使用的功能，是**保守深度**。在这种情况下，如果程序员保证自定义深度大于三角形深度，就可以启用 early-z。对于这个例子，也可以启用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_66a1b8fb0c4036.png) 剔除，但不能启用 early-z 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_07_64df578b1bedb5.png) 剔除。

> 译注：原书本段先说上述保守深度条件下可以启用 early-z，随后又说不能启用 early-z，前后陈述矛盾。此处按原文保留，未擅自改写。

与往常一样，遮挡剔除会受益于从前向后的渲染顺序。另一种名称和目的都相近的技术是 **z 预通道（z-prepass）**。其思想是，程序员先渲染场景，只写入深度，同时禁用像素着色和向颜色缓冲的写入。在渲染后续通道时，使用“等于”测试；由于 z 缓冲已经初始化，这意味着只有最前面的表面才会被着色。参见第 18.4.5 节。

作为本节的结束，我们简要介绍深度流水线的缓存与压缩，对应图 23.13 的右下部分。总体压缩系统与第 23.5 节介绍的系统类似。每个图块可以被压缩到几个选定大小之一，并且始终存在退回到未压缩数据的方案；当压缩无法达到任何一个选定大小时，就采用这一方案。清空深度缓冲时，使用快速清空来节省带宽消耗。由于深度在屏幕空间中是线性的，典型的压缩算法要么以高精度存储平面方程，要么采用结合增量编码的差分之差技术，要么采用某种锚点方法 [679, 1238, 1427]。图块表和 HiZ 缓存可以完全存放在片上缓冲中，也可以像深度缓存那样，通过存储层次结构的其余部分进行通信。片上存储的代价很高，因为这些缓冲必须足够大，能够处理所支持的最高分辨率。


## 23.8 纹理处理

来源：原书第 1017—1019 页（PDF 第 1038—1040 页）。范围从 23.8 节标题开始，到 23.9 节标题之前，包含移至第 1019 页的图 23.15 及其完整图注。

纹理操作包括读取、过滤和解压缩，当然可以完全用运行于 GPU 多处理器上的软件来实现。不过，已有研究表明，用于纹理处理的固定功能硬件速度可达到这种实现的 40 倍 [1599]。纹理单元执行寻址、过滤、钳制以及纹理格式的解压缩（第 6 章）。它与纹理缓存配合使用，以减少带宽用量。我们首先讨论过滤，以及过滤会给纹理单元带来哪些影响。

为了使用 mipmapping 和各向异性过滤等缩小过滤方法，需要知道纹理坐标相对于屏幕空间的导数。也就是说，要计算纹理的细节层次 λ，需要 ∂u/∂x、∂v/∂x、∂u/∂y 和 ∂v/∂y。这些导数告诉我们，片元代表了纹理区域或纹理函数的多大范围。如果直接使用从顶点着色器传入的纹理坐标来访问纹理，就可以通过解析方法计算导数。如果先用某个函数变换纹理坐标，例如 (u′, v′) = (cos v, sin u)，那么解析计算导数就会变得更复杂。不过，仍然可以使用链式法则或符号微分来完成 [618]。尽管如此，图形硬件并不采用这些方法，因为实际情况可能任意复杂。设想一下，使用环境贴图计算一个表面的反射，而该表面的法线又经过了凹凸映射。例如，反射向量由法线贴图上的反射产生，随后用于访问环境贴图，要以解析方式计算这种反射向量的导数就很困难。因此，通常以四像素组（quad），也就是 2 × 2 个像素为基础，使用 x 和 y 方向上的有限差分来数值计算导数。这也是 GPU 架构围绕四像素组进行调度的原因。

一般来说，导数计算在内部自动进行，对用户不可见。实际实现通常是在一个四像素组内使用跨通道指令（shuffle/swizzle），这些指令可以由编译器插入。有些 GPU 则使用固定功能硬件来计算这些导数。关于应当如何计算导数，并没有精确的规范。图 23.14 展示了一些常见方法。OpenGL 4.5 和 DirectX 11 都支持用于计算粗导数和细导数的函数 [1368]。


![图 23.14：粗导数与细导数的计算方式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_8_23.14.png)

**图 23.14** 导数可能采用的计算方式示意图。箭头表示，以箭头终点处的像素减去起点处的像素来计算差值。例如，左上角的水平差分等于右上像素减去左上像素。对于粗导数（左），四像素组内的全部四个像素共用一个水平差分和一个垂直差分。对于细导数（右），则使用离该像素最近的差分。（插图据 Penner [1368] 绘制。）

所有 GPU 都使用纹理缓存 [362, 651, 794, 795] 来减少纹理的带宽用量。有些架构使用专用纹理缓存，甚至采用两级专用纹理缓存；另一些架构则让包括纹理处理在内的各种访问共享缓存。纹理缓存通常由一小块片上存储器（一般为 SRAM）实现。这个缓存存储最近的纹理读取结果，访问速度很快。替换策略和容量取决于具体架构。如果相邻像素需要访问相同或位置接近的纹素，就很可能在缓存中找到它们。如第 23.4 节所述，存储器访问通常采用分块方式，因此纹素不是按扫描线顺序存储，而是存放在小块中，例如每块 4 × 4 个纹素。这能提高效率 [651]，因为一个块中的纹素会一起被读取。以字节计的块大小通常与缓存行大小相同，例如 64 字节。另一种存储纹理的方法是采用交错重排（swizzled）模式。假设纹理坐标已经转换为定点数 (u, v)，其中 u 和 v 各有 n 位。u 中编号为 i 的位记作 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_08_b9eca245191763.png)。那么，将 (u, v) 重映射为交错纹理地址 A 的公式为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_08_432bdc9abae87c.png)


其中，B 是纹理的基地址，T 是一个纹素占用的字节数。这种重映射的优点是，它会产生图 23.15 所示的纹素顺序。可以看出，这是一条空间填充曲线，称为 Morton 序列 [1243]，已知它能够提高相干性 [1825]。这里的曲线是二维的，因为纹理通常也是二维的。

纹理单元还包含专门定制的硬件电路，用来解压缩多种不同的纹理格式（第 6.2.6 节）。与软件实现相比，以固定功能硬件实现这些解压缩操作，效率通常要高出许多倍。注意，当纹理既用作渲染目标，又用于纹理映射时，还会出现其他压缩机会。如果启用了颜色缓冲区压缩（第 23.5 节），那么在把这样的渲染目标当作纹理访问时，有两种设计选择。第一种选择是在渲染目标完成渲染后，将整个渲染目标从其颜色缓冲区压缩格式解压缩，并以未压缩形式存储，以供后续纹理访问。第二种选择是在纹理单元中增加硬件支持，以解压缩颜色缓冲区的压缩格式 [1716]。后一种选择效率更高，因为即使作为纹理被访问，渲染目标也可以保持压缩状态。有关缓存和压缩的更多信息，参见第 23.4 节。

mipmapping 对纹理缓存的局部性很重要，因为它限制了纹素与像素之比的最大值。在遍历一个三角形时，每前进到一个新像素，在纹理空间中大约就前进一个纹素。mipmapping 是渲染中少数几种能够同时改善视觉效果和性能的技术之一。


![图 23.15：纹理交错重排与 Morton 序列](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_8_23.15.png)

**图 23.15** 纹理交错重排提高了纹素存储器访问的相干性。注意，这里的纹素大小为 4 字节，每个纹素的左上角标出了其地址。


## 23.9 架构

来源：原书书页 1019—1023（PDF 物理页 1040—1044）；已核查 PDF 第 1045 页的 23.10 节起始边界。本节包含图 23.16—23.19 和公式（23.15）。

获得更快图形处理速度的最佳办法是利用并行性，而 GPU 中几乎所有阶段都可以这样做。其思想是同时计算多个结果，然后在后续阶段将它们合并。一般而言，并行图形架构具有图 23.16 所示的形式。应用程序向 GPU 发送任务，经过一定的调度之后，多个几何单元便开始并行地进行几何处理。几何处理的结果被转发给一组光栅化单元，由它们执行光栅化。接下来，一组像素处理单元也以并行方式执行像素着色和混合。最后，将生成的图像发送到显示器上供人观看。


![图23.16：高性能并行图形架构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_9_23.16.png)

图 23.16：高性能并行计算机图形架构的一般结构，由多个几何单元（G）、光栅化单元（R）和像素处理单元（P）组成。

无论对于软件还是硬件，都必须认识到：如果代码或硬件中存在串行部分，它就会限制总体性能可能获得的提升。这由阿姆达尔定律（Amdahl’s law）来表述，即：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_09_2244725f93cefb.png)


其中，s 是程序或硬件中串行部分所占的比例，因此 1 − s 就是能够并行化的部分所占的比例。此外，p 是通过将程序或硬件并行化所能实现的最大性能提升倍数。例如，如果原来只有一个多处理器，又增加了三个，那么 p = 4。这里，a(s, p) 是此次改进所带来的加速倍数。假设某个架构中有 10% 的部分是串行的，即 s = 0.1；我们对该架构进行改进，使剩余的非串行部分性能提升 20 倍，即 p = 20，那么就得到：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_09_e8ed6f26b13898.png)


可以看到，我们并没有得到 20 倍的加速，这是因为代码或硬件中的串行部分严重限制了性能。实际上，当 p → ∞ 时，得到 a = 10。应该把精力用于改进并行部分还是串行部分，并不总是显而易见的；但当并行部分已经得到大幅改进之后，串行部分对性能的限制会更加明显。

对于图形架构，多个结果虽然是并行计算的，但绘制调用中的图元应当按照 CPU 提交它们的顺序进行处理。因此，必须执行某种排序，使各个并行单元共同渲染出用户想要的图像。具体来说，需要的是从模型空间到屏幕空间的排序（第 2.3.1 节和第 2.4 节）。应当指出，几何单元和像素处理单元可以映射到相同的单元上，也就是统一的 ALU。案例研究一节中介绍的所有架构都采用统一着色器架构（第 23.10 节）。即便如此，理解这种排序发生在什么位置仍然很重要。下面介绍一种并行架构的分类方法 [417, 1236]。排序可以发生在流水线中的任何位置，由此产生了并行架构中四类不同的工作分配方式，如图 23.17 所示。它们称为前端排序（sort-first）、中间排序（sort-middle）、末端片元排序（sort-last fragment）和末端图像排序（sort-last image）。注意，这些架构会产生不同的方法，将工作分配给 GPU 中的各个并行单元。


![图23.17：并行图形架构分类](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_9_23.17.png)

图 23.17：并行图形架构的分类。A 是应用程序，G 是几何单元，R 是光栅化单元，P 是像素处理单元。从左到右，这些架构依次为前端排序、中间排序、末端片元排序和末端图像排序。（插图据 Eldridge 等人 [417] 绘制。）

基于前端排序的架构在几何阶段之前对图元排序。其策略是将屏幕划分为一组区域，并把某一区域内的图元发送给“拥有”该区域的一条完整流水线。见图 23.18。首先，对图元进行足以确定它需要发送到哪些区域的处理；这就是排序步骤。就单台机器而言，前端排序是研究最少的一种架构 [418, 1236]。当多个屏幕或投影机组成一个大型显示系统时，这种方案确实有所应用：每个屏幕都由一台专用计算机负责 [1513]。已经开发出一种名为 Chromium [787] 的系统，它可以利用工作站集群实现任意类型的并行渲染算法。例如，它能够以很高的渲染性能实现前端排序和末端排序。


![图23.18：前端排序示例](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_9_23.18.png)

图 23.18：前端排序将屏幕划分为各个独立的图块，并为每个图块分配一个处理器，如图所示。然后，将图元发送给它所覆盖图块对应的处理器。这与中间排序架构不同；后者需要在几何处理完成之后对所有三角形进行排序。只有在所有三角形都完成排序之后，才能开始逐像素光栅化。（图片由 Marcus Roth 和 Dirk Reiners 提供。）

Mali 架构（第 23.10.1 节）属于中间排序类型。各几何处理单元获得大致等量的几何数据进行处理。随后，将变换后的几何数据排序到互不重叠的矩形中，这些矩形称为图块（tile），它们共同覆盖整个屏幕。注意，变换后的一个三角形可能与多个图块重叠，因此可能由多个光栅化单元和像素处理单元处理。这里实现高效率的关键是：每一对光栅化单元与像素处理单元都在芯片上拥有一个图块大小的帧缓冲区，这意味着所有帧缓冲区访问都很快。当所有几何数据都已被排序到图块中之后，各个图块的光栅化和像素处理就可以彼此独立地开始。一些中间排序架构会针对不透明几何体，在每个图块上执行一次深度预通道（z-prepass），这意味着每个像素只会着色一次。然而，并非所有中间排序架构都会这样做。

末端片元排序架构在光栅化之后、像素处理之前对片元排序；光栅化有时也称为片元生成。第 23.10.3 节介绍的 GCN 架构就是一个例子。与中间排序一样，图元被尽可能均匀地分配到各个几何单元。末端片元排序的一项优点是不会有重叠，也就是说，生成的片元只发送给一个像素处理单元，这是最理想的。如果某个光栅化单元处理的是大三角形，而另一个只处理小三角形，就可能出现负载不均衡。

最后，末端图像排序架构在像素处理之后进行排序。图 23.19 给出了直观示例。这种架构可以看作一组相互独立的流水线。图元被分配给这些流水线，每条流水线各自渲染一幅带有深度信息的图像。在最后的合成阶段，根据各幅图像的深度缓冲区将所有图像合并。应当指出，末端图像排序系统无法完整实现 OpenGL 和 DirectX 这样的 API，因为这些 API 要求图元按照提交的顺序渲染。PixelFlow [455, 1235] 是末端图像排序架构的一个例子。PixelFlow 架构还值得关注的一点是它采用了延迟着色，也就是说，它只对可见片元着色。不过应当指出，由于流水线末端的带宽消耗很大，目前没有架构采用末端图像排序。


![图23.19：末端图像排序示例](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_9_23.19.png)

图 23.19：在末端图像排序中，场景中的不同物体被发送给不同的处理器。合成分别渲染的图像时，透明性很难处理，因此通常会将透明物体发送给所有节点。（图片由 Marcus Roth 和 Dirk Reiners 提供。）

对于大型拼接显示系统，纯粹的末端图像排序方案存在一个问题：渲染节点之间需要传输的图像和深度数据量极其庞大。Roth 和 Reiners [1513] 利用每个处理器所生成结果的屏幕范围和深度范围，来优化数据传输与合成开销。

Eldridge 等人 [417, 418] 提出了 Pomegranate，这是一种处处排序（sort-everywhere）架构。简而言之，它在几何阶段与光栅化单元（R）之间、R 与像素处理单元（P）之间，以及 P 与显示器之间，都插入了排序阶段。因此，随着系统规模扩大，也就是增加更多流水线，工作负载仍能保持较好的均衡。排序阶段通过具有点对点链路的高速网络实现。仿真表明，随着流水线数量增加，性能几乎呈线性提升。

将图形系统中的所有组成部分——主机、几何处理、光栅化和像素处理——连接在一起，就得到了一个多处理系统。对于这类系统，有两个众所周知、而且几乎总是与多处理相伴的问题：负载均衡和通信 [297]。通常会在流水线的许多不同位置插入 FIFO（先进先出）队列，使作业能够排队，以免流水线的某些部分停顿。例如，可以在几何单元与光栅化单元之间放置一个 FIFO，这样，如果光栅化单元由于三角形尺寸很大等原因，无法跟上几何单元的处理速度，就可以将已经完成几何处理的三角形缓存在队列中。

前面介绍的各种排序架构，在负载均衡方面各有不同的优点和缺点。更多信息可参阅 Eldridge 的博士论文 [418] 或 Molnar 等人的论文 [1236]。程序员也可以影响负载均衡；相关技术将在第 18 章讨论。如果总线带宽过低，或使用不合理，通信就可能成为问题。因此，设计应用程序的渲染系统时，极其重要的一点是避免让任何一条总线成为瓶颈，例如从主机到图形硬件的总线。第 18.2 节介绍了检测瓶颈的各种方法。


## 23.10 案例研究

来源：《Real-Time Rendering, Fourth Edition》书页 1024—1039（PDF 物理页 1045—1060）。范围自 23.10 标题起，至 23.11 标题前，包含全部三个下级小节；插图按正文关联位置编排。

本节将介绍三种不同的图形硬件架构。首先介绍面向移动设备与电视的 ARM Mali G71 Bifrost 架构，随后介绍 NVIDIA 的 Pascal 架构，最后描述名为 Vega 的 AMD GCN 架构。

请注意，图形硬件公司在作出设计决策时，往往以对尚未制造出来的 GPU 进行的大量软件模拟为依据。也就是说，他们让多个应用程序（例如游戏）在参数化模拟器中，以若干种不同配置运行。可调整的参数例如包括 MP 数量、时钟频率、缓存数量、光栅引擎／曲面细分引擎数量以及 ROP 数量。模拟用于收集性能、功耗和内存带宽使用量等方面的信息。最终，他们选择在大多数使用情形下表现最好的配置，并据此制造芯片。此外，模拟还可以帮助找出架构中的典型瓶颈，随后便可采取措施解决，例如增大某个缓存。对于某款 GPU，其各项速度和单元数量之所以如此，原因简单地说就是：“这样效果最好。”

### 23.10.1 案例研究：ARM Mali G71 Bifrost

Mali 产品线涵盖 ARM 的所有 GPU 架构，Bifrost 是其 2016 年的架构。这一架构面向移动与嵌入式系统，例如手机、平板电脑和电视。2015 年，基于 Mali 的 GPU 出货量达到 7.5 亿颗。由于其中许多设备由电池供电，因此设计节能的架构非常重要，不能只关注性能。因此，采用中间排序（sort-middle）架构很有意义：所有帧缓冲访问均保留在芯片内部，从而降低功耗。所有 Mali 架构都属于中间排序类型，有时也称为分块（tiling）架构。图 23.20 展示了 GPU 的高层概貌。可以看到，G71 最多支持 32 个统一着色器引擎。ARM 使用“着色器核心”（shader core）而非“着色器引擎”（shader engine）一词，但为了避免与本章其余内容混淆，我们使用后者。一个着色器引擎能够同时为 12 个线程执行指令，即它具有 12 个 ALU。32 个着色器引擎是 G71 的具体配置选择，但架构本身可以扩展到超过 32 个引擎。

驱动软件向 GPU 提交工作。随后，作业管理器（即调度器）将工作分配给各个着色器引擎。这些引擎通过 GPU 互连（GPU fabric）相连；它是一条总线，引擎可以通过它与 GPU 内的其他单元通信。所有内存访问都通过内存管理单元（MMU），由它把虚拟内存地址转换为物理地址。


![图23.20 Bifrost G71 GPU架构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.20.png)

图 23.20。Bifrost G71 GPU 架构，可扩展至 32 个着色器引擎，每个着色器引擎均采用图 23.21 所示的结构。（根据 Davies [326] 的插图绘制。）

图 23.21 展示了着色器引擎的概貌。可以看到，它包含三个执行引擎，以执行四元组（quad）的着色为中心。因此，它们被设计成 SIMD 宽度为 4 的小型通用处理器。每个执行引擎包含四个用于 32 位浮点数的融合乘加（FMA）单元和四个 32 位加法器等。这意味着每个着色器引擎具有 3 × 4 个 ALU，即 12 条 SIMD 通道。按照本文使用的术语，四元组相当于一个线程束（warp）。为了隐藏纹理访问等操作的延迟，该架构可以让每个着色器引擎至少保有 256 个正在处理中的线程。


![图23.21 Bifrost着色器引擎](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.21.png)

图 23.21。Bifrost 着色器引擎架构。分块内存位于芯片内部，使局部帧缓冲访问能够快速完成。（根据 Davies [326] 的插图绘制。）

请注意，着色器引擎是统一的，可以执行计算着色、顶点着色和像素着色等任务。执行引擎还支持许多超越函数，例如正弦和余弦。此外，采用 16 位浮点精度时，性能最高可达 2 倍。当寄存器中的结果仅用作后续某条指令的输入时，这些单元还支持旁路传递寄存器内容。这样无需访问寄存器文件，因而能够节省功耗。此外，在进行纹理访问或其他内存访问等操作时，四元组管理器可以换入单个四元组，方式类似于其他架构隐藏此类操作延迟的做法。请注意，这种切换的粒度很小：交换的是 4 个线程，而非全部 12 个线程。加载／存储单元负责一般内存访问、内存地址转换以及一致性缓存 [264]。属性单元处理属性索引和寻址，并把访问请求发送给加载／存储单元。插值属性单元（varying unit）对变化的属性执行插值。

分块架构（中间排序）的核心思想是先执行所有几何处理，从而确定每个待渲染图元的屏幕空间位置。与此同时，为帧缓冲中的每个分块建立一个多边形列表，其中包含指向所有与该分块重叠的图元的指针。这一步之后，与分块重叠的图元集合就已确定。因此，可以对一个分块内的图元进行光栅化和着色，并把结果保存在片上分块内存中。当该分块中的全部图元渲染完毕后，分块内存中的数据经由 L2 缓存写回外部内存。这会减少内存带宽使用量。然后对下一个分块进行光栅化，依此类推，直到整帧渲染完成。最早的分块架构是 Pixel-Planes 5 [502]，该系统在高层结构上与 Mali 架构有一些相似之处。

图 23.22 展示了几何处理与像素处理。可以看到，顶点着色器被拆分为两部分：一部分只对位置进行着色，另一部分称为插值属性着色（varying shading），在分块之后执行。与 ARM 先前的架构相比，这样可以节省内存带宽。执行分箱（binning），即确定图元与哪些分块重叠，唯一需要的信息就是顶点的位置。负责分箱的分块器单元以层次化方式工作，如图 23.23 所示。这有助于减小分箱的内存占用，并使其更加可预测，因为内存占用不再与图元大小成正比。


![图23.22 Bifrost的几何数据流](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.22.png)

图 23.22。几何数据在 Bifrost 架构中的流动方式。顶点着色器包含供分块器使用的位置着色，以及仅在需要时、于分块之后执行的插值属性着色。（根据 Choi [264] 的插图绘制。）

当分块器完成场景中所有图元的分箱后，就确切知道哪些图元与某个分块重叠。因此，只要有可并行工作的着色器引擎，就可以对任意数量的分块并行执行剩余的光栅化、像素处理和混合操作。通常，一个分块被提交给一个着色器引擎，由它处理该分块中的所有图元。在对所有分块执行这些工作的同时，也可以开始下一帧的几何处理和分块。这种处理模型意味着分块架构可能具有更大的延迟。

接下来执行光栅化、像素着色器、混合以及其他逐像素操作。分块架构最重要的单一特性是，一个分块的帧缓冲（例如包含颜色、深度和模板）可以保存在快速的片上内存中，这里称为分块内存（tile memory）。由于分块很小（16 × 16 像素），这样做在成本上是可行的。一个分块中的全部渲染完成后，将该分块所需的输出（通常是颜色，也可能包括深度）复制到与屏幕尺寸相同的片外帧缓冲中（位于外部内存）。这意味着逐像素处理期间的全部帧缓冲访问实际上几乎无需付出代价。避免使用外部总线非常有益，因为使用它们会带来很高的能耗 [22]。将片上分块内存的内容换出到片外帧缓冲时，仍可以使用帧缓冲压缩。


![图23.23 Bifrost层次化分块器](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.23.png)

图 23.23。Bifrost 架构的层次化分块器。本例在三个不同层级上进行分箱，每个三角形被分配到它只与一个方格重叠的那个层级。（根据 Bratt [191] 的插图绘制。）

Bifrost 支持像素局部存储（pixel local storage，PLS），这是一组通常由中间排序架构支持的扩展。利用 PLS，可以让像素着色器访问帧缓冲的颜色，进而实现自定义混合技术。相比之下，混合通常由 API 配置，不能像像素着色器那样编程。还可以利用分块内存，为每个像素存储任意固定大小的数据结构。这使程序员能够高效实现延迟着色等技术。第一个遍中将 G 缓冲（例如法线、位置和漫反射纹理）存入 PLS。第二个遍执行光照计算，并将结果累积到 PLS 中。第三个遍利用 PLS 中的信息计算最终像素颜色。请注意，对于单个分块而言，所有这些计算期间，整个分块内存都保留在芯片上，因此速度很快。

所有 Mali 架构在从头设计时就考虑了多重采样抗锯齿（MSAA），并实现了第 143 页介绍的旋转网格超采样（RGSS）方案，每个像素使用四个样本。中间排序架构非常适合抗锯齿，因为滤波恰好在分块离开 GPU、被发送到外部内存之前执行。因此，外部内存中的帧缓冲只需为每个像素存储一种颜色。标准架构则需要四倍大小的帧缓冲。对于分块架构，只需将片上分块缓冲增大到四倍，或者等效地使用更小的分块（宽和高分别减半）。

Mali Bifrost 架构还可以为一批渲染图元选择使用多重采样或超采样。这意味着，必要时可以使用成本更高的超采样，即为每个样本执行像素着色器。例如，渲染一棵采用 alpha 映射的带纹理树木时，需要高质量采样来避免视觉伪影。对于这些图元，可以启用超采样。当复杂情形结束，开始渲染较简单的物体时，可以切回成本较低的多重采样。该架构还支持 8× 和 16× MSAA。

Bifrost（以及前一代名为 Midgard 的架构）还支持一种称为事务消除（transaction elimination）的技术。其思想是：对于场景中逐帧不变的部分，避免从分块内存向片外内存传输数据。当前帧中，每个分块被换出到片外帧缓冲时，都会计算其唯一签名。该签名是一种校验和。下一帧则为即将换出的分块计算签名。如果某个分块上一帧的签名与当前帧相同，架构便不再把颜色缓冲写入片外内存，因为其中已经有正确内容。这对休闲移动游戏（例如《愤怒的小鸟》）特别有用，因为每帧只更新场景中的较小一部分。还应注意，这类技术难以在末端排序（sort-last）架构上实现，因为后者不是按分块工作的。G71 还支持智能合成（smart composition），即将事务消除应用于用户界面合成。如果所有来源和操作均与前一帧相同，就可以避免对一个像素块进行读取、合成和写入。

该架构还大量使用时钟门控和电源门控等底层节能技术。这意味着流水线中未使用或不活跃的部分会被关闭，或以较低能耗保持空闲，从而降低功耗。

为减少纹理带宽，纹理缓存配有专门用于 ASTC 和 ETC 的解压单元。此外，压缩纹理以压缩形式保存在缓存中，而不是先解压，再把纹素放入缓存。这意味着，收到纹素请求时，硬件先从缓存读取相应块，再即时解压该块的纹素。这种配置增加了缓存的有效容量，提高了效率。

总体而言，分块架构的一个优点是，其设计天然适合并行处理分块。例如，可以增加着色器引擎，每个引擎负责在某一时刻独立渲染一个分块。分块架构的一个缺点是，整个场景的数据都需要发送到 GPU 进行分块，处理后的几何数据还要流式写出到内存中。一般来说，中间排序架构并不适合处理几何放大，例如使用几何着色器和曲面细分，因为更多的几何数据会增加来回搬运几何数据所需的内存传输量。在 Mali 架构中，几何着色（第 18.4.2 节）与曲面细分均由 GPU 上的软件处理，Mali 最佳实践指南 [69] 建议完全不要使用几何着色器。对于大多数内容，中间排序架构在移动和嵌入式系统上表现良好。

### 23.10.2 案例研究：NVIDIA Pascal

Pascal 是 NVIDIA 开发的一种 GPU 架构。它既有图形版本 [1297]，也有计算版本 [1298]，后者面向高性能计算和深度学习应用。这里主要关注图形版本，尤其是名为 GeForce GTX 1080 的具体配置。我们将采用自底向上的方式介绍该架构，从最小的统一 ALU 开始，逐步构建出整个 GPU。本小节末尾还会简要提到其他一些芯片配置。

Pascal 图形架构采用的统一 ALU，在 NVIDIA 术语中称为 CUDA 核心，其高层结构与第 1002 页图 23.8 左侧的 ALU 相同。ALU 主要用于浮点数和整数运算，但也支持其他操作。为了提高计算能力，多个此类 ALU 被组合为一个流式多处理器（streaming multiprocessor，SM）。在 Pascal 的图形版本中，SM 由四个处理块组成，每个处理块有 32 个 ALU。这意味着 SM 可以同时执行四个各含 32 个线程的线程束，如图 23.24 所示。

每个处理块，即宽度为 32 的 SIMT 引擎，还具有 8 个加载／存储（LD/ST）单元和 8 个特殊函数单元（SFU）。加载／存储单元负责读取和写入寄存器文件中的寄存器值。每个处理块的寄存器文件容量为 16,384 × 4 字节，即 64 kB，合计每个 SM 为 256 kB。SFU 处理超越函数指令，例如正弦、余弦、以 2 为底的指数、以 2 为底的对数、倒数以及平方根的倒数。它们还支持属性插值 [1050]。


![图23.24 Pascal流式多处理器](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.24.png)

图 23.24。Pascal 流式多处理器（SM）具有 32 × 2 × 2 个统一 ALU，SM 与一个多形体引擎（polymorph engine）封装在一起，共同构成纹理处理簇（TPC）。请注意，上方的深灰色方框在紧下方重复了一次，但重复部分中的一些内容被省略了。（根据 NVIDIA 白皮书 [1297] 的插图绘制。）

SM 中的所有 ALU 共享一个指令缓存，而每个 SIMT 引擎拥有自己的指令缓冲，其中保存最近加载的一组局部指令，以进一步提高指令缓存命中率。线程束调度器每个时钟周期能够分派两条线程束指令 [1298]，例如可以在同一周期中同时向 ALU 和 LD/ST 单元调度工作。还应注意，每个 SM 有两个 L1 缓存，各具有 24 kB 存储空间，即每个 SM 合计 48 kB。之所以设置两个 L1 缓存，很可能是因为更大的 L1 缓存需要更多读写端口，这会提高缓存的复杂度，并增大其芯片实现面积。此外，每个 SM 有 8 个纹理单元。

由于着色必须按 2 × 2 像素四元组执行，线程束调度器会寻找 8 个不同像素四元组的工作，将它们组合起来，在 32 条 SIMT 通道上执行 [1050]。由于采用统一 ALU 设计，线程束调度器可以把顶点、像素、图元或计算着色器工作中的某一种组合成线程束。请注意，一个 SM 可以同时处理不同类型的线程束（例如顶点、像素和图元）。该架构还能够以零开销将当前执行的线程束换出，换入已经准备好执行的线程束。Pascal 如何选择下一个待执行线程束，其细节并未公开，但 NVIDIA 先前的一种架构提供了一些线索。2008 年的 NVIDIA Tesla 架构 [1050] 使用记分牌（scoreboard），在每个时钟周期判断各线程束是否符合发射条件。记分牌是一种允许无冲突乱序执行的通用机制。线程束调度器从已准备好执行的线程束中（例如未在等待纹理加载返回的线程束）选择优先级最高者。用于选择最高优先级线程束的参数包括线程束类型、指令类型和“公平性”。

SM 与多形体引擎（polymorph engine，PM）协同工作。该单元的最初版本在 Fermi 芯片中引入 [1296]。PM 执行若干与几何相关的任务，包括顶点获取、曲面细分、同步多投影、属性设置和流输出。第一阶段从全局顶点缓冲中获取顶点，并向 SM 分派线程束，进行顶点着色与外壳着色。随后是可选的曲面细分阶段（第 17.6 节）：将新生成的曲面片坐标 (u, v) 分派给 SM，进行域着色以及可选的几何着色。第三阶段处理视口变换和透视校正。此外，这里还会执行可选的同步多投影步骤，例如可用于高效的 VR 渲染（第 21.3.1 节）。接下来是可选的第四阶段，将顶点流式输出到内存。最后，结果被转发给相关的光栅引擎。

光栅引擎有三个任务：三角形设置、三角形遍历以及 z 剔除。三角形设置获取顶点、计算边方程，并执行背面剔除。三角形遍历采用层次化的分块遍历技术，访问与三角形重叠的分块。它利用边方程执行分块测试与内部测试。在 Fermi 上，每个光栅器每时钟周期最多处理 8 个像素 [1296]。Pascal 在这方面没有公开数据。z 剔除单元使用第 23.7 节介绍的技术，以分块为单位进行剔除。如果某个分块被剔除，便立即终止对它的处理。对于未被剔除的三角形，逐顶点属性被转换成平面方程，以便在像素着色器中高效求值。

一个流式处理器与一个多形体引擎结合，称为纹理处理簇（texture processing cluster，TPC）。在更高层级上，五个 TPC 组成一个图形处理簇（graphics processing cluster，GPC），由一个光栅引擎为这五个 TPC 提供服务。GPC 可以看作一个小型 GPU，其目标是提供一组相互平衡的图形硬件单元，例如顶点、几何、光栅、纹理、像素和 ROP 单元。如本小节末尾将看到的，把功能划分为独立单元后，设计人员可以更容易地创建一个能力各不相同的 GPU 芯片家族。


![图23.25 GTX1080配置的Pascal GPU](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.25.png)

图 23.25。采用 GTX 1080 配置的 Pascal GPU：包含 20 个 SM、20 个多形体引擎、4 个光栅引擎、8 × 20 = 160 个纹理单元（峰值速率为 277.3 Gtexels/s）、总容量为 256 × 20 = 5120 kB 的寄存器文件，以及总计 20 × 128 = 2560 个统一 ALU。（根据 NVIDIA 白皮书 [1297] 的插图绘制。）

至此，我们已经介绍了 GeForce GTX 1080 的大多数组成模块。它由四个 GPC 组成，整体配置如图 23.25 所示。请注意，这里还有一层由 GigaThread 引擎负责的调度，并配有 PCIe v3 接口。GigaThread 引擎是一个全局工作分配引擎，负责将线程块调度到所有 GPC。

图 23.25 还展示了光栅操作单元，不过位置不太显眼。它们紧邻图中央 L2 缓存的上方和下方。每个蓝色块代表一个 ROP 单元，共有 8 组，每组 8 个，总计 64 个。ROP 单元的主要任务是将输出写入像素及其他缓冲，并执行混合等操作。从图的左右两侧可以看到，总共有八个 32 位内存控制器，合计为 256 位。八个 ROP 单元与一个内存控制器及 256 kB 的 L2 缓存绑定。这使整颗芯片的 L2 缓存总容量达到 2 MB。每个 ROP 都绑定到特定的内存分区，因此负责缓冲中某个特定像素子集。ROP 单元还处理无损压缩。除支持未压缩形式和快速清除外，还有三种不同的压缩模式 [1297]。对于 2∶1 压缩（例如从 256 B 压缩到 128 B），每个分块存储一个参考颜色值，并对像素间的差值进行编码；每个差值使用的位数少于其未压缩形式。4∶1 压缩是 2∶1 模式的扩展，但只有在差值能用更少位数编码时才能启用，因此只适用于内容平滑变化的分块。还有一种 8∶1 模式，它将 2 × 2 像素块的 4∶1 恒定颜色压缩与上述 2∶1 模式结合。8∶1 模式优先于 4∶1，后者又优先于 2∶1，也就是说，始终使用能够成功压缩该分块的最高压缩率模式。如果所有压缩尝试均失败，分块就必须以未压缩形式传输并存入内存。图 23.26 展示了 Pascal 压缩系统的效率。


![图23.26 Maxwell与Pascal缓冲压缩](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.26.png)

图 23.26。左侧为渲染图像，中间和右侧分别可视化了 Pascal 的上一代架构 Maxwell，以及 Pascal 的压缩结果。图像中的紫色越多，缓冲压缩的成功率越高。（图片来自 NVIDIA 白皮书 [1297]。）

使用的显存为 GDDRX5，时钟速率为 10 GHz。上文已知，八个内存控制器合计提供 256 位 = 32 B。因此，总峰值内存带宽为 320 GB/s；但多级缓存与压缩技术相结合，使其有效速率看起来更高。

> 译注：此处原文及图 23.27 将显存名称写为“GDDRX5”，疑为“GDDR5X”的字母顺序排印错误；译文保留原写法。

芯片的基础时钟频率为 1607 MHz，功率预算充足时可运行于加速模式（1733 MHz）。峰值计算能力为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_10_4869f058d47adb.png)


其中，系数 2 来自融合乘加通常被计为两次浮点运算这一事实；从 MFLOPS 换算到 TFLOPS 时，我们除以了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_10_7dd53e0e87e793.png)。GTX 1080 Ti 具有 3584 个 ALU，计算能力为 12.3 TFLOPS。

NVIDIA 长期以来一直开发末端片元排序（sort-last fragment）架构。不过，自 Maxwell 起，它们还支持一种称为分块缓存（tiled caching）的新渲染方式，某种程度上介于中间排序与末端片元排序之间。图 23.27 展示了该架构。其思想是利用局部性和 L2 缓存。几何数据被分成足够小的批块进行处理，使输出能留在这一缓存内。此外，只要与当前分块重叠的几何数据还未完成像素着色，该分块的帧缓冲也保留在 L2 中。


![图23.27 分块缓存架构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.27.png)

图 23.27。分块缓存引入一个分箱器，将几何数据分类到分块中，并使变换后的几何数据保留在 L2 缓存内。当前处理的分块也保留在 L2 中，直到当前几何批块中落入该分块的几何数据处理完毕。

图 23.25 中有四个光栅引擎，但我们知道，图形 API 在大多数情况下必须遵守图元的提交顺序 [1598]。帧缓冲通常以一种广义棋盘格模式划分为分块 [1160]，每个光栅引擎“拥有”其中一组分块。当前三角形会发送给所有至少有一个所属分块与它重叠的光栅引擎，从而为每个分块独立解决顺序问题。这有利于负载均衡。GPU 架构中通常还有多个 FIFO 队列，用来减少硬件单元因没有工作而饥饿的情况。图中没有画出这些队列。

显示控制器的每个颜色分量为 12 位，并支持 BT.2020 广色域。它还支持 HDMI 2.0b 和 HDCP 2.2。视频处理方面，它支持 SMPTE 2084，这是一种用于高动态范围视频的传递函数。Venkataraman [1816] 描述了 NVIDIA 从 Fermi 起的架构如何配备一个或多个复制引擎（copy engine）。这些内存控制器能够执行直接内存访问（DMA）传输。DMA 传输发生在 CPU 与 GPU 之间，通常由其中一方启动。启动传输的处理单元可以在传输期间继续执行其他计算。复制引擎可以发起 CPU 与 GPU 内存之间的数据 DMA 传输，并独立于 GPU 其余部分执行。因此，当信息从 CPU 传输到 GPU，或反向传输时，GPU 仍可以渲染三角形并执行其他功能。

Pascal 架构也可以配置为用于训练神经网络或大规模数据分析等非图形应用。Tesla P100 就是其中一种配置 [1298]。它与 GTX 1080 的一些区别包括：采用第二代高带宽内存（HBM2），内存总线宽度为 4096 位，总内存带宽为 720 GB/s。此外，它原生支持 16 位浮点数，性能最高为 32 位浮点数的 2 倍，双精度处理速度也显著更快。SM 的配置以及寄存器文件设置同样不同 [1298]。

GTX 1080 Ti（titanium，钛）是一种更高端的配置。它具有 3584 个 ALU、352 位内存总线、484 GB/s 总内存带宽、88 个 ROP 和 224 个纹理单元；GTX 1080 对应的数据分别为 2560、256 位、320 GB/s、64 和 160。它配置了六个 GPC，即六个光栅引擎，而 GTX 1080 为四个。其中四个 GPC 与 GTX 1080 完全相同，另外两个稍小，各只有四个 TPC，而非五个。1080 Ti 芯片使用 120 亿个晶体管，1080 则使用 72 亿个。Pascal 架构很灵活，也能够向下缩减。例如，GTX 1070 相当于 GTX 1080 减去一个 GPC，而 GTX 1050 由两个 GPC 组成，每个含三个 SM。

### 23.10.3 案例研究：AMD GCN Vega

AMD 的 Graphics Core Next（GCN）架构被用于多款 AMD 显卡，以及 Xbox One 和 PLAYSTATION 4。这里介绍 GCN Vega 架构 [35] 的一般组成；它是这些游戏机所用架构的演进版本。

GCN 架构的一个核心组成模块是计算单元（compute unit，CU），如图 23.28 所示。CU 具有四个 SIMD 单元，每个单元有 16 条 SIMD 通道，即 16 个统一 ALU（采用第 23.2 节的术语）。每个 SIMD 单元为 64 个线程执行指令，这组线程称为波前（wavefront）。每个 SIMD 单元每个时钟周期可以发射一条单精度浮点指令。由于该架构让每个 SIMD 单元处理含 64 个线程的波前，需要 4 个时钟周期才能完成一个波前的全部发射 [1103]。还应注意，一个 CU 可以同时运行不同内核的代码。由于每个 SIMD 单元具有 16 条通道，每个时钟周期可以发射一条指令，整个 CU 的最大吞吐量为：每个 CU 的 4 个 SIMD 单元 × 每个单元的 16 条 SIMD 通道 = 每时钟周期 64 次单精度浮点运算。CU 执行半精度（16 位浮点）指令的数量还可以达到单精度的两倍，这对于精度要求较低的情形很有用，例如机器学习和着色器计算。请注意，两个 16 位浮点值被打包到一个 32 位浮点寄存器中。每个 SIMD 单元具有 64 kB 的寄存器文件，折合每个线程 65,536 ÷ (4 × 64) = 256 个寄存器，因为单精度浮点数占 4 字节，而每个波前有 64 个线程。ALU 具有四级硬件流水线 [35]。


![图23.28 Vega的GCN计算单元](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.28.png)

图 23.28。Vega 架构的 GCN 计算单元。每个向量寄存器文件的容量为 64 kB，标量寄存器文件（RF）为 12.5 kB，局部数据共享存储为 64 kB。请注意，每个 CU 中有四个用于计算的单元，各具有 16 条 32 位浮点 SIMD 通道（浅绿色）。（根据 Mah [1103] 和 AMD 白皮书 [35] 的插图绘制。）

每个 CU 具有一个指令缓存（图中未画出），最多由四个 SIMD 单元共享。相关指令被转发到 SIMD 单元的指令缓冲（IB）。每个 IB 可存储用于处理 10 个波前的信息；需要时，这些波前可换入或换出 SIMD 单元，以隐藏延迟。这意味着一个 CU 可以处理 40 个波前，相当于 40 × 64 = 2560 个线程。因此，图 23.28 中的 CU 调度器可同时处理 2560 个线程，其任务是将工作分配给 CU 的不同单元。每个时钟周期都会考虑当前 CU 的全部波前是否可以发射指令，并且最多向每个执行端口发射一条指令。CU 的执行端口包括分支、标量／向量 ALU、标量／向量内存、局部数据共享、全局数据共享或导出，以及特殊指令 [32]；换言之，每个执行端口大致对应 CU 的一个单元。

标量单元是一个 64 位 ALU，同样由各 SIMD 单元共享。它拥有自己的标量寄存器文件和标量数据缓存（未画出）。标量寄存器文件为每个 SIMD 单元提供 800 个 32 位寄存器，即 800 × 4 × 4 = 12.5 kB。其执行与波前紧密耦合。由于向一个 SIMD 单元完整发射一条指令需要四个时钟周期，标量单元只能每隔四个时钟周期为某个特定 SIMD 单元服务一次。标量单元处理控制流、指针运算，以及线程束内各线程可以共享的其他计算。有条件和无条件分支指令从标量单元发送到分支与消息单元执行。每个 SIMD 单元有一个 48 位程序计数器（PC），由所有通道共享。因为各通道执行相同指令，这样已经足够。分支被采取时，程序计数器便会更新。该单元可发送的消息包括调试消息、特殊图形同步消息和 CPU 中断 [1121]。

图 23.29 展示了 Vega 10 架构 [35]。上部包含一个图形命令处理器、两个硬件调度器（HWS）以及八个异步计算引擎（ACE）[33]。GPC 的任务是把图形任务分派到 GPU 的图形流水线与计算引擎。HWS 的缓冲以队列形式工作，并在可能时尽快将队列分配给 ACE。ACE 的任务是把计算任务调度到计算引擎。此外，还有两个可以处理复制任务的 DMA 引擎（图中未画出）。GPC、ACE 和 DMA 引擎能够并行工作，并向 GPU 提交工作。由于不同队列的任务可以交错执行，这提高了利用率。任何队列都可以分派工作，而无需等待其他工作完成，因此独立任务能够同时在计算引擎上执行。ACE 可以通过缓存或内存同步。它们可以共同支持任务图，让某个 ACE 的任务依赖另一个 ACE 的任务，或依赖图形流水线的任务。建议将较小的计算和复制任务与负载较重的图形任务交错安排 [33]。

> 译注：上一段原文两次使用缩写“GPC”指代图形命令处理器；这与上一小节表示图形处理簇的 GPC 含义不同，且与“graphics command processor”的词首顺序不一致。此处保留原文缩写。


![图23.29 Vega10 GPU架构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.29.png)

图 23.29。由 64 个 CU 构成的 Vega 10 GPU。请注意，每个 CU 都包含图 23.28 所示的硬件。（根据 AMD 白皮书 [35] 的插图绘制。）

从图 23.29 可以看到，共有四条图形流水线和四个计算引擎。每个计算引擎具有 16 个 CU，合计 64 个。图形流水线有两个模块：几何引擎和绘制流分箱光栅器（draw-stream binning rasterizer，DSBR）。几何引擎包括几何装配器、曲面细分单元和顶点装配器。此外，还支持一种新的图元着色器（primitive shader）。图元着色器旨在实现更灵活的几何处理和更快的图元剔除 [35]。DSBR 结合了中间排序与末端排序架构的优点，这也是分块缓存的目标（第 23.10.2 节）。图像在屏幕空间划分为分块，几何处理之后，每个图元被分配到与其重叠的分块中。对一个分块进行光栅化期间，所需的所有数据（例如分块缓冲）都保存在 L2 缓存中，从而提高性能。像素着色可以自动推迟到一个分块中的所有几何数据处理完毕之后。因此，系统在内部执行一次 z 预遍，像素仅着色一次。延后着色可以开启和关闭；例如，对于透明几何数据，必须关闭。

为处理深度、模板和颜色缓冲，GCN 架构提供了一种称为颜色与深度模块（color and depth block，CDB）的组成模块。它们除执行颜色混合外，还处理颜色、深度和模板的读写。CDB 可以采用第 23.5 节介绍的一般方法压缩颜色缓冲。这里采用差分压缩技术，每个分块以未压缩形式存储一个像素的颜色，其余颜色值均相对于该像素颜色进行编码 [34, 1238]。为提高效率，可以根据访问模式动态选择分块大小。对于最初以 256 字节存储的分块，最大压缩率为 8∶1，即压缩至 32 字节。压缩后的颜色缓冲可以在后续遍中用作纹理，此时纹理单元会解压该压缩分块，进一步节省带宽 [1716]。

光栅器每个时钟周期最多可以光栅化四个图元。与图形流水线和计算引擎连接的 CDB 每时钟周期能够写入 16 个像素。也就是说，小于 16 像素的三角形会降低效率。光栅器还处理粗粒度深度测试（HiZ）和层次化模板测试。HiZ 使用的缓冲称为 HTILE，开发人员可以对它编程，例如用它向 GPU 提供遮挡信息。


![图23.30 Vega缓存层次结构](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_23_10_23.30.png)

图 23.30。Vega 架构的缓存层次结构。

Vega 的缓存层次结构如图 23.30 所示。层次结构的顶端（图中最右侧）是寄存器，随后是 L1 和 L2 缓存。再往下是同样位于显卡上的第二代高带宽内存（HBM2），最后是 CPU 一侧的系统内存。Vega 的一项新特性是图 23.29 底部所示的高带宽缓存控制器（High-Bandwidth Cache Controller，HBCC）。它使显存能够像最后一级缓存那样工作。这意味着，如果一次内存访问对应的内容不在显存（即 HBM2）中，HBCC 就会自动通过 PCIe 总线取来相关的一个或多个系统内存页面，并将其放入显存。作为结果，显存中较久未使用的页面可能被换出。HBM2 与系统内存之间共享的内存池称为 HBCC 内存段（HBCC memory segment，HMS）。所有图形模块也都通过 L2 缓存访问内存，这与先前架构不同。该架构还支持虚拟内存（第 19.10.1 节）。

请注意，所有片上模块，例如 HBCC、XDMA（CrossFire DMA）、PCI Express、显示引擎和多媒体引擎，都通过名为 Infinity Fabric（IF）的互连通信。AMD CPU 也可以连接到 IF。Infinity Fabric 可以连接不同芯片裸片上的模块。IF 还具备一致性，这意味着所有模块对内存内容看到的视图相同。

芯片的基础时钟频率为 1677 MHz，因此其峰值计算能力为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_23_10_fc737adb692c56.png)


其中，FMA 与 TFLOPS 的计算方式与式（23.16）相同。该架构灵活且可扩展，因此预计还会出现更多配置。


## 23.11 光线追踪架构

来源：《Real-Time Rendering》第4版，第23章；书页1039（PDF物理页1060）。本节止于下一页的“延伸阅读与资源”标题之前；章末延伸阅读另见《23.99_延伸阅读与资源》。

本节将简要介绍光线追踪硬件。我们不会列出这一主题的所有近期参考文献，而是提供一组线索，鼓励读者沿着这些线索进一步阅读。Schmittler等人[1571]于2002年开创了这一领域的研究，其重点是遍历与求交，而着色则使用固定功能单元计算。此后，Woop等人[1905]沿着这项工作继续研究，提出了一种具有可编程着色器的架构。

过去几年中，商业界对这一主题的兴趣显著增长。Imagination Technologies[1158]、LG Electronics[1256]和三星[1013]等公司都提出了各自的实时光线追踪硬件架构，这一事实便体现了这种增长。不过，在本书写作时，只有Imagination Technologies推出了商业产品。

这些架构具有若干共同特征。首先，它们通常采用基于轴对齐包围盒的层次包围体（BVH）。其次，它们往往通过降低光线与包围盒求交测试的精度来减少硬件复杂性（第22.7节）。最后，它们使用可编程核心来支持可编程着色，这在当今几乎已是一项必要条件。例如，Imagination Technologies通过增加一个光线追踪单元，扩展其传统芯片设计；该单元可以利用着色器核心来执行着色等工作。光线追踪单元由光线求交处理器和相干性引擎[1158]组成，后者将具有相似属性的光线收集起来并集中处理，以利用局部性来加快光线追踪。Imagination Technologies的架构还包含一个用于构建BVH的专用单元。

这一领域的研究仍在继续探索多个方向，包括通过降低精度来高效实现遍历[1807]、BVH的压缩表示[1045]，以及能源效率[929]。毫无疑问，还有更多研究工作有待开展。


## 延伸阅读与资源

来源：《Real-Time Rendering》第4版，第23章章末未编号内容；书页1040（PDF物理页1061）。本文件使用23.99作为归档文件名，原书此标题没有节号。

Akeley与Hanrahan[20]以及Hwu与Kirk[793]关于计算机图形学架构的课程讲义，是一组很好的学习资源。Kirk与Hwu的著作[903]也是了解如何使用CUDA在GPU上编程的优秀资料。每年的High-Performance Graphics（高性能图形学）和SIGGRAPH会议论文集，是了解新架构特性的良好来源。对于希望深入了解GPU细节的读者，Giesen的图形流水线之旅[530]是一份出色的在线资源。我们还建议有兴趣的读者参阅Hennessy与Patterson的著作[715]，以获得有关存储系统的详细信息。移动端渲染方面的信息分散在许多资料中。值得一提的是，《GPU Pro 5》收录了七篇关于移动端渲染技术的文章。
