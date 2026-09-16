# Real-Time Rendering 第14章 体积与半透明渲染 中文合集

> 依据用户提供的《Real-Time Rendering》第四版 PDF 完整翻译。原图配中文图注；复杂数学已排版为本地图片，简单符号直接显示。保留原书技术年代、公式编号与引用编号。

## 目录

- [14.1 光散射理论](<Real-Time_Rendering_4th_中文/第14章/14.01.md>)

- [14.2 专用体积渲染](<Real-Time_Rendering_4th_中文/第14章/14.02.md>)

- [14.3 通用体积渲染](<Real-Time_Rendering_4th_中文/第14章/14.03.md>)

- [14.4 天空渲染](<Real-Time_Rendering_4th_中文/第14章/14.04.md>)

- [14.5 半透明表面](<Real-Time_Rendering_4th_中文/第14章/14.05.md>)

- [14.6 次表面散射](<Real-Time_Rendering_4th_中文/第14章/14.06.md>)

- [14.7 头发与皮毛](<Real-Time_Rendering_4th_中文/第14章/14.07.md>)

- [14.8 统一方法](<Real-Time_Rendering_4th_中文/第14章/14.08.md>)


## 章首导言

来源：《Real-Time Rendering》第4版，书页589（PDF物理页610）；本文件仅含章首标题、题辞与14.1之前的导言。

> “凡是你想让它看起来更远的东西，都必须按比例画得更蓝；因此，如果某物的距离要远五倍，就把它画得蓝五倍。”
>
> ——列奥纳多·达·芬奇

**参与介质**（participating media）是用来描述充满粒子的体积区域的术语。顾名思义，它们是参与光传输的介质；换言之，它们通过散射或吸收影响穿过其中的光。渲染虚拟世界时，我们通常把注意力放在实体表面上：这些表面既简单，又复杂。这些表面之所以显得不透明，是因为它们由光在致密参与介质上反弹的行为所界定，例如通常使用BRDF建模的电介质或金属。我们熟悉的较低密度介质包括水、雾、水汽，甚至由稀疏分子构成的空气。介质的组成不同，它与穿行其中并在粒子上反弹的光相互作用的方式也不同；这种事件通常称为**光散射**。粒子密度可以是均匀的，例如空气或水；也可以是非均匀的，即随空间位置变化，例如云或水汽。一些常被渲染为实体表面的致密材料也具有很强的光散射，如皮肤或蜡烛的蜡。如9.1节所示，漫反射表面着色模型是微观层面光散射的结果。一切都是散射。


## 14.1 光散射理论

来源：《Real-Time Rendering》第4版，书页589—599（PDF物理页610—620）；并核对PDF物理页621的14.2边界。章首导言另存；本节止于14.1.4“几何散射”末段。

本节介绍参与介质中光的模拟与渲染。这是对9.1.1和9.1.2节讨论过的散射与吸收这两种物理现象的定量论述。许多作者已在包含多次散射的路径追踪背景下阐述了辐射传输方程[479, 743, 818, 1413]。这里我们将着重讨论**单次散射**，并建立对其工作原理的直观认识。单次散射只考虑光在构成参与介质的粒子上反弹一次。**多次散射**则跟踪每条光路上的多次反弹，因此复杂得多[243, 479]。书页646的图14.51给出了考虑与不考虑多次散射的结果。表14.1列出了散射方程中用于表示参与介质属性的符号和单位。注意，本章中许多量，例如![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png)、p、ρ、v和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_2e7534139de451.png)，都与波长有关；在实际应用中，这意味着它们是RGB量。

| 符号 | 说明 | 单位 |
| --- | --- | --- |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png) | 吸收系数 | ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_a58f0dc4a57d43.png) |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png) | 散射系数 | ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_a58f0dc4a57d43.png) |
| ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png) | 消光系数 | ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_a58f0dc4a57d43.png) |
| ρ | 反照率 | 无量纲 |
| p | 相函数 | ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_fdc38415101391.png) |

**表14.1** 用于散射和参与介质的记号。每个参数都可以依赖于波长（即RGB），从而实现有颜色的光吸收或散射。相函数的单位为球面度的倒数（8.1.1节）。

### 14.1.1 参与介质材料

有四类事件会影响沿光线穿过介质传播的辐亮度。图14.1展示了这些事件，可概括如下：

- **吸收**（由![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png)决定）：光子被介质中的物质吸收，并转变为热或其他形式的能量。
- **出散射**（由![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)决定）：光子在介质物质中的粒子上反弹，散射到当前方向之外。这一过程遵循相函数p，后者描述光反弹方向的分布。
- **发射**：介质达到高温时可能发光，例如火焰的黑体辐射。有关发射的更多细节，请参阅Fong等人的课程讲义[479]。
- **入散射**（由![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)决定）：来自任意方向的光子在粒子上反弹后，可能散射进入当前光路，对最终辐亮度作出贡献。某个给定方向入散射的光量还取决于该光照方向对应的相函数p。


![图14.1 四种介质事件](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.1.png)

**图14.1** 不同事件会改变参与介质中沿方向d的辐亮度。

总之，向一条路径中添加光子取决于入散射![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)和发射；从路径中移除光子则取决于**消光**![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)，它同时表示吸收与出散射。按照辐射传输方程的解释，这组系数表示位置𝐱处、朝向𝐯的辐亮度相对于L(𝐱, 𝐯)的导数。因此，这些系数的取值均在[0, +∞]范围内。详情参见Fong等人的讲义[479]。散射系数与吸收系数决定介质的**反照率**ρ，其定义为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_22ad60471d9cc0.png)


它表示在所考虑的每个可见光谱范围内，介质散射相对于吸收的重要程度，也就是介质总体的反射能力。ρ的取值在[0, 1]内。接近0表示大部分光被吸收，介质因而显得暗浊，例如深色的废气烟雾；接近1则表示大部分光被散射而不是被吸收，介质因而显得较明亮，例如空气、云或地球大气。

如9.1.2节所述，介质的外观由散射和吸收属性共同决定。真实世界中参与介质的系数值已经得到测量并发表[1258]。例如，牛奶具有很高的散射值，因此呈现浑浊、不透明的外观。牛奶还有很高的反照率ρ > 0.999，所以显得白。另一方面，红葡萄酒几乎不发生散射，却具有很强的吸收，因此呈现半透明、有颜色的外观。请观察图14.2中的液体渲染结果，并与书页301图9.8中的液体照片比较。


![图14.2 葡萄酒与牛奶](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.2.png)

**图14.2** 渲染的葡萄酒与牛奶，分别表现了不同浓度下的吸收与散射。（图片由Narasimhan等人提供[1258]。）

这些属性与事件都依赖于波长。这意味着在某个给定介质中，不同频率的光被吸收或散射的概率可能不同。理论上，为了考虑这一点，我们应当在渲染中使用光谱值。为提高效率，实时渲染使用RGB值代替；离线渲染除少数例外[660]之外也如此。如果条件允许，应使用颜色匹配函数（8.1.3节），根据光谱数据预先计算![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)等量的RGB值。

前面的章节没有参与介质，因此我们可以假设进入相机的辐亮度与离开最近表面的辐亮度相同。更确切地说，我们曾在书页310假设![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_e6967a54951c4e.png)，其中𝐜是相机位置，𝐩是视线与最近表面的交点，𝐯是从𝐩指向𝐜的单位视向量。

引入参与介质后，这个假设便不再成立，我们需要考虑沿视线发生的辐亮度变化。作为示例，下面介绍计算点状光源散射光所涉及的运算；点状光源是指用单个无穷小点表示的光源（9.4节）：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_308ebb0f691a3e.png)


其中![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_e69474311ec9fa.png)是给定点𝐱与相机位置𝐜之间的**透射率**（14.1.2节）；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_72ef858cea3f04.png)是光线上给定点𝐱处沿视线散射的光（14.1.3节）。图14.3展示了计算中的各个组成部分，后续小节将逐一解释。关于如何从辐射传输方程推导出式14.2，Fong等人的课程讲义[479]中提供了更多细节。


![图14.3 单次散射积分](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.3.png)

**图14.3** 点状光源的单次散射积分示意图。沿视线的采样点用绿色表示，其中一个点的相函数用红色表示，不透明表面S的BRDF用橙色表示。这里，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_de573840d41b7c.png)是指向光源中心的方向向量，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_1bd49f56d1bde6.png)是光源位置，p是相函数，函数v是可见性项。

### 14.1.2 透射率

透射率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_2e7534139de451.png)表示光在穿过一定距离的介质后能够通过的比例，按照下式计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f027d06ca1c13e.png)


这种关系也称为**比尔–朗伯定律**。**光学深度**τ是无量纲量，表示光衰减的程度。消光越强，或穿行距离越长，光学深度就越大，穿过介质的光也就越少。光学深度τ = 1会使约60%的光被移除。例如，如果![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png)的RGB值为(0.5, 1, 2)，则穿过深度d = 1米后的光为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_64ac396e2b5a8e.png)。图14.4展示了这一行为。透射率需要应用于：(i) 来自不透明表面的辐亮度![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_96471259e53d03.png)；(ii) 入散射事件产生的辐亮度![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_72ef858cea3f04.png)；以及(iii) 从散射事件到光源的每一条路径。在视觉上，(i) 会产生类似雾的表面遮蔽；(ii) 会遮蔽散射光，从而为介质厚度提供另一种视觉线索（见图14.6）；(iii) 会产生参与介质的体积自阴影（见图14.5）。由于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png) + ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)，透射率自然会同时受到吸收和出散射分量的影响。


![图14.4 透射率随深度变化](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.4.png)

**图14.4** 透射率随深度变化的函数，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png) = (0.5, 1.0, 2.0)。正如预期，红色分量的消光系数较低，因此有更多红光透射出来。

### 14.1.3 散射事件

对于给定位置𝐱和方向𝐯，可以按下式积分场景中各点状光源产生的入散射：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_96d75b3e322fce.png)


其中n是光源数量，p()是相函数，v()是可见性函数，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_052938a16d0fd4.png)是指向第i个光源的方向向量，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_9df419474ce7c3.png)是第i个光源的位置。此外，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_375be5f902a1f5.png)表示第i个光源的辐亮度，它是到该光源位置的距离的函数，采用9.4节中的定义及5.2.2节中的平方反比衰减函数。可见性函数![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_0ba28f8bf5443b.png)表示从位置![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_9df419474ce7c3.png)的光源到达位置𝐱的光的比例，按照下式计算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_4b2eb4e9e654fc.png)


其中![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_031cf5a2944dcc.png)。在实时渲染中，阴影来自两类遮挡：不透明遮挡和体积遮挡。不透明物体产生的阴影（shadowMap）传统上使用阴影映射或第7章中的其他技术计算。


![图14.5 体积阴影](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.5.png)

**图14.5** 由参与介质构成的斯坦福兔子的体积阴影示例[744]。左：没有体积自阴影；中：有自阴影；右：向场景中的其他元素投下阴影。（模型由斯坦福计算机图形学实验室提供。）

式14.5中的体积阴影项![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_7bff6d0731922a.png)表示从光源位置![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_9df419474ce7c3.png)到采样点𝐱的透射率，取值范围为[0, 1]。体积产生的遮挡是体积渲染的关键组成部分，体积元素既可以产生自阴影，也可以向场景中的其他元素投射阴影。见图14.5。通常先沿着从眼睛穿过体积、到达第一个表面的主光线执行**光线步进**，然后从每个采样点出发，沿着指向每个光源的次级光线路径执行光线步进。“光线步进”是指使用n个采样点对两点之间的路径进行采样，并沿途积分散射光和透射率。关于这种采样方法的更多细节，参见6.8.1节，那里将其用于高度场渲染。对于三维体积，光线步进的方法类似：每条光线逐步前进，在沿途的每个点采样体积材质或光照。参见图14.3，其中主光线的采样点用绿色表示，次级阴影光线用蓝色表示。许多其他文献也详细介绍了光线步进[479, 1450, 1908]。

由于光线步进的复杂度为O(![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_09c317452584b9.png))，其中n为每条路径上的采样数，因此其开销会迅速增大。为了在质量和性能之间进行折中，可以使用专门的体积阴影表示技术，存储从光源出发的各个方向上的透射率。本章后面的相应章节将介绍这些技术。

为了直观理解介质内部的光散射与消光行为，考虑![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png) = (0.5, 1, 2)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png) = (0, 0, 0)的情况。当介质内部的光路较短时，入散射事件将胜过消光；在本例中，消光就是出散射，且深度较小时![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_2e7534139de451.png) ≈ 1。材质会显蓝色，因为蓝色通道的![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)值最高。光在介质中穿得越深，消光使能够通过的光子越少。这时，由消光形成的透射颜色开始占主导。这可以用以下事实解释：由于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f883b72ca81859.png) = (0, 0, 0)，所以![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png) = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)。因此，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_f82b0429092cba.png)作为光学深度![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_6f614c66c02292.png)的函数，其下降速度要比用式14.2对散射光作线性积分时快得多。在本例中，红光通道穿过介质时受到的消光较少，因为该通道的![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_672ed2e5eb107d.png)值最低，因此红色会占主导。图14.6展示了这种行为，这也正是大气和天空中发生的现象。太阳高悬时（例如光沿垂直于地面的方向穿过大气，光路较短），蓝光散射更多，赋予天空自然的蓝色。然而，当太阳位于地平线附近时，光在大气中的路径很长，更多红光被透射，天空便显得更红。这就形成了我们熟悉的美丽日出与日落过渡。关于大气的物质组成，14.4.1节有更详细的介绍。这一效应的另一个例子是书页299图9.6右侧的乳光玻璃。


![图14.6 介质浓度增加](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.6.png)

**图14.6** 随介质浓度增加而变化的斯坦福龙。从左到右：0.1、1.0和10.0，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png) = (0.5, 1.0, 2.0)。（模型由斯坦福计算机图形学实验室提供。）

### 14.1.4 相函数

参与介质由半径不同的粒子组成。粒子的大小分布会影响光相对于其向前传播方向、朝某个给定方向散射的概率。9.1节解释了这种行为背后的物理原理。

在计算入散射时，使用**相函数**便可在宏观层面描述散射方向的概率与分布，如式14.4所示。图14.7展示了这一点。红色相函数采用参数θ，表示蓝色光的向前传播路径与绿色方向𝐯之间的夹角。注意这个相函数示例中的两个主要波瓣：在光路相反方向上的较小后向散射波瓣，以及较大的前向散射波瓣。相机B位于较大的前向散射波瓣方向，因此与相机A相比，它会接收到多得多的散射辐亮度。为了满足能量守恒、保持能量不增不减，相函数在单位球面上的积分必须等于1。


![图14.7 相函数的方向影响](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.7.png)

**图14.7** 相函数（红色）及其对散射光（绿色）的影响随θ变化的示意图。

相函数会根据到达某点的各方向辐亮度信息，改变该点的入散射。最简单的函数是**各向同性**的：光向所有方向均匀散射。这种理想而不现实的行为表示为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_8971ff424cd751.png)


其中θ是入射光方向与出散射方向之间的夹角，4π是单位球面的面积。

基于物理的相函数依赖于粒子的相对大小![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_b4a8043168a085.png)，其定义为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_07b848bc2d2836.png)


