# Real-Time Rendering 第04章 变换 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [4.1 基本变换](<Real-Time_Rendering_4th_中文/第04章/04.01_基本变换.md>)

- [4.2 特殊矩阵变换与运算](<Real-Time_Rendering_4th_中文/第04章/04.02_特殊矩阵变换与运算.md>)

- [4.3 四元数](<Real-Time_Rendering_4th_中文/第04章/04.03_四元数.md>)

- [4.4 顶点混合](<Real-Time_Rendering_4th_中文/第04章/04.04_顶点混合.md>)

- [4.5 变形](<Real-Time_Rendering_4th_中文/第04章/04.05_变形.md>)

- [4.6 几何缓存回放](<Real-Time_Rendering_4th_中文/第04章/04.06_几何缓存回放.md>)

- [4.7 投影](<Real-Time_Rendering_4th_中文/第04章/04.07_投影.md>)


## 章首导言

> 来源：Real-Time Rendering, Fourth Edition，书页 57—58（PDF 第78—79页）。

> “即使愤怒的向量转向，
> 在你沉睡的头颅周围成形，
> 也永远不必畏惧，
> 这可怜世界抽象风暴的暴行。”
>
> ——罗伯特·佩恩·沃伦（Robert Penn Warren）

变换是一种操作，它接收点、向量或颜色等实体，并以某种方式改变它们。对于计算机图形学从业者，掌握变换极其重要。借助变换，你可以对物体、光源和相机进行定位、改变形状并制作动画。你也可以确保所有计算都在同一个坐标系内进行，并以不同方式把物体投影到平面上。这只是变换能够完成的部分操作，但已经足以说明它在实时图形——实际上也在任何类型的计算机图形学——中的重要作用。

线性变换保持向量加法和标量乘法。具体来说：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_94f8fdc5847214.png)


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_bde89ef502e977.png)


例如，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_4a48234cfcfd0a.png) 是一种变换，它接收一个向量，并将向量的每个元素乘以五。要证明它是线性的，需要满足上述两个条件，即式（4.1）和式（4.2）。第一个条件成立，因为任意两个向量分别乘以五后相加，与先将向量相加再乘以五，结果相同。标量乘法条件，即式（4.2），也显然成立。这个函数称为缩放变换，因为它改变了物体的尺度（大小）。旋转变换是另一种线性变换，它使向量绕原点旋转。缩放和旋转变换，事实上所有针对三元素向量的线性变换，都可以用一个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_85783cb3a35041.png) 矩阵表示。

但是，这个尺寸的矩阵通常还不够大。对于三元素向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_45849b287c8af0.png)，像 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_f2953c092cf829.png) 这样的函数并不是线性的。对两个独立向量分别执行该函数，再将结果相加，会使 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_4f27683d88e780.png) 中的每个值被加上两次。向另一个向量加上固定向量实现的是平移，例如，使所有位置都移动相同的量。这是一种有用的变换，我们也希望把多种变换组合起来，例如先把物体缩小到原来的一半，再把它移动到另一个位置。若一直使用目前这些简单的函数形式，就很难方便地把它们组合起来。

可以使用仿射变换来组合线性变换和平移，它通常存储为一个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_f5eb9dabddfc94.png) 矩阵。仿射变换先执行线性变换，再执行平移。我们使用齐次记号表示四元素向量，以相同方式表示点和方向，均采用小写粗体字母。方向向量表示为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_d741c195815601.png)，点则表示为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_00_196ccab4e2dbf1.png)。本章将大量使用可下载的线性代数附录中解释的术语与运算，该附录可在 realtimerendering.com 找到。

所有平移、旋转、缩放、反射和错切矩阵都是仿射矩阵。仿射矩阵的主要特征是保持直线的平行性，但不一定保持长度与角度。仿射变换也可以是将若干个独立仿射变换依次连接得到的任意序列。

本章首先介绍最必要、最基本的仿射变换。这一节可以视为简单变换的“参考手册”。随后介绍更特殊的矩阵，再讨论并描述四元数这一强大的变换工具。接着介绍顶点混合与变形，这是两种简单但有效的网格动画表达方式。最后介绍投影矩阵。表 4.1 汇总了其中大多数变换及其记号、功能和性质；其中，正交矩阵指的是逆矩阵等于转置矩阵的矩阵。

变换是操作几何体的基本工具。大多数图形应用程序编程接口都允许用户设置任意矩阵，有时也可以使用矩阵运算库，其中实现了本章讨论的许多变换。不过，理解函数调用背后实际使用的矩阵及其相互作用，仍然很有价值。知道一次函数调用之后矩阵做了什么，只是起点；理解矩阵本身的性质，会让你走得更远。例如，这种理解能使你判断当前处理的是否为正交矩阵；由于它的逆等于转置，求逆便可以更快。这样的知识能够帮助你加速代码。


## 4.1 基本变换

来源：《Real-Time Rendering, Fourth Edition》，书页 58—69（PDF 物理页 79—90）。

本节介绍最基本的变换，包括平移、旋转、缩放、错切、变换的串接、刚体变换、法线变换（它可不那么“普通”），以及逆矩阵的计算。对于有经验的读者，本节可以作为简单变换的参考手册；对于初学者，则可以作为这一主题的入门介绍。这些内容是阅读本章其余部分以及本书其他章节所必需的背景知识。我们从最简单的变换——平移——开始。

| 记号 | 名称 | 特性 |
| --- | --- | --- |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_0bd7b63528d7ef.png) | 平移矩阵 | 移动一个点。仿射变换。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_3e54f01da18e9f.png) | 旋转矩阵 | 绕 x 轴旋转 ρ 弧度。绕 y 轴和 z 轴的旋转采用类似记号。正交且仿射。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png) | 旋转矩阵 | 任意旋转矩阵。正交且仿射。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_92adf64347c69e.png) | 缩放矩阵 | 根据 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d6125d7522103b.png)，沿 x、y、z 各轴缩放。仿射变换。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_b81375d80d12f8.png) | 错切矩阵 | 相对于分量 j，以因子 s 对分量 i 进行错切。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_8235ba8cfed27a.png)。仿射变换。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_cad80c7bf240e3.png) | 欧拉变换 | 由航向（偏航）、俯仰、翻滚三个欧拉角给出的朝向矩阵。正交且仿射。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_69360811bbe79d.png) | 正交投影 | 平行投影到某个平面或某个体积中。仿射变换。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d930c01513035a.png) | 透视投影 | 以透视方式投影到平面或某个体积中。 |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_65fde732f6b7f2.png) | slerp 变换 | 根据四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_36725ae2385c6f.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d311fd6885486d.png) 及参数 t，生成插值四元数。 |

**表 4.1.** 本章所讨论的大部分变换的汇总。

### 4.1.1 平移

从一个位置到另一个位置的变化由平移矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_794d9654f171ca.png) 表示。这个矩阵将实体平移一个向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a6bae746a8e020.png)。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_794d9654f171ca.png) 由下面的公式（4.3）给出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_971f36df203caf.png)


图 4.1 展示了平移变换作用的一个例子。很容易证明，将点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_f65d9eec048529.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_0bd7b63528d7ef.png) 相乘，会得到新点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_3939bfc8921352.png)，这显然是一次平移。注意，向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_69987ab0b5ebb1.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_794d9654f171ca.png) 相乘后保持不变，因为方向向量不能被平移。相比之下，其余仿射变换会同时影响点和向量。平移矩阵的逆为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_f75b813856842e.png)，也就是将向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_759a289713bfda.png) 取负。


![图 4.1 平移变换](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_1_4.1.png)

**图 4.1.** 左侧正方形经过平移矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_95c3eb12bbb592.png) 的变换，向右移动 5 个距离单位，向上移动 2 个距离单位。

这里还应提到，计算机图形学中有时会采用另一种同样有效的记法：把平移向量放在矩阵的最下面一行。例如，DirectX 就采用这种形式。在这种记法下，矩阵的顺序会反过来，也就是说，应用顺序从左向右阅读。由于向量写成行，这种记法中的向量和矩阵被称为行主序形式。本书采用列主序形式。不论采用哪一种，都只是记法上的差异。当矩阵存储在内存中时，16 个数值中的最后 4 个是三个平移值，后面跟着一个 1。

### 4.1.2 旋转

旋转变换使向量（位置或方向）绕经过原点的给定轴旋转给定角度。与平移矩阵一样，它也是刚体变换，也就是说，它保持变换后各点之间的距离，并保持左右手性（即永远不会使左、右两侧互换）。显然，这两类变换在计算机图形学中非常适合用来确定物体的位置和朝向。朝向矩阵是与相机视图或物体相关联的旋转矩阵，它定义了相机或物体在空间中的朝向，即向上和向前的方向。

二维旋转矩阵很容易推导。假设有向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a3ea095aec4e6d.png)，将其参数化为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_1e5951841df72e.png)。如果将该向量逆时针旋转 φ 弧度，就得到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_c43fe9fb2a60e3.png)。它可以改写为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6fbf6cd9a3a8e7.png)


这里使用了和角公式来展开 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_74acd06cd67d49.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_b6e41efda8d624.png)。在三维空间中，常用的旋转矩阵有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_afc563f7ba4a5c.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_298225d2398313.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_ae95d4ea077cb0.png)，它们分别使实体绕 x、y 和 z 轴旋转 φ 弧度。公式（4.5）—（4.7）给出了这些矩阵：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_32824bbf8b4f13.png)


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_40e8b79c80285f.png)


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d810e96e05892c.png)


从一个 4×4 矩阵中删除最下面一行和最右边一列，就得到一个 3×3 矩阵。对于任意绕某个轴旋转 φ 弧度的 3×3 旋转矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png)，它的迹（即矩阵对角线元素之和）与所选旋转轴无关，并按下式计算 [997]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_ae614288d38e97.png)


旋转矩阵的作用可参见书页 65 的图 4.4。旋转矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_37392dcfd595b7.png) 的特征，除了绕轴 i 旋转 φ 弧度之外，还在于它使旋转轴 i 上的所有点保持不变。注意，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png) 也将用于表示绕任意轴的旋转矩阵。上面给出的各坐标轴旋转矩阵可以组成三个变换的序列，实现任意轴旋转。第 4.2.1 节将讨论这一过程。直接执行绕任意轴旋转的方法见第 4.2.4 节。

所有旋转矩阵的行列式都为 1，而且都是正交矩阵。任意多个这类变换串接起来，也满足这些性质。求逆还有另一种方法：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_4378f27e7c9783.png)，即绕同一根轴朝相反方向旋转。

**示例：绕某个点旋转。** 假设我们希望以某个给定点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_789e716ab03779.png) 为旋转中心，使物体绕 z 轴旋转 φ 弧度。应采用什么变换？图 4.2 描绘了这种情况。绕某点旋转的特点是该点本身不受旋转影响，因此变换首先要平移物体，使 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_789e716ab03779.png) 与原点重合，这由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_63920a92d993ba.png) 完成。接下来执行实际的旋转：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_ae95d4ea077cb0.png)。最后，必须使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_c7ce39ad928b75.png) 将物体平移回原来的位置。所得变换 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_19396f252dcb12.png) 为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_7238cb23fdd8b2.png)


注意上面各矩阵的顺序。□


![图 4.2 绕指定点旋转](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_1_4.2.png)

