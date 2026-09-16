# Real-Time Rendering 第05章 着色基础 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [5.1 着色模型](<Real-Time_Rendering_4th_中文/第05章/05.01.md>)

- [5.2 光源](<Real-Time_Rendering_4th_中文/第05章/05.02.md>)

- [5.3 实现着色模型](<Real-Time_Rendering_4th_中文/第05章/05.03.md>)

- [5.4 走样与抗锯齿](<Real-Time_Rendering_4th_中文/第05章/05.04.md>)

- [5.5 透明度、Alpha 与合成](<Real-Time_Rendering_4th_中文/第05章/05.05.md>)

- [5.6 显示编码](<Real-Time_Rendering_4th_中文/第05章/05.06.md>)


## 章首导言

来源：原书第 103 页章首标题、题辞及导言（PDF 第 124 页）；导言所引用的图 5.1 及图注位于原书第 104 页（PDF 第 125 页）。

> “一幅好画等同于一桩善举。”
>
> ——文森特·梵高（Vincent Van Gogh）

渲染三维物体的图像时，模型不仅应具有正确的几何形状，还应具有所期望的视觉外观。视具体应用而定，这种外观可以是真实感的，也就是与真实物体的照片几乎相同；也可以是出于创作需要而选择的各种风格化外观。图 5.1 展示了这两类外观的示例。

本章将讨论对真实感渲染和风格化渲染同样适用的着色内容。第 15 章专门讨论风格化渲染；本书相当大的一部分，即第 9—14 章，则重点讨论真实感渲染中常用的基于物理的方法。


![图 5.1：真实感与风格化渲染示例](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_1_5.1.png)

**图 5.1** 上图来自使用虚幻引擎渲染的真实感景观场景。下图来自 Campo Santo 的游戏《看火人》（Firewatch），该游戏采用插画式美术风格进行设计。（上图由 Gökhan Karadayı 提供，下图由 Campo Santo 提供。）


## 5.1 着色模型

来源：原书第 103—106 页（PDF 第 124—127 页）。本节从“5.1 Shading Models”开始，到“5.2 Light Sources”之前结束；章首导言及其图 5.1 另存于本章导言文件。

确定渲染物体外观的第一步，是选择一个**着色模型**，用它描述物体的颜色应如何随表面朝向、观察方向和光照等因素而变化。

这里，我们以 Gooch 着色模型 [561] 的一种变体为例。这属于非真实感渲染，第 15 章将讨论这一主题。Gooch 着色模型旨在提高技术插图中细节的可辨识性。

Gooch 着色的基本思想，是比较表面法线与光源位置的关系。如果法线指向光源，就使用较暖的色调为表面着色；如果法线背向光源，则使用较冷的色调。在这两种朝向之间的角度处，对这两种色调进行插值；它们都以用户提供的表面颜色为基础。在这个示例中，我们还向模型加入一种风格化的“高光”效果，使表面呈现光亮的外观。图 5.2 展示了这一着色模型的实际效果。

着色模型通常具有一些用于控制外观变化的属性。设置这些属性的值，是确定物体外观的下一步。我们的示例模型只有一个属性，即表面颜色，如图 5.2 下图所示。


![图 5.2：结合 Gooch 着色与高光效果的模型](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_1_5.2.png)

**图 5.2** 一种将 Gooch 着色与高光效果相结合的风格化着色模型。上图展示了一个表面颜色为中性色的复杂物体。下图展示了具有各种不同表面颜色的球体。（中国龙网格来自 Computer Graphics Archive [1172]，原始模型来自 Stanford 3D Scanning Repository，即斯坦福三维扫描模型库。）

与大多数着色模型一样，这个示例也受到表面相对于观察方向和光照方向的朝向影响。用于着色时，这些方向通常表示为归一化的（单位长度）向量，如图 5.3 所示。


![图 5.3：着色模型的单位向量输入](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_1_5.3.png)

**图 5.3** 示例着色模型（以及大多数其他着色模型）的单位长度向量输入：表面法线 **n**、观察向量 **v** 和光照方向 **l**。

现在，我们已经定义了着色模型的所有输入，可以看看模型本身的数学定义了：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_be5356e037feab.png)


在这个方程中，我们使用了以下中间计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_78a2f2d5bc75a9.png)


这一定义中的若干数学表达式，也经常出现在其他着色模型中。钳制操作在着色中很常见，通常是将数值的下限钳制为 0，或者将数值钳制在 0 与 1 之间。这里，我们使用第 1.2 节介绍的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_a1d5680b19c40a.png) 记号，表示计算高光混合因子 s 时所用的、将数值钳制在 0 与 1 之间的操作。点积运算符出现了三次，每次都是对两个单位长度向量求点积；这是一种极其常见的模式。两个向量的点积等于它们的长度乘积，再乘以它们夹角的余弦。因此，两个单位长度向量的点积就是夹角的余弦，可以用来有效衡量两个向量方向一致的程度。在着色模型中，为了描述两个方向之间的关系，例如光照方向与表面法线之间的关系，由余弦构成的简单函数往往是效果最令人满意、也最准确的数学表达式。

另一种常见的着色操作，是根据一个介于 0 与 1 之间的标量值，在两种颜色之间进行线性插值。这种操作的形式为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_311816376389f2.png)：当 t 的值从 1 变到 0 时，结果相应地从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_4ee4ffca08d941.png) 插值到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_4c92dfa5297c2a.png)。这种模式在本着色模型中出现了两次：第一次在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_4ef0d5da85ddbd.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_501ea49e56ce89.png) 之间插值；第二次则在前一次插值的结果与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_ad848f17ccad06.png) 之间插值。线性插值在着色器中出现得如此频繁，以至于我们见过的每一种着色语言，都将它作为名为 `lerp` 或 `mix` 的内置函数。

“![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_01_d7fd2e01569473.png)”这一行计算反射光向量，也就是将 **l** 关于 **n** 反射。虽然它不像前两种操作那样常见，但使用频率也足够高，因此大多数着色语言同样提供了内置的 `reflect` 函数。

通过以不同方式组合这些操作，并配合各种数学表达式和着色参数，就能定义出产生极其丰富的风格化外观与真实感外观的着色模型。


## 5.2 光源

来源：英文原书《Real-Time Rendering, Fourth Edition》，书页106—116（PDF物理页127—137）；从5.2节标题开始，止于5.3节标题之前。PDF物理页138用于核对下一节边界。

光照对我们示例中的着色模型所起的作用相当简单：它为着色提供了一个主导方向。当然，现实世界中的光照可能非常复杂。场景中可能有多个光源，每个光源都有自己的大小、形状、颜色和强度；间接光照又会增加更多变化。正如我们将在第9章看到的，基于物理的照片级真实感着色模型需要考虑所有这些参数。

相比之下，风格化着色模型可以用许多不同方式使用光照，具体取决于应用的需求和视觉风格。有些高度风格化的模型可能完全没有光照概念，或者像我们的Gooch着色示例那样，只用它来提供某种简单的方向性。

光照复杂度的下一步，是让着色模型以二元方式对有光或无光作出反应。使用这种模型着色的表面，在受到光照时具有一种外观，在不受光照影响时则具有另一种外观。这意味着需要某些标准来区分这两种情况：与光源的距离、阴影遮挡（将在第7章讨论）、表面是否背向光源（即表面法线 **n** 与光照向量 **l** 之间的夹角是否大于90°），或者这些因素的某种组合。

从有光或无光的二元状态，进一步过渡到连续的光强尺度，只需很小的一步。可以将其表示为完全无光与完全有光之间的简单插值，这意味着光强具有一个有界范围，例如0到1；也可以将其表示为一个无界量，以其他方式影响着色。对于后一种情况，一种常见做法是把着色模型分成受光部分与未受光部分，并用光强 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_048dbb18c70c09.png) 对受光部分进行线性缩放：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_205620ff983aae.png)


这很容易扩展到RGB光源颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_9d9b9c24448333.png)


还可以扩展到多个光源：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_de595086ebb2b6.png)


未受光部分 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_1f659294ae78c8.png) 对应于把光照视作二元状态的着色模型中“不受光照影响时的外观”。根据所需的视觉风格和应用需求，它可以具有各种形式。例如，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_7fc623d0577198.png) 会使任何不受光源影响的表面呈现为纯黑色。或者，未受光部分也可以表达未受光物体的某种风格化外观，类似于Gooch模型为背向光源的表面赋予冷色。通常，着色模型的这一部分表达的是某种并非直接来自显式放置光源的光照，例如来自天空的光，或由周围物体反弹的光。这些其他形式的光照将在第10章和第11章讨论。

前面提到，如果光照方向 **l** 与表面法线 **n** 的夹角超过90°，光源便不会影响该表面点，实际上这相当于光从表面下方照来。这可以看作一种更普遍关系的特殊情况：光相对于表面的方向，与它对着色的影响之间存在关系。虽然这一关系具有物理依据，但它可以由简单的几何原理推导出来，而且也适用于许多非物理的风格化着色模型。


![图5.4 光线入射角与表面光线密度](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_2_5.4.png)

图5.4　上排图示给出了光照射到表面时的剖面视图。左图中光线正面垂直射向表面，中图中光线斜着射向表面，右图展示如何利用向量点积计算夹角的余弦。下图展示了剖面平面（其中包含光照向量和视线向量）相对于完整表面的位置关系。

> 译注：图注原文将下图剖面平面描述为包含“光照向量和视线向量”；原图实际标出的向量是 **l** 与 **n**（光照方向与法线）。这里保留原图注的表述，并标明这一疑点。

光对表面的作用可以可视化为一组光线，照到表面上的光线密度对应于表面着色所使用的光强。图5.4展示了一个受光表面的剖面。沿着该剖面，入射光线在表面上的间距与 **l** 和 **n** 夹角的余弦成反比。因此，照到表面上的总体光线密度与 **l** 和 **n** 夹角的余弦成正比；前面已经看到，这个余弦等于这两个单位长度向量的点积。这也说明了为什么把光照向量 **l** 定义为与光传播方向相反会很方便；否则，在计算点积之前，我们还必须先对它取负。

更准确地说，当点积为正时，光线密度（因而也包括该光对着色的贡献）与点积成正比。负值对应于从表面背后射来的光线，它们不产生影响。因此，在把光的着色贡献乘以光照点积之前，需要先将点积的下限钳制为0。使用第1.2节介绍的记号 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_b9875aa6a4d8ef.png)，即把负值钳制为零，可得：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c5964b20e09f01.png)


支持多个光源的着色模型通常采用式（5.5）或式（5.6）中的一种结构：前者更一般，后者则是基于物理的模型所必需的。式（5.6）也可能有利于风格化模型，因为它有助于确保光照在整体上保持一致，尤其是对于背向光源或处于阴影中的表面。不过，某些模型并不适合这种结构；这些模型会使用式（5.5）的结构。

函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_ec12a5c6777df5.png) 最简单的选择是令其为一个恒定颜色：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_af23a1f25d4dd7.png)


这样就得到以下着色模型：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_807808ebe710da.png)


这个模型的受光部分对应于朗伯着色模型（Lambertian shading model），该模型以约翰·海因里希·朗伯（Johann Heinrich Lambert）[967]命名，他早在1760年就发表了它！这个模型适用于理想漫反射表面，也就是完全哑光的表面。我们在这里对朗伯模型作了略为简化的解释，第9章将对它进行更严格的讨论。朗伯模型既可以单独用于简单着色，也是许多着色模型的重要组成部分。

从式（5.3）—（5.6）可以看出，光源通过两个参数与着色模型发生联系：指向光源的向量 **l**，以及光源颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png)。光源有多种不同类型，它们的主要区别在于这两个参数如何随场景中的位置变化。

接下来我们将讨论几种常用的光源类型，它们具有一个共同点：在给定的表面位置，每个光源都只从一个方向 **l** 照亮表面。换言之，从正在着色的表面位置看去，光源是一个无穷小的点。对于现实世界中的光源，这并不严格成立，但大多数光源的尺寸相对于它们与受光表面的距离都很小，因此这是一种合理的近似。在第7.1.2节和第10.1节，我们将讨论从一系列方向照亮同一表面位置的光源，也就是“面光源”。

### 5.2.1 方向光

方向光是最简单的光源模型。**l** 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 在整个场景中都保持不变，唯一的例外是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 可能因阴影遮挡而衰减。方向光没有位置。当然，实际光源在空间中都有特定位置。方向光是一种抽象，当光源距离相对于场景尺寸很大时，这种抽象就能很好地工作。例如，距离20英尺、照亮一个小型桌面立体模型的泛光灯，可以表示为方向光。另一个例子几乎就是所有由太阳照亮的场景，除非所讨论的场景大到类似太阳系内侧行星区域的尺度。

方向光的概念可以作一定扩展：让光照方向 **l** 保持不变，同时允许 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 的值变化。这样做通常出于性能或创作方面的原因，以便把光的影响限制在场景的某个特定部分。例如，可以用两个嵌套的盒状体积（一个位于另一个内部）定义一个区域：在外层盒子之外，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 等于(0, 0, 0)，即纯黑色；在内层盒子之内，它等于某个恒定值；而在两个盒子之间的区域内，则在这两个极值之间平滑插值。

### 5.2.2 局部点状光源

“Punctual light”并不是说光源赴约很准时，而是说这种光源与方向光不同，它具有位置。这类光源也没有几何尺寸，没有形状或大小，与现实世界中的光源不同。这里使用“punctual”一词，其词源是意为“点”的拉丁语 punctus，用它来指代所有从单一局部位置发出的光源构成的类别。我们用“点光源”（point light）一词表示其中一种特定的发光体：它向所有方向均匀发光。因此，点光源和聚光灯是局部点状光源的两种不同形式。光照方向向量 **l** 随当前着色表面点 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_263cc1e7cb1a71.png) 相对于局部点状光源位置 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_1bd49f56d1bde6.png) 的位置而变化：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_0c0ea92df91ab7.png)


这个等式是向量归一化的一个例子：将向量除以它的长度，得到一个指向相同方向的单位长度向量。这也是一种常见的着色运算；与上一节介绍的着色运算一样，大多数着色语言都将它作为内置函数提供。不过，有时我们需要这一运算中的某个中间结果，这就要求使用更基本的运算，分多个步骤显式执行归一化。将这种做法应用于局部点状光源方向的计算，可得：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_664ed477a6b59d.png)


由于两个向量的点积等于它们的长度之积乘以夹角余弦，而0°的余弦为1.0，因此一个向量与自身的点积就是其长度的平方。所以，要计算任意向量的长度，只需计算它与自身的点积，再对结果开平方。

我们需要的中间值是 r，即局部点状光源与当前着色点之间的距离。除了用于归一化光照向量以外，r 还用于计算光源颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 随距离变化的衰减（变暗）。下一部分将进一步讨论这一点。

#### 点光源／全向光源

向所有方向均匀发光的局部点状光源称为点光源或全向光源（omni light）。对于点光源，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 是距离 r 的函数，唯一的变化来源就是前面提到的距离衰减。图5.5用与图5.4中说明余弦因子类似的几何推理，展示了这种变暗现象为何发生。在给定表面上，来自点光源的光线间距与表面到光源的距离成正比。与图5.4中的余弦因子不同，这种间距增大在表面的两个维度上都会发生，因此光线密度（以及光源颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png)）与距离的平方倒数1/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_730f558bb648d5.png)成正比。这样，我们就可以用单个光源属性 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_e5ae74246baada.png) 来指定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 的空间变化；它被定义为在固定参考距离 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_01af0170b4ebbf.png) 处的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 值：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_2777690a9920f2.png)


式（5.11）通常称为光照的平方反比衰减。虽然从技术上说，这是点光源正确的距离衰减规律，但有一些问题使这个等式在实际着色中不够理想。

第一个问题发生在距离较小时。当 r 趋近于0，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 的值就会无界增大。当 r 达到0时，会出现除以零的奇点。为了解决这个问题，一种常见的修改是在分母中加入一个小值 ε [861]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_f021007bc56f03.png)


ε 的具体取值取决于应用；例如，Unreal游戏引擎使用 ε = 1 cm [861]。