其中r为粒子半径，λ为所考虑的波长[743]：

- 当![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_b4a8043168a085.png) ≪ 1时，发生**瑞利散射**（例如空气）。
- 当![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_b4a8043168a085.png) ≈ 1时，发生**米氏散射**。
- 当![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_b4a8043168a085.png) ≫ 1时，发生**几何散射**。

#### 瑞利散射

瑞利勋爵（1842—1919）推导出了空气分子光散射的表达式。这些表达式的应用之一，是描述地球大气中的光散射。该相函数有两个波瓣，如图14.8所示，相对于光的方向分别称为**后向散射**与**前向散射**。函数在角度θ处求值，θ是入射光与出散射方向之间的夹角。该函数为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_888cb2bf398e47.png)


![图14.8 瑞利相函数极坐标图](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.8.png)

**图14.8** 瑞利相函数随θ变化的极坐标图。光从左侧水平入射，图中给出了对应角度θ的相对强度，θ从x轴起逆时针测量。前向散射与后向散射的概率相同。

瑞利散射高度依赖波长。将瑞利散射的散射系数![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png)看作光波长λ的函数时，它与波长的四次方成反比：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_e2aa7554b0b30b.png)


这种关系意味着，短波长的蓝光或紫光比长波长红光受到的散射强得多。可以用光谱颜色匹配函数（8.1.3节），把式14.9给出的光谱分布转换为RGB：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_c1ef88f95d5338.png) = (0.490, 1.017, 2.339)。这个值经过归一化，其亮度为1，应根据所需散射强度进行缩放。蓝光在大气中受到更强散射所产生的视觉效果，将在14.4.1节解释。

#### 米氏散射

米氏散射[776]是一种在粒子大小与光波长大致相当时可采用的模型。这种散射不依赖波长。MiePlot软件可用于模拟这一现象[996]。特定粒子大小所对应的米氏相函数，通常是具有强烈而尖锐方向波瓣的复杂分布；也就是说，相对于光子的传播方向，光子有很高概率散射到某些特定方向。为体积着色计算这类相函数的代价很高，但所幸很少需要这样做。介质通常具有连续的粒径分布。对这些不同大小所对应的米氏相函数求平均，会得到整个介质平滑的平均相函数。因此，可以用相对平滑的相函数表示米氏散射。

> 译注：本段“不依赖波长”忠实保留原书的概括性陈述。

为此常用的一种相函数是**Henyey–Greenstein（HG）相函数**，它最初被提出用于模拟星际尘埃中的光散射[721]。这一函数无法捕捉所有真实散射行为的复杂性，但可以很好地匹配相函数中的一个波瓣[1967]，即朝向主要散射方向的波瓣。它可用于表示各类烟、雾或尘埃状参与介质。这类介质可能表现出很强的后向或前向散射，在光源周围形成很大的视觉光晕。例如雾中的聚光灯，以及朝向太阳的云边缘上强烈的银边效果。

HG相函数可以表示比瑞利散射更复杂的行为，计算式为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_caae743a615eb9.png)


它可以呈现各种形状，如图14.9所示。参数g可以表示后向散射（g < 0）、各向同性散射（g = 0）或前向散射（g > 0），g的取值范围是[−1, 1]。图14.10展示了采用HG相函数的散射结果示例。


![图14.9 HG与Schlick相函数](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.9.png)

**图14.9** Henyey–Greenstein（蓝色）与Schlick近似（红色）相函数随θ变化的极坐标图。光从左侧水平入射。参数g从0增大到0.3、0.6，在右侧形成强波瓣，意味着光将更多地沿着其从左到右的前进路径散射。


![图14.10 HG参数对兔子的影响](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_1_14.10.png)

**图14.10** 由参与介质构成的斯坦福兔子，展示了HG相函数的影响，g的变化对应从各向同性散射到强前向散射。从左到右：g = 0.0、0.5、0.9、0.99和0.999。下排使用的参与介质密度是上排的十倍。（模型由斯坦福计算机图形学实验室提供。）

若要更快地得到与Henyey–Greenstein相函数相似的结果，可以使用Blasi等人[157]提出的近似；它通常以第三位作者命名，称为**Schlick相函数**：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_8ec2d631e25625.png)


它不包含复杂的幂函数，只有平方运算，因此求值快得多。为了将该函数映射到原始HG相函数，需要由g计算参数k。对于g值恒定的参与介质，这一步只需执行一次。从实际应用角度看，Schlick相函数是一种出色的、满足能量守恒的近似，如图14.9所示。

> 译注：原书式14.11的分母为“1 + k cos θ”，且给出正号映射k ≈ 1.55g − 0.![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_01_1b523b0f355f7c.png)；此处按原页保留。在本节相同θ定义下，这与式14.10中正g的前向波瓣方向存在符号一致性疑点，应用时应核查方向约定，不能直接把两式视为同向匹配。

还可以混合多个HG或Schlick相函数，表示更复杂的一般相函数[743]。这样便能表示同时具有强前向散射波瓣和强后向散射波瓣的相函数，类似于云的行为，14.4.2节将对此进行描述并给出图示。

#### 几何散射

当粒子明显大于光的波长时，便会发生几何散射。在这种情况下，光可以在每个粒子内部折射和反射。要在宏观层面模拟这种行为，可能需要复杂的散射相函数。光的偏振也会影响这种散射。例如，现实中的彩虹视觉效应就是一个例子：它由光在空气中水粒子内部的内反射引起，使太阳光在所产生的后向散射中，于很小的视角范围（约3度）内分散成可见光谱。可以使用MiePlot软件[996]模拟这样复杂的相函数。14.4.2节介绍了这类相函数的一个示例。


## 14.2 专用体积渲染

本节来源：《Real-Time Rendering, Fourth Edition》书页600—604（PDF物理页621—625）；已核对PDF物理页626的14.3节起始边界。包含全部下级小节及原页页首的本节图14.11。

本节介绍以基础且有限的方式渲染体积效果的算法。有些人甚至会说，这些是老派技巧，往往依赖临时设计的模型。之所以仍然使用它们，是因为它们依旧很有效。

### 14.2.1 大尺度雾

雾可以近似为一种基于深度的效果。最基本的形式是根据到相机的距离，将雾的颜色通过alpha混合叠加到场景上，通常称为“深度雾”。这种效果向观看者提供视觉线索。首先，它可以提高真实感并强化戏剧效果，如图14.11所示。其次，它是一种重要的深度线索，帮助场景观看者判断物体有多远，见图14.12。第三，它可以用作一种遮挡剔除方式。如果物体因为距离太远而被雾完全遮住，就可以放心地跳过这些物体的渲染，从而提高应用程序的性能。


![图14.11 雾用于强化氛围](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_2_14.11.png)

图14.11：利用雾强化氛围。（图片由NVIDIA Corporation提供。）

表示雾量的一种方式，是用区间[0, 1]内的f表示透射率，即f = 0.1意味着背景表面有10%可见。假设表面的输入颜色为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_c884fba8655644.png)，雾的颜色为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_6bb776b2d9dc21.png)，那么最终颜色![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_40848f2eb71fad.png)由下式确定：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_3b056d3861105b.png)


f的值可以用许多不同方式计算。可以使用下式，使雾线性增加：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_a9e8ea528ee9c4.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_4ea010cc58a026.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_279d3af61dd776.png)是用户参数，决定雾开始和结束的位置（即完全被雾覆盖的位置）；![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_9885dc1fd97f92.png)是从观察者到待计算雾的表面的线性深度。一种物理上准确的雾透射率计算方式，是使其随距离呈指数变化，从而遵循描述透射率的比尔–朗伯定律（第14.1.2节）。使用下式即可实现这一效果：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_b22113ed6c52e4.png)


其中，标量![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_e8bf00b47a76e9.png)是控制雾密度的用户参数。这种传统的大尺度雾，是对大气内部光散射与吸收一般模拟的粗略近似（第14.4.1节），但在当今游戏中仍能产生很好的效果，见图14.12。

> 译注：原文此处说透射率随距离“increase exponentially”（指数增加），但式（14.14）在正雾密度下给出的是透射率随距离指数衰减。上文译为“呈指数变化”，此处明确保留这一原文表述与公式之间的疑点。


![图14.12 战地1关卡中的深度雾与高度雾](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_2_14.12.png)

图14.12：在这张DICE游戏《战地1》的关卡图像中，雾被用来呈现游玩区域的复杂性。深度雾用来表现景物的宏大尺度。右侧地面处可见的高度雾，突出了从山谷中耸立起来的大量建筑。（由DICE提供，© 2018 Electronic Arts Inc.）

旧版OpenGL和DirectX API就是这样提供硬件雾功能的。对于移动设备等硬件上的较简单用例，这些模型仍然值得考虑。许多当今的游戏依赖更高级的后处理来实现雾和光散射等大气效果。透视视图中的雾存在一个问题：深度缓冲区的值是以非线性方式计算的（第23.7节）。可以利用逆投影矩阵的数学运算，将非线性的深度缓冲区值转换回线性深度![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_9885dc1fd97f92.png) [1377]。随后就可以用像素着色器，通过一次全屏通道应用雾，从而实现更高级的结果，例如随高度变化的雾或水下着色。

高度雾表示单个具有参数化高度和厚度的参与介质薄层。对屏幕上的每个像素，密度和散射光都作为观察射线在命中表面之前穿过该薄层的距离的函数来计算。Wenzel [1871]提出了一种闭式解，用于在薄层内参与介质呈指数衰减的情况下计算f。这样做会使薄层边缘附近的雾平滑过渡。图14.12左侧的背景雾就体现了这一点。

深度雾和高度雾可以有许多变体。颜色![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_6bb776b2d9dc21.png)可以是单一颜色，可以通过观察向量对立方体贴图采样来读取，甚至可以是复杂大气散射的结果，并逐像素应用相函数，以产生随方向变化的颜色[743]。还可以用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_ac84a6ae30728e.png)将深度雾透射率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_c23dfcb90aeda0.png)与高度雾透射率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_0fdf3a2badf4fd.png)组合起来，使场景中的这两种雾交织在一起。

深度雾和高度雾都是大尺度的雾效果。人们可能希望渲染更局部的现象，例如洞穴内或墓地中几座墓穴周围彼此分离的雾区。可以用椭球或盒子等形状，在所需位置添加局部雾[1871]。这些雾元素利用各自的包围盒，按照从后往前的顺序渲染。在像素着色器中，计算观察向量与每个形状的前交点![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_e8bf00b47a76e9.png)和后交点![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_84102543a3942f.png)。采用体积深度![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_922ce92c092c6c.png)，其中![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_9885dc1fd97f92.png)是表示最近不透明表面的线性深度，就可以计算透射率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_2e7534139de451.png)（第14.1.2节），覆盖率为α = 1.0 − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_2e7534139de451.png)。随后，要叠加的散射光量![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_6bb776b2d9dc21.png)就可以计算为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_96f3e6d4405062.png)。为了支持根据网格计算的更多样化形状，Oat和Scheuermann [1308]给出了一种巧妙的单通道方法，同时计算体积中最近的进入点与最远的离开点。他们将到表面的距离![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_1038808d61f5e3.png)存入一个通道，并将1 − ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_1038808d61f5e3.png)存入另一个通道。将alpha混合模式设为保存遇到的最小值后，体积渲染结束时，第一个通道就会保存最近值![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_e8bf00b47a76e9.png)，第二个通道则保存最远值![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_84102543a3942f.png)，后者编码为1 − d，因此可以恢复d。

水是一种参与介质，因此也表现出同一类基于深度的颜色衰减。近岸海水每米的透射率约为(0.3, 0.73, 0.63) [261]，所以利用式（14.23）可以恢复出![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_672ed2e5eb107d.png) = (1.2, 0.31, 0.46)。当用不透明表面渲染深色水体时，可以在相机位于水面以下时启用雾的逻辑，而在水面以上时将其关闭。Wenzel [1871]提出了更高级的解决方案。如果相机在水下，就对散射和透射率进行积分，直到碰到实体或水面。如果相机在水面上方，则仅在水体顶面到海床实体几何之间的距离范围内进行这些积分。

### 14.2.2 简单体积光照

参与介质内的光散射可能很难计算。所幸有许多高效技术，能在很多情况下令人信服地近似这种散射。

获得体积效果的最简单方法，是渲染透明网格并将其混合到帧缓冲区上。我们将其称为泼溅法（splatting，第13.9节）。为了渲染穿过窗户、穿过茂密森林或由聚光灯产生的光柱，一种解决方案是使用与相机对齐的粒子，每个粒子带有一张纹理。每个带纹理的四边形都沿光柱方向拉伸，同时始终面向相机（圆柱约束）。

网格泼溅法的缺点是，累积大量透明网格会增加所需的内存带宽，很可能造成瓶颈，而且面向相机的带纹理四边形有时会显露出来。为解决这个问题，人们提出了利用光的单次散射闭式解的后处理技术。假设相函数在空间上均一且在球面上均匀，就可以在介质恒定的前提下，沿一条路径以正确的透射率对散射光进行积分。结果如图14.13所示。下面的GLSL着色器代码片段展示了这一技术的一种实现示例[1098]：


![图14.13 解析积分得到的体积光散射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_2_14.13.png)

图14.13：利用书页603所示代码片段中的解析积分，计算光源产生的体积光散射。假设介质均匀时，可以将它作为后处理效果应用（左）；也可以应用于粒子，假设每个粒子都是具有深度的体积（右）。（图片由Miles Macklin [1098]提供。）

```glsl
float inScattering(
vec3 rayStart, vec3 rayDir,
vec3 lightPos, float rayDistance)
{
    // 计算系数。
    vec3 q = rayStart - lightPos;
    float b = dot(rayDir, q);
    float c = dot(q, q);
    float s = 1.0f / sqrt(c - b*b);

    // 提取部分公共因子。
    float x = s * rayDistance;
    float y = s * b;
    return s * atan((x) / (1.0 + (x + y) * y));
}
```

其中，rayStart是射线起点的位置，rayDir是射线的归一化方向，rayDistance是沿射线的积分距离，lightPos是光源位置。Sun等人[1722]的方案还考虑了散射系数![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_c1ef88f95d5338.png)。它还描述了一个事实应如何影响从朗伯表面和Phong表面反弹的漫反射与镜面反射辐亮度：光在命中任何表面之前，可能已沿非直线路径发生了散射。为了考虑透射率和相函数，可以采用一种需要更多ALU运算的方案[1364]。这些模型都能有效完成各自的任务，但无法考虑深度图产生的阴影，也无法考虑非均匀参与介质。

可以依靠一种称为泛光（bloom）的技术，在屏幕空间中近似光散射[539, 1741]。将帧缓冲区模糊后，取其中一小部分比例加回自身[44]，会使每个明亮物体的辐亮度向周围扩散。这项技术通常用于近似相机镜头的不完美之处，但在某些环境中，它也能很好地近似短距离、无遮挡的散射。第12.3节更详细地介绍了泛光。

Dobashi等人[359]提出了一种渲染大尺度大气效果的方法，使用一系列平面对体积进行采样。这些平面垂直于观察方向，并从后往前渲染。Mitchell [1219]也提出用同样的方法渲染聚光灯光柱，并利用阴影贴图，使不透明物体投射体积阴影。第14.3.1节详细介绍通过泼溅切片来渲染体积的方法。


