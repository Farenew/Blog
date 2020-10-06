---
title: "色彩理论详解 I"
date: 2018-05-23
tags: ["设计", "理论"]
categories: ["研究"]
---

> 这个是关于色彩理论介绍的第一部分, 主要介绍物理上的光和各种色彩模型。

## 光与色

要想了解颜色的由来，须深入其原理，究其根本。

寻其根本，当知色来源于光。不同的光(无论是自身发出，还是反光)构成了不同的色。故关于色的讨论，本质就是关于光的讨论。

学过基础物理的都知道，光是一种波，同时也是一种粒子，称之为波粒二象性。从波的角度来看，光是电磁频谱内的某一部分的电磁辐射。电磁波主要的一个性质就是波长(或者是频率，二者可以在光速常数$c$下互相转换)。对于可见光来讲，大概也就是400~700nm波长的电磁辐射。

![](/img/色彩理论详解I/1.png)

从这个图很容易理解，不同波长的光构成了不同的颜色。然而光靠这个可以描述我们所能看到的各种颜色吗? 恐怕不行。首先，这里面就没有一种最常见的颜色----白色。说到这里，可以很容易地就想到了，不光要靠不同波长的颜色，同时，还需要不同波长的颜色相互组合。那么加上不同波长的光相互组合就够了吗? 恐怕也不行，那么那些熟悉的词，像是饱和度，明度，亮度都又是怎么回事呢? 这里就来详细介绍一下。

### 色彩模式

熟悉Photoshop的都知道，在photoshop里，对于颜色的描述，有**HSB**，**LAB**，**RGB**和**CMYK**四种主要的模式。除了这些模式，还有用于描述黑白的**灰度和位图模式**，为了特殊目的打印的**双色调和多通道模式**，为了减少存储的**索引颜色模式**。

我们关注的重点在于开始的四种，思考一下，他们有什么特点?

很容易想到的，这些模式中都至少有三个参数。这也是需要明白的第一个特点，**从数学上来讲，颜色是三维的一个特性**，因此至少需要三个不同的参数来定义。**把不同的参数在空间上绘制，就构成了一组坐标系，也就构成了一个色域空间**。色域空间中的每一个点，都可以代表一个颜色。类比数学中的极坐标，球坐标，可以明白，使用不同的坐标系的主要目的在于方便。但除了方便使用，不同的体系也代表了不同的科学目的，比如RGB从光的角度来定义，Lab是关于人眼的研究给出的定义等。下面就对几种主要的模式加以说明。

### `RGB`

RGB指的是Red，Green and Blue，这个想必也是大部分人最熟悉的关于色彩的说法了。**RGB色彩，指的是通过光的三原色相互组合，来得到各种不同的颜色。也就是说，RGB指的是三种波长不同的光，即primary colors。** 这里一个有趣的问题就是，为什么偏偏是这三种呢? 按照理论来讲，任意位置的三种光应该都是可以的，关于这个原因，我们在第二部分会讲到。

在Photoshop中，常用的都是8位的色彩，在新建图像的时候就可以确定。8位指的是单一色彩的值，"位"这个词相比很容易就想到2进制，确实如此，8位就表示了2的8次幂，也就是从0~255共256个数字。

![](/img/色彩理论详解I/2.png)

所以在RGB中的调节，每个光原色都有256个值，如果是黑色，那么就是三种光都不发，RGB(0,0,0)，如果是白色，那么就是三种光都最强，RGB(255,255,255)。而其它强度的光相互组合，就生成了RGB这个色域空间。

在Photoshop里，对每一种颜色都用channel(通道)来表示，每个channel都保存了当前图像一种光原色的全部信息。三个通道叠加，就可以看到全部的信息。这样的话，就可以很容易地算出来，在8位RGB的色域空间里，共有$256 \times 256\times256=1677216$种颜色。

> 这里讲到的channel的叠加也是一个很有趣的话题，如果熟悉Photoshop中的图像混合模式的话，可能就会对这个叠加有所怀疑。在这里，叠加包含两方面的含义：
> 
> 1. 光的叠加。对于叠加的结果，是以光的形式加以叠加的。比如开启红色通道，那么屏幕就会点亮LED或者LCD等部件的红色发光件。如果再开启绿色，那么绿色发光件会被开启。
> 2. 色值的叠加，在Photoshop里，色值在直方图中被表示，这里的叠加指的是平均值。比如RGB(13,15,200)，那么色值就是$\frac{13+15+200}{3} = 76$

此外，还会经常听到一个词就是sRGB，所谓的sRGB，是早期为了表示通过显像管来显示图像时的一种标准，由HP和Microsoft共同开发，并得到了广泛的支持，多广泛呢? 几乎你在所有的显示器，无论是电脑显示器，还是照相机显示器，还是手机显示器，基本都是基于这套标准。如果你是一个geek or something，你也许曾经点开过这个：

