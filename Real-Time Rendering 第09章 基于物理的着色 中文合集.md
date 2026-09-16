# Real-Time Rendering 第09章 基于物理的着色 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [9.1 光的物理原理](<Real-Time_Rendering_4th_中文/第09章/09.01.md>)

- [9.2 相机](<Real-Time_Rendering_4th_中文/第09章/09.02.md>)

- [9.3 双向反射分布函数（BRDF）](<Real-Time_Rendering_4th_中文/第09章/09.03.md>)

- [9.4 光照](<Real-Time_Rendering_4th_中文/第09章/09.04.md>)

- [9.5 菲涅耳反射率](<Real-Time_Rendering_4th_中文/第09章/09.05.md>)

- [9.6 微观几何](<Real-Time_Rendering_4th_中文/第09章/09.06.md>)

- [9.7 微表面理论](<Real-Time_Rendering_4th_中文/第09章/09.07.md>)

- [9.8 表面反射的 BRDF 模型](<Real-Time_Rendering_4th_中文/第09章/09.08.md>)

- [9.9 次表面散射的 BRDF 模型](<Real-Time_Rendering_4th_中文/第09章/09.09.md>)

- [9.10 布料的 BRDF 模型](<Real-Time_Rendering_4th_中文/第09章/09.10.md>)

- [9.11 波动光学 BRDF 模型](<Real-Time_Rendering_4th_中文/第09章/09.11.md>)

- [9.12 分层材质](<Real-Time_Rendering_4th_中文/第09章/09.12.md>)

- [9.13 材质的混合与过滤](<Real-Time_Rendering_4th_中文/第09章/09.13.md>)


## 章首导言

来源：《Real-Time Rendering》第4版，书页293（PDF物理页314）；本文件仅含章首标题、题辞与第9.1节之前的导言。

> “无论一个物体的形状如何，光、明暗与透视总能使它美丽。”
>
> ——约翰·康斯特布尔（John Constable）

本章介绍基于物理的着色的各个方面。首先，第9.1节描述光与物质相互作用的物理原理；第9.2至9.4节说明这些物理原理如何与着色过程相联系。第9.5至9.7节专门讨论构建基于物理的着色模型所使用的基本组成部分，而这些模型本身则在第9.8至9.12节中讨论，涵盖广泛的材质类型。最后，第9.13节介绍如何将材质混合在一起，并讨论用于避免走样、保持表面外观的滤波方法。


## 9.1 光的物理原理

来源：《Real-Time Rendering》第4版，书页293—307（PDF物理页314—328），从第9.1节标题起至第9.2节标题前，含全部四个下级小节、图9.1—9.15及公式（9.1）。章首导言另存于09.00_章首导言.md。

光与物质的相互作用构成了基于物理的着色的基础。要理解这些相互作用，先对光的本质有一些基本认识会很有帮助。

在物理光学中，光被建模为电磁横波，即电场和磁场在垂直于传播方向的方向上振荡的波。两种场的振荡相互耦合。磁场和电场矢量彼此垂直，其长度之比是固定的。这个比值等于相速度，后文将讨论相速度。

图9.1展示了一束简单的光波。事实上，它是可能存在的最简单的光波：一个完美的正弦函数。这种波只有一个波长，用希腊字母λ（lambda）表示。正如第8.1节所述，人所感知的光的颜色与波长密切相关。因此，只有一个波长的光称为单色光，意即“单一颜色”。然而，实际遇到的大多数光波都是多色光，包含许多不同的波长。

图9.1中的光波在另一方面也格外简单：它是线偏振光。这意味着，对于空间中的一个固定点，电场和磁场各自沿一条直线来回运动。相比之下，本书着重讨论更为常见的非偏振光。在非偏振光中，场的振荡均匀分布在垂直于传播轴的所有方向上。尽管单色线偏振波很简单，了解其行为仍很有用，因为任何光波都可以分解为这类波的组合。


![图9.1](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.1.png)

**图9.1** 光是一种电磁横波。电场与磁场矢量的振荡方向彼此成90°，并且都与传播方向成90°。图中所示的是可能存在的最简单的光波：它既是单色的（只有一个波长λ），又是线偏振的（电场与磁场各自沿一条直线振荡）。

如果随时间跟踪波上一个具有给定相位的点，例如振幅峰值，就会看到它以恒定速度在空间中移动，这个速度就是波的相速度。对于在真空中传播的光波，相速度为c，通常称为光速，约为每秒300,000千米。

第8.1.1节讨论过，可见光的一个波长约为400—700纳米。为了直观理解这个长度，它大约是一根蛛丝宽度的二分之一到三分之一，而蛛丝本身的宽度又不到人类头发的五十分之一。见图9.2。在光学中，经常用相对于光波长的大小来描述结构尺寸。此时，我们可以说，一根蛛丝的宽度约为2λ—3λ（2—3个光波长），而一根头发的宽度约为100λ—200λ。

光波携带能量。能流密度等于电场和磁场大小的乘积；由于这两个大小彼此成正比，它也与电场大小的平方成正比。我们主要关注电场，因为它对物质的作用远强于磁场。在渲染中，我们关心的是随时间平均的能流，它与波振幅的平方成正比。这种平均能流密度就是辐照度，用字母E表示。辐照度及其与其他光学量的关系已在第8.1.1节讨论。


![图9.2](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.2.png)

**图9.2** 左图展示可见光波长与一根蛛丝的相对大小，这根蛛丝宽度略大于1微米。右图将类似的一根蛛丝放在人类头发旁，以提供进一步的尺度参照。（图片由URnano／罗切斯特大学提供。）

光波以线性方式组合，总波形是各组成波的和。但是，由于辐照度与振幅的平方成正比，这似乎会导致一个悖论。例如，把两个相同的波相加，难道不会使辐照度出现“1 + 1 = 4”的情况吗？而辐照度衡量的是能流，这难道不违反能量守恒吗？这两个问题的答案分别是“有时会”和“不会”。

为说明这一点，我们看一个简单情形：将n个除相位外完全相同的单色波相加。每个波的振幅均为a。如前所述，每个波的辐照度![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_44a84a3087a706.png)与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_5929c46ee0c3fc.png)成正比，换言之，对于某个常数k，有![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_44a84a3087a706.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_a9056dcb3e5641.png)。

图9.3给出这种情形的三个例子。左侧所有波相位一致、彼此加强。合成波的辐照度为单个波的![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_09c317452584b9.png)倍，也就是各个波辐照度之和的n倍。这种情况称为相长干涉。图中央，每对波的相位相反，相互抵消。合成波的振幅和辐照度都为零。这种情况称为相消干涉。


![图9.3](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.3.png)

**图9.3** 将n个频率、偏振和振幅相同的单色波相加的三种情形。从左至右依次为相长干涉、相消干涉和非相干叠加。每种情形都展示了合成波（下方）相对于n个原始波（上方）的振幅和辐照度。

相长干涉和相消干涉是相干叠加的两个特例：各波的波峰和波谷以某种一致的方式对齐。取决于相对相位关系，n个相同波的相干叠加，可以产生辐照度介于单个波的0倍与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_09c317452584b9.png)倍之间的任意波。

不过，波在叠加时通常彼此不相干，意味着它们的相位相对随机。图9.3右侧说明了这种情况。此时合成波的振幅为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_0f970bdca15cb2.png)，各波的辐照度线性相加，得到单个波辐照度的n倍，符合我们的预期。

相消和相长干涉看起来似乎违反能量守恒。但是图9.3没有展示完整情况，它只显示了一个位置上的波相互作用。随着波在空间中传播，波之间的相位关系会因位置而异，如图9.4所示。在某些位置，波发生相长干涉，合成波的辐照度大于各波辐照度之和。在另一些位置，它们发生相消干涉，使合成辐照度小于各波辐照度之和。这并不违反能量守恒定律，因为相长干涉增加的能量与相消干涉减少的能量总会相互抵消。


![图9.4](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.4.png)

**图9.4** 从两个点源向外扩散、频率相同的单色波。它们在不同空间区域发生相长干涉和相消干涉。

物体中的电荷振荡时，就会发射光波。引起振荡的一部分能量——热能、电能或化学能——会转化为光能，从物体向外辐射。在渲染中，这样的物体被视为光源。我们最早在第5.2节讨论光源，第10章将从更加基于物理的角度介绍它们。

光波发射后会在空间中传播，直到遇到可以与之相互作用的物质。大多数光与物质相互作用背后的核心现象很简单，与前述发射情形也很相似。振荡的电场推拉物质中的电荷，使电荷也随之振荡。振荡的电荷发射新的光波，把入射光波的一部分能量转向新的方向。这种反应称为散射，是多种光学现象的基础。

散射光波与原波具有相同频率。通常，原波含有多种频率的光，每种频率都独立地与物质相互作用。一个频率的入射光能量不会对另一个频率的发射光能量作出贡献，只有荧光和磷光等特定且相对少见的情况例外，本书不讨论这些情况。

一个孤立分子会向所有方向散射光，强度随方向略有变化。在接近原始传播轴的方向上，无论向前还是向后，散射的光都更多。分子作为散射体的有效程度——即它附近的光波发生散射的概率——强烈依赖于波长。短波长光比长波长光更容易被散射。

在渲染中，我们关心的是大量分子的集合。光与这类集合的相互作用，不一定类似于与孤立分子的相互作用。相邻分子散射的波源自同一个入射波，因此往往彼此相干，并表现出干涉。本节余下部分将讨论多个分子散射光的几种重要特殊情况。

### 9.1.1 粒子

在理想气体中，分子彼此不产生影响，因此它们的相对位置完全随机且不相关。尽管这是一种抽象，但它能相当好地描述正常大气压下的空气。在这种情况下，不同分子散射的波之间的相位差随机且不断变化。因此，散射波是不相干的，其能量线性相加，如图9.3右侧所示。换言之，n个分子散射的总光能量是单个分子散射光能量的n倍。

相比之下，如果分子紧密聚集成远小于一个光波长的团簇，每个团簇内的散射光波就会同相，发生相长干涉。这使散射波能量按平方关系相加，如图9.3左侧所示。因此，由n个分子组成的小团簇所散射的光强，是单个分子的![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_09c317452584b9.png)倍，也就是相同数量分子处于理想气体中时所散射光强的n倍。这个关系意味着，在每立方米分子密度固定的情况下，将分子聚集为团簇会显著增加散射光强。在保持总体分子密度不变的同时继续增大团簇，还会进一步增加散射光强，直到团簇直径接近一个光波长。超过这一点，再增大团簇尺寸也不会继续提高散射光强[469]。

这个过程解释了云和雾为何如此强烈地散射光。二者都由凝结产生，也就是空气中的水分子聚集成越来越大的团簇的过程。即使水分子总体密度不变，这也会显著增强光散射。云的渲染将在第14.4.2节讨论。

讨论光散射时，“粒子”一词既指孤立分子，也指多分子团簇。直径小于一个波长的多分子粒子的散射，是孤立分子散射通过相长干涉得到的增强版本，因此具有相同的方向变化规律和波长依赖性。对于大气中的粒子，这种散射称为瑞利散射；对于嵌在固体中的粒子，则称为丁达尔散射。

当粒子尺寸增大到超过一个波长时，散射波不再在整个粒子上保持同相，这改变了散射特性。散射越来越偏向前方，而对波长的依赖性逐渐减弱，直到所有可见光波长都被同等散射。这种散射称为米氏散射。第14.1节将更详细地介绍瑞利散射和米氏散射。

### 9.1.2 介质

另一个重要情况是光在均匀介质中传播。均匀介质是由间距均匀的相同分子填充的体积。分子间距不必像晶体中那样完全规则。如果成分纯净（所有分子相同），并且没有空隙或气泡，液体和非晶固体也可以在光学上是均匀的。

在均匀介质中，散射波的排列方式使它们在原传播方向之外的所有方向上都发生相消干涉。将原波与各个分子散射的所有波组合后，最终结果与原波相同，只是相速度以及某些情况下的振幅有所不同。最终的波不表现出任何散射：散射实际上已被相消干涉抑制了。

原波与新波的相速度之比定义了介质的一种光学性质，称为折射率（index of refraction，IOR，也称refractive index），用字母n表示。一些介质具有吸收性。它们将部分光能转化为热，使波振幅随距离呈指数衰减。衰减速率由衰减指数定义，用希腊字母κ（kappa）表示。n和κ通常都随波长变化。这两个数共同完整定义了介质如何影响给定波长的光，常合并成一个复数n + iκ，称为复折射率。折射率抽象掉了分子层面的光相互作用细节，使我们能把介质当作连续体来处理，大大简化了问题。


![图9.5](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.5.png)

**图9.5** 四个盛有不同吸收性质液体的小容器。从左至右为清水、加入石榴糖浆的水、茶和咖啡。

虽然光的相速度本身不会直接影响外观，但速度的变化会产生影响，后文将作解释。另一方面，光吸收直接影响视觉效果，因为它会降低光强，并且在吸收随波长变化时，还会改变光的颜色。图9.5展示了一些光吸收的例子。

非均匀介质常常可以建模为嵌有散射粒子的均匀介质。在均匀介质中抑制散射的相消干涉，源于分子排列均匀，因而它们产生的散射波也均匀排列。分子分布的任何局部变化都会破坏这种相消干涉模式，使散射光波能够传播。这类局部变化可以是不同类型分子的团簇、空气间隙、气泡或密度变化。无论哪种情况，它都会像前面讨论的粒子一样散射光，且散射性质同样取决于团簇尺寸。甚至气体也可以这样建模；其中的“散射粒子”是分子持续运动引起的瞬时密度涨落。这个模型使我们能够为气体建立一个有意义的n值，有助于理解其光学性质。图9.6展示了一些光散射的例子。


![图9.6](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.6.png)

**图9.6** 从左至右为水、加入几滴牛奶的水、约含10%牛奶的水、全脂牛奶以及乳光玻璃。牛奶中的大多数散射粒子大于可见光波长，因此其散射基本不带颜色，中间图像可见淡淡的蓝色。乳光玻璃中的散射粒子都小于可见光波长，因此对蓝光的散射比对红光更强。由于背景分为明暗两部分，左侧更容易看到透射光，右侧更容易看到散射光。

散射和吸收都与尺度有关。在小场景中不产生任何明显散射的介质，到了较大尺度上，散射可能十分明显。例如，在房间中观察一杯水时，看不出空气中的光散射和水中的光吸收。然而，在广阔环境中，这两种效应都可能十分显著，如图9.7所示。


![图9.7](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.7.png)

**图9.7** 左图说明，在数米的传播距离上，水会相当强烈地吸收光，尤其是红光。右图说明，即使没有严重污染或雾，光穿过数英里的空气后，也会出现明显散射。

一般情况下，介质外观由散射和吸收的某种组合产生，如图9.8所示。散射程度决定浑浊程度，强散射会产生不透明外观。除了图9.6的乳光玻璃等相对少见的例外，固体和液体介质中的粒子通常大于一个光波长，往往会同等散射所有可见波长的光。因此，任何色调通常都是吸收的波长依赖性造成的。介质的明度则由两种现象共同决定。特别地，白色是强散射与弱吸收相结合的结果。第14.1节将对此作更详细讨论。


![图9.8](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.8.png)

**图9.8** 容器中的液体表现出各种不同的吸收与散射组合。

### 9.1.3 表面

从光学角度看，物体表面是分隔不同折射率体积的二维界面。在典型渲染情境中，外部体积包含空气，其折射率约为1.003，为简化起见常假定为1。内部体积的折射率取决于构成物体的物质。

> 译注：原书此处确实印为“1.003”，本译文保留原值；该数值疑有排印问题。

当光波击中表面时，表面的两个方面会对结果产生重要影响：两侧的物质，以及表面的几何形状。我们先关注物质，并假定最简单的表面几何形状，即完全平坦的平面。把“外侧”（来波即入射波的来源一侧）的折射率记为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_3a00723a6f5471.png)，把“内侧”（波穿过表面后向其中透射的一侧）的折射率记为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_d13080baeee9f5.png)。

前一小节已经说明，当光波遇到材质成分或密度的不连续，也就是折射率的不连续时，会发生散射。分隔不同折射率的平面表面是一种特殊的不连续，会以特定方式散射光。边界条件要求平行于表面的电场分量连续。换言之，电场矢量在表面平面上的投影，在表面两侧必须相符。这意味着：

1. 在表面上，任何散射波都必须与入射波同相，或相差180°。因此，散射波的波峰必须与入射波的波峰或波谷对齐。这把散射波限制在两个可能方向：一个继续向前进入表面，另一个离开表面向后退去。前者是透射波，后者是反射波。
2. 散射波必须与入射波具有相同频率。这里假设单色波，但只要先将一般的波分解为单色分量，所讨论的原理就能应用于任何波。
3. 光波从一种介质进入另一种介质时，相速度——波在介质中传播的速度——按相对折射率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_3a00723a6f5471.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_d13080baeee9f5.png)成比例变化。由于频率固定，波长也按![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_3a00723a6f5471.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_d13080baeee9f5.png)成比例变化。


![图9.9](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.9.png)

**图9.9** 光波击中分隔折射率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_3a00723a6f5471.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_d13080baeee9f5.png)的平面表面。左侧是侧视图，入射波从左上方到来。红色条带的强度表示波的相位。表面下方的波间距按比值![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_3a00723a6f5471.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_d13080baeee9f5.png)改变，本例中该比值为0.5。相位沿表面对齐，因此间距变化使透射波方向弯折（折射）。构造的三角形展示了斯涅尔定律的推导。为清晰起见，右上方单独展示反射波。它的波间距与入射波相同，因此其方向与表面法线的夹角也相同。右下方展示波的方向矢量。

最终结果如图9.9所示。反射波和入射波的方向与表面法线具有相同夹角![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_ba4635370b500e.png)。透射波方向发生弯折（折射），与法线夹角为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_ca26290f1bb6de.png)，它与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_ba4635370b500e.png)的关系为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_01_d9bd1d310531e1.png)


这个折射方程称为斯涅尔定律。它用于全局折射效果，第14.5.2节将进一步讨论。

虽然折射常与玻璃、水晶等清澈材质联系在一起，但它也发生在不透明物体的表面。当不透明物体发生折射时，光会在物体内部经历散射和吸收。光与物体介质相互作用，就像图9.8中那些盛有不同液体的杯子一样。对于金属，其内部包含许多自由电子（未束缚于分子的电子），会“吸收”折射光能，并将它重新导入反射波中。这就是金属既具有高吸收性又具有高反射率的原因。

我们讨论的表面折射现象——反射和折射——要求折射率发生突变，而且变化发生在小于一个波长的距离内。更渐进的折射率变化不会使光分成两束，而会使光路弯曲，相当于折射中不连续弯折的连续版本。当温度导致空气密度发生变化时，经常可以看到这种效应，例如海市蜃楼和热扰动畸变。见图9.10。


![图9.10](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.10.png)

**图9.10** 折射率逐渐变化导致光路弯曲的例子，本例中变化由温度差异引起。（《EE Lightnings heat haze》，Paul Lucas，依据CC BY 2.0许可使用。）

即使一个物体具有明确的边界，如果浸在折射率相同的物质中，也不会有可见表面。没有折射率变化，就不会发生反射和折射。图9.11展示了一个例子。


![图9.11](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.11.png)

**图9.11** 这些装饰珠子的折射率与水相同。在水面以上，由于珠子的折射率与空气不同，它们有可见表面。在水下，珠子表面两侧折射率相同，所以表面不可见。珠子本身只是由于其有色吸收才可见。

到目前为止，我们关注的是表面两侧物质的影响。现在讨论影响表面外观的另一个重要因素：几何形状。

严格来说，完全平坦的平面表面不可能存在。每个表面都有某种不规则性，哪怕只是组成表面的各个原子。然而，远小于一个波长的表面不规则结构不会影响光；远大于一个波长的不规则结构则实际上使表面倾斜，而不影响其局部平坦性。只有尺寸处于1—100个波长范围的不规则结构，才会通过称为衍射的现象，使表面表现出不同于平面的行为。第9.11节将进一步讨论衍射。

在渲染中，我们通常使用几何光学，忽略干涉、衍射等波动效应。这相当于假定所有表面不规则结构都小于一个光波长，或者远大于一个光波长。在几何光学中，光被建模为射线，而不是波。在光线与表面相交的位置，局部表面被视为平面。图9.9右下方的示意图可以看作几何光学中的反射和折射图景，与该图其他部分的波动图景形成对照。从这里开始，我们将一直停留在几何光学范围内，直到专门讨论基于波动光学的着色模型的第9.11节。

如前所述，远大于一个波长的表面不规则结构会改变表面的局部朝向。当这些结构小到无法分别渲染时，换言之小于一个像素时，我们称之为微观几何。反射和折射方向取决于表面法线。微观几何的作用是在表面不同点改变法线，从而改变反射光与折射光的方向。