**图 4.2.** 绕指定点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_789e716ab03779.png) 旋转的示例。

### 4.1.3 缩放

缩放矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12b00cdfc9eaaa.png) 分别沿 x、y、z 方向，以因子 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_7e1d1d24363f1c.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_35d6144dc285c7.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_4323ba07725e86.png) 对实体进行缩放。这意味着缩放矩阵可以用来放大或缩小物体。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_079d71efe495a8.png)（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_ab865f2729d8b0.png)）越大，缩放后实体在相应方向上就越大。将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d6125d7522103b.png) 的任一分量设为 1，自然就不会改变该方向上的尺度。公式（4.10）给出了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_eef8c14b51948b.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6f8ba6408665cb.png)


书页 65 的图 4.4 展示了缩放矩阵的作用。当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_8d73457123b6db.png) 时，称这种缩放操作为均匀缩放，否则称为非均匀缩放。有时也用各向同性缩放和各向异性缩放来代替均匀缩放和非均匀缩放。其逆矩阵为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d325ef15e2360f.png)。

使用齐次坐标时，创建均匀缩放矩阵的另一种有效方法是修改矩阵中位置 (3,3) 处的元素，即右下角元素。这个值影响齐次坐标的 w 分量，因此会缩放经该矩阵变换的点（不包括方向向量）的每个坐标。例如，要均匀放大 5 倍，可以把缩放矩阵中 (0,0)、(1,1) 和 (2,2) 处的元素设为 5，也可以将 (3,3) 处的元素设为 1/5。实现这一操作的两种不同矩阵如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_33d1b81343726e.png)


与使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_eef8c14b51948b.png) 进行均匀缩放不同，使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_f9991ea2623907.png) 后必须始终进行齐次化处理。这可能效率不高，因为齐次化过程涉及除法；如果右下角（位置 (3,3)）的元素为 1，就不需要除法。当然，如果系统总是执行这种除法而不检查该值是否为 1，那么就没有额外开销。

当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d6125d7522103b.png) 的一个或三个分量为负值时，便得到一种反射矩阵，也称镜像矩阵。如果只有两个缩放因子为 -1，则会产生 π 弧度的旋转。应当注意，旋转矩阵与反射矩阵串接，所得仍是反射矩阵。因此，下式是一个反射矩阵：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_dc50d5de33b4bb.png)


检测到反射矩阵后，通常需要对它进行特殊处理。例如，一个顶点按逆时针顺序排列的三角形，经反射矩阵变换后，会变成顺时针顺序。这种顺序变化可能导致光照和背面剔除出错。要检测给定矩阵是否具有某种反射作用，可以计算该矩阵左上角 3×3 元素的行列式。如果其值为负，矩阵就具有反射性。例如，公式（4.12）中矩阵的行列式为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_47ff81ff1a1c40.png)。

**示例：沿特定方向缩放。** 缩放矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_eef8c14b51948b.png) 只能沿 x、y 和 z 轴缩放。如果要沿其他方向缩放，就需要复合变换。假设要沿单位正交、构成右手坐标系的向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_f8546fe5d73786.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a8b706b3c0ff6f.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_32ee4d00135955.png) 所确定的轴进行缩放。首先构造如下矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6495106d6d7709.png)，用于换基：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_bbf740c7e8ece4.png)


思路是使这三根轴所给出的坐标系与标准坐标轴重合，再使用标准缩放矩阵，最后变换回去。第一步通过乘以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6495106d6d7709.png) 的转置来完成，这个转置也就是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6495106d6d7709.png) 的逆。然后执行实际缩放，接着再变换回去。公式（4.14）给出了这一变换：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_2cf2690081950b.png)


□

### 4.1.4 错切

另一类变换是错切矩阵。例如，游戏中可以用它来扭曲整个场景，营造迷幻效果，或以其他方式使模型的外观变形。共有六种基本错切矩阵，分别记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_43a95d38b13856.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_cd756061b4a0dd.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_274e01ce451aed.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_05e1ada2c3c4db.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_4e6fe7a6a1a267.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_c293cba2d86595.png)。第一个下标表示被错切矩阵改变的坐标，第二个下标表示用于产生错切的坐标。公式（4.15）给出了错切矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_cd756061b4a0dd.png) 的例子。注意，可以利用下标找到参数 s 在下面矩阵中的位置：x（其数值索引为 0）确定第 0 行，z（其数值索引为 2）确定第 2 列，因此 s 就位于那里：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_7b32d753c88263.png)


该矩阵与点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_789e716ab03779.png) 相乘，得到点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_120d06fc477df8.png)。图 4.3 用单位正方形直观地展示了这一效果。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_b81375d80d12f8.png)（相对于第 j 个坐标对第 i 个坐标进行错切，其中 i ≠ j）的逆由反方向的错切生成，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6c934f49d2d995.png)。


![图 4.3 错切单位正方形](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_1_4.3.png)

**图 4.3.** 使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_cd756061b4a0dd.png) 对单位正方形进行错切的效果。变换不影响 y 值和 z 值，而新的 x 值等于原来的 x 值加上 s 与 z 值的乘积，从而使正方形倾斜。这种变换保持面积不变，图中带线条的两块区域面积相等，体现了这一点。

也可以使用一种略有不同的错切矩阵：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_17b915111f3bcf.png)


不过，这里的两个下标都表示要由第三个坐标进行错切的坐标。这两种描述方式之间的关系为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_0cbd745467d228.png)，其中 k 是第三个坐标的索引。采用哪一种矩阵取决于个人偏好。最后应当指出，任意错切矩阵的行列式均为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_61823d3a5869ba.png)，因此这种变换保持体积不变，图 4.3 也说明了这一点。

### 4.1.5 变换的串接

由于矩阵乘法不满足交换律，矩阵出现的顺序十分重要。因此，我们说变换的串接依赖于顺序。

以两个矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_eef8c14b51948b.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png) 为例，说明这种顺序依赖性。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d8d5c95892804f.png) 将 x 分量缩放为原来的 2 倍，将 y 分量缩放为原来的 0.5 倍。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_847ab617f00e7e.png) 使物体绕 z 轴逆时针旋转 π/6 弧度（在右手坐标系中，该轴从本书页面向外指）。这两个矩阵可以按两种顺序相乘，而所得结果完全不同。图 4.4 展示了这两种情况。


![图 4.4 矩阵相乘的顺序依赖性](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_1_4.4.png)

**图 4.4.** 本图说明矩阵相乘时的顺序依赖性。上排先应用旋转矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_847ab617f00e7e.png)，再应用缩放 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_92adf64347c69e.png)，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_2560a407079b3b.png)。因此复合矩阵为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_b2369edc21cca8.png)。下排以相反顺序应用这些矩阵，得到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_87f4166c306769.png)。两种结果明显不同。对于任意矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_264a94c53ec37f.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_f434e93c7307a5.png)，一般有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_c99852f82b3e16.png)。

把一系列矩阵串接成单个矩阵，显然是为了提高效率。例如，设想一个含有数百万个顶点的游戏场景，场景中的所有物体都必须先缩放，再旋转，最后平移。此时，无须将所有顶点分别与这三个矩阵相乘，而是先把三个矩阵串接成一个矩阵，再将这个矩阵应用于顶点。该复合矩阵为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d60a9d6facebd7.png)。注意这里的顺序：缩放矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_eef8c14b51948b.png) 应首先作用于顶点，因此它在复合式中位于右侧。这种顺序意味着 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_030d04753fffe5.png)，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_789e716ab03779.png) 为待变换的点。顺便一提，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12ddf37ed1eeb6.png) 正是场景图系统通常采用的顺序。

需要注意，虽然矩阵串接依赖于顺序，但矩阵可以按需要分组。例如，对于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_1299a7360fe785.png)，假设希望只计算一次刚体运动变换 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e906eb1bff2254.png)，那么可以将这两个矩阵归为一组，写成 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_d7f28597ac1460.png)，并用中间结果替换它们。因此，矩阵串接满足结合律。
### 4.1.6 刚体变换

当一个人拿起一个固体物体，例如从桌上拿起一支笔，将其移到另一个位置，例如衬衫口袋中时，变化的只有物体的朝向和位置，而物体的形状通常不受影响。这种仅由平移和旋转串接而成的变换称为刚体变换。它的特点是保持长度、角度和左右手性不变。

任意刚体矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_19396f252dcb12.png) 都可以写成平移矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_0bd7b63528d7ef.png) 与旋转矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png) 的串接。因此，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_19396f252dcb12.png) 具有公式（4.17）所示的形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_11f35afeae705f.png)


![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_19396f252dcb12.png) 的逆按下式计算：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_364f14f02dee3e.png)。因此，要计算逆矩阵，可以将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png) 左上角的 3×3 矩阵转置，并将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_794d9654f171ca.png) 中的平移值变号。再将这两个新矩阵按相反顺序相乘，即得到逆矩阵。计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_19396f252dcb12.png) 的逆还有另一种方式：使用下面的记法表示 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_12fc0c0941fa56.png)（将其写成 3×3 矩阵）和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_19396f252dcb12.png)（该记法在书页 6、公式（1.2）处有说明）：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a82690a3ad1f4a.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_1a7a38f04ea02b.png) 表示旋转矩阵的第一列（即逗号表示从 0 到 2 的任意值，而第二个下标为 0），![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_48699154526e80.png) 是该列式矩阵的第一行。注意，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a33f957fc5c97d.png) 是一个所有分量都为零的 3×1 列向量。经过一些计算，可以得到公式（4.19）所示的逆矩阵表达式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_c6b48d35f6f429.png)


![图 4.5 确定相机朝向的几何关系](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_1_4.5.png)

**图 4.5.** 计算相机朝向变换所涉及的几何关系：相机位于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_40848f2eb71fad.png)，向上向量为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_4c6a337b3ab626.png)，要使它朝向点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6129e5f9e9210f.png)。为此，需要计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a15c7692963708.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a581829a2f07f0.png)。

**示例：确定相机朝向。** 图形学中的一个常见任务是调整相机朝向，使它看向某个位置。这里介绍 `gluLookAt()` 所执行的操作，该函数来自 OpenGL 实用库（OpenGL Utility Library，简称 GLU）。尽管如今已经很少使用这个函数调用本身，这项任务仍然很常见。假设相机位于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_40848f2eb71fad.png)，希望它看向目标 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_6129e5f9e9210f.png)，并给定相机的向上方向 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_4c6a337b3ab626.png)，如图 4.5 所示。我们希望计算由三个向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_089c8085c97b24.png) 构成的一组基。首先计算视向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_2b1a2a0b94d983.png)，即从目标指向相机位置的归一化向量。然后，指向“右方”的向量可计算为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_db41ce8c075f27.png)。向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_4c6a337b3ab626.png) 往往不能保证精确地指向上方，因此最终的向上向量通过另一次叉积得到，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_84b733fa230cf7.png)。由于构造时已使 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a581829a2f07f0.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png) 归一化且相互垂直，所以能保证 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a15c7692963708.png) 也是归一化的。对于接下来要构造的相机变换矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_264a94c53ec37f.png)，思路是先平移所有内容，使相机位置位于原点 (0,0,0)，然后换基，使 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png) 与 (1,0,0) 对齐，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a15c7692963708.png) 与 (0,1,0) 对齐，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a581829a2f07f0.png) 与 (0,0,1) 对齐。实现方式如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_cf13f5f37332fa.png)