> 译注：原书此处确实写作“ε = 1 cm”。按式（5.12）中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_730f558bb648d5.png) + ε 的量纲，ε 应与距离平方具有相同量纲。这里保留原文单位，不擅自更正。

CryEngine [1591]和Frostbite [960]游戏引擎采用另一种修改方式：将 r 的下限钳制为最小值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_fab91b715d9593.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5fba38e1aa3e9f.png)


前一种方法中 ε 的取值带有一定任意性，与之不同，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_fab91b715d9593.png) 有明确的物理解释：它是发光实体的半径。r 小于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_fab91b715d9593.png) 意味着着色表面穿入了实体光源内部，这是不可能的。


![图5.5 点光源的距离平方反比衰减](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_2_5.5.png)

图5.5　来自点光源的光线间距随距离 r 成比例增大。由于这种间距增大发生在两个维度上，光线密度（因而也包括光强）便按1/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_730f558bb648d5.png)的比例减小。

相比之下，平方反比衰减的第二个问题发生在距离较大时。问题不在视觉效果，而在性能。虽然光强随距离不断减小，但它永远不会降到0。为了高效渲染，我们希望光强在某个有限距离处达到0（第20章）。有许多方法可以修改平方反比等式来实现这一目标。理想情况下，这种修改应尽可能少地改变原来的规律。为了避免光源影响范围的边界出现突兀的截断，最好让修改后函数的导数和函数值在同一距离处都达到0。一种解决方案是将平方反比等式乘以一个具有所需性质的窗函数。Unreal Engine [861]和Frostbite [960]游戏引擎都使用下面这样的函数[860]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_ab921fd267e0ac.png)


上标 +2 表示：如果括号内的值为负，先将其钳制为0，然后再平方。图5.6展示了一条平方反比曲线、式（5.14）中的窗函数，以及两者相乘后的结果。


![图5.6 平方反比曲线与窗函数](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_2_5.6.png)

图5.6　此图展示了一条平方反比曲线（采用 ε 方法避免奇点，ε 的取值为1）、式（5.14）描述的窗函数（将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5dccbae88d09d7.png) 设为3），以及加窗后的曲线。

应用需求会影响方法的选择。例如，当距离衰减函数以较低的空间频率采样时（例如在光照贴图中，或逐顶点计算时），让导数在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5dccbae88d09d7.png) 处等于0尤为重要。CryEngine不使用光照贴图或顶点光照，所以它采用了更简单的调整：在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_08c8e98732ddd0.png) 到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5dccbae88d09d7.png) 的范围内，切换为线性衰减[1591]。

对于某些应用，匹配平方反比曲线并不是优先事项，因此它们会使用完全不同的函数。实际上，这相当于将式（5.11）—（5.14）推广为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_2afa4425136b26.png)


其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_ceb740e60ad75b.png) 是某个距离函数。这类函数称为距离衰减函数。在某些情况下，采用非平方反比衰减函数是出于性能限制。例如，游戏《正当防卫2》（Just Cause 2）需要计算成本极低的光源。因此，其衰减函数既要计算简单，又必须足够平滑，以避免逐顶点光照产生伪影[1379]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_a65be8ee425800.png)


在其他情况下，衰减函数的选择可能由创作上的考虑决定。例如，既用于写实游戏也用于风格化游戏的Unreal Engine提供了两种光照衰减模式：一种是式（5.12）所描述的平方反比模式；另一种是指数衰减模式，可以通过调整得到多种衰减曲线[1802]。游戏《古墓丽影》（Tomb Raider，2013）的开发者使用样条编辑工具制作衰减曲线[953]，从而对曲线形状获得更大的控制力。

#### 聚光灯

与点光源不同，现实世界中几乎所有光源的光照都会同时随方向和距离变化。这种方向变化可以表示为方向衰减函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_a34adc6f65bdc9.png)；它与距离衰减函数相结合，定义光强的总体空间变化：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5e4243b5c01d3a.png)


为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_a34adc6f65bdc9.png) 选择不同形式，可以产生各种光照效果。其中一种重要效果是聚光灯，它在一个圆锥体内投射光。聚光灯的方向衰减函数绕其方向向量 **s** 旋转对称，因此可以表示为角度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_3f63d90bc8b861.png) 的函数；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_3f63d90bc8b861.png) 是 **s** 与指向表面的反向光照向量 −**l** 之间的夹角。之所以要反转光照向量，是因为我们将表面处的 **l** 定义为指向光源，而这里需要的是背离光源的向量。

大多数聚光灯函数采用由 cos ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_3f63d90bc8b861.png) 组成的表达式；如前所述，这也是着色中表示角度最常见的形式。聚光灯通常具有本影角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5b3b02e204183e.png)，它限定光的范围，使得对所有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_3f63d90bc8b861.png) ≥ ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5b3b02e204183e.png) 都有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_cfcb98da0677c6.png)。这个角度可以用于剔除，其方式类似于前面介绍的最大衰减距离 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5dccbae88d09d7.png)。聚光灯通常还具有半影角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_b2d6a486bbd943.png)，它定义了一个光照保持全强度的内锥体。见图5.7。


![图5.7 聚光灯及其角度](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_2_5.7.png)

图5.7　一个聚光灯：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_3f63d90bc8b861.png) 是光源所定义的方向 **s** 与向量 −**l**（指向表面的方向）之间的夹角；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_b2d6a486bbd943.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5b3b02e204183e.png) 分别表示为光源定义的半影角与本影角。

聚光灯使用的方向衰减函数有许多种，但往往大致相似。例如，Frostbite游戏引擎[960]使用函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_2ee1a05ecad938.png)，而three.js浏览器图形库[218]使用函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_5390beeb968ba3.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_4ac68f01a02f12.png)


回顾第1.2节介绍的记号 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_a1d5680b19c40a.png)，它表示把 x 钳制在0与1之间。smoothstep函数是一个三次多项式，常用于着色中的平滑插值。大多数着色语言都将它作为内置函数提供。图5.8展示了目前为止我们讨论过的几种光源类型。


![图5.8 不同光源的照明效果](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_2_5.8.png)

图5.8　几种光源类型。从左到右：方向光、没有衰减的点光源，以及具有平滑过渡的聚光灯。注意，由于光照方向与表面之间的夹角发生变化，点光源照明在靠近边缘时会变暗。

#### 其他局部点状光源

局部点状光源的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 值还可以通过许多其他方式变化。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_a34adc6f65bdc9.png) 函数并不限于前面讨论的简单聚光灯衰减函数；它可以表示任何类型的方向变化，包括从现实世界光源测量得到、以复杂表格记录的分布模式。照明工程学会（Illuminating Engineering Society，IES）为这类测量定义了一种标准文件格式。许多灯具制造商都提供IES光度配置文件，它们已被用于游戏《杀戮地带：暗影坠落》（Killzone: Shadow Fall）[379, 380]，以及Unreal [861]和Frostbite [960]等游戏引擎。Lagarde对解析和使用这种文件格式的相关问题给出了很好的总结[961]。

游戏《古墓丽影》（2013）[953]有一种局部点状光源，沿世界坐标系的 x、y 和 z 轴分别对距离应用独立的衰减函数。在《古墓丽影》中，还可以应用曲线让光强随时间变化，例如生成闪烁的火炬效果。

第6.9节将讨论如何利用纹理来改变光强与颜色。

### 5.2.3 其他光源类型

方向光和局部点状光源的主要特征是光照方向 **l** 的计算方式。采用其他方式计算光照方向，就可以定义不同类型的光源。例如，除了前面提到的光源类型，《古墓丽影》还有胶囊光源，它使用线段而不是点作为光源[953]。对于每个着色像素，取指向该线段上最近点的方向作为光照方向 **l**。

只要着色器拥有可用于计算着色方程的 **l** 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_02_c53c80de5096d9.png) 值，就可以用任何方法来计算这些值。

到目前为止讨论的光源类型都是抽象模型。现实中的光源具有大小和形状，并从多个方向照亮表面点。在渲染中，这类光源称为面光源，它们在实时应用中的使用正在稳步增加。面光源渲染技术分为两类：一类模拟面光源被部分遮挡时产生的阴影边缘软化（第7.1.2节）；另一类模拟面光源对表面着色的影响（第10.1节）。第二类光照效果在平滑、类似镜面的表面上最为显著，因为可以在反射中清楚辨认光源的形状和大小。虽然方向光和局部点状光源不像过去那样无处不在，但它们不太可能被弃用。人们已经开发出一些考虑光源面积、实现成本又相对较低的近似方法，因此这些方法正得到更广泛的应用。GPU性能的提高，也使得采用比过去更精细的技术成为可能。


## 5.3 实现着色模型

本节来源：《Real-Time Rendering, Fourth Edition》书页 117—130（PDF 物理页 138—151）。范围包括 5.3.1—5.3.3，截止于 5.4 标题之前；插图按所属内容就近安排。

这些着色和光照方程当然必须用代码实现，才能发挥作用。本节将介绍设计和编写这些实现时的一些关键考虑因素，并逐步讲解一个简单的实现示例。

### 5.3.1 求值频率

设计着色实现时，需要根据求值频率划分各项计算。首先，判断某项计算的结果在整个绘制调用期间是否始终不变。如果是，这项计算就可以由应用程序完成，通常在 CPU 上执行；不过，对于代价特别高的计算，也可以使用 GPU 计算着色器。计算结果通过 uniform 着色器输入传给图形 API。

即使在这一类别内部，可能的求值频率也有很大的跨度，最低是“总共只计算一次”。最简单的例子是着色方程中的常量子表达式，但任何基于硬件配置、安装选项等很少变化的因素的计算，也都可能属于这种情况。这类着色计算可以在编译着色器时完成，此时甚至不必设置 uniform 着色器输入。另一种做法是在离线预计算阶段、安装时，或者应用程序加载时完成计算。

另一种情况是，着色计算的结果在应用程序运行期间会变化，但变化得很慢，没有必要每一帧都更新。例如，依赖虚拟游戏世界中一天内时刻的光照因素。如果计算代价很高，可以考虑将其分摊到多个帧中完成。

还有一些计算每帧执行一次，例如将观察矩阵与透视矩阵连接起来；一些每个模型执行一次，例如更新依赖位置的模型光照参数；还有一些每次绘制调用执行一次，例如更新一个模型中各个材质的参数。按求值频率对 uniform 着色器输入分组，有助于提高应用程序效率，也可以通过尽量减少常量更新来改善 GPU 性能 [1165]。

如果某项着色计算的结果会在一次绘制调用内部变化，就不能通过 uniform 着色器输入传递给着色器，而必须由第 3 章所述的某个可编程着色器阶段计算；必要时，再通过 varying 着色器输入传给其他阶段。理论上，任何可编程阶段都可以执行着色计算，而各阶段分别对应不同的求值频率：

- 顶点着色器：对曲面细分前的每个顶点求值。
- 外壳着色器：对每个曲面片求值。
- 域着色器：对曲面细分后的每个顶点求值。
- 几何着色器：对每个图元求值。
- 像素着色器：对每个像素求值。


![逐像素与逐顶点着色对比](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_3_5.9.png)

图 5.9。对三个顶点密度不同的模型，比较式 5.19 所示示例着色模型的逐像素求值与逐顶点求值。左列显示逐像素求值的结果，中列显示逐顶点求值的结果，右列则给出各模型的线框渲染，以展示顶点密度。（中国龙网格来自 Computer Graphics Archive [1172]，原始模型来自 Stanford 3D Scanning Repository。）

实际中，大多数着色计算都逐像素执行。它们通常在像素着色器中实现，但使用计算着色器的实现也越来越常见；第 20 章将讨论几个例子。其他阶段主要用于变换、变形等几何操作。为了理解其中的原因，我们将比较逐顶点与逐像素着色求值的结果。在较早的文献中，它们有时分别被称为 Gouraud 着色 [578] 和 Phong 着色 [1414]，不过如今这些术语已经不常使用。这里用于比较的着色模型与式 5.1 有些相似，但经过修改，可以处理多个光源。稍后详细介绍实现示例时，将给出完整模型。

图 5.9 展示了在顶点密度相差很大的模型上，逐像素与逐顶点着色的结果。龙模型的网格极为密集，因此两者差别很小。但是，在茶壶模型上，逐顶点着色求值会产生棱角分明的高光等明显错误；在仅由两个三角形构成的平面上，逐顶点着色版本显然是错误的。这些错误的原因在于，着色方程中的某些部分，尤其是高光，其数值在网格表面上呈非线性变化。因此它们不适合放在顶点着色器中计算，因为顶点着色器的结果会先在三角形上进行线性插值，再送入像素着色器。


![法线插值的长度与方向问题](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_3_5.10.png)

图 5.10。左图表明，在表面上对单位法线进行线性插值，得到的插值向量长度会小于 1。右图表明，对长度差异显著的法线进行线性插值，会使插值后的方向偏向两条法线中较长的一条。

原则上，可以只在像素着色器中计算着色模型的镜面高光部分，而把其余部分放在顶点着色器中计算。这很可能不会产生视觉伪影，理论上也可以节省一些计算。但实际中，这种混合实现往往并非最优。着色模型中线性变化的部分，通常恰好是计算代价最低的部分；以这种方式拆分着色计算，往往会引入足够多的额外开销，例如重复计算和新增的 varying 输入，使其抵消原本的收益。

如前所述，在大多数实现中，顶点着色器负责几何变换、变形等非着色操作。由此得到的几何表面属性会被变换到合适的坐标系，由顶点着色器输出，在三角形上进行线性插值，再作为 varying 着色器输入传入像素着色器。这些属性通常包括表面位置、表面法线；如果法线映射需要，还可以包括表面切向量。

注意，即使顶点着色器始终生成单位长度的表面法线，插值仍然可能改变其长度，见图 5.10 左侧。因此，需要在像素着色器中重新归一化法线，即将长度缩放为 1。不过，顶点着色器生成的法线长度仍然很重要。如果顶点之间的法线长度差异显著，例如受到顶点混合副作用的影响，插值就会发生偏斜，见图 5.10 右侧。由于这两种影响，实现中往往会在插值之前和之后都对被插值的向量归一化，也就是在顶点着色器和像素着色器中都进行归一化。

与表面法线不同，指向特定位置的向量，例如观察向量、点状光源的光照向量，通常不进行插值。相反，像素着色器使用插值后的表面位置来计算这些向量。除归一化以外，每个向量只需要一次很快的向量减法；而我们已经看到，无论如何都必须在像素着色器中进行归一化。如果出于某种原因必须对这些向量进行插值，不要事先将它们归一化，否则会得到错误结果，如图 5.11 所示。


![光照向量插值](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_3_5.11.png)

图 5.11。两个光照向量之间的插值。左图中，在插值前将它们归一化，导致插值后的方向错误。右图中，对未经归一化的向量进行插值，得到正确结果。

前面提到，顶点着色器把表面几何变换到“合适的坐标系”中。通过 uniform 变量传给像素着色器的相机位置和光源位置，通常也由应用程序变换到同一坐标系。这可以尽量减少像素着色器为把着色模型的全部向量统一到同一坐标空间而执行的工作。但是，哪个坐标系才“合适”呢？可选项包括全局世界空间、相机的局部坐标系，以及较少采用的当前所渲染模型的局部坐标系。通常会基于性能、灵活性、简洁性等系统层面的考虑，为整个渲染系统作出统一选择。例如，如果预计所渲染的场景将包含数量极多的光源，可以选择世界空间，以避免变换光源位置。也可以优先选择相机空间，以便更好地优化像素着色器中与观察向量有关的操作，并可能提高精度，见 16.6 节。

虽然大多数着色器实现，包括接下来要讨论的示例，都遵循上述总体框架，但当然也存在例外。例如，一些应用程序出于风格考虑，选择逐图元着色求值产生的分面外观。这种风格通常称为平面着色。图 5.12 给出了两个例子。

原则上，平面着色可以在几何着色器中完成，但近来的实现通常使用顶点着色器。具体做法是将每个图元的属性关联到其第一个顶点，并禁用顶点值的插值。禁用插值可以分别针对每一个顶点值进行设置；这样，第一个顶点的值就会传递到该图元内的所有像素。


![采用平面着色的游戏](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_3_5.12.png)