![图14.14 通过屏幕空间后处理渲染的光柱](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_2_14.14.png)

图14.14：使用屏幕空间后处理渲染的光柱。（图片由Kenny Mitchell [1225]提供。）

Mitchell [1225]以及Rohleder和Jamrozik [1507]提出了另一种在屏幕空间中工作的方法，见图14.14。它可以用来渲染太阳等远处光源产生的光柱。首先，在一个清为黑色的缓冲区中，于远平面上的太阳周围渲染一个模拟的明亮物体，并用深度缓冲测试接受未被遮挡的像素。其次，对图像施加定向模糊，使此前累积的辐亮度从太阳向外扩散。可以使用可分离滤波技术（第12.1节），分两个通道执行，每个通道使用n个样本，从而获得与![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_02_09c317452584b9.png)个样本相同的模糊结果，但渲染速度更快[1681]。最后，将最终模糊后的缓冲区加到场景缓冲区上。该技术效率很高，虽然只有屏幕上可见的光源才能产生光柱，但它能够以很小的代价带来显著的视觉效果。


## 14.3 通用体积渲染

来源：《Real-Time Rendering》第 4 版，书页 605—613（PDF 物理页 626—634）。正文范围从第 14.3 节标题起，至第 14.4 节标题前；包括延续至书页 613 的图 14.21 及完整图注。

本节介绍更加基于物理的体积渲染技术，也就是尝试表示介质材质及其与光源相互作用的技术（第 14.1.1 节）。通用体积渲染关注的是属性随空间变化的参与介质，它们通常用体素表示（第 13.10 节）；体积中的光照相互作用会产生视觉上复杂的散射与阴影现象。通用体积渲染方案还必须考虑体积与其他场景元素（例如不透明或透明表面）的正确合成。这些随空间变化的介质属性，可能来自需要在游戏环境中渲染的烟雾与火焰模拟，同时还要呈现体积光照与阴影相互作用。另一种情况是，在医学可视化等应用中，我们可能希望将实体材料表示为半透明体积。

### 14.3.1 体数据可视化

*体数据可视化*是用于显示与分析体数据的工具，这些数据通常是标量场。计算机断层成像（CT）和磁共振成像（MRI）技术可以生成用于临床诊断的人体内部结构图像。例如，一个数据集可能包含 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_8816c4eee9a2af.png) 个体素，每个位置存放一个或多个数值。这些体素数据可用于形成三维图像。体素渲染既可以显示实体模型，也可以让不同材料（例如皮肤与头骨）呈现为部分透明或完全透明。利用剖切平面，可以仅显示一个子体积或源数据的一部分。除了用于医学、石油勘探等不同领域的可视化，体积渲染还可以生成照片级真实感图像。

体素渲染技术有很多种 [842]。可以使用常规路径追踪或光子映射，在复杂光照环境下将体数据可视化。为了达到实时性能，人们还提出了若干开销较小的方法。

对于实体对象，可以使用第 17.3 节介绍的隐式表面技术，将体素转换为多边形表面。对于半透明现象，可以用一组等间距切片对体数据集进行采样，这些切片层与观察方向垂直。图 14.15 展示了其工作方式。这种方法也可以渲染不透明表面 [797]。在这种情况下，当密度大于给定阈值时，就认为存在实体体积；法线 n 可以通过计算密度场的三维梯度来求得。


![图 14.15 平行于视平面的体积切片](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.15.png)

**图 14.15。** 通过一系列平行于视平面的切片渲染一个体积。左图展示了其中一些切片及其与体积的交集。中图展示仅渲染这些切片的结果。右图展示渲染并混合大量切片后的结果。（图片由德国锡根大学 Christof Rezk-Salama 提供。）

对于半透明数据，可以为每个体素存储颜色与不透明度。为了减少内存占用，并让用户能够控制可视化效果，人们提出了传递函数。第一种方案使用一维传递纹理，将体素密度这个标量映射为颜色与不透明度。然而，这种方案不能用不同颜色独立识别特定材料间的过渡，例如人体鼻窦中的骨骼到空气、或骨骼到软组织的过渡。为解决这一问题，Kniss 等人 [912] 建议使用二维传递函数，以密度 d 和密度场梯度的长度 ‖∇d‖ 作为索引。发生变化的区域具有较大的梯度幅值。这种方法可以为密度过渡赋予更有意义的颜色。参见图 14.16。


![图 14.16 一维与二维传递函数的比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.16.png)

**图 14.16。** 使用一维（左）和二维（右）传递函数计算体积材质与不透明度 [912]。在第二种情况下，可以保留树干的棕色，而不让它被表示较低密度树叶的绿色覆盖。图像下部表示传递函数，其中 x 轴为密度，y 轴为密度场梯度的长度 ‖∇d‖。（图片由 Joe Michael Kniss [912] 提供。）

Ikits 等人 [797] 深入讨论了这项技术及相关问题。Kniss 等人 [913] 对该方法进行了扩展，改为根据半角进行切片。切片仍然按从后到前的顺序渲染，但其朝向位于光照方向与观察方向之间的半角处。使用这种方法，可以从光源视角渲染辐亮度与遮挡，并在观察空间中逐片累积。渲染下一张切片时，可以将切片纹理用作输入，利用光照方向的遮挡计算体积阴影，并利用辐亮度估计多次散射，也就是光在到达眼睛之前在介质内部发生多次反弹。由于对前一张切片的采样是在一个圆盘内取多个样本，这项技术只能合成由圆锥范围内的前向散射产生的次表面现象。最终图像具有很高的质量，参见图 14.17。Schott 等人 [1577, 1578] 扩展了这种半角方法，用于计算环境光遮蔽与景深模糊效果，从而改善用户观察体素数据时对深度和体积的感知。


![图 14.17 半角切片与前向次表面散射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.17.png)

**图 14.17。** 利用光在半角切片之间的传播，实现带有前向次表面散射的体积渲染。（图片由 Ikits 等人 [797] 提供。）

如图 14.17 所示，半角切片能够渲染高质量的次表面散射。然而，每张切片都必须支付光栅化带来的内存带宽开销。Tatarchuk 和 Shopf [1744] 在着色器中使用光线步进进行医学成像，因此只需支付一次光栅化带宽开销。光照和阴影可以通过下一节介绍的方法实现。

### 14.3.2 参与介质渲染

通过渲染参与介质，实时应用能够呈现更加丰富的场景。当涉及一天中的时刻、天气或建筑物破坏等环境变化因素时，这些效果的渲染要求就会提高。例如，森林中的雾在正午与黄昏时看起来会有所不同。穿过树木间隙的光束应当随着太阳方向与颜色的变化而调整，也应随树木的运动而动态变化。假如通过爆炸移除一些树木，遮挡物减少以及扬起的尘埃都会使该区域的散射光发生变化。篝火、手电筒及其他光源也会在空气中产生散射。本节讨论能够实时模拟这些动态视觉现象的技术。

有些技术专门用于渲染来自单个光源、带有阴影的大尺度散射。Yusov [1958] 详细介绍了其中一种方法。该方法沿极线对内散射进行采样；这里的光线在相机成像平面上投影为同一条直线。使用从光源视角生成的深度图，可以判断一个采样点是否处于阴影中。算法从相机出发执行光线步进。沿光线建立的最小值/最大值层次结构用于跳过空白空间，只在深度不连续处执行光线步进，也就是仅在确实需要准确计算体积阴影的位置进行采样。除了沿极线采样这些不连续处，也可以渲染由光源空间深度图生成的网格，在观察空间中完成这项工作 [765]。在观察空间中，计算最终散射辐亮度只需要正面与背面之间的体积。为此，计算内散射时，将正面产生的散射辐亮度加到视图中，并减去背面产生的散射辐亮度。

这两种方法可以有效重现单次散射事件，以及不透明表面遮挡产生的阴影 [765, 1958]。然而，两者都无法表示非均匀参与介质，因为它们均假设介质的材质恒定。此外，这些技术无法考虑非不透明表面产生的体积阴影，例如参与介质的自阴影，或粒子产生的透明阴影（第 13.8 节）。这些方法仍然在游戏中发挥着很好的作用，因为它们可以高分辨率渲染，而且借助空白空间跳过技术，运行速度很快 [1958]。

为处理非均匀介质这一更一般的情况，人们提出了沿光线采样体积材质的泼溅（splatting）方法。不考虑任何输入光照时，Crane 等人 [303] 使用泼溅法渲染流体模拟产生的烟雾、火焰与水。对于烟雾和火焰，每个像素都会生成一条光线，在体积内进行步进，沿光线以固定间隔从材质中收集颜色和遮挡信息。对于水，一旦遇到光线与水面的第一个交点，就终止体积采样。在每个采样位置，以密度场梯度求得表面法线。为了确保水面平滑，使用三立方插值对密度值进行滤波。图 14.18 展示了运用这些技术的例子。


![图 14.18 流体模拟与体积渲染](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.18.png)

**图 14.18。** 将体积渲染技术与 GPU 上的流体模拟结合，渲染雾与水。（左图出自《Hellgate: London》，由 Flagship Studios, Inc. 提供；右图由 NVIDIA Corporation [303] 提供。）

在考虑太阳、点光源与聚光灯时，Valient [1812] 将一组包围体渲染到半分辨率缓冲区中；这些包围体限定了每个光源应当发生散射的区域。对每个光照体积进行光线步进时，为每个像素的步进起始位置添加一个随机偏移。这样会引入少量噪声，其好处是消除了固定步长产生的条带伪影。每帧使用不同的噪声值，是隐藏伪影的一种方式。将前一帧重投影并与当前帧混合后，噪声会被平均，从而消失。渲染非均匀介质时，将平面粒子体素化到一个映射至相机视锥体的三维纹理中，其分辨率为屏幕分辨率的八分之一。该体积在光线步进时用作材质密度。为了把半分辨率散射结果合成到全分辨率主缓冲区上，可以先使用双边高斯模糊，再使用双边上采样滤波器 [816]，并考虑像素之间的深度差。如果相对于中心像素的深度差过大，就舍弃该样本。这种高斯模糊在数学上不可分离（第 12.1 节），但实际效果很好。算法的复杂度取决于泼溅到屏幕上的光照体积数量，并随它们覆盖的像素范围而变化。

这种方法还可以通过使用蓝噪声加以扩展，蓝噪声更有利于在一帧的像素上产生均匀分布的随机值 [539]。在用双边滤波器进行上采样并在空间上混合样本时，这样能获得更平滑的视觉效果。对半分辨率缓冲区上采样，也可以通过混合四个随机样本来实现。结果仍然有噪声，但由于它产生的是全分辨率的逐像素噪声，因此很容易通过时间抗锯齿后处理消除（第 5.4 节）。

所有这些方法的缺点是，将体积元素与其他任意透明表面按深度排序并泼溅，始终无法保证结果在视觉上具有正确的前后顺序，例如面对大型非凸透明网格或大尺度粒子效果时就是如此。把体积光照应用到透明表面时，这些算法都需要某种特殊处理，例如使用一个在体素中存放内散射与透射率的体积 [1812]。那么，为什么不从一开始就使用基于体素的表示，既表示随空间变化的参与介质属性，也表示光散射和透射产生的辐亮度分布呢？电影行业很早就已经在使用此类技术 [1908]。

Wronski [1917] 提出了一种方法，将太阳及场景中各光源产生的散射辐亮度体素化到三维体积纹理 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_f1e4460333e732.png) 中，并将该纹理映射到视图裁剪空间。对于每个体素中心的世界空间位置，计算散射辐亮度；体积的 x 轴和 y 轴对应屏幕坐标，而 z 坐标映射到相机视锥体的深度。该体积纹理的分辨率远低于最终图像。此技术的典型实现中，x 轴和 y 轴上的体素分辨率均为屏幕分辨率的八分之一。沿 z 坐标的细分程度取决于质量与性能之间的权衡，通常选择 64 张切片。纹理在 RGB 通道中存储内散射辐亮度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_19f7b428ed9577.png)，在 alpha 通道中存储消光系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_672ed2e5eb107d.png)。根据这些输入数据，从近到远逐张迭代切片，生成最终散射体积 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_fdab5e8800cf28.png)，计算式为


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_b93261203df259.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_c887102d9bcd30.png)，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_58bdff574e799c.png)，且 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_6087b2d22118fd.png)。这样就依据前一张切片 z − 1 的数据，在世界空间切片深度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_1038808d61f5e3.png) 上更新切片 z。最终，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_fdab5e8800cf28.png) 的每个体素都包含到达观察者的散射辐亮度，以及作用于背景的透射率。注意，在式（14.15）中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_19f7b428ed9577.png) 只受先前切片的透射率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_b2cda7deecc8e8.png) 影响。这种处理是不正确的，因为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_19f7b428ed9577.png) 还应受到当前切片内部由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_672ed2e5eb107d.png) 产生的透射率的影响。Hillaire [742, 743] 讨论了这一问题。他针对给定深度内消光系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_672ed2e5eb107d.png) 恒定的情况，提出了对 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_19f7b428ed9577.png) 积分的解析解：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_e0a208cbd51419.png)


> **译注（原书排印疑点）：** 以上两式及其变量定义均按书页 610 原样保留。原文把前一层累积数据写成从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_f1e4460333e732.png) 读取，但按前后文所述的递推过程，这里似应读取最终散射体积中已经累积的前一层数据。另外，式（14.16）的新增散射项未乘先前切片的透射率；若内散射辐亮度仍沿用前文的局部量定义，则该项通常还需这一因子。此处仅指出与上下文的疑点，未将推断改入原式。原式还隐含 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_672ed2e5eb107d.png) 非零的除法条件；零消光情形应按相应极限处理。

对于辐亮度为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_68dd707edb2490.png) 的不透明表面，其最终像素辐亮度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_4a356304a16efc.png) 会由 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_fdab5e8800cf28.png) 中的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_c68b8a4884a66e.png) 和 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_2e7534139de451.png) 修正。这些量使用裁剪空间坐标采样，结果为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_47328b63cda791.png)。由于 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_fdab5e8800cf28.png) 的分辨率较粗，相机运动以及高频的明亮光照或阴影会使其产生走样。可以将前一帧的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_fdab5e8800cf28.png) 重投影，再通过指数移动平均与新的 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_fdab5e8800cf28.png) 合并 [742]。

在这一框架的基础上，Hillaire [742] 提出了基于物理的参与介质材质定义方法，参数如下：散射系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_c1ef88f95d5338.png)、吸收系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_f883b72ca81859.png)、相函数参数 g，以及发射辐亮度 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_013c0484d8a8c2.png)。将这种材质映射到相机视锥体，并存入参与介质材质体积纹理 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_bc357c8e44d4ce.png)；它相当于存储不透明表面材质的 G 缓冲区的三维版本（第 20.1 节）。Hillaire 展示了：仅考虑单次散射时，尽管进行了体素离散化，采用这种基于物理的材质表示仍然能够得到接近路径追踪的视觉效果。与网格类似，放置在世界中的参与介质体积会被体素化到 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_bc357c8e44d4ce.png) 中（见图 14.19）。在每个这样的体积中，定义一种材质，再通过从输入三维纹理采样密度来添加变化，从而产生非均匀参与介质。结果如图 14.20 所示。虚幻引擎 [1802] 也实现了同一种方法，不过它使用粒子作为参与介质来源，而不是使用盒形体积，即假定体积为球形而非盒形。还可以使用稀疏结构表示材质体积纹理 [1191]：使用一个最顶层体积，其中每个体素要么为空，要么指向包含参与介质材质数据的更精细体积。