注意，将平移矩阵与换基矩阵串接时，平移 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_122b4338d95f6d.png) 位于右侧，因为它应当首先应用。记住 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a15c7692963708.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a581829a2f07f0.png) 各分量位置的一种办法如下。我们希望 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png) 变成 (1,0,0)，因此将换基矩阵与 (1,0,0) 相乘时，可以看出矩阵的第一行必须由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png) 的各元素构成，因为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a25d28d2fe59f0.png)。此外，第二行和第三行必须由垂直于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_e2aea9477eb220.png) 的向量构成，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_20617d2381178c.png)。对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a15c7692963708.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_a581829a2f07f0.png) 也采用相同思路，就能得到上面的换基矩阵。□

### 4.1.7 法线变换

一个矩阵可以一致地用于变换点、直线、三角形以及其他几何对象。同一个矩阵也可以变换沿这些直线或沿三角形表面的切向量。然而，对于一种重要的几何属性——表面法线（以及用于顶点光照的法线）——这个矩阵却不一定适用。图 4.6 展示了采用同一个矩阵可能出现的情况。


![图 4.6 法线的正确与错误变换](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_1_4.6.png)

**图 4.6.** 左图是原始几何对象，即从侧面观察的三角形及其法线。中图展示了模型沿 x 轴缩放为 0.5 倍，同时法线也使用同一矩阵变换时的结果。右图展示了法线的正确变换。

正确的方法并不是乘以矩阵本身，而是使用该矩阵的伴随矩阵的转置 [227]。伴随矩阵的计算方法见本书在线的线性代数附录。伴随矩阵总是存在。法线在变换后不一定仍为单位长度，因此通常需要将其归一化。

关于如何变换法线，传统答案是计算逆矩阵的转置 [1794]。这种方法通常有效。不过，并不需要计算完整的逆矩阵，而且有时逆矩阵无法求出。逆矩阵等于伴随矩阵除以原矩阵的行列式。如果行列式为零，矩阵就是奇异矩阵，其逆不存在。

即使只是计算完整 4×4 矩阵的伴随矩阵，也可能代价很高，而且通常没有必要。由于法线是向量，平移不会影响它。此外，大部分建模变换都是仿射变换，它们不改变传入齐次坐标的 w 分量，也就是说，它们不执行投影。在这些（常见的）情况下，进行法线变换时，只需要计算左上角 3×3 部分的伴随矩阵。

通常，甚至连这一伴随矩阵也无须计算。假设已知变换矩阵完全由平移、旋转和均匀缩放操作串接而成（没有拉伸或压扁）。平移不影响法线。均匀缩放因子只会改变法线的长度。剩下的是一系列旋转，而它们的合成结果总是某种旋转，不会有其他效果。逆矩阵的转置可以用于变换法线。旋转矩阵的特征是其转置就是其逆。代入法线变换后，两次转置（或两次求逆）就得到原来的旋转矩阵。将这些结论综合起来，在这些情况下，也可以直接用原变换本身来变换法线。

最后，并不总是需要对所得法线进行完整的重新归一化。如果只串接平移和旋转，法线经矩阵变换后长度不会改变，因此不需要重新归一化。如果还串接了均匀缩放，那么可以使用总缩放因子（如果它已知，或者可以提取出来，见第 4.2.3 节）直接归一化所得法线。例如，如果已知一系列缩放使物体放大为原来的 5.2 倍，那么直接用这个矩阵变换得到的法线，只需除以 5.2 就能重新归一化。另一种方法是将原矩阵左上角的 3×3 部分一次性除以该缩放因子，从而创建一个能够产生归一化结果的法线变换矩阵。

注意，对于在变换后从三角形推导表面法线的系统（例如使用三角形两条边的叉积），法线变换就不成问题。切向量在本质上不同于法线，始终可以直接使用原矩阵进行变换。

### 4.1.8 逆矩阵的计算

许多情况下都需要逆矩阵，例如在不同坐标系之间来回转换时。根据已知的变换信息，可以使用以下三种方法之一计算矩阵的逆：

- 如果矩阵是单个变换，或者是参数已知的一系列简单变换，那么通过“将参数取逆”并反转矩阵顺序，就能轻松计算逆矩阵。例如，如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_31350769f6ca12.png)，那么 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_0df230e8f058b8.png)。这种方法很简单，而且保持了变换的精度，这在渲染巨大世界时十分重要 [1381]。
- 如果已知矩阵为正交矩阵，那么 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_01_745e7ef88be6e9.png)，即转置就是逆。任意旋转序列的合成结果仍是旋转，因此也是正交的。
- 如果没有任何已知信息，则可以使用伴随矩阵法、克拉默法则、LU 分解或高斯消元来计算逆矩阵。一般更倾向于采用克拉默法则和伴随矩阵法，因为它们的分支操作更少；在现代体系结构上，应尽量避免 `if` 测试。有关如何使用伴随矩阵对法线进行逆变换，参见第 4.1.7 节。

在优化时，也可以考虑求逆的目的。例如，如果逆矩阵将用于变换向量，那么通常只需要对矩阵左上角的 3×3 部分求逆（见上一节）。


## 4.2 特殊矩阵变换与运算

来源：*Real-Time Rendering, 4th Edition*，书页 70—76（PDF 第 91—97 页）。

本节将介绍并推导几种对实时图形学至关重要的矩阵变换与运算。首先，我们介绍欧拉变换（以及从中提取参数的方法），它是一种描述朝向的直观方式。随后，我们简要讨论如何从单个矩阵中还原出一组基本变换。最后，推导一种使实体绕任意轴旋转的方法。

### 4.2.1 欧拉变换

这种变换提供了一种直观的方法，可以构造一个矩阵，使你自己（即相机）或任何其他实体朝向某个方向。它的名称来自伟大的瑞士数学家莱昂哈德·欧拉（Leonhard Euler，1707—1783）。

首先，必须确定某种默认观察方向。最常见的做法是沿负 z 轴观察，头顶朝向 y 轴，如图 4.7 所示。欧拉变换是三个矩阵的乘积，即图中所示的三个旋转。更正式地说，将该变换记为 **E**，则它由式（4.21）给出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_fa41e3aff52105.png)


这些矩阵的顺序有 24 种不同的选择方式 [1636]；这里介绍这一种，是因为它经常被使用。由于 **E** 是旋转的复合，因此它显然也是正交矩阵。所以，其逆可以表示为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_0187960490f452.png)，当然，直接使用 **E** 的转置会更简单。

欧拉角 h、p 和 r 表示航向（head）、俯仰（pitch）和滚转（roll）应当按什么顺序、分别绕各自的轴旋转多少。有时这些角度都被称作“滚转”，例如，这里所说的“航向”就是“y 滚转”，“俯仰”就是“x 滚转”。另外，“head”有时也称为“yaw”（偏航），例如在飞行模拟中就是如此。

这种变换很直观，因此容易用普通人能理解的语言来讨论。例如，改变航向角会使观察者像表示“不”那样摇头；改变俯仰角会使其点头；而滚转则会使其把头向侧面倾斜。我们谈论的是改变航向、俯仰和滚转，而不是绕 x、y、z 轴旋转。注意，这种变换不仅可以确定相机的朝向，也可以确定任意物体或实体的朝向。这些变换既可以使用世界空间的全局坐标轴来执行，也可以相对于局部参考系来执行。


![图 4.7 欧拉变换与航向、俯仰、滚转](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_2_4.7.png)

图 4.7. 欧拉变换，以及它与改变航向、俯仰和滚转角的方式之间的关系。图中显示了默认观察方向：沿负 z 轴观察，向上方向沿 y 轴。

需要注意，有些介绍欧拉角的资料将 z 轴作为初始向上方向。这种区别纯粹是记号约定的变化，但可能造成混淆。在计算机图形学中，对于如何看待世界、进而如何构建内容，存在两种不同约定：y 向上或 z 向上。大多数制造流程，包括 3D 打印，都把世界空间中的 z 方向视为向上；航空器和船舶则把 −z 视为向上。建筑和 GIS 通常采用 z 向上，因为建筑平面图或地图是以 x 和 y 表示的二维图。与媒体相关的建模系统往往把世界坐标中的 y 方向视为向上，这与计算机图形学中始终采用的相机屏幕向上方向的描述方式一致。这两种世界向上向量的选择之间只差一次 90° 的旋转（也可能还需要一次反射），但若不知道采用的是哪一种约定，就可能出现问题。本书除非另有说明，都使用 y 向上的世界方向约定。

我们还想指出，相机在其观察空间中的向上方向，与世界的向上方向并没有什么必然关系。将头侧倾，看到的景象就会倾斜，其在世界空间中的向上方向也会与世界的向上方向不同。再举一个例子，假设世界采用 y 向上，而相机径直向下看地面，形成鸟瞰视角。这种朝向意味着相机向前俯仰了 90°，因而它在世界空间中的向上方向是 (0,0,−1)。在这种朝向下，相机的向上方向没有 y 分量，而是把世界空间中的 −z 视为向上；但按照定义，在观察空间中，“y 向上”仍然成立。

虽然欧拉角适合用于小角度变化或观察者朝向，但它也有一些其他的严重局限。将两组欧拉角结合起来处理很困难。例如，从一组角度插值到另一组，并不是简单地对每个角度分别插值就行。事实上，两组不同的欧拉角可能给出相同的朝向，因此它们之间的任何插值都不应使物体发生旋转。这些都是值得探索其他朝向表示方法的原因，例如本章稍后讨论的四元数。使用欧拉角还可能遇到一种称为万向节锁的现象，接下来的 4.2.2 节将对此进行解释。

### 4.2.2 从欧拉变换中提取参数

在某些情况下，能够从正交矩阵中提取欧拉参数 h、p 和 r 的过程很有用。式（4.22）给出了这一过程所依据的关系：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_0761720778b75e.png)


这里我们不再使用 4×4 矩阵，而改用 3×3 矩阵，因为后者包含了旋转矩阵所需的全部信息。也就是说，对应的 4×4 矩阵的其余元素始终为零，只有右下角的元素为一。

将式（4.22）中的三个旋转矩阵复合，得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_a1caa171314af8.png)


由此可以明显看出，俯仰参数满足 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_ba450d9f19d6ae.png)。另外，用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_dc4e2c09003541.png) 除以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_0fb18820a5f481.png)，并类似地用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_7f478fa0b8bc61.png) 除以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_dd9afa0b9c4529.png)，便得到以下用于提取航向和滚转参数的等式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_27fc3e74fc1a48.png)


因此，可使用函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_756d97ae05cf46.png)（见第 1 章第 8 页），按照式（4.25）从矩阵 **E** 中提取欧拉参数 h（航向）、p（俯仰）和 r（滚转）：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_ec1f58791e4f45.png)


不过，还有一种特殊情况需要处理。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_692fb5a81933a3.png)，就会发生万向节锁（4.2.2 节），旋转角 r 和 h 将绕同一根轴旋转（但方向可能不同，这取决于旋转角 p 是 −π/2 还是 π/2），所以只需求出一个角度。如果任意设定 h=0 [1769]，便得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_a8ad2451781065.png)