![](/img/色彩理论详解I/3.png)

从这里就可以看到计算机使用的便是sRGB。Microsoft面向设计师推出的Surface Studio，就是把更换色彩配置文件加了一个快捷键。比如切换成Adobe RGB。

那Adobe RGB又是什么呢? Adobe RGB是由于sRGB在打印的时候，对于蓝色和绿色会存在一些损失，为了更加适应打印的效果，Adobe提出了一种Adobe RGB的方案，也得到了比较广泛的支持和使用。但相比起更适合人眼的sRGB来讲，它的颜色表现不够自然。此外，要表现这样的效果，需要软件和硬件的支持，浏览器并不支持这种方案，而打印店的打印设备，也极少会专业到需要选择这样的色彩模式来工作。就我个人日常所接触到的一些设计师也多半对此一知半解，甚至还有一些号称是设计师结果用一块不到1000块钱的屏幕做设计的。。。。。(明明是美工好伐！！！大师不是想做就能做的！！！)


### `HSB`

说起来HSB可算是源远流长。先说简单的，HSB在绘画，摄影等方面被谈的比较多。特别是画家的调色，基本都是基于Hue来进行tine, shade和tone。在Photoshop里，HSB指的是hue，saturation和brightness。

除了HSB，可能还会听到HSV，HSL等说法，这些都属于类似的color model，他们同样是使用三个参数来描述色彩，这里的三个参数分别是**Hue**，**类似Saturation的一个参数**，**类似Luminance的一个参数**。而后两个参数的一些细微的差异，就构成了HSV，HSB等不同的color model。**其中Photoshop的HSB，实际上就是HSV**。

这个模型最早是Munsell在1930s提出的，他提出的这个color model既在科学上反映了人眼对于色彩的感知，同时也方便用在艺术，设计等方面的创作和教学，是一项跨时代的重大创造。后来对人眼的进一步研究以及色彩的理解，国际照明委员会(International Commission on Illumination，CIE)提出来更加先进的CIELAB(即phtoshop里的Lab)以及CIECAM02.

![](/img/色彩理论详解I/Munsell-system.svg)

从这张图可以看出来，在Munsell提出来的时候，使用了Hue，Chroma和Value三个值来表示颜色，在Hue中，他提出了五种基础的Hue，分别是Red, Yellow, Green, Blue, and Purple，还有一个中间色YR(Yellow Red)。

---

基于Munsell的理论，在1970s计算机图形学者们提出了HSL和HSV，并在计算机色彩中广泛应用。

依据三个坐标，类似Munsell的绘制方法，可以把色域空间绘制在一个圆柱中：

HSL            |  HSV
:---:|:---:
![](/img/色彩理论详解I/4.png)  |  ![](/img/色彩理论详解I/5.png)

如果对交换后两个坐标，那么就可以绘制成圆锥和双锥：

HSL            |  HSV
:---:|:---:
![](/img/色彩理论详解I/6.png)  |  ![](/img/色彩理论详解I/7.png)

可以看出来，在HSV中，V达到最大值(最大值为1)的时候，得到了纯色，而在HSL中，只要L达到最大(最大值是100%)，那么一定是白色。

#### 关于参数的进一步讲解：

- Hue，指的是色环中颜色的位置，不同的Hue表示了光的不同波长。红色(Red)位于0°，绿色(Green)位于120°，蓝色(Blue)位于240°。它们之间就是混合而成的黄色(Yellow)，青色(Cyan)和洋红(Magenta)。
- 光强，在HSL中，使用Lightness表示，取值0~100; 在HSV中，使用Value表示，取值0~1。这个参数与Luminous intensity有关(其单位属于SI单位之一)，表征了光源给定方向上单位立体角内发光强弱程度。**注意这个强度千万不能理解成光波的振幅**，电磁波的intensity和机械波是不一样的，计算的公式也是不同的，关于如何计算属于物理的范畴，这里就不赘述了。
- Saturation，指的是颜色的纯度，这个参数与光强度和它在不同波长光谱中的分布量决定。是一个和当前的光强度以及色环角度同时有关的参数，对于HSL和HSV中略有不同。

	当Saturation达到100%，在HSL中，L达到50%; 在HSV中，V达到1，就得到了所谓的pure color(纯色)。

  - 在HSV中，对纯色shade不会改变Saturation。
  - 在HSB中，对纯色shade或者tint都不会改变Saturation。

#### 对于RGB和HSL，HSV之间的相互转换：

- **RGB与HSV的转换(这里使用8位的RGB)：**

$$R'=R/255$$

$$G'=G/255$$

$$B'=B/255$$

