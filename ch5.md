## [5.JPEG压缩模式](https://images.guide/#jpeg-compression-modes)

JPEG图像格式有多种不同的[压缩模式](http://cs.haifa.ac.il/~nimrod/Compression/JPEG/J5mods2007.pdf)。流行的三种模式是基线(顺序)、渐进式JPEG(PJPEG)和无损。

**基线(又叫顺序)JPEG和渐进式JPEG有什么不同呢？**

基线JPEG(大多数图像编辑和优化工具默认的压缩模式)是以一种相对简单的方式编码和解码的：自上而下。当基线JPEG加载在缓慢或不稳定的连接上时，用户会看到图像的顶部，并将更多的图像显示为图像加载。与无损JPEG类似，但压缩比较小。

![Performance](https://images.guide/images/book-images/Modern-Image6-large.jpg)

基线JPEG从上到下的模式加载图像，而渐进式JPEG从模糊到锐利的模式加载图像。

渐进式JPEG将图像划分为多个扫描区域。第一次扫描以模糊或低质量设置显示图像，后续扫描可提高图像质量。把这个过程看作是“逐步”改进它。图像的每一次“扫描”都会增加更多的细节。当合并在一起后，创造了一个完整的质量图像。

![Performance](https://images.guide/images/book-images/Modern-Image7-large.jpg)

基线JPEG从上到下加载图像。PJPEG从低分辨率（模糊）到高分辨率加载图像。Pat Meenan还编写了一个交互式工具来测试和学习渐进式JPEG扫描。

通过把数码相机或编辑器添加的[EXIF数据删除](http://www.verexif.com/en/)，优化图像的[Huffman表](https://en.wikipedia.org/wiki/Huffman_coding)或重新扫描图像，都可以实现无损JPEG优化。 像[jpegtran](http://jpegclub.org/jpegtran/)这样的工具通过重新排列压缩数据而不会降低图像质量来实现无损压缩。 [jpegrescan](https://github.com/kud/jpegrescan)、[jpegoptim](https://github.com/tjko/jpegoptim)和[mozjpeg](https://github.com/mozilla/mozjpeg)（我们将在稍后介绍）也支持JPEG无损压缩。

<h3 id="the-advantages-of-progressive-jpegs"><a href="https://images.guide/#the-advantages-of-progressive-jpegs">5.1 渐进式JPEG的优点</a></h3>

PJPEG能够在加载图像时提供低分辨率的“预览” —— 用户可以感觉到与自适应图像相比，图像加载速度更快。

在较慢的3G网络连接上，只接收到部分文件时，用户可以(粗略地)查看图像中的内容，并调用是否等待文件完全加载。这可能比基线JPEG提供的自上而下显示图像的方式更令人愉快。

![Performance](https://images.guide/images/book-images/pjpeg-graph-large.png)

在2015年，[Facebook改用了PJPEG(用于他们iOS应用程序)](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)节省少了10%数据流量。他们能够比以前显示一个高质量的图像快了15%，优化感知到加载时间，如上图所示。

对于超过10KB的图像，PJPEG可以提高压缩性能，与基线/简单JPEG相比，带宽减少[2%-10%](http://www.bookofspeed.com/chapter5.html)。它们的压缩比更高，这要归功于JPEG中的每次扫描都能够有自己的专用的可选的[Huffman表](https://en.wikipedia.org/wiki/Huffman_coding)。现代JPEG编码器(如[libjpeg-turbo](http://libjpeg-turbo.virtualgl.org/)、MozJPEG等)利用PJPEG的灵活性更好地打包数据。

> 注意：为什么PJPEG压缩更好？基线JPEG一次编码一个块。使用PJPEG，可以对多个块相似的**[离散余弦变换](https://en.wikipedia.org/wiki/Discrete_cosine_transform)**系数进行编码，从而获得更好的压缩效果。

<h3 id="whos-using-progressive-jpegs-in-production"><a href="https://images.guide/#whos-using-progressive-jpegs-in-production">5.2 谁在生产中使用渐进式JPEG？</a></h3>

+ [twitter.com推出了“渐进式JPEG”](https://www.webpagetest.org/performance_optimization.php？test=170717_NQ_1K9P&run=2#compress_images) 以85%的质量为基准。他们测量了用户感知的延迟(第一次扫描的时间和总体的加载时间)，发现总的来说，PJPEG在满足低文件大小、可接受的代码转换和解码时间方面更具有竞争力。

+ [Facebook为他们的iOS应用程序发布了渐进式JPEG。](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)他们发现它将减少了10%数据使用，并使他们显示高质量的图像能够提速15%。

+ [Yelp转向使用渐进式JPEG](https://engineeringblog.yelp.com/2017/06/making-photos-smaller.html)发现这在一定程度上是他们减少图像大小约4.5%的原因。他们还使用MozJPEG额外节省了13.8%。

许多其他图像丰富的网站，如[Pinterest](https://pinterest.com/)，也在生产中使用渐进式JPEG。

![Performance](https://images.guide/images/book-images/pinterest-loading-medium.png)

Pinterest的JPEG图像都是渐进式编码的。通过逐个扫描加载优化了用户体验。

<h3 id="the-disadvantages-of-progressive-jpegs"><a href="https://images.guide/#the-disadvantages-of-progressive-jpegs">5.3 渐进式JPEG的缺点</a></h3>

PJPEG的解码速度可能比基线JPEG慢 —— 有时需要3倍的时间。在拥有强大CPU的台式计算机上，这可能不是什么值得关注的问题，但在资源有限的移动设备上却不是如此。显示不完整的图层需要完成更多的工作，因为这相当于基本上是多次解码图像。 这些多次传递可能会占用CPU周期。  
<br>
渐进式JPEG也并不总是很小。对于非常小的图像(如缩略图)，渐进式JPEG可能比它们的基线对应的图像要大。然而，对于如此小的缩略图，渐进式渲染可能并没有提供太多的价值。  
<br>
这意味着，在决定是否发布PJPEG时，你需要对文件大小、网络延迟和CPU周期的使用进行实验，并找到合适的平衡点。  
<br>
注意：PJPEG(和所有JPEG)有时可以在移动设备上进行硬件解码。它不会改善RAM的影响，但可以消除一些CPU问题。并非所有Android设备都支持硬件加速，但高端设备支持，所有iOS设备都支持。  
<br>
一些用户可能认为渐进加载是一个缺点，因为很难判断图像何时已经完成加载。由于这可能会对每个用户有很大的影响，所以请评估对你自己的用户有意义的内容。

<h3 id="the-disadvantages-of-progressive-jpegs"><a href="https://images.guide/#the-disadvantages-of-progressive-jpegs">5.4 如何创建渐进式JPEG？</a></h3>

像[ImageMagick](https://www.imagemagick.org/)、[libjpeg](http://libjpeg.sourceforge.net/)、[jpegtran](http://jpegclub.org/jpegtran/)、[jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive)和[imagemin](https://github.com/imagemin/imagemin)这些工具和库都支持导出渐进式JPEG。如果你有一个现有的图像优化管道，添加渐进加载支持很可能是很直截了当的：

```JavaScript
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');

gulp.task('images', function () {
    return gulp.src('images/*.jpg')
        .pipe(imagemin({
            progressive: true
        }))
        .pipe(gulp.dest('dist'));       
});
```

大多数图像编辑工具默认情况下将图像保存为基线JPEG文件。

![Performance](https://images.guide/images/book-images/photoshop-medium.jpg)

大多数图像编辑工具默认将图像保存为基线JPEG文件。你可以在Photoshop中创建的任何图像保存为渐进式JPEG，方法是点击文件 -> 导出 -> 保存为Web（旧版），然后单击渐进式选项。Sketch也支持导出渐进式JPEG —— 导出为JPG，并在保存图像时选中“渐进式”复选框。

<h3 id="chroma-subsampling"><a href="https://images.guide/#chroma-subsampling">5.5 色度(或颜色)子采样</a></h3>

我们的眼睛对图像(色度)中颜色细节的丢失比亮度(简称亮度——亮度的度量)更宽容。[色度子采样](https://en.wikipedia.org/wiki/Chroma_subsampling)是一种压缩形式，它降低了有利于亮度的信号中颜色的精度。这可以减少文件大小(在某些情况下减少了[15%-17%](https://calendar.perfplanet.com/2015/why-arent-your-images-using-chroma-subsampling/))，而不会对图像质量产生负面影响，并且可以用于JPEG图像。子采样还可以减少图像的使用内存。

![Performance](https://images.guide/images/book-images/luma-signal-medium.jpg)

由于对比度负责形成我们在图像中看到的形状，因此定义它的亮度非常重要。 较旧或过滤的黑白照片可能不包含颜色，但由于亮度，它们可以像彩色照片一样详细。 色度（颜色）对视觉感知的影响较小。

![Performance](https://images.guide/images/book-images/no-subsampling-medium.jpg)

JPEG支持许多不同的子采样类型：none、horizontal、horizontal和vertical。这张图表来自Frédéric Kayser的**[马蹄蟹的JPEG](http://frdx.free.fr/JPEG_for_the_horseshoe_crabs.pdf)**。

在讨论子采样时，有许多常见的样本。通常，```4:4:4```,```4:2:2```和```4:2:0```。但是这些代表什么呢？假设一个子样本的格式是A:B:C。A是一行中的像素数，对于JPEG，通常是4。B表示第一行的颜色量，C表示第二行的颜色量。

+ ```4:4:4```没有压缩，所以颜色和亮度完全被传输。

+ ```4:2:2```水平半采样，垂直全采样。

+ ```4:2:0```从第一行的一半像素中采样颜色，忽略第二行。

> **注意:**jpegtran和cjpeg支持亮度和色度独立配置质量。可以添加```-sample```标志(例如```-sample 2x1```)。一些好的通用规则：子采样(```-sample 2x2```)对于照片来说非常好。无子采样(```-sample 1x1```)是最适合截图，横幅和按钮。最后一个折中方式(```2x1```)是你不确定用什么时候的选择。

通过减少色度组件中的像素，可以显著减少颜色组件的大小，最终减少字节大小。

![Performance](https://images.guide/images/book-images/subsampling-medium.jpg)

Chrome子采样配置的JPEG是质量为80。

对于大多数类型的图像，色度子采样是值得考虑的。 它确实有一些值得注意的例外：由于子采样依赖于我们眼睛的局限性，它并不适用于压缩那些颜色细节可能与亮度同样重要的图像(例如医学图像)。

包含字体的图像也会受到影响，因为文本的子采样不好会降低其可读性。锐利的边缘难以用JPEG压缩，因为JPEG的设计是为了更好地处理过渡柔和的照片场景。

![Performance](https://images.guide/images/book-images/Screen_Shot_2017-08-25_at_11.06.27_AM-medium.jpg)

[理解JPEG](http://compress-or-die.com/Understanding-JPG/)推荐坚持使用4:4:4(1×1)的子采样处理包含文本的图像。

了解更多：在JPEG规范中没有指定色度子采样的确切方法，因此不同的解码器处理它的方式不同。MozJPEG和libjpeg-turbo使用相同的缩放方法。旧版本的libjpeg使用了一种不同的方法，该方法以颜色添加边缘振荡效应。

> ** 注意：**Photoshop在使用“Save for web”功能时自动设置色度子采样。当图像质量设置在51-100之间时，完全不使用子采样(4:4:4)。当质量低于此值时，将使用4:2:0子采样。这就是为什么在将质量从51切换到50时可以观察到更大的文件大小缩减。  
<br>

> **注意: **在子采样讨论中，经常提到[YCbCr](https://en.wikipedia.org/wiki/YCbCr)这个术语。这个模型可以表示经过伽玛校正的[RGB](https://en.wikipedia.org/wiki/RGB_color_model)颜色空间。Y为伽玛校正亮度，Cb为蓝色的色度分量，Cr为红色的色度分量。如果查看ExifData，你将看到YCbCr接近于抽样级别。

有关色度子采样的进一步阅读，请参见[为什么你的图像不使用色度子采样？](https://calendar.perfplanet.com/2015/why-arent-your-images-using-chroma-subsampling/)。

<h3 id="how-far-have-we-come-from-the-jpeg"><a href="https://images.guide/#how-far-have-we-come-from-the-jpeg">5.6 我们离JPEG还有多远？</a></h3>

**以下是网络上图像格式的现状:**

*摘要 —— 我们经常需要有条件地为不同的浏览器提供不同的格式，以享受任何现代的东西的好处。*

![Performance](https://images.guide/images/book-images/format-comparison-medium.jpg)

不同的现代图像格式(和优化器)用于演示在目标文件大小为26KB时可以实现的功能。我们可以使用[SSIM](https://en.wikipedia.org/wiki/Structural_similarity)(结构相似性)或[Butteraugli](https://github.com/google/butteraugli)来比较质量，稍后我们将更详细地介绍这两种方法。

+ **[JPEG 2000](https://en.wikipedia.org/wiki/JPEG_2000) (2000)** —— 从基于离散余弦变换到基于小波方法的JPEG切换改进。**浏览器支持：Safari桌面版和iOS**

+ **[JPEG XR](https://en.wikipedia.org/wiki/JPEG_XR) (2009)** —— 替代JPEG和JPEG 2000支持[HDR](http://wikivisually.com/wiki/High_dynamic_range_imaging)和宽[色域](http://wikivisually.com/wiki/Gamut)空间。以稍慢的编码/解码速度生成比JPEG更小的文件。**浏览器支持: Edge, IE**。

+ **[WebP](https://en.wikipedia.org/wiki/WebP) (2010)** —— 基于块预测的格式，谷歌支持有损和无损压缩方式。通常用于提供与JPEG相关的字节节省和透明支持字节密集型PNG。缺乏色度子采样配置和渐进式加载。解码时间也比JPEG解码慢。**浏览器支持：Chrome, Opera。用Safari和Firefox做过实验**。

+ **[FLIF](https://en.wikipedia.org/wiki/Free_Lossless_Image_Format) (2015)** —— 无损图像格式声称优于PNG、无损WebP、无损BPG和无损JPEG 2000的压缩比。**没有浏览器支持。值得一提的是，有一个[浏览器中的JS解码器](https://github.com/UprootLabs/poly-flif)支持**。

+ **HEIF and BPG** 从压缩的角度来看，它们是相同的，但是有不同的包装:

+ **[BPG](https://en.wikipedia.org/wiki/Better_Portable_Graphics) (2015)** —— 旨在基于HEVC([高效视频编码](http://wikivisually.com/wiki/High_Efficiency_Video_Coding))，更高效地替代JPEG。与MozJPEG和WebP相比，它提供了更好的文件大小。由于许可证问题，不太可能获得广泛关注。**没有浏览器支持。值得一提的是，有一个[浏览器中JS解码器](https://bellard.org/bpg/)支持**。

+ **[HEIF](https://en.wikipedia.org/wiki/High_Efficiency_Image_File_Format) (2015)** —— 用于存储具有约束内部预测HEVC编码的图像和图像序列格式。苹果公司在[WWDC](https://www.cnet.com/news/apple-ios-boosts-heif-photos-over-jpeg-wwdc/)宣布，他们会在iOS上探索从JPEG切换到使用HEIF，引用文件大小可节省2倍。**浏览器支持：编写本文时不支持。Safari桌面版和iOS 11以及更新版本支持**。

如果你更在意视觉效果，你可能会喜欢上面[这些](http://xooyoozoo.github.io/yolo-octo-bugfixes/#cologne-cathedral&jpg=s&webp=s)可视化比较工具中的[一个](https://people.xiph.org/~xiphmont/demo/daala/update1-tool2b.shtml)。

因此，**浏览器支持是参差不齐的**，如果你希望享受上述任何一种支持的好处，你可能需要有条件地为每个目标浏览器提供反馈。在谷歌上，我们已经看到了WebP的一些承诺，所以我们将很快深入探讨它。

你还可以使用.jpg扩展名(或任何其他扩展名)提供图像格式(例如WebP、JPEG 2000)，因为这些都是浏览器可以渲染的图像，它可以决定媒体类型。这允许服务器端[内容类型协商](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)来决定发送哪个图像，而根本不需要更改HTML。像Instart Logic这样的服务在向客户交付图像时使用的就是这种方法。

接下来，让我们讨论**优化JPEG编码器**，这是在你不能有条件地提供不同的图像格式时的一种选择。

<h3 id="optimizing-jpeg-encoders"><a href="https://images.guide/#optimizing-jpeg-encoders">5.7 优化JPEG编码器</a></h3>

现代JPEG编码器试图生成更小、更高保真度的JPEG文件，同时保持与现有浏览器和图像处理应用程序的兼容性。它们避免了在生态系统中引入新的图像格式或者更改图像格式，以尽可能实现压缩增益的需要。MozJPEG和Guetzli就是这样的两个编码器。

*简要：你应该使用哪种优化的JPEG编码器？*

+ 一般的web资源: MozJPEG

+ 质量是你主要关心的问题，你不介意编码时间长: 使用Guetzli

+ 如果你需要更多选项可配置:

    - [JPEGRecompress](https://github.com/danielgtaylor/jpeg-archive)(它在底层使用的是MozJPEG)

    - [JPEGMini](http://www.jpegmini.com/) 它类似于Guetzli —— 自动选择最好的质量。它在技术上没有Guetzli那么复杂，但速度更快，目标是输出更适合web的质量范围的图像。

    - [ImageOptim API](https://imageoptim.com/api) ([这里](https://imageoptim.com/online)有免费的在线界面工具) —— 它在处理颜色方面是独一无二的。你可以选择颜色质量与整体质量分开。它会自动选择色度子采样级别，以在截图中保留高分辨率的颜色，但避免在自然照片的平滑颜色上浪费字节。

<h3 id="what-is-mozjpeg"><a href="https://images.guide/#what-is-mozjpeg">5.8 MozJPEG是什么？</a></h3>

Mozilla以[MozJPEG](https://github.com/mozilla/mozjpeg)的形式提供了一个现代化的JPEG编码器。它[声称](https://research.mozilla.org/2014/03/05/introducing-the-mozjpeg-project/)可以将JPEG文件减少10%的大小。使用MozJPEG压缩的文件可以跨浏览器工作，它的一些特性包括渐进式扫描优化、[网格量化](https://en.wikipedia.org/wiki/Trellis_quantization)(压缩丢弃最少的细节)和一些不错的[预置量化表](https://calendar.perfplanet.com/2014/mozjpeg-3-0/)，这些预置有助于创建更平滑的高分辨率图像(尽管如此，如果你愿意仔细研究XML配置，使用ImageMagick可以做到这一点)。

这两种[ImageOptim](https://github.com/ImageOptim/ImageOptim/issues/45)都支持MozJPEG，并且有一个相对可靠的[imagemin插件](https://github.com/imagemin/imagemin-mozjpeg)。下面是Gulp实现的一个示例:

```JavaScript
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminMozjpeg = require('imagemin-mozjpeg');

gulp.task('mozjpeg', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([imageminMozjpeg({
        quality: 85
    })]))
    .pipe(gulp.dest('dist'))
);
```

![Performance](https://images.guide/images/book-images/Modern-Image10-large.jpg)

![Performance](https://images.guide/images/book-images/Modern-Image11-large.jpg)

MozJPEG: 比较文件大小和不同质量视觉相似度的得分。

我使用[jpeg-archive](https://github.com/imagemin/imagemin-jpeg-recompress)项目中的[jpeg-compress](https://github.com/danielgtaylor/jpeg-archive)来计算源图像的SSIM(结构相似性)分数。SSIM是一种测量两幅图像之间相似度的方法，其中SSIM评分是一幅图像在另一幅图像被认为“完美”的情况下的质量度量。

根据我的经验，MozJPEG是一个很好的选择，可以在高视觉质量下压缩网络图像，同时减少文件大小。 对于中小尺寸的图像，我发现MozJPEG (quality=80-85)可以节省30%-40％的文件大小，同时保持可接受的SSIM，比jpeg-turbo提高了5%-6%。它确实比基线JPEG的[编码成本要低](http://www.libjpeg-turbo.org/About/Mozjpeg)，但是你可能不会发现这也是一个问题。

> **注意：**如果你需要一个支持MozJPEG的工具，该工具支持额外的配置和一些免费、实用的图像比较，请查看[jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive)。《Web性能实践》一书的作者Jeremy Wagner在使用[这种](https://twitter.com/malchata/status/884836650563579904)配置时取得了一些成功。

<h3 id="what-is-guetzli"><a href="https://images.guide/#what-is-guetzli">5.9 Guetzli是什么？</a></h3>

[Guetzli](https://github.com/google/guetzli)是来自谷歌一个很有前途的、虽然很慢、但是可以感知到的JPEG编码器，它试图找到最小的JPEG，这种JPEG在感知上与人眼无法区分。它执行一系列的实验，为最终的JPEG生成了一个建议，解释了每个建议的心理视觉错误。其中，它选择得分最高的提案作为最终输出。

为了测量图像之间的差异，Guetzli使用[Butteraugli](https://github.com/google/butteraugli)，一种基于人类感知的图像差异测量模型(下文讨论)。Guetzli可以考虑一些其他JPEG编码器所没有的视觉特性。例如，看到的绿光的数量和对蓝色的敏感度之间存在关系，所以在绿色附近的蓝色变化可以不那么精确地编码。

> **注意**：图像文件大小更依赖于**质量**的选择，而不是**编解码器**的选择。与通过切换编解码器可能节省的文件大小相比，最低质量的JPEG和最高质量的JPEG之间的文件大小差异要大得多。使用最低可接受质量是非常重要的。如果不太重要，请避免将质量设置得过高。

Guetzli[声称](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html)，与其他压缩器相比，Butteraugli给图像定的评分数据大小可以减少20%-30%。使用Guetzli需要注意的是，它非常非常慢，目前只适用于静态内容。从README可以看出，Guetzli需要大量的内存 —— 每百万像素需要1分钟和200MB内存。在[这个GitHub线程](https://github.com/google/guetzli/issues/50)中有一个关于Guetzli的真实体验的很好的线程。当你将优化图像作为静态站点构建过程的一部分时，它是理想的，但是在按需执行时，它就不那么理想了。

> **注意：**当你把图像优化作为静态网站的构建过程的一部分时，或者不需要按需执行图像优化时，Guetzli可能更适合。

像ImageOptim这样的工具支持Guetzli优化（在[最新版本](https://imageoptim.com/)中）。

```JavaScript
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminGuetzli = require('imagemin-guetzli');

gulp.task('guetzli', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([
        imageminGuetzli({
            quality: 85
        })
    ]))
    .pipe(gulp.dest('dist'))

);
```

![Performance](https://images.guide/images/book-images/Modern-Image12-large.jpg)

使用Guetzli对3x3MP图像进行编码几乎花费了7分钟(和很高的CPU使用率)，节省了大量的时间。对于存档高分辨率的照片，我可以看到它提供了一些价值。

![Performance](https://images.guide/images/book-images/Modern-Image13-large.jpg)

Guetzli：比较不同质量的文件大小和视觉相似度的得分。

> **注意：**建议在高质量的图像(例如输入未压缩的图像、PNG源和100%质量的JPEG)上运行Guetzli。虽然它可以用于其他图像(例如质量为84或更低的JPEG)，但效果可能较差。

虽然用Guetzli压缩图片非常(非常)耗时，而且会让你的粉丝晕头转向，但对于大图片来说，这是值得的。我看到过很多例子，在保持视觉保真度的情况下，它可以在任何地方节省40%的文件大小。这使得它非常适合存档照片。在中小型图像上，我仍然看到了一些节省(在10-15KB范围内)，但是它们没有那么明显。Guetzli可以在压缩较小的图像时引入更多的流体式失真。

你可能还对Eric Portis研究感兴趣，将Guetzli与Cloudinary的自动压缩进行[比较](https://cloudinary.com/blog/a_closer_look_at_guetzli)，以获得有效的不同数据点。

<h3 id="mozjpeg-vs-guetzli"><a href="https://images.guide/#mozjpeg-vs-guetzli">5.10 MozJPEG与Guetzli相比如何？</a></h3>

比较不同的JPEG编码器是复杂的 — 需要比较压缩图像的质量和保真度以及最终的文件大小。作为图像压缩专家Kornel Lesiński指出，一个而不是两个这些方面的基准测试可能会导致[无效](https://kornel.ski/faircomparison)的结论。

MozJPEG与Guetzli相比如何？ – Kornel的观点:

+ Guetzli为高质量的图像进行了调优(butteraugli被认为```q=90+```是最佳选择，MozJPEG的最佳点在```q=75```左右)

+ Guetzli的压缩速度要慢得多(两者都产生标准的JPEG，因此解码速度和往常一样快)

+ MozJPEG不会自动选择质量设置，但是你可以使用外部工具(例如[jpeg-archive](https://github.com/danielgtaylor/jpeg-archive))找到最优的质量

有许多方法可用于确定压缩图像在视觉上是否与源文件相似或在感知上是否相似。图像质量研究通常使用像[SSIM](https://en.wikipedia.org/wiki/Structural_similarity)(结构相似性)这样的方法。然而Guetzli对Butteraugli进行了优化。

<h3 id="butteraugli"><a href="https://images.guide/#butterauglii">5.11 Butteraugli</a></h3>

[Butteraugli](https://github.com/google/butteraugli)是谷歌的一个项目，它估计一个人可能注意到两幅图像的视觉退化(心理视觉相似性)的时间点。那些在几乎没有明显差异的区域内它给出了可靠的图像打分。Butteraugli不仅给出了一个标量分数，而且还计算了差异级别的空间地图。当查看SSIM关注图像的错误集合时，Butteraugli关注最糟糕的部分。

![Performance](https://images.guide/images/book-images/Modern-Image14-medium.jpg)

上面的例子使用Butteraugli找到了JPEG质量阈值的最小值，而此时用户还没有注意到一些不清楚的地方。它使总文件大小减少了65%。

在实践中，你将为视觉质量定义一个目标，然后运行许多不同的图像优化策略，查看Butteraugli评分，然后选择最适合文件大小和压缩级别的平衡点。

![Performance](https://images.guide/images/book-images/Modern-Image15-large.jpg)

总而言之,我花了大约30分钟安装后启动本地Butteraugli和在我的MAC上正确地编译C++源码构建。然后使用它相对简单: 指定两个图像比较(源和压缩版),它会给你一个分数。

**Butteraugli与其他视觉相似性的方法比较有何不同？**

来自Guetzli项目成员的[评论](https://github.com/google/guetzli/issues/10#issuecomment-276295265)表明，Guetzli在Butteraugli上得分最高，SSIM和MozJPEG上得分最低。这与我对自己的图像优化策略的研究是一致的。我在图像上运行Butteraugli和一个节点模块(如[img-ssim](https://www.npmjs.com/package/img-ssim))，将源代码与Guetzli和MozJPEG之前/之后的SSIM评分进行比较。

**结合编码器吗？**

对于较大的图像，我发现将Guetzli与MozJPEG中的**无损压缩**(jpegtran，而不是cjpeg，以避免丢弃Guetzli所做的工作)相结合，可以进一步减少10%-15%的文件大小(总体减少55%)，而SSIM只减少很小的一部分。我想提醒大家的是，这需要实验和分析，这一领域的其他人，如[Ariya Hidayat](https://ariya.io/2017/03/squeezing-jpeg-images-with-guetzli)，也曾尝试过，结果值得期待。

MozJPEG是一个对初学者友好的网络资源编码器，它的速度相对较快，可以生成高质量的图像。由于Guetzli是资源密集型的，最适合处理文件大小更大、质量更高的图像，所以我将它保留给中级到高级用户。
