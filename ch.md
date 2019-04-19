## [摘要](https://images.guide/#the-tldr)

**我们都应该实现图像压缩的自动化。**

图像优化应该是自动化的。它很容易被忽略，最佳实践发生变化，而且不经过构建管道的内容很容易丢失。自动化：在构建过程中使用[Imagemin](https://github.com/imagemin/imagemin)或[libvip](https://github.com/jcupitt/libvips)。这些有许多替代方案。

大多数CDN(如[Akamai](https://www.akamai.com/us/en/solutions/why-akamai/image-management.jsp))和第三方解决方案，如[Cloudinary](https://cloudinary.com/)、[imgix](https://imgix.com/)、[Fastly’s Image Optimizer](https://www.fastly.com/io/)、[Instart Logic’s SmartVision](https://www.instartlogic.com/technology/machine-learning/smartvision)或[ImageOptim API](https://imageoptim.com/api)都提供了全面的图像优化自动化解决方案。

在阅读博客文章和调整配置上花费的时间超过了服务的月费(Cloudinary有一个[免费](http://cloudinary.com/pricing)套餐)。如果您不想关心这项工作外包的成本或延迟问题，那么上面的开源选项是可靠的。像[Imageflow](https://github.com/imazen/imageflow)或[Thumbor](https://github.com/thumbor/thumbor)这样的项目启用了自我托管的替代方案。

**每个人都应该高效地压缩自己的图像。**

至少：使用[ImageOptim](https://imageoptim.com/)。它可以显着地减小图像的大小，同时保持视觉质量。也可以使用Windows和Linux的[替代方案](https://imageoptim.com/versions.html)。

更具体地说：通过[MozJPEG](https://github.com/mozilla/mozjpeg)运行JPEG(对于Web内容来说，Q=80或更低都没问题)并考虑[渐进式JPEG](http://cloudinary.com/blog/progressive_jpegs_and_green_martians)支持，PNG通过[pngquant](https://pngquant.org/)，SVG通过[SVGO](https://github.com/svg/svgo)。显式删除元数据(-带用于pngquant)以避免膨胀。代替超级巨大的GIF动画，提供[H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)视频(或为Chrome、Firefox和Opera提供[WebM](https://www.webmproject.org/))！如果你至少不能用[Giflossy](https://github.com/pornel/giflossy)。如果您要节省额外的CPU周期，需要高于web的平均质量，并且对缓慢的编码时间没有问题：试试[Guetzli](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html)。

一些浏览器通过接受请求头来宣布对图像格式的支持。这可以用于有条件地提供格式：例如，基于[Blink](https://zh.wikipedia.org/wiki/Blink)的浏览器(如Chrome)使用有损[WebP](https://developers.google.com/speed/webp/)，而对于其他浏览器则使用JPEG/PNG。

总是有很多你能做到的。存在生成和服务```srcset```断点的工具。在具有[客户端提示](https://developers.google.com/web/updates/2015/09/automating-resource-selection-with-client-hints)的基于Blink的浏览器中，资源选择可以自动化，并且如果用户选择在浏览器中“保存数据”，那么可以通过遵从[保存数据](https://developers.google.com/web/updates/2016/02/save-data)提示来减少字节数。

您可以使你的图像文件大小越小，您就可以为用户提供更好的网络体验-尤其是在移动平台上。在这篇文章中，我们将探讨通过现代压缩技术减少图像大小的方法，并将对图像质量的影响降到最低。