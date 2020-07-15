---
title: 使用GDAL完成影像切片
date: 2020-07-15 12:12:10
tags:
---


2020年已经过去一半多了，今天才更新2020年的第一篇博客文章，惭愧！如果能找什么借口的话，过去的一段时间把精力放在了[可视化团队知识库][1]的建设上面，忽视了博客的建设。

知识库里面的文章的主要目的其一是作为团队工程经验的积累，其二是让团队新人能够快速掌握时空大数据可视化相关的知识。基于以上目的，知识库里面的文章偏向于科普向和入门向，注重于知识的广度而不是深度，放到博客上似乎有些“低技术含量”了。同时，平时在做文章选题时，经常会在知识库和博客之间犹豫徘徊，犹豫来犹豫去，最后写作的热情也就没有了😂。

不过最近我的想法又有了些变化，“低技术含量”的文章不见得让本博客掉价。每个人的技术专精的方向不一样，我认为简单的技术，读者可能不太了解，需要一些“低技术含量”的文章帮助他快速了解和入门。所以，今后本博客的文章不会太刻意去追求技术深度，只要我觉得有价值的点，都会形成文章分享出来。

今天的这篇文章就是一篇“低技术含量”的文章，主要讲使用GDAL来完成遥感影像的切片。

对了，最近我更新了[简历][2]，如果您有什么好的工作机会介绍给我，那就太感谢了🙏。

前段时间做一个可视化大屏，需要对遥感影像进行切片。由于近几年一直在研究矢量瓦片技术，对栅格切片这块的工具比较生疏。在网上搜了一下，发现合适的工具并不多。要么是诸如GeoServer这样的巨无霸，要么就是一些只支持限定数据源和限定切片规则的小工具。仔细研究了下GDAL，发现组合使用栅格格式转换工具`gdal_translate`和金字塔生成工具`gdaladdo`就能够实现支持任意数据源和任意切片规则的影像切片工作流。

开始之前，需要选定一种存储栅格瓦片的容器格式。看了一下发现选择并不多，暂时时只发现MBTiles、GeoPackage和COG（Cloud Optimized GeoTIFF）三种。MBtiles只支持EPSG:3857一种切片规则，不符合需要支持任意切片规则的要求。COG则是基于GeoTiff的一种Hack实现，技术过程过于复杂，也不予以考虑。所以能够选择的实际上只有一种：GeoPackage。

接下来使用如下命令就可以将影像转换为栅格瓦片，并输出到GeoPackage中：

```
gdal_translate -of GPKG chengdu.tif chengdu.gpkg -co TILING_SCHEME=InspireCRS84Quad
```

以上命令中，`-of`用来指定输出格式。`TILING_SCHEME`用来指定切片规则，比较常用的是`GoogleMapsCompatible`（EPSG:3857）和`InspireCRS84Quad`（EPSG:4326）,也可以自定义切片规则，具体做法请查看[文档][3]。

需要注意的是，以上命令只是根据切片规则选择一个合适的级别，生成一个单一级别的瓦片，并不是我们需要的多级别的瓦片。生成多级别的瓦片，需要使用接下来的命令：

```
gdaladdo chengdu.gpkg
```

如果GeoPackage里面有多个瓦片集，可以使用以下命令来指定为某个瓦片集生成金字塔：

```
gdaladdo GPKG:chengdu.gpkg:layername
```

从GDAL 3.2开始还支持并行，使用以下命令使用所有的CPU来加速生成：

```
gdaladdo chengdu.gpkg --config GDAL_NUM_THREADS ALL_CPUS
```

[1]: https://www.yuque.com/geoway-vision/vision
[2]: /assets/简历_GIS_研发主管_彭金金_武汉大学_18771991849.pdf
[3]: https://gdal.org/drivers/raster/gpkg.html#creation-options