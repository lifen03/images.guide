## [7.SVG优化](https://images.guide/#svg-optimization)

保持SVG精简意味着去掉任何不必要的东西。使用编辑器创建的SVG文件通常包含大量冗余信息(元数据、注释、隐藏层等)。这些内容通常可以安全地删除或转换为更小的形式，而不会影响SVG的最终呈现。

![Performance](https://images.guide/images/book-images/Modern-Image26-large.jpg)

由Jake Archibald创建的[SVGOMG](https://jakearchibald.github.io/svgomg/)是一个GUI界面，通过选择优化，通过输出标记的实时预览，您可以根据自己的喜好优化SVG

**SVG优化(SVGO)的一些一般规则:**

+ 缩小和压缩SVG文件。SVG实际上只是用XML表示的文本资源，如CSS、HTML和JavaScript，应该进行压缩和gzip，以提高性能。

+ 使用预定义的SVG形状代替路径，如```<rect>```、```<circle>```、```<ellipse>```、```<line>```和```<polygon>```。选择预定义的形状可以减少生成最终图像所需的标记，这意味着浏览器解析和光栅化的代码更少。降低SVG的复杂性意味着浏览器可以更快地显示它。

+ 如果你必须使用路径，尽量减少你的曲线和路径。尽可能地简化和组合它们。llustrator的[简化工具](http://jlwagner.net/talks/these-images/#/2/10)擅长去除多余的点，即使是复杂的艺术品，同时消除不规则。

+ 避免使用组。如果不能，试着简化它们。

+ 删除不可见的层。

+ 避免任何Photoshop或Illustrator效果。它们可以转换成大光栅图像。

+ 仔细检查任何不支持svg的嵌入式光栅图像

+ 使用工具优化您的SVG。[SVGOMG](https://jakearchibald.github.io/svgomg/)是一个非常方便的基于web的GUI，由Jake Archibald为[SVGO](https://github.com/svg/svgo)开发，我发现它非常有价值。如果使用Sketch，可以在导出文件时使用[运行SVGO的Sketch插件](https://www.sketchapp.com/extensions/plugins/svgo-compressor/)来缩小文件大小。

![Performance](https://images.guide/images/book-images/svgo-precision-large.jpg)

通过SVGO以高精度模式运行SVG源（导致尺寸改善29％）与低精度模式（38％尺寸改进）的示例。

[SVGO](https://www.sketchapp.com/extensions/plugins/svgo-compressor/)是一种基于节点的SVG优化工具。SVGO可以通过降低定义中数字的精度来减少文件大小。一个点之后的每一位数字都会增加一个字节，这就是为什么改变精度(位数)会严重影响文件大小的原因。但是，要非常非常小心地更改精度，因为它会在视觉上影响形状的外观。

![Performance](https://images.guide/images/book-images/Modern-Image28-large.jpg)

需要注意的是，虽然SVGO在前面的示例中没有过度简化路径和形状，但在很多情况下可能不是这样。观察上面火箭上的光带是如何以较低的精度扭曲的。

**在命令行使用SVGO:**

SVGO可以安装为一个[global npm CLI ](https://www.npmjs.com/package/svgo)，如果你喜欢它多于GUI:

```Bash
npm i -g svgo
```

然后可以对一个本地SVG文件运行如下操作:

```Bash
svgo input.svg -o output.svg
```

它支持您可能期望的所有选项，包括调整浮点精度:

```Bash
svgo input.svg --precision=1 -o output.svg
```

有关支持的选项的完整列表，请参见SVGO[自述文件](https://github.com/svg/svgo)。

**不要忘记压缩SVG!**

![Performance](https://images.guide/images/book-images/before-after-svgo-large.jpg)

另外，不要忘记给[SVG资源开Gzip](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)或使用Brotli提供。由于它们是基于文本的，所以可以很好地压缩(大约50%的原始源代码)。

当谷歌发布一个新徽标时，我们宣布[最小的](https://twitter.com/addyosmani/status/638753485555671040)版本只有305字节。

![Performance](https://images.guide/images/book-images/Modern-Image30-large.jpg)

您可以使用[许多高级SVG技巧](https://www.clicktorelease.com/blog/svg-google-logo-in-305-bytes/)将其进一步缩减(一直到146字节)!我只想说，无论是通过工具还是手动清理，您都可以删除更多的SVG。

**SVG精灵**

SVG可以为图标提供[强大的](https://css-tricks.com/icon-fonts-vs-svg/)功能，它提供了一种将可视化表示为精灵的方法，而不需要为图标字体提供[奇怪的](https://www.filamentgroup.com/lab/bulletproof_icon_fonts.html)转变方法。它具有比图标字体(SVG笔画属性)更细粒度的CSS样式控制、更好的定位控制(不需要修改伪元素和CSS```display```),而且SVG[更容易访问](http://www.sitepoint.com/tips-accessible-svg/)。

像[svg-sprite](https://github.com/jkphl/svg-sprite)和[IcoMoon](https://icomoon.io/)这样的工具可以自动将SVG组合成可以使用的雪碧图，通过[CSS雪碧图](https://css-tricks.com/css-sprites/)、[符号雪碧图](https://css-tricks.com/svg-use-with-external-reference-take-2)或[堆栈雪碧图](http://simurai.com/blog/2012/04/02/svg-stacks)。Una Kravetz[写了](https://una.im/svg-icons/#%F0%9F%92%81)一篇关于如何将gulp-svg-sprite用于SVG sprite工作流的实用文章，值得一看。Sara Soudein还在她的博客中介绍了[从图标字体到SVG的转换](https://www.sarasoueidan.com/blog/icon-fonts-to-svg/)。

**了解更多**

Sara Soueidan的[web优化SVG交付的技巧](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)和Chris Coyier的[实用SVG书籍](https://abookapart.com/products/practical-svg)都非常优秀。
我还发现Andreas Larsen的SVG优化文章([第1部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-1-67e8f2d4035)和[第2部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-2-6711cc15df46))很有启发。.[在Sketch中准备和导出SVG图标](https://medium.com/sketch-app-sources/preparing-and-exporting-svg-icons-in-sketch-1a3d65b239bb)也是一个不错的文章。