尽管表面上每一个具体点只向单一方向反射光，但每个像素覆盖了许多表面点，它们向各种方向反射光。外观由所有这些不同反射方向的综合结果决定。图9.12展示了两个表面，它们的宏观形状相似，但微观几何差异显著。


![图9.12](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.12.png)

**图9.12** 左侧是两个表面的照片，右侧是其微观结构示意图。上方表面的微观几何略为粗糙。入射光线击中朝向略有不同的表面点，并在一个狭窄锥体内的方向上反射。可见效果是反射稍微模糊。下方表面的微观几何更粗糙。入射光线击中的表面点具有明显不同的朝向，反射光在宽阔的锥体内散开，造成更模糊的反射。

对于渲染，我们不显式建模微观几何，而是进行统计处理，把表面看作具有随机分布的微观结构法线。因此，我们把表面建模为在连续的一片方向范围内反射（和折射）光。这个范围的宽度，以及由此决定的反射和折射细节的模糊程度，取决于微观几何法线矢量的统计方差，也就是表面在微观尺度上的粗糙度。见图9.13。


![图9.13](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.13.png)

**图9.13** 从宏观角度观察时，可以将表面视为向多个方向反射和折射光。

### 9.1.4 次表面散射

折射光继续与物体内部体积相互作用。如前所述，金属反射大部分入射光，并迅速吸收其余部分。相比之下，非金属表现出多种多样的散射与吸收行为，类似于图9.8中各杯液体的情况。散射和吸收都很低的材质是透明的，任何折射光都能透射穿过整个物体。第5.5节讨论了不考虑折射时渲染此类材质的简单方法，而第14.5.2节将详细介绍折射。本章主要关注不透明物体：透射光在其中经历多次散射和吸收事件，最终一部分光从表面重新射出。见图9.14。


![图9.14](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.14.png)

**图9.14** 折射光在材质中传播时会被吸收。在这个例子中，吸收主要发生在较长波长，剩下的主要是短波长蓝光。此外，光还被材质内部的粒子散射。最终，一部分折射光被散射回表面之外，如从表面沿各个方向射出的蓝色箭头所示。

这些经过次表面散射的光，会在距离入射点不同距离的位置离开表面。入射点与出射点间距的分布，取决于材质中散射粒子的密度和性质。这些距离与着色尺度（像素大小，或着色样本之间的距离）的关系很重要。如果相对于着色尺度，入射点与出射点的距离很小，那么为了着色，可以认为这个距离实际上为零。这样就能把次表面散射与表面反射合并为局部着色模型，使某点的出射光仅取决于同一点的入射光。不过，由于次表面散射光与表面反射光的外观明显不同，把它们拆分成独立的着色项会更方便。镜面反射项对表面反射建模，漫反射项对局部次表面散射建模。

如果相对于着色尺度，入射点与出射点的距离很大，就需要专门的渲染技术，来表现光从表面一点进入、又从另一点离开的视觉效果。这些全局次表面散射技术将在第14.6节详细介绍。图9.15说明了局部与全局次表面散射的区别。


![图9.15](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_1_9.15.png)

**图9.15** 左侧正在渲染一种具有次表面散射的材质。图中用黄色和紫色表示两种不同的采样大小。黄色大圆代表一个着色样本，它覆盖的区域大于次表面散射距离。因此，可以忽略这些距离，从而将次表面散射视为局部着色模型中的漫反射项，如右侧独立示意图所示。如果靠近表面，着色样本的面积就会变小，如紫色小圆所示。此时，与一个着色样本覆盖的区域相比，次表面散射距离变得很大。要从这些样本生成真实的图像，就需要全局技术。

需要注意，局部与全局次表面散射技术建模的是完全相同的物理现象。每种情况下的最佳选择不仅取决于材质性质，也取决于观察尺度。例如，在渲染一个孩子玩塑料玩具的场景时，要准确渲染孩子的皮肤，很可能需要全局技术；而对于玩具，局部漫反射着色模型就足够了。这是因为皮肤中的散射距离比塑料中大得多。但是，如果相机足够远，皮肤中的散射距离会小于一个像素，此时局部着色模型对孩子和玩具都能准确适用。反过来，在极近距离特写镜头中，塑料也会表现出明显的非局部次表面散射，此时要准确渲染玩具就需要全局技术。


## 9.2 相机

原书来源：书页 307—308（PDF 第 328—329 页）；图 9.16 及其完整图注位于书页 309（PDF 第 330 页）。

如第 8.1.1 节所述，在渲染中，我们计算从被着色的表面点射向相机位置的辐亮度。这是在模拟胶片相机、数码相机或人眼等成像系统的简化模型。

这类系统包含一个由许多离散的小型传感器组成的感光面。例如，人眼中的视杆细胞和视锥细胞、数码相机中的光电二极管，以及胶片中的染料颗粒，都属于这样的传感器。每个传感器检测其表面上的辐照度值，并产生一个颜色信号。辐照度传感器本身无法生成图像，因为它们会对来自所有入射方向的光线求平均。因此，完整的成像系统包含一个不透光的外壳，其上只有一个小孔径（开口），用来限制光能够进入并照射到传感器上的方向。在孔径处放置透镜，可以使光聚焦，从而让每个传感器仅接收来自一小组入射方向的光。外壳、孔径和透镜共同作用，使传感器具有方向选择性。它们对一小块区域和一小组入射方向上的光求平均。这些传感器测量的并非平均辐照度——正如我们在第 8.1.1 节中所见，辐照度量化的是来自所有方向的光通量的面密度——而是平均辐亮度，后者量化的是单条光线的亮度和颜色。

过去，渲染一直采用一种特别简单的成像传感器模型，称为针孔相机，如图 9.16 的上图所示。针孔相机的孔径极小——理想情况下是一个大小为零的数学点——并且没有透镜。这个点状孔径使感光面上的每个点只能收集一条光线，而一个离散传感器收集的是一个狭窄光锥中的光线：光锥的底面覆盖该传感器表面，顶点位于孔径处。渲染系统以略有不同（但等价）的方式对针孔相机建模，如图 9.16 的中图所示。针孔孔径的位置由点 **c** 表示，通常称为“相机位置”或“视点位置”。这个点也是透视变换的投影中心（第 4.7.2 节）。

渲染时，每个着色样本对应一条光线，因而也对应感光面上的一个采样点。抗锯齿过程（第 5.4 节）可以解释为重建每个离散传感器表面上所收集的信号。不过，由于渲染不受物理传感器的限制，我们可以更一般地将这一过程视为从离散样本重建连续图像信号。

虽然人们确实制造过真正的针孔相机，但对于实际使用的大多数相机以及人眼而言，针孔相机都不是很好的模型。图 9.16 的下图展示了一个使用透镜的成像系统模型。加入透镜后就可以采用更大的孔径，从而大幅增加成像系统收集到的光量。不过，这也会使相机的景深受到限制（第 12.4 节），使过近或过远的物体变得模糊。

除了限制景深之外，透镜还会产生另一种影响。即使对于完全合焦的点，每个传感器位置接收到的仍然是一个光锥中的光线。在理想化模型中，每个着色样本代表一条视线，这有时会引入数学奇点、数值不稳定性或视觉混叠。在渲染图像时牢记物理模型，有助于我们识别并解决这类问题。


![图 9.16：针孔相机、渲染系统中的针孔相机模型与带透镜的相机](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_2_9.16.png)

**图 9.16。** 这里的每幅相机模型图都包含一个像素传感器阵列。实线界定了其中三个传感器从场景中收集的光线集合。每幅图中的插图展示了像素传感器上的单个点样本所收集的光线。上图是针孔相机；中图是典型渲染系统对同一针孔相机的建模方式，其中相机点为 **c**；下图是带有透镜、在物理上更准确的相机。红色球体合焦，另外两个球体失焦。


## 9.3 双向反射分布函数（BRDF）

原书来源：书页 308—315（PDF 物理页 329—336），从 9.3 节标题开始，至 9.4 节标题之前；书页 309 的图 9.16 属于 9.2 节。本节无下级小节。

归根结底，基于物理的渲染就是计算沿某一组观察射线进入相机的辐亮度。采用第 8.1.1 节引入的入射辐亮度记号，对于给定的观察射线，我们需要计算的量是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_b456bbb8033d74.png)(**c**, −**v**)，其中 **c** 是相机位置，−**v** 是沿观察射线的方向。这里使用 −**v**，源于两项记号约定。首先，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_b456bbb8033d74.png)() 中的方向向量始终指向远离给定点的方向，此处给定点就是相机位置。其次，观察向量 **v** 始终指向相机。

在渲染中，场景通常被建模为一组物体，以及存在于物体之间的介质（“media”一词实际上源自表示“在中间”或“在两者之间”的拉丁语）。所讨论的介质往往是适量且相对洁净的空气，它不会明显影响射线的辐亮度，因此在渲染时可以忽略。有时，射线经过的介质会通过吸收或散射，对其辐亮度产生显著影响。这类介质称为**参与介质**，因为它们参与了光在场景中的传输。第 14 章将详细介绍参与介质。本章假设不存在参与介质，因此，进入相机的辐亮度等于最近物体表面朝相机方向离开的辐亮度：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_af0eaecfbd7973.png)


其中，**p** 是观察射线与最近物体表面的交点。

根据公式（9.2），我们的新目标是计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_4a356304a16efc.png)(**p**, **v**)。这一计算是第 5.1 节所讨论的着色模型求值的基于物理版本。有时，辐亮度由表面直接发射。更常见的情况是，离开表面的辐亮度源自其他地方，并通过第 9.1 节所述的物理相互作用，被表面反射到观察射线中。本章暂不考虑透明现象（第 5.5 节和第 14.5.2 节）以及全局次表面散射（第 14.6 节）。换言之，我们关注局部反射现象：把照到当前着色点上的光重新向外改变方向。这些现象包括表面反射和局部次表面散射，并且只依赖于入射光方向 **l** 与出射观察方向 **v**。局部反射由**双向反射分布函数**（bidirectional reflectance distribution function，BRDF）量化，记为 f(**l**, **v**)。

在最初的推导 [1277] 中，BRDF 是针对均匀表面定义的。也就是说，假定整个表面上的 BRDF 都相同。然而，现实世界中（以及渲染场景中）的物体，其表面材质属性很少处处均匀。即使一个物体只由一种材质构成，例如一尊银制雕像，也会有划痕、失去光泽的斑点、污渍及其他变化，使其视觉属性随表面位置而改变。严格来说，描述 BRDF 随空间位置变化的函数称为**空间变化 BRDF**（spatially varying BRDF，SVBRDF），或**空间 BRDF**（spatial BRDF，SBRDF）。不过，这种情况在实际应用中非常普遍，因此通常仍使用较短的名称 BRDF，并默认它依赖于表面位置。

入射方向和出射方向各有两个自由度。一种常用的参数化方式使用两个角度：相对于表面法线 **n** 的仰角 θ，以及绕 **n** 的方位角（水平旋转角）φ。一般情况下，BRDF 是四个标量变量的函数。**各向同性 BRDF** 是一种重要的特殊情况。当入射方向和出射方向绕表面法线旋转、同时保持两者之间的相对夹角不变时，这类 BRDF 保持不变。图 9.17 展示了这两种情况下使用的变量。各向同性 BRDF 是三个标量变量的函数，因为只需要一个角度 φ 来表示光与相机之间的相对旋转。这意味着，如果把均匀的各向同性材质放在转台上旋转，那么在光源和相机固定时，它在所有旋转角度下看起来都相同。


![图9.17 BRDF角度定义](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_3_9.17.png)

**图 9.17.** BRDF。方位角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_037c95df242b67.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_3eb1dee49c74ad.png) 是相对于给定切向量 **t** 定义的。用于各向同性 BRDF、取代 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_037c95df242b67.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_3eb1dee49c74ad.png) 的相对方位角 φ，不需要参考切向量。

由于我们忽略了荧光和磷光等现象，因此可以假定：给定波长的入射光，在反射后仍保持同一波长。反射光的量可能随波长而变化，这可以用两种方式之一建模：将波长作为 BRDF 的额外输入变量，或者让 BRDF 返回一个具有光谱分布的值。第一种方式有时用于离线渲染 [660]，而实时渲染始终使用第二种方式。由于实时渲染器用 RGB 三元组表示光谱分布，这实际上就意味着 BRDF 返回一个 RGB 值。

为了计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_4a356304a16efc.png)(**p**, **v**)，我们将 BRDF 纳入**反射方程**：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_06766bfb0007d0.png)


积分号的下标 **l** ∈ Ω 表示，对位于表面上方单位半球内的 **l** 向量进行积分（该半球以表面法线 **n** 为中心方向）。注意，**l** 连续遍历整个入射方向半球；它不是某个特定的“光源方向”。这里的含义是：任何入射方向都可能有与之对应的辐亮度，而且通常确实如此。我们用 d**l** 表示 **l** 周围的微分立体角（立体角见第 8.1.1 节）。

总之，反射方程表明：出射辐亮度等于入射辐亮度、BRDF 与 **n** 和 **l** 的点积三者乘积的积分，积分范围是 Ω 内的 **l**。

为简洁起见，本章后文将从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_b456bbb8033d74.png)()、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_4a356304a16efc.png)() 和反射方程中省略表面点 **p**：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_3836a8a8f23964.png)


计算反射方程时，常用球面坐标 φ 和 θ 对半球进行参数化。在这种参数化下，微分立体角 d**l** 等于 sin ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ba4635370b500e.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_0b43b31b633e82.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ff1db2880685de.png)。使用这种参数化，可以推导出公式（9.4）的球面坐标二重积分形式（回顾一下，**n** · **l** = cos ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ba4635370b500e.png)）：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_cdda0d91748af9.png)


角度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ba4635370b500e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_037c95df242b67.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_dc860a56f09a19.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_3eb1dee49c74ad.png) 如图 9.17 所示。

在某些情况下，使用略有不同的参数化更方便：以仰角的余弦 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_8166b35af25981.png) = cos ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ba4635370b500e.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_83ffb449fca926.png) = cos ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_dc860a56f09a19.png) 为变量，替代角度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ba4635370b500e.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_dc860a56f09a19.png) 本身。在这种参数化下，微分立体角 d**l** 等于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_11fda24a75d22f.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ff1db2880685de.png)。采用（μ, φ）参数化可得到如下积分形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_159eb496a4c477.png)


BRDF 仅在光方向和观察方向都位于表面上方时才有定义。对于光方向位于表面下方的情况，可以将 BRDF 乘以零，或者从一开始就不对这些方向计算 BRDF，以避开这种情况。但是，如果观察方向位于表面下方，也就是点积 **n** · **v** 为负，又该怎么办？理论上，这种情况不应该发生，因为此时表面背向相机，因此不可见。然而，实时应用中常见的插值顶点法线和法线映射，在实践中可能产生这种情况。可以将 **n** · **v** 钳制到 0，或者取其绝对值，以避免对表面下方的观察方向计算 BRDF，但两种方法都可能产生伪影。Frostbite 引擎使用 **n** · **v** 的绝对值，再加上一个很小的数（0.00001），以避免除以零 [960]。另一种可能的方法是“软钳制”：随着 **n** 与 **v** 的夹角增大并超过 90°，使其逐渐趋于零。

物理定律对任何 BRDF 都施加了两个约束。第一个约束是**亥姆霍兹互易性**，意即交换输入角度和输出角度后，函数值保持不变：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_b7987a0c25f152.png)


在实际应用中，渲染所使用的 BRDF 经常违反亥姆霍兹互易性，却不会产生明显伪影；例外是明确要求互易性的离线渲染算法，例如双向路径追踪。不过，互易性仍是判断一个 BRDF 是否具有物理合理性的有用工具。

第二个约束是**能量守恒**：出射能量不能大于入射能量（不计自身发光的表面，它们作为特殊情况处理）。路径追踪等离线渲染算法需要能量守恒来保证收敛。对于实时渲染，并不需要精确的能量守恒，但近似的能量守恒很重要。若使用明显违反能量守恒的 BRDF 渲染表面，该表面就会过亮，从而可能显得不真实。

**方向—半球反射率** R(**l**) 是一个与 BRDF 相关的函数，可用于衡量 BRDF 在多大程度上满足能量守恒。虽然名字有些令人望而生畏，但方向—半球反射率的概念很简单：它衡量来自某个给定方向的光，有多少被反射到了以表面法线为中心的半球中的任意出射方向。实际上，它衡量的是给定入射方向下的能量损失。该函数的输入是入射方向向量 **l**，其定义如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_6ff5ee1e95e372.png)


注意，这里的 **v** 与反射方程中的 **l** 一样，遍历整个半球，并不表示某个单独的观察方向。

另一个类似、但在某种意义上与之相反的函数，是**半球—方向反射率** R(**v**)，同样可以定义为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_2a5224c498043e.png)


如果 BRDF 满足互易性，那么半球—方向反射率和方向—半球反射率相等，二者可以用同一个函数计算。在这两种反射率可以互换使用时，可以用**方向反照率**作为它们的统称。

由于能量守恒，方向—半球反射率 R(**l**) 的值必须始终处于 [0, 1] 范围内。反射率为 0 表示所有入射光均被吸收或以其他方式损失。如果所有光都被反射，反射率就是 1。在大多数情况下，它位于这两个值之间。与 BRDF 一样，R(**l**) 的值随波长而变化，因此在渲染中用 RGB 向量表示。由于每个分量（红、绿、蓝）都限制在 [0, 1] 范围内，R(**l**) 的一个取值可以视为一种简单的颜色。注意，这个限制不适用于 BRDF 的取值。BRDF 是分布函数，如果它描述的分布高度不均匀，那么在某些方向上（例如高光的中心），它可以具有任意高的值。BRDF 满足能量守恒的要求是：对于 **l** 的所有可能取值，R(**l**) 均不大于 1。

最简单的 BRDF 是**朗伯 BRDF**，它对应于第 5.2 节简要讨论的朗伯着色模型。朗伯 BRDF 的值为常数。众所周知、用来体现朗伯着色特征的（**n** · **l**）因子并不是 BRDF 的一部分，而是公式（9.4）的一部分。尽管朗伯 BRDF 很简单，但实时渲染中经常用它来表示局部次表面散射（不过，如第 9.9 节所述，它正在被更准确的模型取代）。朗伯表面的方向—半球反射率也是常数。令 f(**l**, **v**) 为常数并计算公式（9.8），可以得到方向—半球反射率与 BRDF 之间的如下关系：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_12a9168fa7b350.png)


朗伯 BRDF 的恒定反射率通常称为**漫反射颜色** ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ecec7c34ec71c2.png)，或**反照率** ρ。本章为了强调它与次表面散射的联系，将这个量称为**次表面反照率** ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_54cc8781620e9a.png)。第 9.9.1 节将详细讨论次表面反照率。由公式（9.10）可得如下 BRDF：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_bd750864e23cbb.png)


因子 1/π 的出现，是因为将余弦因子在半球上积分所得的值为 π。BRDF 中经常可以看到这样的因子。

理解 BRDF 的一种方式，是固定输入方向并将其可视化，见图 9.18。对于给定的入射光方向，显示所有出射方向上的 BRDF 值。交点周围的球状部分是漫反射分量，因为出射辐亮度有相等的机会反射到任何方向。椭球状部分是**镜面反射瓣**。自然地，这些瓣位于入射光所对应的反射方向，瓣的厚度对应于反射的模糊程度。根据互易性原理，这些相同的可视化也可以理解为：各个不同的入射光方向对某个单一出射方向分别贡献了多少。


![图9.18 BRDF示例](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_3_9.18.png)

**图 9.18.** BRDF 示例。每幅图中从右侧伸来的绿色实线表示入射光方向，绿白相间的虚线表示理想反射方向。上排左图是朗伯 BRDF（一个简单的半球）；中图是在朗伯项上加入 Blinn–Phong 高光；右图是 Cook–Torrance BRDF [285, 1779]。注意，镜面高光并不是在反射方向上最强。下排左图是 Ward 各向异性模型的近景，在这种情况下，其效果是使镜面反射瓣倾斜；中图是 Hapke/Lommel–Seeliger“月球表面”BRDF [664]，它具有很强的逆向反射；右图展示 Lommel–Seeliger 散射，在这种散射中，积尘表面将光散射到掠射角方向。（图片由 Szymon Rusinkiewicz 提供，取自他的“bv”BRDF 浏览器。）

> 译注：原书公式（9.5）、（9.6）的入射辐亮度写作 L，未带下标 i，此处依原页保留。原书将相对于法线的 θ 称为 elevation（仰角），其几何定义以图 9.17 为准；改用 μ = cos θ 时，原文 d**l** = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_11fda24a75d22f.png) ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_03_ff1db2880685de.png) 表示积分使用的正测度，变量代换的负号通过反转积分上下限处理。


## 9.4 光照

来源：原书第315—316页（PDF第336—337页）。本节从“9.4 Illumination”标题起，至“9.5 Fresnel Reflectance”标题前止。

