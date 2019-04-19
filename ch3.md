## [3.如何选择图像格式?](https://images.guide/#choosing-an-image-format)

正如Ilya Grigorik在其出色的[图像优化指南](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization)中所指出的，图像的“正确格式”是所期望的视觉结果和功能需求的结合。你是在处理光栅图形还是矢量图形？

![Performance](https://images.guide/images/book-images/rastervvector-large.png)

[光栅图形](https://en.wikipedia.org/wiki/Raster_graphics)通过对矩形像素网格内的每个像素的值进行编码来表示图像。 它们不是分辨率或独立缩放的。 WebP或广泛支持的格式（如JPEG或PNG）可以很好地处理这些图形，其中照片写实是必需的。 我们讨论过的Guetzli，MozJPEG和其他方法都很适用于光栅图形。

[矢量图形](https://en.wikipedia.org/wiki/Vector_graphics)使用点、线和多边形来表示图像和格式，处理使用简单的几何形状（例如logo）提供高分辨率和像SVG一样的缩放的用例更好。

错误的格式会使你付出代价。选择正确格式的逻辑流程又可能充满风险，因此尝试其他格式可以在一定程度上节省费用。

Jeremy Wagner在他的图像优化演讲中谈到了在评估格式时值得考虑的[权衡点](http://jlwagner.net/talks/these-images/#/2/2)。