图 5.12。两款将平面着色作为风格选择的游戏：上图是《Kentucky Route Zero》，下图是《That Dragon, Cancer》。（上图由 Cardboard Computer 提供，下图由 Numinous Games 提供。）

### 5.3.2 实现示例

现在介绍一个着色模型的实现示例。如前所述，我们要实现的着色模型类似于式 5.1 中的扩展 Gooch 模型，但经过修改以支持多个光源。其表达式为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_ee5da842b54ca8.png)


其中包含以下中间计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_6f0d73fef78b83.png)


这一表达式符合式 5.6 的多光源结构，为方便起见，在此重列：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_26f448e418350b.png)


此时，受光项和不受光项为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_8a54a196abf47c.png)


这里调整了冷色的不受光贡献，使结果在外观上更接近原始方程。

在大多数典型渲染应用程序中，表面颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_9b0b16be612ab5.png) 等材质属性的可变值存储在顶点数据中，或者更常见地存储在纹理中，见第 6 章。不过，为了使这个实现示例保持简单，我们假定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_9b0b16be612ab5.png) 在整个模型上恒定。

这个实现将使用着色器的动态分支能力，循环遍历所有光源。对于相当简单的场景，这种直接的方法能够很好地工作；但它不能很好地扩展到包含大量光源、几何复杂的大场景。第 20 章将介绍高效处理大量光源的渲染技术。另外，为简单起见，我们只支持一种光源：点光源。虽然实现很简单，但仍遵循前面介绍的最佳实践。

着色模型并不是孤立实现的，而是处于一个更大的渲染框架中。本例在一个简单的 WebGL 2 应用程序中实现，修改自 Tarek Sherif 的 WebGL 2 示例“Phong-shaded Cube”[1623]；相同原则也适用于更复杂的框架。

我们将讨论应用程序中的一些 GLSL 着色器代码和 JavaScript WebGL 调用。目的不是讲授 WebGL API 的具体用法，而是展示通用的实现原则。我们将按“由内向外”的顺序介绍：先看像素着色器，再看顶点着色器，最后看应用程序端的图形 API 调用。

在正式的着色器代码之前，着色器源码包含输入与输出的定义。如 3.3 节所述，按 GLSL 的术语，着色器输入分为两类。一类是 uniform 输入，由应用程序设置数值，在一次绘制调用中保持不变。另一类是 varying 输入，其数值可以在不同的着色器调用之间变化，也就是在像素或顶点之间变化。下面给出像素着色器的 varying 输入及输出定义；在 GLSL 中，这些输入用 `in` 标记：

```glsl
in vec3 vPos;
in vec3 vNormal;
out vec4 outColor;
```

这个像素着色器只有一个输出，即最终的着色颜色。像素着色器输入与顶点着色器输出相匹配，后者先在三角形上进行插值，然后才送入像素着色器。该像素着色器有两个 varying 输入：表面位置和表面法线，两者都位于应用程序的世界空间坐标系中。uniform 输入的数量要多得多，因此为简洁起见，我们只展示其中两个的定义，它们都与光源有关：

```glsl
struct Light {
    vec4 position;
    vec4 color;
};
uniform LightUBlock {
    Light uLights[MAXLIGHTS];
};
uniform uint uLightCount;
```

由于这些是点光源，每个光源的定义都包含位置和颜色。它们被定义为 `vec4` 而不是 `vec3`，以符合 GLSL 的 `std140` 数据布局标准的限制。虽然 `std140` 布局可能像本例一样浪费一些空间，但它简化了确保 CPU 与 GPU 数据布局一致的工作，这就是本例采用它的原因。`Light` 结构体数组定义在一个具名 uniform 块内部；这是 GLSL 的一项功能，用来将一组 uniform 变量绑定到缓冲区对象，以加快数据传输。数组长度设为应用程序允许在一次绘制调用中使用的最大光源数量。后面将看到，应用程序在编译着色器之前，会将着色器源码中的字符串 `MAXLIGHTS` 替换为正确的数值，本例中为 10。uniform 整数 `uLightCount` 则是这次绘制调用中实际启用的光源数量。

接下来看像素着色器代码：

```glsl
vec3 lit(vec3 l, vec3 n, vec3 v) {
    vec3 r_l = reflect(-l, n);
    float s = clamp(100.0 * dot(r_l, v) - 97.0, 0.0, 1.0);
    vec3 highlightColor = vec3(2,2,2);
    return mix(uWarmColor, highlightColor, s);
}
void main() {
    vec3 n = normalize(vNormal);
    vec3 v = normalize(uEyePosition.xyz - vPos);
    outColor = vec4(uFUnlit, 1.0);
    for (uint i = 0u; i < uLightCount; i++) {
        vec3 l = normalize(uLights[i].position.xyz - vPos);
        float NdL = clamp(dot(n, l), 0.0, 1.0);
        outColor.rgb += NdL * uLights[i].color.rgb * lit(l,n,v);
    }
}
```

这里定义了一个计算受光项的函数，由 `main()` 调用。总体而言，这是式 5.20 和式 5.21 的直接 GLSL 实现。注意，不受光函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_1267c5be6815dd.png) 的值和暖色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_4ef0d5da85ddbd.png) 通过 uniform 变量传入。由于它们在整个绘制调用中恒定，可以由应用程序计算，从而节省一些 GPU 周期。

该像素着色器使用了几个 GLSL 内置函数。`reflect()` 将一个向量相对于由第二个向量定义的平面进行反射；本例中，前者是光照向量，后者是表面法线。由于我们希望光照向量和反射向量都指向远离表面的方向，必须在传入 `reflect()` 之前将光照向量取负。`clamp()` 有三个输入，其中两个定义一个范围，将另一个输入限制在这个范围内。在大多数 GPU 上，将数值限制到 0—1 范围这一特殊情况，对应 HLSL 的 `saturate()` 函数，执行很快，往往实际上没有额外代价。因此我们在这里使用它，尽管对于已知不会超过 1 的值，只需限制下限为 0。`mix()` 也有三个输入：它根据第三个输入，也就是 0—1 之间的混合参数，对另外两个输入进行线性插值；本例中，被插值的是暖色和高光颜色。在 HLSL 中，此函数称为 `lerp()`，即“线性插值”的缩写。最后，`normalize()` 将向量除以其长度，使其长度缩放为 1。

现在看顶点着色器。由于已经看过像素着色器中 uniform 定义的例子，这里不再展示顶点着色器的 uniform 定义，但值得看看 varying 输入和输出的定义：

```glsl
layout(location=0) in vec4 position;
layout(location=1) in vec4 normal;
out vec3 vPos;
out vec3 vNormal;
```

注意，如前所述，顶点着色器输出与像素着色器的 varying 输入相匹配。输入中还包含指定数据在顶点数组中如何布局的指令。接下来是顶点着色器代码：

```glsl
void main() {
    vec4 worldPosition = uModel * position;
    vPos = worldPosition.xyz;
    vNormal = (uModel * normal).xyz;
    gl_Position = viewProj * worldPosition;
}
```

这些是顶点着色器中的常见操作。着色器将表面位置和法线变换到世界空间，并传给像素着色器用于着色。最后，将表面位置变换到裁剪空间，再传给 `gl_Position`。它是供光栅化器使用的特殊系统定义变量，也是任何顶点着色器都必须提供的那个输出。

注意，顶点着色器没有将法线向量归一化。这里不需要归一化，因为原始网格数据中的法线长度为 1，而且本应用程序没有执行顶点混合、非均匀缩放等会不均匀地改变其长度的操作。模型矩阵可以包含均匀缩放因子，但这会成比例地改变所有法线的长度，因此不会产生图 5.10 右侧所示的问题。

应用程序使用 WebGL API 完成各种渲染和着色器设置。每个可编程着色器阶段分别设置，然后全部绑定到一个程序对象。以下是像素着色器的设置代码：

```javascript
var fSource = document.getElementById("fragment").text.trim();
var maxLights = 10;
fSource = fSource.replace(/MAXLIGHTS/g, maxLights.toString());
var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fSource);
gl.compileShader(fragmentShader);
```

注意代码中对“fragment shader”（片元着色器）的引用。这是 WebGL 以及作为其基础的 OpenGL 使用的术语。如本书前面所述，虽然“pixel shader”（像素着色器）在某些方面不够精确，但它的使用更普遍，所以本书沿用这一术语。字符串 `MAXLIGHTS` 也在这里被替换为合适的数值。大多数渲染框架都会执行类似的编译前着色器处理。

应用程序端还有设置 uniform、初始化顶点数组、清除、绘制等更多代码，可以查看程序 [1623]；许多 API 指南也有相应解释。这里的目标，是让读者了解着色器如何被当作独立的处理器对待，并拥有各自的编程环境。因此，我们的逐步讲解到此结束。

### 5.3.3 材质系统

渲染框架很少像我们的简单示例那样只实现一个着色器。通常需要一个专用系统，处理应用程序使用的各种材质、着色模型和着色器。

如前几章所述，着色器是运行于 GPU 某个可编程着色器阶段的程序。因此，它是低层图形 API 资源，而不是美术人员会直接交互的对象。相比之下，材质是面向美术人员、对表面视觉外观的封装。有时材质还描述碰撞属性等非视觉方面，但这些超出了本书范围，不再深入讨论。

虽然材质通过着色器实现，但两者并非简单的一一对应关系。在不同渲染情形下，同一个材质可能使用不同的着色器；一个着色器也可能被多个材质共享。最常见的情况是参数化材质。在最简单的形式中，材质参数化需要两类材质实体：材质模板和材质实例。每个材质模板描述一类材质，并包含一组参数；根据参数类型，可以为其赋予数值、颜色或纹理值。每个材质实例则对应一个材质模板，加上该模板所有参数的一组具体取值。一些渲染框架，例如 Unreal Engine [1802]，允许更复杂的层次结构，使材质模板可以在多个层级上从其他模板派生。

参数可以在运行时通过向着色器程序传入 uniform 输入来确定，也可以在编译时通过在着色器编译前替换数值来确定。一种常见的编译时参数是布尔开关，用于控制某项材质功能是否启用。美术人员可以通过材质用户界面中的复选框设置它，材质系统也可以以程序化方式设置。例如，对于某功能的视觉效果可以忽略的远处物体，将其关闭以降低着色器开销。

材质参数可能与着色模型参数一一对应，但并非总是如此。材质可以把某个着色模型参数，例如表面颜色，固定为常量。另一种情况是，通过一系列复杂操作，以多个材质参数以及插值后的顶点值或纹理值为输入，计算出一个着色模型参数。有时，表面位置、表面朝向，甚至时间，也会参与计算。基于表面位置和朝向的着色在地形材质中尤其常见。例如，可以利用高度和表面法线控制积雪效果，在高海拔的水平和接近水平的表面上混入白色表面颜色。基于时间的着色则常见于动画材质，例如闪烁的霓虹灯招牌。

材质系统最重要的任务之一，是把各种着色器功能划分为独立元素，并控制它们如何组合。在很多情况下，这类组合都很有用，包括：

- 将表面着色与几何处理组合，例如刚体变换、顶点混合、变形、曲面细分、实例化和裁剪。这些功能彼此独立变化：表面着色依赖材质，几何处理依赖网格。因此，分别编写它们，再由材质系统按需组合，会很方便。
- 将表面着色与像素丢弃、混合等合成操作组合。这对移动 GPU 尤其相关，因为它们通常在像素着色器中执行混合。通常希望能够独立于表面着色所用的材质来选择这些操作。
- 将计算着色模型参数的操作与着色模型本身的计算组合。这样只需编写一次着色模型实现，就可以将它与各种不同的着色模型参数计算方法组合使用。
- 将可以单独选择的材质功能彼此组合，并与选择逻辑及着色器其余部分组合。这样就可以分别编写各项功能的实现。
- 将着色模型及其参数计算与光源求值组合，也就是在着色点为每个光源计算光照颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_c53c80de5096d9.png) 和光照方向 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_6129e5f9e9210f.png)。延迟渲染等技术，见第 20 章，会改变这种组合的结构。在支持多种此类技术的渲染框架中，这又增加了一层复杂性。

如果图形 API 能把这种着色器代码模块化作为核心功能提供，事情就会很方便。遗憾的是，与 CPU 代码不同，GPU 着色器不允许在编译之后链接代码片段。每个着色器阶段的程序都作为一个整体编译。不同着色器阶段之间的分离确实提供了有限的模块化，在一定程度上符合上述列表的第一项：将通常在像素着色器中执行的表面着色，与通常在其他着色器阶段执行的几何处理组合起来。但这种对应并不完美，因为每个着色器还执行其他操作，而其他类型的组合仍需要处理。鉴于这些限制，材质系统要实现所有这些组合类型，唯一的方法就是在源码层面完成。这主要涉及连接、替换等字符串操作，往往通过 C 风格的预处理指令执行，例如 `#include`、`#if` 和 `#define`。

早期渲染系统的着色器变体相对较少，而且通常逐个手工编写。这有一些好处，例如可以在充分了解最终着色器程序的情况下优化每个变体。但是，随着变体数量增长，这种方法很快就不再实际。考虑所有不同的组成部分和选项之后，可能出现的着色器变体数量极其庞大。这就是模块化和可组合性如此重要的原因。

设计处理着色器变体的系统时，首先需要解决的问题是：不同选项之间的选择，是在运行时通过动态分支完成，还是在编译时通过条件预处理完成。在较早的硬件上，动态分支往往无法实现，或者极为缓慢，因此运行时选择不可行。当时所有变体都在编译时处理，包括不同光源类型数量的所有可能组合 [1193]。

相比之下，当今的 GPU 能够很好地处理动态分支，尤其是在一次绘制调用中所有像素的分支行为都相同时。如今，光源数量等许多功能差异都在运行时处理。然而，向一个着色器中加入大量功能变化，会带来另一种代价：寄存器数量增加，占用率随之下降，性能也因而降低。详见 18.4.5 节。因此，编译时变体仍然很有价值：它可以避免包含永远不会执行的复杂逻辑。

举例来说，假设一个应用程序支持三种不同类型的光源。其中两种很简单：点光源和平行光源。第三种是广义聚光灯，支持表格化照明分布和其他复杂功能，需要大量着色器代码才能实现。但假设这种广义聚光灯用得相对较少，占应用程序中全部光源的比例不到 5%。过去，为了避免动态分支，会针对三种光源数量的每一种可能组合分别编译一个着色器变体。如今虽然不再需要这样做，但编译两个独立变体可能仍然有益：一个用于广义聚光灯数量大于或等于 1 的情况，另一个用于其数量恰好为 0 的情况。第二个变体的代码更简单，而且最常使用，因此很可能占用更少的寄存器，从而具有更高性能。

现代材质系统同时采用运行时和编译时着色器变体。虽然全部负担已不再仅由编译时处理承担，但总体复杂度和变化数量仍在持续增长，因此仍需编译大量着色器变体。例如，在游戏《Destiny: The Taken King》的某些区域，单帧使用的已编译着色器变体超过 9000 个 [1750]。可能的变体数量还可以大得多，例如 Unity 渲染系统中的某些着色器有接近 1000 亿种可能的变体。只有实际使用的变体才会被编译，但仍不得不重新设计着色器编译系统，以处理数量如此庞大的可能变体 [1439]。

材质系统设计者采用不同策略来实现这些设计目标。虽然有时这些策略被描述为彼此排斥的系统架构 [342]，但它们可以，而且通常确实会，在同一个系统中结合使用。这些策略包括：

- **代码复用**：在共享文件中实现函数，使用 `#include` 预处理指令，让任何需要这些函数的着色器都能访问它们。
- **减法式**：用一个常被称为 übershader 或 supershader，即超级着色器的着色器 [1170, 1784]，汇集大量功能，再结合编译时预处理条件和动态分支，去掉不使用的部分，并在相互排斥的选项之间切换。
- **加法式**：将各种功能片段定义为具有输入、输出连接端口的节点，再将这些节点组合起来。这类似于代码复用策略，但结构更明确。可以通过文本 [342] 或可视化图编辑器组合节点。后者旨在让技术美术等非工程人员更容易创建新的材质模板 [1750, 1802]。通常只有着色器的一部分允许通过可视化图编写。例如，在 Unreal Engine 中，图编辑器只能影响着色模型输入的计算 [1802]，见图 5.13。
- **基于模板**：定义一个接口，只要符合接口，就可以接入不同的实现。这比加法式策略更加正式一些，通常用于较大的功能块。一个常见的接口例子，是将着色模型参数的计算与着色模型本身的计算分离。Unreal Engine [1802] 有不同的“材质域”，包括用于计算着色模型参数的 Surface 域，以及 Light Function 域；后者计算一个标量，用于调制给定光源的光照颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_03_c53c80de5096d9.png)。Unity 中也存在类似的“表面着色器”结构 [1437]。注意，第 20 章讨论的延迟着色技术强制采用类似结构，其中 G 缓冲区充当接口。