![图 14.19 参与介质体积到相机视锥体的体素化](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.19.png)

**图 14.19。** 美术人员在关卡中放置参与介质体积，并将其体素化到相机视锥体空间的示例 [742, 1917]。左图将三维纹理映射到体积上，此例中纹理呈球形。纹理定义体积的外观，类似于三角形上的纹理。右图在考虑体积的世界变换后，将其体素化到相机视锥体中。计算着色器将贡献累积到该体积覆盖的每个体素中。得到的材质随后可用于计算每个体素中的光散射相互作用 [742]。注意，映射到相机裁剪空间时，体素呈现为小视锥体形状，因此称为 *froxel*（视锥体素）。

基于相机视锥体体积的方法 [742, 1917]，唯一的缺点是：为了在性能较弱的平台上达到可接受的性能，并保持合理的内存用量，必须采用较低的屏幕空间分辨率。而这正是前文介绍的泼溅法擅长的方面，因为它们可以生成清晰的视觉细节。如前所述，泼溅法需要更多内存带宽，而且解决方案不够统一；例如，在没有排序问题的前提下将其应用于其他任意透明表面，或让参与介质向自身投射体积阴影，都更加困难。

不只是直接光，已经发生反弹或散射的光照也可以在介质中散射。与 Wronski [1917] 的方法类似，虚幻引擎可以烘焙体积光照贴图，将辐照度存储在体积中；在体素化到视图体积时，再让这些光散射回介质 [1802]。为了在参与介质中实现动态全局光照，也可以依靠光传播体积 [143]。

体积阴影是一项重要功能。缺少体积阴影时，浓雾场景的最终图像可能显得过亮，也比应有的效果更加平面化 [742]。此外，阴影是重要的视觉线索：它们有助于观察者感知深度和体积 [1846]，使图像更真实，并能够增强沉浸感。Hillaire [742] 提出了实现体积阴影的统一方案。按照 clipmap 分布方案 [1777]，将参与介质体积与粒子体素化到相机周围三个级联体积中，称为*消光体积*。这些体积包含计算 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_2e7534139de451.png) 所需的消光系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_672ed2e5eb107d.png)，构成一个统一的采样数据源，可借助不透明度阴影贴图实现体积阴影 [742, 894]。参见图 14.21。这种方案让粒子和参与介质能够产生自阴影、相互投射阴影，也能向场景中的其他任意不透明与透明元素投射阴影。


![图 14.20 体积光照与阴影的场景比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.20.png)

**图 14.20。** 场景在不使用（上）和使用（下）体积光照与阴影时的渲染结果。场景中的每个光源都与参与介质发生相互作用。利用每个光源的辐亮度、IES 配光数据和阴影贴图，累积其散射光贡献 [742]。（图片由 Frostbite 提供，© 2018 Electronic Arts Inc.）

体积阴影可以用不透明度阴影贴图表示。然而，如果需要高分辨率来捕捉细节，使用体积纹理很快就会成为限制。因此，人们提出了其他表示方式，以更高效地表示 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_03_2e7534139de451.png)，例如使用正交函数基，包括傅里叶变换 [816] 或离散余弦变换 [341]。详情参见第 7.8 节。


![图 14.21 体积阴影与消光体积调试视图](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_3_14.21.png)

**图 14.21。** 上排是在不使用（左）和使用（右）体积阴影时渲染的场景。下排是体素化粒子消光（左）与体积阴影（右）的调试视图。绿色越强，表示透射率越低 [742]。（图片由 Frostbite 提供，© 2018 Electronic Arts Inc.）


## 14.4 天空渲染

来源：《Real-Time Rendering, Fourth Edition》，书页613—623（PDF物理页634—644）。本节从14.4标题开始，至14.5标题之前结束；包含全部下级小节、图14.22—14.32和公式（14.17）—（14.18）。

渲染一个世界，自然需要行星的天空、大气效果和云。我们在地球上所说的蓝天，是阳光在大气的参与介质中散射的结果。天空为什么在白天呈蓝色，而在太阳位于地平线时呈红色，已在第14.1.3节解释。大气也是一种关键的视觉线索，因为它的颜色与太阳方向有关，而太阳方向又与一天中的时刻有关。大气有时呈现出的雾状外观，有助于观察者感知场景中各元素的相对距离、位置和大小。因此，准确渲染这些组成部分非常重要：越来越多的游戏和其他应用具有动态的昼夜时刻、不断变化并影响云形的天气，以及可供探索、驾车穿行甚至飞越的大型开放世界，都需要这些效果。

### 14.4.1 天空与空气透视

为了渲染大气效果，我们需要考虑两个主要组成部分，如图14.22所示。首先，模拟阳光与空气粒子的相互作用，产生依赖波长的瑞利散射。这会形成天空的颜色以及一层薄雾，后者也称为**空气透视**。其次，需要考虑集中在地面附近的大颗粒对阳光的影响。这些大颗粒的浓度取决于天气状况、污染等因素。大颗粒会引起不依赖波长的米氏散射。这种现象会在太阳周围形成明亮的光晕，尤其是在颗粒浓度很高时。


![图14.22 两类大气光散射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.22.png)

图14.22．两种不同类型的大气光散射：上排仅有瑞利散射，下排同时具有米氏散射和通常的瑞利散射。从左到右分别为：密度为0、文献[203]所述的通常密度，以及夸大的密度。（图片由Frostbite提供，© 2018 Electronic Arts Inc. [743]。）

第一个基于物理的大气模型[1285]通过模拟单次散射，从太空视角渲染地球及其大气。使用O’Neil提出的方法[1333]可以获得类似结果。借助单遍着色器中的光线步进，可以从地面到太空的各种视角渲染地球。渲染天空穹顶时，用于积分米氏散射和瑞利散射的昂贵光线步进在每个顶点上执行。不过，视觉上具有高频变化的相位函数是在像素着色器中求值的。这使外观保持平滑，避免插值暴露出天空几何体的结构。也可以将散射存入纹理，并把求值分散到多帧中，以更新延迟换取更好的性能，从而获得相同结果[1871]。

解析技术使用数学模型，对测量得到的天空辐亮度[1443]，或通过昂贵的大气光散射路径追踪生成的参考图像[778]进行拟合。与参与介质材质的参数相比，其输入参数集合通常受到更多限制。例如，使用**浑浊度**表示产生米氏散射的颗粒的贡献，而不是使用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_c1ef88f95d5338.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_672ed2e5eb107d.png)系数。Preetham等人[1443]提出的这类模型，可以根据浑浊度和太阳仰角求出任意方向上的天空辐亮度。后续改进增加了对光谱输出的支持，改善了太阳周围散射辐亮度的方向性，并新增了地面反照率输入参数[778]。解析天空模型求值很快。不过，它们仅限于地面视角，而且不能通过改变大气参数来模拟地外行星，或实现特定的美术目标。


![图14.23 从地面和太空观察地球大气](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.23.png)

图14.23．使用查找表，从地面（左）和太空（右）实时渲染地球大气。（图片由Bruneton和Neyret提供[203]。）

另一种天空渲染方法，是假定地球为完美球体，其外部环绕着一层由非均匀参与介质构成的大气。Bruneton和Neyret[203]以及Hillaire[743]对大气组成进行了详尽描述。利用这些事实，可以使用预计算表，按照当前观察高度r、视线向量与天顶夹角的余弦![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_2dc380b43a0796.png)、太阳方向与天顶夹角的余弦![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_279fa9d53d1729.png)，以及方位角平面内视线向量相对于太阳方向的夹角余弦ν，存储透射率和散射。例如，从观察点到大气边界的透射率可以由r和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_2dc380b43a0796.png)两个参数表示。在预计算步骤中，对大气中的透射率进行积分，并将其存入二维查找表（LUT）纹理![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_cfcf8437450507.png)；运行时可以使用相同的参数化方式对它采样。该纹理可以用于将大气透射率应用到太阳、恒星或其他天体等天空元素上。

对于散射，Bruneton和Neyret[203]介绍了一种将其存入四维LUT ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_894f098d1291f2.png)的方法，其参数就是上一段提到的全部参数。他们还给出了一种通过迭代n次来求取n阶多次散射的方法：（i）计算单次散射表![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_894f098d1291f2.png)；（ii）利用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_55876db7aab541.png)计算![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_b2b690083c20ab.png)；（iii）将结果加到![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_894f098d1291f2.png)中。将（ii）和（iii）执行n−1次。关于该过程的更多细节以及源代码，可参见Bruneton和Neyret[203]。图14.23给出了结果示例。Bruneton和Neyret的参数化有时会在地平线处出现视觉伪影。Yusov[1957]提出了一种改进的变换。还可以忽略ν，只使用三维LUT[419]。采用这种方案，地球不会在大气中投下阴影，这可能是可以接受的取舍。它的优点是LUT会小得多，更新和采样开销也更低。


![图14.24 地球和其他行星的大气](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.24.png)

图14.24．使用完全参数化的模型进行实时渲染，可以模拟地球大气（上排）和其他行星的大气，例如火星的蓝色日落（下排）。（上排图片由Bruneton和Neyret提供[203]，下排图片由Frostbite提供，© 2018 Electronic Arts Inc. [743]。）

上述最后一种三维LUT方法已被许多采用Electronic Arts Frostbite引擎的实时游戏使用，例如《极品飞车》《镜之边缘：催化剂》和《FIFA》[743]。在这种情况下，美术师可以调整基于物理的大气参数，以达到目标天空外观，甚至模拟地外大气。见图14.24。改变大气参数后，必须重新计算LUT。为了更高效地更新这些LUT，也可以用一个函数近似大气中材质的积分，而不必在其中进行光线步进[1587]。通过在时间上分散LUT和多次散射的求值，可以把更新LUT的摊销开销降低到原来的6%。具体做法是：对于给定的散射阶数n，只更新![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_b2b690083c20ab.png)的一部分，同时在最近两张已求解的LUT之间进行插值，接受数帧延迟。另一项优化是，为避免每个像素多次采样不同LUT，将米氏散射和瑞利散射烘焙到一个低分辨率体纹理的体素中，该体纹理映射到摄像机视锥体。视觉上具有高频变化的相位函数在像素着色器中求值，以在太阳周围生成平滑的散射光晕。使用这类体纹理，还可以逐顶点地对场景中任何透明物体应用空气透视。

### 14.4.2 云

云是天空中复杂的元素。当预示暴风雨即将来临时，它们可以显得来势汹汹；也可以显得低调、壮观、轻薄或厚重。云变化缓慢，大尺度形状和小尺度细节都会随时间演化。具有天气和昼夜时刻变化的大型开放世界游戏属于更复杂的情况，需要动态云渲染方案。根据目标性能和视觉质量，可以采用不同技术。

云由水滴构成，具有很高的散射系数和复杂的相位函数，由此形成特有的外观。通常使用第14.1节所述的参与介质来模拟云。测量表明，云材质具有很高的单次散射反照率ρ＝1；层云（低空的水平云层）的消光系数![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_672ed2e5eb107d.png)在区间[0.04, 0.06]内，积云（低空中彼此分离、像棉花一样蓬松的云）的消光系数在区间[0.05, 0.12]内[743]。见图14.25。鉴于ρ接近1，可以假定![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_c1ef88f95d5338.png)＝![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_672ed2e5eb107d.png)。


![图14.25 地球上的云类型](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.25.png)

图14.25．地球上不同类型的云。（图片由Valentin de Bruyn提供。）

一种经典的云渲染方法，是使用单张全景纹理，通过alpha混合将其合成到天空上。这对于静态天空渲染很方便。Guerrette[620]提出了一种视觉流动技术，使人产生云受全局风向影响、在天空中运动的错觉。这种方法效率很高，比使用一组静态全景云纹理有所改进。不过，它无法表现云形和光照的任何变化。

#### 以粒子表示云

Harris将云渲染为由粒子和替身图像（impostor）构成的体积[670]。参见第13.6.2节以及书页557的图13.9。

Yusov[1959]提出了另一种基于粒子的云渲染方法。他使用称为**体积粒子**的渲染图元。每个图元都由一个四维LUT表示，能够根据太阳光方向和观察方向，获取朝向观察者的四边形粒子上的散射光和透射率。见图14.26。这种方法非常适合渲染层积云。参见图14.25。


![图14.26 体积粒子云](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.26.png)

图14.26．将云渲染为粒子体积。（图片由Egor Yusov提供[1959]。）

将云渲染为粒子时，常常会出现离散化和突变伪影，尤其是在绕着云旋转观察时。使用**体积感知混合**可以避免这些问题。这种能力借助称为光栅化顺序视图的GPU特性实现（第3.8节）。体积感知混合能够按图元同步像素着色器对资源的操作，从而实现结果确定的自定义混合操作。最近的n个粒子的深度层保存在一个缓冲区中，其分辨率与正在写入的渲染目标相同。读取该缓冲区，并考虑相交深度来混合当前渲染的粒子，最后再将缓冲区写回，供下一个待渲染粒子使用。结果见图14.27。


![图14.27 常规混合和体积感知混合](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.27.png)

图14.27．左：以常规方式渲染的云粒子。右：使用体积感知混合渲染的粒子。（图片由Egor Yusov提供[1959]。）

#### 以参与介质表示云

将云视为独立元素时，Bouthors等人[184]使用两个组成部分表示一朵云：一个网格表现整体形状；一个超纹理[1371]在网格表面之下、直至云内部一定深度处添加高频细节。使用这种表示，可以在云边缘进行精细的光线步进以获取细节，而内部则可以视为均匀介质。在沿云结构进行光线步进时积分辐亮度，并根据散射阶数使用不同算法收集散射辐亮度。单次散射使用第14.1节介绍的解析方法积分。多次散射求值则利用离线预计算的传输表加速，这些表来自放置在云表面的圆盘形光收集器。最终结果具有很高的视觉质量，如图14.28所示。


![图14.28 网格和超纹理云](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.28.png)

图14.28．使用网格和超纹理渲染的云。（图片由Bouthors等人提供[184]。）

除了将云渲染为独立元素，还可以将它们建模为大气中的一层参与介质。Schneider和Vos基于光线步进，提出了一种以这种方式高效渲染云的方法[1572]。只需少量参数，就可以在动态昼夜时刻的光照条件下，渲染复杂、具有动画且细节丰富的云形，如图14.29所示。云层使用两级程序化噪声构建。第一级赋予云基本形状，第二级通过侵蚀该形状添加细节。在这种情况下，据报告，混合Perlin噪声[1373]和Worley噪声[1907]，可以很好地表示积云及类似云朵的花椰菜状形态。用于生成这类纹理的源代码和工具已经公开共享[743, 1572]。光照则通过沿视线在云层中分布采样点，对来自太阳的散射光进行积分来实现。


![图14.29 光线步进云层](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.29.png)

图14.29．对使用Perlin-Worley噪声的云层进行光线步进，渲染出具有动态体积光照和阴影的云。（结果来自Schneider和Vos[1572]，版权© 2017 Guerrilla Games。）