反射方程（式9.4）中的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_53dd1f51aceccb.png)（入射辐亮度）项表示从场景其他部分照射到当前着色表面点的光。全局光照算法通过模拟光在整个场景中的传播和反射来计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_53dd1f51aceccb.png)。这些算法使用渲染方程[846]，反射方程是它的一个特例。第11章将讨论全局光照。在本章和下一章中，我们关注局部光照：它利用反射方程，在每个表面点局部计算着色。在局部光照算法中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_53dd1f51aceccb.png) 是给定的，不需要计算。

在真实场景中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_53dd1f51aceccb.png) 包含来自所有方向的非零辐亮度，这些光可能直接由光源发出，也可能由其他表面反射而来。与第5.2节讨论的方向光源和点状光源不同，现实世界中的光源都是占据非零立体角的面光源。本章使用一种受限形式的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_53dd1f51aceccb.png)，其中只包含方向光源和点状光源，把更一般的光照环境留到第10章讨论。这种限制使讨论能够更加集中。

尽管点状光源和方向光源是非物理的抽象，但可以将它们推导为物理光源的近似。这种推导很重要，因为它使我们能够在清楚了解所引入误差的前提下，将这些光源纳入基于物理的渲染框架。

取一个位于远处的小面光源，并将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_de573840d41b7c.png) 定义为指向其中心的向量。我们还将光源颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_c53c80de5096d9.png) 定义为一个正对光源的白色朗伯表面的反射辐亮度，此时 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_2000bb380d101b.png)。对于内容创作而言，这种定义很直观，因为光源的颜色直接对应于它的视觉效果。

根据这些定义，方向光源可以推导为如下极限情形：保持 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_c53c80de5096d9.png) 的值不变，同时将面光源的尺寸缩小到零[758]。在这种情况下，反射方程（式9.4）中的积分简化为一次 BRDF 求值，计算开销显著降低：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_8bc416e89ed407.png)


点积 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_e483cf8b2005ba.png) 通常会将负值钳制为零，从而方便地跳过位于表面下方的光源所产生的贡献：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_50977f699bcbcb.png)


注意第1.2节引入的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_b9875aa6a4d8ef.png) 记号，它表示将负值钳制为零。

点状光源也可以用类似方式处理。仅有的区别在于：面光源不必位于远处，而且 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_c53c80de5096d9.png) 按照到光源距离的平方反比衰减，如式5.11（第111页）所示。如果存在多个光源，就多次计算式9.12，并将结果相加：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_6db4a72fb8a3b4.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_052938a16d0fd4.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_04_7edee064c7add9.png) 分别为第 i 个光源的方向和颜色。注意它与式5.6（第109页）的相似之处。

式9.14中的 π 因子会与 BRDF 中经常出现的 1/π 因子（例如式9.11）相消。这种抵消将除法运算移出了着色器，也让着色方程更容易阅读。不过，在将学术论文中的 BRDF 调整后用于实时着色方程时，必须小心。通常，BRDF 在使用前需要乘以 π。


## 9.5 菲涅耳反射率

来源：原书第 316—327 页（PDF 物理页 337—348）。范围从 9.5 节标题起，至 9.6 节标题前，包含全部下级小节、图 9.19—9.24、表 9.1—9.4 及公式（9.15）—（9.20）。

在 9.1 节中，我们从总体上讨论了光与物质的相互作用。在 9.3 节中，我们介绍了用数学表达这些相互作用的基本工具：BRDF 和反射率方程。现在，我们可以开始深入研究具体的现象，将其量化，以便用于着色模型。首先讨论平坦表面上的反射；这个主题最早在 9.1.3 节中介绍过。

物体表面是周围介质（通常为空气）与物体自身物质之间的界面。光与两种物质之间平面界面的相互作用遵循奥古斯丁·让·菲涅耳（Augustin-Jean Fresnel，1788—1827）提出的菲涅耳方程（Fresnel 的发音为 freh-nel）。菲涅耳方程要求界面平坦，并满足几何光学的假设。换句话说，假定表面不存在尺寸介于 1 个光波长和 100 个光波长之间的不规则起伏。比这一范围更小的起伏不会影响光，而更大的起伏实际上会使表面倾斜，却不影响其局部平坦性。

入射到平坦表面的光分为反射和折射两部分。反射光的方向（用向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d02133e9bb8f1f.png) 表示）与表面法线 **n** 的夹角，等于入射方向 **l** 与法线的夹角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)。反射向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d02133e9bb8f1f.png) 可以由 **n** 和 **l** 计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_83183130c3499e.png)


见图 9.19。反射光的量（占入射光的比例）由菲涅耳反射率 F 描述，它取决于入射角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)。


![图9.19 平面表面上的反射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_9.19.png)

**图 9.19** 平面表面上的反射。光照向量 **l** 关于法线 **n** 反射，生成 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d02133e9bb8f1f.png)。首先，将 **l** 投影到 **n** 上，得到法线的一个缩放版本：( **n** · **l** ) **n**。然后将 **l** 取反，再加上投影向量的两倍，就得到反射向量。

正如 9.1.3 节所讨论的，反射和折射受到平面两侧物质折射率的影响。这里继续沿用此前的记号。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) 是界面“上方”物质的折射率，入射光和反射光在其中传播；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png) 是界面“下方”物质的折射率，折射光在其中传播。

菲涅耳方程描述 F 对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png) 的依赖关系。由于方程本身有些复杂，这里不直接列出，而是介绍其重要特征。

### 9.5.1 外反射

外反射指 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) < ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png) 的情况。也就是说，光来自表面折射率较低的一侧。这一侧通常是空气，其折射率约为 1.003。为简便起见，我们假定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) = 1。相反的过程，即从物体向空气的传播，称为内反射，将在后面的 9.5.3 节讨论。

> 译注：此处空气折射率的 1.003 按原书保留，疑为小数位笔误；通常可见光下空气折射率约为 1.0003。这不影响原文随后采用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) = 1 的近似。

对于给定物质，可以把菲涅耳方程理解为定义了一个仅依赖入射光角度的反射率函数 F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png))。原则上，F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 的值随可见光谱连续变化。为了用于渲染，将其值视为 RGB 向量。函数 F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 有以下特征：

- 当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) = 0°、光垂直于表面（**l** = **n**）时，F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 的值是该物质的一项固有属性。这个值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 可以看作该物质的特征镜面反射颜色。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) = 0° 的情况称为法向入射。
- 随着 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 增大、光以越来越接近掠射的角度照到表面，F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 的值趋于增大，并在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) = 90° 时对所有频率都达到 1（白色）。

图 9.20 用几种不同的可视化方式展示了几种物质的 F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 函数。这些曲线具有很强的非线性：直到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 约为 75° 时，它们都几乎没有变化，随后迅速上升至 1。从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 到 1 的增大过程大多是单调的，不过某些物质（例如图 9.20 中的铝）在变成白色之前会略微下降。


![图9.20 三种物质的菲涅耳外反射率](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_9.20.png)

**图 9.20** 三种物质的菲涅耳外反射率 F：从左至右为玻璃、铜和铝。第一行是 F 随波长和入射角变化的三维曲面图。第二行将每个入射角对应的 F 光谱值转换为 RGB，并分别绘出各颜色通道的曲线。由于玻璃的菲涅耳反射率无色，它的各条曲线重合。第三行以入射角的正弦为横轴绘制 R、G、B 曲线，以计入图 9.21 所示的透视缩短效应。最下方的色带使用相同的横轴，以颜色显示 RGB 值。

对于镜面反射，出射角（或观察角）与入射角相同。这意味着相对于入射光处于掠射角度、即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 接近 90° 的表面，相对于眼睛也处于掠视角度。因此，反射率的增加主要出现在物体边缘。此外，从相机视角看，反射率增加最显著的表面部分发生了透视缩短，所以它们只占据相对较少的像素。为了使菲涅耳曲线各部分的展示比例与其视觉显著程度相称，图 9.22 和图 9.20 下半部分的菲涅耳反射率曲线及色带使用 sin(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 作为横轴，而不是直接使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)。图 9.21 说明了为什么 sin(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 适合作为这一目的下的坐标轴。


![图9.21 表面倾斜造成的透视缩短](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_9.21.png)

**图 9.21** 背离眼睛倾斜的表面会发生透视缩短。这种缩短与按照 **v** 和 **n** 夹角的正弦投影表面点相一致（对于镜面反射，这个角度就是入射角）。因此，图 9.20 和图 9.22 都以入射角的正弦为横轴绘制菲涅耳反射率。

从这里开始，我们通常用 F(**n**, **l**) 代替 F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 表示菲涅耳函数，以强调所涉及的向量。回顾一下，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 是向量 **n** 和 **l** 之间的夹角。当菲涅耳函数作为 BRDF 的一部分时，往往会用另一个向量替代表面法线 **n**。详情见 9.8 节。

在渲染文献中，掠射角下反射率增大的现象常被称为菲涅耳效应（在其他领域，这个术语有不同含义，与无线电波的传播有关）。你可以用一个小实验亲自观察菲涅耳效应。拿一部智能手机，坐在计算机显示器等明亮区域前方。不要打开手机，先将手机靠近胸前，低头看它，稍微调整角度，使屏幕反射显示器。此时手机屏幕上应当出现一个相对较弱的显示器反射像。这是因为玻璃的法向入射反射率很低。现在将手机抬起，使它大致位于眼睛与显示器之间，再次调整屏幕角度以反射显示器。这时，手机屏幕上的显示器反射像应当几乎与显示器本身一样明亮。


![图9.22 Schlick近似与完整菲涅耳方程的比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_9.22.png)

**图 9.22** Schlick 的菲涅耳反射率近似与六种物质外反射正确值的比较。上排的三种物质与图 9.20 相同，从左至右为玻璃、铜和铝。下排三种物质为铬、铁和锌。每种物质各有一组 RGB 曲线，实线表示完整菲涅耳方程，点线表示 Schlick 近似。每幅曲线图下面的上方色带显示完整菲涅耳方程的结果，下方色带显示 Schlick 近似的结果。

除了复杂之外，菲涅耳方程还具有其他使其难以直接用于渲染的性质。它们需要在可见光谱上采样的折射率值，而这些值可能是复数。图 9.20 的曲线提示了一种基于特征镜面反射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 的更简单方法。Schlick [1568] 给出了以下菲涅耳反射率近似：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_7cc5b713eb263a.png)


这个函数在白色和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 之间进行 RGB 插值。尽管形式简单，该近似仍相当准确。

图 9.22 中有几种物质偏离 Schlick 曲线，在变白之前出现明显的“下凹”。实际上，选择下排物质正是因为它们与 Schlick 近似的偏差特别大。即便如此，由图中各曲线下方的色带可见，最终产生的误差仍很细微。在少数必须精确再现此类材质行为的场合，可以使用 Gulbrandsen [623] 给出的另一种近似。对于金属，这种近似能够很好地匹配完整菲涅耳方程，但计算开销比 Schlick 近似更大。一个更简单的选择是修改 Schlick 近似，允许最后一项使用 5 以外的幂次（如公式 9.18）。这会改变在 90° 处过渡到白色的“陡峭程度”，从而可能获得更好的匹配。Lagarde [959] 总结了菲涅耳方程及其几种近似。

使用 Schlick 近似时，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 是控制菲涅耳反射率的唯一参数。这很方便，因为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 的有效范围明确，为 [0, 1]；它可以通过标准颜色选择界面轻松设置，也可以使用面向颜色的纹理格式进行纹理化。此外，许多现实材质都有可供参考的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值。折射率也可以用于计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png)。

通常假定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) = 1，这是空气折射率的一个很好的近似，并用 n 代替 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png) 表示物体的折射率。这样简化后得到：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_a8e2b299a3047a.png)


即使折射率为复数（例如金属的折射率），只要取所得（复数）结果的模，这个公式仍然适用。如果折射率在可见光谱范围内变化明显，为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 计算准确的 RGB 值就需要先在密集采样的各个波长处计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png)，再用 8.1.3 节介绍的方法将得到的光谱向量转换为 RGB 值。

一些应用 [732, 947] 使用更一般的 Schlick 近似形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_dac8fe1ed6b9cc.png)


这既能控制菲涅耳曲线在 90° 处过渡到的颜色，也能控制过渡的“陡峭程度”。使用这种更一般的形式通常是为了增加艺术控制能力，但在某些情况下，它也有助于匹配物理现实。如前所述，修改幂次能让某些材质得到更好的拟合。此外，将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_edf7b8e061030b.png) 设为白色以外的颜色，有助于匹配菲涅耳方程无法很好描述的材质，例如覆盖着细尘的表面，其中尘粒尺寸与单个光波长相当。

### 9.5.2 典型的菲涅耳反射率值

按光学性质，物质主要分为三类：电介质，即绝缘体；金属，即导体；以及半导体，其性质介于电介质和金属之间。

#### 电介质的菲涅耳反射率值

日常生活中遇到的大多数材质都是电介质，例如玻璃、皮肤、木材、毛发、皮革、塑料、石材和混凝土。水也是电介质。这一点可能令人惊讶，因为日常生活中我们知道水能够导电，但这种导电性来自各种杂质。电介质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值相当低，通常不超过 0.06。法向入射时较低的反射率，使菲涅耳效应在电介质上尤其明显。电介质的光学性质在可见光谱中通常变化不大，因此反射率值是无色的。表 9.1 列出了几种常见电介质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值。这里的值是标量而不是 RGB，因为这些材质的 RGB 通道之间没有明显差异。为使用方便，表 9.1 同时列出了线性值和经过 sRGB 传递函数编码的 8 位值（通常在纹理绘制软件中使用的形式）。

| 电介质 | 线性值 | 纹理值 | 备注 |
| --- | --- | --- | --- |
| 水 | 0.02 | 39 | |
| 生物组织 | 0.02—0.04 | 39—56 | 含水较多的组织接近下限，较干燥的组织更高 |
| 皮肤 | 0.028 | 47 | |
| 眼睛 | 0.025 | 44 | 干燥角膜（泪液的值与水相近） |
| 毛发 | 0.046 | 61 | |
| 牙齿 | 0.058 | 68 | |
| 织物 | 0.04—0.056 | 56—67 | 聚酯最高，其他大多数低于 0.05 |
| 石材 | 0.035—0.056 | 53—67 | 石材中最常见矿物的数值 |
| 塑料、玻璃 | 0.04—0.05 | 56—63 | 不含水晶玻璃 |
| 水晶玻璃 | 0.05—0.07 | 63—75 | |
| 宝石 | 0.05—0.08 | 63—80 | 不含钻石及仿钻材料 |
| 类钻石材料 | 0.13—0.2 | 101—124 | 钻石及仿钻材料（例如立方氧化锆、莫桑石） |

![表9.1 原表及颜色色块，行序对应上方中文表](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_table9.1.png)

**表 9.1** 各种电介质发生外反射时的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值。每个值分别以线性数值、纹理值（经非线性编码的 8 位无符号整数）和颜色色块表示。如果给出的是数值范围，色块取该范围的中间值。请记住，这些是镜面反射颜色。例如，宝石常有鲜艳的颜色，但那些颜色来自物质内部的吸收，与其菲涅耳反射率无关。原表“Color”列的色块保留在上图中，与中文表逐行对应。

其他电介质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值可以根据表中的类似物质推断。对于未知电介质，0.04 是一个合理的默认值，与大多数常见材质相差不远。

光一旦透入电介质，就可能进一步发生散射或被吸收。9.9 节将更详细地讨论这一过程的模型。如果材质透明，光会继续传播，直到“从内部”碰到物体表面，详情见 9.5.3 节。

#### 金属的菲涅耳反射率值

金属的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值很高，几乎总是 0.5 或更大。有些金属的光学性质随可见光谱变化，因此其反射率带有颜色。表 9.2 列出了几种金属的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值。

| 金属 | 线性 RGB 值 | 纹理 RGB 值 |
| --- | --- | --- |
| 钛 | 0.542, 0.497, 0.449 | 194, 187, 179 |
| 铬 | 0.549, 0.556, 0.554 | 196, 197, 196 |
| 铁 | 0.562, 0.565, 0.578 | 198, 198, 200 |
| 镍 | 0.660, 0.609, 0.526 | 212, 205, 192 |
| 铂 | 0.673, 0.637, 0.585 | 214, 209, 201 |
| 铜 | 0.955, 0.638, 0.538 | 250, 209, 194 |
| 钯 | 0.733, 0.697, 0.652 | 222, 217, 211 |
| 汞 | 0.781, 0.780, 0.778 | 229, 228, 228 |
| 黄铜（C260） | 0.910, 0.778, 0.423 | 245, 228, 174 |
| 锌 | 0.664, 0.824, 0.850 | 213, 234, 237 |
| 金 | 1.000, 0.782, 0.344 | 255, 229, 158 |
| 铝 | 0.913, 0.922, 0.924 | 245, 246, 246 |
| 银 | 0.972, 0.960, 0.915 | 252, 250, 245 |

![表9.2 原表及颜色色块，行序对应上方中文表](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_table9.2.png)

**表 9.2** 各种金属（以及一种合金）发生外反射时的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值，按明度递增排序。金的实际红色通道值略微超出 sRGB 色域。这里显示的是钳制后的值。原表颜色列的色块保留在上图中。

与表 9.1 类似，表 9.2 同时提供线性值和用于纹理的 8 位 sRGB 编码值。不过，这里给出的是 RGB 值，因为许多金属的菲涅耳反射率带有颜色。这些 RGB 值采用 sRGB（以及 Rec. 709）的基色和白点定义。金的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值有些特殊。它的颜色最强烈，红色通道值略高于 1（刚刚超出 sRGB/Rec. 709 色域），蓝色通道值则特别低（是表 9.2 中唯一明显低于 0.5 的值）。金也是最明亮的金属之一，从它在按明度递增排列的表格中的位置就可以看出。金既明亮又具有强烈色彩的反射，可能有助于解释它在历史上独特的文化和经济意义。

回顾一下，金属会立即吸收所有透入的光，因此既没有次表面散射，也不透明。金属所有可见颜色都来自 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png)。

#### 半导体的菲涅耳反射率值

正如预期，半导体的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值介于最明亮的电介质和最暗的金属之间，如表 9.3 所示。实际中很少需要渲染此类物质，因为大多数渲染场景中不会遍地都是晶体硅块。就实际应用而言，除非有意模拟特殊或不现实的材质，否则应避免使用 0.2 到 0.45 之间的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值。

| 物质 | 线性 RGB 值 | 纹理 RGB 值 |
| --- | --- | --- |
| 钻石 | 0.171, 0.172, 0.176 | 115, 115, 116 |
| 硅 | 0.345, 0.369, 0.426 | 159, 164, 174 |
| 钛 | 0.542, 0.497, 0.449 | 194, 187, 179 |

![表9.3 原表及颜色色块，行序对应上方中文表](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_table9.3.png)

**表 9.3** 一种代表性半导体（晶体形态的硅）的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值，与明亮电介质（钻石）和较暗金属（钛）的比较。原表颜色列的色块保留在上图中。

#### 水中的菲涅耳反射率值

在讨论外反射率时，我们一直假定被渲染的表面被空气包围。如果不是如此，反射率就会改变，因为它取决于界面两侧折射率的比值。如果不能再假定 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) = 1，就需要用相对折射率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png) 替换公式 9.17 中的 n。由此得到以下更一般的公式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_e13ab9a9eba0ad.png)


> 译注：原书此处写相对折射率为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png)，按原文保留；若承接前文“n 代表物体相对于空气的折射率”的定义，常用写法是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png)。不过，将这两个互为倒数的比值代入公式 9.17，平方后都会得到公式 9.19。

最常遇到的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) ≠ 1 情况，大概就是渲染水下场景。由于水的折射率约为空气的 1.33 倍，水下的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值有所不同。这种效应对电介质的影响比对金属更大，见表 9.4。

| 物质 | 线性值（有三个数时为 RGB） | 纹理值（有三个数时为 RGB） |
| --- | --- | --- |
| 皮肤（空气中） | 0.028 | 47 |
| 皮肤（水中） | 0.0007 | 2 |
| 肖特 K7 玻璃（空气中） | 0.042 | 58 |
| 肖特 K7 玻璃（水中） | 0.004 | 13 |
| 钻石（空气中） | 0.172 | 115 |
| 钻石（水中） | 0.084 | 82 |
| 铁（空气中） | 0.562, 0.565, 0.578 | 198, 198, 200 |
| 铁（水中） | 0.470, 0.475, 0.492 | 182, 183, 186 |
| 金（空气中） | 1.000, 0.782, 0.344 | 255, 229, 158 |
| 金（水中） | 1.000, 0.747, 0.261 | 255, 224, 140 |
| 银（空气中） | 0.972, 0.960, 0.915 | 252, 250, 245 |
| 银（水中） | 0.964, 0.950, 0.899 | 251, 249, 243 |

