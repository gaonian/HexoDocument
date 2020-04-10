---
title : iOS图片移到assets问题记录
categories: iOS
---



之前项目中的图片都是以文件夹的形式存放的。最近因为一些事情，决定把图片移到image.assets里。

最开始实际就是在assets里放的，但是发现在xcode10的时候因为这个问题有崩溃，所以就换回到了文件夹的形式



说一下换完之后遇到的问题，在打包平台上偶然发现包比之前大了二十兆左右。



由于通过asset管理图片，最后会把图片资源打包成assets.car文件，这个打包文件较大，所以在打包平台上看着包体积增大了，因为打包平台是打包的通用包。

实际上通过assets存放的图片最后下载的时候只会根据机型下载对应 2x、3x的图片，所以实际的单个机型的下载包体积会变的比以前小。图片越多小的越多



下图为调研的结果比较：

![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/assets_img/asset1.png)



这两张是appstore估算的大小，可以看到3.0.3实际比3.0.2小于10m左右，包体积变小也算是意外的收获

![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/assets_img/asset2.png)



![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/assets_img/asset2.png)