![Unreal Engine 材质编辑器](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_3_5.13.png)

图 5.13。Unreal Engine 材质编辑器。请注意节点图右侧那个高大的节点。该节点的输入连接端口对应渲染引擎使用的各种着色输入，其中包括全部着色模型参数。（材质示例由 Epic Games 提供。）

若要了解更多具体例子，可以阅读《WebGL Insights》[301]，这本书如今已免费开放；其中若干章讨论了各种引擎如何控制自己的着色器流水线。除了组合之外，现代材质系统还有其他一些重要的设计考虑，例如需要在尽量不重复着色器代码的情况下支持多个平台。这包括为适应平台、着色语言和 API 之间的性能与能力差异而作出的功能变化。《Destiny》的着色器系统 [1750] 是解决这类问题的代表性方案。它使用专有的预处理器层，接收用定制着色语言方言编写的着色器。这使开发者能够编写平台无关的材质，并自动将其翻译为不同的着色语言和实现。Unreal Engine [1802] 和 Unity [1436] 也有类似系统。

材质系统还需要保证良好的性能。除了对着色变体进行专门编译之外，材质系统还可以执行其他几种常见优化。《Destiny》着色器系统和 Unreal Engine 会自动检测在一次绘制调用中恒定的计算，例如前述实现示例中的暖色与冷色计算，并将它们移到着色器之外。另一个例子是《Destiny》使用的作用域系统，它区分以不同频率更新的常量，例如每帧一次、每个光源一次、每个物体一次，并在合适的时机更新各组常量，以减少 API 开销。

我们已经看到，实现一个着色方程，需要决定哪些部分可以简化、各种表达式应以什么频率计算，以及用户能够如何修改和控制外观。渲染流水线最终输出颜色值和混合值。接下来有关抗锯齿、透明和图像显示的各节，将详细介绍这些值如何组合并修改，以供显示。


## 5.4 走样与抗锯齿

来源：《Real-Time Rendering, Fourth Edition》，书页 130—148（PDF 物理页 151—169）。本节从 5.4 标题开始，至 5.5 标题之前结束，包含 5.4.1、5.4.2 及其全部下级内容。技术表述保留原书出版时的背景。

设想一个很大的黑色三角形在白色背景上缓慢移动。当三角形覆盖一个屏幕网格单元时，代表该单元的像素值，其强度应当平滑下降。然而，在各种类型的基础渲染器中，通常发生的情况却是：网格单元中心一被覆盖，像素颜色便立即从白色变为黑色。标准 GPU 渲染也不例外。见图 5.14 最左一列。

三角形在像素中的表现只有“存在”或“不存在”两种状态。绘制线条时也有类似问题。因此，边缘看起来呈锯齿状，这种视觉伪影被称为“锯齿”（jaggies）；在动画中，它们又变成了“爬动的锯齿”（crawlies）。更正式地说，这一问题称为**走样**（aliasing），旨在避免它的措施称为**抗锯齿技术**（antialiasing techniques）。

采样理论与数字滤波的内容足以单独写成一本书 [559, 1447, 1729]。由于这是渲染中的一个关键领域，我们将介绍采样与滤波的基本理论，然后着重讨论当前能够实时执行、用于减轻走样伪影的方法。


![图5.14 不同采样数的抗锯齿效果](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.14.png)

**图 5.14** 上排显示三角形、一条线以及一些点在三种不同抗锯齿程度下的图像。下排是上排图像的放大版本。最左列每个像素仅使用一个样本，也就是没有使用抗锯齿。中间一列每个像素使用四个样本，以网格模式排列；最右列每个像素使用八个样本，采用 4 × 4 棋盘格，在其中一半格子内采样。

### 5.4.1 采样与滤波理论

渲染图像的过程，本质上是一项采样任务。这是因为，生成图像就是对三维场景进行采样，以获得图像中每个像素的颜色值，而图像是一个离散像素数组。使用纹理映射时（第 6 章），必须对纹素重新采样，才能在各种条件下取得良好结果。为了生成动画中的图像序列，通常还会按均匀的时间间隔对动画采样。本节介绍采样、重建与滤波。为简单起见，大部分内容将以一维情况说明。这些概念可以自然推广到二维，因此也可用于处理二维图像。

图 5.15 显示了如何以均匀间隔对连续信号采样，也就是将其离散化。这一采样过程的目标是以数字形式表示信息。在此过程中，信息量会减少。不过，为了恢复原始信号，必须对采样后的信号进行**重建**。这是通过对采样信号进行**滤波**实现的。


![图5.15 采样与重建](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.15.png)

**图 5.15** 对连续信号（左）进行采样（中），然后通过重建恢复原始信号（右）。

只要进行采样，就可能发生走样。这是不希望出现的伪影，我们必须抑制走样，才能生成赏心悦目的图像。老西部片中一个经典例子，是电影摄影机拍摄旋转的车轮。由于辐条运动速度远高于摄影机记录图像的速度，车轮可能看起来在缓慢旋转，方向或向后、或向前，甚至看起来完全没有转动。图 5.16 展示了这一现象。这种效应源于车轮图像是在一系列离散时间步上拍摄的，称为**时间走样**（temporal aliasing）。


![图5.16 旋转车轮的时间走样](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.16.png)

**图 5.16** 第一排显示旋转的车轮，即原始信号。第二排对它的采样不足，使车轮看起来沿相反方向运动，这是采样率过低造成走样的一个例子。第三排的采样率恰好是每转两次采样，我们无法确定车轮朝哪个方向旋转。这就是奈奎斯特极限。第四排的采样率高于每转两次采样，于是我们突然能够看出车轮正沿正确方向旋转。

计算机图形学中常见的走样例子包括：光栅化线条或三角形边缘上的“锯齿”、被称为“萤火虫”的闪烁高光，以及棋盘格纹理缩小时出现的现象（6.2.2 节）。

当信号以过低的频率采样时，就会发生走样。此时，采样后的信号看起来像是一个比原信号频率更低的信号，图 5.17 对此作了说明。为了正确采样一个信号，也就是说，能够从样本中重建原始信号，采样频率必须高于被采样信号最高频率的两倍。这通常称为**采样定理**，这一采样频率称为**奈奎斯特率** [1447] 或**奈奎斯特极限**，以瑞典科学家 Harry Nyquist（1889—1976）的名字命名；他于 1928 年发现了这一规律。图 5.16 也展示了奈奎斯特极限。定理中使用“最高频率”一词，意味着信号必须是**带限的**，也就是不存在高于某个界限的频率。换句话说，相对于相邻样本之间的间距，信号必须足够平滑。


![图5.17 信号的欠采样](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.17.png)

**图 5.17** 蓝色实线是原始信号，红色圆点是均匀分布的采样点，绿色虚线是重建信号。上图的采样率过低，因此重建信号看起来具有更低的频率，也就是原始信号的一个混叠信号。下图的采样率恰好为原始信号频率的两倍，此时重建信号是一条水平线。可以证明，只要再把采样率提高哪怕一点点，就能够进行完美重建。

使用点样本渲染时，三维场景通常并不是带限的。三角形边缘、阴影边界等现象会产生不连续变化的信号，因此其频率可延伸至无穷高 [252]。而且，无论样本排得多密，物体仍然可能小到根本没有被采到。因此，使用点样本渲染场景时，不可能完全避免走样，而我们几乎总是在使用点采样。不过，有时可以判断信号是否带限。一个例子是把纹理应用到表面上：可以计算纹理样本的频率相对于像素采样率的关系。如果该频率低于奈奎斯特极限，就不必采取特殊措施，也能正确采样纹理。如果频率过高，则可用多种算法对纹理进行带限处理（6.2.2 节）。

#### 重建

给定一个带限的采样信号，我们现在讨论如何从采样信号重建原始信号。为此必须使用滤波器。图 5.18 展示了三种常用滤波器。注意，滤波器的面积应始终为 1，否则重建信号的幅值可能会增大或缩小。


![图5.18 三种重建滤波器](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.18.png)

**图 5.18** 左上是盒式滤波器，右上是帐篷滤波器。下方是 sinc 滤波器，此处沿 x 轴截取了有限的显示范围。

图 5.19 使用盒式滤波器，即最近邻滤波器，重建采样信号。这是效果最差的滤波器，因为生成的信号是不连续的阶梯形。尽管如此，它因简单而经常用于计算机图形学。如图所示，将盒式滤波器放到每个采样点上，再进行缩放，使滤波器顶部与采样点重合。所有这些经过缩放、平移的盒函数相加，就得到右侧的重建信号。


![图5.19 盒式滤波器重建](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.19.png)

**图 5.19** 使用盒式滤波器重建采样信号（左）。方法是在每个采样点处放置盒式滤波器，并沿 y 方向缩放，使滤波器高度与该采样点相同。求和后得到重建信号（右）。

盒式滤波器可以替换为其他任何滤波器。图 5.20 使用帐篷滤波器，也称三角形滤波器，重建采样信号。注意，这个滤波器在相邻采样点之间实现线性插值，因此优于盒式滤波器，因为此时重建信号是连续的。


![图5.20 帐篷滤波器重建](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.20.png)

**图 5.20** 使用帐篷滤波器重建采样信号（左），重建结果显示在右侧。

然而，帐篷滤波器重建信号的平滑性仍然较差；在采样点处，斜率会突然变化。这是因为帐篷滤波器并不是完美的重建滤波器。要实现完美重建，必须使用**理想低通滤波器**。信号的一个频率分量是一条正弦波 sin(2πf)，其中 f 是该分量的频率。由此，低通滤波器会去除频率高于滤波器所定义某一频率的所有分量。直观而言，低通滤波器会移除信号中的尖锐特征，也就是对其进行模糊。理想低通滤波器就是 sinc 滤波器（图 5.18 下方）：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_b205ea489bf320.png)


> 译注：上文正弦波的写法 sin(2πf) 忠实保留原文，原书此处未显式写出时间或空间自变量。式（5.22）在 x = 0 处按连续延拓取值 1。

傅里叶分析理论 [1447] 解释了为什么 sinc 滤波器是理想低通滤波器。简而言之，理由如下：理想低通滤波器在频域中是一个盒式滤波器；将它与信号相乘，就能去除超出滤波器宽度的所有频率。把这个盒式滤波器从频域变换到空间域，就会得到 sinc 函数。与此同时，乘法运算变换成卷积运算；本节一直在使用这种运算，只是尚未正式介绍“卷积”这个术语。


![图5.21 sinc滤波器重建](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.21.png)

**图 5.21** 此处使用 sinc 滤波器重建信号。sinc 滤波器是理想低通滤波器。

使用 sinc 滤波器重建信号会得到更平滑的结果，如图 5.21 所示。采样过程向信号中引入高频分量，即突变，而低通滤波器的任务就是将其去除。实际上，sinc 滤波器会消除频率高于采样率一半的所有正弦波。当采样频率为 1.0 时，式（5.22）给出的 sinc 函数是完美重建滤波器，也就是说，被采样信号的最高频率必须小于 ½。更一般地，假设采样频率为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_148dc48c180755.png)，也就是相邻样本之间的间隔为 1/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_148dc48c180755.png)。在这种情况下，完美重建滤波器为 sinc(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_148dc48c180755.png)x)，它会消除高于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_148dc48c180755.png)/2 的所有频率。这在对信号重新采样时很有用，下一部分会讨论。然而，sinc 的滤波器宽度是无限的，而且某些区间内取负值，因此在实践中很少能直接使用。

在低质量的盒式、帐篷滤波器与不切实际的 sinc 滤波器之间，存在有用的折中。大多数广泛使用的滤波函数 [1214, 1289, 1413, 1793] 都处于这两个极端之间。这些滤波函数在某种程度上近似 sinc 函数，但限制了其影响的像素数量。最接近 sinc 函数的滤波器，在其定义域的一部分上会取负值。对于不希望出现负滤波值、或无法实际处理负值的应用，通常使用没有负瓣的滤波器；这些滤波器常被统称为高斯滤波器，因为它们或由高斯曲线推导而来，或与其形状相似 [1402]。12.1 节将更详细地讨论滤波函数及其使用。

使用任何一种滤波器之后，都能得到连续信号。不过，在计算机图形学中，我们不能直接显示连续信号，却可以利用它对信号重新采样，得到不同大小的结果，也就是放大或缩小信号。下面讨论这个问题。

#### 重采样

重采样用于放大或缩小采样信号。假设原始采样点位于整数坐标 0、1、2、……处，也就是采样间隔为一个单位。进一步假设，重采样后，希望新采样点均匀分布，相邻间隔为 a。a > 1 时发生缩小，即下采样；a < 1 时发生放大，即上采样。

两种情况中，放大较简单，所以先从它开始。假设按照上一部分介绍的方法重建采样信号。直观而言，既然现在信号已经被完美重建并且连续，只需按所需间隔对重建信号重新采样即可。图 5.22 展示了这一过程。


![图5.22 放大与重采样](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.22.png)

**图 5.22** 左侧为采样信号及其重建信号。右侧以两倍采样率对重建信号重新采样，也就是进行了放大。

然而，缩小时这种技术不适用。相对于新的采样率，原始信号的频率过高，无法避免走样。已有研究表明，此时应使用 sinc(x/a) 滤波器，将采样信号转为连续信号 [1447, 1661]，然后才能按所需间隔重新采样，见图 5.23。换句话说，此处使用 sinc(x/a) 作为滤波器，相当于增大低通滤波器的宽度，从而去除更多高频内容。如图所示，各个 sinc 滤波器的宽度加倍，以便将重采样率降为原采样率的一半。对应到数字图像，这类似于先将图像模糊以移除高频，再以较低分辨率对图像重新采样。


![图5.23 缩小前加宽滤波器](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.23.png)

**图 5.23** 左侧为采样信号及其重建信号。右侧将滤波器宽度加倍，以便把样本间隔加倍，也就是进行了缩小。

以采样和滤波理论作为框架，下面讨论实时渲染中用于减轻走样的各种算法。

### 5.4.2 基于屏幕的抗锯齿

如果没有妥善采样和滤波，三角形边缘就会产生明显伪影。阴影边界、镜面高光，以及其他颜色快速变化的现象也会引发类似问题。本节讨论的算法有助于改善这些情况下的渲染质量。它们的共同点是都基于屏幕，也就是说，只对流水线输出的样本进行操作。不存在一种最佳抗锯齿技术，因为各种技术在质量、捕捉清晰细节或其他现象的能力、运动时的外观、内存成本、GPU 要求以及速度等方面，各有优势。

在图 5.14 的黑色三角形例子中，一个问题就是采样率低。每个像素网格单元仅在中心取得一个样本，因此，对该单元所掌握的信息，最多只是其中心是否被三角形覆盖。通过在每个屏幕网格单元内使用更多样本，并以某种方式混合它们，就能计算出更好的像素颜色。图 5.24 对此作了说明。


![图5.24 多样本估计覆盖率](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.24.png)

**图 5.24** 左侧在像素中心使用一个样本渲染红色三角形。由于三角形没有覆盖该样本，像素会是白色，尽管像素的相当一部分已被红色三角形覆盖。右侧每个像素使用四个样本，其中两个被红色三角形覆盖，因此得到粉红色像素。

基于屏幕的抗锯齿方案，其一般策略是为屏幕选择一种采样模式，再对样本加权求和，生成像素颜色 **p**：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_e3b9e34c425df4.png)


