# [图像优化自动化实用指南](https://www.yuque.com/ysfe/ykx/imgopt)

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

<div>
    <script>
    var _hmt = _hmt || [];
    (function() {
        var hm = document.createElement("script");
        hm.src = "https://hm.baidu.com/hm.js?4f01de5cc0f84f20fea5a4202233614f";
        var s = document.getElementsByTagName("script")[0]; 
        s.parentNode.insertBefore(hm, s);
        _hmt.push(['_trackEvent', '图像优化', '概要', 1]);
    })();
    </script>
</div>