![表9.4 原表及颜色色块，行序对应上方中文表](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_table9.4.png)

**表 9.4** 各种物质在空气中和水中的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值比较。由公式 9.19 可以预料到，折射率接近水的电介质受到的影响最大。相比之下，金属几乎不受影响。原表颜色列的色块保留在上图中。

#### 菲涅耳值的参数化

一种常用的参数化方法将镜面反射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 和漫反射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_13f96b5049ec4e.png) 结合起来（漫反射颜色将在 9.9 节进一步讨论）。这种方法利用了以下观察：金属没有漫反射颜色，而电介质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 只能在有限范围内取值。它包含一个 RGB 表面颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_78c02c2f5ab04c.png) 和一个标量参数 m，后者称为“metallic”或“metalness”（金属度）。如果 m = 1，则将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 设为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_78c02c2f5ab04c.png)，并将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_13f96b5049ec4e.png) 设为黑色。如果 m = 0，则将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 设为电介质的某个值（可以是常量，也可以由一个附加参数控制），并将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_13f96b5049ec4e.png) 设为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_78c02c2f5ab04c.png)。

“金属度”参数最早出现在布朗大学使用的一个早期着色模型中 [1713]；当前形式的参数化则最早由皮克斯用于电影《机器人总动员》（Wall-E）[1669]。对于从《无敌破坏王》（Wreck-It Ralph）开始用于迪士尼动画电影的 Disney 原则性着色模型，Burley 增加了一个标量“specular”（镜面反射）参数，用来在有限范围内控制电介质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) [214]。Unreal Engine 使用这种形式的参数化 [861]，而 Frostbite 引擎采用了略有不同的形式，允许电介质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 取更大的范围 [960]。游戏《使命召唤：无限战争》（Call of Duty: Infinite Warfare）使用了一个变体，将金属度和镜面反射参数打包到单个数值中 [384]，以节省内存。

对于采用金属度参数化、而不是直接使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_13f96b5049ec4e.png) 的渲染应用，其动机包括便于用户使用，以及节省纹理或 G 缓冲区存储。在《使命召唤：无限战争》中，这种参数化有一种不寻常的用法：美术人员绘制 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_13f96b5049ec4e.png) 纹理，随后自动转换为金属度参数化，将其作为一种压缩方法。

使用金属度也有一些缺点。它无法表达某些类型的材质，例如具有有色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值的涂层电介质。在金属与电介质的边界处还可能出现伪影 [960, 1163]。

一些实时应用还使用另一种参数化技巧，利用的事实是：除特殊的减反射涂层外，没有材质的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值低于 0.02。该技巧用来抑制代表空腔或空洞的表面区域中的镜面高光。它不使用单独的镜面反射遮蔽纹理，而用低于 0.02 的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 值来“关闭”菲涅耳边缘增亮。这项技术最早由 Schüler [1586] 提出，并用于 Unreal [861] 和 Frostbite [960] 引擎。

### 9.5.3 内反射

虽然渲染中更常遇到外反射，但内反射有时也很重要。内反射发生在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) > ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png) 时。换句话说，当光在透明物体内部传播，并“从内部”碰到物体表面时，就会发生内反射。见图 9.23。


![图9.23 平面表面上的内反射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_9.23.png)

**图 9.23** 平面表面上的内反射，其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_3a00723a6f5471.png) > ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d13080baeee9f5.png)。

斯涅尔定律表明，对于内反射，sin ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png) > sin ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)。由于这两个角度都在 0° 与 90° 之间，这一关系还意味着 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png) > ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)，如图 9.23 所示。对于外反射，情况恰好相反，可以与第 302 页的图 9.9 比较。这一区别是理解内反射与外反射差异的关键。对于外反射，sin ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 在 0 到 1 之间的每一个可能值，都有一个有效的、更小的 sin ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png) 值与之对应。内反射则并非如此。当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 大于某个临界角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_cc2d6ed4dfeee1.png) 时，斯涅尔定律意味着 sin ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png) > 1，这是不可能的。实际情况是，不存在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png)。当 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) > ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_cc2d6ed4dfeee1.png) 时，不发生透射，所有入射光都被反射。这一现象称为全内反射。

> 译注：原书“这两个值都在 0° 与 90° 之间”指的是角度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)，而非前一句的正弦值；译文明确了所指。

菲涅耳方程具有对称性：交换入射向量和透射向量，反射率保持不变。结合斯涅尔定律，这种对称性意味着内反射的 F(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png)) 曲线类似于外反射曲线的“压缩”版本。两种情况的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 相同，而内反射曲线在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_cc2d6ed4dfeee1.png) 处就达到完全反射，不必等到 90°。图 9.24 展示了这一点，同时也表明，平均而言内反射的反射率更高。例如，这就是水下看到的气泡呈现高度反光、银亮外观的原因。


![图9.24 玻璃与空气界面的内反射和外反射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_5_9.24.png)

**图 9.24** 玻璃—空气界面上内反射率和外反射率曲线的比较。内反射率曲线在临界角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_cc2d6ed4dfeee1.png) 处达到 1.0。

内反射只发生在电介质中，因为金属和半导体会迅速吸收在其内部传播的光 [285, 286]。由于电介质的折射率是实数，从折射率或 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_d8e52c5a12790b.png) 计算临界角很直接：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_a7f4a133bb2115.png)


公式 9.16 所示的 Schlick 近似适用于外反射。将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 换成透射角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png)，也可以把它用于内反射。如果已经计算了透射方向向量 **t**（例如为了渲染折射，见 14.5.2 节），就可以用它求 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png)。否则，可以利用斯涅尔定律根据 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ba4635370b500e.png) 计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_05_ca26290f1bb6de.png)，但这样开销较大，而且需要折射率，而折射率可能并不可用。


## 9.6 微观几何

来源：原书第 327—331 页（PDF 第 348—352 页）。本节从“9.6 Microgeometry”开始，到“9.7 Microfacet Theory”标题之前结束，包含图 9.25—9.30。

正如前面第 9.1.3 节所讨论的，对于远小于一个像素的表面凹凸，显式建模并不可行，因此 BRDF 改为用统计方法对它们的综合效应建模。目前，我们仍限定在几何光学的范围内：它假定这些凹凸要么小于光的波长（因而不会影响光的行为），要么远大于光的波长。处于“波动光学范围”内的凹凸（尺寸约为 1—100 个波长）所产生的效应，将在第 9.11 节讨论。

每个可见的表面点都包含许多微表面法线，它们使反射光射向不同方向。由于各个微表面的朝向具有一定随机性，用统计分布对其建模是合理的。对于大多数表面，微观几何表面法线的分布是连续的，并在宏观表面法线方向上具有一个显著的峰值。这种分布的“集中程度”由表面粗糙度决定。表面越粗糙，微观几何法线就越“分散”。

微观尺度粗糙度增大所带来的可见效果，是反射的环境细节变得更加模糊。对于小而明亮的光源，这种模糊会使镜面高光变得更宽、更暗。较粗糙表面的高光更暗，是因为光能被分散到了更宽的方向锥内。在第 305 页图 9.12 的照片中可以看到这一现象。


![图 9.25 从可见细节到微观尺度的逐渐过渡](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_6_9.25.png)

**图 9.25** 从可见细节到微观尺度的逐渐过渡。图像的顺序是先从左到右看上排，再从左到右看下排。表面形状与光照保持不变，只有表面细节的尺度发生变化。

图 9.25 展示了各个微观尺度表面细节的反射如何汇总成可见的反射效果。这组图像展示了一个由单个光源照亮的弯曲表面，其凸起的尺度不断缩小，直到最后一幅图中的凸起远小于单个像素。众多小高光中的统计规律，最终成为汇总后高光形状中的细节。例如，外围各个凸起上的高光相对稀疏，最终表现为汇总后的高光在远离中心处相对较暗。

对于大多数表面，微观尺度表面法线的分布是各向同性的，这意味着它具有旋转对称性，没有任何内在的方向性。另一些表面的微观尺度结构则是各向异性的。这类表面具有各向异性的表面法线分布，从而使反射和高光产生方向性的模糊。见图 9.26。

有些表面的微观几何具有高度组织化的结构，由此形成各种微观尺度法线分布和表面外观。织物就是一个常见的例子：天鹅绒和缎子独特的外观，正是由它们的微观几何结构造成的 [78]。织物模型将在第 9.10 节讨论。

虽然多种表面法线是微观几何影响反射率的主要方式，但其他效应也可能很重要。阴影遮蔽（shadowing）指的是微观尺度的表面细节对光源的遮挡，如图 9.27 左侧所示。视线遮蔽（masking）则是某些微表面从相机的视角遮住其他微表面，如图中央所示。


![图 9.26 各向异性表面与显微照片](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_6_9.26.png)

**图 9.26** 左侧是一个各向异性表面（拉丝金属）。请注意反射的方向性模糊。右侧是展示类似表面的显微照片，请注意细节的方向性。（显微照片由康奈尔大学计算机图形学项目提供。）


![图 9.27 微观尺度结构的几何效应](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_6_9.27.png)

**图 9.27** 微观尺度结构的几何效应。左侧的黑色虚线箭头表示一个被其他微观几何投下阴影的区域，即光线被遮挡而无法到达的区域。中央的红色虚线箭头表示一个被其他微观几何遮住的区域，即从观察方向不可见的区域。右侧展示了光在微观尺度结构之间的相互反射。

如果微观几何的高度与表面法线之间存在相关性，那么阴影遮蔽和视线遮蔽便能改变实际起作用的法线分布。例如，设想一个表面，其凸起部分已因风化或其他过程而变得光滑，而较低的部分仍然粗糙。在掠射角度下，表面较低的部分往往会受到阴影遮蔽或视线遮蔽，结果是表面实际表现得更加光滑。见图 9.28。

对于所有类型的表面，随着入射方向与法线之间的夹角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_06_ba4635370b500e.png) 增大，表面凹凸的可见尺寸都会减小。在极端的掠射角度下，这种效应可能使所观察到的凹凸尺寸减小到光的波长以下，于是就光的响应而言，它们便“消失”了。这两种效应与菲涅耳效应共同作用，使得当观察角度和光照角度趋近于 90° 时，表面显得反射率很高，并具有镜面般的外观 [79, 1873, 1874]。

你可以亲自验证这一点。把一张没有光泽的纸卷成长筒。不要透过筒孔看，而是把眼睛稍微抬高一点，使视线沿着纸筒的长度方向望去。将纸筒指向明亮的窗户或计算机屏幕。当观察方向几乎平行于纸面时，你会在纸面上看到窗户或屏幕清晰的反射像。这个角度必须极其接近 90°，才能看到该效果。


![图 9.28 高度与法线相关时的角度相关粗糙度](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_6_9.28.png)

**图 9.28** 图示微观几何的高度与表面法线之间具有很强的相关性：凸起区域光滑，较低区域粗糙。在上图中，表面受到来自接近宏观表面法线方向的光照。在这个角度下，许多入射光线能够进入粗糙的凹坑，因此许多光线被散射到不同方向。在下图中，表面受到掠射角度的光照。阴影遮蔽挡住了大部分凹坑，因此只有少数光线射入其中，而大多数光线都从表面的光滑部分反射出去。在这种情况下，表观粗糙度强烈依赖于光照角度。

被微观尺度表面细节挡住的光并不会消失。它会发生反射，而且可能反射到其他微观几何上。光可能以这种方式经历多次反弹后才到达眼睛。这类相互反射如图 9.27 右侧所示。由于光在每次反弹时都会按菲涅耳反射率衰减，因此相互反射在电介质中通常并不明显。在金属中，由于不存在次表面散射，多次反弹的反射就是所有可见漫反射的来源。有色金属的多次反弹反射，比初次反射的颜色更浓，因为这是光与表面多次相互作用的结果。

到目前为止，我们讨论的是微观几何对镜面反射率，也就是表面反射率的影响。在某些情况下，微观尺度表面细节也会影响次表面反射率。如果微观几何凹凸的尺寸大于次表面散射的距离，那么阴影遮蔽和视线遮蔽就可能造成回归反射效应，即光倾向于沿入射来向反射回去。之所以出现这种效应，是因为当观察方向与光照方向差别很大时，阴影遮蔽和视线遮蔽会遮住受光区域。见图 9.29。回归反射往往会使粗糙表面显得平坦。见图 9.30。


![图 9.29 微观尺度粗糙度引起的回归反射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_6_9.29.png)

**图 9.29** 微观尺度粗糙度引起的回归反射。两幅图都展示了一个菲涅耳反射率较低、散射反照率较高的粗糙表面，因此次表面反射在视觉上很重要。左图中，观察方向与光照方向相近。微观几何中被明亮照射的部分，也是最容易被看见的部分，因此外观明亮。右图中，观察方向与光照方向差别很大。此时，明亮的受光区域被挡住而不可见，可见区域则处于阴影中，因此外观较暗。


![图 9.30 具有非朗伯回归反射行为的物体照片](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_6_9.30.png)

**图 9.30** 两个物体的照片，它们由于微观尺度表面粗糙度而呈现出非朗伯的回归反射行为。（右侧照片由 Peter-Pike Sloan 提供。）


## 9.7 微表面理论

来源：原书第 331—336 页（PDF 第 352—357 页）；范围从“9.7 Microfacet Theory”标题起，到“9.8 BRDF Models for Surface Reflection”标题前止。本节无下级小节。

许多 BRDF 模型都以一种对微观几何如何影响反射率的数学分析为基础，这种分析称为**微表面理论**（microfacet theory）。这一工具最早由光学领域的研究者提出 [124]。Blinn 于 1977 年将其引入计算机图形学 [159]，Cook 和 Torrance 又于 1981 年将其引入该领域 [285]。这一理论把微观几何建模为一组**微面元**（microfacets）。

每个这样的微小面元都是平坦的，具有唯一的微面元法线 𝐦。各微面元分别按照微观 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_c6bdae054ef589.png) 反射光，所有微面元的反射贡献相加，就得到整个表面的 BRDF。通常会把每个微面元设为一个理想的菲涅耳镜面，由此得到用于建模表面反射的镜面微表面 BRDF。不过，也可以作出其他选择。漫反射微观 BRDF 已被用于建立若干局部次表面散射模型 [574, 657, 709, 1198, 1337]。衍射微观 BRDF 则被用于建立一种同时结合几何光学与波动光学效应的着色模型 [763]。


![图 9.31 微观表面投影面积的关系](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_7_9.31.png)

**图 9.31** 微观表面的侧视图。左图表明，对 D(𝐦)(𝐧·𝐦) 积分，也就是对微面元投影到宏观表面平面上的面积积分，会得到宏观表面的面积（在这个侧视图中表现为长度），按约定该值为 1。右图中，对 D(𝐦)(𝐯·𝐦) 积分，也就是对微面元投影到垂直于 𝐯 的平面上的面积积分，会得到宏观表面在该平面上的投影，即 cos ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_dc860a56f09a19.png) 或 (𝐯·𝐧)。当多个微面元的投影相互重叠时，背向微面元的负投影面积会抵消“多余”的正向微面元投影面积。（根据 Matej Drame 的插图改编。）

微表面模型的一个重要属性，是微面元法线 𝐦 的统计分布。该分布由表面的**法线分布函数**（normal distribution function，NDF）定义。一些参考文献使用 distribution of normals（法线的分布）这一说法，以避免与高斯正态分布混淆。我们在公式中用 D(𝐦) 表示 NDF。

NDF D(𝐦) 是按微观几何表面积统计的微面元表面法线分布 [708]。在微面元法线的整个球面上对 D(𝐦) 积分，会得到微观表面的面积。更有用的是，对 D(𝐦)(𝐧·𝐦)，即 D(𝐦) 在宏观表面平面上的投影进行积分，会得到宏观表面片的面积；按约定，这个面积等于 1，如图 9.31 左图所示。换句话说，投影 D(𝐦)(𝐧·𝐦) 是归一化的：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_12b4050ddba3be.png)


该积分的范围是整个球面，这里以 Θ 表示；它与本章前面那些球面积分不同，后者只在以 𝐧 为中心的半球上积分，该半球用 Ω 表示。大多数图形学出版物都采用这种记号，不过有些参考文献 [708] 用 Ω 表示整个球面。实际上，图形学中使用的大多数微观结构模型都是高度场，这意味着对于 Ω 之外的所有方向 𝐦，都有 D(𝐦) = 0。不过，式 9.21 对非高度场的微观结构也同样成立。

更一般地，微观表面与宏观表面在垂直于任意观察方向 𝐯 的平面上的投影相等：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_4ff2ad1fe8bb95.png)


![图 9.32 可见微面元的投影面积](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_7_9.32.png)

**图 9.32** 对可见微面元（亮红色部分）的投影面积积分，会得到宏观表面在垂直于 𝐯 的平面上的投影面积。

式 9.21 和式 9.22 中的点积没有以 0 为下限进行截断。图 9.31 右图说明了原因。式 9.21 和式 9.22 规定了函数 D(𝐦) 成为有效 NDF 所必须满足的约束。

直观地说，NDF 就像微面元法线的直方图。在微面元法线更可能指向的方向上，它的值较高。大多数表面的 NDF 都在宏观表面法线 𝐧 处呈现一个很强的峰值。第 9.8.1 节将介绍渲染中使用的若干 NDF 模型。

再看一遍图 9.31 右图。虽然许多微面元的投影相互重叠，但对于渲染，我们最终只关心可见微面元，也就是每组投影重叠的微面元中最靠近相机的那个。这一事实提示了另一种方法，可将微面元的投影面积与宏观几何的投影面积联系起来：可见微面元的投影面积之和，等于宏观表面的投影面积。为了用数学表达这一点，我们定义**遮蔽函数** ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯)，它给出法线为 𝐦 的微面元中，沿观察向量 𝐯 可见的比例。于是，在球面上对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯)D(𝐦)![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_80063c3130e7a6.png) 积分，就得到宏观表面在垂直于 𝐯 的平面上的投影面积：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_b660c758fa80b7.png)


正如图 9.32 所示。与式 9.22 不同，式 9.23 中的点积以零为下限截断。这一操作用第 1.2 节引入的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_b9875aa6a4d8ef.png) 记号表示。背向微面元不可见，因此这里不将它们计入。乘积 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯)D(𝐦) 就是**可见法线分布** [708]。

> 译注：原书式 9.23 的积分下标印为“∈ Θ”，缺少积分变量；上式忠实保留原印刷形式。根据上下文与积分微元，这里应理解为“𝐦 ∈ Θ”。

虽然式 9.23 对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯) 施加了约束，但它并不能唯一确定这个函数。对于给定的微面元法线分布 D(𝐦)，存在无穷多个满足该约束的函数 [708]。这是因为 D(𝐦) 并未完整描述微观表面。它告诉我们有多少微面元的法线指向某些方向，却没有说明这些微面元如何排列。

多年来，人们提出了各种 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png) 函数，而关于该使用哪一个的难题，已经由 Heitz 的一篇出色论文 [708] 解决了，至少目前如此。Heitz 讨论了 Smith 遮蔽函数；该函数最初针对高斯法线分布推导出来 [1665]，后来被推广到任意 NDF [202]。Heitz 表明，在文献提出的遮蔽函数中，只有两个函数，即 Smith 函数和 Torrance–Sparrow 的“V 形凹腔”（V-cavity）函数 [1779]，满足式 9.23，因此在数学上有效。他进一步表明，与 Torrance–Sparrow 函数相比，Smith 函数更接近随机微观表面的行为。Heitz 还证明，Smith 遮蔽函数是唯一既满足式 9.23，又具有**法线与遮蔽相互独立**这一便利性质的函数。这意味着，只要 𝐦 不背向观察方向，也就是只要 𝐦·𝐯 ≥ 0，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯) 的值就不依赖于 𝐦 的方向。Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png) 函数具有如下形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_f1489845dcdfd8.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_83433dc5e3b0b5.png)(x) 是正值示性函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_7d3e1547ae0ccf.png)


每种 NDF 对应的 Λ（lambda）函数都不相同。Walter 等人 [1833] 和 Heitz [708] 的出版物介绍了如何为给定 NDF 推导 Λ。

Smith 遮蔽函数确实有一些缺点。从理论上看，它所要求的条件与真实表面的结构不一致 [708]，甚至可能无法在物理上实现 [657]。从实践上看，虽然它对随机表面相当准确，但对于法线方向与遮蔽之间依赖关系较强的表面，例如图 9.28 所示的表面，其准确度预计会降低；当表面具有某种重复结构时尤其如此，大多数织物就是这种情况。尽管如此，在找到更好的替代方案之前，它仍然是大多数渲染应用的最佳选择。

给定包含微观 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_c6bdae054ef589.png)、法线分布函数 D(𝐦) 和遮蔽函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯) 的微观几何描述，就可以推导出整个宏观表面的 BRDF [708, 1833]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_1c2255bc34635a.png)


