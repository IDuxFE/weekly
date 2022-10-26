在梳理jpeg压缩原理的过程中，感受到了这是一项将开发技术与人类现有的生物特征很好地融合的一次运用，希望自己也能像前辈们那样，在计算机发展应用史上留下些许印记。

## 基础：人类的视觉生物特征
太阳光到达地球之后的光谱图与可见光的范围：
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac78efad01254c4da4bdd3b117e9bc5e~tplv-k3u1fbpfcp-watermark.image?)
我们人眼可见的光谱范围刚好是太阳辐射功率最强的区域，这不是偶然，而是生物在阳光下长期进化的结果。

***人类的眼睛并不完美，它们有自己的细微差别***

人眼中，有两种感光细胞：**视杆细胞**与**视锥细胞**；

- **视杆细胞**：对颜色不敏感，但是对弱光条件下看事物至关重要，对亮度敏感；
- **视锥细胞**：具有红、绿、蓝感色能力，对颜色敏感；

人类每只眼睛大约有**1亿**个视杆细胞，而视锥细胞只有**600万**个；因此，人眼对于图像的明暗程度（亮度）的感知能力，比对色彩（色度）的感知能力要强得多；

在此基础上，我们能利用如图所示的生物特性，做一些有意思的事情。

<br><br>

## 问：为什么要压缩图片
有一张长宽各为400个像素的彩色图，除去文件其他信息之外，仅仅是颜色信息，每一个像素都用R、G、B三个分量表示。因为每个分量有256个级别，要用8位（bit），即一个字节（byte）来表示，所以每个像素需要用3个字节。整个图像要占用400×400×3，约**480k**的磁盘空间，**它太大了**。完整、精确的去存储图片颜色信息，在大多数时候，是一种对磁盘、对带宽不必要的浪费。

<br><br>

## jpeg压缩的基本思路
简而言之，jpeg会分析图片的各个部分，合并相似的颜色，找到并删除人眼不易察觉的元素。

例1: 以下两种颜色，虽然在B分量上相差了60个单位，但我们大多数人的眼中，基本上可以忽略这两颜色上的差异

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffbe853564d74f85b617110bbe99c7b2~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e92d492d48fa4cf09bca23ca26587e08~tplv-k3u1fbpfcp-watermark.image?)


基于此，jpeg格式图片文件背后的算法： 
- 色彩空间转换，将RGB转换为YUV色彩空间，YUV的数据更好处理
- 色度缩减采样，将蓝红色度层的“分辨率”变小，因为人眼对颜色不敏感
- 离散余弦变换，找出人眼不敏感的高频信息 
- 量化，删除人眼不敏感的高频信息 
- 游程（zigzag）编码与霍夫曼编码，通用数据压缩

<br>

### 色彩空间转换
**从RGB到YUV -- 无损**

根据每个像素的R、G、B的三个值， 算出三个新的数值：
**亮度、少量的绿色色度（Y）**、**蓝色色度（Cb）**、**红色色度（Cr）**。这个过程是可逆的，转换过程中没有删除任何数据。
RGB 转化 YUV 的公式（经过 PAL制式 CRT伽玛校正）如下：


> Y = 0.299R’ + 0.587G’ + 0.114B';
> 
> U = -0.147R’ - 0.289G’ + 0.436B';
> 
> V = 0.615R’ - 0.515G’ - 0.100B';

未经伽玛校正（计算机色彩空间）的矩阵表示：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70986ad8ac5149e2a247df317d7b3971~tplv-k3u1fbpfcp-watermark.image?)

V、U的值对于颜色的影响：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a80ed72da2b4eca99bfc78387c411c3~tplv-k3u1fbpfcp-watermark.image?)
（图片来自 Tonyle, Wikimedia Commons, [File:YUV UV plane.svg](https://link.zhihu.com/?target=https%3A//commons.wikimedia.org/wiki/File%3AYUV_UV_plane.svg)）

<br>

### 色度缩减采样
基础步骤：
1. 将**蓝色**和**红色**色度分量层上的像素按照2x2像素成一个区块划分；![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea07417de33a4825907f5c0d4142402e~tplv-k3u1fbpfcp-watermark.image?)
2. 然后计算每个区块的平均色度值，并删掉重复的信息；
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b22162312f364517be92dc0ec75534c7~tplv-k3u1fbpfcp-watermark.image?)
3. 缩小图像，使得含有一个平均值的由四个像素组成的区块只占一个像素的空间，因此，那些人眼不易感知的红蓝色度信息的量，被缩减到原来的四分之一，而亮度保持不变；

信息量对比：
- before： 1 + 1 + 1 = **3.0**;
- after:   1/4 + 1/4 + 1 = **1.5**;

经过以上基础步骤，图像的大小就变成了原来的一半。
然后重新根据亮度、蓝色色度、红色色度的值重新计算出RGB值。

<br>

### 离散余弦变化（DCT）
通过利用人眼不擅长感知图像中的高频率元素这一事实来实现。

