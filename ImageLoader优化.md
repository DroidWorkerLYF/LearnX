#ImageLoader优化
###背景
公司在用的Imageloader是很早前根据[google官方教程](http://developer.android.com/intl/zh-cn/training/displaying-bitmaps/index.html)编写的，后期加入了对Gif的支持，但是比较古老，技术已经out了。

###准备
目前先了解一下[Glide](https://github.com/bumptech/glide)，然后在动手。看了一些分析的文章，也看到了源码，Glide虽然评价不错，不过确实代码量不小，而且东西不少，这也是支持类型多而必然的吧，然后把[Picasso](https://github.com/square/picasso)也粗略看了，这个所有类全都在一个包下，真是。。。  

关于公司项目本身，现在的问题是，对于加载的这个transformation没有考虑，现在是没有的，所以图片出现的比较生硬；速度上，还有提升空间，同时辅以webp及预加载的夹持，可以做到更上一层楼；网络这部分，会替换掉AsyncTask，同时由于项目本身的原因(网络请求这部分也急需重构，重写，现在的问题很多)，争取后边可以将网络请求直接调用封装好的lib；有种想用Rxjava来实现的想法，还需要考虑一下是否有意义。

###预期
1. 提升图片的加载速度
2. 加入webp
3. 优化原有的请求网络部分的处理
4. 支持加载进度
5. 支持渐变的加载

To be continued