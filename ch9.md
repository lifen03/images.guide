## [9.减少不必要的图解码并调整成本](https://images.guide/#reduce-unnecessary-image-decode-costs)

我们之前已经为我们的用户提供了比我们用户所需的大、更高分辨率的图像。 这需要付出代价的。对于一般的移动硬件来说，解码和调整图像大小是非常昂贵的操作。如果向下发大图像并使用CSS或宽/高属性进行缩放，您可能会看到这种情况，它会影响性能。

![Performance](https://images.guide/images/book-images/image-pipeline-large.jpg)

当浏览器获取图像时，它必须将图像从原始源格式（例如JPEG）解码为内存中的位图。 通常需要调整图像的大小（例如，宽度已设置为其容器的百分比）。 解码和调整图像大小非常昂贵，并且可能会延迟显示图像所需的时间。

发送浏览器完全不需要调整大小就可以呈现的图像是最理想的。因此，利用[```srcset```和```size```](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)的优势，为您的目标屏幕大小和分辨率提供最小的图像——我们将很快介绍```srcset```。

省略图像上的```width```或```height```属性也会对性能产生负面影响。 如果没有它们，浏览器会为图像指定一个较小的占位符区域，直到有足够的字节到达它以便知道正确的尺寸。 此时，文档布局必须在称为回流的昂贵步骤中进行更新。

![Performance](https://images.guide/images/book-images/devtools-decode-large.jpg)

浏览器必须经过许多步骤才能在屏幕上绘制图像。除了获取图像外，还需要对图像进行解码并经常调整大小。这些事件可以在Chrome DevTools [Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference)中进行审计。

较大的图像还会增加内存大小成本。解码后的图像每像素约4字节。如果你并不关心，你会让浏览器崩溃;在低端设备上，启动内存交换并不需要那么多。因此，请关注您的图像解码、调整大小和内存成本。

![Performance](https://images.guide/images/book-images/image-decoding-mobile-large.jpg)

在普通和低端手机上解码图像的成本非常高。 在某些情况下，解码速度可能会慢5倍（如果不是更长）。

在构建新的[移动网络体验](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)时，Twitter通过确保为用户提供适当大小的图像来提高图像解码性能。 这需要Twitter许多图像的解码时间Timeline从大约400毫秒一直降到大约19毫秒！

![Performance](https://images.guide/images/book-images/image-decoding-large.jpg)

Chrome DevTools时间轴/性能面板突出显示Twitter Lite优化其图像管道之前和之后的图像解码时间（绿色）。

<h3 id="delivering-hidpi-with-srcset"><a href="https://images.guide/#delivering-hidpi-with-srcset">9.1 使用```srcset``提供HiDPI图像`</a></h3>

用户可以通过一系列具有高分辨率屏幕的移动和桌面设备访问您的网站。 [设备像素比率](https://stackoverflow.com/a/21413366)（DPR）（也称为“CSS像素比率”）决定了CSS如何解释设备的屏幕分辨率。 DPR由手机制造商创建，旨在提高移动屏幕的分辨率和清晰度，而不会使元素显得太小。

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

> 注意: 在这个免费的Udacity课程和Web基础上的图像指南中，您可以了解更多关于设备像素比和响应图像的信息。一个友好的提醒，[客户端提示](https://www.smashingmagazine.com/2016/01/leaner-responsive-images-client-hints/)也可以提供一种替代方案，在您的响应图像标记中指定每个可能的像素密度和格式。相反，它们将这些信息附加到HTTP请求中，以便web服务器能够选择最适合当前设备屏幕密度的信息。

<h3 id="dart-direction"><a href="https://images.guide/#art-direction">9.2 艺术方向</a></h3>

虽然向用户提供正确的解决方案很重要，但有些网站还需要从[艺术方向](http://usecases.responsiveimages.org/#art-direction)考虑这一点。 如果用户位于较小的屏幕上，您可能需要裁剪或放大并显示主题以充分利用可用空间。 尽管艺术方向超出了本文的范围，但像[Cloudinary](http://cloudinary.com/blog/automatically_art_directed_responsive_images%20)这样的服务提供的API可以尽可能地尝试自动化。

![Performance](https://images.guide/images/book-images/responsive-art-direction-large.jpg)

艺术方向：埃里克·波蒂斯（Eric Portis）汇集了一个优秀的[样本](https://ericportis.com/etc/cloudinary/)，展示了在艺术方向上将如何响应式图像。 此示例调整主要横幅在不同断点处的视觉特征，以充分利用可用空间。