**图像基础**<br>
图像的频率：灰度值变化剧烈程度的指标，是灰度在平面空间上的梯度<br>
低频就是颜色缓慢地变化,也就是灰度缓慢地变化,就代表着那是连续渐变的一块区域,这部分就是低频. 对于一幅图像来说，除去高频的就是低频了，也就是边缘以内的内容为低频，而边缘内的内容就是图像的大部分信息，即图像的大致概貌和轮廓，是图像的近似信息。<br>
反过来, 高频就是频率变化快.图像中什么时候灰度变化快?就是相邻区域之间灰度相差很大,这就是变化得快.图像中,一个影像与背景的边缘部位,通常会有明显的差别,也就是说变化那条边线那里,灰度变化很快,也即是变化频率高的部位.因此，图像边缘的灰度值变化快，就对应着频率高，即高频显示图像边缘。图像的细节处也是属于灰度值急剧变化的区域，正是因为灰度值的急剧变化，才会出现细节。
综上，图像的主要成分是低频信息，它形成了图像的基本灰度等级；中频信息决定了图像的基本结构，形成了图像的主要边缘结构；高频信息形成了图像的边缘和细节，是在中频信息上对图像内容的进一步强化

**基本思路**<br>
遍历图像的各个部分，找到具有高频率的色度或亮度的像素频繁出现的区域，然后将这些人眼很难感知的元素删除；

**基本步骤**（以亮度层为例）<br>
1. 将图像按8x8个像素划分成多个区块，每个区块64个像素，每个像素用0~255的数值代表亮度；（采用比 8*8 更大的像素组，会大幅增加 DCT 的运算量，且编码质量也不会明显提升；采用比 8*8 更小的像素组会导致分组增多降低精度。所以8*8 的像素组是效率最优的结果）
2. 将值通过减去128的方式，改变数值的取值范围为-128~127；其中-128为黑色，127为白色；（贴近余弦波形）
3. 这些区块可以被 8*8 个余弦波精确表示，如下图所示有64个基本余弦波
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/969d58a70373427b87f40132dcae136a~tplv-k3u1fbpfcp-watermark.image?)
4. 这64个余弦波，可以组合成任意 8*8 的图形。我们只要用系数（系数表示每个单独的余弦波对整体图像所做的贡献）对这64个余弦波进行加权，就可以表示出任何的图形。
JPEG 中使用的是 DCT II 的公式，通过公式，可以得出该图像在频域上的系数（经过取整，取值范围 -1024～1023）

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ad8db1598ab4ace91723c65e14ff0e0~tplv-k3u1fbpfcp-watermark.image?)

其中，f(i)为原始的信号，F(u)是DCT变换后的系数，N为原始信号的点数，c(u)可以认为是一个补偿系数，可以使DCT变换矩阵为正交矩阵。


如下所示，左侧的图形A可以由右侧的带上不同系数的64个余弦波组成：
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3e9a02d264448309aeb1bffc60d8fa5~tplv-k3u1fbpfcp-watermark.image?)

<br>

### 量化
JPEG算法提供了两张标准的量化系数矩阵（不同的压缩程度，有各自对应的系数矩阵），分别用于处理亮度数据Y和色差数据Cr以及Cb。

![jpeg_032.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ede88b6edc224af1b31b11f00db988f8~tplv-k3u1fbpfcp-watermark.image?)

![jpeg_033.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccffc3f3c37a48bfb8c5235a57621681~tplv-k3u1fbpfcp-watermark.image?)

将DCT得到的每个区块的系数矩阵中的值，除以量化表的对应值，并将每个结果四舍五入为最接近的整数。
如图：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7fb485af5f64c0c82188b9a39bbce1c~tplv-k3u1fbpfcp-watermark.image?)

**此过程为有损的过程，在解码时，反量化会乘回量化表的相应值。由于存在取整，低频段会有所损失，高频段的0字段则会被舍弃，最终导致图像质量降低。**

### 游程（zigzag）编码与霍夫曼编码
列出所有区块中的亮度和色度数值

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71360c6e1c8143568d4cdefda5003c6e~tplv-k3u1fbpfcp-watermark.image?)

在列出来的数字中，使用游程编码算法：不列出所有的0，只说有多少个0：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a171a1fc6604c55a24e682bfea1ebba~tplv-k3u1fbpfcp-watermark.image?)

而后将获得的数据进行[霍夫曼编码](https://www.bilibili.com/video/BV1pL4y1E7EQ/?vd_source=71bf2881d4b01fdfc5fb281b02e1720f)，实现一个区块数据的压缩。
遍历图片里的所有区块，即可实现整张图片的压缩保存。


于是乎，整张图片的所有的颜色信息，即能按照想要的压缩等级，将颜色近似存储下来。能在保证我们获取到图片里的信息至于，还能为我们节省磁盘、带宽。



<br><br>

## 一些由于jpeg压缩引发的趣事
早些年，智能机刚起步那几年，经常能看到一些网图变绿，其实也与jpeg压缩有关：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1e738272e7342fe846aee43c8d9853c~tplv-k3u1fbpfcp-watermark.image?)；

在Android 7 之前的系统版本里，提供的压缩接口中，为达到更快的速度，而降低了计算的精度（从16位定点数，降低到8位定点数）， 并直接阶段了小数部分，即全部往负数方向取整。如果app直接调用的Android系统提供的接口处理图片的话，由图得知，YUV 值向负方向取整，结果是呼之欲出的：变暗，变绿

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a80ed72da2b4eca99bfc78387c411c3~tplv-k3u1fbpfcp-watermark.image?)
<br>（图片来自 Tonyle, Wikimedia Commons, [File:YUV UV plane.svg](https://link.zhihu.com/?target=https%3A//commons.wikimedia.org/wiki/File%3AYUV_UV_plane.svg)）

```
int y = ( CYR*r + CYG*g + CYB*b ) >> CSHIFT; 
int u = ( CUR*r + CUG*g + CUB*b ) >> CSHIFT; 
int v = ( CVR*r + CVG*g + CVB*b ) >> CSHIFT;
```



So，写代码要小心。。。

Over.