其中 n 是为一个像素取得的样本数。函数 **c**(i, x, y) 是样本颜色；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_3e929edc96716e.png) 是一个位于 [0, 1] 范围内的权重，表示该样本对最终像素颜色的贡献。采样位置取决于它在序列 1、……、n 中是第几个样本；函数还可以选择使用像素位置 (x, y) 的整数部分。换句话说，每个样本在屏幕网格上的采样位置不同，采样模式还可以因像素而异。在实时渲染系统中，样本通常是点样本；实际上，大多数其他渲染系统也是如此。因此，可以把函数 **c** 看成两个函数。首先，函数 **f**(i, n) 获取屏幕上需要采样的浮点位置 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_cedb8fedde4bd2.png)。然后，对屏幕上的这个位置采样，也就是取得该精确位置的颜色。选择采样方案并配置渲染流水线之后，便可在特定亚像素位置计算样本；这通常依据每帧或每个应用的设置进行。

抗锯齿中的另一个变量是每个样本的权重 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_3e929edc96716e.png)。这些权重之和为 1。实时渲染系统中的大多数方法为样本赋予均匀权重，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_04_710bb69d130300.png)。图形硬件的默认模式是在像素中心取一个样本，它是上述抗锯齿方程最简单的情况：只有一项，该项的权重为 1，采样函数 **f** 始终返回待采样像素的中心。

每个像素计算多个完整样本的抗锯齿算法，称为**超采样**（supersampling，或 oversampling）方法。概念上最简单的**全场景抗锯齿**（FSAA），也称**超采样抗锯齿**（SSAA），先以更高分辨率渲染场景，再对相邻样本滤波，生成图像。例如，希望得到一幅 1280 × 1024 像素的图像，可以先在屏幕外渲染 2560 × 2048 图像，再对每个 2 × 2 像素区域求平均，得到所需图像。这样，每个最终像素使用四个样本，并采用盒式滤波器滤波。注意，这对应于图 5.25 中的 2 × 2 网格采样。这种方法成本很高，因为所有子样本都必须完成完整的着色和填充，而且每个样本都要保存一个 z 缓冲深度。FSAA 的主要优点是简单。这种方法还有一些质量较低的版本，只在屏幕一个轴向上采用两倍采样率，因而称为 1 × 2 或 2 × 1 超采样。为了简单起见，通常采用二的幂次分辨率缩放及盒式滤波器。NVIDIA 的动态超级分辨率功能是一种更复杂的超采样形式：以某个更高分辨率渲染场景，再使用一个 13 样本的高斯滤波器生成显示图像 [1848]。

另一种与超采样相关的采样方法，基于**累积缓冲区**的思想 [637, 1115]。它不使用一个很大的离屏缓冲区，而使用与目标图像分辨率相同、但每个颜色通道位数更多的缓冲区。为了对场景实现 2 × 2 采样，需要生成四幅图像，并根据需要在屏幕 x 或 y 方向上将视图移动半个像素。每幅生成的图像都对应网格单元内的不同采样位置。每帧多次重新渲染场景并把结果复制到屏幕的额外成本，使这一算法对实时渲染系统而言代价很高。在性能不重要时，它却很适合生成高质量图像，因为每个像素可以使用任意数量、任意位置的样本 [1679]。累积缓冲区过去是一个独立的硬件部件，OpenGL API 曾直接支持它，但在 3.0 版中将其列为弃用功能。在现代 GPU 上，可以为输出缓冲区使用更高精度的颜色格式，在像素着色器中实现累积缓冲区的概念。

当物体边缘、镜面高光或锐利阴影等现象导致颜色突变时，就需要额外样本。常常可以把阴影做得更柔和、高光做得更平滑，以避免走样。某些类型的物体，例如电线，可以增大尺寸，以保证沿其全长的每个位置至少覆盖一个像素 [1384]。物体边缘的走样仍然是主要采样问题。也可以使用解析方法，在渲染过程中检测物体边缘，并把边缘的影响纳入计算，但这些方法往往比单纯增加样本更昂贵、稳健性更差。不过，保守光栅化、光栅器顺序视图等 GPU 功能打开了新的可能性 [327]。

超采样、累积缓冲等技术会生成具有完整信息的样本，每个样本都单独计算着色结果与深度。总体收益相对有限，而成本很高，因为每个样本都必须执行像素着色器。


![图5.25 像素采样方案比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.25.png)

**图 5.25** 一些像素采样方案的比较，按每个像素的样本数由少到多排列。Quincunx 共享角点样本，并使中心样本占最终像素颜色的一半权重。对于近乎水平的边缘，2 × 2 旋转网格比常规 2 × 2 网格能够捕捉更多灰度级。同样，对于这类线条，8 车模式虽然样本更少，却比 4 × 4 网格能捕捉更多灰度级。

**多重采样抗锯齿**（MSAA）通过对每个像素只计算一次表面着色，并在样本之间共享结果，减轻高昂的计算成本。例如，一个片元在某个像素中可以具有四个 (x, y) 采样位置，每个位置有自己的颜色和 z 深度，但对作用于该像素的每个物体片元，像素着色器只执行一次。如果所有 MSAA 位置样本都被片元覆盖，则在像素中心计算着色样本。如果片元覆盖的位置样本较少，可以移动着色样本的位置，以更好地代表被覆盖的位置。例如，这样能够避免在纹理边缘之外进行着色采样。这种位置调整称为**质心采样**或**质心插值**；启用后，由 GPU 自动完成。质心采样避免了采样点落在三角形外的问题，但可能使导数计算返回错误结果 [530, 1041]。见图 5.26。


![图5.26 MSAA与EQAA的样本存储](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.26.png)

**图 5.26** 中间显示两个物体与同一个像素重叠。红色物体覆盖三个样本，蓝色物体仅覆盖一个。像素着色器的求值位置用绿色表示。由于红色三角形覆盖像素中心，因此在中心计算着色器。蓝色物体的像素着色器在其样本位置求值。对于 MSAA，四个位置都分别存储颜色与深度。右侧显示 EQAA 的 2f4x 模式。现在，四个样本各有一个 ID 值，用于索引存有两组颜色与深度的表。

由于片元只着色一次，MSAA 比纯超采样方案更快。它将计算量集中用于以更高采样率估计片元的像素覆盖情况，并共享计算出的着色结果。进一步解耦采样与覆盖率还能节省更多内存，进而进一步加快抗锯齿：访问的内存越少，渲染越快。NVIDIA 于 2006 年推出**覆盖采样抗锯齿**（CSAA），AMD 随后推出**增强质量抗锯齿**（EQAA）。这些技术仅以更高采样率存储片元覆盖信息。例如，EQAA 的“2f4x”模式存储两组颜色和深度值，由四个采样位置共享。颜色和深度不再针对特定位置存储，而是存入一张表中。于是，四个样本中的每个只需要一位，就能指明其位置对应两个存储值中的哪一个。见图 5.26。覆盖样本决定每个片元对最终像素颜色的贡献。如果所需颜色数量超过存储容量，就会淘汰一个已存储颜色，并将它的样本标为未知。这些样本不再贡献最终颜色 [382, 383]。多数场景中，同一个像素内出现三个或更多着色差异显著、可见且不透明的片元的情况相对较少，因此该方案在实践中表现良好 [1405]。不过，为了获得最高质量，游戏《极限竞速：地平线 2》最终采用了 4× MSAA，尽管 EQAA 在性能上有优势 [1002]。

全部几何体渲染到多样本缓冲区后，会执行一次**解析**（resolve）操作。这一过程将样本颜色平均，确定像素的颜色。需要注意，当多重采样与高动态范围颜色值结合使用时，可能出现问题。此时，为了避免伪影，通常需要在解析之前对这些值进行色调映射 [1375]。这可能很昂贵，因此可以使用更简单的色调映射函数近似或其他方法 [862, 1405]。

默认情况下，MSAA 使用盒式滤波器进行解析。2007 年 ATI 推出**自定义滤波抗锯齿**（CFAA）[1625]，能够使用窄、宽两种帐篷滤波器，略微延伸到其他像素单元中。此后，这种模式被 EQAA 支持所替代。在现代 GPU 上，像素着色器或计算着色器可以访问 MSAA 样本，并使用任何所需的重建滤波器，包括从周围像素的样本中取样的滤波器。更宽的滤波器能够减轻走样，但会损失清晰细节。Pettineo [1402, 1405] 发现，宽度为 2 或 3 个像素的三次 smoothstep 和 B 样条滤波器，整体上效果最好。不过，这也有性能成本：即使用自定义着色器模拟默认的盒式滤波解析，也会耗时更长；更宽的滤波核意味着更高的样本访问成本。

NVIDIA 内置的 TXAA 支持同样在大于单个像素的区域上使用更好的重建滤波器，以获得更好结果。它与更新的 MFAA，即多帧抗锯齿方案，都使用了**时间抗锯齿**（TAA）。这是一大类利用前几帧结果来改善图像的技术。其中一部分技术之所以能够实现，是因为提供了允许程序员逐帧设置 MSAA 采样模式的功能 [1406]。这些技术能够应对旋转车轮之类的走样问题，也能改善边缘渲染质量。

设想“手动”实现一种采样模式：生成一系列图像，每次渲染都在像素内不同的位置取样。这个偏移通过在投影矩阵上附加一个很小的平移来实现 [1938]。生成并平均的图像越多，结果越好。时间抗锯齿算法使用了这种结合多幅偏移图像的概念。先生成一幅图像，可能使用 MSAA 或其他方法，然后混入先前的图像。通常只使用二至四帧 [382, 836, 1405]。较旧图像的权重可以按指数衰减 [862]，但如果观察者和场景都不动，这可能使画面产生闪烁，因此常常只对上一帧和当前帧赋予相同权重。每帧样本位于不同的亚像素位置，对这些样本加权求和，就能比单帧更好地估计边缘覆盖率。因此，将最近两帧平均的系统能够得到更好结果。每帧都不需要增加样本，这正是此类方法如此吸引人的原因。甚至可以借助时间采样，生成低分辨率图像，再将其放大到显示器的分辨率 [1110]。此外，原本需要许多样本才能获得良好结果的照明方法或其他技术，也可以每帧使用较少样本，因为结果会跨多帧混合 [1938]。

虽然这类算法无需额外采样成本便能为静态场景提供抗锯齿，但用于时间抗锯齿时也存在几个问题。如果各帧权重不相等，静态场景中的物体可能出现闪烁。快速运动的物体或快速移动的相机可能造成**重影**，即由于先前帧的贡献，在物体后方留下拖尾。一种解决方法是只对缓慢移动的物体执行这种抗锯齿 [1110]。另一个重要方法是使用**重投影**（12.2 节），更好地对应前后两帧中的物体。在这类方案中，物体生成运动向量，并将其存储在独立的“速度缓冲区”中（12.5 节）。这些向量用于将上一帧与当前帧关联起来：从当前像素位置减去该向量，就能找到上一帧中对应这一物体表面位置的颜色像素。不太可能属于当前帧该表面的样本会被舍弃 [1912]。

由于时间抗锯齿不需要额外样本，额外工作量也相对较小，近年来这类算法引起了广泛兴趣，并得到更广泛采用。部分关注也源于延迟着色技术（20.1 节）与 MSAA 及其他多重采样支持不兼容 [1486]。各种方法有所不同；根据应用内容和目标，人们开发出一系列避免伪影、改善质量的技术 [836, 1154, 1405, 1533, 1938]。例如，Wihlidal 的演讲 [1885] 展示了如何将 EQAA、时间抗锯齿，以及作用于棋盘格采样模式的各种滤波技术结合，在降低像素着色器调用次数的同时保持质量。Iglesias-Guitian 等人 [796] 总结了以往工作，并提出利用像素历史与预测来最大限度减少滤波伪影的方案。Patney 等人 [1357] 将 Karis 和 Lottes 在 Unreal Engine 4 实现中的 TAA 工作 [862] 扩展到虚拟现实应用，加入可变大小采样及眼球运动补偿（21.3.2 节）。

#### 采样模式

有效的采样模式，是减轻时间走样及其他走样的关键因素。Naiman [1257] 表明，人类最容易受到近乎水平、近乎垂直边缘上的走样干扰，其次是接近 45° 的边缘。**旋转网格超采样**（RGSS）使用旋转过的正方形模式，在像素内部提供更高的垂直与水平分辨能力。图 5.25 展示了这种模式的一个例子。

RGSS 模式是**拉丁超立方体采样**或 **N 车采样**（N-rooks sampling）的一种形式：把 n 个样本放入 n × n 网格中，每行、每列各有一个样本 [1626]。在 RGSS 中，四个样本分别位于 4 × 4 亚像素网格的不同列、不同行。与规则 2 × 2 采样模式相比，这类模式特别擅长捕捉近乎水平、近乎垂直的边缘；在规则模式中，这类边缘很可能一次覆盖偶数个样本，因此有效覆盖级数较少。

N 车采样是创建良好采样模式的起点，但还不充分。例如，所有样本都可以沿亚像素网格的一条对角线排列；对于几乎平行于该对角线的边缘，这样的结果就很差。见图 5.27。


![图5.27 N车采样的分布差异](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.27.png)

**图 5.27** N 车采样。左侧是合法的 N 车模式，但在捕捉沿样本连线方向的对角三角形边缘时表现不佳，因为当三角形移动时，所有采样位置会一起位于三角形内部或一起位于外部。右侧的模式能够更有效地捕捉这条边缘及其他方向的边缘。

为了更好地采样，我们希望避免让两个样本彼此过近，也希望样本均匀分布，均匀铺开在整个区域中。为形成这类模式，可以把拉丁超立方体采样等**分层采样**技术，与抖动、Halton 序列、泊松圆盘采样等方法结合 [1413, 1758]。

实际中，GPU 厂商通常把这类采样模式固定实现于多重采样抗锯齿硬件中。图 5.28 展示了一些实际使用的 MSAA 模式。对于时间抗锯齿，程序员可以任意选择覆盖采样模式，因为采样位置能够逐帧变化。例如，Karis [862] 发现，一个基本的 Halton 序列就比 GPU 提供的任何 MSAA 模式更好。Halton 序列生成的空间样本看似随机，却具有**低差异性**，也就是说，它们在空间中分布良好，不会聚集成团 [1413, 1938]。


![图5.28 GPU中的MSAA采样模式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.28.png)

**图 5.28** AMD 与 NVIDIA 图形加速器的 MSAA 采样模式。绿色方块是着色样本的位置，红色方块是计算并保存的位置样本。从左至右分别是 2×、4×、6×（AMD）和 8×（NVIDIA）采样。（由 D3D FSAA Viewer 生成。）

> 译注：原图注使用“方块”（square）一词，但本版图中的绿色和红色标记实际绘为圆点，此处保留原图注措辞。

亚像素网格模式虽然能更好地近似每个三角形对网格单元的覆盖情况，但并不理想。场景中的物体在屏幕上可以任意小，这意味着任何采样率都无法完美捕捉它们。如果这些微小物体或特征形成规律图案，按固定间隔采样就可能产生摩尔纹及其他干涉图案。超采样所用的网格模式尤其容易发生走样。

一种解决方法是使用**随机采样**（stochastic sampling），让模式更加随机化。图 5.28 那样的模式当然也符合这一思路。设想远处一把细齿梳子，每个像素覆盖几个梳齿。规则模式会在采样模式与梳齿频率时而同相、时而异相的过程中产生严重伪影。采用较不规则的采样模式可以打散这些图案。随机化倾向于把重复性的走样效应替换成噪声，而人类视觉系统对噪声宽容得多 [1413]。结构性较弱的模式会有帮助，但如果每个像素都重复相同模式，仍然可能发生走样。一种办法是在每个像素使用不同采样模式，或随时间改变每个采样位置。在过去数十年中，硬件偶尔曾支持**交错采样**：一组像素中的每个像素使用不同采样模式。例如，ATI 的 SMOOTHVISION 允许每个像素最多 16 个样本，以及最多 16 种用户自定义采样模式；这些模式可混合成重复图案，例如一个 4 × 4 像素块。Molnar [1234] 以及 Keller 和 Heidrich [880] 发现，采用交错随机采样，可以最大限度减少每个像素都使用同一模式时产生的走样伪影。

> 译注：原书此段在“Interleaved sampling”之后混入了“indexsampling!interleaved”索引标记，属于排印残留，不是正文内容。