体积阴影可以这样实现：对于云层内部的少量采样点，朝太阳方向进行测试，求取透射率[743, 1572]，这相当于第二次光线步进。对于这些阴影采样点，可以采样噪声纹理的较低分辨率mipmap层级，以改善性能，并平滑仅使用少量采样点时可能出现的伪影。另一种避免对每个采样点执行第二次光线步进的方法，是每帧使用多种可用技术之一，将来自太阳方向的透射率曲线编码到纹理中一次（第13.8节）。例如，《最终幻想XV》[416]使用了透射率函数映射[341]。

如果希望捕捉每一个微小细节，使用光线步进以高分辨率渲染云可能变得很昂贵。为获得更好性能，可以低分辨率渲染云。一种方法是在每个4×4像素块内仅更新一个像素，再重投影前一帧的数据以填充其余像素[1572]。Hillaire[743]提出了一种变体：始终以固定的较低分辨率渲染，并在沿视线进行光线步进的起始位置上添加噪声。可以重投影前一帧结果，再使用指数移动平均[862]将其与新帧合并。这种方法以较低分辨率渲染，但能够更快收敛。

云的相位函数很复杂[184]。这里介绍两种可用于实时求值的方法。可以将函数编码为纹理，再根据θ对其采样。如果这样做需要过多内存带宽，则可以组合第14.1.4节中的两个Henyey-Greenstein相位函数[743]来近似该函数：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_33db8d6921e1eb.png)


其中，两个主要散射偏心率![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_604c2430f0ec2a.png)和![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_4997b6311784ab.png)，以及混合因子w，都可以由美术师设定。这对于同时表现主要的前向散射和后向散射方向十分重要，使我们在背向或朝向太阳、月亮等光源观察时，都能显露云中的细节。见图14.30。

有多种方法可以近似云中由环境光照产生的散射光。一种直接的方法是，将天空渲染到立方体贴图纹理中，然后对其均匀积分，得到单个输入辐亮度。也可以使用一个自下而上、由暗到亮的渐变来缩放环境光照，近似云自身造成的遮挡。还可以将这一输入辐亮度分成来自下方和上方的两部分，例如地面和天空[416]。然后，假定云层内部的介质密度恒定，就能对两种贡献的环境散射进行解析积分[1149]。


![图14.30 动态日光和月光下的云](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.30.png)

图14.30．按照Hillaire[743]所述，使用基于物理的参与介质表示，对云层进行光线步进，渲染具有动态光照和阴影的云。（图片由BioWare的Sören Hesse〈上〉和Ben McGrath〈下〉提供，© 2018 Electronic Arts Inc.。）

#### 多次散射近似

云明亮而洁白的外观，是光在其内部多次散射的结果。如果没有多次散射，厚云大多只会在体积边缘受到照亮，而其他地方都显得很暗。多次散射是避免云看起来像烟雾或污浊物的关键因素。使用路径追踪计算多次散射代价极其高昂。Wrenninge[1909]提出了一种在光线步进时近似这一现象的方法。它对o个倍频层（octave）的散射进行积分，并将其求和：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_e00e979ee4c397.png)


其中，计算![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_c68b8a4884a66e.png)时进行以下替换（例如，用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_1a8ae125adb34f.png)替代![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_c1ef88f95d5338.png)）：![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_73b0367512f9cd.png)、![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_9be818b783d41e.png)以及![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_e4631cd2273ff3.png)。这里a、b和c是用户控制的参数，取值范围为[0, 1]，它们使光能够穿透参与介质。这些值越接近0，云的外观就越柔和。为确保计算![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_a022511fe8d381.png)时该技术保持能量守恒，必须保证a≤b。否则，由于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_c1ef88f95d5338.png)最终可能大于![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_672ed2e5eb107d.png)，等式![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_672ed2e5eb107d.png)＝![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_f883b72ca81859.png)＋![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_c1ef88f95d5338.png)将不再成立，从而可能散射出更多的光。这种方案的优点是，能够在光线步进过程中即时对每个不同倍频层的散射光积分。视觉改进如图14.31所示。它的缺点是，对于光可能向任意方向散射的复杂多次散射行为，表现不佳。不过，云的外观得到了改善；而且，由于可实现的效果范围更广，这种方法允许灯光美术师通过少量参数轻松控制视觉效果、表达其创意。采用这种方法，光可以穿透介质，显露更多内部细节。

> 译注：上述替换中的消光符号原书印为![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_cf66fa22f17441.png)，而紧接着的能量守恒说明使用![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_04_672ed2e5eb107d.png)；这里忠实保留原书的符号差异。式（14.18）的求和项通过本段所述替换依赖n，原书未在该项中显式标出这一依赖。图14.31图注的n亦按原书保留。


![图14.31 多次散射近似](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.31.png)

图14.31．使用公式（14.18）作为多次散射近似来渲染云。从左到右，n分别设为1、2和3。这使阳光能够以可信的方式穿透云层。（图片由Frostbite提供，© 2018 Electronic Arts Inc. [743]。）

#### 云与大气的相互作用

渲染有云的场景时，为使视觉效果保持一致，必须考虑云与大气散射的相互作用。见图14.32。

由于云是大尺度元素，应当对其应用大气散射。可以对穿过云层时取得的每个采样点，计算第14.4.1节介绍的大气散射，但这样做的开销会迅速增加。另一种方法是，依据一个表示云的平均深度与透射率的单一深度值，对云应用大气散射[743]。


![图14.32 云与大气相互作用](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_4_14.32.png)

图14.32．考虑大气后，渲染完全覆盖天空的云[743]。左：未对云应用大气散射，导致视觉效果不协调。中：加入大气散射，但由于没有阴影，环境显得过亮。右：云遮挡天空，从而影响大气中的光散射，产生协调的视觉效果。（图片由Frostbite提供，© 2018 Electronic Arts Inc. [743]。）

如果增大云量以模拟阴雨天气，应减少云层下方大气中的阳光散射。只有经过云层散射并穿过云层的光，才应在其下方的大气中散射。可以通过减小天空光照对空气透视的贡献，并将散射光重新加入大气，来调整照明[743]。图14.32展示了视觉效果的改进。

总之，可以利用先进的基于物理的材质表示和光照来实现云渲染。使用程序化噪声能够获得逼真的云形和细节。最后，正如本节所述，为了得到协调一致的视觉结果，还必须顾及整体，例如云与天空之间的相互作用。


## 14.5 半透明表面

来源：《Real-Time Rendering, Fourth Edition》，书页 623—632（PDF 物理页 644—653）。范围从 14.5 节标题开始，到 14.6 节标题之前，包含全部三级小节、图 14.33—14.41 和公式（14.19）—（14.29）。

半透明表面通常指吸收系数较高、散射系数较低的材质。这类材质包括玻璃、水，以及书页 592 图 14.2 所示的葡萄酒。此外，本节还将讨论表面粗糙的半透明玻璃。许多出版物也详细讨论了这些主题 [1182, 1185, 1413]。

### 14.5.1 覆盖率与透射率

如第 5.5 节所述，可以把透明表面看作具有由 α 表示的覆盖率，例如不透明的织物或组织纤维遮住其后方一定比例的内容。对于玻璃等材质，我们希望计算半透明性：实体体积允许每个波长的光以一定比例通过，依据透射率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_03a7e41edb871d.png)（第 14.1.2 节）对背景进行滤色。设输出颜色为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_9c1542bf5c49c0.png)，表面辐亮度为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_2ce4c3f8e5a5ae.png)，背景颜色为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_4c92dfa5297c2a.png)，则把透明度视作覆盖率的表面采用以下混合运算：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_5249f3a22b8258.png)


对于半透明表面，混合运算为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_e1e7e1c9ee892e.png)


其中，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_2ce4c3f8e5a5ae.png) 包含实体表面（例如玻璃或凝胶）的镜面反射。注意，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_03a7e41edb871d.png) 是一个具有三个分量的透射颜色向量。为了实现有色半透明效果，可以利用任何现代图形 API 的双源颜色混合功能，指定这两种输出颜色，使其与目标缓冲区颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_4c92dfa5297c2a.png) 混合。Drobot [386] 介绍了可采用的不同混合运算，具体取决于给定表面的反射和透射是否带有颜色。


![图14.33](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.33.png)

图 14.33．光穿过网格的多个层时，不同吸收因子产生的半透明效果 [115]。（图片由 Louis Bavoil 提供 [115]。）

一般情况下，可以使用一种通用混合运算，同时指定覆盖率和半透明性 [1185]。此时应采用的混合函数为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_09bd5533439d89.png)


厚度变化时，可以使用公式（14.3）计算透过的光量，该公式可简化为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_320b74fe77dab7.png)


其中 d 是光在材质体积内传播的距离。物理消光参数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_bb3c21d121c401.png) 表示光在介质中传播时衰减的速率。为了让美术人员能够直观地创作，Bavoil [115] 将目标颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_3c1a23f4257978.png) 设为某个给定距离 d 处的透射量。于是，可以反求消光系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_bb3c21d121c401.png)：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_a10ac9a8c31c65.png)


例如，目标透射颜色 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_3c1a23f4257978.png) = (0.3, 0.7, 0.1)，距离 d = 4.0 米，则可求得：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_a1f5ab53ae2417.png)


注意，透射率为 0 的情况需要特殊处理。一种解决办法是从 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_03a7e41edb871d.png) 的每个分量中减去一个很小的 ε，例如 0.000001。图 14.33 展示了滤色效果。

> 译注：原文确实写作“减去”。但对零分量减去正 ε 会得到负数，不能作为公式（14.23）中实数对数的输入；这里保留原文并标出疑点。实现时通常会将对数输入限制在正的最小值以上。此外，公式（14.24）的数值对应自然对数。

对于表面由单层薄半透明材质构成的空壳网格，背景颜色应根据光在介质内传播的路径长度 d 而受到遮挡。因此，沿法线或沿切向观察这种表面，会因厚度 t 而产生不同程度的背景遮挡，因为路径长度会随角度变化。Drobot [386] 提出了这样的方案，其中透射率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_03a7e41edb871d.png) 计算如下：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_77fe73cd448cbd.png)


![图14.34](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.34.png)

图 14.34．依据观察射线 v 在厚度为 t 的透明表面内部传播的距离 d，计算有色透射率。（右图由 Activision Publishing, Inc. 提供，2018 年。）

图 14.34 展示了结果。关于薄膜和多层表面的更多细节，见第 9.11.2 节。

对于实心半透明网格，有许多方法可以计算射线在透射介质中实际传播的距离。一种常见方法是先渲染观察射线离开体积时经过的表面。该表面可以是水晶球的背面，也可以是海底（即水体结束的位置）。将此表面的深度或位置保存下来，然后渲染体积的表面。在着色器中读取保存的出口深度，并计算出口与当前像素所在表面之间的距离，再用这个距离计算应施加于背景的透射率。

只要能够保证体积封闭且为凸形，即像水晶球一样，每个像素都只有一个入口点和一个出口点，这种方法就可以工作。海底的例子同样适用，因为一旦离开水体就会遇到不透明表面，不会再发生进一步透射。对于更复杂的模型，例如玻璃雕塑或其他具有凹陷的物体，入射光可能会在两个或更多彼此分离的区段中被吸收。利用第 5.5 节讨论的深度剥离，可以按精确的由后向前顺序渲染体积表面。每次渲染正面时，都计算光在体积内经过的距离，并据此计算透射率。依次应用这些透射率，就能得到正确的最终透射率。注意，如果所有体积均由浓度相同的同种材质构成，且表面没有反射分量，那么可以将距离求和，最后只计算一次透射率。还可以使用在单次通道中直接保存物体片元的 A 缓冲区或 K 缓冲区方法，以便在较新的 GPU 上获得更高效率 [115, 230]。图 14.33 展示了这种多层透射的例子。

对于大尺度海水，可以直接使用场景深度缓冲区来表示作为背面的海底。渲染透明表面时，必须考虑第 9.5 节介绍的菲涅耳效应。大多数透射介质的折射率明显高于空气。在掠射角下，所有光都会从界面反弹，没有光透过。图 14.35 展示了这一效果：向下直视水中时，水下物体可见；而看向更远处、以掠射角观察时，水面会基本遮住波浪下面的物体。有几篇文章介绍了如何处理大面积水体的反射、吸收与折射 [261, 977]。


![图14.35](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.35.png)

图 14.35．考虑透射与反射效果的水体渲染。向下看时，因为透射率较高且呈蓝色，我们可以透过水面看到带有浅蓝色调的水下景物。靠近地平线时，海底变得不那么明显，这是因为透射率降低（光必须在水体内传播很远），以及菲涅耳效应使反射增强、透射相应减弱。（图片来自《Crysis》，由 Crytek 提供。）

### 14.5.2 折射

在处理透射率时，我们假设入射光沿直线直接来自网格体积后方。当网格前后表面平行、厚度不大时，例如一块窗玻璃，这个假设是合理的。对于其他透明介质，折射率起着重要作用。第 9.5 节介绍了斯涅尔定律，它描述光遇到网格表面时如何改变方向。

由于能量守恒，未被反射的光都会透射，因此透射通量与入射通量之比为 1 − f，其中 f 是反射光的比例。然而，透射辐亮度与入射辐亮度的比值并不相同。由于入射光线与透射光线的投影面积和立体角存在差异，辐亮度关系为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_93a3d4d7265663.png)


![图14.36](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.36.png)

图 14.36．随入射角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_ba4635370b500e.png) 与透射角 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_ca26290f1bb6de.png) 变化的折射与透射辐亮度。

图 14.36 说明了这种行为。将斯涅尔定律与公式（14.26）结合，可得到透射辐亮度的另一种形式：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_4ef0c80a30ae85.png)


Bec [123] 提出了一种高效计算折射向量的方法。为便于阅读（因为斯涅尔方程中习惯用 n 表示折射率），这里用 N 表示表面法线，用 l 表示指向光源的方向：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_d8299b9b99b047.png)


其中 n = ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_3a00723a6f5471.png)/![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_d13080baeee9f5.png) 是相对折射率，并且：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_05_79990e27f79fb2.png)


所得折射向量 t 返回时已归一化。水的折射率约为 1.33，玻璃通常约为 1.5，而空气实际上可视为 1.0。

折射率随波长变化。也就是说，透明介质会把每种颜色的光弯折不同的角度。这种现象称为色散，它解释了棱镜为什么会把白光展开为彩虹色光锥，也解释了彩虹为什么会出现。色散会在透镜中引发一种称为色差的问题。在摄影中，这种现象称为紫边，在日光下的高对比度边缘处可能尤其明显。在计算机图形学中，我们通常忽略这一效应，因为它通常是希望避免的瑕疵。正确模拟这种效果需要额外计算，因为每条进入透明表面的光线都会产生一组必须继续追踪的光线。因此，通常只使用一条折射光线。值得指出的是，一些虚拟现实渲染器会应用逆色差变换，以补偿头戴设备透镜的色差 [1423, 1823]。


![图14.37](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.37.png)

图 14.37．左：玻璃天使对立方体环境贴图产生折射，环境贴图本身也用作天空盒背景。右：带有色差的玻璃球产生的反射与折射。（左图来自 three.js 示例 webgl_materials_cube_map_refraction [218]，Lucy 模型来自 Stanford 3D 扫描模型库，纹理由 Humus 制作。右图由 Lee Stemkoski 提供 [1696]。）