$$Cmax = max(R',G',B')$$

$$Cmin = min(R',G',B')$$

$$\Delta = Cmax - Cmin$$

Hue:

$$H =
  \begin{cases}
    0^{\circ}     & \quad \Delta = 0\\\\\\
    60^{\circ}  \times (\frac{G'-B'}{\Delta}mod\;6) & \quad Cmax = R' \\\\\\
    60^{\circ}  \times (\frac{B'-R'}{\Delta} + 2) & \quad Cmax = G' \\\\\\
    60^{\circ}  \times (\frac{G'-B'}{\Delta} + 4) & \quad Cmax = B'
  \end{cases}
$$

Saturation:

$$S=
  \begin{cases}
    0 & \quad Cmax = 0\\\\\\
    \frac{\Delta}{Cmax}   & \quad Cmax \neq 0
  \end{cases}
$$

Value:

$$V = Cmax$$

- **RGB与HSL的转换(这里使用8位的RGB)：**


$$R'=R/255$$

$$G'=G/255$$

$$B'=B/255$$

$$Cmax = max(R',G',B')$$

$$Cmin = min(R',G',B')$$

$$\Delta = Cmax - Cmin$$

Hue:

$$H =
  \begin{cases}
    0^{\circ}        & \quad \Delta = 0\\\\\\
    60^{\circ}  \times (\frac{G'-B'}{\Delta}mod\;6) & \quad Cmax = R' \\\\\\
    60^{\circ}  \times (\frac{B'-R'}{\Delta} + 2) & \quad Cmax = G' \\\\\\
    60^{\circ}  \times (\frac{G'-B'}{\Delta} + 4) & \quad Cmax = B'
  \end{cases}
$$

Saturation:

$$S=
  \begin{cases}
    0 & \quad \Delta = 0\\\\\\
    \frac{\Delta}{1 - |2L-1|}   & \quad Cmax \neq 0
  \end{cases}
$$

Lightness:

$$V = (Cmax + Cmin)/2$$


> **一些补充说明的内容：**
>
> 1. 	对Hue的三种调整：
>
>           Tint, 指的是添加白色, lightness会增加
>
>           Shade, 指的是添加黑色, lightness会降低
>
>           Tone,  指的是添加灰色,
> 2. 关于 Munsell提出的color system的详细介绍参考wikipedia：[Munsell Color System](https://en.wikipedia.org/wiki/Munsell_color_system)
> 3. Magenta在Photoshop里被翻译为洋红，有的翻译为品红，这里出现的中文以Photoshop为主。
> 4. 对HSV和HSB更详细的介绍，参考wikipedia: [HSL and HSV](https://en.wikipedia.org/wiki/HSL_and_HSV)
> 5. 关于HSV，HSL和RGB的转换有不同种类的公式，这里取了其中一种形式上看着较为舒服的。参考[RGB to HSV](https://www.rapidtables.com/convert/color/rgb-to-hsv.html), [RGB to HSL](https://www.rapidtables.com/convert/color/rgb-to-hsl.html)

### `CMYK`

CMYK就很好理解了, 它的出现并不属于色彩学意义上的创新, 使用CMYK仅仅是因为我们使用CMYK(Cyan, Magenta, Yellow以及Black)四种油墨作为打印。依据使用的油墨, 建立这种颜色模型。

这里很自然的, 会想到两个问题：

1. 为什么使用这四种油墨? 而不是使用Red, Green, Blue这三种油墨?
2. 等等.....油墨的话, 怎么能体现色彩的连续变化呢? 打印的时候可没说可以羼水啊。

首先说油墨的选择。选择油墨很自然会想到两个问题, 1. 能不能正确地表现所创作的内容。2. 价格低廉。

油墨是怎么表现颜色的呢? 从物理学中很好理解, 既然是打印内容, 那么它们本身就是不发光的, 我们所看到的内容是通过光穿过油墨, 打在纸上, 反射回到我们的眼睛里而看到的。在这个过程中, 穿过油墨的时候, 一部分光会被吸收, 剩下没有被吸收的光会反映出色彩。这时候, 就体现出CMY的优势了, **RGB是光的三原色, 使用RGB的颜料时, 每一种颜料都会吸收另外两种光, 而CMY是二次色, 它们仅仅会吸收一种原色**。

通过一个表格来说明不同的颜料的吸收效果：

颜料颜色  | 吸收的光  |  看到的颜色
:--:|:---:|:--:
  |   |  
Red  | Green&Blue  | Red
Green  | Red&Blue  | Green
Blue  | Red&Green  | Blue  
Cyan  | Red  | Cyan
Magenta  | Green  |  Magenta
Yellow  |  Blue | Yellow  
Black  | Red&Green&Blue  | Black  
  |   |  
Red+Green  | Red&Green&Blue  | Black  
Red+Blue  | Red&Green&Blue  |  Black
Blue+Green  | Red&Green&Blue  | Black
  |   |  
Cyan+Magenta  | Red&Green  | Blue
Cyan+Yellow  | Red&Blue  |  Green
Magenta+Yellow | Green&Blue | Red

根据这些内容就能看出来CMY颜料的好处了。然而用这几种颜料就能表现创作的内容吗? 根据这个表格, 可以看到使用CMYK颜料下, 我们可以获得RGB,CMY以及white和black共八种颜色。其它的颜色呢?难道需要每种颜色都提供一个墨盒吗? 这里就是在印刷业的另一个创新, 通过Hafltone(半色调)来在几种颜色的基础上, 创造出连续的颜色。

Halftone说起来很简单, 既然我们不能给油墨加水来获得不同程度的颜色, 那么我们就**在打印的时候选择多打印或者少打印来体现色彩的变化**, 在高的分辨率下, 人眼是无法识别出这种多少的变化而是会当做不同的颜色处理的。直接用一张图就能很容易地说明了：

![](/img/色彩理论详解I/Halftoning_introduction.svg)

此外, 在实际打印中, CMYK在打印的时候会改变打印角度, 来进一步提升打印质量并且减少莫尔条纹。这部分内容就涉及过多的印刷知识了, 不多介绍了。

### `LAB`

到了LAB色域空间的时候, 情况就开始变得复杂了起来。引入Lab是源于科学的进步, 对人眼和颜色的进一步认识。

Lab是来自CIE XYZ坐标的非线性压缩变换得到。而CIE XYZ坐标则是来自于LMS坐标的平均化, LMS坐标表征了人眼中视锥细胞对不同刺激的反应程度。L,S,M分别指的是对不同峰值的光谱, L表示长波段(Long, 560nm~580nm), S表示短波段(Short, 420nm~440nm), M表示中波段(Middle, 530nm~540nm)。这里仅做一个简单说明, 关于人眼对色彩的感知在下一部分展开叙述。

---

Lab中的L指的是Lightness, a和b分别表示一种色彩维度, a表示green-red颜色, b表示blue-yellow颜色。在a通道和b通道里, 两个极值分别就是两种颜色。

Lab的好处有两个：

- 首先, 它是基于人眼对于光线的感知给出的定义。也就是说, 它所定义的色彩和人眼对色彩的感知分布是相似的(perceptually uniform), 也就是说在这个定义下, 对于Lab数值的改变, 和人眼感受的变化的程度是差不多相同的。
- Lab的另一个优势在于Lab是设备无关的, 并且具有极大的色域空间。这个好处在于当我们使用Lab描述颜色的时候, 不需要考虑设备的性质。(前面讲到的, sRGB就是为了适应阴极射线管而创造的)。

虽然Lab很强大, 不过这个模型较为复杂, 因此就我接触的一些人来看。。。很少有熟练使用Lab来进行调色等工作的。实际上Lab由于很适合人眼感知, 因此其调色也更为自然而不是基于物理模型。例如L通道的亮度几乎和人眼感知的亮度是完全一致的, 而在a和b通道中, 由于完全剥离了亮度信息, 因此颜色的修改更为纯粹, 完全不会引起关于亮度的变化。如果有大佬对这方面内容比较熟悉, 还望不吝赐教。

Lab的缺点在于：

- 一方面, 要完整地表现Lab色域空间, 图像需要存储更多信息, 这个在Lab提出的早期比较麻烦, 但现在大数据时代这个已经不再成为问题。
- 另一方面, Lab定义下的色彩是一个非常大的色域空间, 它不仅真包含了RGB和CMYK等色域空间, 同时一些显示也是人眼所无法识别的。因此在一些图片编辑软件中, 会按照人眼所能识别的最近颜色, 把那些人眼无法看到的颜色加以转换。

### Further Reading

1. 关于不同的Color System的参考：[color system](http://www.colorsystem.com/?lang=en)
2. Munsell Color的历史：[History for Color System](http://munsell.com/color-blog/history-of-color-systems/)
3. NSC color system: [NSC color system](http://ncscolour.com/about-us/how-the-ncs-system-works/)
4. Introduction to Color Theory: [Color Theory](http://infohost.nmt.edu/tcc/help/pubs/colortheory/web/index.html)
5. Saturation: [Colorfulness](https://en.wikipedia.org/wiki/Colorfulness)
6. CMYK ink：[why use CMYK](https://www.quora.com/Why-do-consumer-printers-mix-cyan-magenta-and-yellow-instead-of-red-green-and-blue-RGB-seems-to-be-the-way-many-graphics-apps-present-the-color-spectrum-and-it-seems-to-be-the-way-that-displays-represent-color-so-why-do-printers-then-use-CMYK)


## 参考

[1. 关于颜色模式的叙述](https://helpx.adobe.com/cn/photoshop/using/color-modes.html)

[2. 关于sRGB vs. Adobe RGB](https://kenrockwell.com/tech/adobe-rgb.htm)