由于 p 不影响第一列的值，因此在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_692fb5a81933a3.png) 时，可以使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_143d4f75e1d22f.png)，从而得到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_264ddd3c28aed6.png)。

注意，根据反正弦函数的定义，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_78cecf44a1cc8d.png)。这意味着，如果创建 **E** 时采用的 p 值超出这个区间，就无法提取出原始参数。h、p 和 r 不唯一，意味着不止一组欧拉参数能够产生相同的变换。有关欧拉角转换的更多内容，可参阅 Shoemake 于 1994 年发表的文章 [1636]。上面概述的简单方法可能导致数值不稳定问题；牺牲一些速度可以避免这一问题 [1362]。

使用欧拉变换时，可能发生一种称为*万向节锁*的现象 [499, 1633]。当旋转使系统失去一个自由度时，就会出现这种情况。例如，假设变换顺序是 x/y/z。考虑只绕 y 轴旋转 π/2，这是所执行的第二次旋转。这样会使局部 z 轴转到与原来的 x 轴对齐，从而使最后绕 z 轴的旋转变得多余。

从数学上看，我们已经在式（4.26）中见过万向节锁，当时假定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_692fb5a81933a3.png)，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_66916b9cbae6e3.png)，其中 k 是整数。取这样的 p 值时，我们就失去了一个自由度，因为矩阵只依赖于一个角度，即 r+h 或 r−h（但不会同时依赖两者）。

尽管建模系统中的欧拉角通常按 x/y/z 顺序给出，即分别绕各个局部坐标轴旋转，但其他顺序也可行。例如，动画中会采用 z/x/y，而动画和物理学中都会采用 z/x/z。这些都是指定三次独立旋转的有效方式。最后一种顺序 z/x/z 对某些应用可能更好，因为只有绕 x 轴旋转 π 弧度（半圈）时才会发生万向节锁。不存在能够避免万向节锁的完美顺序。尽管如此，欧拉角仍然被广泛使用，因为动画师喜欢通过曲线编辑器来指定角度随时间的变化方式 [499]。

**示例：约束变换。** 想象你手里拿着一把夹住螺栓的（虚拟）扳手。为了把螺栓拧到位，你必须使扳手绕 x 轴旋转。现在假设输入设备（鼠标、VR 手套、空间球等）为扳手的运动提供了一个旋转矩阵，也就是一次旋转。问题在于，把这个变换直接应用于扳手很可能不对，因为扳手应该只绕 x 轴旋转。要把输入变换 **P** 限制为绕 x 轴的旋转，只需用本节所述的方法提取欧拉角 h、p 和 r，然后创建一个新矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_dce889f7071e24.png)。这就是所要寻找的、使扳手绕 x 轴旋转的变换（前提是 **P** 此时包含这种运动）。□

### 4.2.3 矩阵分解

到目前为止，我们一直假设自己知道所使用的变换矩阵的来源和形成过程。但实际情况往往并非如此。例如，某个经过变换的物体所关联的信息，可能只有一个复合矩阵。从复合矩阵中还原各种变换的工作称为*矩阵分解*。

需要还原一组变换的原因有很多，其用途包括：

- 只提取物体的缩放因子。
- 找出某个特定系统所需的变换。（例如，一些系统可能不允许使用任意的 4×4 矩阵。）
- 判断模型是否只经历了刚体变换。
- 当只有物体的矩阵可用时，在动画关键帧之间进行插值。
- 从旋转矩阵中去除错切。

我们已经介绍过两种分解：从刚体变换中求出平移矩阵和旋转矩阵（4.1.6 节），以及从正交矩阵中求出欧拉角（4.2.2 节）。

正如我们已经看到的，还原平移矩阵很简单，因为只需要 4×4 矩阵最后一列中的元素。通过检查矩阵的行列式是否为负，还可以判断是否发生过反射。要将旋转、缩放和错切分离出来，则需要付出更多努力。

幸运的是，已有几篇文章讨论这个主题，网上也有可用代码。Thomas [1769] 和 Goldman [552, 553] 分别针对不同类别的变换提出了略有差异的方法。Shoemake [1635] 针对仿射矩阵改进了这些技术：他的算法与参考系无关，并尝试通过矩阵分解获得刚体变换。

### 4.2.4 绕任意轴旋转

有时，能够使实体绕任意轴旋转某个角度的过程会很方便。假设旋转轴 **r** 已归一化，需要创建一个绕 **r** 旋转 α 弧度的变换。

为此，我们首先变换到一个空间，使我们想要绕其旋转的轴成为 x 轴。这通过一个称为 **M** 的旋转矩阵来实现。随后执行实际旋转，再用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_d0882072c7fe2c.png) 变换回去 [314]。图 4.8 展示了这个过程。


![图 4.8 绕任意轴旋转](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_2_4.8.png)

图 4.8. 绕任意轴 **r** 的旋转，通过寻找由 **r**、**s** 和 **t** 构成的标准正交基来实现。随后将这组基与标准基对齐，使 **r** 与 x 轴对齐。在那里执行绕 x 轴的旋转，最后再变换回去。

为了计算 **M**，需要找到另外两根轴，使它们与 **r** 以及彼此之间都满足单位正交关系。我们集中考虑如何找到第二根轴 **s**，因为第三根轴 **t** 就是第一根轴与第二根轴的叉积，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_be3ad444bdeea9.png)。一种数值稳定的做法是，找出 **r** 中绝对值最小的分量，并将其置为 0。交换剩余两个分量，然后将其中第一个取反（事实上，两个非零分量中的任意一个都可以取反）。用数学形式表示如下 [784]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_8419121a259f72.png)


这能保证 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_b7d65de515331d.png) 与 **r** 正交（垂直），并且 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_eea35ced2d4640.png) 是一组标准正交基。Frisvad [496] 提出了一种代码中完全没有分支的方法，速度更快，但精度较低。Max [1147] 和 Duff 等人 [388] 提高了 Frisvad 方法的精度。无论采用哪种技术，都使用这三个向量来构造旋转矩阵：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_b13d83f04acbc8.png)


这个矩阵将向量 **r** 变换到 x 轴，将 **s** 变换到 y 轴，将 **t** 变换到 z 轴。因此，绕归一化向量 **r** 旋转 α 弧度的最终变换为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_1d60720a8c4f27.png)


用文字来说，这意味着首先进行变换，使 **r** 成为 x 轴（使用 **M**）；然后绕这根 x 轴旋转 α 弧度（使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_188f70f29e37e0.png)）；最后使用 **M** 的逆变换回去。由于 **M** 是正交矩阵，此处它的逆就是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_3d230eb1193d35.png)。

Goldman [550] 提出了另一种绕任意归一化轴 **r** 旋转 φ 弧度的方法。这里仅给出他的变换：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_02_31c9ca799fd0c1.png)


在 4.3.2 节中，我们还将介绍一种使用四元数解决这个问题的方法。该节也包含针对相关问题的更高效算法，例如从一个向量旋转到另一个向量。


## 4.3 四元数

来源：《Real-Time Rendering, Fourth Edition》，第 4 章第 4.3 节，书页 76—84（PDF 物理页 97—105）。

虽然威廉·罗恩·汉密尔顿爵士（Sir William Rowan Hamilton）早在 1843 年就发明了四元数，作为复数的扩展，但直到 1985 年，Shoemake [1633] 才将它们引入计算机图形学领域。注1 四元数用于表示旋转和朝向。它们在若干方面优于欧拉角和矩阵。任何三维朝向都可以表示为绕某个特定轴的一次旋转。给定这种轴与角的表示形式，将其转换为四元数或从四元数转换回来都很直接，而欧拉角的双向转换都较为困难。四元数可以对朝向进行稳定、匀速的插值，欧拉角却难以很好地做到这一点。

复数具有实部和虚部。每个复数都由两个实数表示，其中第二个实数乘以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_47bddc0a7a0c8c.png)。类似地，四元数具有四个部分。前三个值与旋转轴密切相关，而旋转角会影响全部四个部分（第 4.3.2 节将进一步讨论）。每个四元数由四个实数表示，每个实数对应不同的部分。由于四元数有四个分量，我们选择将其表示为向量，但为了区分，在其上方加一个帽号：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)。我们先介绍四元数的一些数学背景，再利用这些知识构造各种有用的变换。

### 4.3.1 数学背景

我们从四元数的定义开始。

**定义。** 四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 可以用以下几种彼此等价的方式定义。


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_3348be9e2af9f9.png)


变量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_57c7a152b5541d.png) 称为四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 的实部。虚部为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_bc8503a4091dd4.png)，而 i、j 和 k 称为虚数单位。□

对于虚部 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_bc8503a4091dd4.png)，可以使用所有通常的向量运算，例如加法、缩放、点积、叉积等。利用四元数的定义，可以如下推导出两个四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png) 之间的乘法运算。注意，虚数单位的乘法不满足交换律。

**乘法：**


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b2378f93bd98f0.png)


从这个等式可以看出，计算两个四元数的乘积时，我们同时使用叉积和点积。

除了四元数的定义，还需要加法、共轭、范数和单位元的定义：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_24d69da69fb39d.png)


化简 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_890086b8b9f37a.png) 时（结果如上所示），虚部相互抵消，只剩下实部。范数有时记作 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_dd97294837c1af.png) [1105]。由上述结果可以推导出乘法逆元，记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a68bd8c10a5dc0.png)。逆元必须满足等式 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_046e7e59fda774.png)（这是乘法逆元通常应满足的条件）。从范数的定义可推导出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_beaf2fdcb58ec4.png)


由此得到如下乘法逆元：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_e5841d566cfe9b.png)


逆元公式使用了标量乘法。这种运算可以从式 4.3.1 中的乘法推导出来：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b72464fa9e64af.png)，且 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b8af871968617b.png)，这意味着标量乘法满足交换律：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_02d8cac2d8192e.png)。

由定义很容易推导出下面这些规则：

**共轭规则：**


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_05db634b7c949a.png)


**范数规则：**


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a6d311441ded94.png)


**乘法定律：**


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_8cf9af687d0c96.png)


单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a1a9b061b3199e.png) 满足 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ecb311ab52f1a0.png)。由此可知，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 可以写成


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_5e9d3f498f42c4.png)


其中某个三维向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_04a4a051b9fb04.png) 满足 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_00e1c4ebc08fac.png)，因为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_bb283116a0e024.png)


当且仅当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_fd3ef15ad5c94c.png)。下一节将看到，单位四元数非常适合以极高的效率构造旋转和朝向。但在此之前，我们还要介绍一些针对单位四元数的额外运算。

对于复数，二维单位向量可以写成 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_33d3003d87a94f.png)。四元数中对应的形式为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_29f0a2de89cf5f.png)


由式（4.41）可得到单位四元数的对数函数和幂函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a65f9db8650fa9.png)


### 4.3.2 四元数变换

现在，我们研究四元数集合中的一个子类，即长度为单位长度的四元数，称为*单位四元数*。关于单位四元数，最重要的一点是：它们可以表示任意三维旋转，而且这种表示极为紧凑、简单。

下面说明单位四元数为何对旋转和朝向如此有用。首先，将点或向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_de604c916be7ac.png) 的四个坐标放入四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_cdddfa52c07e87.png) 的各个分量，并假设有一个单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a89cf49a6943c8.png)。可以证明，


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a097db0d0373c3.png)