该积分在以 𝐧 为中心的半球 Ω 上进行，以避免收集来自表面下方的光照贡献。式 9.26 没有使用遮蔽函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯)，而是使用**联合遮蔽—阴影函数** ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png)(𝐥, 𝐯, 𝐦)。该函数由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png) 推导而来，给出法线为 𝐦 的微面元中，同时从观察向量 𝐯 与光照向量 𝐥 这两个方向可见的比例。加入 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png) 函数后，式 9.26 使 BRDF 能够同时考虑遮蔽与阴影，但不能考虑微面元之间的相互反射（见第 329 页图 9.27）。缺少微面元间的相互反射，是由式 9.26 推导出的所有 BRDF 的共同局限。因此，这类 BRDF 会显得略暗。第 9.8.2 节和第 9.9 节将讨论一些为解决这一局限而提出的方法。

Heitz [708] 讨论了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png) 函数的若干版本。最简单的是**可分离形式**：用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png) 分别计算遮蔽和阴影，然后将二者相乘：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_190e8eeff75cdb.png)


这种形式相当于假设遮蔽和阴影是不相关的事件。实际并非如此，这一假设会使采用这种 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png) 形式的 BRDF 过度变暗。

考虑观察方向与光照方向相同这一极端例子。此时 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png) 应当等于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)，因为所有可见面元都不在阴影中；但使用式 9.27，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png) 却会等于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_cb8e18c2f54e76.png)。

如果微观表面是高度场，而渲染所用的微观表面模型通常如此，那么只要 𝐯 与 𝐥 之间的相对方位角 φ 等于 0°，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png)(𝐥, 𝐯, 𝐦) 就应当等于 min(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐯, 𝐦), ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐥, 𝐦))。φ 的示意见第 311 页图 9.17。这一关系提示了一种通用方法，可用来考虑遮蔽和阴影之间的相关性，并适用于任意 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png) 函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_337b258010baa2.png)


其中 λ(φ) 是某个随着角度 φ 增大而从 0 增至 1 的函数。Ashikhmin 等人 [78] 建议采用标准差为 15°（约 0.26 弧度）的高斯函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_aa4baac3468738.png)


van Ginneken 等人 [534] 提出了另一种 λ 函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_25f5edbb33e236.png)


> 译注：原书从式 9.27 起，在可分离形式及式 9.28 中将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png) 的参数写为“方向、微面元法线”，即 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐯, 𝐦) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐥, 𝐦)；前文定义与式 9.24 则使用“微面元法线、方向”的顺序 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯)。这里保留原文的参数顺序；按前文定义理解时，两者分别对应 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐯) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_d61bd232afc4ce.png)(𝐦, 𝐥)。

无论光照方向与观察方向的相对朝向如何，给定表面点处的遮蔽与阴影存在相关性，还有另一个原因：两者都与该点相对于表面其余部分的高度有关。位置较低的点被遮蔽的概率更大，处于阴影中的概率也更大。如果采用 Smith 遮蔽函数，就可以通过 **Smith 高度相关遮蔽—阴影函数**精确地计入这种相关性：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_c9aeb02f19e859.png)


Heitz 还描述了一种同时结合方向相关性和高度相关性的 Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_df633855eaaf23.png) 形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_bca259d10968d2.png)


其中，函数 λ(𝐯, 𝐥) 可以是式 9.29 和式 9.30 那样的经验函数，也可以是专门针对给定 NDF 推导的函数 [707]。

在这些候选方案中，Heitz [708] 推荐采用 Smith 函数的高度相关形式（式 9.31），因为它的计算开销与不相关形式近似，而准确度更高。这种形式在实践中使用最为广泛 [861, 947, 960]，不过也有一些实践者使用可分离形式（式 9.27）[214, 1937]。

通用微表面 BRDF（式 9.26）并不直接用于渲染。它的用途是：在具体选定微观 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_07_8579a6ce542b3d.png) 后，推导一个精确或近似的闭式解。下一节将展示这类推导的第一个例子。


## 9.8 表面反射的 BRDF 模型

来源：《Real-Time Rendering, Fourth Edition》，书页 336—347（PDF 物理页 357—368）。范围从 9.8 标题开始，到 9.9 标题之前，包含全部下级小节、图 9.33—9.39 和公式（9.33）—（9.60）。

除了少数例外，基于物理的渲染中使用的镜面 BRDF 项都推导自微表面理论。对于镜面表面反射，每个微表面都是完全光滑的菲涅耳镜面。回想一下，这种镜面会把每条入射光线反射到唯一的反射方向。这意味着，除非 v 平行于 l 的反射方向，否则每个面的微 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_7c230bb3026288.png) 都等于零。对于给定的 l 和 v 向量，这种构型等价于微表面法线 m 与一个恰好指向 l 和 v 正中间的向量对齐。这个向量就是**半程向量 h**。见图 9.33。将 v 与 l 相加并归一化即可得到它：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_f639b5903cbfad.png)


![图 9.33](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.33.png)

**图 9.33** 半程向量 h 与光照向量和观察向量形成相等的角度（以红色标出）。


![图 9.34](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.34.png)

**图 9.34** 由微表面组成的表面。只有红色微表面的表面法线与半程向量 h 对齐，能够将来自入射光照向量 l 的光反射到观察向量 v。

从公式（9.26）推导镜面微表面模型时，菲涅耳镜面的微 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_7c230bb3026288.png) 对所有 m ≠ h 都为零，这一点非常便利，因为它使积分简化为在 m = h 处对被积函数求值。由此得到镜面 BRDF 项：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_291716222d9c85.png)


推导的细节见 Walter 等人 [1833]、Heitz [708] 和 Hammon [657] 的出版物。Hammon 还给出了一种优化 BRDF 实现的方法：无需计算向量 h 本身，就能计算 n·h 和 l·h。

我们用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_70aa7f04e7c389.png) 表示公式（9.34）中的 BRDF 项，以表明它仅对表面（镜面）反射建模。在完整 BRDF 中，它很可能会与一个对次表面（漫反射）着色建模的附加项搭配使用。要直观理解公式（9.34），可以考虑：只有那些法线恰巧与半程向量对齐（m = h）的微表面，其朝向才适合把光从 l 反射到 v，见图 9.34。因此，反射光量取决于三个因素：法线等于 h 的微表面的集中程度，由 D(h) 给出；这些微表面中，从光照方向和观察方向都可见的比例，等于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png)(l, v, h)；以及每个这样的微表面反射的光所占的比例，由 F(h, l) 指定。计算菲涅耳函数时，以向量 h 代替表面法线，例如计算书页 320 的公式（9.16）中的 Schlick 近似时就是如此。

在遮蔽—阴影函数中使用半程向量，还允许作一个小的简化。由于涉及的角度不可能大于 90°，可以去掉公式（9.24）、（9.31）和（9.32）中的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_83433dc5e3b0b5.png) 项。

### 9.8.1 法线分布函数

法线分布函数会显著影响渲染表面的外观。在微表面法线所对应的球面上绘出的 NDF 形状，决定了反射光线锥体（镜面波瓣）的宽度和形状，进而决定镜面高光的大小与形状。NDF 不仅影响表面粗糙度的整体感知，还影响更细微的视觉特征，例如高光是否具有清晰的边缘，或是否被一层薄雾般的光晕包围。


![图 9.35](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.35.png)

**图 9.35** 左侧图像使用非物理的 Phong 反射模型渲染。该模型的镜面波瓣绕反射向量旋转对称。计算机图形学早期经常使用这类 BRDF。中间的图像使用基于物理的微表面 BRDF 渲染。左上与中上显示的是以掠射角照明的平面表面。左上呈现出错误的圆形高光，而中间展现了微表面 BRDF 所特有的高光拉长现象。中间的结果符合现实，右侧照片即为例证。下方两幅渲染图中的球体，其高光形状差异要细微得多，因为此时表面曲率才是决定高光形状的主导因素。（照片由 Elan Ruskin 提供。）

然而，镜面波瓣并不是 NDF 形状的简单复制。波瓣及其决定的高光形状，会随着表面曲率和观察角度而发生程度不同的畸变。对于以掠射角观察的平坦表面，这种畸变尤其明显，如图 9.35 所示。Ngan 等人 [1271] 分析了这种畸变产生的原因。

#### 各向同性法线分布函数

渲染中使用的大多数 NDF 都是各向同性的，即绕宏观表面法线 n 旋转对称。在这种情况下，NDF 只依赖一个变量：n 与微表面法线 m 之间的夹角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_8724dc31392959.png)。理想情况下，NDF 可以写成 cos ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_8724dc31392959.png) 的表达式，从而通过 n 与 m 的点积高效计算。

Beckmann NDF [124] 是光学界最初开发的微表面模型所采用的法线分布，直到今天仍在该领域广泛使用。Cook–Torrance BRDF [285, 286] 也选择了它。归一化的 Beckmann 分布形式如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_930ea9e94dc355.png)


![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_83433dc5e3b0b5.png)(n·m) 项保证：对于所有指向宏观表面下方的微表面法线，NDF 的值都是 0。这个性质说明，该 NDF 与本节将讨论的其他所有 NDF 一样，描述的是高度场微表面。参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_015ce8c51d037b.png) 控制表面粗糙度，它正比于微几何表面斜率的均方根（RMS），因此 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_015ce8c51d037b.png) = 0 表示完全光滑的表面。

要推导 Beckmann NDF 的 Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 函数，需要相应的 Λ 函数，并将它代入公式（9.24）（使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 的可分离形式时）、（9.31）（使用高度相关形式时）或（9.32）（使用方向与高度相关形式时）。

Beckmann NDF 具有**形状不变性**，这简化了 Λ 的推导。按照 Heitz [708] 的定义，如果各向同性 NDF 的粗糙度参数产生的效果等价于缩放（拉伸）微表面，那么该 NDF 就是形状不变的。具有形状不变性的 NDF 可以写成以下形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_f2ba06d9cc5be9.png)


其中 g 表示任意一元函数。对于任意各向同性 NDF，Λ 函数依赖两个变量：第一个是粗糙度 α，第二个是计算 Λ 时所用向量（v 或 l）的入射角。然而，对于形状不变的 NDF，Λ 函数只依赖变量 a：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_842162d094b5b8.png)


其中向量 s 代表 v 或 l。此时 Λ 只依赖一个变量，这给实现带来了便利。一元函数更容易用近似曲线拟合，也可以制成一维数组形式的查找表。

Beckmann NDF 的 Λ 函数为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d50b28cfce9778.png)


公式（9.38）包含误差函数 erf，计算开销很大。因此通常改用以下近似 [1833]：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_20e0421f89b8b0.png)


接下来讨论 Blinn–Phong NDF。它过去在计算机图形学中应用广泛，但近来大部分已被其他分布取代。在计算资源十分紧张的场合（例如移动硬件），仍然会使用 Blinn–Phong NDF，因为它比本节讨论的其他 NDF 计算成本更低。

Blinn [159] 通过修改非物理的 Phong 着色模型 [1414]，推导出了 Blinn–Phong NDF：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_5295c74274b3ab.png)


指数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) 是 Phong NDF 的粗糙度参数。较大的值表示光滑表面，较小的值表示粗糙表面。对于极为光滑的表面，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) 可以任意大；理想镜面需要 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) = ∞。令 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) 为 0，即可得到随机程度最大的表面（均匀 NDF）。直接操纵 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) 并不方便，因为它的视觉影响极不均匀：在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) 较小时，数值的小幅变化就会带来很大的视觉变化；而在数值较大时，即使显著改变数值，视觉影响也不大。因此，通常通过非线性映射，从用户操纵的参数推导出 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png)。例如，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_03b257953c010d.png)，其中 s 是 0 到 1 之间的参数值，m 是特定应用中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_501fa05cfd75db.png) 的上限。若干游戏采用了这种映射，其中《Call of Duty: Black Ops》将 m 设为 8192 [998]。

当 BRDF 参数的行为在感知上不均匀时，这类“界面映射”通常很有用。这些映射用于解释通过滑块设置或在纹理中绘制的参数。

利用关系 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_52ddac3f20351d.png) [1833]，可以找到 Beckmann 与 Blinn–Phong 粗糙度参数的等效值。以这种方式匹配参数时，两种分布非常接近，尤其对于相对光滑的表面，如图 9.36 左上所示。

Blinn–Phong NDF 不具有形状不变性，其 Λ 函数也不存在解析形式。Walter 等人 [1833] 建议结合参数等效关系 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_52ddac3f20351d.png)，使用 Beckmann 的 Λ 函数。

在 1977 年将 Phong 着色函数改造成微表面 NDF 的同一篇论文 [159] 中，Blinn 还提出了另外两种 NDF。在这三种分布中，他推荐了 Trowbridge 和 Reitz [1788] 推导的一种分布。这个建议没有受到广泛重视，但 30 年后，Walter 等人 [1833] 独立地重新发现了 Trowbridge–Reitz 分布，并将它命名为 GGX 分布。这一次，这颗种子生根了。短短几年内，GGX 分布便开始在电影 [214, 1133] 和游戏 [861, 960] 行业中传播，如今它很可能已成为这两个行业使用最频繁的分布。看来 Blinn 的推荐比时代早了 30 年。虽然严格来说正确名称应为“Trowbridge–Reitz 分布”，但由于 GGX 这个名称已深入人心，本书仍使用它。

GGX 分布为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_6fc3babc02bc2a.png)


参数 αɡ 提供的粗糙度控制与 Beckmann 参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_015ce8c51d037b.png) 类似。在 Disney 原则化着色模型中，Burley [214] 以 αɡ = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_730f558bb648d5.png) 的方式向用户提供粗糙度控制，其中 r 是用户界面中取值在 0 到 1 之间的粗糙度参数。让用户通过滑块操纵 r，会使效果以更接近线性的方式变化。大多数使用 GGX 分布的应用都采用了这种映射。


![图 9.36](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.36.png)

**图 9.36** 左上比较了 Blinn–Phong（蓝色虚线）与 Beckmann（绿色）分布，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_015ce8c51d037b.png) 的取值范围为 0.025 到 0.2，使用参数关系 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_52ddac3f20351d.png)。右上比较了 GGX（红色）与 Beckmann（绿色）分布。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_015ce8c51d037b.png) 的值与左图相同；αɡ 的值通过目测调整，以匹配高光大小。下方球体的渲染也使用了这些相同的值，上排使用 Beckmann NDF，下排使用 GGX。

GGX 分布具有形状不变性，其 Λ 函数相对简单：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_652c5b106a721c.png)


变量 a 在公式（9.42）中仅以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_5929c46ee0c3fc.png) 的形式出现，这一点很方便，因为可以避免计算公式（9.37）中的平方根。

由于 GGX 分布和 Smith 遮蔽—阴影函数十分流行，人们专门投入了精力来优化两者的组合。Lagarde [960] 观察到，GGX 的高度相关 Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png)（公式（9.31））与镜面微表面 BRDF 的分母（公式（9.34））组合后，有些项会相消。组合项可以简化为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_7a4129438e6d59.png)


为简洁起见，公式使用变量替换 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_8166b35af25981.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_3e52ebf88021af.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_83ffb449fca926.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_5df16950eb6124.png)。


![图 9.37](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.37.png)

**图 9.37** 对 MERL 数据库中实测铬材质拟合得到的 NDF。左侧绘出了镜面峰值随 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_8724dc31392959.png) 变化的曲线：铬为黑色，GGX 为红色（αɡ = 0.006），Beckmann 为绿色（![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_015ce8c51d037b.png) = 0.013），Blinn–Phong 为蓝色虚线（n = 12000）。右侧为铬、GGX 和 Beckmann 的高光渲染结果。（图片由 Brent Burley [214] 提供。）

Karis [861] 提出了 GGX 的 Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d61bd232afc4ce.png) 函数的近似形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_1c3b56ee9d27ec.png)


其中 s 可以替换为 l 或 v。Hammon [657] 指出，这个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d61bd232afc4ce.png) 的近似形式能够导出一个高效近似，用于计算高度相关 Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 函数与镜面微表面 BRDF 分母构成的组合项：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_b41280e9999954.png)


其中使用了线性插值算子 lerp(x, y, s) = x(1 − s) + ys。

比较图 9.36 中的 GGX 与 Beckmann 分布，可以明显看出两者的形状有根本差异。GGX 的峰比 Beckmann 更窄，峰周围的“尾部”也更长。从图底部的渲染图像中可以看到，GGX 较长的尾部在高光核心周围形成了薄雾或光晕般的外观。

许多现实材质都有类似的朦胧高光，其尾部通常甚至比 GGX 分布更长 [214]，见图 9.37。这一认识是 GGX 分布日益流行的重要推动因素，也促使人们继续寻找能更准确拟合实测材质的新分布。

Burley [214] 提出了**广义 Trowbridge–Reitz（GTR）NDF**，目的是更充分地控制 NDF 的形状，特别是分布尾部：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_e1c0c041dbc8cf.png)


参数 γ 控制尾部形状。当 γ = 2 时，GTR 与 GGX 相同。γ 减小时，分布尾部变长；γ 增大时，尾部变短。当 γ 较大时，GTR 分布类似于 Beckmann。k(α, γ) 项是归一化因子；由于它比其他 NDF 的归一化因子更复杂，我们用单独的公式给出：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_483ebcfb20f0ab.png)


> 译注：公式（9.46）分母中的粗糙度在原书排作 αɡ，而归一化因子和公式（9.47）使用 α；此处保留原书记号，不作暗改。

GTR 分布不具有形状不变性，这使其 Smith ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 遮蔽—阴影函数的求解更复杂。NDF 发表三年之后，才有 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 的解发表 [355]。这个 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 解相当复杂，需要为某些 γ 值列出解析解表，中间值则必须插值。GTR 的另一个问题是，参数 α 和 γ 对感知粗糙度与“光晕”的影响并不直观。

**Student t 分布（STD）**[1491] 和**指数幂分布（EPD）**[763] NDF 都包含形状控制参数。与 GTR 不同，这些函数相对于粗糙度参数具有形状不变性。在本书写作时，它们刚刚发表，尚不清楚是否会进入实际应用。

为了更好地匹配实测材质，除了增加 NDF 的复杂度，另一种办法是使用多个镜面波瓣。Cook 和 Torrance [285, 286] 提出了这个想法。Ngan [1271] 对其进行了实验检验，发现对于许多材质，增加第二个波瓣确实能显著改善拟合。Pixar 的 PxrSurface 材质 [732] 有一个“roughspecular”波瓣，其用途就是与主镜面波瓣一起实现这一目的。附加波瓣是一个完整的镜面微表面 BRDF，带有所有相关参数和项。Imageworks 采取了更有针对性的办法 [947]：混合两个 GGX NDF，并将其作为扩展 NDF 提供给用户，而非另加一个完整的镜面 BRDF 项。这样，唯一需要增加的参数就是第二个粗糙度值和混合量。

#### 各向异性法线分布函数

虽然大多数材质具有各向同性的表面统计特征，但有些材质的微结构具有显著各向异性，足以明显影响外观，例如书页 329 的图 9.26。为了准确渲染这些材质，需要同样具有各向异性的 BRDF，尤其是各向异性 NDF。

与各向同性 NDF 不同，各向异性 NDF 不能仅用角度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_8724dc31392959.png) 求值，还需要额外的朝向信息。一般情况下，需要将微表面法线 m 变换到由法线、切线和副切线向量（分别为 n、t 和 b）定义的局部坐标系，也就是切线空间，见书页 210 的图 6.32。实践中，这种变换通常表示为三个独立的点积：m·n、m·t 和 m·b。

将法线映射与各向异性 BRDF 结合时，必须确保法线贴图不仅扰动法线，也扰动切线和副切线向量。通常，对扰动后的法线 n 以及经过插值的顶点切线和副切线向量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9d5212cdbc9a74.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_ad0d7a1ea0a9ba.png) 应用修正的 Gram–Schmidt 过程来完成这一操作（以下假定 n 已经归一化）：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_cf562f9a0c2f20.png)


另一种办法是在完成第一行之后，取 n 与 t 的叉积来构造正交的 b 向量。

对于拉丝金属或卷发等效果，需要逐像素修改切线方向，通常由**切线贴图**提供。这种纹理存储每个像素的切线，类似于法线贴图存储逐像素法线。切线贴图最常存储的是切线向量在垂直于法线的平面上的二维投影。这种表示能很好地配合纹理过滤，也能像法线贴图一样压缩。有些应用改为存储一个标量旋转量，用它使切线向量绕 n 旋转。虽然这种表示更加紧凑，但当旋转角从 360° 回绕到 0° 时，容易产生纹理过滤伪影。

创建各向异性 NDF 的常用办法，是将已有的各向同性 NDF 推广。这种通用方法适用于任何具有形状不变性的各向同性 NDF [708]，这也是优先选用形状不变 NDF 的另一个理由。回顾一下，各向同性的形状不变 NDF 可以写成：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_4e67cb83c82542.png)


其中 g 是表达 NDF 形状的一维函数。其各向异性版本为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_bff42104d8faaf.png)