一种营造折射观感的通用方式，是从折射物体的位置生成立方体环境贴图（EM）。之后渲染该物体时，可以使用为正面表面计算出的折射方向访问 EM。图 14.37 展示了一个例子。Sousa [1675] 提出了一种不使用 EM 的屏幕空间方法。首先，照常将场景渲染到场景纹理 s 中，但不包含任何折射物体。其次，将折射物体渲染到 s 的 alpha 通道，该通道事先清为 1；若像素通过深度测试，就写入 0。最后，完整渲染折射物体，并在像素着色器中依据屏幕上的像素位置、加上扰动偏移来采样 s，以模拟折射。偏移量例如可以来自经过缩放的表面法线切向 xy 分量。在这里，只有 α = 0 时才使用扰动采样的颜色。这项测试是为了避免采到位于折射物体前方的表面，否则这些表面的颜色也会被拉入，好像它们位于物体后面一样。注意，也可以不设置 α = 0，而改用场景深度图，将像素着色器的深度与扰动后场景采样的深度进行比较 [294]。若中心像素更远，就意味着偏移采样更近；此时忽略该采样，并用正常的场景采样替换，效果就像没有折射一样。

这些技术能够营造折射的观感，却与物理实际相去甚远。光线进入透明实体时改变了方向，但到了应当离开物体的位置，却从未再次弯折。出口界面始终没有发挥作用。有时，这个缺陷无关紧要，因为人眼对外观应当有多准确是相当宽容的 [1185]。

许多游戏都支持穿过单层表面的折射。对于粗糙的折射表面，按照材质粗糙度模糊背景非常重要，这样才能模拟微观几何法线的分布导致的折射光线方向扩散。在游戏《DOOM》（2016）[1682] 中，首先照常渲染场景，然后将其下采样到一半分辨率，再进一步生成四个 mipmap 层级。每个 mipmap 层级都采用模拟 GGX BRDF 波瓣的高斯模糊进行下采样。最后，在全分辨率场景之上渲染折射网格 [294]。通过采样场景的 mipmap 纹理，并将材质粗糙度映射到 mipmap 层级，把背景合成到表面后方。表面越粗糙，背景越模糊。Drobot [386] 基于通用材质表示也提出了同样的方法。McGuire 和 Mara [1185] 的统一透明度框架同样使用了类似技术。在他们的方法中，使用高斯点扩散函数，在单个通道中采样背景。见图 14.38。


![图14.38](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.38.png)

图 14.38．图像下部的透明玻璃呈现出基于粗糙度的背景散射。玻璃后方的物体出现程度不一的模糊，以模拟折射光线的扩散。（图片由 Frostbite 提供，© 2018 Electronic Arts Inc.。）

还可以处理穿过多个层的折射这一更复杂的情况。每层都可以被渲染，并将深度和法线保存在纹理中。然后，可以使用一种思路类似浮雕映射（第 6.8.1 节）的过程，追踪光线穿过这些层。保存的深度被当作高度场，每条射线沿其步进，直到找到交点。Oliveira 和 Brauwers [1326] 提出了这样的框架，用于处理穿过网格背面的折射。此外，附近的不透明物体可以转换为颜色图和深度图，提供最后一个不透明层 [1927]。所有这些图像空间折射方案都有一个限制：屏幕边界以外的内容既不能产生折射，也不能被折射呈现。

### 14.5.3 焦散与阴影

计算经折射和衰减的光所产生的阴影与焦散，是一项复杂的任务。在非实时环境中，可以使用双向路径追踪、光子映射等多种方法实现这一目标 [822, 1413]。幸运的是，也有许多方法能够实时近似这种现象。

焦散是光偏离原本直线路径所产生的视觉结果，例如光经过玻璃或水面之后。其结果是，光从某些区域散开，产生阴影；同时又在另一些区域聚集，光线路径变得更密集，从而形成更强的入射照明。这些路径取决于光遇到的曲面。反射的一个经典例子是咖啡杯内看到的心形焦散。折射焦散更加明显，例如光穿过水晶饰物、透镜或一杯水后发生聚焦。见图 14.39。光被弯曲水面反射和折射后，也会在水面上下产生焦散。会聚的光在不透明表面上集中，形成焦散。在水面下，会聚的光路还会在水体内部显现。这是因为光子受到水中粒子的散射，产生了人们熟悉的光束。焦散是一个独立因素，应在体积边界的菲涅耳相互作用造成的光量下降、以及光穿过体积时的透射衰减之外另行考虑。


![图14.39](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.39.png)

图 14.39．现实世界中由反射和折射产生的焦散。

要从水面生成焦散，可以将离线生成的焦散动画纹理作为光照贴图施加到表面上，也可以把它叠加到通常的光照贴图之上。许多游戏都采用了这种方法，例如运行于 CryEngine 的《Crysis 3》[1591]。关卡中的水域使用水体体积来制作。体积的顶面可以利用凹凸贴图纹理动画或物理模拟进行动画处理。将凹凸贴图产生的法线沿竖直方向投影到水面上下时，可以把法线朝向映射为辐亮度贡献，从而生成焦散。距离衰减由美术人员设定的、基于高度的最大影响距离控制。也可以模拟水面，使其对世界中物体的运动作出反应，从而生成与环境中实际发生的事件相匹配的焦散变化。图 14.40 展示了一个例子。


![图14.40](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.40.png)

图 14.40．水中焦散效果演示。（图片来自 WebGL Water 演示，由 Evan Wallace 提供 [1831]。）

在水下，同一个动画水面也可以用来生成水介质内部的焦散。Lanza [977] 提出了一种生成光束的两步方法。首先，从光源视点渲染光的位置和折射方向，并将其保存到纹理中。接着，在当前视图中光栅化线段；这些线段从水面开始，沿折射方向延伸。通过加法混合累积这些线段，最后可以执行一次后处理模糊，将结果模糊化，以掩盖线段数量较少的问题。


![图14.41](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_5_14.41.png)

图 14.41．左侧，佛像同时折射附近物体和周围的天空盒 [1927]。右侧，通过性质上类似阴影贴图的层次化贴图生成焦散 [1929]。（图片由爱荷华大学 Chris Wyman 提供。）

Wyman [1928, 1929] 提出了一种用于焦散渲染的图像空间技术。该技术首先计算光经过透明物体正面和背面折射后的光子位置与入射方向。这通过第 14.5.2 节介绍的背景折射技术 [1927] 实现。不过，这里并不保存折射后的辐亮度，而是用纹理保存与场景的交点位置、折射后的入射方向，以及由菲涅耳效应决定的透射率。每个纹素保存一个光子，然后可以将该光子以正确的强度泼溅回当前视图。实现这一目标有两种选择：在观察空间中，或在光源空间中，将光子作为具有高斯衰减的四边形进行泼溅。图 14.41 展示了其中一个结果。McGuire 和 Mara [1185] 提出了一种更简单的方法，通过根据透明表面的法线改变透射率，产生类似焦散的阴影：由于菲涅耳效应，垂直入射表面时透射更多，否则透射较少。第 7.8 节还介绍了其他体积阴影技术。


## 14.6 次表面散射

本节来源：《Real-Time Rendering, Fourth Edition》书页 632—640（PDF 物理页 653—661），从 14.6 节标题起，至 14.7 节标题之前。

次表面散射是一种复杂现象，出现在具有较高散射系数的固体材料中（更多细节见 9.1.4 节）。这类材料包括蜡、人体皮肤和牛奶，如书页 592 的图 14.2 所示。

14.1 节已经解释了一般的光散射理论。在某些情况下，散射的尺度相对较小，例如人体皮肤等光学深度较高的介质。散射光会从表面上靠近原入射点的位置重新射出。这种位置上的偏移意味着，无法用 BRDF（9.9 节）来建模次表面散射。也就是说，当散射发生的距离大于一个像素时，它更具全局性的性质就会显现出来。必须采用专门的方法来渲染这样的效果。


![图14.42 光在物体内部的散射路径](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_6_14.42.png)

**图 14.42** 光穿过物体时发生散射。最初，透射进入物体的光沿折射方向传播，但散射使它反复改变方向，直至离开材料。每条穿过材料的路径的长度，决定了光因吸收而损失的百分比。

图 14.42 展示了光穿过物体时的散射。散射使入射光沿许多不同路径传播。由于逐个模拟光子并不现实（即使在离线渲染中也是如此），必须用概率方法解决这一问题：对可能的路径进行积分，或对这样的积分作近似。光在材料内部传播时，除了散射，也会发生吸收。

区分图 14.42 中各条光路径的一个重要因素，是散射事件的次数。对于某些路径，光只散射一次就离开材料；对于另一些路径，光会散射两次、三次乃至更多次。散射路径通常分为单次散射和多重散射两类。往往会对这两类分别使用不同的渲染技术。对于某些材料，单次散射在整体效果中所占的部分相对较弱，而多重散射占主导，例如皮肤。因此，许多次表面散射渲染技术都着重模拟多重散射。本节将介绍几种近似次表面散射的技术。

### 14.6.1 环绕光照

次表面散射方法中，最简单的或许是环绕光照（wrap lighting）[193]。书页 382 已经把这项技术作为面光源的近似方法加以讨论。将其用于近似次表面散射时，可以加入颜色偏移 [586]，以考虑光在材料内部传播时被部分吸收的情况。例如，渲染皮肤时可以施加偏红的颜色变化。

以这种方式使用时，环绕光照试图建模多重散射对弯曲表面着色的影响。光从相邻点“渗漏”到当前着色点，会使表面弯向背离光源方向的位置处，明暗之间的过渡区域变得柔和。Kolchin [922] 指出，这种效果取决于表面曲率，并推导了一个基于物理的版本。虽然所推导表达式的求值成本有些高，但其背后的思想很有用。

### 14.6.2 法线模糊

Stam [1686] 指出，可以将多重散射建模为一个扩散过程。Jensen 等人 [823] 进一步发展了这一思想，推导出一个解析的双向表面散射分布函数（BSSRDF）模型。BSSRDF 是将 BRDF 推广到全局次表面散射情形的结果 [1277]。扩散过程会对出射辐亮度产生空间模糊效果。

这种模糊仅应用于漫反射。镜面反射发生在材料表面，不受次表面散射影响。由于法线贴图往往编码小尺度的变化，一项实用的次表面散射技巧是，仅对镜面反射应用法线贴图 [569]；漫反射则使用平滑、未受扰动的法线。由于没有额外开销，在使用其他次表面散射方法时，结合使用这项技术通常很值得。

对于许多材料，多重散射发生在相对较短的距离内。皮肤就是一个重要例子，其中大部分散射发生在几毫米的距离内。对于此类材料，单独采用不扰动漫反射着色法线的技术，或许就已经足够。Ma 等人 [1095] 根据测量数据扩展了这一方法。他们测定散射物体的反射光，发现镜面反射以几何表面法线为依据，而次表面散射使漫反射表现得仿佛使用了模糊后的表面法线。此外，模糊程度可以随可见光谱中的波长而变化。他们提出一种实时着色技术：为镜面反射，以及漫反射的 R、G、B 通道分别独立采集法线贴图 [245]。每个通道使用不同的法线贴图，便会产生颜色渗透。由于这些漫反射法线贴图通常类似于镜面反射法线贴图的模糊版本，因此很容易将该技术改为只使用一张法线贴图，并调整 mipmap 层级；代价是会丢失颜色偏移，因为各通道使用了相同的法线。

### 14.6.3 预积分皮肤着色

Penner [1369] 将环绕光照与法线模糊的思想结合起来，提出了一种预积分皮肤着色方案。

对散射和透射率进行积分，并将结果存储在一张二维查找表中。查找表（LUT）的第一条轴依据 **n**·**l** 进行索引；第二条轴依据 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_656ab9f51479c7.png) 进行索引，它表示表面曲率。曲率越高，对透射和散射颜色的影响就越大。由于每个三角形内的曲率为常量，这些值必须离线烘焙并平滑处理。

为处理次表面散射对微小表面细节的影响，Penner 修改了上一小节讨论过的 Ma 等人 [1095] 的技术。Penner 不再分别采集 R、G、B 漫反射法线贴图，而是根据次表面材料各颜色通道的扩散剖面，对原始法线贴图进行模糊，以生成这些贴图。由于使用四张独立的法线贴图会占用大量内存，他采用一种优化：只使用一张平滑后的法线贴图，并针对每个颜色通道，将它与顶点法线混合。

由于默认只依赖曲率，这项技术会忽略跨越阴影边界的光扩散。要使散射剖面跨越阴影边界，可以利用阴影半影的剖面对 LUT 坐标施加偏置。因此，这项快速技术能够近似下一小节介绍的高质量方法 [345]。

### 14.6.4 纹理空间扩散

模糊漫反射法线能够表现多重散射的某些视觉效果，但无法表现其他一些效果，例如柔化阴影边缘。纹理空间扩散的概念可以用来解决这些局限。Lensch 等人 [1032] 最初将这一思想作为另一项技术的一部分提出，但 Borshukov 和 Lewis [178, 179] 给出的版本影响最为深远。他们将“多重散射是一种模糊过程”这一思想形式化。首先，将表面辐照度（漫反射光照）渲染到纹理中。实现时，使用纹理坐标作为光栅化的位置坐标；真实位置则单独插值，供着色使用。对该纹理进行模糊，然后在渲染时将其用于漫反射着色。滤波器的形状和尺寸取决于材料与波长。例如，对于皮肤，R 通道使用比 G、B 通道更宽的滤波器进行滤波，从而使阴影边缘附近发红。对于大多数材料，能够正确模拟次表面散射的滤波器在中心具有狭窄尖峰，在底部则宽而平缓。这项技术最初为离线渲染而提出，但 NVIDIA [345, 586] 和 ATI [568, 569, 803, 1541] 的研究人员很快便提出了实时 GPU 实现。

d’Eon 和 Luebke [345] 的介绍是对这项技术最完整的论述之一，其中包括支持模拟多层次表面结构效果的复杂滤波器。Donner 和 Jensen [369] 表明，这种结构能够产生最逼真的皮肤渲染。d’Eon 和 Luebke 介绍的完整 NVIDIA 皮肤渲染系统能够产生出色的结果（示例见图 14.43），但代价相当高，需要大量模糊遍。不过，可以很容易地缩减它的规模以提高性能。


![图14.43 纹理空间多层扩散的线性组合](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_6_14.43.png)

**图 14.43** 纹理空间多层扩散。使用 RGB 权重组合六种不同的模糊结果。最终图像是这一线性组合再加上镜面反射项所得的结果。（图片由 NVIDIA Corporation 提供 [345]。）

Hable [631] 没有应用多遍高斯模糊，而是提出了一个具有 12 个样本的单一核。该滤波器既可以作为预处理在纹理空间中应用，也可以在将网格光栅化到屏幕时，在像素着色器中应用。这样做牺牲一些真实感，却能大幅加快人脸渲染。近距离观察时，较少的采样可能表现为可见的色带。不过，在中等距离观察时，质量差异并不明显。

### 14.6.5 屏幕空间扩散

