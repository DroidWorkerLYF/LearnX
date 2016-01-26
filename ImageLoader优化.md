#ImageLoader优化
###背景
公司在用的Imageloader是很早前根据[google官方教程](http://developer.android.com/intl/zh-cn/training/displaying-bitmaps/index.html)编写的，后期加入了对Gif的支持，但是比较古老，技术已经out了。
###预期
1. 提升图片的加载速度
2. 加入webp
3. 优化原有的请求网络部分的处理
4. 支持加载进度
5. 支持渐变的加载

###准备
目前先了解一下[Glide](https://github.com/bumptech/glide)，然后在动手。

To be continued