参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_a387494e1dbfe5.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9436b5e7858502.png) 分别表示沿 t 和 b 方向的粗糙度。如果 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_a387494e1dbfe5.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9436b5e7858502.png)，公式（9.50）就退化回各向同性形式。

各向异性 NDF 的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_df633855eaaf23.png) 遮蔽—阴影函数与各向同性版本相同，唯一的区别是传入 Λ 函数的变量 a 采用不同的计算方式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_635787a4207414.png)


其中，与公式（9.37）一样，s 代表 v 或 l。


![图 9.38](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.38.png)

**图 9.38** 使用各向异性 NDF 渲染的球体：上排为 Beckmann，下排为 GGX。两排都保持 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9436b5e7858502.png) 不变，让 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_a387494e1dbfe5.png) 从左到右增大。

采用这种方法，可以推导出 Beckmann NDF 的各向异性版本：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_93655c35f83de2.png)


以及 GGX NDF 的各向异性版本：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d3d0763e27520d.png)


两者都展示在图 9.38 中。

虽然各向异性 NDF 最直接的参数化办法，是将各向同性的粗糙度参数化使用两次，分别用于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_a387494e1dbfe5.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9436b5e7858502.png)，但有时也会使用其他参数化方式。在 Disney 原则化着色模型 [214] 中，各向同性粗糙度参数 r 与第二个标量参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_0106f57d323d08.png) 组合，后者的范围为 [0, 1]。由这些参数计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_a387494e1dbfe5.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9436b5e7858502.png) 的方式如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_4c541e7c5ef93e.png)


因子 0.9 将长宽比限制为 10 : 1。

Imageworks [947] 使用另一种参数化方式，允许任意程度的各向异性：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_2b4db77e153dec.png)


### 9.8.2 多次弹射的表面反射

如前面 9.7 节所述，微表面 BRDF 框架没有考虑在微表面上反射（“弹射”）多次的光。这一简化会造成一定的能量损失和过度变暗，对粗糙金属尤其如此 [712]。

Imageworks [947] 使用的一项技术融合了此前研究 [811, 878] 中的要素，构造出一个可加到 BRDF 上的项，用于模拟多次弹射的表面反射：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_aef0013b9a30f1.png)


其中 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_85731b81f882db.png) 是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_36fa538fe264cc.png) 的方向反照率（9.3 节），而后者是把 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d8e52c5a12790b.png) 设为 1 的镜面 BRDF 项。函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_85731b81f882db.png) 依赖粗糙度 α 和仰角 θ。它相对平滑，因此可以通过数值方法预计算（使用公式（9.8）或（9.9）），并存入一张小型二维纹理。Imageworks 发现 32 × 32 的分辨率就已足够。

函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d32e641118852f.png) 是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_85731b81f882db.png) 在半球上的余弦加权平均值。它只依赖 α，因此可以存入一维纹理，也可以用一条计算成本很低的曲线拟合这些数据。由于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_85731b81f882db.png) 绕 n 旋转对称，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_d32e641118852f.png) 可以通过一维积分计算。我们还使用变量代换 μ = cos θ（见书页 312 的公式（9.6））：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_88f9055d6c462f.png)


最后，F̅ 是菲涅耳项的余弦加权平均值，以同样方式计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_93fd9000ae9024.png)


对于 F 使用广义 Schlick 形式（公式（9.18））的情况，Imageworks 给出了公式（9.58）的闭式解：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_eab34b4e682567.png)


如果使用原始的 Schlick 近似（公式（9.16）），解则简化为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_c7523187ec29cb.png)


在各向异性的情况下，Imageworks 使用介于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_a387494e1dbfe5.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_9436b5e7858502.png) 之间的粗糙度来计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_55a14e5395884b.png)。这种近似避免了增加 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_08_85731b81f882db.png) 查找表的维数，而且引入的误差很小。

Imageworks 多次弹射镜面项的效果见图 9.39。


![图 9.39](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_8_9.39.png)

**图 9.39** 所有行的表面粗糙度都从左到右递增。最上面的两行展示金材质：第一行渲染时未使用 Imageworks 多次弹射项，第二行使用了该项。对于更粗糙的球体，差异最为明显。接下来的两行展示黑色电介质材质：第三行未使用多次弹射项，第四行使用了该项。由于镜面反射率低得多，此时差异也更加细微。（图片由 Christopher Kulla [947] 提供。）


## 9.9 次表面散射的 BRDF 模型

来源：《Real-Time Rendering, Fourth Edition》，书页 347—355（PDF 物理页 368—376）；另核对书页 356（PDF 377）以确认下一节边界。本节从 9.9 标题开始，包含全部四个下级小节；图 9.39 属于 9.8，图 9.42 属于 9.10。

上一节讨论了表面反射，也就是镜面反射。本节讨论问题的另一面：折射进入表面之下的光会发生什么。如 9.1.4 节所述，这些光会经历散射与吸收的某种组合，其中一部分又从原来的表面重新射出。这里重点讨论不透明电介质中局部次表面散射（也称漫反射表面响应）的 BRDF 模型。金属与此无关，因为金属内部不存在显著的次表面光相互作用。透明电介质，以及表现出全局次表面散射的电介质材料，将在第 14 章讨论。

我们首先讨论漫反射颜色这一属性，以及现实材料中这种颜色可能具有的取值。接着解释表面粗糙度对漫反射着色的影响，并说明为给定材料选择光滑表面还是粗糙表面着色模型的判断标准。最后两个小节分别讨论光滑表面模型和粗糙表面模型本身。

### 9.9.1 次表面反照率

不透明电介质的**次表面反照率** ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png)，是逃离表面的光能量与进入材料内部的光能量之比。![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 的值介于 0（所有光都被吸收）与 1（没有光被吸收）之间，并且可能随波长而变化，因此在渲染中将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 建模为 RGB 向量。在材质制作中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 常被称为表面的**漫反射颜色**，正如法向入射时的菲涅耳反射率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d8e52c5a12790b.png) 通常被称为**镜面反射颜色**。次表面反照率与 14.1 节讨论的散射反照率密切相关。

电介质会透射大部分入射光，而不是在表面将其反射，因此次表面反照率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 通常比镜面反射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d8e52c5a12790b.png) 更亮，在视觉上也就更重要。它与镜面反射颜色来自不同的物理过程：前者来自内部吸收，后者来自表面的菲涅耳反射。因此，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 通常具有不同于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d8e52c5a12790b.png) 的光谱分布，进而具有不同的 RGB 颜色。例如，彩色塑料由清澈透明的基质及嵌入内部的颜料颗粒组成。镜面反射的光不会着色，而漫反射的光会因颜料颗粒的吸收而着色；例如，红色塑料球具有白色高光。

可以把次表面反照率看作吸收与散射之间的一场“赛跑”的结果：光是否还没来得及被散射回物体外，就已经被吸收了？这也解释了为什么液体上的泡沫比液体本身明亮得多。起泡过程不会改变液体的吸收能力，但大量新增的空气与液体界面会显著增加散射量。于是，大部分入射光在被吸收之前就发生散射，形成较高的次表面反照率和明亮的外观。新雪也是高反照率物质的例子。雪粒与空气之间的界面会产生大量散射，但吸收很少，因此其整个可见光谱范围内的次表面反照率可达 0.8 或更高。白漆略低，约为 0.7。日常生活中常见的许多物质，如混凝土、石头和土壤，平均值在 0.15 至 0.4 之间。煤则是次表面反照率极低的材料，接近 0.0。

许多材料受潮后变暗的过程，与液体泡沫这个例子正好相反。如果材料具有孔隙，水就会渗入原先由空气占据的空间。电介质材料的折射率与水的折射率接近得多，而与空气相差较大。相对折射率的这种降低会减少材料内部的散射，使光在逃出材料之前平均传播更长的距离。这样一来，更多光被吸收，次表面反照率就变暗了 [821]。

一种常见的误解是：制作真实材质时，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 的值绝不应低于约 0.015—0.03 的下限（以 8 位非线性 sRGB 编码表示时为 30—50）。甚至一些备受推崇的材质制作指南 [1163] 也体现了这种误解。然而，这一下限所依据的颜色测量同时包含表面（镜面）反射率和次表面（漫反射）反射率，因此定得过高。真实材料完全可以具有更低的数值。例如，“OSHA Black”黑色涂料标准的联邦规范 [524] 规定 Y 值为 0.35（满值为 100）。结合其测量条件与表面光泽，这个 Y 值对应的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 约为 0.0035（以 8 位非线性 sRGB 编码表示时为 11）。

从现实表面采集 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 的单点数值或纹理时，必须把镜面反射分离出去。通过谨慎使用受控照明与偏振滤镜，可以完成这种提取 [251, 952]。为了获得准确的颜色，还应进行校准 [1153]。

并不是每一个 RGB 三元组都能表示合理的、甚至物理上可能存在的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 值。反射光谱比发射光的光谱功率分布受到更多限制：它在任何波长处都不能超过 1，而且通常相当平滑。这些限制在色彩空间中界定出一个体积，其中包含 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 所有合理的 RGB 值。即便是范围相对较小的 sRGB 色域，也包含落在这个体积之外的颜色，因此设置 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 时必须谨慎，避免指定不自然的高饱和度、高亮度颜色。除了削弱真实感，这类颜色还可能在预计算全局光照时造成过亮的二次反射（11.5.1 节）。Meng 等人于 2015 年发表的论文 [1199] 是这一主题很好的参考资料。

### 9.9.2 次表面散射与粗糙度的尺度

有些局部次表面散射 BRDF 模型会考虑表面粗糙度，通常使用微表面理论，并采用漫反射微 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_8579a6ce542b3d.png)；另一些模型则不考虑。选择哪一类模型的决定因素，并不只是表面有多粗糙，尽管这是一种常见误解。正确的决定因素是表面凹凸的尺寸与次表面散射距离之间的相对大小。

见图 9.40。如果微几何的凹凸尺寸大于次表面散射距离（图的左上部分），次表面散射就会表现出与微几何相关的效应，例如逆向反射（书页 331 的图 9.29）。这种表面应使用粗糙表面漫反射模型。如上所述，这类模型通常基于微表面理论，把次表面散射视为局限于每个微表面的局部过程，因此它只影响微 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_8579a6ce542b3d.png)。

如果散射距离都大于表面凹凸的尺寸（图 9.40 右上），那么在对次表面散射建模时，应把表面视为平坦表面，此时不会出现逆向反射等效应。次表面散射不再局限于单个微表面，因而不能通过微表面理论建模。在这种情况下，应使用光滑表面漫反射模型。


![图 9.40 微几何尺度与次表面散射距离](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_9_9.40.png)

**图 9.40** 三种表面具有相似的法线分布函数（NDF），但微几何尺度与次表面散射距离之间的关系不同。左上：次表面散射距离小于表面凹凸尺寸。右上：散射距离大于表面凹凸尺寸。下图展示了在多个尺度上具有粗糙度的微表面。红色虚线表示有效表面，它只包含尺寸大于次表面散射距离的微结构。

在介于两者之间的情况下，表面同时在大于和小于散射距离的尺度上具有粗糙度。此时应使用粗糙表面漫反射模型，但应基于一个只包含尺寸大于散射距离的凹凸的**有效表面**。漫反射和镜面反射都可以用微表面理论建模，但两者分别使用不同的粗糙度值。镜面反射项使用依据实际表面粗糙度得到的数值，漫反射项则使用依据有效表面粗糙度得到的较低数值。

观察尺度也与这个问题有关，因为它决定了“微几何”的定义。例如，月球常被用作应当使用粗糙表面漫反射模型的例子，因为它表现出显著的逆向反射。当我们从地球观看月球时，在这样的观察尺度下，即便一块五英尺大的巨石也属于“微几何”。因此，我们会观察到逆向反射等粗糙表面漫反射效应，也就不足为奇了。

### 9.9.3 光滑表面的次表面模型

这里讨论光滑表面的次表面模型。当材料的表面凹凸尺寸小于次表面散射距离时，适合使用这类模型。在这样的材料中，漫反射着色不会直接受到表面粗糙度影响。如果漫反射项与镜面反射项相耦合——本节中的部分模型就是如此——那么表面粗糙度可能间接影响漫反射着色。

如 9.3 节所述，实时渲染应用通常用朗伯项来建模局部次表面散射。此时，BRDF 的漫反射项是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png) 除以 π：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_05790887030c0a.png)


朗伯模型没有考虑一个事实：在表面被反射的光不能再参与次表面散射。为了改进该模型，表面（镜面）反射项与次表面（漫反射）反射项之间应当存在能量上的此消彼长。菲涅耳效应意味着，这种表面与次表面之间的能量分配会随入射光角度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_ba4635370b500e.png) 而变化。随着入射角越来越接近掠射，镜面反射率增加，漫反射率则减小。考虑这种平衡的一种基本方式，是把漫反射项乘以一减去镜面反射项中的菲涅耳部分 [1626]。如果镜面反射项对应平面镜，则得到的漫反射项为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d40696ce92c1e2.png)


如果镜面反射项是微表面 BRDF 项，则得到的漫反射项为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_3e6053b925d211.png)


公式（9.62）和（9.63）使出射光均匀分布，因为 BRDF 值不依赖于出射方向 **v**。这种行为有一定道理：光在重新射出之前，通常会经历多次散射，因此其出射方向会被随机化。不过，有两个理由使人怀疑出射光并非完全均匀分布。首先，公式（9.62）中的漫反射 BRDF 项随入射方向变化，根据亥姆霍兹互易性，它也必须随出射方向变化。其次，光在射出的过程中必须经历折射，这会使出射光对某些方向有所偏好。

> 译注：上一段关于公式（9.63）不依赖出射方向的说法按原文保留。但其中半程向量 **h** 由入射与出射方向共同决定，因此该式一般仍会通过 **h** 间接依赖 **v**；原文此处的概括存在疑点。

Shirley 等人为平坦表面提出了一个耦合漫反射项，既处理菲涅耳效应及表面与次表面反射之间的能量分配，又满足能量守恒与亥姆霍兹互易性 [1627]。其推导假定使用 Schlick 近似 [1568]（公式 9.16）来计算菲涅耳反射率：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_127a3483dc45ad.png)


公式（9.64）只适用于镜面反射等同于完美菲涅耳镜面的表面。Ashikhmin 与 Shirley [77] 提出了一个推广版本，之后 Kelemen 与 Szirmay-Kalos [878] 又作了进一步改进。它能够计算出满足互易性与能量守恒的漫反射项，并与任意镜面反射项耦合：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d42f3c80fe574a.png)


这里，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 是镜面反射项的方向反照率（9.3 节），![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_eaa86c8bc61cdc.png) 是它在半球上的余弦加权平均值。可以利用公式（9.8）或（9.9）预计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 并存入查找表。平均值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_eaa86c8bc61cdc.png) 的计算方法，与先前遇到的类似平均值 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d32e641118852f.png) 相同（公式 9.57）。

公式（9.65）的形式与公式（9.56）明显相似。这并不奇怪，因为 Imageworks 的多次反弹镜面反射项就是从 Kelemen-Szirmay-Kalos 耦合漫反射项推导而来的。不过，两者有一个重要区别。这里使用的不是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_85731b81f882db.png)，而是 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png)：它是完整镜面反射 BRDF 项的方向反照率，包含菲涅耳项；如果使用了多次反弹镜面反射项 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_55a14e5395884b.png)，也要将其包括在内。这一区别增加了 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 查找表的维数，因为它不仅依赖粗糙度 α 和仰角 θ，还依赖菲涅耳反射率。

Imageworks 在实现 Kelemen-Szirmay-Kalos 耦合漫反射项时，使用三维查找表，以折射率作为第三条轴 [947]。他们发现，把多次反弹项纳入积分后，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 比 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_85731b81f882db.png) 更平滑，因此 16 × 16 × 16 的表就足够了。图 9.41 展示了结果。


![图 9.41 朗伯项与耦合漫反射项的比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_9_9.41.png)

**图 9.41** 第一行和第三行展示了镜面反射项与朗伯项相加的结果。第二行和第四行展示了相同镜面反射项与 Kelemen-Szirmay-Kalos 耦合漫反射项结合的结果。上面两行的粗糙度值低于下面两行。每一行内部，粗糙度均从左向右增加。（图片由 Christopher Kulla 提供 [947]。）

如果 BRDF 使用 Schlick 菲涅耳近似，并且不包含多次反弹镜面反射项，那么 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d8e52c5a12790b.png) 可以从积分中提取出来。正如 Karis [861] 所讨论的，这样就可以为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 使用二维表，在每个表项中存储两个量，而不必使用三维表。另一种方法是 Lazarov [999] 给出的拟合 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 的解析函数；该方法同样将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_d8e52c5a12790b.png) 从积分中提取出来，以简化拟合函数。

Karis 和 Lazarov 都把镜面反射方向反照率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f5de4bd8a08b89.png) 用于另一种与基于图像的光照有关的用途。该技术的更多细节见 10.5.2 节。如果同一应用同时实现了这两种技术，就可以让它们共用相同的查表结果，从而提高效率。

这些模型是通过考虑表面（镜面）项与次表面（漫反射）项之间能量守恒的含义而建立的。另一些模型则从物理原理出发。许多此类模型依赖 Subrahmanyan Chandrasekhar（1910—1995）的工作，他为半无限、各向同性散射的体积建立了一个 BRDF 模型。Kulla 与 Conty [947] 的演示表明，只要平均自由程足够短，这个 BRDF 模型就能与任意形状的散射体积完美匹配。Chandrasekhar 的著作 [253] 中给出了该 BRDF；Dupuy 等人的论文 [397] 中，公式（30）和（31）则采用我们熟悉的渲染记号，给出了更易理解的形式。

由于不包含折射，Chandrasekhar BRDF 只能用于建模**折射率匹配表面**，即表面两侧折射率相同的表面，如书页 304 的图 9.11 所示。为了建模折射率不匹配的表面，必须修改 BRDF，考虑光进入和离开表面时的折射。Hanrahan 与 Krueger [662] 以及 Wolff [1898] 的研究重点就是这种修改。

### 9.9.4 粗糙表面的次表面模型

作为 Disney 原则性着色模型的一部分，Burley [214] 加入了一个漫反射 BRDF 项，用于包含粗糙度效应，并匹配实测材料：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_06e50d3c79f6ed.png)


其中


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_5580bc8d1bf08a.png)


α 为镜面反射粗糙度。对于各向异性，使用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_a387494e1dbfe5.png) 与 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_9436b5e7858502.png) 之间的一个中间值。这一公式通常被称为 **Disney 漫反射模型**。

次表面项 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_6618de0d9131a6.png) 受到 Hanrahan-Krueger BRDF [662] 的启发，目的是为远处物体的全局次表面散射提供一个低成本替代方案。该漫反射模型根据用户控制的参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_f10ca0217ebc2a.png)，在 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_6618de0d9131a6.png) 与粗糙漫反射项 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_68c720b3301fb8.png) 之间混合。

Disney 漫反射模型既用于电影 [214]，也用于游戏 [960]（不过游戏中未使用次表面项）。完整的 Disney 漫反射 BRDF 还包含一个 sheen（绒光）项，主要用于建模织物，同时也有助于补偿缺少多次反弹镜面反射项所造成的能量损失。Disney 绒光项将在 9.10 节讨论。数年后，Burley 提出了一个更新模型 [215]，用于与全局次表面散射渲染技术结合。

Disney 漫反射模型使用与镜面反射 BRDF 项相同的粗糙度，因此建模某些材料时可能遇到困难，见图 9.40。不过，只需做一个很简单的修改，就能使用独立的漫反射粗糙度值。

其他大多数粗糙表面漫反射 BRDF 都利用微表面理论建立，但对 NDF D、微 BRDF ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_8579a6ce542b3d.png) 和遮蔽—阴影函数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_df633855eaaf23.png) 采用了不同选择。其中最著名的模型由 Oren 与 Nayar [1337] 提出。Oren-Nayar BRDF 使用朗伯微 BRDF、球面高斯 NDF，以及 Torrance-Sparrow 的“V 形凹腔”遮蔽—阴影函数。该 BRDF 的完整形式对一次二次反弹进行了建模。Oren 与 Nayar 还在论文中给出了一个简化的“定性”模型。多年来，人们对 Oren-Nayar 模型提出了多种改进，包括优化 [573]；在不增加成本的情况下调整“定性”模型，使其更接近完整模型 [504]；以及把微 BRDF 改为更准确的光滑表面漫反射模型 [574, 1899]。

Oren-Nayar 模型所假定的微表面，其法线分布和遮蔽—阴影函数与当前镜面反射模型使用的函数有很大差异。另有两个漫反射微表面模型采用各向同性 GGX NDF 和高度相关的 Smith 遮蔽—阴影函数推导而成。第一个由 Gotanda [574] 提出：使用公式（9.64）中与镜面反射耦合的漫反射项作为微 BRDF，对通用微表面公式（9.26）作数值积分，再对所得数据拟合一个解析函数。Gotanda 的 BRDF 不考虑微表面之间的相互反射，而且拟合函数相对复杂。

