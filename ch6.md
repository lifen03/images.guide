## [6.WebP是什么?](https://images.guide/#what-is-webp)

[WebP](https://developers.google.com/speed/webp/)是来自谷歌的一种最新的图像格式，旨在提供更小的文件大小，在可接受的视觉质量下进行无损和有损压缩。它包括对alpha通道透明度和动画的支持。

在过去的一年中，WebP在有损和无损模式下的压缩性能比传统的压缩性能提高了几个百分点，并且在速度方面，算法的速度提高了两倍，减压效率提高了10％。WebP不是一个用于所有目的的工具，但是它在图像压缩社区中有一定的地位和不断增长的用户基础。让我们看看这是为什么。

![Performance](https://images.guide/images/book-images/Modern-Image16-large.jpg)

WebP: 比较文件大小和不同质量下的视觉相似度得分。

<h3 id="how-does-webp-perform"><a href="https://images.guide/#how-does-webp-perform">6.1 WebP的性能如何?</a></h3>

**有损压缩**

使用VP8或VP9视频关键帧编码变体的WebP有损文件平均被WebP团队引用为比JPEG文件小25-34%。

在低质量范围(0-50)，WebP比JPEG有很大的优势，因为它可以模糊掉难看的块状工件。中等质量的设置(- m4 - q75)是默认的平衡速度/文件大小。在较高的范围(80-99)，WebP的优势缩小。建议在速度比质量更重要的地方使用WebP。

**无损压缩**

[WebP无损文件比PNG文件小26%](https://developers.google.com/speed/webp/docs/webp_lossless_alpha_study)。与PNG相比，无损加载时间减少了3%。也就是说，您通常不希望在web上为用户提供无损的服务。无损边缘和锐边(如non-JPEG)是有区别的。无损的WebP可能更适合存档内容。

**透明度**

WebP有一个无损的8位透明通道，仅比PNG多22%的字节。它还支持有损RGB透明性，这是WebP独有的特性。

**元数据**

WebP文件格式支持EXIF照片元数据和XMP数字文档元数据。它还包含ICC颜色配置文件。

WebP提供了更好的压缩，但代价是CPU更密集。早在2013年,WebP的压缩速度比JPEG慢了大约10×，但现在可以忽略不计(一些图片可能慢2倍)。对于作为构建的一部分进行处理的静态图像，这应该不是一个大问题。动态生成的映像可能会造成可感知的CPU开销，您需要对其进行评估。

> 注意：WebP有损质量设置不能与JPEG直接比较。“70%质量”的JPEG与“70%质量”的WebP图像将大不相同，因为WebP通过丢弃更多的数据来实现更小的文件大小。

<h3 id="whos-using-webp-in-production"><a href="https://images.guide/#whos-using-webp-in-production">6.2 谁在生产中使用WebP?</a></h3>

许多大公司在生产中使用WebP来降低成本和减少网页加载时间。

谷歌报告说，使用WebP比其他有损压缩方案节省30-35%，每天处理430亿张图像请求，其中26%是无损压缩。这是大量的请求和大量的节省。如果[浏览器支持](http://caniuse.com/#search=webp)更好、更广泛，节省的成本无疑会更大。谷歌也在Google Play和YouTube等网站的生产环境上使用。

Netflix、Amazon、Quora、Yahoo、Walmart、Ebay、The Guardian、Fortune和USA Today都使用WebP为支持它的浏览器压缩和提供图像。VoxMedia为Chrome用户切换到WebP，将Verge的[加载时间缩短了1-3秒](https://product.voxmedia.com/2015/8/13/9143805/performance-update-2-electric-boogaloo)。在切换到为Chrome用户提供服务时，[500px](https://iso.500px.com/500px-color-profiles-file-formats-and-you/)的图像文件大小平均减少了25%，图像质量接近或更好。

这个样本列表中指出的公司里还有不少其他公司支持WebP。

![Performance](https://images.guide/images/book-images/webp-conversion-large.jpg)

Google使用WebP：每天在YouTube，Google Play，Chrome数据保护程序和G+上WebP图像请求每天430亿次。

<h3 id="how-does-webp-encoding-work"><a href="https://images.guide/#how-does-webp-encoding-work">6.3 WebP编码是如何工作的?</a></h3>

WebP的有损编码针对与JPEG静止图像相比。WebP有损编码有三个关键阶段:

**Macro-blocking**——把图像分解为16×16(宏观)亮度像素的小块和两个8×8色像素的小块。这听起来很像JPEG进行颜色空间转换、色度通道下采样和图像细分的想法。

![Performance](https://images.guide/images/book-images/Modern-Image18-medium.png)

**Prediction**——每一个4×4宏模块的子块都有预测模型应用,有效过滤。它定义了一个块周围的两组像素——A(它上面的行)和L(它左边的列)。使用这两个编码器测试块填满4×4像素和确定哪些创建值最接近原来的块。Colt McAnlis更深入地讨论了[WebP有损模式的工作方式](https://medium.com/@duhroach/how-webp-works-lossly-mode-33bd2b1d0670)。

![Performance](https://images.guide/images/book-images/Modern-Image19-small.png)

应用离散余弦变换（DCT），其步骤类似于JPEG编码。 关键的区别在于使用[算法压缩器](https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H)和JPEG的霍夫曼。

如果您想深入了解，Google Developer上的文章[WebP Compression Techniques](https://developers.google.com/speed/webp/docs/compression)将深入探讨这一主题。

<h3 id="webp-browser-support"><a href="https://images.guide/#webp-browser-support">6.4 WebP浏览器支持</a></h3>

并不是所有的浏览器都支持WebP，但是[根据CanIUse.com的数据](http://caniuse.com/webp)，全球用户支持率约为74%。Chrome和Opera都支持它。Safari、Edge和Firefox都曾试用过，但尚未正式发布。这通常将获取WebP图像的任务不是留给用户，而留给web开发人员。稍后将对此进行详细介绍。

以下是主要的浏览器和支持信息:

+ Chrome: Chrome在23版本开始全面支持。

+ Android版Chrome:从Chrome 50开始

+ Android: 自Android 4.2开始

+ Opera: 12.1

+ Opera Mini: 所有版本

+ Firefox: 一些测试版支持

+ Edge: 一些测试版支持

+ Internet Explorer: 不支持

+ Safari: 一些测试版支持

WebP也有它的缺点。它缺乏全分辨率的颜色空间选项，不支持渐进式解码。也就是说，WebP的工具是不错的，并且支持浏览器，尽管在编写本文时仅限于Chrome和Opera，但它可能已经覆盖了足够多的用户，因此值得考虑使用一个后备工具。

<h3 id="how-do-i-convert-to-webp"><a href="https://images.guide/#how-do-i-convert-to-webp">6.5 我如何把图像转成WebP?</a></h3>

一些商业和开源的图像编辑和处理包支持WebP。一个特别有用的应用程序是XnConvert: 一个免费的、跨平台的批处理图像转换器。

> 注意：避免将低质量或平均质量的JPEG转换为WebP非常重要。 这是一个常见错误，可以使用JPEG压缩工件生成WebP图像。 这可能导致WebP效率降低，因为它必须保存图像和JPEG添加的失真，导致您在质量上损失两次。 Feed转换应用程序是可用的最佳质量源文件，最好是原始文件。

**[XnConvert](http://www.xnview.com/en/xnconvert/)**

XnConvert支持批处理图像，兼容500多种图像格式。您可以组合80多个单独的操作以多种方式转换或编辑图像。

![Performance](https://images.guide/images/book-images/Modern-Image20-large.png)

XnConvert支持批处理图像优化，允许直接从源文件转换为WebP和其他格式。除了压缩，XnConvert还可以帮助进行元数据剥离、裁剪、颜色深度定制和其他转换。xnview网站上列出的一些选项包括:

+ 元数据: 编辑

+ 变换: 旋转，裁剪，调整大小

+ 调整: 亮度，对比度，饱和度

+ 滤镜: 模糊，浮雕，锐化

+ 效果: 掩蔽，水印，渐晕

操作的结果可以导出到大约70种不同的文件格式，包括WebP。XnConvert对于Linux、Mac和Windows是免费的。强烈推荐使用XnConvert，特别是对于小型企业。

**Node modules**

[Imagemin](https://github.com/imagemin/imagemin)是一个流行的图像压缩模块，它还有一个将图像转换为WebP的附加组件([Imagemin-WebP](https://github.com/imagemin/imagemin-webp))。它同时支持有损和无损模式。

安装imagemin和imagemin-webp运行:

```Bash
> npm install --save imagemin imagemin-webp
```

然后，我们可以在两个模块中都require()，并在项目目录中的任何图像(例如JPEG)上运行它们。下面我们使用有损编码与WebP编码器质量为60:

```javascript
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

与JPEG类似，可以在输出中注意压缩伪像。 评估什么样的质量设置对您自己的图像有意义。 Imagemin-webp还可用于通过``` lossless: true```选项来编码无损质量的WebP图像（支持24位颜色和完全透明）：

```javascript
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

由Sindre Sorhus在imagemin-webp上构建也可以使用[Gulp WebP插件](https://github.com/sindresorhus/gulp-webp)或[WebPack WebP加载器](https://www.npmjs.com/package/webp-loader)。Gulp插件接受imagemin插件的所有选项:

```javascript
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

```javascript
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

XNConvert支持批处理图像压缩，但是如果您希望避免使用应用程序或构建系统，bash和图像优化二进制文件可以使事情变得简单很多。

你可以使用[cwebp](https://developers.google.com/speed/webp/docs/cwebp)批量把你的图像转换成WebP:

```Bash
find ./ -type f -name '*.jpg' -exec cwebp -q 70 {} -o {}.webp \;
```

或者可以使用MozJPEG 的 [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive) 批量优化你的原图像：

```Bash
find ./ -type f -name '*.jpg' -exec jpeg-recompress {} {} \;
```

并使用[svgo](https://github.com/svg/svgo)(我们将在后面介绍)来压缩这些SVG:

```Bash
find ./ -type f -name '*.svg' -exec svgo {} \;
```

Jeremy Wagner有一篇关于[使用Bash进行图像优化](https://jeremywagner.me/blog/bulk-image-optimization-in-bash)的更全面的文章，还有一篇关于[并行](https://jeremywagner.me/blog/faster-bulk-image-optimization-in-bash)完成这项工作的文章，值得一读。

**其他WebP图像处理和编辑应用包括:**

+ Leptonica — 一个完整的开源图像处理和分析应用程序网站。

+ Sketch支持直接输到WebP

    - GIMP — 免费的，开源的Photoshop替代品,图像编辑器。

    - ImageMagick - 创建、组合、转换或编辑位图图像。免费的。命令行应用程序。

    - Pixelmator - Mac的商业图像编辑器。

    - Photoshop WebP插件-免费。图像从谷歌导入和导出。
    
**Android:**您可以使用Android Studio将现有的BMP、JPG、PNG或静态GIF图像转换为WebP格式。有关更多信息，请参见[使用Android Studio创建WebP图像](https://developer.android.com/studio/write/convert-webp.html)。

<h3 id="how-do-i-view-webp-on-my-os"><a href="https://images.guide/#how-do-i-view-webp-on-my-os">6.6 我如何查看我的操作系统上的WebP图像?</a></h3>

虽然你可以拖拽WebP图片到基于flash的浏览器(Chrome, Opera, Brave)来预览它们，但你也可以使用Mac或Windows的附加组件直接从你的操作系统中预览它们。

几年前，[Facebook尝试使用WebP](https://www.cnet.com/news/facebook-tries-googles-webp-image-format-users-squawk/)，发现那些试图右键单击照片并将其保存到磁盘的用户注意到，由于WebP格式的照片，它们不会显示在浏览器之外。这里有三个关键问题:

+ “另存为”，但无法在本地查看WebP文件。这是固定的Chrome注册自己为“.webp”处理程序。

+ “另存为”，然后将图片附加到电子邮件中，与没有Chrome的人分享。Facebook解决了这个问题，它在用户界面中引入了一个明显的“下载”按钮，并在用户请求下载时返回一个JPEG。

+ 右击 -> 复制URL -> 在web上共享URL。这是通过[内容类型协商](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)解决的。

这些问题对您的用户来说可能很少出现，但是顺便提一下，这是一个关于社交可共享性的有趣说明。值得庆幸的是，现在有一些实用程序可以在不同的操作系统上查看和使用WebP。

在Mac上，试试[WebP的Quick Look插件](https://github.com/Nyx0uf/qlImageSize)(qlImageSize)。它工作得很好:

![Performance](https://images.guide/images/book-images/Modern-Image22-large.jpg)

在Windows上，您还可以下载[WebP编解码包](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/WebpCodecSetup.exe)，允许在文件管理器和Windows照片查看器中预览WebP图像。

<h3 id="how-do-i-serve-webp"><a href="https://images.guide/#how-do-i-serve-webp">6.7 我如何提供WebP?</a></h3>

没有WebP支持的浏览器最终可能根本不显示图像，这是不理想的。为了避免这种情况，我们可以使用一些策略来基于浏览器支持有条件地提供WebP。

![Performance](https://images.guide/images/book-images/play-format-webp-large.jpg)

Chrome DevTools网络面板的“Type”列下，有条件地为基于Blink的浏览器提供WebP文件的高亮显示。

![Performance](https://images.guide/images/book-images/play-format-type-large.jpg)

当Play store向Blink提供WebP时，它又回到了Firefox等浏览器的jpeg模式。

以下是从服务器向用户提供WebP图像的一些选项：

**使用.htaccess提供WebP副本**

下面是当服务器上存在匹配的.webp版本的JPEG/PNG文件时，如何使用.htaccess文件向支持的浏览器提供WebP文件。

Vincent Orback推荐这种方法:

浏览器可以通过[Accept头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)[显式地发出WebP支持的信号](http://vincentorback.se/blog/using-webp-images-with-htaccess/)。如果您控制后端，则可以返回存在于磁盘上的图像的WebP版本，而不是JPEG或PNG等格式。但是这并不总是可能的(例如，对于静态主机，如GitHub pages或S3)，所以在考虑这个选项之前一定要检查一下。

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

> 注意: Vincent Orback有一个用于服务WebP的[htaccess配置](https://github.com/vincentorback/WebP-images-with-htaccess)示例供参考，Ilya Grigorik维护了一组[用于服务WebP的配置脚本](https://github.com/igrigorik/webp-detect)，这些脚本可能很有用。

**使用 ```<picture>``` 标签**

浏览器本身可以通过使用```<picture>```标记来选择要显示的图像格式。```<picture>```标签使用多个```<source>```元素，使用一个```<img>```标签是包含图像的实际DOM元素。浏览器循环遍历源并检索第一个匹配项。如果用户浏览器中不支持```<picture>```标记，则呈现```<div>```并使用```<img>```标记。

> 注意: 因为顺序关系要注意```<source>```的位置。不要将iamge/webp源放在遗留格式之后，而是将它们放在前面。理解它的浏览器将使用它们，不理解它的浏览器将转移到更广泛支持的框架上。如果图像的物理大小相同(不使用media属性时)，也可以将它们按文件大小排序。通常，这与将遗留放在最后的顺序相同。

下面是一些HTML示例:

```html
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

一些CDN支持自动转换到WebP，并可以使用客户端提示，[随时](http://cloudinary.com/documentation/responsive_images#automating_responsive_images_with_client_hints)提供WebP图像。检查您的CDN，看看他们的服务中是否包含WebP支持。你可能有一个简单的解决方案在等着你。

**WordPress WebP支持**

**Jetpack** —— Jetpack是一个非常流行的WordPress插件，包括一个名为[Photon](https://jetpack.com/support/photon/)的CDN图像服务。有了Photon,你能无缝支持WebP图像。Photon CDN包含在Jetpack的空闲级别，因此这是一个很好的价值和不互相干涉的实现方式。缺点是Photon会调整图像大小，在URL中放入查询字符串，并且每个图像都需要额外的DNS查找。

**缓存启用器和优化器** —— 如果你使用WordPress，至少有一个半开源的选项。开源插件[缓存启用](https://wordpress.org/plugins/cache-enabler/)程序有一个菜单复选框选项，用于缓存可用的WebP图像，当前用户的浏览器支持它们。这使得提供WebP图像变得容易。有一个缺点: 缓存启用程序需要使用名为Optimizer的姐妹程序，这需要按年付费。对于真正的开源解决方案来说，这似乎不符合其特性。

**短像素** —— 另一个与缓存启用器一起使用的选项，也是有代价的，是短像素。短像素函数很像上面描述的Optimizer。你可以每月免费优化多达100张图片。

**压缩动画gif和为什么```<video>```更好**

动画GIF格式非常有限，但它仍然得到了广泛的使用。尽管从社交网络到流行媒体网站，所有东西都大量嵌入了动画GI，但这种格式从未设计用于视频存储或动画。事实上，[GIF89a规范](https://www.w3.org/Graphics/GIF/spec-gif89a.txt)指出“GIF并不是一个动画平台”。[颜色的数量，帧数和尺寸](http://gifbrewery.tumblr.com/post/39564982268/can-you-recommend-a-good-length-of-clip-to-keep-gifs)都会影响GIF动画的大小。转换到视频可以提供了最大的节省。

![Performance](https://images.guide/images/book-images/animated-gif-large.jpg)

动画GIF与视频: 不同格式的文件大小在~同等质量下的比较。

**与MP4视频传输相同的文件通常可以减少80%或更多的文件大小。**gif不仅常常浪费大量带宽，而且加载时间更长，包含的颜色更少，而且通常提供只有部分的用户体验。你可能已经注意到上传到Twitter的动态gif在Twitter上的表现比在其他网站上要好。[Twitter上的动画gif实际上并不是gif](http://mashable.com/2014/06/20/twitter-gifs-mp4/#fiiFE85eQZqW)。为了改善用户体验和减少带宽消耗，上传到Twitter的动画gif实际上被转换成视频。类似地，在上传时[Imgur将gif图片转换成视频](https://thenextweb.com/insider/2014/10/09/imgur-begins-converting-gif-uploads-mp4-videos-new-gifv-format/)，并无声地将其转换成MP4格式。

为什么gif要大很多倍?动画GIF存储每个帧作为无损GIF图像 —— 是的，无损。我们经常经历的质量下降是由于gif被限制为256种颜色。与H.264等视频编解码器不同，这种格式通常比较大，因为它不考虑用于压缩的相邻帧。MP4视频将每个关键帧存储为有损的JPEG, JPEG会丢弃一些原始数据以实现更好的压缩。

**如果你可以切换到视频**

+ 使用[ffmpeg](https://www.ffmpeg.org/)将动画GIF(或源文件)转换为H.264 MP4。我使用来自[Rigor](http://rigor.com/blog/2015/12/optimizing-animated-gifs-with-html5-video)的一行代码:```ffmpeg -i animated.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)2:trunc(ih/2)2" video.mp4```

+ ImageOptim API还支持[将动画GIF转换为WebM/H.264视频](https://imageoptim.com/api/ungif)，[消除GIF的抖动](https://github.com/pornel/undither#examples)，这可以帮助视频编解码器压缩更多。

**如果你必须使用动画GIF**

+ 像Gifsicle这样的工具可以剥离元数据，未使用的调色板条目并最小化帧之间的变化

+ 考虑有损GIF编码器。 Gifsossy的[Gifsossy](https://github.com/pornel/giflossy)前缀带有-lossy标识，可以减少约60-65％的尺寸。 还有一个很好的基于它的工具，叫做[Gifify](https://github.com/vvo/gifify)。 对于非动画GIF，请将它们转换为PNG或WebP。

有关更多信息，请查看Rigor的[GIF书籍](https://rigor.com/wp-content/uploads/2017/03/TheBookofGIFPDF.pdf)。