还有几种 GPU 支持的算法值得一提。NVIDIA 较早的 Quincunx 方法 [365] 是一种允许样本影响多个像素的实时抗锯齿方案。“Quincunx”意指五个物体的排列方式：四个构成正方形，第五个位于中心，就像六面骰子上五点那一面的图案。Quincunx 多重采样抗锯齿采用这种模式，把四个外侧样本放在像素角点。见图 5.25。每个角点样本值分配给其四个相邻像素。它不像大多数其他实时方案那样给各个样本相同权重，而是让中心样本权重为 ½，每个角点样本权重为 ⅛。由于这种共享，平均每个像素只需要两个样本，结果却明显优于双样本 FSAA 方法 [1678]。这种模式近似二维帐篷滤波器；如前一部分所述，它优于盒式滤波器。

Quincunx 采样也可以在每个像素只取一个样本的条件下，用于时间抗锯齿 [836, 1677]。每帧相对前一帧沿两个轴各偏移半个像素，偏移方向在帧间交替。上一帧提供像素角点样本，利用双线性插值快速计算其对每个像素的贡献，再将结果与当前帧平均。各帧权重相同，意味着静态视图不会出现闪烁伪影。运动物体的对齐问题仍然存在，但该方案本身编码简单，在每帧每像素只使用一个样本的同时，能显著改善外观。

在单帧中使用时，Quincunx 借助像素边界上的样本共享，将成本降低到每像素仅两个样本。而 RGSS 模式更擅长捕捉近水平、近垂直边缘上的更多渐变级。最初为移动图形开发的 FLIPQUAD 模式，将这两个优点结合起来 [22]。它的优势是每个像素只需两个样本，而质量接近每像素四个样本的 RGSS。图 5.29 展示了这一采样模式。Hasselgren 等人 [677] 还研究了其他利用样本共享的低成本采样模式。


![图5.29 FLIPQUAD样本共享](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.29.png)

**图 5.29** 左侧显示 RGSS 采样模式，其成本为每个像素四个样本。把这些位置移到像素边缘后，就能跨边缘共享样本。不过，为了实现共享，必须让交替相邻的像素采用镜像反射后的采样模式，如右图所示。得到的采样模式称为 FLIPQUAD，成本为每像素两个样本。

与 Quincunx 一样，双样本 FLIPQUAD 模式也可以结合时间抗锯齿，将两个样本分布在两帧中。Drobot [382, 383, 1154] 在其**混合重建抗锯齿**（HRAA）工作中，研究了哪种双样本模式最佳的问题。他探索不同的时间抗锯齿采样模式，发现 FLIPQUAD 是所测试五种模式中最好的。棋盘格模式也曾用于时间抗锯齿。El Mansouri [415] 讨论了使用双样本 MSAA 创建棋盘格渲染，在解决走样问题的同时降低着色器成本。Jimenez [836] 使用 SMAA、时间抗锯齿及多种其他技术，给出了一种能根据渲染引擎负载调整抗锯齿质量的方案。Carpentier 和 Ishiyama [231] 在边缘上采样，将采样网格旋转 45°。他们把这种时间抗锯齿方案与后面讨论的 FXAA 结合，高效地面向更高分辨率显示器进行渲染。

#### 形态学方法

走样往往源于边缘，例如几何体、锐利阴影或明亮高光形成的边缘。知道走样具有一定结构，就可以利用这一点取得更好的抗锯齿结果。2009 年，Reshetov [1483] 提出一种遵循这一思路的算法，称为**形态学抗锯齿**（MLAA）。“形态学”意指“与结构或形状有关”。此前该领域已有相关工作 [830]，最早可以追溯到 Bloomenthal 于 1983 年的研究 [170]。Reshetov 的论文重新激发了对多重采样替代方法的研究兴趣，着重强调搜索并重建边缘 [1486]。

这种抗锯齿以**后处理**形式执行：先按通常方式完成渲染，再把结果交给一个生成抗锯齿结果的处理过程。2009 年以来，人们开发出许多技术。依赖深度、法线等额外缓冲区的方法能够提供更好的结果，例如**亚像素重建抗锯齿**（SRAA）[43, 829]，但这些方法只能用于几何边缘的抗锯齿。**几何缓冲区抗锯齿**（GBAA）和**边缘距离抗锯齿**（DEAA）等解析方法，让渲染器额外计算三角形边缘位置的信息，例如边缘距离像素中心有多远 [829]。

最通用的方案只需颜色缓冲区，因此也能改善阴影、高光以及之前各种后处理技术产生的边缘，例如轮廓边缘渲染（15.2.3 节）。例如，**方向局部化抗锯齿**（DLAA）[52, 829] 基于以下观察：近乎垂直的边缘应沿水平方向模糊；同样，近乎水平的边缘应与其相邻像素沿垂直方向模糊。

更复杂的边缘检测方法试图找到可能含有任意角度边缘的像素，并确定其覆盖率。它们检查潜在边缘周围的邻域，尽可能重建原始边缘的位置。随后，利用该边缘对像素的影响，将相邻像素的颜色混合进来。图 5.30 展示了这一过程的概念。


![图5.30 形态学抗锯齿原理](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_4_5.30.png)

**图 5.30** 形态学抗锯齿。左侧是有走样的图像，目标是确定形成它的边缘可能具有的方向。中间，算法检查相邻像素，判断存在边缘的可能性；根据已有样本，图中显示两个可能的边缘位置。右侧使用最可能的边缘估计，按估算覆盖率将相邻颜色混入中心像素。对图像中的每个像素重复这一过程。

Iourcha 等人 [798] 通过检查像素内的 MSAA 样本改善边缘搜索，从而计算出更好的结果。注意，边缘预测与混合能够获得比基于样本的算法更高精度的结果。例如，每个像素使用四个样本的技术，对于一个物体边缘只能给出五个混合级：零个、一个、两个、三个或四个样本被覆盖。估计的边缘位置可以有更多取值，因此能够提供更好的结果。

基于图像的算法可能以多种方式出错。首先，如果两个物体的颜色差异低于算法阈值，就可能检测不到边缘。三个或更多不同表面重叠的像素，很难解释。含有高对比度或高频元素、颜色在像素间快速变化的表面，会导致算法漏掉边缘。尤其是，对文字应用形态学抗锯齿时，质量通常会下降。物体拐角也是挑战，有些算法会让它们看起来变圆。假设边缘是直线，也可能对曲线产生不利影响。单个像素的变化就可能导致边缘重建方式大幅改变，从而产生明显的帧间伪影。一种减轻这一问题的方法，是利用 MSAA 覆盖掩码改善边缘判断 [1484]。

形态学抗锯齿方案只能使用提供给它的信息。例如，一个宽度小于像素的物体，如电线或绳索，在没有恰好覆盖像素中心位置的地方，屏幕上就会出现缺口。在这些情况下，增加样本能够改善质量，仅靠基于图像的抗锯齿则无能为力。此外，执行时间可能随所看内容变化。例如，对一片草地视图进行抗锯齿的耗时，可能是天空视图的三倍 [231]。

尽管如此，基于图像的方法以适中的内存与处理成本提供抗锯齿支持，因此被许多应用采用。仅使用颜色的版本还与渲染流水线解耦，便于修改或禁用，甚至可以作为 GPU 驱动选项提供。最流行的两种算法是**快速近似抗锯齿**（FXAA）[1079, 1080, 1084] 和**亚像素形态学抗锯齿**（SMAA）[828, 830, 834]；部分原因是两者都为多种机器提供了可靠且免费的源代码实现。两种算法都仅需要颜色输入，而 SMAA 还具有能够访问 MSAA 样本的优势。各自都有多种设置，可在速度与质量之间权衡。成本通常在每帧 1 至 2 毫秒范围内，主要因为这就是电子游戏愿意付出的预算。最后，两种算法都可以利用时间抗锯齿 [1812]。Jimenez [836] 给出了一个比 FXAA 更快的改进版 SMAA 实现，并介绍了一种时间抗锯齿方案。作为结尾，我们推荐读者阅读 Reshetov 和 Jimenez [1486] 对形态学技术及其在电子游戏中应用的广泛综述。


## 5.5 透明度、Alpha 与合成

来源：*Real-Time Rendering, Fourth Edition*，书页 148—160（PDF 物理页 169—181）。本节从 5.5 标题开始，到 5.6 标题之前结束，包含 5.5.1—5.5.3。技术陈述按原书出版时的内容翻译。

半透明物体允许光穿过的方式有很多。对于渲染算法，这些方式大致可以分为基于光照的效果与基于观察的效果。基于光照的效果，是指物体使光衰减或改变方向，从而使场景中其他物体受到的照明及其渲染结果发生变化。基于观察的效果，则是指对半透明物体本身进行渲染。

本节讨论最简单的一种基于观察的透明效果：半透明物体使其后方物体的颜色衰减。更复杂的基于观察和光照的效果，例如磨砂玻璃、光的弯折（折射）、透明物体厚度造成的光衰减，以及反射率和透射随观察角度发生的变化，将在后续章节讨论。

一种产生透明错觉的方法称为**纱门透明**（screen-door transparency）[1244]。其思路是使用与像素对齐的棋盘格填充图案来渲染透明三角形。也就是说，三角形内每隔一个像素才渲染一个，使后方物体仍有一部分可见。通常屏幕像素排列得足够紧密，因而看不出棋盘格图案本身。这种方法的一个主要缺点是：在屏幕的某个区域，往往只能令人信服地渲染一个透明物体。例如，在蓝色物体上方渲染一个透明红色物体和一个透明绿色物体时，棋盘格图案只能显示这三种颜色中的两种。此外，50% 的棋盘格覆盖率也构成限制。可以用其他更大的像素掩码表示其他百分比，但这样往往会产生可察觉的图案 [1245]。

不过，这项技术的一个优点就是简单。透明物体可以在任何时候、以任何顺序渲染，而且不需要特殊硬件。只要让所有物体在其覆盖的像素上都成为不透明物体，透明问题就消失了。同样的思想也用于镂空纹理边缘的抗锯齿，不过是在亚像素层面，通过一种称为 **alpha-to-coverage**（Alpha 转覆盖率）的功能来实现（第 6.6 节）。

Enderton 等人 [423] 提出的**随机透明**（stochastic transparency）把亚像素纱门掩码与随机采样结合起来。它使用随机点画图案表示片元的 Alpha 覆盖率，从而生成虽然带有噪声、但还算合理的图像。见图 5.31。为了使结果看起来合理，每个像素需要大量采样点，也需要相当大的内存来保存全部亚像素样本。这种方法吸引人的地方在于不需要混合：抗锯齿、透明度以及任何其他产生部分覆盖像素的现象，都由同一种机制处理。


![图 5.31 随机透明](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.31.png)

**图 5.31** 随机透明。放大区域展示了产生的噪声。（图像取自 NVIDIA SDK 11 [1301] 示例，由 NVIDIA Corporation 提供。）

大多数透明算法将透明物体的颜色与其后方物体的颜色混合。为此，需要引入 **Alpha 混合**的概念 [199, 387, 1429]。当物体被渲染到屏幕上时，每个像素都对应一个 RGB 颜色和一个 z 缓冲深度。对于物体覆盖的每个像素，还可以定义另一个称为  **Alpha（α）** 的分量。Alpha 描述的是：对于给定像素，一个物体片元的不透明程度及覆盖程度。Alpha 为 1.0 表示物体不透明，并且完全覆盖像素所关注的区域；0.0 则表示像素完全没有被遮挡，即片元完全透明。

根据具体情形，像素的 Alpha 可以表示不透明度、覆盖率，或者同时表示两者。例如，肥皂泡边缘可能覆盖一个像素的四分之三，即 0.75；同时又几乎透明，让十分之九的光透过并到达眼睛，因此不透明度为十分之一，即 0.1。此时其 Alpha 就是 0.75 × 0.1 = 0.075。不过，如果使用 MSAA 或类似的抗锯齿方案，覆盖率就已经由采样点本身计入了。四分之三的采样点会受到肥皂泡影响，而在其中每个采样点上，应使用不透明度值 0.1 作为 Alpha。

### 5.5.1 混合顺序

要让一个物体看起来透明，就以小于 1.0 的 Alpha 将其渲染到已有场景上方。物体覆盖的每个像素都会从像素着色器接收一个 RGBα（也称为 RGBA）结果。通常使用 **over 运算符**将这个片元的值与原有像素颜色混合：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_a086ef398cc6c1.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_2ce4c3f8e5a5ae.png) 是透明物体的颜色，称为**源**（source）；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9d279bf9dd1d56.png) 是该物体的 Alpha；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_10e90fea1452e8.png) 是混合前的像素颜色，称为**目标**（destination）；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9c1542bf5c49c0.png) 是把透明物体放在已有场景上方之后得到的颜色。当渲染流水线传入 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_2ce4c3f8e5a5ae.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9d279bf9dd1d56.png) 时，像素原有的颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_10e90fea1452e8.png) 会被结果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9c1542bf5c49c0.png) 替换。如果传入的 RGBα 实际上是不透明的（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9d279bf9dd1d56.png) = 1.0），这个公式就简化为用物体颜色完全替换像素颜色。

**示例：混合。** 将一个红色半透明物体渲染到蓝色背景上。假设在某个像素处，物体的 RGB 颜色为 (0.9, 0.2, 0.1)，背景为 (0.1, 0.1, 0.9)，物体不透明度设为 0.6。两种颜色的混合为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_7dc12c42c9ebf4.png)


得到颜色 (0.58, 0.16, 0.42)。□

over 运算符使正在渲染的物体呈现半透明外观。这样实现的透明效果是有效的，因为只要能透过某个东西看见其后方的物体，我们就会把它感知为透明的 [754]。使用 over 模拟的是现实中薄纱织物的效果。织物后方物体的视图有一部分被遮住，因为织物的纱线是不透明的。在实际情况中，疏松织物的 Alpha 覆盖率会随角度变化 [386]。这里要强调的是：Alpha 模拟的是材料覆盖像素的程度。


![图 5.32 薄纱与塑料滤光片](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.32.png)

**图 5.32** 一块红色薄纱方巾和一片红色塑料滤光片，产生了不同的透明效果。注意它们的阴影也不相同。（照片由 Morgan McGuire 提供。）

over 运算符在模拟其他透明效果时就不那么令人信服了，最典型的是透过有色玻璃或塑料观察物体。在现实中，把红色滤光片放在一个蓝色物体前方，通常会让蓝色物体显得很暗，因为这个物体反射的光中，只有很少一部分能穿过红色滤光片。见图 5.32。使用 over 进行混合时，结果却是把一部分红色和一部分蓝色相加。更好的做法是将这两种颜色相乘，再加上透明物体本身反射的光。这种物理透射将在第 14.5.1 和 14.5.2 节讨论。

在混合阶段的基本运算符中，over 是通常用来实现透明效果的运算符 [199, 1429]。另一种也有应用的操作是**加法混合**（additive blending），它直接把像素值相加，即


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_be2eb63e4cbfa4.png)


这种混合模式很适合闪电或火花等发光效果：它们不衰减后方像素，只使后方像素变亮 [1813]。但是，这种模式用于透明效果时看起来并不正确，因为不透明表面看起来没有经过滤光 [1192]。对于烟雾或火焰等由若干层半透明表面构成的现象，加法混合会使其颜色趋于饱和 [1273]。

要正确渲染透明物体，就需要在不透明物体之后绘制它们。做法是先关闭混合，渲染全部不透明物体，然后开启 over，渲染透明物体。理论上也可以始终开启 over，因为不透明物体的 Alpha 为 1.0 时，会输出源颜色并遮住目标颜色，但这样开销更高，却没有实际收益。

z 缓冲的一个限制是每个像素只能存储一个物体。如果多个透明物体在同一像素处重叠，单靠 z 缓冲无法保存全部可见物体的影响并在之后进行解析。使用 over 时，在任意给定像素处，透明表面通常需要按从后到前的顺序渲染。否则可能产生错误的感知线索。一种实现这种顺序的方法是对各个物体排序，例如按照其质心沿观察方向的距离排序。这种粗略排序可以取得相当不错的效果，但在各种情况下仍有不少问题。首先，这个顺序只是近似值，因此被归为更远的物体实际上可能处于被认为更近的物体前方。对于彼此穿插的物体，除非将各个网格拆分成独立部分，否则不可能仅以网格为单位，在所有观察角度下正确处理。图 5.33 左图给出了一个例子。即使只有单个带凹陷的网格，在某些观察方向上，其投影在屏幕上与自身重叠时，也会出现排序问题。


