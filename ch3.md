## [3.如何选择图像格式?](https://images.guide/#choosing-an-image-format)

正如Ilya Grigorik在其出色的[图像优化指南](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization)中所指出的，图像的“正确格式”是视觉所期望的结果和功能需求的结合。你是在处理光栅图形还是矢量图形呢？

![](https://images.guide/images/book-images/rastervvector-large.png)

[光栅图形](https://en.wikipedia.org/wiki/Raster_graphics)通过对矩形像素网格内的每个像素的值进行编码来表示图像。 它们不是分辨率或者独立的缩放。 WebP和广泛支持的格式（如JPEG或PNG）可以很好地处理这些图像，其中照片写实是必需的。 我们讨论过的Guetzli，MozJPEG以及一些其他方法都很适用于光栅图形。

[矢量图形](https://en.wikipedia.org/wiki/Vector_graphics)是使用点、线和多边形来表示图像，是使用简单的几何形状（例如logo）提供高分辨率和像SVG一样更好地处理缩放情况的图像格式。

错误的格式会使你付出代价。选择正确图像格式逻辑流程又可能充满风险，因此尝试其他格式可以在一定程度上节省费用。

Jeremy Wagner在他的图像优化演讲中谈到了在评估格式时值得考虑的[权衡点](http://jlwagner.net/talks/these-images/#/2/2)。

<div>
    <script>
    var _hmt = _hmt || [];
    (function() {
        var hm = document.createElement("script");
        hm.src = "https://hm.baidu.com/hm.js?4f01de5cc0f84f20fea5a4202233614f&tt=ch3&key=" + Date.now();
        var s = document.getElementsByTagName("script")[0]; 
        s.parentNode.insertBefore(hm, s);
        _hmt.push(['图像优化', 'ch3.md', 'pv', '第3章', '如何选择图像格式?']);
    })();
    </script>
</div>