会将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_cdddfa52c07e87.png)（因而也将点 **p**）绕轴 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_04a4a051b9fb04.png) 旋转 2φ。注意，由于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 是单位四元数，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_59b3c3b2ec9bab.png)。参见图 4.9。


![图4.9 单位四元数所表示的旋转变换](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_3_4.9.png)

**图 4.9。** 单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a89cf49a6943c8.png) 所表示的旋转变换示意图。该变换绕轴 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_04a4a051b9fb04.png) 旋转 2φ 弧度。

![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 的任意非零实数倍也表示相同的变换，这意味着 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_d808ff97a8ada6.png) 表示相同的旋转。也就是说，将轴 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_04a4a051b9fb04.png) 和实部 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_57c7a152b5541d.png) 都取反，所构造的四元数产生的旋转与原四元数完全相同。这也意味着，从矩阵中提取四元数时，返回的可能是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)，也可能是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_d808ff97a8ada6.png)。

给定两个单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png)，对四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_cdddfa52c07e87.png)（可以解释为点 **p**）先应用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)、再应用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png)，其串接由式（4.44）给出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_099bca380aa7da.png)


这里，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a53a61e0232f13.png) 是表示单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png) 串接的单位四元数。

#### 矩阵转换

由于经常需要组合若干不同的变换，而其中大多数采用矩阵形式，因此需要一种将式（4.43）转换成矩阵的方法。四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 可以转换为矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_7c83d725f3c23f.png)，如式（4.45）所示 [1633, 1634]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_a6042866efacf5.png)


这里，标量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_253ff22b5fc2b3.png)。对于单位四元数，可简化为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_25c3776d35438e.png)


一旦构造出四元数，就不必再计算任何三角函数，因此这种转换过程在实际应用中很高效。

反向转换，即将正交矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_7c83d725f3c23f.png) 转换成单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)，则稍微复杂一些。这个过程的关键是从式（4.46）的矩阵中得到以下差值：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_9daacd8351c5e5.png)


这些等式意味着，如果已知 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_57c7a152b5541d.png)，就可以计算向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_378baf49af7ec4.png) 的各个值，从而求出 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_7c83d725f3c23f.png) 的迹计算如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_05dd530c89b818.png)


由此可得到单位四元数的以下转换公式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_dca8d8b69be9cb.png)


为了使程序具有数值稳定性 [1634]，应避免除以很小的数。因此，首先令 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_8516c95ebb11a1.png)，由此得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac5d6d10ee1bee.png)


这进一步意味着，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_e2895bb807cfdb.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_625733c1130fde.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_bfcb904a8d7209.png) 和 u 中最大的一个决定了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_fbd6bc3baf2822.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_987be5b16638b9.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_535b7ec7c9114f.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_57c7a152b5541d.png) 中哪一个最大。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_57c7a152b5541d.png) 最大，就用式（4.49）求出四元数。否则，注意以下关系成立：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_e6474c172e762d.png)


然后，使用上述等式中适当的一个，计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_fbd6bc3baf2822.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_987be5b16638b9.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_535b7ec7c9114f.png) 中最大的分量，再用式（4.47）计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 的其余分量。Schüler [1588] 提出了一种无分支的变体，但它需要计算四次平方根。

#### 球面线性插值

球面线性插值是一种运算：给定两个单位四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png)，以及参数 t∈[0,1]，计算出插值四元数。例如，这对物体动画很有用。它对于相机朝向的插值则没有那么有用，因为相机的“向上”向量在插值过程中可能发生倾斜，这种效果通常令人不适。

这种运算的代数形式由下面的复合四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6e49f9db16c4ef.png) 表示：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_70ecc7492f59bb.png)


不过，对于软件实现，下面这种形式更加合适，其中 slerp 表示球面线性插值：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_9cd362bd1865e8.png)


为了计算这个等式所需的 φ，可以利用以下关系：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_90fa9bf7187b7d.png) [325]。对于 t∈[0,1]，slerp 函数计算出（唯一的注2）插值四元数，它们共同构成四维单位球面上从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)（t=0）到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png)（t=1）的最短弧。这条弧位于一个圆上，该圆是由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png) 和原点确定的平面与四维单位球面相交形成的。图 4.10 对此作了说明。计算出的旋转四元数绕固定轴以恒定速度旋转。像这样速度恒定、因而加速度为零的曲线称为*测地曲线* [229]。球面上的*大圆*由经过原点的平面与球面相交生成，这种圆的一部分称为*大圆弧*。


![图4.10 单位四元数之间的球面线性插值](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_3_4.10.png)

**图 4.10。** 单位四元数表示为单位球面上的点。slerp 函数用于在四元数之间进行插值，插值路径是球面上的一条大圆弧。注意，从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_3039c39a465895.png) 插值到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b7f78f981659aa.png)，与从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_3039c39a465895.png) 经 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_3dd95f202e35e6.png) 再插值到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b7f78f981659aa.png) 并不相同，尽管它们最终到达相同的朝向。

slerp 函数非常适合在两个朝向之间插值，而且表现良好（固定轴、恒定速度）。使用多个欧拉角进行插值时则并非如此。实际应用中，直接计算 slerp 的开销很大，因为需要调用三角函数。Malyshau [1114] 讨论了如何将四元数整合进渲染流水线。他指出，对于 90 度的夹角，如果不用 slerp，而只在像素着色器中对四元数进行归一化，三角形朝向的最大误差为 4 度。在光栅化三角形时，这样的误差水平可以接受。Li [1039, 1040] 提供了计算 slerp 的增量方法，速度快得多，而且不牺牲任何精度。Eberly [406] 提出了一种只用加法和乘法计算 slerp 的快速技术。

当存在两个以上的朝向，例如 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_efc95c5339b066.png)，并且希望从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_aad63ae4d40811.png) 插值到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_3039c39a465895.png)，再到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b7f78f981659aa.png)，依此类推，直到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_e4d7f14dbf3237.png) 时，可以直接使用 slerp。此时，当接近例如 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png) 时，会将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_5b3b9c7181ca6f.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png) 作为 slerp 的参数。经过 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png) 后，再将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_d7d58ac6ef48c5.png) 作为 slerp 的参数。这会在朝向插值中造成突然的顿挫，图 4.10 中可以看到这一点。这与对点进行线性插值时的情形类似；参见第 720 页图 17.3 的右上部分。有些读者可能希望在读过第 17 章有关样条的内容之后，再回过头来阅读下面这一段。

更好的插值方式是使用某种样条。我们在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_d7d58ac6ef48c5.png) 之间引入四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_c66b024efdd195.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_575c319304adab.png)。可以在由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_c66b024efdd195.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_575c319304adab.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_d7d58ac6ef48c5.png) 构成的四元数集合中定义球面三次插值。令人惊讶的是，这些额外的四元数按如下方式计算 [404]注3：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_df4e9ac063f33e.png)


![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_c66b024efdd195.png) 将用于借助平滑的三次样条对四元数进行球面插值，如式（4.55）所示：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_b6ad9a504b35f5.png)


如上所示，squad 函数由使用 slerp 的重复球面插值构成（关于对点进行重复线性插值的信息，参见第 17.1.1 节）。插值将经过初始朝向 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6bb66555d7e552.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_8c02f4f48a434d.png)，但不经过 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_c66b024efdd195.png)；后者用于指示各个初始朝向处的切线朝向。

#### 从一个向量旋转到另一个向量

一种常见的运算是沿尽可能短的路径，从一个方向 **s** 变换到另一个方向 **t**。四元数的数学性质大大简化了这一过程，并显示出四元数与这种表示形式之间的密切关系。首先，将 **s** 和 **t** 归一化。然后计算单位旋转轴，记为 **u**，其计算方式为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_7ffdaa17c10b8e.png)。接着，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_c315a48dea970c.png)，且 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_793ba184a5e2cb.png)，其中 2φ 为 **s** 与 **t** 之间的夹角。于是，表示从 **s** 旋转到 **t** 的四元数为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_6e314917126d1e.png)。实际上，利用半角关系和三角恒等式化简 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ab1fd4b1e66bd3.png)，可得到 [1197]


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_92995cefc9ec87.png)


与将叉积 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ca97cfb5baa554.png) 归一化相比，直接采用这种方式生成四元数，可以避免 **s** 和 **t** 指向几乎相同方向时的数值不稳定问题 [1197]。当 **s** 和 **t** 指向相反方向时，两种方法都会出现稳定性问题，因为此时会发生除以零。一旦检测到这种特殊情况，就可以使用任意垂直于 **s** 的旋转轴，将其旋转到 **t**。

有时，我们需要从 **s** 旋转到 **t** 的矩阵表示。对式（4.46）进行一些代数和三角化简后，旋转矩阵变为 [1233]


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_3c564e2cb32858.png)


在这个等式中，使用了以下中间计算结果：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_9b4b6fa04a6e55.png)


可以看到，经过这些化简，所有平方根和三角函数都消失了，因此这是一种高效的矩阵构造方法。注意，式（4.57）的结构与式（4.30）相似，而这里的这种形式不需要三角函数。

注意，当 **s** 和 **t** 平行或接近平行时，必须谨慎处理，因为此时 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_fce0ffe2dcdd5a.png)。如果 φ≈0，可以返回单位矩阵。不过，如果 2φ≈π，则可以绕任意一个轴旋转 π 弧度。这个轴可以通过计算 **s** 与任意另一个不平行于 **s** 的向量的叉积来找到（第 4.2.4 节）。Möller 和 Hughes 使用豪斯霍尔德矩阵，以另一种方式处理这种特殊情况 [1233]。

**注1：** 公平地说，Robinson [1502] 早在 1958 年就已将四元数用于刚体模拟。
**注2：** 当且仅当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_643d5e65b15c12.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_03_ac3952f889439a.png) 不互为相反数。
**注3：** Shoemake [1633] 给出了另一种推导。


## 4.4 顶点混合

来源：原书第 84—87 页（PDF 物理页 105—108）；图 4.13 及其图注续排于原书第 88 页（PDF 物理页 109）。

设想一个数字角色的手臂由前臂和上臂两部分构成，并对它制作动画，如图 4.11 左侧所示。可以使用刚体变换（第 4.1.6 节）为这个模型制作动画。然而，这样一来，两部分之间的关节就不像真正的肘关节。这是因为使用了两个独立的物体，因此关节由这两个独立物体的重叠部分组成。显然，只使用一个物体会更好。但是，静态的模型部件并不能解决让关节具有柔性的问题。

**顶点混合**（vertex blending）是解决这一问题的一种常用方法 [1037, 1903]。这项技术还有其他几个名称，例如**线性混合蒙皮**（linear-blend skinning）、**包络**（enveloping）或**骨架子空间变形**（skeleton-subspace deformation）。虽然这里介绍的算法的确切起源并不清楚，但定义骨骼并让皮肤对骨骼的变化作出响应，是计算机动画中一个由来已久的概念 [1100]。在最简单的形式中，仍像之前一样分别为前臂和上臂制作动画，但在关节处，两部分通过具有弹性的“皮肤”连接起来。因此，这个弹性部分中的一组顶点由前臂矩阵进行变换，另一组顶点则由上臂矩阵进行变换。这样一来，同一个三角形的各个顶点可能由不同的矩阵变换，而不是每个三角形只使用一个矩阵。见图 4.11。


