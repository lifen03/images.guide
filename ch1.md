## [1.介绍](https://images.guide/#introduction)

**图片仍然是造成网络膨胀的首要原因。**

图像占用了大量的互联网带宽，因为它们通常有较大的文件大小。根据[HTTP存档](http://httparchive.org/)，60%用于获取网页的数据是由JPEG、PNG和GIF组成的图像。截至2017年7月，在平均3.0MB大小的站点加载的网页内容中图像占了[1.7MB](http://httparchive.org/interesting.php#bytesperpage)。

根据Tammy Everts，将图像添加到页面或使用更多的现有图像已经被[证明](https://calendar.perfplanet.com/2014/images-are-king-an-image-optimization-checklist-for-everyone-in-your-organization/)可以提高转化率。图像不太可能消失，因此投资于有效的压缩策略以尽量减少网络膨胀变得非常重要。

![Performance](https://images.guide/images/book-images/Modern-Image00-medium.jpg)

根据2016年[Soasta/Google的研究](https://www.thinkwithgoogle.com/marketing-resources/experience-design/mobile-page-speed-load-time/)，图片是第二大转换预测指标，最佳页面的图像数量占比少于38%。

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