![图 5.33 透明网格排序与深度剥离](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.33.png)

**图 5.33** 左图使用 z 缓冲，以透明方式渲染模型。按任意顺序渲染网格会造成严重错误。右图使用深度剥离，以增加渲染遍数为代价获得正确的外观。（图像由 NVIDIA Corporation 提供。）

尽管如此，由于简单、速度快，而且不需要额外内存或特殊 GPU 支持，粗略排序仍然是常用的透明处理方法。如果采用这种方法，通常最好在处理透明度时关闭 z 深度写入。也就是说，仍然正常进行 z 缓冲测试，但通过测试的表面不改变已存储的 z 深度；最近的不透明表面的深度保持不变。这样一来，所有透明物体至少都会以某种形式出现，而不会在相机旋转导致排序变化时突然出现或消失。其他技术也能帮助改善外观，例如依次处理每个透明网格时绘制两遍，先渲染背面，再渲染正面 [1192, 1255]。

也可以修改 over 公式，使从前到后的混合得到相同结果。这种混合模式称为 **under 运算符**：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_54d9cbd3eb48dc.png)


注意，under 要求目标保存一个 Alpha 值，而 over 没有这个要求。换句话说，目标是那个较近的透明表面，新表面要混合到它的下方；目标本身具有透明性，因而必须具有 Alpha 值。under 的表达形式类似于 over，只是交换了源和目标。另外请注意，Alpha 的计算公式与顺序无关：源 Alpha 与目标 Alpha 可以互换，最终 Alpha 仍然相同。

Alpha 公式来自把片元 Alpha 看作覆盖率的思路。Porter 和 Duff [1429] 指出，由于我们不知道两个片元各自覆盖区域的形状，因此假设每个片元都按自身 Alpha 的比例覆盖另一个片元。例如，如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9d279bf9dd1d56.png) = 0.7，那么像素就以某种方式划分为两个区域，其中 0.7 被源片元覆盖，0.3 未被覆盖。在没有其他信息的情况下，假如目标片元的覆盖率为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_3b5bf9af75e2d7.png) = 0.6，那么它也会按相应比例被源片元覆盖。这个公式具有图 5.34 所示的几何解释。


![图 5.34 两个片元的覆盖面积](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.34.png)

**图 5.34** 一个像素以及两个片元 s 和 d。通过使两个片元沿不同的轴对齐，每个片元都按比例覆盖另一个片元，也就是说，它们彼此不相关。两个片元覆盖的面积等于 under 的输出 Alpha 值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9d279bf9dd1d56.png) − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_9d279bf9dd1d56.png)![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_3b5bf9af75e2d7.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_3b5bf9af75e2d7.png)。这相当于将两个面积相加，再减去它们重叠的面积。

图内文字：片元 s，面积 = 0.7；片元 d，面积 = 0.6；重叠面积 = 0.7 × 0.6。给定两个面积（Alpha）分别为 0.7 和 0.6 的片元，总覆盖面积 = 0.7 − 0.7 × 0.6 + 0.6 = 0.88。

### 5.5.2 顺序无关透明

使用 under 公式时，先将全部透明物体绘制到单独的颜色缓冲中，再使用 over，将该颜色缓冲叠加到场景的不透明视图上。under 运算符的另一种用途，是实现一种称为**深度剥离**（depth peeling）的**顺序无关透明**（order-independent transparency，OIT）算法 [449, 1115]。顺序无关意味着应用程序不需要执行排序。深度剥离的思路是使用两个 z 缓冲和多遍渲染。首先执行一遍渲染，将所有表面（包括透明表面）的 z 深度写入第一个 z 缓冲。第二遍渲染全部透明物体。如果某物体的 z 深度与第一个 z 缓冲中的值相同，就知道它是最近的透明物体，并将其 RGBα 保存到独立的颜色缓冲中。同时，我们还会“剥去”这一层：在比第一个 z 深度更远的透明物体中，找出最近者（如果存在），保存其 z 深度。这个 z 深度就是第二近的透明物体的距离。后续各遍继续剥离，并使用 under 加入透明层。在执行一定遍数后停止，再将透明图像混合到不透明图像上方。见图 5.35。


![图 5.35 深度剥离的各层](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.35.png)

**图 5.35** 每一遍深度剥离都绘制其中一个透明层。左图为第一遍，显示眼睛直接可见的那一层。中图为第二层，显示每个像素处第二近的透明表面，在此例中就是物体的背面。右图的第三层是第三近的透明表面的集合。最终结果见书页 624 的图 14.33。（图像由 Louis Bavoil 提供。）

这种方案已经发展出若干变体。例如，Thibieroz [1763] 给出一种从后向前执行的算法，其优点是可以立即混合透明值，也就不需要单独的 Alpha 通道。深度剥离的一个问题是如何知道执行多少遍才足以捕获全部透明层。一种硬件解决方案是提供像素绘制计数器，用来告知渲染过程中写入了多少像素；当某一遍没有渲染任何像素时，渲染便已完成。使用 under 的优点在于，最重要的透明层，也就是眼睛最先看见的那些层，会较早被渲染。每个透明表面总会增大它所覆盖像素的 Alpha 值。如果像素的 Alpha 值接近 1.0，说明各层混合后的贡献已使该像素几乎不透明，因此更远物体的影响可以忽略 [394]。当某一遍渲染的像素数小于某个最小值时，就可以提前终止从前到后的剥离；也可以指定固定的遍数。对于从后到前的剥离，这样做就不太合适，因为最近的层（通常也是最重要的层）最后才绘制，提前终止可能使它们丢失。

深度剥离虽然有效，但可能很慢，因为每剥离一层，都需要单独执行一遍全部透明物体的渲染。Bavoil 和 Myers [118] 提出了**双向深度剥离**（dual depth peeling）：每一遍剥去剩余层中最近和最远的两层，从而将渲染遍数减半。Liu 等人 [1056] 探索了一种桶排序方法，可以在一遍中捕获多达 32 层。这类方法的一个缺点是，需要相当大的内存来维持全部层的排序。通过 MSAA 或类似方法进行抗锯齿，会使开销急剧增长。

以交互速率正确混合透明物体的问题，并不是缺少算法，而是如何将这些算法高效地映射到 GPU 上。1984 年，Carpenter 提出了另一种多重采样形式：**A 缓冲**（A-buffer）[230]。在 A 缓冲中，每个被渲染的三角形都会为其完全或部分覆盖的每个屏幕网格单元创建一个**覆盖掩码**。每个像素存储一个包含全部相关片元的列表。与 z 缓冲类似，不透明片元可以剔除其后方的片元。对于透明表面，则保存所有片元。在全部列表建立之后，遍历片元并解析各个采样点，生成最终结果。

DirectX 11 提供的新功能使在 GPU 上创建片元链表的设想成为可能 [611, 1765]。所用功能包括第 3.8 节介绍的无序访问视图（UAV）和原子操作。能够访问覆盖掩码，并在每个采样点执行像素着色器，使 MSAA 抗锯齿成为可能。该算法对每个透明表面进行光栅化，并将生成的片元插入一个长数组。除了颜色和深度，还生成独立的指针结构，将每个片元链接到此前为该像素存储的片元。然后再执行单独的一遍，渲染一个覆盖整个屏幕的四边形，使每个像素都执行像素着色器。这个着色器沿链接取出各个像素的全部透明片元。每取出一个片元，就将其与之前的片元一起进行排序。随后按从后到前的顺序混合这个有序列表，得到最终像素颜色。由于混合由像素着色器执行，如果需要，可以为每个像素指定不同的混合模式。GPU 和 API 的持续发展通过降低原子操作的使用成本，提高了性能 [914]。

A 缓冲的优点是只为各个像素分配实际需要的片元，GPU 上的链表实现也是如此。从某种意义上说，这也可能是缺点，因为在开始渲染一帧之前，不知道所需的存储量。包含毛发、烟雾或其他可能出现大量透明表面重叠的物体的场景，会产生数量巨大的片元。Andersson [46] 指出，在复杂游戏场景中，枝叶等物体的透明网格最多可能有 50 个相互重叠，半透明粒子则最多可能有 200 个相互重叠。


![图 5.36 多层 Alpha 混合](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.36.png)

**图 5.36** 左上采用传统的从后到前 Alpha 混合，排序错误导致渲染错误。右上使用 A 缓冲，得到完美但无法达到交互速度的结果。左下是多层 Alpha 混合的渲染结果。右下显示 A 缓冲图像与多层图像之间的差异，为便于观察放大了 4 倍 [1532]。（图像由 Intel Corporation 的 Marco Salvi 和 Karthik Vaidyanathan 提供。）

GPU 通常预先分配缓冲和数组等内存资源，基于链表的方法也不例外。使用者需要决定分配多少内存才足够，而内存耗尽会导致明显的伪影。Salvi 和 Vaidyanathan [1532] 提出了一种解决此问题的方法，称为**多层 Alpha 混合**（multi-layer alpha blending），它使用 Intel 引入的 GPU 功能——**像素同步**（pixel synchronization）。见图 5.36。该功能以低于原子操作的开销提供可编程混合。他们的方法重新组织存储和混合，使内存耗尽时能够平缓地降低质量。粗略排序可以改善这种方案的表现。DirectX 11.3 引入了光栅化顺序视图（第 3.8 节），这是一种缓冲类型，使这种透明处理方法可以在任何支持该功能的 GPU 上实现 [327, 328]。移动设备上也有一种称为**瓦片局部存储**（tile local storage）的类似技术，可以用来实现多层 Alpha 混合 [153]。不过，这类机制本身有性能成本，因此这种算法可能开销较高 [1931]。

这种方法建立在 Bavoil 等人 [115] 提出的 **k 缓冲**思想之上：保存最前面的几个可见层，并尽可能排序；更深层则尽可能丢弃或合并。Maule 等人 [1142] 使用 k 缓冲，并通过**加权平均**计入这些更远的深层。**加权求和** [1202] 与**加权平均** [118] 透明技术都是顺序无关的，只需一遍渲染，而且几乎能在所有 GPU 上运行。问题在于，它们没有考虑物体的顺序。例如，使用 Alpha 表示覆盖率时，薄纱红围巾位于薄纱蓝围巾上方会得到紫色，而正确的外观应该是一条红围巾，透出少量蓝色。虽然对接近不透明的物体效果较差，但这类算法对可视化很有用，也适合高度透明的表面和粒子。见图 5.37。


![图 5.37 不透明度与顺序的重要性](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.37.png)

**图 5.37** 随着不透明度增大，物体顺序变得更加重要。（图像据 Dunn [394] 绘制。）图中不透明度从左到右依次为 10%、40%、70%、100%。

加权求和透明的公式为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_5bb242740a232f.png)


其中，n 是透明表面的数量，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_c884fba8655644.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_af38c6e2ba07c4.png) 表示这一组透明值，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_10e90fea1452e8.png) 是场景中不透明部分的颜色。渲染透明表面时，分别累积并存储这两个和；透明渲染遍结束后，在每个像素处计算这个公式。这种方法的问题是：第一项求和会饱和，也就是产生大于 (1.0, 1.0, 1.0) 的颜色值；另外，由于 Alpha 之和可能超过 1.0，背景颜色也可能产生负向贡献。

通常更倾向于使用加权平均公式，因为它避免了上述问题：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_51e1fc6cd93e4f.png)


第一行表示透明渲染过程中生成的两个独立缓冲中的结果。对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_987620811bbba3.png) 有贡献的每个表面，都按自身 Alpha 加权其影响；接近不透明的表面贡献更多自身颜色，接近透明的表面影响则很小。将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_987620811bbba3.png) 除以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_1471a0ee7ceb40.png)，就得到加权平均的透明颜色。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_364aaf59aef254.png) 是全部 Alpha 值的平均值。u 是对 n 个透明表面应用 n 次这个平均 Alpha 后，目标（不透明场景）的估计可见程度。最后一行实际上就是 over 运算符，其中 (1 − u) 表示源 Alpha。

加权平均的一个限制是：当各个 Alpha 相同时，无论顺序如何，它都会把所有颜色等量混合。McGuire 和 Bavoil [1176, 1180] 引入了**加权混合顺序无关透明**（weighted blended order-independent transparency），以获得更令人信服的结果。在他们的表达式中，到表面的距离也影响权重，较近的表面具有更大的影响。此外，不再对 Alpha 求平均，而是将各项 (1 − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_af38c6e2ba07c4.png)) 相乘，再用 1 减去所得乘积来计算 u，从而得到这组表面的真实 Alpha 覆盖率。这种方法可以产生视觉上更令人信服的结果，如图 5.38 所示。

> **译注（原文符号疑点）：** 原书此句确实将“各项相乘再用 1 减去”的结果称为 u，并称其为总 Alpha 覆盖率；但式（5.28）及上段把 u 定义为背景的剩余可见程度。沿用式（5.28）的定义时，剩余可见程度应为各项 (1 − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_af38c6e2ba07c4.png)) 的乘积，而总 Alpha 覆盖率应为 1 减去该乘积。正文忠实保留原文表述，此处指出两者的区别。


![图 5.38 加权混合顺序无关透明](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_5_5.38.png)

**图 5.38** 从两个不同的相机位置观察同一个引擎模型，两幅图都采用加权混合顺序无关透明渲染。按距离加权有助于分辨哪些表面更接近观察者 [1185]。（图像由 Morgan McGuire 提供。）

一个缺点是：在大型环境中，彼此相近的物体可能具有几乎相同的距离权重，使结果与加权平均相差不大。此外，当相机到透明物体的距离改变时，深度权重的实际影响也可能变化，不过这种变化是渐进的。

McGuire 和 Mara [1181, 1185] 扩展了这一方法，加入一种看起来合理的透射颜色效果。如前所述，本节讨论的全部透明算法都是将各种颜色混合，而不是进行滤光，它们模拟的是像素覆盖率。为了产生滤色效果，像素着色器读取不透明场景，然后每个透明表面都将该场景中它所覆盖的像素乘以自身颜色，并把结果保存到第三个缓冲中。此时该缓冲中的不透明物体已经被透明物体染色，随后在解析透明缓冲时，用它替代原来的不透明场景。这种方法之所以有效，是因为有色透射与由覆盖率产生的透明效果不同，它与顺序无关。

还有一些算法使用了这里介绍的多种技术的组成部分。例如，Wyman [1931] 根据内存需求、插入和合并方法、使用 Alpha 还是几何覆盖率，以及如何处理丢弃片元，对已有工作进行了分类。他通过寻找以往研究的空白，提出两种新方法。他的**随机分层 Alpha 混合**方法使用 k 缓冲、加权平均和随机透明。另一种算法是 Salvi 和 Vaidyanathan 方法的变体，用覆盖掩码代替 Alpha。

透明内容的类型、渲染方法和 GPU 能力多种多样，因此渲染透明物体不存在完美的解决方案。感兴趣的读者可以阅读 Wyman 的论文 [1931]，以及 Maule 等人对交互式透明算法所作的更详细综述 [1141]。McGuire 的报告 [1182] 从更广的角度介绍这一领域，还涉及体积光照、有色透射和折射等其他相关现象，本书后文会更深入地讨论这些内容。

### 5.5.3 预乘 Alpha 与合成

over 运算符也用于将照片或物体的合成渲染图混合到一起。这一过程称为**合成**（compositing）[199, 1662]。在这些情况下，每个像素的 Alpha 值与物体的 RGB 颜色值一同存储。Alpha 通道构成的图像有时称为**遮罩**（matte），它显示物体的轮廓形状。书页 203 的图 6.27 给出了一个例子。然后便可以使用这张 RGBα 图像，将其与其他类似元素或背景混合。