![图 4.11：刚体变换与顶点混合的手臂关节对比](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4.4_4.11.png)

**图 4.11。** 左侧，一条由前臂和上臂组成的手臂通过对两个独立物体施加刚体变换来制作动画。肘部的外观并不真实。右侧，在单一物体上使用顶点混合。右起第二条手臂展示了用一层简单的皮肤直接连接两部分、覆盖肘部时的情况。最右侧的手臂展示了使用顶点混合，并对一些顶点采用不同权重进行混合时的情况：(2/3, 1/3) 表示该顶点对来自上臂的变换赋予 2/3 的权重，对来自前臂的变换赋予 1/3 的权重。最右侧的示意图也展示了顶点混合的一个缺点：可以看到肘部内侧出现了折叠。使用更多骨骼，并更仔细地选择权重，可以获得更好的结果。

再进一步，可以允许同一个顶点由多个不同的矩阵变换，然后将得到的位置加权混合在一起。具体做法是为动画物体建立一个由骨骼组成的骨架，其中每根骨骼的变换都可以通过用户定义的权重来影响每个顶点。由于整条手臂都可能具有“弹性”，也就是说，所有顶点都可能受到不止一个矩阵的影响，因此整个网格通常称为覆盖在骨骼上的**皮肤**（skin）。见图 4.12。许多商业建模系统都具有这种骨架与骨骼建模功能。尽管名为骨骼，它们并不一定必须是刚性的。例如，Mohr 和 Gleicher [1230] 提出了添加额外关节以实现肌肉隆起等效果的想法。James 和 Twigg [813] 讨论了使用可以压缩和拉伸的骨骼进行动画蒙皮。


![图 4.12：顶点混合的实际手臂示例](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4.4_4.12.png)

**图 4.12。** 顶点混合的一个实际示例。左上图展示了一条手臂处于伸展姿势时的两根骨骼。右上图展示了网格，其中用颜色表示每个顶点属于哪根骨骼。下图：姿势略有不同的手臂网格经过着色后的效果。（图片由 Jeff Lander [968] 提供。）

在数学上，这可以用式（4.59）表示，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_789e716ab03779.png) 是原始顶点，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_5efaf10b49d3a3.png) 是变换后的顶点，其位置取决于时间 t：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_702d0bca6acbc4.png)


共有 n 根骨骼影响 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_789e716ab03779.png) 的位置，而 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_789e716ab03779.png) 是以世界坐标表示的。数值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_3e929edc96716e.png) 是骨骼 i 对顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_789e716ab03779.png) 的权重。矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_e77bf4c2b91cbe.png) 将初始骨骼坐标系中的坐标变换到世界坐标。通常，骨骼的控制关节位于其坐标系的原点。例如，对于前臂骨骼，会把肘关节移到原点，再用一个随动画变化的旋转矩阵，使手臂的这一部分绕该关节运动。矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_c4b9f556bd3c1c.png) 是第 i 根骨骼的世界变换，它随时间变化，从而使物体产生动画；它通常由多个矩阵串接而成，例如层级结构中前序骨骼的变换以及局部动画矩阵。

Woodland [1903] 深入讨论了维护和更新 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_c4b9f556bd3c1c.png) 矩阵动画函数的一种方法。每根骨骼都相对于自身的参考坐标系，将顶点变换到一个位置，最后再从计算得到的这一组点插值得出最终位置。有些关于蒙皮的讨论并未显式列出矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_e77bf4c2b91cbe.png)，而是把它视为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_c4b9f556bd3c1c.png) 的一部分。我们在这里将它列出，是因为这是一个有用的矩阵，而且几乎总是矩阵串接过程的一部分。

在实践中，对每一帧动画中的每根骨骼，都会将矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_c4b9f556bd3c1c.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_572b2d1ca90cb6.png) 串接起来，并使用各个得到的矩阵来变换顶点。顶点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_789e716ab03779.png) 由不同骨骼的串接矩阵进行变换，然后使用权重 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_3e929edc96716e.png) 加以混合，“顶点混合”这一名称由此而来。各权重非负且总和为一，因此实际发生的过程是：先将顶点变换到几个位置，再在这些位置之间进行插值。所以，对于固定的 t，变换后的点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_a15c7692963708.png) 将位于所有 i = 0, …, n − 1 对应的点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_e16227d031752e.png) 所构成的点集的凸包内。通常，也可以用式（4.59）变换法线。根据所使用的变换，有时可能需要改用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_1efecdf329436a.png) 的逆矩阵的转置；例如，当某根骨骼受到相当大程度的拉伸或压缩时，就是如此，第 4.1.7 节对此作了讨论。

顶点混合非常适合在 GPU 上使用。网格的顶点集可以放入静态缓冲区，只需发送到 GPU 一次，之后便可重复使用。在每一帧中，只有骨骼矩阵发生变化，由顶点着色器计算这些矩阵对已存储网格的影响。这样便可尽量减少 CPU 处理的数据量以及从 CPU 传输的数据量，让 GPU 高效地渲染网格。如果模型的整套骨骼矩阵能够一起使用，实现起来最为简单；否则就必须拆分模型，并复制部分骨骼。另一种方式是把骨骼变换存入顶点可以访问的纹理中，从而避免触及寄存器存储容量的限制。通过用四元数表示旋转，每个变换只需两张纹理即可存储 [1639]。如果支持无序访问视图存储，则可以复用蒙皮结果 [146]。

也可以指定超出 [0, 1] 范围、或总和不为一的权重集合。不过，只有在使用其他混合算法时，这样做才有意义，例如变形目标（morph targets，第 4.5 节）。

基本顶点混合的一个缺点是可能出现不希望有的折叠、扭曲和自相交 [1037]。见图 4.13。一个更好的解决办法是使用**对偶四元数**（dual quaternions）[872, 873]。这种蒙皮技术有助于保持原始变换的刚性，从而避免肢体出现“糖纸式”扭曲。其计算开销不到线性混合蒙皮的 1.5 倍，效果又很好，因此这项技术迅速得到了采用。不过，对偶四元数蒙皮可能产生鼓胀效果，Le 和 Hodgins [1001] 提出了**旋转中心蒙皮**（center-of-rotation skinning）作为更好的替代方案。他们基于以下假设：局部变换应当是刚体变换，而具有相似权重 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_04_3e929edc96716e.png) 的顶点应当具有相似的变换。算法预先计算每个顶点的旋转中心，同时施加正交（刚体）约束，以防止肘部塌陷和糖纸式扭曲伪影。在运行时，该算法与线性混合蒙皮相似：GPU 实现先对旋转中心进行线性混合蒙皮，然后执行四元数混合步骤。


![图 4.13：线性混合蒙皮与对偶四元数蒙皮对比](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4.4_4.13.png)

**图 4.13。** 左侧展示了使用线性混合蒙皮时关节处出现的问题。右侧，使用对偶四元数进行混合改善了外观。（图片由 Ladislav Kavan 等人提供，模型由 Paul Steed 制作 [1693]。）


## 4.5 变形

来源：原书第 87—91 页（PDF 物理页 108—112）；本节结束于第 92 页的 4.6 节标题之前。

在制作动画时，将一个三维模型变形为另一个三维模型会很有用 [28, 883, 1000, 1005]。设想在时刻 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_9d5212cdbc9a74.png) 显示一个模型，而我们希望到时刻 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_52e22163357e7d.png) 时，它已经变成另一个模型。对于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_9d5212cdbc9a74.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_52e22163357e7d.png) 之间的所有时刻，可以利用某种插值方法，得到一个连续变化的“混合”模型。图 4.14 展示了一个变形的例子。


![图 4.14 顶点变形](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4.5_4.14.png)

**图 4.14** 顶点变形。为每个顶点定义两个位置和两个法线。在每一帧中，顶点着色器通过线性插值得到中间的位置与法线。（图片由 NVIDIA Corporation 提供。）

变形需要解决两个主要问题，即*顶点对应问题*和*插值问题*。给定两个任意模型，它们可能具有不同的拓扑结构、不同的顶点数量以及不同的网格连接关系，因此通常必须先建立这些顶点之间的对应关系。这是一个困难的问题，该领域已经有大量研究。感兴趣的读者可以参阅 Alexa 的综述 [28]。

不过，如果两个模型之间已经存在一一对应的顶点关系，就可以逐顶点进行插值。也就是说，第一个模型中的每个顶点，在第二个模型中必须有且仅有一个对应顶点，反之亦然。这使得插值成为一项容易完成的任务。例如，可以直接对顶点进行线性插值（其他插值方法见第 17.1 节）。为了计算时刻 t∈[![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_9d5212cdbc9a74.png),![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_52e22163357e7d.png)] 的变形后顶点，我们首先计算 s=(t−![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_9d5212cdbc9a74.png))/(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_52e22163357e7d.png)−![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_9d5212cdbc9a74.png))，然后进行线性顶点混合：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_88e1053e4b0104.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_c2f9d040fb9beb.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_121bbb5295ca5a.png) 对应于同一个顶点，只是分别处于不同的时刻 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_9d5212cdbc9a74.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_52e22163357e7d.png)。

一种能让用户进行更直观控制的变形方法称为*变形目标*（morph targets），或*混合形状*（blend shapes）[907]。可以借助图 4.15 说明其基本思想。我们从一个中性模型开始，这里是一张人脸。将这个模型记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_bae0f8bacb0571.png)。此外，我们还有一组不同的面部姿态。示意图中只有一种姿态，即微笑的面孔。一般而言，我们可以允许 k≥1 种不同的姿态，将它们记为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_d5825c1f8994e5.png)，i∈[1,…,k]。作为预处理，计算“差分面孔”：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_0d837aa681e024.png)，也就是从每种姿态中减去中性模型。


![图 4.15 中性面孔、微笑面孔和差分向量](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4.5_4.15.png)

**图 4.15** 给定两种嘴部姿态，可以计算出一组差分向量，以控制插值，甚至外推。在变形目标方法中，利用差分向量将动作“添加”到中性面孔上。为差分向量赋予正权重，会得到微笑的嘴形；负权重则可以产生相反的效果。

此时，我们有一个中性模型 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_bae0f8bacb0571.png)，以及一组差分姿态 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_c673336d617c1d.png)。于是，可以用下式得到变形后的模型 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_ea2ce10af50487.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_a822da73df4108.png)


这以中性模型为基础，再通过权重 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_3e929edc96716e.png)，按需要将不同姿态的特征添加到它上面。对于图 4.15，设定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_50251dbe8a1da7.png)=1，得到的恰好就是示意图中间的微笑面孔。使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_05_50251dbe8a1da7.png)=0.5，会得到一张半微笑的面孔，依此类推。也可以使用负权重以及大于一的权重。

对于这个简单的人脸模型，我们还可以添加一张眉毛呈现“悲伤”表情的面孔。随后，为眉毛使用负权重，就可以产生“快乐”的眉毛。由于位移是相加的，因此可以将这一眉毛姿态与微笑嘴部的姿态结合使用。

