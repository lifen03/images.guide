## [8.避免使用有损编解码器重新压缩图像](https://images.guide/#avoid-recompressing-images-lossy-codecs)

建议始终对原始图像进行压缩。重新压缩图像会有很严重的后果。假设你有一个JPEG，它已经被压缩，质量为60。如果你用有损编码重新压缩此图像，它看起来会更糟。每一轮额外的压缩都将引入分代丢失 —— 压缩过的图像构建会导致信息丢失。即使你在高质量的环境下重新压缩。

为了避免这个陷阱，**首先设置你愿意接受的最低质量的文件**，将从一开始就获得文件大小的最大可节省的字节数。然后你就可以避免这个陷阱，因为仅从降低质量的角度来看，任何文件大小的减少都是不好的。

重新编码一个有损耗的文件总是会得到一个较小的文件，但这并不意味着你可以从中获得你可能认为的那么高质量。

![Performance](https://images.guide/images/book-images/generational-loss-large.jpg)

> 上面，从Jon Sneyers的这段[优秀的视频](https://www.youtube.com/watch?v=w7vXJbLhTyI)和[附带的文章](http://cloudinary.com/blog/why_jpeg_is_like_a_photocopier)中，我们可以看到使用这几种格式进行重新压缩的世代损失影响。如果从社交网络中保存(已压缩)图像并重新上传(导致重新压缩)，你可能会遇到这个问题。质量损失会增加。

由于网格量化，MozJPEG（可能是偶然的）对再压缩降级具有更好的抵抗力。 它不是精确地压缩所有DCT值，而是检查+1/-1范围内的接近值，以查看类似值是否压缩为更少的字节。有损FLIF有一个类似于有损PNG的hack，在(重新)压缩之前，它可以查看数据并决定丢弃什么。它可以检测重新压缩的PNG是否有“漏洞”，以避免进一步改变数据。

**在编辑源文件时，以无损的格式(如PNG或TIFF)存储它们，以便尽可能地保持质量。**然后，你的构建工具或图像压缩服务将以最小的质量损失向用户提供你所输出的压缩版本。

<div>
    <script>
    var _hmt = _hmt || [];
    (function() {
        var hm = document.createElement("script");
        hm.src = "https://hm.baidu.com/hm.js?4f01de5cc0f84f20fea5a4202233614f&tt=ch8&key=" + Date.now();
        var s = document.getElementsByTagName("script")[0]; 
        s.parentNode.insertBefore(hm, s);
        _hmt.push(['图像优化', 'ch8.md', 'pv', '第8章', '避免使用有损编解码器重新压缩图像']);
    })();
    </script>
</div>
