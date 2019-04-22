# [图像优化](https://images.guide/)

[原文地址](https://images.guide/)

![Performance](https://images.guide/images/icons/logo.svg#alt=Performance)

作者：[Addy Osmani](https://twitter.com/addyosmani)

鸣谢: [Kornel Lesiński](https://twitter.com/kornelski), [Estelle Weyl](https://twitter.com/estellevw), [Jeremy Wagner](https://twitter.com/malchata), [Tim Kadlec](https://twitter.com/tkadlec), [Nolan O’Brien](https://twitter.com/NolanOBrien), [Pat Meenan](https://twitter.com/patmeenan), [Kristofer Baxter](https://twitter.com/kristoferbaxter), [Henri Helvetica](https://twitter.com/HenriHelvetica), [Houssein Djirdeh](https://twitter.com/hdjirdeh), [Una Kravets](https://twitter.com/una), [Elle Osmani](https://twitter.com/ARebelBelle) and [Ilya Grigorik](https://twitter.com/igrigorik).<br />
感谢他们的帮助和评论。

*翻译&校验：freedom*

## [摘要](https://images.guide/#the-tldr)

**我们都应该实现图像压缩的自动化。**

图像优化应该是自动化的。图像优化的最佳实践发生变化很容易被忽略，而且不经过构建管道的内容很容易丢失。为了实现自动化：在构建过程中使用[Imagemin](https://github.com/imagemin/imagemin)或[libvip](https://github.com/jcupitt/libvips)。这些有许多替代方案。

大多数CDN(如[Akamai](https://www.akamai.com/us/en/solutions/why-akamai/image-management.jsp))和第三方解决方案，如[Cloudinary](https://cloudinary.com/)、[imgix](https://imgix.com/)、[Fastly’s Image Optimizer](https://www.fastly.com/io/)、[Instart Logic’s SmartVision](https://www.instartlogic.com/technology/machine-learning/smartvision)或[ImageOptim API](https://imageoptim.com/api)都提供了全面的图像优化自动化解决方案。

在阅读博客文章和调整配置上花费的时间可能会超过服务月费(Cloudinary有一个[免费](http://cloudinary.com/pricing)套餐)。如果你不想关心这项工作外包的成本或延迟问题，那么上面的开源选项是可靠的。像[Imageflow](https://github.com/imazen/imageflow)或[Thumbor](https://github.com/thumbor/thumbor)这样的项目启用了自我托管的替代方案。

**每个人都应该高效地压缩自己的图像。**

至少：使用[ImageOptim](https://imageoptim.com/)。它可以显著地减小图像的大小，同时保持视觉质量。也可以使用Windows和Linux的[替代方案](https://imageoptim.com/versions.html)。

更具体地说：通过[MozJPEG](https://github.com/mozilla/mozjpeg)运行JPEG(对于Web内容，```Q=80```或更低都没问题)，并考虑支持[渐进式JPEG](http://cloudinary.com/blog/progressive_jpegs_and_green_martians)，PNG通过[pngquant](https://pngquant.org/)优化，SVG通过[SVGO](https://github.com/svg/svgo)优化。显式删除元数据(pngquant用```--strip```)以避免网络膨胀。代替超级巨大的GIF动画，提供[H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)视频(或为Chrome、Firefox和Opera提供[WebM](https://www.webmproject.org/))！如果你连[Giflossy](https://github.com/pornel/giflossy)都不能用。你又要节省额外的CPU周期，需要的图像质量高于web的平均质量，并且编码慢一点也能接受的话：试试[Guetzli](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html)。

一些浏览器通过接受请求头来宣布对图像格式的支持。这可以用于有条件地提供格式：例如，基于[Blink](https://zh.wikipedia.org/wiki/Blink)的浏览器(如Chrome)使用有损[WebP](https://developers.google.com/speed/webp/)，而对于其他浏览器则使用JPEG/PNG。

总是有很多你能做到的。有工具可以生成和服务```srcset```断点。在具有[客户端提示](https://developers.google.com/web/updates/2015/09/automating-resource-selection-with-client-hints)的基于Blink的浏览器中，资源选择可以自动化，并且如果用户选择在浏览器中“保存数据”，那么可以通过遵从[保存数据](https://developers.google.com/web/updates/2016/02/save-data)提示来减少字节数。

你可以使你的图像文件大小越小，你就可以为用户提供更好的网络体验 —— 尤其是在移动平台上。在这篇文章中，我们将探讨通过现代压缩技术减少图像大小的方法，并将对图像质量的影响降到最低。

## [1.介绍](https://images.guide/#introduction)

**图像仍然是造成网络膨胀的首要原因。**

图像占用了大量的互联网带宽，因为它们通常有较大的文件大小。根据[HTTP存档](http://httparchive.org/)，60%用于获取网页的数据是由JPEG、PNG和GIF组成的图像。截至2017年7月，在平均3.0MB大小的站点加载的网页内容中图像占了[1.7MB](http://httparchive.org/interesting.php#bytesperpage)。

根据Tammy Everts，将图像添加到页面或使用更多的现有图像已经被[证明](https://calendar.perfplanet.com/2014/images-are-king-an-image-optimization-checklist-for-everyone-in-your-organization/)可以提高转化率。图像不太可能消失，因此投资于有效的压缩策略以尽量减少网络膨胀变得非常重要。

![Performance](https://images.guide/images/book-images/Modern-Image00-medium.jpg)

根据2016年[Soasta/Google的研究](https://www.thinkwithgoogle.com/marketing-resources/experience-design/mobile-page-speed-load-time/)，图像是第二大转换预测指标，最佳页面的图像数量占比少于38%。

图像优化包含各种不同的方法，它们都可以减少图像的文件大小。这最终取决于你对图像所要求的视觉保真度。

![Performance](https://images.guide/images/book-images/image-optimisation-medium.jpeg)

**图像优化**：选择正确的格式，谨慎压缩，并将关键图像优先于那些可能被延迟加载的图像。

常见的图像优化包括压缩，使用[``<picture>``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture)/[``<img srcset>``](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)根据屏幕大小响应式为它们服务，以及调整它们的大小以降低图像解码成本。

![Performance](https://images.guide/images/book-images/chart_naedwl-medium.jpg)

根据[HTTP存档](http://jsfiddle.net/rviscomi/rzneberp/embedded/result/)，每个图像在第95%(查看累积分布函数)的位置可以节省30KB流量！

因此有足够的空间让我们一起把图像优化做得更好。

![Performance](https://images.guide/images/book-images/image-optim-medium.jpg)

ImageOptim是免费的，通过现代压缩技术和剥离不必要的EXIF元数据来减少图像大小。

如果你是一个设计师，也有一个[Sketch的ImageOptim插件](https://github.com/ImageOptim/Sketch-plugin)，可以优化你导出的资源。我发现它可以节省很多时间。

## [2.如何判断我的图像是否需要优化?](https://images.guide/#do-my-images-need-optimization)

通过[WebPageTest.org](https://www.webpagetest.org/)执行站点审核，它将有去更好地优化图像(参见“压缩图像”)更明显的机会。

![Performance](https://images.guide/images/book-images/Modern-Image1-medium.jpg)

网页测试报告中的“压缩图像”部分列出了可以更有效地压缩的图像以及这样做估计可以节省的文件大小。

![Performance](https://images.guide/images/book-images/Modern-Image2-large.jpg)

[Lighthouse](https://developers.google.com/web/tools/lighthouse/)是审核性能的最佳实践工具。包括对图像优化的审核，并可以为可能被进一步压缩的图像提供建议，或者指出屏幕外可延迟加载的图像。

从Chrome 60开始，Lighthouse现在是Chrome DevTools里目前最权威的[审核面板](https://developers.google.com/web/updates/2017/05/devtools-release-notes#lighthouse)：

![Performance](https://images.guide/images/book-images/hbo-medium.jpg)

Lighthouse是可以审核Web性能的最佳实践和改进Web应用功能的工具。

你也可能熟悉其他性能审核工具，如[PageSpeed](https://developers.google.com/speed/pagespeed/insights/)、[Insight](https://developers.google.com/speed/pagespeed/insights/)或者通过Cloudinary的[Website Speed Test](https://webspeedtest.cloudinary.com/)，其中包括详细的图像分析审核。

## [3.如何选择图像格式?](https://images.guide/#choosing-an-image-format)

正如Ilya Grigorik在其出色的[图像优化指南](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization)中所指出的，图像的“正确格式”是视觉所期望的结果和功能需求的结合。你是在处理光栅图形还是矢量图形呢？

![](https://images.guide/images/book-images/rastervvector-large.png)

[光栅图形](https://en.wikipedia.org/wiki/Raster_graphics)通过对矩形像素网格内的每个像素的值进行编码来表示图像。 它们不是分辨率或者独立的缩放。 WebP和广泛支持的格式（如JPEG或PNG）可以很好地处理这些图像，其中照片写实是必需的。 我们讨论过的Guetzli，MozJPEG以及一些其他方法都很适用于光栅图形。

[矢量图形](https://en.wikipedia.org/wiki/Vector_graphics)是使用点、线和多边形来表示图像，是使用简单的几何形状（例如logo）提供高分辨率和像SVG一样更好地处理缩放情况的图像格式。

错误的格式会使你付出代价。选择正确图像格式逻辑流程又可能充满风险，因此尝试其他格式可以在一定程度上节省费用。

Jeremy Wagner在他的图像优化演讲中谈到了在评估格式时值得考虑的[权衡点](http://jlwagner.net/talks/these-images/#/2/2)。

## [4.普通的JPEG](https://images.guide/#the-humble-jpeg)

[JPEG](https://en.wikipedia.org/wiki/JPEG)很可能是世界上使用最广泛的图像格式。如前所述，在[HTTP存档](https://httparchive.org/)爬虫的站点上看到的图像中，有[45%是JPEG](https://httparchive.org/reports/state-of-the-web?start=latest)。你的手机，你的数码单反相机，旧的网络摄像头 —— 一切都支持这个编解码器。它也很古老，可以追溯到1992年第一次发布。在这段时间里，有大量的研究试图改进它所提供的东西。

JPEG使用的是一种为了节省空间而丢弃信息的有损压缩算法，并试图在尽量保持文件大小的同时保持视觉保真度的图像格式。

**实际情况允许你可以接受什么样的图像质量？**

JPEG等格式最适合具有多个颜色区域的照片或图像。 大多数优化工具将允许你设置你满意的压缩级别; 较高的压缩会减小更多文件大小，但会引入伪像，晕圈或块状降级。

![Performance](https://images.guide/images/book-images/Modern-Image5-large.jpg)

JPEG：当我们从最佳质量转换到最低质量时，可感知的JPEG压缩伪像会增加。 请注意，一个工具中的图像质量分数与另一个工具中的质量分数又很大的不同。

在选择要设置的质量选项时，请考虑你的图像属于哪个质量范畴：

+ **最好的质量** - 当质量比带宽更重要的时候。这可能是因为图像在你的设计中具有很高的重要性，或者是要以完全分辨率显示。

+ **较好的质量** - 当你想要更小的文件大小时，但又不想对图像质量产生太大的负面影响。用户仍然关心某种程度的图像质量。

+ **较低低质量** - 当你足够关心带宽，图像退化也是可以接受的。这些图像适用于杂乱无章的网络环境。

+ **最低的质量** - 节省带宽是最重要的。用户希望有一个不错的体验，但为了更快地加载页面，用户将接受图像一定程度的降级体验。

接下来，让我们谈谈JPEG的压缩模式，因为这些模式会对感知的性能产生很大的影响。

> **注意：**我们有时可能高估了用户所需的图像质量。图像质量可以被认为是偏离理想的，非压缩源。这也可以是主观的。

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

虽然用Guetzli压缩图像非常(非常)耗时，而且会让你的粉丝晕头转向，但对于大图像来说，这是值得的。我看到过很多例子，在保持视觉保真度的情况下，它可以在任何地方节省40%的文件大小。这使得它非常适合存档照片。在中小型图像上，我仍然看到了一些节省(在10-15KB范围内)，但是它们没有那么明显。Guetzli可以在压缩较小的图像时引入更多的流体式失真。

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

## [6.WebP是什么?](https://images.guide/#what-is-webp)

[WebP](https://developers.google.com/speed/webp/)是来自谷歌的一种最新的图像格式，旨在提供更小的文件大小，在可接受的视觉质量下进行无损和有损压缩。它包括对alpha通道透明度和动画的支持。

在过去的一年中，WebP在有损和无损模式下的压缩性能比传统的压缩性能提高了几个百分点，并且在速度方面，算法的速度提高了两倍，减压效率提高了10％。WebP不是一个面向所有用户的工具，但是它在图像压缩社区中有一定的地位和不断增长的用户基础。让我们看看这是为什么。

![Performance](https://images.guide/images/book-images/Modern-Image16-large.jpg)

WebP: 比较文件大小和不同质量下的视觉相似度的得分。

<h3 id="how-does-webp-perform"><a href="https://images.guide/#how-does-webp-perform">6.1 WebP的性能如何?</a></h3>

**有损压缩**

使用VP8或VP9视频关键帧编码变体的WebP有损文件，WebP团队引用的图像大小平均比JPEG文件小25%-34%。

在低质量范围(0-50)，WebP比JPEG有很大的优势，因为它可以模糊掉难看的块状伪像。中等质量的设置(-m4 -q75)是默认的平衡速度和文件大小。在较高的范围(80-99)，WebP的优势缩小。建议在速度比质量更重要的时候使用WebP。

**无损压缩**

[WebP无损文件比PNG文件小26%](https://developers.google.com/speed/webp/docs/webp_lossless_alpha_study)。与PNG相比，无损加载时间减少了3%。也就是说，你通常不希望在web上为用户提供无损服务。无损边缘和锐边(如non-JPEG)是有区别的。无损的WebP可能更适合存档内容。

**透明度**

WebP有一个无损的8位透明通道，仅比PNG多22%的字节。它还支持有损RGB透明度，这是WebP独有的特性。

**元数据**

WebP文件格式支持EXIF照片元数据和XMP数字文档元数据。它还包含ICC颜色配置文件。

WebP提供了更好的压缩，但代价是CPU更密集。早在2013年，WebP的压缩速度比JPEG慢了大约10倍，只是现在可以忽略不计(一些像的压缩速度可能是慢了2倍)。进行处理的静态图像作为构建的一部分，这应该不是一个大问题。动态生成的映像可能会造成额外可感知的CPU开销，你需要对其进行评估。

> **注意：**WebP有损质量设置不能与JPEG直接比较。“70%质量”的JPEG与“70%质量”的WebP图像大不相同，因为WebP通过丢弃更多的数据来实现更小的文件大小。

<h3 id="whos-using-webp-in-production"><a href="https://images.guide/#whos-using-webp-in-production">6.2 谁在生产中使用WebP?</a></h3>

许多大公司在生产中使用WebP来降低成本和减少网页加载时间。

谷歌报告说，使用WebP比其他有损压缩方案节省了30%-35%文件大小，每天处理430亿张图像请求，其中26%是无损压缩。这有大量的请求可以节省大量的成本。如果[浏览器支持](http://caniuse.com/#search=webp)更好、更广泛，节省的成本无疑会更大。谷歌的Google Play和YouTube等网站也在生产环境上使用。

Netflix、Amazon、Quora、Yahoo、Walmart、Ebay、The Guardian、Fortune和USA Today都为支持WebP的浏览器使用WebP压缩和提供图像。VoxMedia为Chrome用户切换到WebP，将Verge的[加载时间缩短了1-3秒](https://product.voxmedia.com/2015/8/13/9143805/performance-update-2-electric-boogaloo)。在切换到为Chrome用户提供服务时，[500px](https://iso.500px.com/500px-color-profiles-file-formats-and-you/)的图像文件大小平均减少了25%，图像质量接近甚至更好。

这个样本列表中指出的公司里还有不少其他公司支持WebP。

![Performance](https://images.guide/images/book-images/webp-conversion-large.jpg)

Google使用WebP：每天在YouTube，Google Play，Chrome数据保护程序和G+上的WebP图像请求每天达到430亿次。

<h3 id="how-does-webp-encoding-work"><a href="https://images.guide/#how-does-webp-encoding-work">6.3 WebP编码是如何工作的?</a></h3>

WebP的有损编码是针对与JPEG静止图像相比的。WebP有损编码有三个关键阶段:

**宏块** —— 把图像分解为16×16(宏观)亮度像素的小块和两个8×8色像素的小块。这个想法听起来很像JPEG进行颜色空间转换、色度通道下采样和图像细分。

![Performance](https://images.guide/images/book-images/Modern-Image18-medium.png)

**预测** ——宏块的每个4×4子块具有应用的预测模型，其有效地进行滤波。它定义了一个块周围的两组像素 —— A（它正上方的行）和L（它左边的列）。使用这两个编码器填满4×4像素测试块，并确定哪个创建最接近原始块的值。Colt McAnlis在[WebP有损模式的工作原理](https://medium.com/@duhroach/how-webp-works-lossly-mode-33bd2b1d0670)中更深入地讨论了这一点。

![Performance](https://images.guide/images/book-images/Modern-Image19-small.png)

步骤类似于JPEG编码，应用离散余弦变换（DCT）。 和JPEG的霍夫曼相比关键的区别在于使用的[算法压缩器](https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H)。

如果你想进一步加深了解，Google Developer上的文章[WebP Compression Techniques](https://developers.google.com/speed/webp/docs/compression)将深入探讨这一主题。

<h3 id="webp-browser-support"><a href="https://images.guide/#webp-browser-support">6.4 WebP浏览器支持</a></h3>

并不是所有的浏览器都支持WebP，但是[根据CanIUse.com的数据](http://caniuse.com/webp)，全球用户支持率约为74%。Chrome和Opera都支持它。Safari、Edge和Firefox都曾试用过支持，但尚未正式发布。他们通常将获取WebP图像的任务不是留给用户，而留给web开发人员。稍后将对此进行详细介绍。

以下是主要的浏览器和支持信息:

+ Chrome：Chrome在23版本开始全面支持。

+ Android版Chrome：从Chrome 50开始支持

+ Android：自Android 4.2开始支持

+ Opera：从12.1开始支持

+ Opera Mini：所有版本都支持

+ Firefox：一些测试版支持

+ Edge：一些测试版支持

+ Internet Explorer：不支持

+ Safari：一些测试版支持

WebP并非没有缺点。它缺乏全分辨率的色彩空间选项，不支持渐进式解码。也就是说，WebP的工具虽然是不错的，并且浏览器支持良好，尽管在编写本文时仅限于Chrome和Opera，但它可能已经覆盖了足够多的用户，因此考虑使用一个应变的工具也是有意义的。

<h3 id="how-do-i-convert-to-webp"><a href="https://images.guide/#how-do-i-convert-to-webp">6.5 如何把图像转成WebP?</a></h3>

一些商业和开源的图像编辑和处理包都支持WebP。XnConvert是一个特别有用的应用程序: 它是一个免费的、跨平台的批处理图像转换器。

> **注意：**避免将低质量或平均质量的JPEG转换为WebP非常重要。 使用JPEG压缩过的图像生成WebP图像是一个常见的错误。 这可能导致WebP效率降低，因为它必须保存图像和JPEG添加的伪像，导致你在质量上损失两次。 原始文件是最好的提供给转换应用程序可用的最佳质量的源文件。

**[XnConvert](http://www.xnview.com/en/xnconvert/)**

XnConvert支持批处理图像，兼容500多种图像格式。你可以组合80多个单独的操作以多种方式转换和编辑图像。

![Performance](https://images.guide/images/book-images/Modern-Image20-large.png)

XnConvert支持批处理图像优化，允许直接从源文件转换为WebP和其他格式。除了压缩，XnConvert还可以帮助图像进行元数据的剥离、裁剪、颜色深度定制和其他转换。xnview网站上列出的一些选项包括:

+ 元数据: 编辑

+ 变换: 旋转，裁剪，调整大小

+ 调整: 亮度，对比度，饱和度

+ 滤镜: 模糊，浮雕，锐化

+ 效果: 掩蔽，水印，渐晕

操作的结果可以导出到大约70种不同的文件格式，包括WebP。XnConvert在Linux、Mac和Windows上是免费的。强烈推荐使用XnConvert，特别是对于小型企业。

**Node模块**

[Imagemin](https://github.com/imagemin/imagemin)是一个流行的图像压缩模块，它还有一个将图像转换为WebP的附加模块([Imagemin-WebP](https://github.com/imagemin/imagemin-webp))。它同时支持有损和无损两种模式。

安装imagemin和imagemin-webp运行:

```Bash
> npm install --save imagemin imagemin-webp
```

接着，我们可以把两个模块通过require()引进来，并在项目中的任何图像(例如JPEG)目录上运行它们。下面示例我们使用的是有损编码与WebP编码器质量为60:

```JavaScript
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg}'], 'images', {
    use: [
        imageminWebp({quality: 60})
    ]
}).then(() => {
    console.log('Images optimized');
});
```

与JPEG类似，可以在输出中注意压缩伪像。 评估什么样的质量设置对你自己的图像有意义。 Imagemin-webp还可用于通过``` lossless: true```选项来编码输出无损质量的WebP图像（支持24位颜色和完全透明）：

```JavaScript
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg,png}'], 'build/images', {
    use: [
        imageminWebp({lossless: true})
    ]
}).then(() => {
    console.log(‘Images optimized’);
});
```

关于imagemin-webp的构建可以使用Sindre Sorhus提供的[Gulp的WebP插件](https://github.com/sindresorhus/gulp-webp)或[WebPack的WebP加载器](https://www.npmjs.com/package/webp-loader)。Gulp插件可以执行imagemin插件的所有配置选项，如:

```JavaScript
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        quality: 80,
        preset: 'photo',
        method: 6
    }))
    .pipe(gulp.dest('dist'))
);
```

或无损模式:

```JavaScript
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp-lossless', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        lossless: true
    }))
    .pipe(gulp.dest('dist'))
);
```

**使用Bash批处理图像优化**

XNConvert支持批处理图像压缩，但是如果你希望避免使用应用程序或构建系统，bash和图像优化二进制文件可以使事情变得简单很多。

你可以使用[cwebp](https://developers.google.com/speed/webp/docs/cwebp)批量把你的图像转换成WebP:

```Bash
find ./ -type f -name '*.jpg' -exec cwebp -q 70 {} -o {}.webp \;
```

或者可以使用MozJPEG的 [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive) 批量优化你的源图像：

```Bash
find ./ -type f -name '*.jpg' -exec jpeg-recompress {} {} \;
```

并使用[svgo](https://github.com/svg/svgo)(我们将在后面介绍)来压缩那些SVG文件:

```Bash
find ./ -type f -name '*.svg' -exec svgo {} \;
```

Jeremy Wagner有一篇关于[使用Bash进行图像优化](https://jeremywagner.me/blog/bulk-image-optimization-in-bash)更全面的文章，还有一篇关于[使用Bash并行成图像优化](https://jeremywagner.me/blog/faster-bulk-image-optimization-in-bash)完这项工作的文章，值得一读。

**其他处理和编辑WebP图像的应用包括：**

+ Leptonica — 一个完整的开源的处理和分析图像的应用程序网站。

+ Sketch支持直接输出WebP

    - GIMP — 是Photoshop免费而且开源的图像编辑器替代品。

    - ImageMagick — 免费创建、组合、转换和编辑位图图像的命令行应用程序。

    - Pixelmator — Mac的商业图像编辑器。

    - Photoshop WebP插件 — 图像从谷歌导入和导出的免费插件。
    
**Android：**你可以使用Android Studio将现有的BMP、JPG、PNG或静态GIF图像转换为WebP格式。有关更多信息，请参见[使用Android Studio创建WebP图像](https://developer.android.com/studio/write/convert-webp.html)。

<h3 id="how-do-i-view-webp-on-my-os"><a href="https://images.guide/#how-do-i-view-webp-on-my-os">6.6 如何查看我的操作系统上的WebP图像？</a></h3>

你可以拖拽WebP图像到基于flash的浏览器(Chrome, Opera, Brave)来预览它们，你也可以使用Mac或Windows的附加组件直接在你的操作系统中预览它们。

几年前，[Facebook尝试使用WebP](https://www.cnet.com/news/facebook-tries-googles-webp-image-format-users-squawk/)，发现那些试图右键单击照片并将其保存到磁盘的用户注意到，由于WebP格式的照片，它们不会显示在浏览器之外。这里出现了三个关键问题:

+ “另存为” 无法在本地查看WebP文件。因为这是固定的Chrome注册自己为“.webp”处理程序。

+ “另存为” 然后将图像附加到电子邮件中与没有Chrome的人分享。Facebook解决了这个问题，它在用户界面中引入了一个明显的“下载”按钮，并在用户请求下载时返回一个JPEG。

+ 右击 -> 复制URL -> 在web上共享URL。这是通过[内容类型协商](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)解决的。

这些问题对你的用户来说可能很少出现，但顺便值得提一下的是，这是一个关于社交可共享性的有趣说明。庆幸的是，现在有一些实用程序可以在不同的操作系统上查看和使用WebP。

在Mac上，试试[WebP的Quick Look插件](https://github.com/Nyx0uf/qlImageSize)(qlImageSize)。它起得很好的作用:

![Performance](https://images.guide/images/book-images/Modern-Image22-large.jpg)

在Windows上，你还可以下载[WebP编解码包](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/WebpCodecSetup.exe)，允许在文件管理器和Windows照片查看器中预览WebP图像。

<h3 id="how-do-i-serve-webp"><a href="https://images.guide/#how-do-i-serve-webp">6.7 如何提供WebP？</a></h3>

不支持WebP的浏览器最终可能根本不显示图像，这是不理想的。为了避免这种情况，我们可以使用一些策略来基于浏览器支持选择性提供WebP。

![Performance](https://images.guide/images/book-images/play-format-webp-large.jpg)

Chrome DevTools网络面板的“Type”列下，选择性为基于Blink的浏览器提供WebP文件的高亮显示。

![Performance](https://images.guide/images/book-images/play-format-type-large.jpg)

当Play store向Blink提供WebP时，它又回到了Firefox等浏览器的JPEG模式。

以下是从服务器向用户提供WebP图像的一些可选配置：

**使用.htaccess提供WebP副本**

下面是当服务器上存在匹配的.webp版本的JPEG/PNG文件时，如何使用.htaccess文件向支持的浏览器提供WebP文件。

Vincent Orback推荐这种方法:

浏览器可以通过[Accept头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)[显式地发出WebP支持的信号](http://vincentorback.se/blog/using-webp-images-with-htaccess/)。如果你控制后端，则可以返回存在于磁盘上的图像的WebP版本，而不是JPEG或PNG等格式。但是这并不总是可能的(例如，对于静态主机，如GitHub pages或S3)，所以在考虑这个选项之前一定要检查一下服务器是否支持。

下面是Apache web服务器的.htaccess文件示例:

```XML
<IfModule mod_rewrite.c>

  RewriteEngine On

  # Check if browser support WebP images
  RewriteCond %{HTTP_ACCEPT} image/webp

  # Check if WebP replacement image exists
  RewriteCond %{DOCUMENT_ROOT}/$1.webp -f

  # Serve WebP image instead
  RewriteRule (.+)\.(jpe?g|png)$ $1.webp [T=image/webp,E=accept:1]

</IfModule>

<IfModule mod_headers.c>

    Header append Vary Accept env=REDIRECT_accept

</IfModule>
```

```Bash
AddType  image/webp .webp
```

如果页面上出现的.webp图像有问题，请确保在服务器上启用了image/webp MIME类型。

Apache: 将以下代码添加到.htaccess文件中：

```Bash
AddType image/webp .webp
```

Nginx: 将以下代码添加到mime.types文件中：

```Bash
image/webp webp;
```

> **注意：** Vincent Orback有一个用于服务WebP的[htaccess配置](https://github.com/vincentorback/WebP-images-with-htaccess)示例供参考，Ilya Grigorik维护了一组[用于服务WebP的配置脚本](https://github.com/igrigorik/webp-detect)，这些脚本是很有用的。

**使用 ```<picture>``` 标签**

浏览器本身可以通过使用```<picture>```标记来选择要显示的图像格式。```<picture>```标签使用多个```<source>```元素，使用一个```<img>```标签是包含图像的实际DOM元素。浏览器循环遍历源并检索第一个匹配项。如果用户浏览器中不支持```<picture>```标记，则呈现```<div>```并使用```<img>```标记。

> **注意：**因为顺序的关系需要注意```<source>```的位置。不要将iamge/webp源放在遗留格式之后，而是将它们放在前面。理解它的浏览器将使用它们，不理解它的浏览器将转移到更广泛支持的框架上。如果图像的物理大小相同(不使用media属性时)，也可以将它们按文件大小排序。通常，这与放在最后的文件顺序相同。

下面是一些HTML示例:

```HTML
<picture>
  <source srcset="/path/to/image.webp" type="image/webp">
  <img src="/path/to/image.jpg" alt="">
</picture>

<picture>   
    <source srcset='paul_irish.jxr' type='image/vnd.ms-photo'>  
    <source srcset='paul_irish.jp2' type='image/jp2'>
    <source srcset='paul_irish.webp' type='image/webp'>
    <img src='paul_irish.jpg' alt='paul'>
</picture>

<picture>
   <source srcset="photo.jxr" type="image/vnd.ms-photo">
   <source srcset="photo.jp2" type="image/jp2">
   <source srcset="photo.webp" type="image/webp">
   <img src="photo.jpg" alt="My beautiful face">
</picture>
```

**CDN自动转换为WebP**

一些CDN支持图像自动转换成WebP格式，并可以使用客户端提示，[随时](http://cloudinary.com/documentation/responsive_images#automating_responsive_images_with_client_hints)提供WebP图像。检查你的CDN，看看他们的服务中是否包含WebP支持。你可能有一个简单的解决方案在等着你。

**WordPress WebP支持**

**Jetpack** —— Jetpack是一个非常流行的WordPress插件，包括一个名为[Photon](https://jetpack.com/support/photon/)的CDN图像服务。有了Photon，你能无缝支持WebP图像。Photon CDN包含在Jetpack的免费级别中，因此这是一个很好的价值和不干涉实现的图像服务。缺点是Photon会调整图像大小，在URL中放入查询字符串，并且每个图像都需要额外的DNS查找。

**缓存启用器和优化器** —— 如果你使用WordPress，至少有一个半开源的选项。开源插件[缓存启用](https://wordpress.org/plugins/cache-enabler/)程序有一个菜单复选框选项，如果当前用户的浏览器支持WebP，就用于缓存可用的WebP图像。这使得提供WebP图像变得容易。但是有一个缺点: 缓存启用程序需要使用名为优化器的姐妹程序，这个程序需要按年付费。这个特性似乎不符合一个真正的开源解决方案。

**短像素** —— 另一个与缓存启用器一起使用的选项，也是需要付费的，是短像素。短像素函数很像上面描述的优化器。但是你可以每月免费优化多达100张图像。

**为什么```<video>```比GIF压缩动画更好**

GIF动画的格式非常有限，但是它仍然得到了广泛的使用。尽管从社交网络到流行媒体网站，所有东西都大量嵌入了GIF动画，但这种格式从未设计用于视频存储和动画。事实上，[GIF89a规范](https://www.w3.org/Graphics/GIF/spec-gif89a.txt)指出“GIF并不是一个动画平台”。[颜色的数量，帧数和尺寸](http://gifbrewery.tumblr.com/post/39564982268/can-you-recommend-a-good-length-of-clip-to-keep-gifs)都会影响GIF动画的大小。GIF动画转换成视频可以提供了节省最大的成本。

![Performance](https://images.guide/images/book-images/animated-gif-large.jpg)

GIF动画与视频: 不同格式的文件大小在大约相等质量下的比较。

**MP4视频与GIF动画相比传输相同的文件通常可以减少80%或更多的文件大小。**GIF不仅常常浪费大量带宽，而且加载时间更长，包含的颜色更少，而且通常只提供了部分的用户体验。你可能已经注意到上传到Twitter的动态GIF，在Twitter上的表现比在其他网站上要好。[Twitter上的GIF动画实际上并不是GIF](http://mashable.com/2014/06/20/twitter-gifs-mp4/#fiiFE85eQZqW)。为了改善用户体验和减少带宽消耗，上传到Twitter的GIF动画实际上被转换成了视频。类似地，在上传时[Imgur将GIF图像转换成视频](https://thenextweb.com/insider/2014/10/09/imgur-begins-converting-gif-uploads-mp4-videos-new-gifv-format/)，并悄无声息地将其转换成MP4格式。

为什么GIF比视频要大很多倍？GIF动画存储每个帧作为无损的GIF图像 —— 是的，无损！我们经常经历的质量下降是由于GIF被限制为256种颜色。与H.264等视频编解码器不同，这种格式通常比较大，因为它不考虑用于压缩的相邻帧。MP4视频将每个关键帧存储为有损的JPEG, JPEG会丢弃一些原始数据以实现更好的压缩。

**如果你可以切换到视频**

+ 使用[ffmpeg](https://www.ffmpeg.org/)将GIF动画(或源文件)转换为H.264的MP4文件。我使用来自[Rigor](http://rigor.com/blog/2015/12/optimizing-animated-gifs-with-html5-video)的一行代码:```ffmpeg -i animated.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)2:trunc(ih/2)2" video.mp4```

+ ImageOptim API还支持[将GIF动画转换为WebM视频和H.264视频](https://imageoptim.com/api/ungif)，[消除GIF的抖动](https://github.com/pornel/undither#examples)，这可以帮助视频编解码器压缩更多的文件大小。

**如果你必须使用动GIF画**

+ 像Gifsicle这样的工具可以剥离元数据和未使用的调色板条目，并使帧之间的变化最小化

+ 考虑有损GIF编码器。 Gifsicle的分支[Gifsossy](https://github.com/pornel/giflossy)支持带有```-lossy```标签，它可以减少约60%-65%的文件大小。 还有一个很好的基于Gifsossy的工具，叫做[Gifify](https://github.com/vvo/gifify)。 对于非动画GIF，把它们转换为PNG或WebP。

更多的相关信息，请查看Rigor的《[Book of GIF](https://rigor.com/wp-content/uploads/2017/03/TheBookofGIFPDF.pdf)》。

## [7.SVG优化](https://images.guide/#svg-optimization)

保持SVG精简意味着去掉任何不必要的东西。使用编辑器创建的SVG文件通常包含大量冗余信息(元数据、注释、隐藏层等)。这些内容通常可以安全地删除或转换为更小的形式，而不会影响SVG的最终呈现。

![Performance](https://images.guide/images/book-images/Modern-Image26-large.jpg)

由Jake Archibald创建的[SVGOMG](https://jakearchibald.github.io/svgomg/)是一个GUI界面工具，通过选择优化，通过输出标记的实时预览，你可以根据自己的喜好优化SVG。

**一般的SVG优化(SVGO)规则：**

+ 缩小和压缩SVG文件。SVG实际上只是用XML表示的文本资源，如CSS、HTML和JavaScript，应该进行压缩和gzip，以提高性能。

+ 使用预定义的SVG形状代替路径，如```<rect>```、```<circle>```、```<ellipse>```、```<line>```和```<polygon>```。选择预定义的形状可以减少生成最终图像所需的标记，这意味着浏览器解析和光栅化的代码更少。降低SVG的复杂性意味着浏览器可以更快地显示它。

+ 如果你必须使用路径，尽量减少你的曲线和路径。尽可能地简化和组合它们。llustrator的[简化工具](http://jlwagner.net/talks/these-images/#/2/10)擅长删除复杂艺术品中的多余点，同时消除不规则性。

+ 避免使用组。如果不能，试着简化它们。

+ 删除不可见的层。

+ 避免任何Photoshop或Illustrator效果。它们会把矢量图像转换成大光栅图像。

+ 仔细检查任何不支持SVG的嵌入式光栅图像

+ 使用工具优化你的SVG。[SVGOMG](https://jakearchibald.github.io/svgomg/)是一个非常方便的基于web的GUI，由Jake Archibald为[SVGO](https://github.com/svg/svgo)开发，我发现它非常有价值。如果使用Sketch，可以在导出文件时使用[运行SVGO的Sketch插件](https://www.sketchapp.com/extensions/plugins/svgo-compressor/)来缩小文件大小。

![Performance](https://images.guide/images/book-images/svgo-precision-large.jpg)

通过SVGO以高精度模式运行SVG源（可以让文件大小节省29%）与低精度模式（可以让文件大小节省38%）的示例。

[SVGO](https://www.sketchapp.com/extensions/plugins/svgo-compressor/)是一种基于节点的SVG优化工具。SVGO可以通过降低定义中数字的精度来减少文件大小。一个点之后的每一位数字都会增加一个字节，这就是为什么改变精度(位数)会严重影响文件大小的原因。但是，要非常非常小心地更改精度，因为它会在视觉上影响形状的外观。

![Performance](https://images.guide/images/book-images/Modern-Image28-large.jpg)

需要注意的是，虽然SVGO在前面的示例中没有过度简化路径和形状，但在很多情况下可能不是这样。观察上面火箭上的光带是如何以较低的精度扭曲的。

**在命令行使用SVGO:**

如果你喜欢命令行多于GUI，可以全局安装一个SVGO[npm CLI ](https://www.npmjs.com/package/svgo):

```Bash
npm i -g svgo
```

然后可以对一个本地SVG文件运行如下操作:

```Bash
svgo input.svg -o output.svg
```

它支持你可能期望的所有选项，包括调整浮点精度:

```Bash
svgo input.svg --precision=1 -o output.svg
```

有关支持的选项的完整列表，请参见SVGO[自述文件](https://github.com/svg/svgo)。

**不要忘记压缩SVG！**

![Performance](https://images.guide/images/book-images/before-after-svgo-large.jpg)

另外，不要忘记给[SVG资源开Gzip](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)或使用Brotli提供。由于它们是基于文本的，所以可以很好地压缩(源代码大约可以压缩50%)。

当谷歌发布一个新徽标时，我们宣布[最小的](https://twitter.com/addyosmani/status/638753485555671040)版本只有305字节。

![Performance](https://images.guide/images/book-images/Modern-Image30-large.jpg)

你可以使用[许多高级SVG技巧](https://www.clicktorelease.com/blog/svg-google-logo-in-305-bytes/)将其进一步缩减(一直到146字节)！我只想说，无论是通过工具还是手动清理，你都可以删除更多的SVG节点。

**SVG精灵**

SVG可以为图标提供[强大的](https://css-tricks.com/icon-fonts-vs-svg/)功能，它提供了一种将可视化表示为精灵的方法，而不需要为图标字体提供[奇怪的](https://www.filamentgroup.com/lab/bulletproof_icon_fonts.html)转变方法。它具有比图标字体(SVG笔画属性)更细粒度的CSS样式控制、更好的定位控制(不需要修改伪元素和CSS```display```),而且SVG[更容易访问](http://www.sitepoint.com/tips-accessible-svg/)。

像[svg-sprite](https://github.com/jkphl/svg-sprite)和[IcoMoon](https://icomoon.io/)这样的工具可以自动将SVG组合成可以使用的精灵，通过[CSS精灵](https://css-tricks.com/css-sprites/)、[符号精灵](https://css-tricks.com/svg-use-with-external-reference-take-2)或[堆栈精灵](http://simurai.com/blog/2012/04/02/svg-stacks)。Una Kravetz[写了](https://una.im/svg-icons/#%F0%9F%92%81)一篇关于如何将gulp-svg-sprite用于SVG精灵工作流的实用文章，值得一看。Sara Soudein还在她的博客中介绍了[从图标字体到SVG的转换](https://www.sarasoueidan.com/blog/icon-fonts-to-svg/)。

**了解更多**

Sara Soueidan的《[web优化SVG交付的技巧](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)》和Chris Coyier的《[SVG实践](https://abookapart.com/products/practical-svg)》都非常优秀。
我还发现Andreas Larsen的SVG优化文章([第1部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-1-67e8f2d4035)和[第2部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-2-6711cc15df46))很有启发。《[在Sketch中准备和导出SVG图标](https://medium.com/sketch-app-sources/preparing-and-exporting-svg-icons-in-sketch-1a3d65b239bb)》也是一个不错的文章。

## [8.避免使用有损编解码器重新压缩图像](https://images.guide/#avoid-recompressing-images-lossy-codecs)

建议始终对原始图像进行压缩。重新压缩图像会有后果。假设你有一个JPEG，它已经被压缩，质量为60。如果你用有损编码重新压缩此图像，它看起来会更糟。每一轮额外的压缩都将引入分代丢失——信息将丢失，压缩工件将开始构建。即使你在高质量的环境下重新压缩。

为了避免这个陷阱，**首先设置你愿意接受的最低质量的文件**，你将从一开始就获得最大的文件节省。然后你就可以避免这个陷阱，因为仅从降低质量的角度来看，任何文件大小的减少都是不好的。

重新编码一个有损耗的文件几乎总是会得到一个较小的文件，但这并不意味着你可以从中获得你可能认为的那么高的质量。

![Performance](https://images.guide/images/book-images/generational-loss-large.jpg)

上面，从Jon Sneyers的这段[优秀的视频](https://www.youtube.com/watch?v=w7vXJbLhTyI)和[附带的文章](http://cloudinary.com/blog/why_jpeg_is_like_a_photocopier)中，我们可以看到使用几种格式进行重新压缩的世代损失影响。如果从社交网络中保存(已压缩)图像并重新上传(导致重新压缩)，你可能会遇到这个问题。质量损失会增加。

由于网格量化，MozJPEG（可能是偶然的）对再压缩降级具有更好的抵抗力。 它不是精确地压缩所有DCT值，而是检查+1/-1范围内的接近值，以查看类似值是否压缩为更少的位。有损FLIF有一个类似于有损PNG的hack，在(重新)压缩之前，它可以查看数据并决定丢弃什么。重新压缩的png有“漏洞”，它可以检测，以避免进一步改变数据。

**在编辑源文件时，以无损的格式(如PNG或TIFF)存储它们，以便尽可能地保持质量。**然后，你的构建工具或图像压缩服务将以最小的质量损失向用户提供你所输出的压缩版本。

## [9.减少不必要的图解码并调整成本](https://images.guide/#reduce-unnecessary-image-decode-costs)

我们之前已经为我们的用户提供了比我们用户所需的大、更高分辨率的图像。 这需要付出代价的。对于一般的移动硬件来说，解码和调整图像大小是非常昂贵的操作。如果向下发大图像并使用CSS或宽/高属性进行缩放，你可能会看到这种情况，它会影响性能。

![Performance](https://images.guide/images/book-images/image-pipeline-large.jpg)

当浏览器获取图像时，它必须将图像从原始源格式（例如JPEG）解码为内存中的位图。 通常需要调整图像的大小（例如，宽度已设置为其容器的百分比）。 解码和调整图像大小非常昂贵，并且可能会延迟显示图像所需的时间。

发送浏览器完全不需要调整大小就可以呈现的图像是最理想的。因此，利用[```srcset```和```size```](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)的优势，为你的目标屏幕大小和分辨率提供最小的图像——我们将很快介绍```srcset```。

省略图像上的```width```或```height```属性也会对性能产生负面影响。 如果没有它们，浏览器会为图像指定一个较小的占位符区域，直到有足够的字节到达它以便知道正确的尺寸。 此时，文档布局必须在称为回流的昂贵步骤中进行更新。

![Performance](https://images.guide/images/book-images/devtools-decode-large.jpg)

浏览器必须经过许多步骤才能在屏幕上绘制图像。除了获取图像外，还需要对图像进行解码并经常调整大小。这些事件可以在Chrome DevTools [Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference)中进行审计。

较大的图像还会增加内存大小成本。解码后的图像每像素约4字节。如果你并不关心，你会让浏览器崩溃;在低端设备上，启动内存交换并不需要那么多。因此，请关注你的图像解码、调整大小和内存成本。

![Performance](https://images.guide/images/book-images/image-decoding-mobile-large.jpg)

在普通和低端手机上解码图像的成本非常高。 在某些情况下，解码速度可能会慢5倍（如果不是更长）。

在构建新的[移动网络体验](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)时，Twitter通过确保为用户提供适当大小的图像来提高图像解码性能。 这需要Twitter许多图像的解码时间Timeline从大约400毫秒一直降到大约19毫秒！

![Performance](https://images.guide/images/book-images/image-decoding-large.jpg)

Chrome DevTools时间轴/性能面板突出显示Twitter Lite优化其图像管道之前和之后的图像解码时间（绿色）。

### [9.1 使用```srcset``提供HiDPI图像`](https://images.guide/#delivering-hidpi-with-srcset)

用户可以通过一系列具有高分辨率屏幕的移动和桌面设备访问你的网站。 [设备像素比率](https://stackoverflow.com/a/21413366)（DPR）（也称为“CSS像素比率”）决定了CSS如何解释设备的屏幕分辨率。 DPR由手机制造商创建，旨在提高移动屏幕的分辨率和清晰度，而不会使元素显得太小。

为了匹配用户可能期望的图像质量，请向其设备提供最合适的分辨率图像。 可以将锐利的高DPR图像（例如2倍，3倍）提供给支持它们的设备。 低和标准DPR图像应该在没有高分辨率屏幕的情况下提供给用户，因为这样的2倍+图像通常会产生更多的字节。

![Performance](https://images.guide/images/book-images/device-pixel-ratio-large.jpg)

设备像素比率：许多网站都会跟踪常用设备的DPR，包括[material.io](https://material.io/devices/)和[mydevice.io](https://mydevice.io/devices/)。

[srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)允许浏览器韦每个设备选择最佳可用图像,例如为一个2x移动显示器选择一个2×图像。不支持```srcset```的浏览器可以退回到```<img>```标记中指定的默认```src```标签。

```html
<img srcset="paul-irish-320w.jpg,
             paul-irish-640w.jpg 2x,
             paul-irish-960w.jpg 3x"
     src="paul-irish-960w.jpg" alt="Paul Irish cameo">
```

像[Cloudinary](http://cloudinary.com/blog/how_to_automatically_adapt_website_images_to_retina_and_hidpi_devices)和[Imgix](https://docs.imgix.com/apis/url/dpr)这样的图像CDN都支持控制图像密度，以便从单一规范源向用户提供最佳密度。

> 注意: 在这个免费的Udacity课程和Web基础上的图像指南中，你可以了解更多关于设备像素比和响应图像的信息。一个友好的提醒，[客户端提示](https://www.smashingmagazine.com/2016/01/leaner-responsive-images-client-hints/)也可以提供一种替代方案，在你的响应图像标记中指定每个可能的像素密度和格式。相反，它们将这些信息附加到HTTP请求中，以便web服务器能够选择最适合当前设备屏幕密度的信息。

### [9.2 艺术方向](https://images.guide/#art-direction)

虽然向用户提供正确的解决方案很重要，但有些网站还需要从[艺术方向](http://usecases.responsiveimages.org/#art-direction)考虑这一点。 如果用户位于较小的屏幕上，你可能需要裁剪或放大并显示主题以充分利用可用空间。 尽管艺术方向超出了本文的范围，但像[Cloudinary](http://cloudinary.com/blog/automatically_art_directed_responsive_images%20)这样的服务提供的API可以尽可能地尝试自动化。

![Performance](https://images.guide/images/book-images/responsive-art-direction-large.jpg)

艺术方向：埃里克·波蒂斯（Eric Portis）汇集了一个优秀的[样本](https://ericportis.com/etc/cloudinary/)，展示了在艺术方向上将如何响应式图像。 此示例调整主要横幅在不同断点处的视觉特征，以充分利用可用空间。

## [10.色彩管理](https://images.guide/#color-management)

色彩至少有三种不同的视角:生物、物理和印刷。在生物学中，颜色是一种[感知现象](http://hubel.med.harvard.edu/book/ch8.pdf)。物体反射不同波长的光。我们眼睛中的光感受器将这些波长转换成我们所知的颜色。在物理学中，光很重要——光的频率和亮度。印刷更多的是关于色轮，油墨和艺术模型。

理想情况下，世界上的每个屏幕和web浏览器显示的颜色完全相同。不幸的是，由于一些固有的不一致，它们做不到。色彩管理允许我们通过色彩模型、空间和配置文件在显示色彩方面达成妥协。

**颜色模型**

[颜色模型](https://en.wikipedia.org/wiki/Gamma_correction)是一种从较小的一组原色生成完整的颜色范围的系统。有不同类型的颜色空间，它们使用不同的参数来控制颜色。有些颜色空间的控制参数比其他颜色空间少，例如灰度只有一个参数来控制黑色和白色之间的亮度。

两种常见的颜色模型是加色和减色。加色模型(如RGB，用于数字显示)使用光来显示颜色，而减色模型(如CMYK，用于打印)则通过去除光来工作。

![Performance](https://images.guide/images/book-images/colors_ept6f2-large.jpg)

在RGB中，红色、绿色和蓝色的光以不同的组合方式被加入，以产生广谱的颜色。CYMK(青色、品红、黄色和黑色)通过不同颜色的墨水从白纸上减去亮度来工作。

[了解颜色模型和专色系统](https://www.designersinsights.com/designer-resources/understanding-color-models/)对其他颜色模型和模式有很好的描述，如HSL、HSV和LAB。

**颜色空间**

[颜色空间](http://www.dpbestflow.org/color/color-space-and-color-profiles#space)是一个特定的颜色范围，可以表示为给定的图像。例如，如果一幅图像包含多达1670万种颜色，不同的颜色空间允许使用这些颜色的更窄或更宽的范围。一些开发人员将颜色模型和颜色空间称为同一事物。

[sRGB](https://en.wikipedia.org/wiki/SRGB)是为web设计的[标准](https://www.w3.org/Graphics/Color/sRGB.html)颜色空间，它基于RGB。它是一个很小的颜色空间，通常被认为是跨浏览器颜色管理的最小公分母，也是最安全的选项。其他颜色空间(如[Adobe RGB](https://en.wikipedia.org/wiki/Adobe_RGB_color_space)或[ProPhoto RGB](https://en.wikipedia.org/wiki/ProPhoto_RGB_color_space)——在Photoshop和Lightroom中使用)可以比sRGB表示更鲜艳的颜色，但由于后者在大多数web浏览器、游戏和显示器中更普遍，所以它是人们通常关注的焦点。

![Performance](https://images.guide/images/book-images/color-wheel_hazsbk-large.jpg)

上面我们可以看到色域的可视化——颜色空间可以定义的颜色范围。

颜色空间有三个通道(红色、绿色和蓝色)。在8位模式下，每个通道可以有255种颜色，总共1670万种。16位图像可以显示数万亿种颜色。

![Performance](https://images.guide/images/book-images/srgb-rgb_ntuhi4-large.jpg)

使用[标尺](https://yardstick.pictures/tags/img%3Adci-p3)中的图像比较sRGB、Adobe RGB和ProPhoto RGB。在sRGB中显示这个概念是非常困难的，因为无法显示看不到的颜色。一张普通的sRGB和宽色域的照片应该有所有相同的东西，除了大多数饱和的“多汁”颜色。

颜色空间(如sRGB、Adobe RGB和ProPhoto RGB)的差异在于它们的色域(它们可以通过阴影再现的颜色范围)、光源和[伽马](http://blog.johnnovak.net/2016/09/21/what-every-coder-should-know-about-gamma/)曲线。sRGB比Adobe RGB小~20%，ProPhoto RGB比Adobe RGB大~[50%](http://www.petrvodnakphotography.com/Articles/ColorSpace.htm)。上面的图像源来自[剪切路径](http://clippingpathzone.com/blog/essential-photoshop-color-settings-for-photographers)。

[宽色域](http://www.astramael.com/)是描述色域大于sRGB的颜色空间的术语。这种类型的显示器正变得越来越普遍。也就是说，许多数字显示器仍然无法显示明显优于sRGB的彩色配置文件。在Photoshop中保存网页时，考虑使用“转换到sRGB”选项，除非用户使用的是更高端的宽屏幕。

> 注意: 在使用原始照片时，避免使用sRGB作为基本颜色空间。它比大多数相机支持的颜色空间要小，还会造成剪切。相反，你可以在更大的颜色空间(如ProPhoto RGB)上工作，并在导出web时输出到sRGB。

**在任何情况下，广泛的范围对web内容有意义吗?**

有的。如果一个图像包含非常饱和的、多汁的、充满活力的颜色，而你关心的是它在支持它的屏幕上是同样多汁的。然而，在真实的照片中，这种情况很少发生。通常很容易调整颜色，使其看起来充满活力，而实际上不超过sRGB的色域。

这是因为人类对颜色的感知不是绝对的，而是相对于我们周围的环境而言的，很容易被愚弄。如果你的图像包含荧光高亮颜色，那么你将有一个更容易的时间与广泛的范围。

**伽玛校正与压缩**

[伽玛校正](https://en.wikipedia.org/wiki/Gamma_correction)(或仅仅伽玛)控制图像的整体亮度。改变伽马也可以改变红绿蓝的比例。没有伽玛校正的图像可能看起来颜色被漂白或太暗。

在视频和计算机图形学中，伽玛用于压缩，类似于数据压缩。这允许你在更少的位(8位而不是12或16位)中压缩有用的亮度级别。人类对亮度的感知并不正比于物理光的量。在为人眼编码图像时，用真实的物理形式表示颜色是一种浪费。伽玛压缩被用来在更接近人类感知的尺度上对亮度进行编码。

使用伽玛压缩有用的亮度范围适合8位精度(0-255使用的大多数RGB颜色)。所有这些都来自这样一个事实:如果颜色使用与物理1:1关系的单位，那么RGB值将从1到100万，其中0-1000的值看起来不同，但是999000-1000000之间的值看起来相同。想象一下，在一个只有一支蜡烛的黑暗房间里。点第二根蜡烛，你会发现房间灯光的亮度明显增加。再加第三根蜡烛，它看起来会更亮。现在想象一下，在一个有100支蜡烛的房间里。点燃第101根蜡烛，第102根。你不会注意到亮度的变化。

即使在这两种情况下，物理上，增加的光量是完全相同的。因此，由于眼睛对明亮的光线不那么敏感，伽玛压缩“压缩”了明亮的值，所以从物理角度来说，明亮的级别不那么精确，但是尺度是为人类调整的，所以从人类的角度来看，所有的值都是一样精确的。

> 注意: 这里的伽马压缩/校正不同于你可能在Photoshop中配置的图像伽马曲线。当伽马压缩正常工作时，它看起来不像任何东西。

**颜色配置文件**

颜色配置文件是描述设备的颜色空间的信息。它用于在不同的颜色空间之间进行转换。配置文件试图确保图像在这些不同类型的屏幕和介质上看起来尽可能相似。

图像可以像[国际色彩协会](http://www.color.org/icc_specs2.xalter)(ICC)所描述的那样嵌入颜色配置文件，以精确地表示颜色应该如何显示。包括JPEG、PNG、SVG和[WebP](https://developers.google.com/speed/webp/docs/riff_container)在内的不同格式都支持这一点，大多数主流浏览器都支持嵌入式ICC配置文件。当一个图像显示在一个应用程序中，它知道显示器的功能，这些颜色可以根据颜色配置文件进行调整。

> 注意: 有些监视器的颜色配置文件类似于sRGB，不能显示更好的配置文件，因此根据目标用户的显示，嵌入这些配置文件的价值可能有限。检查你的目标用户是谁。

嵌入的颜色配置文件也会大大增加图像的大小(偶尔会增加100KB以上)，所以要小心嵌入。像ImageOptim这样的工具如果找到颜色配置文件，就会[自动](https://imageoptim.com/color-profiles.html)删除它们。相反，如果以减小大小的名义删除ICC配置文件，浏览器将被迫在监视器的颜色空间中显示图像，这可能导致预期的饱和度和对比度的差异。评估这个的权衡对你的用例是有意义的。

如果你有兴趣了解更多关于配置文件的信息，[下面的9个Degree](https://ninedegreesbelow.com/photography/articles.html)提供了一组关于ICC配置文件颜色管理的优秀资源。

**颜色配置文件和web浏览器**

早期版本的Chrome对色彩管理的支持不是很好，但随着2017年[色彩正确渲染](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/ptuKdRQwPAo)的出现，这一功能正在改善。不是sRGB(新款MacBook pro)的显示器将把颜色从sRGB转换为显示配置文件。这意味着在不同的系统和浏览器中颜色应该看起来更相似。Safari、Edge和Firefox现在也可以将ICC配置文件考虑在内，因此具有不同颜色配置文件(例如ICC)的图像现在可以正确地显示它们，无论你的屏幕是否具有广域。

> 注意: 有关如何将颜色应用于更广泛的web工作方式的指南，请参阅由Sarah Drasner编写的书呆子色彩指南。

## [11.图像精灵](https://images.guide/#image-sprites)

[图像精灵](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images#use_image_sprites)(或CSS精灵)在web上有很长的历史，所有浏览器都支持它，通过将它们组合成一个更大的图像切片来减少页面加载的图像数量，这是一种流行的方法。

![Performance](https://images.guide/images/book-images/i2_2ec824b0_1-large.jpg)

图像精灵仍然广泛应用于大型生产环境站点，包括谷歌主页。

在HTTP/1.x下，一些开发人员使用spriting来减少HTTP请求。这样做有很多好处，但是需要小心，因为你很快就会遇到缓存失效的挑战——对图像精灵的任何一小部分的更改都会使用户缓存中的整个图像无效。

然而，Spriting现在可能是[HTTP/2](https://hpbn.co/http2/)反模式。 使用HTTP/2，最好[加载单个图像](https://deliciousbrains.com/performance-best-practices-http2/)，因为现在可以在单个连接中进行多个请求。 评估你自己的网络设置是否属于这种情况。

## [12.延迟加载非关键图像](https://images.guide/#lazy-load-non-critical-images)

延迟加载是一种web性能模式，它会延迟浏览器中图像的加载，直到用户需要看到它为止。例如，在滚动时，图像按需异步加载。这可以进一步补充使用图像压缩策略所节省的字节数。

![Performance](https://images.guide/images/book-images/scrolling-viewport-large.jpg)

必须出现在“折叠上方”的图像，或者当web页面首次出现时立即加载。然而，“折叠下方”的图像对用户来说还不可见。它们不必立即加载到浏览器中。只有当用户向下滚动并需要显示它们时，才可以稍后加载它们，或者延迟加载它们。

浏览器本身还不支持延迟加载(尽管过去已经[讨论](https://discourse.wicg.io/t/a-standard-way-to-lazy-load-images/1153/10)过)。相反，我们使用JavaScript添加此功能。

**为什么延迟加载有用?**

这种“懒惰”的方式只在必要时加载图像，有很多好处:

+ **减少数据消耗**: 由于你没有假设用户需要提前获取每个图像，所以你只加载了最少数量的资源。这总是一件好事，尤其是在数据流量更加受限的移动设备上。

+ **减少电池消耗**: 减少用户浏览器的工作量，节省电池寿命。

+ **提高下载速度**: 在一个大量图像的网站上，将页面加载时间从几秒减少到几乎为零，这对用户体验是一个巨大的提升。事实上，这可能是用户留下来享受你的网站和另一个反弹统计之间的差异。

**但是就像所有的工具一样，强大的力量伴随着巨大的责任。**

**避免在折叠上方延迟加载图像。**将其用于长图像列表(例如产品)或用户头像列表。不要将其用于主页横幅。折叠上方的延迟加载图像可以使加载明显变慢，无论是在技术上还是在人类感知上。它可以杀死浏览器的预加载程序，逐步加载，JavaScript可以为浏览器创建额外的工作。

**滚动时要小心延迟加载图像。**如果你一直等到用户滚动页面，他们很可能会看到占位符，并最终获得图像，如果他们还没有滚动它们。一种建议是在加载了上述图像之后开始延迟加载，加载所有独立于用户交互的图像。

**谁使用延迟加载?**

对于延迟加载的例子，请查看大多数承载大量图像的主要站点。一些著名的网站是[Medium](https://medium.com/)和[Pinterest](https://www.pinterest.com/)。

![Performance](https://images.guide/images/book-images/Modern-Image35-large.jpg)

一个高斯模糊内联预览图像在Medium.com的例子

许多站点(如Medium)显示一个很小的、高斯模糊的内联预览(只有100多字节)，它在获取后(延迟加载)转换为高质量的图像。

José M. Pérez写过关于如何使用[CSS过滤器](https://jmperezperez.com/medium-image-progressive-loading-placeholder/)实现媒体效果的文章，并尝试了[不同的图像格式](https://jmperezperez.com/webp-placeholder-images/)来支持这种占位符。Facebook还写了一篇文章，介绍了他们著名的200字节方法，即为[封面照片](https://code.facebook.com/posts/991252547593574/the-technology-behind-preview-photos/)提供占位符，值得一读。如果你是Webpack用户，[LQIP loader](https://lqip-loader.firebaseapp.com/)可以帮助你自动完成一些工作。

实际上，你可以搜索你最喜欢的高分辨率照片来源，然后向下滚动页面。几乎在所有情况下，你都会体验到网站一次只加载几个全分辨率图像，其余都是占位符颜色或图像。当你继续滚动时，占位符图像将被全分辨率图像替换。这是懒加载。

最近一直在进行的一项技术是矢量而不是基于像素的低质量图像预览，由Tobias Baldauf在他的工具[SQIP](https://github.com/technopagan/sqip)中进行试验。 这种方法利用实用程序[Primitive](https://github.com/fogleman/primitive)生成SVG预览，该预览由几个简单的形状组成，这些形状近似于目标图像中可见的主要特征，使用[SVGO](https://github.com/svg/svgo)优化SVG，最后为其添加高斯模糊滤波器; 生成SVG占位符，该占位符的权重仅为800-1000字节，在所有屏幕上看起来都很清晰，并提供了图像内容的视觉提示。 延迟加载和低质量图像预览都可以明显地[结合](https://calendar.perfplanet.com/2017/progressive-image-loading-using-intersection-observer-and-sqip/)起来。

**如何将懒加载应用于页面?**

有许多技术和插件可用于延迟加载。我推荐Alexander Farkas的[Lazysizes](https://github.com/aFarkas/lazysizes)，因为它有很好的性能、特性、与[Intersection Observer](https://developers.google.com/web/updates/2016/04/intersectionobserver)可选集成以及对插件的支持。

**我可以用Lazysizes做什么?**

Lazysizes是一个JavaScript库。它不需要配置。下载压缩过的js文件，并include在你的网页。

下面是自述文件中的一些示例代码:

将“lazyload”类与data-src和/或data-srcset属性一起添加到图像或者iframes中。

你也可以选择添加一个src属性与一个低质量的图像:

```html
<!-- non-responsive: -->
<img data-src="image.jpg" class="lazyload" />

<!-- responsive example with automatic sizes calculation: -->

<img
    data-sizes="auto"
    data-src="image2.jpg"
    data-srcset="image1.jpg 300w,
    image2.jpg 600w,
    image3.jpg 900w" class="lazyload" />

<!-- iframe example -->

<iframe frameborder="0"
    class="lazyload"
    allowfullscreen=""
    data-src="//www.youtube.com/embed/ZfV-aYdU4uE">
</iframe>
```

在这本书的web版本中，我将Lazysizes(尽管你可以使用任何替代方案)与Cloudinary配对，用于按需响应图像。这让我可以自由地尝试不同的规模、质量、格式的值，以及是否以最小的价值逐步加载:

![Performance](https://images.guide/images/book-images/cloudinary-responsive-images-large.jpg)

**Lazysizes功能包括:**

+ 自动检测当前和未来lazyload元素的可见性变化

+ 包括标准响应图像支持(图像和srcset)

+ 为媒体查询功能添加自动大小计算和别名

+ 可以使用数百图像或者iframe的包含CSS和js很多的网页或web应用程序

+ 可扩展:支持插件

+ 轻量级但成熟的解决方案

+ SEO改进:不隐藏图像或其他资源的爬虫

**更多延迟加载选项**

Lazysizes不是你唯一的选择。这里有更多的懒加载库:

+ [Lazy Load XT](http://ressio.github.io/lazy-load-xt/)

+ [BLazy.js](https://github.com/dinbror/blazy) (or [Be]Lazy)

+ [Unveil](http://luis-almeida.github.io/unveil/)

+ [yall.js (另一个懒加载器)](https://github.com/malchata/yall.js)，大约1KB，在支持的地方使用Intersection Observer。

**懒加载的陷阱是什么?**

+ 屏幕阅读器、一些搜索机器人和任何禁用JavaScript的用户将无法查看用JavaScript懒加载了图像。然而，我们可以使用```<noscript>```回退来解决这个问题。

+ 滚动侦听器(如用于确定何时加载延迟加载的图像)可能对浏览器滚动性能产生不利影响。它们会导致浏览器多次重绘，将进程拖得像乌龟爬一样慢——然而，聪明的懒加载库将使用节流来缓解这种情况。一个可能的解决方案是由Lazysizes支持的Intersection Observer。

延迟加载图像是一种用于减少带宽、降低成本和改善用户体验的普遍模式。评估它对你的经验是否有意义。有关进一步阅读，请参见[延迟加载图像](https://jmperezperez.com/lazy-loading-images/)和[实现媒体的渐进加载](https://jmperezperez.com/medium-image-progressive-loading-placeholder/)。

## [13.避免```display:none```陷阱](https://images.guide/#display-none-trap)

旧的响应图像解决方案在设置CSS ```display```属性时错误地理解了浏览器如何处理图像请求。这可能会导致比你预期的要多得多的图像被请求，这也是```<picture>```和```<img srcset>```更适合加载响应性图像的另一个原因。

你是否写过设置要显示的图像的媒体查询在某些断点```display:none```?

```html
<img src="img.jpg">
<style>
@media (max-width: 640px) {
    img {
        display: none;
    }
}
</style>
```

或者使用```display:none```类来切换隐藏的图像?

```html
<style>
.hidden {
  display: none;
}
</style>
<img src="img.jpg">
<img src=“img-hidden.jpg" class="hidden">
```

对Chrome DevTools的Network面板的快速检查将验证使用这些方法隐藏的图像仍然会被获取，即使我们预期是它们不会被获取。根据嵌入式资源规范，这种行为实际上是正确的。

![Performance](https://images.guide/images/book-images/display-none-images-large.jpg)

**```display:none```避免触发对图像```src```的请求?**

```html
<div style="display:none">
  <img src="img.jpg">
</div>
```

不行的。指定的图像仍然会被请求。库这里不能依赖```display:none```没有显示，因为在JavaScript更改src之前已经将请求图像。

**```display:none```是否避免触发对```background: url()```的请求?**

```html
<div style="display:none">
  <div style="background: url(img.jpg)"></div>
</div>
```

可以的。CSS背景不会在元素被解析时立即获取。计算带有```display:none```的子元素的CSS样式时，这种样式会不起作用，因为它们不会影响文档的呈现。子元素的背景图像不会被计算或者下载。

Jake Archibald的[Request Quest](https://jakearchibald.github.io/request-quest/)有一个关于使用```display:none```进行响应式图像加载的缺陷进行了很好的测验。 如果对特定浏览器如何处理图像请求加载有疑问，请打开他们的DevTools并自行验证。

同样，在可能的情况下，使用```<picture>```和```<img srcset>```代替```display:none```。

## [14.图像处理CDN对你有意义吗？](https://images.guide/#image-processing-cdns)

*你将花费在阅读博客文章来设置自己的图像处理管道和调整配置上的时间通常是远远多于买服务的费用。有了[Cloudinary](http://cloudinary.com/)提供的免费服务，[Imgix](https://www.imgix.com/)提供的免费试用，[Thumbor](https://github.com/thumbor/thumbor)作为OSS的备选方案已经存在，你可以使用许多选项来实现自动化。*

为了获得最佳的页面加载时间，你需要优化图像加载。这种优化需要一个响应式图像策略，可以从服务器上的图像压缩、自动选择最佳格式和响应式调整大小中获益。重要的是，你要尽可能快地将正确大小的图像以正确的分辨率发送到适当的设备。做这件事不像人们想象的那么容易。

**使用服务器vs.a CDN**

由于图像处理的复杂性和不断发展的本质，我们将引用在该领域有经验的人的话，然后继续提出建议。

"如果你的产品不是图像处理，那就不要自己动手。像Cloudinary(或imgix, Ed.)这样的服务比你预期的要高效得多，所以要使用它们。如果你担心成本，想想你在开发和维护上要花多少钱，还有托管、存储和交付成本。"——[Chris Gmyr](https://medium.com/@cmgmyr/moving-from-self-hosted-image-service-to-cloudinary-bd7370317a0d)

目前，我们同意并建议你考虑使用CDN来满足你的图像处理需求。我们将检查两个CDN，看它们与我们前面提出的任务列表相比如何。

**Cloudinary和imgix**

[Cloudinary](http://cloudinary.com/)和[imgix](https://www.imgix.com/)是两种成熟的图像处理CDN。它们是全球数十万开发者和公司的选择，包括Netflix和Red Bull。让我们更详细地看看它们。

**基础是什么?**

除非你是像他们这样的服务器网络的所有者，否则与使用你自己的解决方案相比，他们的第一个巨大优势是使用分布式全局网络系统使镜像的副本更接近你的用户。对于CDN来说，随着趋势的变化，“未来证明”你的图像加载策略也要容易得多——这需要你自己进行维护、跟踪浏览器对新兴格式的支持以及跟踪图像压缩社区。

其次，每个服务都有一个分层定价方案，Cloudinary提供[免费级别](http://cloudinary.com/pricing)，imgix相对于他们的高容量付费方案，以较低的价格为他们的标准级别定价。Imgix提供免费[试用](https://www.imgix.com/pricing)，并提供服务积分，因此几乎等同于免费级别。

第三，两个服务都提供API访问。开发人员可以通过编程方式访问CDN并自动处理。客户端库、框架插件和API文档也是可用的，其中一些功能仅限于较高的付费级别。

**我们来看看图像处理**

现在，我们只讨论静态图像。Cloudinary和Imgix都提供了一系列的图像处理方法，并且都在它们的标准和免费方案中支持压缩、调整大小、裁剪和创建缩略图等主要功能。

![Performance](https://images.guide/images/book-images/Modern-Image36-large.jpg)

Cloudinary媒体库: 默认情况下，Cloudinary对[非渐进式JPEG](http://cloudinary.com/blog/progressive_jpegs_and_green_martians)进行编码。要选择生成它们，请检查“More options”中的“Progressive”选项或传递“fl_progressive”标志。

Cloudinary列出了[7个广泛的图像转换](http://cloudinary.com/documentation/image_transformations)类别，其中共有48个子类别。Imgix广告超过[100个图像处理操作](https://docs.imgix.com/apis/url?_ga=2.52377449.1538976134.1501179780-2118608066.1501179780)。

**默认情况下会发生什么?**

+ Cloudinary在默认情况下执行以下优化:

+ [使用MozJPEG编码JPEG](https://twitter.com/etportis/status/891529495336722432)(选择不使用Guetzli作为默认值)

+ 从转换后的图像文件中删除所有相关的元数据(原始图像保持不变)。若要覆盖此行为并交付转换后的图像，且其元数据完整，请添加keep_iptc标志。

+ 可以生成WebP, GIF, JPEG，和JPEG- xr格式的自动质量。要覆盖默认的调整，请在转换中设置quality参数。

+ 在生成PNG、JPEG或GIF格式的图像时，运行[优化](http://cloudinary.com/documentation/image_optimization#default_optimizations)算法以最小化文件大小，对视觉质量的影响最小。

Imgix没有像Cloudinary那样的默认优化。它有一个可设置的默认图像质量。对于imgix, auto参数可以帮助你在整个图像目录中实现基线优化级别的自动化。

目前有四种不同的方法:

+ 压缩（Compression）

+ 视觉增强（Visual enhancement）

+ 文件格式转换（File format conversion）

+ 去除噪点（Redeye removal）

Imgix支持以下图像格式:JPEG, JPEG2000, PNG, GIF, GIF动画，TIFF, BMP, ICNS, ICO, PDF, PCT, PSD, AI。

Cloudinary支持以下图像格式:JPEG, JPEG 2000, JPEG XR, PNG, GIF，GIF动画, WebP，WebP动画, BMPs, TIFF, ICOs, PDF, EPS, PSD, SVG, AI, DjVu, FLIF, TARGA。

**性能怎么样?**

CDN输送性能主要与延迟和速度有关。

对于完全未缓存的图像，延迟总是有所增加。 但是，一旦图像被缓存并在网络服务器之间分配，全局CDN可以找到最短的用户访问路径，加上正确处理的图像的字节节省，与处理不当相比，几乎总能缓解延迟问题。 试图穿越地球的图像或孤零零的服务器。

这两种服务都使用快速和广泛的CDN。这种配置减少了延迟并提高了下载速度。下载速度影响页面加载时间，这是用户体验和转换的最重要指标之一。

**那么它们是如何比较的呢?**

Cloudinary拥有[16万用户](http://cloudinary.com/customers)，包括Netflix、eBay和Dropbox。Imgix没有报告它有多少客户，但它比Cloudinary小。
即便如此，imgix的基础包括重量级的图像用户，如Kickstarter、Exposure、unsplash和Eventbrite。

在图像处理中有太多的不可控变量，因此很难在两个服务之间进行面对面的性能比较。这在很大程度上取决于你需要处理多少图像(这需要可变的时间)，以及最终输出需要多大的大小和分辨率，这将影响速度和下载时间。对你来说，成本可能是最重要的因素。

cdn需要成本。一个大量图像、大流量的网站每月的CDN费用可能高达数百美元。要充分利用这些服务，需要具备一定的知识和编程技能。如果你不做任何太花哨的事情，你可能不会有任何麻烦。

但是如果你不习惯使用图像处理工具或API，那么你将看到一条学习曲线。为了适应CDN服务器位置，你需要更改本地链接中的一些url。做正确的尽职调查:)

**结论**

如果你目前正在提供自己的图像或计划提供，也许你可以考虑一下CDN。

## [15.缓存图像资源](https://images.guide/#caching-image-assets)

资源可以使用HTTP缓存标头指定缓存策略。 具体来说，Cache-Control可以定义谁可以缓存响应以及响应时间。

资源可以使用[HTTP缓存标头](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control)指定缓存策略。具体来说，```Cache-Control```可以定义谁可以缓存响应多长时间。

你向用户提供的大多数图像都是静态资源，将来[不会发生变化](http://kean.github.io/post/image-caching)。 这类资产的最佳缓存策略是主动缓存。

设置HTTP缓存头文件时，设置Cache-Control的最大使用年限为一年(例如: ```Cache-Control: public; max-age = 31536000```)。这种类型的主动缓存对于大多数类型的图像都很有效，特别是那些长时间存在的图像，比如头像和标题图像。

> 注意：如果你使用PHP提供图像，则由于默认的[session_cache_limiter](http://php.net/manual/en/function.session-cache-limiter.php)设置，它可能会破坏缓存。 这可能是图像缓存的灾难，你可能希望通过设置session_cache_limiter（'public'）来[解决此问题](https://stackoverflow.com/a/3905468)，该会话将设置public，max-age =。 禁用和设置自定义缓存控制标头也很好。

## [16.预加载关键图像资源](https://images.guide/#preload-critical-image-assets)

关键的图像资产可以使用[```<link rel=preload>```](https://www.w3.org/TR/preload/)预加载。

```<link rel=preload>```是一个声明式获取，允许你在不阻塞文档的```onload```事件的情况下强制浏览器发出对资源的请求。它允许增加对资源的请求的优先级，否则在文档解析过程的后面才会发现这些资源。

可以通过指定图像的```as```值来预加载```image```:

```html
<link rel="preload" as="image" href="logo.jpg"/>
```

对于```<img>```， ```<picture>```， ```srcset```和SVG的图像资源都可以利用这种优化。

> 注意：```<link rel="preload">```在Chrome和基于Blink的浏览器(如Opera, [Safari Tech Preview](https://developer.apple.com/safari/technology-preview/release-notes/))中是被[支持](http://caniuse.com/#search=preload)的，并且已经在Firefox中[实现](https://bugzilla.mozilla.org/show_bug.cgi?id=1222633)。

像[Philips](https://www.usa.philips.com/)、[Flipkart](https://www.flipkart.com/)和[Xerox](https://www.xerox.com/)这样的网站使用```<link rel=preload>```来预加载他们的主要logo资源(通常在文档的早期使用)。[Kayak](https://kayak.com/)还使用preload来确保他们的头部的英雄图像被尽快加载。

![Performance](https://images.guide/images/book-images/preload-philips-large.jpg)

**什么是链接预加载头?**

可以使用HTML标记或[HTTP链接头部](https://www.w3.org/wiki/LinkHeader)指定预加载链接。 在任何一种情况下，预加载链接都会指示浏览器开始将资源加载到内存缓存中，这表明该页面需要高可信度才能使用该资源，并且不希望等待预加载扫描程序或解析程序发现它。

链接预加载头的图像将类似于这样:

```
Link: <https://example.com/logo-hires.jpg>; rel=preload; as=image
```

当英国《金融时报》在其网站上引入链接预加载页眉时，他们将显示报头图像的时间缩[短了1秒](https://twitter.com/wheresrhys/status/843252599902167040):

![Performance](https://images.guide/images/book-images/preload-financial-times-large.jpg)

底部:带```<link rel="preload">```，顶部:不带。在WebPageTest上比较Moto G4在3G以上的使用[前](https://www.webpagetest.org/result/170319_Z2_GFR/)[后](https://www.webpagetest.org/result/170319_R8_G4Q/)。

类似地，Wikipedia通过[案例研究](https://phabricator.wikimedia.org/phame/post/view/19/improving_time-to-logo_performance_with_preload_links/)中介绍的链接预加载标题改进了标记的时间性能。

**使用此优化时应考虑哪些注意事项?**

一定要确保预加载图像资产是值得的，因为如果它们对用户体验不重要，那么页面上的其他内容可能值得你更早地加载。通过对图像请求进行优先排序，你可能最终会将其他资源进一步推入队列。

重要的是要避免在没有广泛浏览器支持的情况下使用```rel=preload```来预加载图像格式(例如WebP)。对于```srcset```中定义的响应图像，最好避免使用它，因为在srcset中检索的源可能会根据设备条件发生变化。

要了解有关预加载的更多信息，请参阅[Chrome和预装中的预加载](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf)和[预取和优先级：它有什么用？](https://www.smashingmagazine.com/2016/02/preload-what-is-it-good-for/).

## [17.图像的性能预算](https://images.guide/#performance-budgets)

性能预算是团队尝试不超过的网页性能的“预算”。 例如，“任何页面上的图像不会超过200KB”或“用户体验必须在3秒内可用”。 如果没有达到预算，请探究原因以及如何回到目标上。

预算为与利益相关者讨论绩效提供了有用的框架。 如果设计或业务决策可能会影响网站性能，请参阅预算。 它们是在可能损害网站用户体验时延迟或重新考虑更改的参考。

我发现在监控自动化时，团队在绩效预算方面取得了最大的成功。 自动化可以标记何时超出预算，而不是手动检查网络瀑布的预算回归。 对性能预算跟踪有用的两个此类服务是[Calibre](https://calibreapp.com/docs/metrics/budgets)和[SpeedCurve](https://speedcurve.com/blog/tag/performance-budgets/)。

一旦定义了图像大小的性能预算，SpeedCurve就会开始监控并在超出预算时提醒你：

![Performance](https://images.guide/images/book-images/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0-large.jpg)

Calibre提供了类似的功能，支持为你定位的每个设备级设置预算。 这非常有用，因为你通过WiFi在桌面上调整图像大小的预算可能会因移动设备的预算而有很大差异。

![Performance](https://images.guide/images/book-images/budgets-large.jpg)

## [18.写在最后](https://images.guide/#closing-recommendations)

最终，选择图像优化策略将归结为你向用户提供的图像类型，以及你决定的一组合理的评估标准。 它可能正在使用SSIM或Butteraugli，或者，如果它是一组足够小的图像，那么人类的感知就会变得最有意义。

**以下是我最后的建议:**

如果你**不能**基于浏览器支持有条件地提供格式:

+ Guetzli + MozJPEG的jpegtran是JPEG质量>90的优秀优化器。

    - 对于web, ```q=90```是非常浪费的。你可以尝试```q=80```, 2倍屏显示即使```q=50```。因为Guetzli没有那么低，对于web，你可以使用MozJPEG。

    - Kornel Lesiński最近改善mozjpeg的cjpeg命令来添加小sRGB概要文件来帮助Chrome在屏幕上显示自然色

+ PNG pngquant + advpng具有很好的速度/压缩比

+ 如果你可以有条件的服务(使用```<picture>```，[接受头](https://www.igvita.com/2013/05/01/deploying-webp-via-accept-content-negotiation/)或[图像填充](https://scottjehl.github.io/picturefill/)):

    - 为支持WebP的浏览器提供WebP服务

        + 从原始的100%质量的图像创建WebP图像。否则，你将会给浏览器提供支持它的更糟糕的图像与JPEG失真和WebP失真!如果你使用WebP压缩未压缩的源图像，它会有较少的可见WebP失真，也可以压缩得更好。

        + WebP团队使用的```-m 4 - q 75```的默认设置通常适用于他们优化速度/比率的大多数情况。

        + WebP还有一种特殊的无损模式(```- m 6 - q 100```)，它可以通过探索所有参数组合将文件缩减到最小大小。虽然速度慢了一个数量级，但对于静态资产来说是值得的。

    - 作为一种退路，将Guetzli/MozJPEG压缩源提供给其他浏览器


压缩快乐!

> 注意：有关如何优化图像的更实用的指导，我强烈推荐Jeremy Wagner的[Web Performance in Action](https://www.manning.com/books/web-performance-in-action)。 [高性能图像](http://shop.oreilly.com/product/0636920039730.do)也充满了关于这个主题的优秀、细致的建议。

## [19.了解更多](https://images.guide/#trivia)

+ [JPEG XT](https://jpeg.org/jpegxt/)定义了1992 JPEG规范的扩展。为了使扩展能够在旧JPEG之上呈现完美的像素效果，规范必须澄清旧的1992规范，并选择[libjpeg-turbo](https://libjpeg-turbo.org/)作为参考实现(基于流行程度)。

+ [PIK](https://github.com/google/pik)是一种值得关注的新型图像编解码器。它与JPEG兼容，具有更有效的颜色空间，并利用了Guetzli中发现的类似优点。它的解码速度是JPEG的2/3，比libjpeg节省54%的文件。它比guetzli化的jpeg更快地解码和压缩。一项关于现代图像编码的心理视觉相似性的[研究](https://encode.ru/threads/2814-Psychovisual-analysis-on-modern-lossy-image-codecs)表明，PIK的大小还不到替代品的一半。不幸的是，目前编解码器还处于早期阶段，编码速度非常慢(2017年8月)。

+ [ImageMagick](https://www.imagemagick.org/script/index.php)经常被推荐用于图像优化。这篇文章认为它是一个很好的工具，但是它的输出通常需要更多的优化，其他工具可以提供更好的输出。我们建议使用[libvips](https://github.com/jcupitt/libvips)，但是它的级别较低，需要更多的技术技能。ImageMagick还历史性地[指出](https://imagetragick.com/#moreinfo)了你可能需要注意的安全漏洞。

+ Blink (Chrome使用的渲染引擎)从主线程上解码图像。将解码工作转移到合成器线程将释放主线程来处理其他任务。我们称之为延迟解码。在延迟译码的情况下，译码工作仍然处于帧显示的关键路径上，仍然会造成动画抖动。[img.decode()](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-img-decode) API应该有助于解决jank问题。

本书的内容遵循知识共享[属性-非商业-NoDerivs 2.0 Generic (CC BY-NC-ND 2.0)许可](https://creativecommons.org/licenses/by-nc-nd/2.0/)，代码示例遵循[Apache 2.0许可](http://www.apache.org/licenses/LICENSE-2.0)。版权归谷歌所有，2018。