使用合成 RGBα 数据的一种方式是采用**预乘 Alpha**（premultiplied alpha，也称为**关联 Alpha**，associated alpha）。也就是说，在使用 RGB 值之前，先将其乘以 Alpha 值。这使合成的 over 公式更高效：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_3391e7ddc49c84.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_e1fb0820272ea8.png) 是预乘后的源通道，替代式（5.25）中的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_05_7d3dc70b586357.png)。预乘 Alpha 还使得无需改变混合状态就能使用 over 和加法混合，因为此时源颜色是在混合过程中直接相加的 [394]。注意，对于预乘的 RGBα 值，RGB 分量通常不大于 Alpha 值，不过也可以让它们大于 Alpha，以产生特别明亮的半透明值。

> **译注（原文交叉引用疑点）：** 原书此处写的是“式（5.25）”。式（5.25）为加法混合，而式（5.29）的完整结构对应于将式（5.24）的 over 公式中的源颜色项预乘。这里保留原书引用，不作无标记的更改。

合成图像的渲染与预乘 Alpha 自然契合。在黑色背景上渲染一个经过抗锯齿的不透明物体，默认就会得到预乘值。假设一个白色 (1, 1, 1) 三角形的边缘覆盖某个像素的 40%。通过（极其精确的）抗锯齿，像素值将设为 0.4 的灰色，也就是说，为该像素保存的颜色是 (0.4, 0.4, 0.4)。如果保存 Alpha 值，它也会是 0.4，因为这就是三角形覆盖的面积比例。RGBα 值将是 (0.4, 0.4, 0.4, 0.4)，也就是一个预乘值。

图像的另一种存储方式是使用**未乘 Alpha**（unmultiplied alpha），也称为**非关联 Alpha**（unassociated alpha），甚至还有一个读起来颇费脑筋的名称：**非预乘 Alpha**（nonpremultiplied alpha）。未乘 Alpha 正如其名称所示：RGB 值没有乘以 Alpha 值。对于白色三角形的例子，未乘的颜色将是 (1, 1, 1, 0.4)。这种表示的优点是能够保存三角形的原始颜色，但在显示之前，总需要将该颜色乘以已存储的 Alpha。在执行过滤和混合时，最好始终使用预乘数据，因为在线性插值等操作中，使用未乘 Alpha 不能得到正确结果 [108, 164]，可能导致物体边缘出现黑边等伪影 [295, 648]。进一步讨论见第 6.6 节末尾。预乘 Alpha 也允许更清晰的理论处理 [1662]。

对于图像处理应用，非关联 Alpha 很有用：它可以为照片添加遮罩，却不影响底层图像的原始数据。此外，非关联 Alpha 还意味着可以使用颜色通道的完整精度范围。不过，必须注意在未乘的 RGBα 值与计算机图形学计算所用的线性空间之间正确转换。例如，没有浏览器正确地完成这一转换，而且它们也不大可能改成正确行为，因为用户现在已经预期它们会表现出这种错误行为 [649]。支持 Alpha 的图像文件格式包括 PNG（仅支持非关联 Alpha）、OpenEXR（仅支持关联 Alpha）和 TIFF（支持这两种 Alpha）。

与 Alpha 通道相关的另一个概念是**色键抠像**（chroma-keying）[199]。这个术语来自视频制作：在绿色或蓝色背景前拍摄演员，再将其与背景混合。在电影行业，这一过程称为**绿幕抠像**（green-screening）或**蓝幕抠像**（blue-screening）。其思路是将一种特定色相（用于电影制作）或一个精确颜色值（用于计算机图形学）指定为透明；只要检测到它，就显示背景。这样，仅使用 RGB 颜色就可以赋予图像某种轮廓形状，而不需要存储 Alpha。这种方案的一个缺点是：在任意像素处，物体不是完全不透明，就是完全透明，即 Alpha 实际上只有 1.0 或 0.0。例如，GIF 格式允许将一种颜色指定为透明色。


## 5.6 显示编码

原书来源：《Real-Time Rendering, Fourth Edition》第 5 章，第 160—165 页（PDF 第 181—186 页）。范围从“5.6 Display Encoding”起，至“Further Reading and Resources”之前；章末延伸阅读另见《05.99_延伸阅读与资源.md》。

当我们计算光照、纹理映射或其他操作的效果时，所使用的数值都被假定为**线性**的。非正式地说，这意味着加法和乘法会按预期工作。然而，为了避免各种视觉伪影，显示缓冲区和纹理使用了非线性编码，我们必须将其考虑在内。一个简短而粗略的回答是：取着色器输出的、位于 [0, 1] 范围内的颜色值，对其求 1/2.2 次幂，执行所谓的 **伽马校正**（gamma correction）。对输入的纹理和颜色则执行相反操作。在大多数情况下，可以让 GPU 替你完成这些事情。本节将解释这段简述中的做法及其原因。

我们从**阴极射线管**（cathode-ray tube，CRT）说起。在数字成像的早期，CRT 显示器是主流。这些设备的输入电压与显示辐亮度之间呈幂律关系。随着施加到像素上的能量水平提高，发出的辐亮度并不线性增长，而是出人意料地与该水平的大于 1 次幂成正比。例如，假定指数为 2。一个设置为 50% 的像素，发出的光只有设置为 1.0 的像素的四分之一，即 0.![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_06_b91b7e37e15334.png) = 0.25 [607]。虽然 LCD 和其他显示技术具有不同于 CRT 的固有色调响应曲线，但它们在制造时都配有转换电路，使其模拟 CRT 的响应。

这个幂函数几乎与人类视觉明度敏感性的反函数相吻合 [1431]。这一幸运的巧合，使得这种编码大致具有**感知均匀性**。也就是说，在整个可显示范围内，一对编码值 N 与 N + 1 之间的感知差异大致保持不变。以**阈值对比度**衡量，在很广泛的条件下，我们能够察觉约 1% 的明度差异。当颜色存储在精度有限的显示缓冲区中时，这种近乎最优的数值分布能最大限度地减少**色带伪影**（第 23.6 节）。同样的好处也适用于通常采用相同编码的纹理。

**显示传递函数**描述了显示缓冲区中的数字值与显示器发出的辐亮度水平之间的关系。因此，它也称为**电光传递函数**（electrical optical transfer function，EOTF）。显示传递函数是硬件的一部分，计算机显示器、电视机和电影放映机各有不同的标准。在这一过程的另一端，即图像与视频采集设备处，也存在一种标准传递函数，称为**光电传递函数**（optical electric transfer function，OETF）[672]。

对线性颜色值进行编码以供显示时，我们的目标是抵消显示传递函数的影响，使计算得到的任何数值都能产生与之对应的辐亮度水平。例如，如果计算值加倍，我们希望输出辐亮度也加倍。为了保持这种联系，我们应用显示传递函数的反函数，以抵消其非线性影响。这种消除显示器响应曲线影响的过程也叫作伽马校正，其命名原因很快就会说明。解码纹理值时，则需要应用显示传递函数，以生成可用于着色的线性值。图 5.39 展示了解码和编码在显示过程中的应用。


![图 5.39 显示过程中的解码、编码与显示传递函数](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_6_5.39.png)

**图 5.39** 左侧，GPU 着色器访问一张 PNG 颜色纹理，将其非线性编码值转换为线性值（蓝色）。经过着色和色调映射（第 8.2.2 节）之后，最终计算值经过编码（绿色），存入帧缓冲区。该值与显示传递函数共同决定发出的辐亮度大小（红色）。绿色和红色函数组合起来会相互抵消，因此发出的辐亮度与计算得到的线性值成正比。

个人计算机显示器的标准传递函数由名为 **sRGB** 的色彩空间规范定义。大多数控制 GPU 的 API 都可以设置为：在从纹理读取数值或向颜色缓冲区写入数值时，自动执行适当的 sRGB 转换 [491]。如第 6.2.2 节所述，mipmap 的生成也会考虑 sRGB 编码。纹理值之间的双线性插值会先转换为线性值，再进行插值，从而正确工作。Alpha 混合也会正确执行：先将存储的值解码回线性值，再混入新值，最后对结果进行编码。

务必在渲染的最后阶段，即为显示而向帧缓冲区写入数值时，应用这一转换。如果在显示编码之后再进行后处理，这些效果就会在非线性值上计算，这通常是不正确的，而且往往会引起伪影。显示编码可以看作一种压缩形式，它能尽可能地保留数值的感知效果 [491]。理解这一领域的一种好方法是：我们使用线性值进行物理计算；而每当要显示结果，或者访问颜色纹理之类可供显示的图像时，都需要使用正确的编码或解码变换，在数据与其显示编码形式之间转换。

如果确实需要手动应用 sRGB，可以使用标准转换公式，也可以使用几个简化版本之一。实际而言，显示器由每个颜色通道的一定数量的位来控制，例如消费级显示器通常为 8 位，给出 [0, 255] 范围内的一组等级。这里忽略位数，将显示编码后的等级表示为 [0.0, 1.0] 范围。线性值也位于 [0.0, 1.0] 范围内，表示浮点数。我们以 x 表示这些线性值，以 y 表示帧缓冲区中存储的非线性编码值。为了将线性值转换为 sRGB 非线性编码值，我们应用 sRGB 显示传递函数的反函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_06_0f7c3a2b311946.png)


其中 x 代表线性 RGB 三元组的一个通道。这个公式应用于每个通道，生成的三个数值用于驱动显示器。手动应用转换函数时要小心。一种错误来源是使用了编码后的颜色，而非其线性形式；另一种则是对同一个颜色解码或编码两次。

这两个变换表达式中，下面一个是简单的乘法，它来自数字硬件对变换完全可逆的要求 [1431]。上面一个表达式包含求幂运算，适用于输入值 x 的几乎整个 [0.0, 1.0] 范围。考虑偏移和缩放之后，这个函数可以由一个更简单的公式很好地近似 [491]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_06_17100fe2114042.png)


其中 γ = 2.2。希腊字母 γ 正是“伽马校正”这一名称的由来。

正如计算值必须经过编码才能显示，静态相机或摄像机捕获的图像也必须先转换为线性值，才能用于计算。你在显示器或电视上看到的任何颜色，都有一个显示编码后的 RGB 三元组，可以通过屏幕截图或取色器获得。这些值就是 PNG、JPEG 和 GIF 等文件格式中存储的数值，这些格式无需转换便可直接送入帧缓冲区，在屏幕上显示。换句话说，按照定义，你在屏幕上看到的任何内容都是显示编码后的数据。在着色计算中使用这些颜色之前，必须将它们从这种编码形式转换回线性值。从显示编码到线性值所需的 sRGB 变换为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_06_dc48107d44ccba.png)


其中 y 表示归一化后的显示通道值，也就是图像或帧缓冲区中存储的值，以 [0.0, 1.0] 范围内的数值表示。这个解码函数是前述 sRGB 公式的反函数。这意味着，如果着色器访问一张纹理并原样输出，它看起来就会与处理之前相同，符合预期。解码函数与显示传递函数相同，是因为纹理中存储的值已经过编码，以便正确显示。这里进行转换的目的，是得到线性值，而非得到线性响应的显示。

更简单的伽马显示传递函数是公式（5.31）的反函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_06_b2ef6460c3e145.png)


有时你还会见到一对更简单的转换，尤其是在移动端和浏览器应用中 [1666]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_05_06_87209c4c1b3e4d.png)


也就是说，为显示而转换时，对线性值取平方根；执行逆变换时，只需将数值与其自身相乘。虽然这只是粗略近似，但仍比完全忽略这个问题要好。


![图 5.40 两束重叠聚光灯的伽马校正对比](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_6_5.40.png)

**图 5.40** 两束相互重叠的聚光灯照亮一个平面。左图中，将 0.6 和 0.4 这两个光照值相加后，并未执行伽马校正。加法实际上作用于非线性值，因此产生错误。注意，左侧光束看起来明显比右侧亮，而重叠区域显得不真实地明亮。右图中，数值相加后执行了伽马校正。光束本身按比例变亮，并在重叠处正确地组合。

如果不注意伽马，较低的线性值在屏幕上会显得过暗。另一个相关错误是，不执行伽马校正可能使某些颜色的色相发生偏移。假设 γ = 2.2。我们希望显示像素发出的辐亮度与计算得到的线性值成正比，这意味着必须对线性值求 1/2.2 次幂。线性值 0.1 得到 0.351，0.2 得到 0.481，0.5 得到 0.730。如果不进行编码，直接使用这些线性值，就会使显示器发出的辐亮度低于所需值。注意，0.0 和 1.0 在这些变换中始终保持不变。在开始使用伽马校正之前，场景建模人员常常人为提高暗表面的颜色值，相当于把显示逆变换的效果纳入其中。

忽略伽马校正的另一个问题是：原本对物理上线性的辐亮度值才正确的着色计算，被应用到了非线性值上。图 5.40 展示了一个例子。

忽略伽马校正也会影响抗锯齿边缘的质量。例如，假设一个三角形的边缘覆盖了四个屏幕网格单元（图 5.41）。三角形的归一化辐亮度为 1（白色），背景则为 0（黑色）。从左到右，各单元的覆盖比例分别为 ⅛、⅜、⅝ 和 ⅞。因此，如果采用盒式滤波器，我们希望将各像素的归一化线性辐亮度表示为 0.125、0.375、0.625 和 0.875。正确做法是在各线性值上执行抗锯齿，再对得到的四个结果值应用编码函数。如果不这样做，各像素所表示的辐亮度就会过暗，使人感觉边缘发生了变形，如图的右侧所示。这种伪影叫作 **绳索效应**（roping），因为边缘看起来有些像扭绞的绳子 [167, 1265]。图 5.42 展示了这种效果。


![图 5.41 覆盖比例及未经伽马校正的边缘变形](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_6_5.41.png)

**图 5.41** 左侧，黑色背景（图中以灰色表示）上一个白色三角形的边缘覆盖了四个像素，图示为真实的面积覆盖情况。如果不执行伽马校正，中间色调会变暗，导致感知到的边缘形状失真，如右侧所示。


![图 5.42 抗锯齿线条的伽马校正程度对比](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_5_6_5.42.png)

**图 5.42** 左侧，一组抗锯齿线条经过了伽马校正；中间，只进行了部分校正；右侧，未进行伽马校正。（图片由 Scott R. Nelson 惠允提供。）

sRGB 标准制定于 1996 年，已成为大多数计算机显示器的标准。然而，自那时以来，显示技术已有发展。人们研制出了亮度更高、能够显示更广泛颜色范围的显示器。第 8.1.3 节讨论颜色显示与亮度，第 8.2.1 节介绍高动态范围显示器的显示编码。Hart 的文章 [672] 对先进显示器有尤为详尽的介绍，可供进一步阅读。


## 第 5 章 延伸阅读与资源

原书来源：《Real-Time Rendering, Fourth Edition》第 5 章末尾“Further Reading and Resources”，第 165—166 页（PDF 第 186—187 页）。本部分在原书中未编号，独立保存，不重复纳入第 5.6 节。

Pharr 等人 [1413] 更深入地讨论了采样模式和抗锯齿。Teschner 的课程讲义 [1758] 展示了多种采样模式生成方法。Drobot [382, 383] 梳理了实时抗锯齿的既有研究，解释了各种技术的特性和性能。有关多种形态学抗锯齿方法的信息，可在相关 SIGGRAPH 课程的讲义 [829] 中找到。Reshetov 和 Jimenez [1486] 对游戏中使用的形态学抗锯齿及相关时间抗锯齿研究提供了更新的回顾。

关于透明性研究，我们再次建议感兴趣的读者参阅 McGuire 的演讲 [1182] 和 Wyman 的研究 [1931]。Blinn 的文章《什么是像素？》（“What Is a Pixel?”）[169] 在讨论不同定义的同时，精彩地介绍了计算机图形学的多个领域。Blinn 的《Dirty Pixels》（《脏像素》）和《Notation, Notation, Notation》（《记号，记号，记号》）两本书 [166, 168] 收录了一些关于滤波和抗锯齿的入门文章，以及关于 alpha、合成和伽马校正的文章。Jimenez 的演讲 [836] 详细介绍了用于抗锯齿的前沿技术。

Gritz 和 d’Eon [607] 对伽马校正问题作了出色的总结。Poynton 的书 [1431] 扎实地介绍了各种媒体中的伽马校正，以及其他与颜色有关的主题。Selan 的白皮书 [1602] 是较新的资料，解释了显示编码及其在电影行业中的应用，还介绍了许多其他相关信息。