Hammon [657] 使用与 Gotanda 相同的 NDF、遮蔽—阴影函数和微 BRDF，对 BRDF 进行包含相互反射的数值模拟。他展示了相互反射对这种微表面配置的重要性：对于较粗糙的表面，相互反射可占总反射率的一半。不过，第二次反弹已包含几乎全部缺失能量，因此 Hammon 使用两次反弹模拟的数据。此外，可能是因为加入相互反射使数据变得更平滑，Hammon 得以用一个相当简单的函数拟合模拟结果：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_9d552ed34b12a2.png)


其中


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_a4cc34712d764c.png)


αɡ 为 GGX 镜面反射粗糙度。为使表达清晰，这里的各项采用了与 Hammon 原始报告略有不同的因式分解方式。注意，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_11c6b519c5c557.png) 就是公式（9.64）的耦合漫反射 BRDF 去掉 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_13f96b5049ec4e.png)/π 因子后的形式，因为公式（9.68）已在外部乘上这个因子。Hammon 讨论了“混合”BRDF，即以其他光滑表面漫反射 BRDF 替换 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_09_11c6b519c5c557.png)，从而提高性能，或改善与基于旧模型制作的资源之间的兼容性。

总体而言，Hammon 的漫反射 BRDF 成本低，并建立在可靠的理论基础上，尽管他没有展示与实测数据的比较。需要留意的是，表面凹凸尺寸大于散射距离这一假设，是该 BRDF 推导的根本前提，因此可能限制它能够准确建模的材料类型。见图 9.40。

许多实时渲染应用仍然实现了公式（9.61）所示的简单朗伯项。除了计算成本低以外，与其他漫反射模型相比，朗伯项更容易用于间接光照和烘焙光照；而且它与更复杂模型之间的视觉差异往往很细微 [251, 861]。尽管如此，对照片级真实感的持续追求，仍推动着更准确模型的日益广泛使用。


## 9.10 布料的 BRDF 模型

本节来源：原书第 356—359 页（PDF 第 377—380 页）。包含 9.10.1—9.10.3；图 9.42 在原版中排于本节标题之前，属于本节，随文收入。

布料的微观几何结构往往不同于其他类型的材料。根据织物的种类，它可能具有高度重复的编织微结构、垂直突出于表面的圆柱体（纱线），或兼有两者。因此，布料表面具有独特的外观，通常需要专门的着色模型来表现，例如各向异性的镜面高光、微突起散射（asperity scattering）[919]（光穿过突出于表面的半透明纤维并发生散射所造成的明亮边缘效果），甚至还有随观察方向变化的颜色偏移（由织物中穿行的不同颜色纱线造成）。

除了 BRDF，大多数织物还具有高频的空间变化，这也是营造可信布料外观的关键 [825]。见图 9.42。


![图9.42 布料着色及逐像素变化](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_10_9.42.png)

图 9.42：使用为游戏《神秘海域 4》构建的布料系统制作的一种材质。左上方球体使用标准 BRDF，包含 GGX 微表面镜面反射和朗伯漫反射。上方中间的球体使用织物 BRDF。其余球体各自加入一种不同的逐像素变化，按从左到右、从上到下的顺序分别为：织物编织细节、织物老化、瑕疵细节和细小褶皱。（UNCHARTED 4 A Thief’s End ©/TM 2016 SIE。由 Naughty Dog LLC 创作及开发。）

布料 BRDF 模型主要分为三类：根据观察建立的经验模型、基于微表面理论的模型，以及微圆柱模型。下面将介绍每一类中的一些重要实例。

### 9.10.1 经验布料模型

在游戏《神秘海域 2》[631] 中，布料表面使用如下漫反射 BRDF 项：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_78e7ad9186b615.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_8587b005d1f146.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_7c030f16b3ab49.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_e30e0cc6402d99.png) 分别是用户控制的缩放因子，对应于轮廓光项、提亮正对观察者的（内部）表面的项，以及朗伯项。此外，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_415f5cdebca985.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_ea66ee9125a945.png) 控制轮廓项与内部项的衰减。这种行为不符合物理规律，因为其中有多种依赖观察方向的效果，却没有依赖光照方向的效果。

> 译注：原书式 9.70 中，带 rim 下标的项在观察方向与法线对齐时较大，带 inner 下标的项则在掠射观察方向较大，与紧接公式的文字命名似乎相反。这里保留原式及原文解释，不擅自交换两项。

相比之下，《神秘海域 4》[825] 中的布料根据织物种类，在镜面反射项中采用微表面模型或微圆柱模型（详见随后两小节），并在漫反射项中采用一种“环绕光照”的经验次表面散射近似：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_da0babf57a74c6.png)


这里使用第 1.2 节引入的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_62bd502285dddd.png) 记号，表示将数值钳制在 0 与 1 之间。这个不常见的记法 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_36c47fafb1f913.png) 表明，此模型不仅影响 BRDF，也影响光照。箭头右侧的项替代左侧的项。用户指定的参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_ded9fa41a7f867.png) 是散射颜色；数值 w 的范围为 [0, 1]，控制环绕光照的宽度。

为了对布料建模，Disney 在其漫反射 BRDF 项 [214]（第 9.9.4 节）上添加一个柔光（sheen）项，以模拟微突起散射：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_9fe24f01faee70.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_e4880c24f648fa.png) 是调节柔光项强度的用户参数。柔光颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_489036a8ded182.png) 是白色与按亮度归一化的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_54cc8781620e9a.png) 之间的混合值，混合比例由另一个用户参数控制。换句话说，将 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_54cc8781620e9a.png) 除以其亮度，以单独取出色相和饱和度。

### 9.10.2 微表面布料模型

Ashikhmin 等人 [78] 提出使用反转高斯法线分布函数（NDF）来模拟天鹅绒。后续工作 [81] 对该 NDF 做了小幅修改，同时还提出了微表面 BRDF 的一种变体，用于一般材料的建模；该变体没有遮蔽—阴影项，并修改了分母。

游戏《教团：1886》[1266] 使用的布料 BRDF，将修改后的微表面 BRDF、Ashikhmin 与 Premože 后来报告 [81] 中天鹅绒 NDF 的推广形式，以及式 9.63 的漫反射项结合起来。推广后的天鹅绒 NDF 为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_6d1681f9cdcfe0.png)


其中，α 控制反转高斯函数的宽度，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_4382f5cc4fbdfd.png) 控制其幅度。完整的布料 BRDF 为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_d5c390b89d9d81.png)


这一 BRDF 的一个变体被用于游戏《神秘海域 4》[825] 中，以表现羊毛和棉布等粗糙织物。

Imageworks [947] 为柔光项采用了另一种反转 NDF，这一柔光项可以添加到任意 BRDF 上：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_10_7e386eb973bee5.png)


虽然针对这一 NDF 的 Smith 遮蔽—阴影函数没有闭式解，但 Imageworks 用一个解析函数近似了数值解。Estevez 与 Kulla [442] 讨论了遮蔽—阴影函数的细节，以及柔光项与 BRDF 其余部分之间的能量守恒问题。图 9.43 展示了一些使用 Imageworks 柔光项渲染的实例。


![图9.43 Imageworks柔光项的不同粗糙度](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_10_9.43.png)

图 9.43：将 Imageworks 的柔光镜面反射项添加到红色漫反射项上。从左到右，柔光粗糙度的取值分别为 α = 0.15、0.25、0.40、0.65 和 1.0。（图片由 Alex Conty 提供 [442]。）

目前介绍过的每一种布料模型，都仅限于特定的织物种类。下一小节讨论的模型试图以更一般的方式对布料建模。

### 9.10.3 微圆柱布料模型

用于布料的微圆柱模型与用于毛发的模型十分相似，因此，第 14.7.2 节关于毛发模型的讨论可提供更多背景。这些模型背后的思路是，假设表面覆盖着一维线条。Kajiya 与 Kay 针对这种情况开发了一个简单的 BRDF 模型 [847]，Banks [98] 则为其建立了坚实的理论基础。它既被称为 Kajiya-Kay BRDF，也被称为 Banks BRDF。其概念基于以下观察：由一维线条组成的表面，在任意给定位置都有无穷多个法线，这些法线由该位置处垂直于切向量 **t** 的法平面定义。尽管从这一框架发展出了许多较新的微圆柱模型，最初的 Kajiya-Kay 模型因其简单，仍然得到一些应用。例如，在游戏《神秘海域 4》[825] 中，Kajiya-Kay BRDF 被用作丝绸和天鹅绒等光亮织物的镜面反射项。

Dreamworks [348, 1937] 为织物采用了一个相对简单且可由美术人员控制的微圆柱模型。纹理可用于改变粗糙度、颜色和纱线方向；纱线可以指向表面平面之外，以模拟天鹅绒及类似织物。可以为经纱和纬纱设置不同参数，从而模拟闪色绸等颜色变化复杂的织物。该模型经过归一化，以满足能量守恒。

Sadeghi 等人 [1526] 根据对织物样本和单根纱线的测量，提出了一个微圆柱模型。该模型还考虑了纱线之间的遮蔽与阴影。

在某些情况下，实际的毛发 BSDF 模型（第 14.7 节）也用于布料。RenderMan 的 PxrSurface 材质 [732] 具有一个“绒毛”（fuzz）波瓣，它使用了 Marschner 等人 [1128] 的毛发模型（第 14.7 节）中的 R 项。Wu 与 Yuksel [1924, 1926] 的实时布料渲染系统所实现的模型中，有一个派生自 Disney 用于动画电影的毛发模型 [1525]。


## 9.11 波动光学 BRDF 模型

本节来源：《Real-Time Rendering, Fourth Edition》书页 359—363（PDF 物理页 380—384）。范围自 9.11 节标题起，至 9.12 节标题前，包含 9.11.1、9.11.2 及图 9.44—9.47。

前面几节讨论的模型都依赖几何光学；几何光学将光视为沿光线传播，而不是以波的形式传播。如书页 303 所述，几何光学建立在这样一个假设之上：表面的任何不规则起伏，要么小于一个波长，要么大于约 100 个波长。

现实世界中的表面可不会如此配合。它们往往在各种尺度上都有不规则起伏，包括 1—100 个波长这一范围。我们把这种尺寸的不规则起伏称为**纳米几何**（nanogeometry），以区别于前面几节讨论的**微几何**（microgeometry）起伏；后者虽然小到无法逐一渲染，却仍大于光的 100 个波长。几何光学无法建模纳米几何对反射率的影响。这些效应取决于光的波动性，必须使用**波动光学**（wave optics，也称 physical optics，即物理光学）来建模。

厚度接近光的一个波长的表面层，也就是薄膜，同样会产生与光的波动性有关的光学现象。

本节将简要介绍衍射和薄膜干涉等波动光学现象，并讨论它们对于真实感渲染的重要性；这种重要性有时令人惊讶，因为所渲染的材质在其他方面看起来可能相当普通。

### 9.11.1 衍射模型


![图 9.44 惠更斯–菲涅耳原理、边缘衍射与平面反射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_11_9.44.png)

**图 9.44。** 左图中，平面波前在空旷空间中传播。如果把波前上的每一点都视为一个新球面波的波源，这些新波就会在除前进方向以外的所有方向上发生相消干涉，从而再次形成平面波前。中图中，波遇到了障碍物。障碍物边缘处的球面波，其右侧没有其他波与之发生相消干涉，因此有些波会绕过边缘发生衍射，也就是从边缘“漏”过去。右图中，平面波前从平坦表面反射。平面波前先到达左侧的表面点，再到达右侧的表面点，因此左侧表面点发出的球面波有更长的传播时间，尺寸也就更大。这些大小不同的球面波前沿反射平面波前的边缘发生相长干涉，在其他方向上则发生相消干涉。

纳米几何会引起一种称为**衍射**（diffraction）的现象。为了解释它，我们需要用到**惠更斯–菲涅耳原理**：波前，也就是具有相同波相位的点所组成的集合，其上的每一点都可以视为一个新球面波的波源。见图 9.44。当波遇到障碍物时，惠更斯–菲涅耳原理表明，它们会在拐角处稍稍弯曲、绕行，这就是衍射的一个例子。几何光学无法预测这种现象。对于入射到平面表面上的光，几何光学确实能正确预测光会沿单一方向反射。不过，菲涅耳–惠更斯原理还能提供进一步的认识：表面上的球面波恰好排列成能够产生反射波前的形式，而其他所有方向上的波都通过相消干涉被消除。当我们考察具有纳米级不规则起伏的表面时，这一认识就十分重要。由于各个表面点的高度不同，表面上的球面波不再排列得那么整齐。见图 9.45。


![图 9.45 纳米几何引起的镜面反射与衍射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_11_9.45.png)

**图 9.45。** 左图显示平面波前入射到具有粗糙纳米几何的表面上。中图显示依据菲涅耳–惠更斯原理在表面上形成的球面波。右图显示，在发生相长和相消干涉之后，产生的波中有一部分（红色）形成了平面反射波。其余的波（紫色）发生衍射；在各个方向上传播的光量不同，并且取决于波长。

如图所示，光散射到不同方向。其中一部分发生镜面反射，即在反射方向上叠加成一个平面波前。其余光则按照某种方向分布衍射出去，这种分布取决于纳米几何的某些性质。镜面反射光与衍射光之间的分配取决于纳米几何凸起的高度；更准确地说，取决于高度分布的方差。衍射光在镜面反射方向周围的角度扩展范围，取决于纳米几何凸起相对于光波长的宽度。有一点不太符合直觉：越宽的起伏，造成的扩展范围反而越小。如果不规则起伏的尺寸大于光的 100 个波长，衍射光与镜面反射光之间的夹角就会小到可以忽略。起伏尺寸越小，衍射光的扩展范围越大，直到起伏小于光的一个波长；此时就不再发生衍射。

在具有周期性纳米几何的表面上，衍射最为明显，因为重复图案通过相长干涉增强了衍射光，从而产生色彩丰富的虹彩。这种现象可以在 CD、DVD 光盘以及某些昆虫身上观察到。虽然非周期性表面也会发生衍射，但计算机图形学界多年来一直认为这种效应很微弱。因此，多年以来，除了少数例外 [89, 366, 686, 1688]，计算机图形学文献大多忽略了衍射。

然而，Holzschuch 和 Pacanowski [762] 最近对实测材质的分析表明，许多材质中都存在显著的衍射效应；这或许能够解释为什么使用现有模型拟合这些材质始终很困难。同一组作者的后续工作 [763] 提出了一种结合微表面理论与衍射理论的模型：它采用通用微表面 BRDF（公式 9.26），并以一个考虑衍射的微 BRDF 作为其中的组成部分。与此同时，Toisoul 和 Ghosh [1772, 1773] 提出了捕获周期性纳米几何所产生的虹彩衍射效应的方法，以及在点光源和基于图像的光照下实时渲染这些效应的方法。

### 9.11.2 薄膜干涉模型

**薄膜干涉**是一种波动光学现象：从薄电介质层的顶面和底面反射的光路相互干涉，便会产生这种现象。见图 9.46。


![图 9.46 薄膜中的多条相干光路](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_11_9.46.png)

**图 9.46。** 光入射到反射性基底上方的一层薄膜。除了初次反射以外，还存在多条光路：光先折射进入薄膜，再从基底反射，随后或者在薄膜顶面内侧反射，或者折射穿出该表面。这些光路都是同一个波的副本，只因路径长度不同而产生了短暂的相位延迟，因此会相互发生相干干涉。

不同波长的光会发生相长干涉或相消干涉，具体取决于波长与路径长度差之间的关系。由于路径长度差会随角度而变化，不同波长便会在相长干涉与相消干涉之间转换，最终表现为虹彩般的颜色变化。

这种效应之所以要求膜足够薄，与**相干长度**（coherence length）的概念有关。相干长度是一个光波的副本仍能与原波发生相干干涉时，所允许的最大位移距离。该长度与光的**带宽**成反比；这里的带宽是指其光谱功率分布（SPD）所覆盖的波长范围。激光的带宽极窄，因此相干长度极长；根据激光类型的不同，相干长度可能达到数英里。这种关系很好理解，因为一个简单的正弦波即使平移许多个波长，仍然能与原波发生相干干涉。如果激光是真正的单色光，它就会具有无限长的相干长度；但在实际中，激光的带宽并不为零。反过来，带宽极宽的光会具有杂乱的波形。这样的波形副本只需平移很短的距离，就不再与原波发生相干干涉，这也很容易理解。

理论上，理想白光是所有波长的混合，其相干长度应为零。不过，就可见光光学而言，决定相干长度的是人类视觉系统的带宽；人眼只能感知 400—700 nm 范围内的光，因此相干长度约为 1 微米。所以，在大多数情况下，“薄膜增厚到什么程度后就不再引起可见干涉？”这个问题的答案是“约 1 微米”。

与衍射类似，多年以来，人们一直把薄膜干涉看作一种特殊情形下的效应，认为它只发生在肥皂泡、油渍等表面上。然而，Akin [27] 指出，薄膜干涉确实会为许多日常表面带来细微的色彩，并展示了对这种效应进行建模如何增强真实感。见图 9.47。他的文章使人们对基于物理的薄膜干涉的兴趣大幅增加，包括 RenderMan 的 PxrSurface [732] 和 Imageworks 着色模型 [947] 在内的多种着色模型，都加入了对这种效应的支持。


![图 9.47 皮革材质的薄膜干涉对比](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_11_9.47.png)

**图 9.47。** 左图为未考虑薄膜干涉的皮革材质渲染，右图为考虑了薄膜干涉的渲染。薄膜干涉造成的镜面反射着色增强了图像的真实感。（图像由 Next Limit Technologies 的 Atilla Akin 制作 [27]。）

适用于实时渲染的薄膜干涉技术已经存在一段时间了。Smits 和 Meyer [1667] 提出了一种高效方法，用来考虑一阶与二阶光路之间的薄膜干涉。他们观察到，得到的颜色主要是路径长度差的函数，而路径长度差可以根据薄膜厚度、观察角度和折射率高效计算。他们的实现需要一个存储 RGB 颜色的一维查找表。表中的内容可以通过密集的光谱采样计算，并在预处理阶段转换为 RGB 颜色，因此这项技术相当快速。游戏《使命召唤：无限战争》（Call of Duty: Infinite Warfare）采用了另一种快速的薄膜近似方法，作为其分层材质系统的一部分 [386]。这些技术没有建模光在薄膜中的多次弹射，也没有建模其他一些物理现象。Belcour 和 Barla [129] 提出了一种更精确、计算代价也更高，但仍以实时实现为目标的技术。


## 9.12 分层材质

来源：原书第 363—365 页（PDF 第 384—386 页）；范围为 9.12 节正文及图 9.48，止于 9.13 节标题之前。

在现实生活中，材质往往一层叠在另一层之上。表面可能覆有灰尘、水、冰或雪；也可能出于装饰或保护的目的，涂有漆或其他涂层；还可能像许多生物材料一样，其基本构造本身就包含多个层次。

分层中最简单、视觉影响也最显著的情形之一是透明涂层（clear coat），即覆盖在另一种材质的基底之上的光滑透明层。例如，在粗糙木材表面涂上一层光滑的清漆。迪士尼原则化着色模型 [214] 包含透明涂层项，虚幻引擎 [1802]、RenderMan 的 PxrSurface 材质 [732]，以及 Dreamworks Animation [1937] 和 Imageworks [947] 等公司的着色模型也都如此。

透明涂层最显著的视觉效果，是光分别从透明涂层和下方基底反射所产生的双重反射。当基底是金属时，第二重反射最为明显，因为这时电介质透明涂层与基底之间的折射率差异最大。当基底是电介质时，其折射率接近透明涂层的折射率，因此第二重反射相对较弱。这种效果与第 325 页表 9.4 所示的水下材质相似。

透明涂层也可以带有颜色。从物理角度看，这种着色是吸收造成的。根据比尔–朗伯定律（第 14.1.2 节），被吸收的光量取决于光在透明涂层内传播的路径长度。这一长度取决于观察方向和光照方向的角度，以及材质的折射率。较简单的透明涂层实现，例如迪士尼原则化模型和虚幻引擎中的实现，没有模拟这种与视角的相关性。另一些实现则会模拟它，例如 PxrSurface、Imageworks 和 Dreamworks 着色模型中的实现。Imageworks 模型还允许将任意数量、不同类型的层串接起来。

在一般情况下，不同层可以具有不同的表面法线。例如，流过平坦路面的一道道细流、覆盖在凹凸不平土壤之上的光滑冰层，或包裹纸板箱的褶皱塑料薄膜。电影行业使用的大多数分层模型都支持为每一层单独设置法线。这种做法在实时应用中并不普遍，不过虚幻引擎的透明涂层实现将其作为一项可选功能提供。