变形目标是一项强大的技术，能为动画师提供很大的控制空间，因为模型的不同特征可以彼此独立地操纵。Lewis 等人 [1037] 提出了*姿态空间变形*（pose-space deformation），将顶点混合与变形目标结合起来。Senior [1608] 使用预先计算的顶点纹理，存储并读取目标姿态之间的位移。支持流输出（stream-out）和逐顶点 ID 的硬件，允许在单个模型中使用多得多的目标，并且完全在 GPU 上计算这些效果 [841, 1074]。先使用低分辨率网格，再通过曲面细分阶段和位移映射生成高分辨率网格，可以避免对高精细度模型的每一个顶点都执行蒙皮所产生的开销 [1971]。


![图 4.16 使用混合形状制作的 Delsin 面部动画](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4.5_4.16.png)

**图 4.16** 《inFAMOUS Second Son》中角色 Delsin 的面部使用混合形状制作动画。所有这些画面都使用同一张处于静止姿态的面孔，然后通过修改不同的权重，使面孔呈现不同的样貌。（图片由 Naughty Dog LLC 提供。《inFAMOUS Second Son》© 2014 Sony Interactive Entertainment LLC。inFAMOUS Second Son 是 Sony Interactive Entertainment LLC 的商标。由 Sucker Punch Productions LLC 开发。）

图 4.16 展示了一个同时使用蒙皮与变形的实际例子。Weronko 和 Andreason [1872] 在《The Order: 1886》中使用了蒙皮与变形。


## 4.6 几何缓存回放

来源：原书第 92 页（PDF 第 113 页）。

在过场动画中，可能希望使用质量极高的动画，例如，某些运动无法用前面介绍的任何方法来表示。一种朴素的做法是存储所有帧的全部顶点，从磁盘读取这些顶点并更新网格。然而，即使只是在一段短动画中使用一个包含 30,000 个顶点的简单模型，这种做法的数据量也可能达到 50 MB/s。Gneiting [545] 提出了几种方法，可以将内存开销降低到原来的约 10%。

首先，使用量化。例如，对于位置和纹理坐标，每个坐标分量都用 16 位整数存储。这一步是有损的，也就是说，压缩之后无法恢复原始数据。为了进一步减少数据量，可以进行空间预测和时间预测，并对差值进行编码。对于空间压缩，可以使用平行四边形预测 [800]。对于一个三角形带，只需在当前三角形所在的平面内，围绕当前三角形的一条边将该三角形反射过去，使之形成一个平行四边形，就能得到下一个顶点的预测位置。随后，对实际位置与这个新位置之间的差值进行编码。如果预测准确，大多数数值都会接近零，这对于许多常用的压缩方案来说非常理想。与 MPEG 压缩类似，也可以在时间维度上进行预测。也就是说，每隔 n 帧执行一次空间压缩；在这些帧之间，则在时间维度上进行预测。例如，如果某个顶点从第 n−1 帧到第 n 帧移动了一个位移向量，那么它在移动到第 n+1 帧时，很可能也会有相近的位移。这些技术充分降低了存储需求，使得该系统能够用于实时数据流传输。


## 4.7 投影

来源：《Real-Time Rendering, Fourth Edition》，书页 92—102（PDF 物理页 113—123）。本节包含 4.7.1 和 4.7.2；不含章末“延伸阅读与资源”。

在真正渲染场景之前，必须将场景中所有相关对象投影到某种平面上，或变换到某种简单的体积中。此后再进行裁剪和渲染（第 2.3 节）。

本章到目前为止介绍的变换都不影响第四个坐标，即 w 分量。也就是说，点和向量在变换之后仍保持各自的类型。此外，4×4 矩阵的最下面一行始终为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_af96c6b36ca855.png)。透视投影矩阵在这两点上都是例外：最下面一行包含会改变向量和点的数值，而且往往需要进行齐次归一化。也就是说，w 往往不等于 1，因此需要除以 w 才能得到非齐次点。本节首先讨论的正交投影是一种较简单、同样常用的投影。它不影响 w 分量。

本节假定观察者沿着相机的 z 轴负方向观察，y 轴向上，x 轴向右。这是一个右手坐标系。有些文献和软件（例如 DirectX）使用左手坐标系，观察者沿相机的 z 轴正方向观察。两种坐标系都有效，最终能够获得相同的效果。

### 4.7.1 正交投影

正交投影的一个特征是，平行线在投影后仍然平行。使用正交投影观察场景时，无论对象距相机多远，其大小都保持不变。下面给出的矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_0fdada31469c13.png) 是一个简单的正交投影矩阵：它保持点的 x 和 y 分量不变，同时将 z 分量置为零，也就是正交投影到平面 z=0 上：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_66e0eb2f72f82b.png)


这种投影的效果如图 4.17 所示。显然，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_0fdada31469c13.png) 不可逆，因为其行列式 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_1e02be59c5aae4.png)。换句话说，这一变换将三维降为二维，而被丢弃的维度无法恢复。使用这种正交投影进行观察时存在一个问题：它会将 z 值为正和为负的点都投影到投影平面上。通常，将 z 值（以及 x 和 y 值）限制在某个区间内会很有用，例如从 n（近平面）到 f（远平面）。注4 这就是接下来这个变换的目的。


![图 4.17 正交投影的三个视图](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_7_4.17.png)

**图 4.17**　式（4.62）生成的简单正交投影的三个不同视图。可以将此投影理解为观察者沿 z 轴负方向观察，这意味着投影只是忽略 z 坐标（或将其置为零），同时保留 x 和 y 坐标。注意，位于 z=0 两侧的对象都会被投影到投影平面上。

一种更常用的正交投影矩阵由六元组 (l,r,b,t,n,f) 表示，这六个值分别表示左、右、下、上、近、远六个平面。该矩阵对这些平面构成的轴对齐包围盒（AABB；定义见第 22.2 节）进行缩放和平移，将其变为以原点为中心、与坐标轴对齐的立方体。AABB 的最小角点为 (l,b,n)，最大角点为 (r,t,f)。必须认识到，这里 n>f，因为我们沿 z 轴负方向看向这块空间体积。直觉会告诉我们，近处的数值应小于远处，因此可以允许用户按这种方式提供数值，然后在内部将它们取负。

在 OpenGL 中，这个轴对齐立方体的最小角点为 (−1,−1,−1)，最大角点为 (1,1,1)；在 DirectX 中，其边界从 (−1,−1,0) 到 (1,1,1)。这个立方体称为**规范视体**，其中的坐标称为**归一化设备坐标**。变换过程如图 4.18 所示。之所以要变换到规范视体，是因为在其中进行裁剪更有效率。


![图 4.18 将轴对齐包围盒变换为规范视体](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_7_4.18.png)

**图 4.18**　将轴对齐包围盒变换到规范视体。首先平移左侧的盒子，使其中心与原点重合。然后对其缩放，使其具有右侧所示规范视体的大小。

变换到规范视体之后，待渲染几何体的顶点相对于该立方体进行裁剪。最终，将剩余的单位正方形映射到屏幕上，渲染不在立方体外部的几何体。这个正交变换如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_bb1c4cf816ef49.png)


如该式所示，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_0fdada31469c13.png) 可以写成平移 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_b80e54b5038b3a.png) 与随后执行的缩放矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_3b4c7a0cd2e2e5.png) 的级联，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_d7790c56b8bd3e.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_db53d1311cdf87.png)。该矩阵可逆，注5 即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ca0fdb8a53a08e.png)。

在计算机图形学中，投影之后通常使用左手坐标系，也就是说，对于视口，x 轴向右，y 轴向上，z 轴指向视口内部。按照我们定义 AABB 的方式，远平面的值小于近平面的值，因此正交变换总会包含一个镜像变换。为了看清这一点，假定原始 AABB 的大小与目标（规范视体）相同。那么，AABB 的 (l,b,n) 坐标为 (−1,−1,1)，(r,t,f) 坐标为 (1,1,−1)。将它们代入式（4.63），得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_d510d1f118f532.png)


这是一个镜像矩阵。正是这一镜像操作，将右手观察坐标系（沿 z 轴负方向观察）转换为左手的归一化设备坐标。

DirectX 将 z 深度映射到 [0,1] 范围，而 OpenGL 使用 [−1,1]。只需在正交矩阵之后应用一个简单的缩放和平移矩阵，即可实现这一点，也就是


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_3cd27d49143656.png)


因此，DirectX 使用的正交矩阵为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_9d5dac3495f6a5.png)


该矩阵通常以转置形式给出，因为 DirectX 采用行主序形式书写矩阵。

**注4：** 近平面也称为前平面（front plane）或 hither；远平面也称为后平面（back plane）或 yon。

**注5：** 当且仅当 n≠ f、l≠ r 且 t≠ b 时可逆；否则不存在逆矩阵。

### 4.7.2 透视投影

透视投影是一种比正交投影更复杂的变换，在大多数计算机图形学应用中广泛使用。在这里，平行线在投影之后通常不再平行，而是可能在无限延伸处汇聚于一点。透视更接近我们感知世界的方式，即越远的对象看起来越小。

首先，我们给出一个有助于理解的透视投影矩阵推导，它将点投影到平面 z=−d（d>0）上。我们从世界空间开始推导，以便更容易理解从世界空间到观察空间的转换过程。随后再给出更常规的矩阵，例如 OpenGL 中使用的矩阵 [885]。

假定相机（视点）位于原点，我们希望将点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_702399136f4ebc.png) 投影到平面 z=−d（d>0）上，得到新点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_62f5d59fb85994.png)。这一情形如图 4.19 所示。根据图中的相似三角形，可以得到针对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_e3d1d3f5c5ad00.png) 的 x 分量的下述推导：


![图 4.19 透视投影矩阵推导](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_7_4.19.png)

**图 4.19**　推导透视投影矩阵时使用的记号。点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_702399136f4ebc.png) 被投影到平面 z=−d（d>0）上，得到投影点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_e3d1d3f5c5ad00.png)。投影以相机位置为视点进行，本例中的相机位于原点。右侧给出了推导 x 分量时使用的相似三角形。


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_17eb0e20f43175.png)


![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_e3d1d3f5c5ad00.png) 的其他分量为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_18e71d9b8fd6d4.png)（以与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_f31b04fcdf2226.png) 类似的方式得到）以及 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_1e9eb891ab1be2.png)。将它们与上式结合，可得如下透视投影矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_749df2677ec6a1.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_fa2c3b6e662b47.png)


通过下式可以验证，这个矩阵确实产生了正确的透视投影：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_86676c19967c6c.png)


最后一步将整个向量除以 w 分量（本例中为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_f8ab481619c20e.png)），从而使最后一个位置的值为 1。所得的 z 值始终为 −d，因为我们正是投影到这个平面上。

从直观上不难理解为什么齐次坐标能够实现投影。对齐次归一化过程的一种几何解释是：它将点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_0eb27b3a3253de.png) 投影到平面 w=1 上。

与正交变换类似，也存在一种透视变换，它并不真正投影到一个平面上（那样的变换不可逆），而是将视锥体变换到前面所述的规范视体中。这里假定视锥体从 z=n 开始，到 z=f 结束，且 0>n>f。位于 z=n 处的矩形，其最小角点为 (l,b,n)，最大角点为 (r,t,n)。如图 4.20 所示。


![图 4.20 视锥体变换为规范视体](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_7_4.20.png)

**图 4.20**　矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_749df2677ec6a1.png) 将视锥体变换为称作规范视体的单位立方体。