为场景中的所有网格渲染光照贴图并进行模糊，很快就会在计算和内存两方面变得昂贵。此外，网格需要渲染两次：一次渲染到光照贴图，一次渲染到视图中；光照贴图还必须具有合理的分辨率，才能表现小尺度细节所产生的次表面散射。

为解决这些问题，Jimenez 提出了一种屏幕空间方法 [831]。首先，照常渲染场景，并在模板缓冲区中标记需要次表面散射的网格，例如人脸。然后，对存储的辐亮度执行一个两遍的屏幕空间处理，以模拟次表面散射；借助模板测试，仅在需要的地方，即包含半透明材料的像素中，应用这一昂贵算法。附加的两遍处理分别沿水平和垂直方向应用两个一维双边模糊核。彩色模糊核是可分离的，但有两个原因使其无法以完全可分离的方式应用。首先，必须考虑线性视图深度，才能根据表面距离将模糊伸缩到正确的宽度。其次，双边滤波可以避免光在不同深度的材料之间泄漏，也就是避免本不应相互影响的表面发生相互作用。此外，还必须考虑法线朝向，使模糊滤波器不仅作用于屏幕空间，而且沿着表面的切向应用。最终，模糊核的可分离性就成了一种近似，但仍是高质量的近似。后来，又提出了一种改进的可分离滤波器 [833]。由于该算法依赖材料在屏幕上所占的面积，渲染人脸特写时成本较高。不过，这样的成本是合理的，因为这些区域恰恰需要高质量效果。当场景包含许多角色时，这一算法尤其有价值，因为可以同时处理所有角色。见图 14.44。


![图14.44 扫描人脸模型的屏幕空间次表面散射](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_6_14.44.png)

**图 14.44** 扫描人脸模型的高质量渲染。屏幕空间次表面散射使我们能够只用一次后处理，就为许多角色渲染逼真的人体皮肤材料。（左图：渲染图由萨拉戈萨大学的 Jorge Jimenez 和 Diego Gutierrez 提供；扫描数据由 XYZRGB Inc. 提供。右图：渲染图由 Jorge Jimenez 等人、Activision Publishing, Inc.（2013）以及萨拉戈萨大学提供；扫描数据由 Infinite-Realities 的 Lee Perry-Smith 提供 [831]。）

为进一步优化这一过程，可以将线性深度存储在场景纹理的 alpha 通道中。一维模糊所使用的样本数较少，因此在人脸特写中可能会看到欠采样。为避免这一问题，可以逐像素旋转滤波核，用噪声来掩盖重影伪影 [833]。使用时间抗锯齿（5.4.2 节）可以显著降低这种噪声的可见程度。

实现屏幕空间扩散时，必须注意只模糊辐照度，而不要模糊漫反射反照率或镜面光照。实现这一目标的一种方法，是将辐照度和镜面光照渲染到不同的屏幕空间缓冲区中。如果使用延迟着色（20.1 节），那么存储漫反射反照率的缓冲区已经存在。为降低内存带宽，Gallagher 和 Mittring [512] 提出，以棋盘格模式将辐照度和镜面光照存储在同一个缓冲区中。辐照度模糊完成后，将漫反射反照率与模糊后的辐照度相乘，再叠加镜面光照，合成最终图像。

在这一屏幕空间框架内，也可以渲染大尺度次表面散射现象，例如穿过鼻子或耳朵的光。在渲染网格的漫反射光照时，Jimenez 等人 [827] 的技术还会加入来自背面贡献的次表面透射：使用取反的表面法线 −**n**，对来自另一侧的入射光进行采样。然后用估计的透射率值调制结果。该估计利用从光源视点渲染的传统阴影贴图，通过采样取得深度，与下一小节介绍的 Dachsbacher 和 Stamminger [320] 的方法相似。为表示一个圆锥内的前向散射，可以多次采样阴影贴图。为降低渲染成本、减少每像素的样本数，可以对每个像素进行两次带随机偏移或旋转的阴影采样。这样会带来大量不希望出现的视觉噪声。幸运的是，实现半透明次表面光扩散本来就需要屏幕空间次表面模糊核，而这个核可以自动滤除上述噪声，无需额外成本。因此，每个光源只需多采样一次深度贴图，就能渲染高质量的半透明效果，模拟光在圆锥内前向散射并穿过人脸较薄部位的现象。

### 14.6.6 深度贴图技术

目前讨论的技术建模的是相对较短距离上的光散射，例如皮肤中的散射。对于表现出大尺度散射的材料，例如光穿过一只手的情况，就需要其他技术。其中许多技术专注于单次散射，因为它比多重散射更容易建模。

图 14.45 左图展示了大尺度单次散射的理想模拟。由于折射，光路径在进入和离开物体时会改变方向。为对单个表面点着色，需要将所有路径的效果相加。还必须考虑吸收：一条路径上的吸收量取决于它在材料内部的长度。即使对离线渲染器而言，为一个着色点计算所有这些折射光线也十分昂贵，因此通常忽略进入材料时的折射，只考虑离开材料时的方向变化 [823]。由于投射的光线总是朝向光源，Hery [729, 730] 指出，可以访问通常用于阴影的光源空间深度贴图，来代替光线投射。见图 14.45 中图。对于根据相位函数散射光的介质，散射角也会影响散射光量。


![图14.45 大尺度散射的路径与深度贴图近似](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_6_14.45.png)

**图 14.45** 左图为理想情况，光在进入和离开物体时都会折射；通过在材料内部进行光线步进，正确收集所有在离开物体时会恰当折射的散射贡献。计算消光 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_672ed2e5eb107d.png) 时，会计入每条路径的长度。可以使用路径追踪或少数实时近似方法来实现这种情况 [320]。中图展示了一种计算更简单的情况，光线只在出射时折射。这是实时渲染中的常见近似，因为从采样点（黄色）出发并考虑折射来寻找入射点（红色），并不容易直接做到。右图展示了另一种近似：只考虑一条光线，而不是沿折射光线进行多次采样，因此速度更快 [586]。

执行深度贴图查询比光线投射更快，但 Hery 的方法需要多个样本，对于大多数实时渲染应用仍然太慢。Green [586] 提出了一种更快的近似，如图 14.45 右图所示。虽然这种方法的物理依据较弱，其结果却可以令人信服。一个问题是，物体背面的细节可能会透显出来，因为物体厚度的每次变化都会直接影响着色颜色。尽管如此，Green 的近似仍足够有效，以至于 Pixar 将它用于《料理鼠王》（Ratatouille）等电影 [609]。Pixar 将这项技术称为软糖光照（gummi lights）。Hery 实现中的另一个问题是，深度贴图不应包含多个物体，也不应包含高度非凸的物体。这是因为该方法假设着色点（蓝色）与交点（红色）之间的整条路径都位于物体内部。Pixar 使用一种深度阴影贴图（deep shadow map）来绕过这一问题 [1066]。

实时建模大尺度多重散射相当困难，因为每个表面点都可能受到来自其他任意表面点的光的影响。Dachsbacher 和 Stamminger [320] 提出了一种称为半透明阴影映射（translucent shadow mapping）的阴影映射扩展，用于建模多重散射。辐照度和表面法线等附加信息被存储在光源空间纹理中。从这些纹理（包括深度贴图）中获取多个样本，并将其组合，形成对散射辐亮度的估计。NVIDIA 的皮肤渲染系统采用了这项技术的一种变体 [345]。Mertens 等人 [1201] 提出了一种类似方法，但它使用屏幕空间纹理，而不是光源空间纹理。

树叶也表现出强烈的次表面散射效果：光从后面照来时，叶片呈现明亮的绿色。除了反照率纹理和法线纹理，还可以在表面上映射一张表示穿过叶片体积的透射率 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_47bf3f045170e7.png) 的纹理 [1676]。然后，可以使用一个经验模型来近似光源产生的额外次表面贡献。由于树叶很薄，可以将取反的法线用作对另一侧法线 **n** 的近似。背光贡献可以计算为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_cfc3e492e42ed7.png)，其中 **l** 为光照方向，**v** 为观察方向。然后将它与表面反照率相乘，再叠加到直接光照贡献之上。

以类似的方式，Barré-Brisebois 和 Bouchard [105] 提出了一种成本较低的经验近似，用于网格上的大尺度次表面散射。首先，他们为每个网格生成一张灰度纹理，存储平均局部厚度：它等于 1 减去使用朝内法线 −**n** 计算得到的环境光遮蔽值。这张称为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_9271374fddb301.png) 的纹理被视作透射率的近似，可应用于从表面另一侧照来的光。叠加在常规表面光照之上的次表面散射计算为：


![数学公式](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_4154c700f8cffc.png)


其中，**l** 和 **v** 分别为归一化的光照向量和观察向量，p 是用于近似相位函数的指数（如书页 599 的图 14.10 所示），![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_06_1cd8968facd836.png) 为次表面反照率。随后，将这一表达式乘以光源颜色、强度和距离衰减。这个模型既不基于物理，也不满足能量守恒，但能够在单遍中快速渲染看起来可信的次表面光照效果。见图 14.46。


![图14.46 局部厚度纹理和次表面光散射效果](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_6_14.46.png)

**图 14.46** 左图为赫柏（Hebe）雕像生成的局部厚度纹理。中图展示利用它可以实现的次表面光散射效果。右图为使用相同技术渲染半透明立方体的另一个场景。（图片由 Colin Barré-Brisebois 和 Marc Bouchard 提供 [105]。）


## 14.7 头发与皮毛

来源：《Real-Time Rendering, Fourth Edition》，书页 640—648（PDF 物理页 661—669）。本节从 14.7 标题开始，至 14.8 标题之前结束，包含全部下级小节；技术陈述保留原书出版时的语境。

毛发是从哺乳动物真皮层生长出来的蛋白质丝状物。对于人类，毛发分布在身体的不同部位，包括头发、胡须、眉毛和睫毛等不同类型。其他哺乳动物通常覆盖着皮毛（浓密、长度有限的毛发），而皮毛的性质往往随动物身体部位的不同而变化。毛发可以是直的、波浪状的或卷曲的，各自具有不同的强度和粗糙度。毛发天然的颜色可以是黑色、棕色、红色、金色、灰色或白色，也可以染成彩虹中的各种颜色，只是染色效果不尽相同。

头发与皮毛的结构基本相同。它们由三层构成 [1052, 1128]，如图 14.47 所示：

- 最外层是毛小皮（cuticle），构成纤维的表面。这个表面并不光滑，而是由相互重叠的鳞片组成；鳞片相对于毛发方向倾斜约 α = 3°，使法线朝毛根方向倾斜。
- 中间层是皮质（cortex），其中含有赋予纤维颜色的黑色素 [346]。其中一种色素是真黑色素（eumelanin），产生棕色，其系数为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_b71ebfdad6949c.png)；另一种是褐黑色素（pheomelanin），产生红发，其系数为 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_797c0d2f6a8d17.png)。
- 内层是髓质（medulla）。在人类头发中，它很小，建模时常被忽略 [1128]。然而，对于动物皮毛，它占据毛发体积的比例更大，因此更为重要 [1052]。


![图14.47 毛发纵剖面](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_7_14.47.png)

图 14.47：一根毛发的纵剖面，展示了构成它的不同材料，以及沿方向 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_3dded78f3f358f.png) 入射的光所产生的各个光照分量。图内标注：root 为毛根，tip 为毛尖，cortex 为皮质，medulla 为髓质，cuticle scales 为毛小皮鳞片。

可以把毛发纤维看成类似粒子的东西，即对体积的一种离散化，只不过使用曲线代替了点。毛发纤维与光的相互作用由双向散射分布函数（BSDF）描述。它与 BRDF 对应，但光的积分范围是整个球面，而不只是半球面。BSDF 汇总了毛发纤维内部、穿越各个层次时发生的全部相互作用。第 14.7.2 节将详细介绍它。光在纤维内部散射，也会在许多纤维之间反弹；由此产生的多重散射现象会形成复杂的有色辐亮度。此外，纤维吸收光的程度取决于其材料和色素，因此，表现毛发体积内部出现的体积自阴影也十分重要。本节介绍较新的技术如何使我们能够渲染胡须等短毛、头发，以及最后的皮毛。

### 14.7.1 几何与 Alpha

可以利用顶点着色器代码，围绕美术人员绘制的毛发引导曲线挤出毛发四边形，将毛发渲染为沿引导线延伸的四边形带。每条四边形带都依照沿皮肤指定的朝向，跟随与其对应的毛发引导曲线，表示一簇毛发 [863, 1228, 1560]。这种方法非常适合胡须，或较短且大多静止的毛发。它也很高效，因为较大的四边形能够产生更大的视觉覆盖范围，因此覆盖整个头部所需的带状物更少，从而提高性能。如果需要更多细节，例如通过物理模拟驱动的细长头发，就可以使用更窄的四边形带，并渲染数千条。在这种情况下，最好还使用沿毛发曲线切线的圆柱约束，使生成的四边形朝向观察方向 [36]。即使毛发模拟只使用少数几条引导线，也可以通过插值周围引导线的属性来实例化新的发丝 [1954]。

所有这些元素都可以作为采用 alpha 混合的几何体来渲染。如果使用这种方式，就需要保证头发的渲染顺序正确，以避免透明度伪影（第 5.5 节）。为缓解这个问题，可以使用预先排序的索引缓冲区，先渲染靠近头部的发丝，最后渲染外层发丝。这对短而没有动画的头发很有效，但不适用于长而相互交错、具有动画的发丝。使用 alpha 测试，可以依靠深度测试解决排序问题。不过，对于高频几何和纹理，这可能产生严重的锯齿问题。可以采用 MSAA，并逐样本执行 alpha 测试 [1228]，代价是额外的样本和内存带宽。也可以采用任意一种顺序无关透明度方法，例如第 5.5 节讨论的那些方法。例如，TressFX [36] 存储最近的 k = 8 个片元，并在像素着色器中更新它们，只保持前七层有序，从而实现多层 alpha 混合 [1532]。

另一个问题是，在使用 mipmap 缩小 alpha 纹理时，会产生 alpha 测试伪影（第 6.6 节）。解决这个问题的两种办法是采用更智能的 alpha mipmap 生成方法，或者使用更高级的哈希 alpha 测试 [1933]。渲染细长发丝时，还可以根据它的像素覆盖率修改毛发的不透明度 [36]。

胡须、睫毛和眉毛等小范围毛发，比整个头部的完整头发体积更容易渲染。睫毛和眉毛甚至可以采用蒙皮几何体，以配合头部和眼睑的运动。这些小部件上的毛发表面可以使用不透明 BRDF 材质来计算光照。也可以使用下一小节介绍的 BSDF 为发丝着色。

### 14.7.2 头发

Kajiya 和 Kay [847] 开发了一种 BRDF 模型，用来渲染由有组织排列、无限细小的圆柱纤维组成的体积。第 9.10.3 节讨论过这个模型。它最初是为了渲染毛茸茸的物体而开发的，方法是在表示表面上方密度的体积纹理中进行光线步进。该 BRDF 用于表示光与体积相互作用时的镜面和漫反射响应，也可以用于头发。