Weidlich 和 Wilkie [1862, 1863] 提出了一种分层微表面模型，假设层的厚度相对于微表面的尺寸很小。他们的模型支持任意数量的层，并跟踪从最上层向下到达最底层、再返回上方这一过程中发生的反射和折射。该模型足够简单，可以实时实现 [420, 573]，但没有考虑层间的多次反射。Jakob 等人 [811, 812] 提出了一个全面而精确的分层材质模拟框架，其中包含多次反射。虽然这一系统不适合实时实现，但可以用于与真实值进行比较，而且其采用的思路或许能为未来的实时技术提供启发。

游戏《使命召唤：无限战争》（Call of Duty: Infinite Warfare）使用的分层材质系统 [386] 尤其值得关注。它允许用户合成任意数量的材质层，支持层间的折射、散射和基于路径长度的吸收，也支持每一层采用不同的表面法线。结合高效的实现，该系统能够实时呈现复杂程度前所未有的材质；考虑到这是一款以 60 Hz 运行的游戏，这一点尤为令人印象深刻。见图 9.48。


![图 9.48 《使命召唤：无限战争》多层材质系统的测试表面](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_12_9_48.png)

**图 9.48** 展示《使命召唤：无限战争》多层材质系统各项特性的测试表面。尽管每一面仅由两个三角形构成，该材质却模拟出了具有畸变和散射效果的复杂几何表面。（图片由 Activision Publishing, Inc. 提供，2018 年。）


## 9.13 材质的混合与过滤

来源：《Real-Time Rendering》第 4 版，书页 365—372（PDF 物理页 386—393）；从第 9.13 节标题起，至“延伸阅读与资源”之前，包含第 9.13.1 节。章末延伸阅读另见 09.99_延伸阅读与资源.md。

材质混合是将多种材质的属性，也就是 BRDF 参数，组合起来的过程。例如，要为一张带有锈斑的金属板建模，可以绘制一张遮罩纹理来控制锈斑的位置，并利用它在铁锈与金属的材质属性之间进行混合，这些属性包括镜面反射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_d8e52c5a12790b.png)、漫反射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_54cc8781620e9a.png) 和粗糙度 α。参与混合的各个材质本身也可以随空间位置变化，其参数存储在纹理中。混合可以作为预处理来创建新的纹理，这通常称为“烘焙”；也可以在着色器中即时进行。严格来说，表面法线 **n** 并不是 BRDF 参数，但它的空间变化对外观十分重要，因此材质混合通常也包括法线贴图的混合。

材质混合对许多实时渲染应用都至关重要。例如，游戏《教团：1886》（The Order: 1886）具有一个复杂的材质混合系统 [1266, 1267, 1410]：用户可以从一个庞大的材质库中选取材质，创作任意深度的材质堆栈，并通过各种空间遮罩进行控制。大部分材质混合都作为离线预处理完成，但某些合成操作可按需要推迟到运行时。这类运行时处理通常用于环境，为平铺纹理增添独特的变化。广受欢迎的材质创作工具 Substance Painter 和 Substance Designer 使用类似的方法进行材质合成，纹理绘制工具 Mari 也是如此。

即时混合纹理元素既能节省内存，又能产生丰富多样的效果。游戏将材质混合用于多种用途，例如：

- 显示建筑物、载具以及活体（或不死）生物受到的动态损伤 [201, 603, 1488, 1778, 1822]。
- 允许用户自定义游戏中的装备与服装 [604, 1748]。
- 增加角色 [603, 1488] 和环境 [39, 656, 1038] 的视觉多样性。书页 891 的图 20.5 给出了一个例子。

有时，一种材质以低于 100% 的不透明度混合到另一种材质上；但即使是完全不透明的混合，遮罩边界处仍然会有一些像素需要进行部分混合；如果烘焙到纹理中，则对应的是纹素。无论哪种情况，严格正确的做法都是分别计算每种材质的着色模型，再混合计算结果。不过，先混合 BRDF 参数、再仅计算一次着色，要快得多。对于与最终着色颜色具有线性或近似线性关系的材质属性，例如漫反射颜色和镜面反射颜色参数，这种插值几乎不会引入误差，或者完全没有误差。在许多情况下，即便参数与最终着色颜色之间具有高度非线性的关系，例如镜面反射粗糙度，遮罩边界处产生的误差也不会令人难以接受。

法线贴图的混合需要特别考虑。将这一过程视为对生成这些法线贴图的高度图进行混合，通常可以取得良好效果 [1086, 1087]。在某些情况下，例如将细节法线贴图叠加到基础表面之上，其他混合方式更为合适 [106]。

材质过滤是一个与材质混合密切相关的话题。材质属性通常存储在纹理中，并通过 GPU 双线性过滤、mipmap 等机制进行过滤。然而，这些机制以一个假设为基础：被过滤的量，即着色方程的输入，与最终颜色，即着色方程的输出，具有线性关系。同样，某些量满足这种线性关系，但一般情况并非如此。对法线贴图，或对包含粗糙度等非线性 BRDF 参数的纹理，使用线性的 mipmap 方法可能会产生伪影。这些伪影可能表现为镜面反射走样，即闪烁的高光；也可能表现为表面与相机之间的距离改变时，表面光泽或亮度出现意外变化。在这两类问题中，镜面反射走样明显得多；用于减轻这些伪影的技术通常称为镜面反射抗锯齿技术。下面将讨论其中的几种方法。


![图 9.49：法线贴图降采样与分布拟合的比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_13_9.49.png)

**图 9.49。** 左侧圆柱使用原始法线贴图渲染。中间使用分辨率低得多的法线贴图，其中存储的是经过平均并重新归一化的法线，如图 9.50 左下所示。右侧圆柱所使用的纹理具有同样低的分辨率，但存储的是针对理想 NDF 拟合得到的法线值和光泽值，如图 9.50 右下所示。右图对原始外观的表现明显更好。在低分辨率下渲染时，这种表面也较不容易产生走样。（图片由 ILM 的 Patrick Conran 提供。）

> 译注：原书图 9.49 图注将“平均并重新归一化”的情况指向图 9.50 的“左下”。对照图 9.50 及其图注，该情况实际上位于下排中间；这里保留原文指向，并注明这一疑似排印错误。

### 9.13.1 法线与法线分布的过滤

材质过滤伪影中的绝大部分，主要是镜面反射走样，以及最常使用的相应解决方案，都与法线和法线分布函数的过滤有关。由于这一方面十分重要，我们将较深入地讨论它。

要理解这些伪影为何出现以及如何解决，首先回忆一下：NDF 是对亚像素表面结构的统计描述。当相机与表面之间的距离增加时，原本覆盖多个像素的表面结构可能会缩小到亚像素尺寸，从凹凸贴图所描述的范围转入 NDF 所描述的范围。这一转换与 mipmap 链密切相关，因为 mipmap 链封装了纹理细节缩小到亚像素尺寸的过程。

考虑如何为图 9.49 左侧圆柱这样的物体建立外观模型，以供渲染使用。外观建模总是假定一个特定的观察尺度。宏观尺度，即大尺度的几何结构，用三角形建模；中观尺度，即中等尺度的几何结构，用纹理建模；小于单个像素的微观尺度几何结构，则通过 BRDF 建模。

对于图中所示的尺度，将圆柱建模为光滑网格来表达宏观结构，并用法线贴图表达中观尺度的凹凸，是合适的。我们选择具有固定粗糙度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_f2fa0bd2ca6222.png) 的 Beckmann NDF，来描述微观尺度的法线分布。这种组合表示能够很好地描述圆柱在该尺度下的外观。但是，当观察尺度改变时，会发生什么？

请仔细观察图 9.50。上方黑框中的图表示表面的一小部分，由四个法线贴图纹素覆盖。假设我们正在某个尺度下渲染表面，使每个法线贴图纹素平均被一个像素覆盖。对于每个纹素，法线，也就是分布的平均值或均值，以红色箭头表示；其周围的黑色部分表示 Beckmann NDF。法线与 NDF 隐式指定了底层的表面结构，图中以横截面显示。中间的大隆起是法线贴图中的一个凸起，细小的起伏则是微观尺度的表面结构。法线贴图中的每个纹素与粗糙度相结合，可以看作汇集了该纹素所覆盖表面区域内的法线分布。


![图 9.50：四个法线分布的三种平均方式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_13_9.50.png)

**图 9.50。** 图 9.49 中表面的一部分。上排显示法线分布、以红色表示的平均法线，以及隐含的微观几何结构。下排显示将四个 NDF 平均成一个 NDF 的三种方式，这正是 mipmap 生成时所做的操作。左侧为真实基准，即对法线分布求平均；中间显示分别对均值（法线）和方差（粗糙度）求平均的结果；右侧显示针对平均 NDF 拟合的一个 NDF 波瓣。

现在假设相机移得离物体更远，使一个像素覆盖了法线贴图中的全部四个纹素。在这一分辨率下，理想的表面表示应准确表达每个像素所覆盖的更大表面区域内汇集的全部法线的分布。对最高分辨率 mipmap 层中四个纹素的 NDF 求平均，即可得到这一分布。左下图显示了这种理想的法线分布。如果用这一结果进行渲染，就能最准确地表现表面在较低分辨率下的外观。

下排中图显示分别对法线和粗糙度求平均的结果；法线是每个分布的均值，而粗糙度对应每个分布的宽度。结果具有正确的平均法线，以红色表示，但分布过窄。这一误差会使表面看起来过于光滑。更糟的是，由于 NDF 太窄，它容易引起走样，表现为闪烁的高光。

我们无法用 Beckmann NDF 直接表示理想的法线分布。不过，如果使用粗糙度贴图，Beckmann 粗糙度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_f2fa0bd2ca6222.png) 就可以随纹素而变化。设想对于每个理想 NDF，我们都找到一个在朝向和总体宽度上与它最接近的定向 Beckmann 波瓣。我们将该 Beckmann 波瓣的中心方向存储在法线贴图中，将其粗糙度值存储在粗糙度贴图中。结果如右下所示。这一 NDF 与理想情况接近得多。与简单地对法线求平均相比，这种处理能更忠实地表示圆柱的外观，图 9.49 展示了这一点。

为了获得最佳结果，mipmap 等过滤操作应施加于法线分布，而不是法线或粗糙度值。这意味着需要以稍有不同的方式来理解 NDF 与法线之间的关系。通常，NDF 定义在由法线贴图的逐像素法线所确定的局部切线空间中。然而，当跨越不同法线对 NDF 进行过滤时，更有用的理解方式是：法线贴图与粗糙度贴图的组合，在底层几何表面的切线空间中定义了一个偏斜的 NDF，也就是其平均法线并不竖直向上的 NDF。

早期解决 NDF 过滤问题的尝试 [91, 284, 658] 使用数值优化，将一个或多个 NDF 波瓣拟合到平均分布上。这种方法存在稳健性和速度方面的问题，如今已很少使用。相反，目前使用的大多数技术通过计算法线分布的方差来工作。Toksvig [1774] 提出了一个巧妙的观察：如果对法线求平均后不重新归一化，那么平均法线的长度与法线分布的宽度负相关。也就是说，原始法线指向不同方向的程度越大，由它们平均得到的法线就越短。他提出了根据这一法线长度修改 NDF 粗糙度参数的方法。使用修改后的粗糙度计算 BRDF，可以近似表现过滤后的法线所产生的展宽效果。

Toksvig 最初的方程针对 Blinn–Phong NDF：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_b6259f61fb01cb.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_501fa05cfd75db.png) 是原始粗糙度参数值，α′ₚ 是修改后的值，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_056a93a32dc869.png) 是平均法线的长度。由于两种 NDF 的形状非常接近，利用等价关系 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_52ddac3f20351d.png)，该方程也可以用于 Beckmann NDF；这一关系来自 Walter 等人 [1833]。将该方法用于 GGX 则没有这么直接，因为 GGX 与 Blinn–Phong 或 Beckmann 之间没有明确的等价关系。将针对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_f2fa0bd2ca6222.png) 的等价关系用于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_dbbbd023cd6e77.png)，会在高光中心得到相同的数值，但高光外观却相当不同。更令人担忧的是，GGX 分布的方差没有定义，这使得这一类基于方差的技术在用于 GGX 时，理论基础并不稳固。尽管存在这些理论困难，将式（9.76）用于 GGX 分布仍相当常见，通常采用 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_09_13_69e559e59d1c9e.png)。这种做法在实践中的效果相当不错。

Toksvig 方法的一个优点是能够考虑 GPU 纹理过滤所引入的法线方差。它还适用于最简单的法线 mipmap 生成方案，即进行线性平均而不归一化。这一特点对水面涟漪等动态生成的法线贴图尤其有用，因为这些贴图必须即时生成 mipmap。该方法对静态法线贴图则不那么理想，因为它与主流法线贴图压缩方法不能很好地配合。这些压缩方法依赖于法线具有单位长度。由于 Toksvig 方法依赖平均法线长度的变化，与该方法配合使用的法线贴图可能必须保持未压缩状态。即便如此，存储缩短后的法线仍可能带来精度问题。

Olano 和 Baker 的 LEAN 贴图技术 [1320] 基于对法线分布的协方差矩阵进行贴图。与 Toksvig 技术一样，它能很好地配合 GPU 纹理过滤与线性 mipmap 使用。它还支持各向异性的法线分布。类似于 Toksvig 方法，LEAN 贴图很适合动态生成的法线；但用于静态法线时，为了避免精度问题，它需要大量存储空间。Hery 等人 [731, 732] 独立开发了一种类似技术，并将其用于皮克斯动画电影，渲染金属薄片和细小划痕等亚像素细节。LEAN 贴图的简化变体 CLEAN 贴图 [93] 以失去各向异性支持为代价，减少了存储需求。LEADR 贴图 [395, 396] 则扩展了 LEAN 贴图，进一步考虑位移贴图对可见性的影响。

实时应用中使用的法线贴图大多是静态的，而不是动态生成的。对于这类贴图，通常采用方差贴图这一类技术。这些技术在生成法线贴图的 mipmap 链时，计算因求平均而丢失的方差。Hill [739] 指出，Toksvig 技术、LEAN 贴图和 CLEAN 贴图的数学表达形式都可以用于以这种方式预计算方差，从而消除这些技术以原始形式使用时的许多缺点。在某些情况下，预计算的方差值存储在一张独立方差纹理的 mipmap 链中。更常见的是，利用这些值修改已有粗糙度贴图的 mipmap 链。例如，《使命召唤：黑色行动》（Call of Duty: Black Ops）使用的方差贴图技术就采用了这种方法 [998]。修改后的粗糙度值通过以下过程计算：先将原始粗糙度值转换成方差值，加上法线贴图中的方差，再将结果转换回粗糙度。对于《教团：1886》，Neubelt 和 Pettineo [1266, 1267] 以类似方式使用了 Han 的一项技术 [658]。他们将法线贴图的 NDF 与其 BRDF 镜面反射项的 NDF 进行卷积，将结果转换为粗糙度，再存储到粗糙度贴图中。

付出一些额外存储空间的代价，可以分别计算纹理空间 x、y 方向上的方差，并将其存储在各向异性粗糙度贴图中，从而改善结果 [384, 740, 1823]。单独使用这项技术时，只能表示与坐标轴对齐的各向异性；这种各向异性在人造表面上很常见，在自然形成的表面上则较少。再多存储一个值，就还能支持具有任意朝向的各向异性 [740]。

与 Toksvig、LEAN 和 CLEAN 贴图的原始形式不同，方差贴图技术没有考虑 GPU 纹理过滤引入的方差。为弥补这一点，方差贴图的实现往往会用一个小型滤波器对法线贴图的最高分辨率 mip 层进行卷积 [740, 998]。组合多个法线贴图时，例如细节法线贴图 [106]，需要注意正确组合这些法线贴图的方差 [740, 960]。

法线方差既可能由法线贴图引入，也可能来自高曲率几何结构。此前讨论的技术无法减轻后者引起的伪影。另有一组方法专门处理几何法线方差。如果几何表面具有唯一的纹理映射，这在角色上很常见，在环境上则较少，就可以将几何曲率“烘焙”进粗糙度贴图 [740]。也可以使用像素着色器的导数指令即时估计曲率 [740, 857, 1229, 1589, 1775, 1823]。这一估计可在渲染几何体时进行；如果有法线缓冲区，也可在后处理通道中进行。

到目前为止讨论的方法主要关注镜面反射响应，但法线方差也会影响漫反射着色。考虑法线方差对 **n** · **l** 项的影响，有助于同时提高漫反射与镜面反射着色的准确性，因为反射积分中的两者都乘以这一因子 [740]。

方差贴图技术将法线分布近似为一个光滑的高斯波瓣。如果每个像素覆盖数十万个凹凸，使它们能够平滑地平均起来，那么这是一种合理近似。然而，在许多情况下，一个像素可能只覆盖几百或几千个凹凸，这就可能产生“闪点”外观。书页 328 的图 9.25 展示了一个例子：在这一系列球体图像中，凹凸的尺寸逐图减小。右下图显示凹凸已经小到足以平均成光滑高光时的结果；而左下图与下排中图中的凹凸虽然小于像素，却还没有小到足以平滑平均的程度。如果观察这些球体的动画渲染，带有噪声的高光就会表现为在连续帧之间忽明忽暗的闪点。

如果绘出这种表面的 NDF，其外观就会类似图 9.51 左图。随着球体运动，向量 **h** 在 NDF 上移动，经过明亮区域和黑暗区域，从而产生“闪烁”的外观。如果对这一表面使用方差贴图技术，实际上就相当于用一个类似图 9.51 右图的光滑 NDF 来近似该 NDF，因此会丢失闪烁细节。

电影行业通常通过大量超采样来解决这一问题；但这在实时渲染应用中不可行，即使在离线渲染中也不是理想做法。


![图 9.51：局部随机凹凸表面的 NDF 与光滑 Beckmann 波瓣](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_9_13_9.51.png)

**图 9.51。** 左侧是随机凹凸表面上一个小片区的 NDF，该片区每边包含几十个凹凸。右侧是宽度大致相同的 Beckmann NDF 波瓣。（图片由 Miloš Hašan 提供。）

为解决这一问题，人们已经开发出多种技术。其中一些不适合实时使用，但可能为未来研究提供方向 [84, 810, 1941, 1942]。有两种技术专为实时实现而设计。Wang 和 Bowles [187, 1837] 提出的一项技术被用于游戏《迪士尼无限 3.0》（Disney Infinity 3.0），以渲染闪烁的积雪。这项技术旨在产生可信的闪烁外观，而不是模拟某个特定 NDF。它针对雪这类闪点相对稀疏的材质。Zirr 和 Kaplanyan 的技术 [1974] 则在多个尺度上模拟法线分布，在空间和时间上都具有稳定性，并允许实现更丰富的外观。

我们没有足够篇幅涵盖关于材质过滤的大量文献，因此这里只列出几个值得注意的参考资料。Bruneton 等人 [204] 提出了一项技术，用于处理海洋表面从几何到 BRDF 各尺度上的方差，并包括环境光照。Schilling [1565] 讨论了一种类似方差贴图的技术，支持使用环境贴图进行各向异性着色。Bruneton 和 Neyret [205] 对这一领域的早期工作作了全面综述。


## 第9章 延伸阅读与资源

来源：《Real-Time Rendering》第 4 版，书页 372—373（PDF 物理页 393—394）的章末“Further Reading and Resources”；并核对 PDF 物理页 395 的未编号尾页。

McGuire 的《Graphics Codex》[1188] 和 Glassner 的《数字图像合成原理》（Principles of Digital Image Synthesis）[543, 544]，都是本章许多主题的良好参考资料。Dutré 的《全局光照纲要》（Global Illumination Compendium）[399] 的某些部分稍显过时，尤其是 BRDF 模型一节，但对于渲染数学，例如球面积分和半球面积分，它仍是很好的参考资料。Glassner 和 Dutré 的这两部著作都可以在网上免费获取。

对于想进一步了解光与物质相互作用的读者，我们推荐 Feynman 无与伦比的讲义 [469]，可在线获取。在撰写本章物理学部分时，这些讲义对我们自身的理解帮助极大。其他有用的参考资料包括 Fowles 的《现代光学导论》（Introduction to Modern Optics）[492]，这是一本简短易懂的入门教材；以及 Born 和 Wolf 的《光学原理》（Principles of Optics）[177]，这是一本无论在内容还是实际重量上都更“厚重”的书，提供了更深入的概述。Nassau 的《颜色的物理与化学》（The Physics and Chemistry of Color）[1262] 则极其全面、细致地描述了物体颜色背后的物理现象。

> 未编号尾页说明：PDF 物理页 395 无章节正文，仅有出版社标识“Taylor & Francis”（泰勒与弗朗西斯）、“Taylor & Francis Group”（泰勒与弗朗西斯出版集团）及网址 http://taylorandfrancis.com。