参数 (l,r,b,t,n,f) 决定了相机的视锥体。水平视场角由视锥体左、右平面之间的夹角决定（这两个平面由 l 和 r 决定）。同样，垂直视场角由上、下平面之间的夹角决定（这两个平面由 t 和 b 决定）。视场角越大，相机“看到”的范围就越大。令 r≠−l 或 t≠−b，便可创建非对称视锥体。例如，非对称视锥体可用于立体观察和虚拟现实（第 21.2.3 节）。

视场角是营造场景感受的重要因素。相对于计算机屏幕，人眼本身有一个实际的视场角。其关系为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_0d2f3dcfec28cf.png)


其中 φ 为视场角，w 为物体在垂直于视线方向上的宽度，d 为到物体的距离。例如，25 英寸显示器的宽度约为 22 英寸。在距离 12 英寸处，水平视场角为 85 度；在 20 英寸处为 58 度；在 30 英寸处为 40 度。同一个公式也可以将相机镜头焦距换算为视场角，例如，35 毫米相机（画幅宽度为 36 毫米）配备标准 50 毫米镜头时，有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_627322045904e6.png) 度。

如果采用的视场角比实际观察设置的视场角更窄，透视效果就会减弱，因为观察者看到的是放大后的场景。设置更宽的视场角会使物体显得失真（如同使用广角镜头），屏幕边缘附近尤其明显，而且会夸大近处对象的尺度。不过，更宽的视场角会让观察者觉得对象更大、更具震撼力，同时还能向用户提供更多周围环境的信息。

将视锥体变换为单位立方体的透视变换矩阵由式（4.71）给出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ea405e8436e99a.png)


对一个点应用这个变换后，会得到另一个点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_5f7649c588b6ec.png)。这个点的 w 分量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_57c7a152b5541d.png) 通常既不为零，也不等于 1。要得到投影后的点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_702399136f4ebc.png)，需要除以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_57c7a152b5541d.png)，即


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_54ab3dcf413b5e.png)


矩阵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_749df2677ec6a1.png) 始终保证将 z=f 映射为 +1，将 z=n 映射为 −1。

远平面之外的对象会被裁剪，因此不会出现在场景中。透视投影可以处理位于无穷远处的远平面，此时式（4.71）变为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_dab6a44b7e7e20.png)


总的来说，先应用透视变换 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_749df2677ec6a1.png)（无论采用哪种形式），然后进行裁剪和齐次归一化（除以 w），最终得到归一化设备坐标。

为了得到 OpenGL 使用的透视变换，首先乘以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_c167a5642338b6.png)，原因与正交变换相同。这只是将式（4.71）第三列中的数值取负。应用这一镜像变换之后，近、远平面的值以正数输入，满足 0<n′<f′，与通常向用户提供的方式一致。不过，它们仍然表示沿世界坐标系 z 轴负方向的距离，而该方向正是观察方向。作为参考，下面给出 OpenGL 的公式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_4306ff2c9c7119.png)


一种更简单的设置方式是只提供垂直视场角 φ、宽高比 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_1ce0c1a50ace66.png)（其中 w× h 为屏幕分辨率）、n′ 和 f′。这样得到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_dec92fd8e8c5eb.png)


其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_5aa9bd37dedb84.png)。该矩阵的作用与旧的 `gluPerspective()` 完全相同，后者属于 OpenGL 实用库（GLU）。

有些 API（例如 DirectX）将近平面映射到 z=0（而不是 z=−1），将远平面映射到 z=1。此外，DirectX 使用左手坐标系定义其投影矩阵。这意味着 DirectX 沿 z 轴正方向观察，并将近、远平面的值表示为正数。下面是 DirectX 的公式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_81a131a487e7d7.png)


DirectX 的文档使用行主序形式，因此这个矩阵通常以转置形式给出。

使用透视变换的一个结果是，计算出的深度值不会随输入的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ca6c2954bc9df7.png) 值线性变化。将式（4.74）至（4.76）中的任一个矩阵与点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_702399136f4ebc.png) 相乘，可以看到


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_eb559ac0fccff0.png)


其中省略了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_95f3e0054b5dc9.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_d1e0a19427061b.png) 的具体表达式，常数 d 和 f 取决于所选矩阵。例如，若使用式（4.74），则 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_76138ba0e501a3.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_989798c161fb48.png)，且 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_c07e6b298f8b48.png)。译注1 为了得到归一化设备坐标（NDC）中的深度，需要除以 w 分量，结果为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_0efb257dcff1ed.png)


对于 OpenGL 投影，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_615be5c64f00b7.png)。可以看到，输出深度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_f82d41f781b0cf.png) 与输入深度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ca6c2954bc9df7.png) 成反比。译注1

例如，若 n′=10、f′=110（使用 OpenGL 的术语），当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ca6c2954bc9df7.png) 位于沿 z 轴负方向 60 个单位处（即中点）时，归一化设备坐标中的深度值为 0.833，而不是 0。图 4.21 展示了改变近平面到原点距离的效果。近、远平面的位置会影响 z 缓冲的精度。第 23.7 节将进一步讨论这一影响。


![图 4.21 近平面位置对深度分布的影响](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_7_4.21.png)

**图 4.21**　改变近平面到原点距离的效果。距离 f′−n′ 保持为 100。当近平面越来越接近原点时，靠近远平面的点在归一化设备坐标（NDC）深度空间中占用的范围越来越小。这会降低 z 缓冲在较远距离处的精度。

有几种方法可以提高深度精度。一种常用方法称为**反向 z**（reversed z），它使用浮点深度或整数存储 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_e5c8f9e2c23569.png) [978]。图 4.22 给出了比较。Reed [1472] 通过模拟表明，将浮点缓冲与反向 z 配合使用能够得到最佳精度；对于整数深度缓冲（通常每个深度值使用 24 位），这也是优先采用的方法。对于标准映射（即非反向 z），正如 Upchurch 和 Desbrun [1803] 所建议的那样，在变换中将投影矩阵单独应用，可以降低出错率。例如，使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_e8123010b73922.png) 可能比使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_87898f64b0113b.png) 更好，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_6d556060041d16.png)。此外，在 [0.5,1.0] 范围内，fp32 和 int24 的精度十分接近，因为 fp32 有 23 位尾数。让 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_f82d41f781b0cf.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ad4fee8392283b.png) 成正比的原因在于，这样可以简化硬件，并使深度压缩更有效；第 23.7 节将对此作进一步讨论。


![图 4.22 不同深度缓冲设置的精度比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_4_7_4.22.png)

**图 4.22**　使用 DirectX 变换（即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_26ee1cd7a82e8b.png)）设置深度缓冲的不同方式。左上：标准整数深度缓冲，这里展示的是 4 位精度（因此 y 轴上有 16 个标记）。右上：将远平面设置为 ∞，两个坐标轴上的微小偏移表明，这样做不会损失太多精度。左下：浮点深度使用 3 位指数和 3 位尾数。注意，y 轴上的分布是非线性的，这使 x 轴上的分布变得更加糟糕。右下：反向浮点深度，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_b526ad89a5f2a9.png)，所得分布好得多。（插图由 Nathan Reed 提供。）

Lloyd [1063] 提出对深度值取对数，以提高阴影贴图的精度。Lauritzen 等人 [991] 使用前一帧的 z 缓冲来确定尽可能大的近平面值和尽可能小的远平面值。对于屏幕空间深度，Kemen [881] 提议逐顶点使用以下重新映射：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_77129040dc3545.png)


其中，w 是顶点经投影矩阵变换之后的 w 值，z 是顶点着色器输出的 z 值。常数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_62b6ad131cdc88.png)，其中 f 是远平面。如果只在顶点着色器中应用这个变换，GPU 仍会在三角形上，对经过非线性变换的各顶点深度之间进行线性插值（式（4.79））。由于对数是单调函数，只要分段线性插值与准确的非线性变换深度值之间的差异足够小，遮挡剔除硬件和深度压缩技术仍然能够工作。对于经过充分曲面细分的几何体，大多数情况下都满足这一条件。不过，也可以逐片元应用该变换。做法是为每个顶点输出一个值 e=1+w，然后由 GPU 在三角形上对其插值。像素着色器随后将片元深度修改为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_02519f40db2e7e.png)，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_bd170c8f1da858.png) 是 e 的插值结果。当 GPU 不支持浮点深度，且渲染涉及很大的深度距离范围时，这种方法是一个很好的替代方案。

Cozzi [1605] 提出使用多个视锥体，这实际上可以将精度提高到任意所需程度。沿深度方向，将视锥体划分为若干互不重叠的较小子视锥体，它们的并集恰好等于原视锥体。按照从后向前的顺序，对这些子视锥体进行渲染。首先清空颜色缓冲和深度缓冲，并将所有待渲染对象归入与其重叠的各个子视锥体。对于每一个子视锥体，设置其投影矩阵，清空深度缓冲，然后渲染与该子视锥体重叠的对象。

**译注1：** 译注：此处按原书第 100 页保留。原文写作“常数 d 和 f”及“![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_c07e6b298f8b48.png)”，按式（4.77）应分别为“d 和 e”及“![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_a61777dc1a3bc2.png)”。原书式（4.78）右端印为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_e9b1a4f2b8f7eb.png)，但按其左端和前文对 d 的定义，代数化简应为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_aad3096ba9db17.png)。这里所谓“成反比”描述的是深度包含 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_07_ad4fee8392283b.png) 项的非线性关系，另有常数偏移。


## 延伸阅读与资源

Immersive Linear Algebra 网站 [1718] 提供了一本介绍线性代数基础的交互式书籍，通过鼓励读者操作图形来帮助建立直觉。其他交互式学习工具和变换代码库的链接，可在 [realtimerendering.com](https://realtimerendering.com/) 找到。

若想以轻松的方式培养对矩阵的直觉，Farin 和 Hansford 的《The Geometry Toolbox》[461] 是最好的书籍之一。另一部有用的著作是 Lengyel 的《Mathematics for 3D Game Programming and Computer Graphics》[1025]。想从不同角度了解矩阵基础，还可以参考许多计算机图形学教材，例如 Hearn 和 Baker [689]、Marschner 和 Shirley [1129]，以及 Hughes 等人 [785] 的著作。Ochiai 等人的课程 [1310] 介绍了矩阵基础以及矩阵的指数和对数，并讨论它们在计算机图形学中的用途。《Graphics Gems》系列 [72, 540, 695, 902, 1344] 介绍了各种与变换有关的算法，其中许多算法的代码可以在线获得。若想严肃、系统地研究一般矩阵技术，可以从 Golub 和 Van Loan 的《Matrix Computations》[556] 入手。关于骨架子空间变形／顶点混合及形状插值的更多内容，可阅读 Lewis 等人的 SIGGRAPH 论文 [1037]。

Hart 等人 [674] 和 Hanson [663] 提供了四元数的可视化。Pletinckx [1421] 和 Schlag [1566] 介绍了在一组四元数之间进行平滑插值的不同方法。Vlachos 和 Isidoro [1820] 推导了四元数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_04_99_666ecb60e6f4fb.png) 插值的公式。与四元数插值相关的一个问题，是沿曲线计算一致的坐标系，Dougan [374] 对此进行了讨论。

Alexa [28] 以及 Lazarus 和 Verroust [1000] 对许多不同的变形技术作了综述。Parent 的书 [1354] 是学习计算机动画技术的优秀资料。