Marschner 等人 [1128] 的开创性工作测量了人类毛发纤维中的光散射，并根据这些观测提出了一个模型。他们观察到单根发丝中不同的散射分量，图 14.47 展示了所有这些分量。首先，R 分量表示光在毛小皮的空气与纤维交界面处发生的反射，产生一个向毛根偏移的白色镜面峰值。其次，TT 分量表示光穿过毛发纤维的过程：第一次从空气透射进入毛发材料，第二次从毛发透射回空气。最后，第三个 TRT 分量表示光透射进入纤维，被纤维另一侧反射，随后再次透射到毛发材料之外的传播过程。变量名中的“R”表示一次内部反射。TRT 分量呈现为相对于 R 发生偏移的次级镜面高光；由于光在纤维材料内部传播时受到吸收，因此这个分量带有颜色。


![图14.48 金发与棕发的闪光](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_7_14.48.png)

图 14.48：通过路径追踪生成的金发（左）和棕发（右）参考图，其中渲染了由纤维横截面偏心率产生的镜面闪光。（图片由 d’Eon 等人提供 [346]。）

在视觉上，R 分量表现为头发上的无色镜面反射。当一团头发受到背面照明时，TT 分量表现为明亮的高光。TRT 分量对于渲染逼真的头发至关重要，因为它会在具有偏心率的发丝上产生闪光；也就是说，现实中的毛发横截面并非完美的圆，而更接近椭圆。闪光对于真实感十分重要，因为它使头发不会显得整齐划一。见图 14.48。

Marschner 等人 [1128] 提出了用于建模 R、TT 和 TRT 分量的函数，将它们作为毛发 BSDF 的组成部分，用来表示毛发纤维对光的响应。该模型正确考虑了透射与反射事件中的菲涅耳效应，但忽略了其他更复杂的光路，例如 TRRT、TRRRT 以及更长的路径。

然而，这个原始模型并不满足能量守恒。d’Eon 等人 [346] 对此进行了研究和修正。他们重新表述了 BSDF 的各个分量，通过更充分地考虑粗糙度与镜面反射锥的收缩，使其满足能量守恒。同时，这些分量也被扩展为包含 TR*T 等更长的路径。透射率还通过实测的黑色素消光系数进行控制。与 Marschner 等人 [1128] 的工作类似，他们的模型能够忠实地渲染具有偏心率的发丝上的闪光。Chiang 等人 [262] 提出了另一个能量守恒模型。该模型对粗糙度和多重散射颜色提供了更便于美术人员直观创作的参数化方式，使他们无需调整高斯方差或黑色素浓度系数。

美术人员可能希望为角色头发的镜面项设计特定外观，例如改变粗糙度参数。对于基于物理、满足能量守恒且保持能量的模型，头发体积深处的散射光也会随之改变。为了提供更多艺术控制，可以把最初的几种散射路径（R、TT、TRT）与多重散射部分分离 [1525]。实现方法是维护第二套 BSDF 参数，并仅将其用于多重散射路径。此外，BSDF 的 R、TT 和 TRT 分量随后可以用简单的数学形状来表示，让美术人员能够理解和调整，进一步完善外观。根据入射和出射方向对 BSDF 进行归一化，仍可使整套模型满足能量守恒。


![图14.49 实时头发散射分量](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_7_14.49.png)

图 14.49：包含 R、TT、TRT 以及多重散射分量的实时头发渲染。图内 Multi 表示多重散射，Complete model 表示完整模型。（图片由 Epic Games, Inc. 提供 [863, 1802]。）

以上介绍的每个 BSDF 模型都很复杂，求值代价高昂，主要用于电影制作的路径追踪环境。幸运的是，它们也存在实时版本。Scheuermann 提出了一种特设的 BSDF 模型，易于实现、渲染速度快，并且用于以大型四边形带渲染的头发时，外观很有说服力 [1560]。更进一步，可以将 BSDF 存入以入射和出射方向为索引参数的查找表（LUT）纹理，从而实时使用 Marschner 模型 [1128, 1274]。不过，这种方法可能使具有空间变化外观的头发难以渲染。为避免这个问题，一个较新的基于物理的实时模型 [863] 使用简化的数学形式近似已有工作中的分量，取得了令人信服的结果。见图 14.49。然而，所有这些实时头发渲染模型与离线结果相比，质量上仍有差距。简化算法通常不具备高级的体积阴影或多重散射。这些效果对于吸收较弱的头发，例如金发，尤其重要。

对于体积阴影，较新的解决方案 [36, 863] 依赖一个透射率值：沿光照方向，从最先遇到的毛发到当前纤维的距离记为 d，根据恒定的吸收系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_f883b72ca81859.png) 进行计算。这种方法实用且直接，因为它依赖任何引擎中都具备的常规阴影贴图。然而，它无法表示发丝聚成簇时产生的局部密度变化，而这一点对受到明亮照明的头发尤其重要。见图 14.50。为解决这个问题，可以使用体积阴影表示（第 7.8 节）。


![图14.50 头发体积阴影比较](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_7_14.50.png)

图 14.50：左：使用相对于第一个遮挡物的深度差，并采用恒定的消光系数，会产生过于平滑的体积阴影。中：使用深度阴影贴图 [1953]，能够获得更多透射率变化，与头发体积内部发丝聚簇的方式相吻合。右：将深度阴影贴图与 PCSS 结合，可以根据到第一个遮挡物的距离，获得更柔和的体积阴影（更多细节见第 7.6 节）。（图像使用 USC-HairSalon 提供的头发模型渲染 [781]。）

在渲染头发时，多重散射是一项求值代价高昂的项。适合实时实现的解决方案并不多。Karis [863] 提出了一种近似多重散射的方法。这个特设模型使用伪造法线（类似弯曲法线）、包裹式漫反射光照，并在毛发基色与光照相乘之前，先将基色提升到一个与深度有关的幂次，以近似光经过许多发丝散射之后的颜色饱和度。

Zinke 等人 [1972] 提出了一种更高级的双重散射技术。见图 14.51。之所以称为“双重”，是因为它根据两个因子评估散射光的量。首先，将着色像素与光源位置之间遇到的每根发丝的 BSDF 组合起来，求得全局透射率因子 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_fd4c43cf2ee3f9.png)。因此，![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_fd4c43cf2ee3f9.png) 给出了着色位置的入射辐亮度应当乘以的透射率。可以在 GPU 上通过统计一条光路上的毛发数量，并计算发丝的平均朝向，来求得这个值；后者会影响 BSDF，进而也影响透射率。可以利用深度不透明度映射 [1953] 或占用图 [1646] 来累积这些数据。其次，局部散射分量 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_c1349cb05287f2.png) 近似如下事实：到达着色位置的透射辐亮度，会在当前发丝周围的毛发纤维中散射，并对辐亮度产生贡献。这两项以 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_07_5e5a8ed01e8efa.png) 的形式相加，再通过该像素处发丝的 BSDF 计算，以累积光源贡献。这项技术代价更高，但它是对头发体积内部光多重散射现象的一种精确实时近似。它还可以与本章介绍的任意一种 BSDF 配合使用。


![图14.51 双重散射近似](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_7_14.51.png)

图 14.51：前两幅图展示用路径追踪渲染的头发参考结果：先仅包含三个毛发散射分量（R、TT、TRT），然后加入多重散射。后两幅图展示采用双重散射近似的结果：先用路径追踪渲染，再在 GPU 上实时渲染。（图片由 Arno Zinke 和 Cem Yuksel 提供 [1953]。）

对于具有动画的半透明材料，环境光照也是一个求值复杂的输入。常见做法是直接从球谐表示中采样辐照度。还可以根据头发静止姿态计算无方向性的预积分环境光遮蔽，用它对光照加权 [1560]。Karis 使用与多重散射相同的伪造法线，提出了一种环境光照的特设模型 [863]。

若要了解更多信息，可以在线获取 Yuksel 和 Tariq [1954] 提供的一套全面的实时头发渲染课程。在阅读研究论文、了解更多细节之前，这套课程会带你全面了解头发渲染的诸多领域，包括模拟、碰撞、几何、BSDF、多重散射以及体积阴影。头发在实时应用中已经可以呈现令人信服的效果。不过，要更好地近似基于物理的环境光照和毛发中的多重散射，仍然需要大量研究。

### 14.7.3 皮毛

与头发相比，皮毛通常被看作短而呈半有序排列的毛发，一般出现在动物身上。体积纹理这一概念与使用多层纹理进行体积渲染的方法相关；它是一种由多层二维半透明纹理表示的体积描述 [1203]。

例如，Lengyel 等人 [1031] 使用一组八张纹理来表示表面上的皮毛。每张纹理表示一组毛发在距表面某个给定距离处的切片。模型被渲染八次，每一次都使用顶点着色器程序，沿顶点法线将每个三角形稍微向外移动。这样，每个后续模型都表示表面上方的不同高度。以这种方式创建的嵌套模型称为壳层（shells）。这种渲染技术在物体轮廓边缘处会失效，因为层与层散开后，毛发会断裂成点。为了掩盖这种伪影，还会沿轮廓边缘生成鳍片（fins），并在鳍片上应用另一种毛发纹理来表示皮毛。见图 14.52，以及书页 855 的图 19.28。沿轮廓挤出鳍片的思路也可以用于增加其他类型模型的视觉复杂度。例如，Kharlamov 等人 [887] 使用鳍片与浮雕映射，为树木网格提供复杂的轮廓。


![图14.52 壳层与鳍片皮毛渲染](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_7_14.52.png)

图 14.52：使用体积纹理的皮毛。模型渲染八次，每一遍都让表面向外扩张少许。左图是八遍渲染的结果，注意轮廓处毛发的断裂。中图展示鳍片渲染。右图是同时使用鳍片和壳层的最终渲染结果。（图片来自 NVIDIA SDK 10 [1300] 的“Fur—Shells and Fins”示例，由 NVIDIA Corporation 提供。）

几何着色器的引入，使得在带有皮毛的表面上真正挤出折线毛发成为可能。《Lost Planet》使用了这项技术 [1428]。先渲染一个表面，并在每个像素保存皮毛颜色、长度和角度。几何着色器随后处理这幅图像，将每个像素变成一条半透明折线。通过为每个被覆盖的像素创建一根毛发，可以自动维持细节层次。皮毛分两遍渲染。首先渲染在屏幕空间中指向下方的皮毛，按屏幕从下到上的顺序排列。这样就能按照从后到前的顺序正确执行混合。在第二遍中，将其余指向上方的皮毛按照从上到下的顺序渲染，同样可以正确混合。随着 GPU 的发展，新的技术逐渐变得可行，并具有实际效益。

也可以使用前面几节介绍的技术。毛发可以渲染为从蒙皮表面上挤出的四边形几何体，例如《Star Wars Battlefront》游戏中的丘巴卡，或 TressFX 的 Rat 演示 [36]。

在把毛发渲染成细丝时，Ling-Qi 等人 [1052] 已经证明，将毛发模拟为均匀圆柱体是不够的。对于动物皮毛，相对于毛发半径，其髓质更暗、也更大。这会减弱光散射的影响。因此，他们提出了一个双圆柱纤维 BSDF 模型，能够模拟更广泛的头发与皮毛 [1052]。它考虑了 TttT、TrRrT、TttRttT 等更详细的路径，其中小写字母表示与髓质的相互作用。这种复杂方法能够产生更逼真的视觉效果，尤其适合模拟更粗糙的皮毛和精细的散射效果。这类皮毛渲染技术涉及大量毛发实例的光栅化，因此任何有助于缩短渲染时间的方法都值得采用。Ryu [1523] 提出，根据运动幅度和距离减少所渲染的毛发实例数量，以此作为细节层次方案。该方法已用于离线电影渲染，看起来也很容易应用于实时程序。


## 14.8 统一方法

来源：《Real-Time Rendering》第4版，书页648—649（PDF物理页669—670）。正文位于书页648，图14.53及图注位于书页649；章末“延伸阅读与资源”另存。

如今，体积渲染已经发展到了实时应用能够负担其开销的阶段。未来又可能实现什么呢？

在本章开头，我们曾说过“一切皆为散射”。考察参与介质材质时，可以通过采用较大的散射系数 ![数学符号](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/math/eq_14_08_0802b3e7faca08.png) 来得到不透明介质。再结合一种复杂的各向异性相函数，让它决定漫反射与镜面反射响应，就能形成不透明的表面材质。由此看来，是否存在一种方法，可以统一固体材质与体积材质的表示呢？

截至本书写作时，体积材质与不透明材质的渲染仍彼此分离，因为GPU当前的计算能力迫使我们针对不同使用情形采用各自高效的专门方法。我们用网格表示不透明表面，用进行alpha混合的网格表示透明材质，用粒子公告板表示烟雾体，并用光线步进实现参与介质中的某些体积光照效果。

正如Dupuy等人[397]所暗示的那样，或许可以用一种统一的表示方式来表示固体与参与介质。一种可能的表示方法是采用对称GGX（symmetrical GGX，SGGX）[710]，它是第9.8.1节介绍的GGX法线分布函数的扩展。在这种情况下，用于表示体积内具有朝向的薄片粒子的微薄片理论，取代了用于表示表面法线分布的微表面理论。从某种意义上说，与网格相比，细节层次会变得更加实用，因为它可以简单地转化为对材质属性进行体积滤波。这有望使大型且细节丰富的世界获得更一致的光照与表示，同时保持光照、形状，以及作用于背景的遮挡或透射率。例如，如图14.53所示，使用经过体积滤波的树木表示来渲染森林，可以消除可见的树木网格LOD切换，对细薄几何体进行平滑滤波，避免树枝引起的走样；与此同时，考虑到每个体素内实际包含的树木几何体，还能为背景提供正确的遮挡值。


![图14.53 使用SGGX及不同细节层次渲染的森林](https://raw.githubusercontent.com/ahuibo/Real-Time-Rendering-4th-CN/main/Real-Time_Rendering_4th_%E4%B8%AD%E6%96%87/assets/fig_14_8_14.53.png)

图14.53：上部为使用SGGX渲染的森林，细节层次从左向右逐渐降低。下部显示未经滤波的原始体素。（图片由Eric Heitz等人[710]提供。）


## 延伸阅读与资源

来源：《Real-Time Rendering》第4版，第14章章末，书页648—649（PDF物理页669—670）。本段跨页续接；夹排的图14.53及图注已归入14.8节，不在此重复。

这些资源在正文各处已经提及，但它们尤其值得关注，因此有必要在这里特别列出。Fong等人的课程讲义[479]讲解了一般的体积渲染，提供了大量背景理论、优化细节，以及电影制作中采用的解决方案。关于天空与云的渲染，本章以Hillaire内容丰富的课程讲义[743]为基础，其中的细节比我们在这里所能容纳的更多。体积材质的动画超出了本书的讨论范围。我们建议读者阅读这些关于实时模拟的文章[303, 464, 1689]，尤其是Bridson的整本专著[197]。McGuire的演讲[1182]以及McGuire与Mara的文章[1185]，可以帮助读者更广泛地理解与透明度有关的效果，以及可用于各种元素的一系列策略和算法。关于毛发与皮毛的渲染和模拟，我们再次建议读者参阅Yuksel与Tariq内容丰富的课程讲义[1954]。

出版标识页（PDF物理页671，对应书页650）：Taylor & Francis（泰勒与弗朗西斯）；Taylor & Francis Group（泰勒与弗朗西斯集团）；网址：http://taylorandfrancis.com。此页没有其他正文。
