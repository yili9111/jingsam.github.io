---
title: 《Mapbox矢量瓦片标准》深度解析
date: 2016-04-16 01:14:28
tags:
---


[Mapbox矢量瓦片标准][1]至今已发布至2.1版，各版本间只有细微的差别。标准的内容并不长，相信大部分人能够在半小时内读完。标准对矢量瓦片的编码格式做了详细地描述，大部分内容简洁易懂，但是仍然有些细节容易被误解。本文提炼出几点内容，对矢量瓦片标准进行补充说明。

## 坐标

矢量数据切片为瓦片后，其坐标从地理坐标转换为屏幕坐标，以整数形式存储。整型比浮点型所需的存储空间更小，大大降低了瓦片的传输成本。

我们通常可以将矢量瓦片看作是一种带元数据的图片，因为它和栅格瓦片一样，都是存储的屏幕坐标，但是两者之间在坐标上还是有差别的。

在栅格瓦片中，坐标(0, 0)代表瓦片最左上角的一个像素点；而在矢量瓦片中，坐标(0, 0)代表的是瓦片的左边缘与上边缘的交点。所以，如下图示例，我们可以认为栅格瓦片的坐标是在格网中心，而矢量瓦片的坐标位于格网的交点。

{% asset_img raster-vector.png %}

瓦片的`extent`并不限定瓦片内的所有坐标必须在`extent`范围内。由于瓦片buffer的存在，矢量瓦片存储的坐标可以为负值或者大于`extent`。

另外值得注意的是，瓦片的`extent`只是指明了瓦片的范围，并不是瓦片的渲染之后的大小。例如一张`extent`为4096的矢量瓦片，并不意味着最终渲染出来的是一张4096 X 4096大小的图片。渲染出的图片大小并不由`extent`决定，通常会被渲染为256 X 256或512 X 512大小的图片。假设将一张`extent`为4096的矢量瓦片渲染为256 X 256大小的图片，那么(0, 0)~(16, 16)坐标范围内的点都将被综合到图片上的(0, 0)像素点。


## `version`字段

矢量瓦片由一组命名的图层构成，每个图层必须包含一个`version`字段。标准并未规定所有的图层的`version`字段必须一致，所以我们可以认为矢量瓦片的图层可以包含不同`version`字段。按照旧版本编码的图层可以添加到新版本的图层中而互不影响，保持了对就版本的兼容性。

虽然理论上图层的`version`可以不同，实践上还是应该保持所有`version`字段一致，免得为解析带来不必要的麻烦。


## 游标

矢量瓦片存储的是相对坐标，总是存储相对于上一点在XY方向上的偏移量。游标在数据库中表示当前所指向的行，通过移动游标来遍历所有的记录。矢量瓦片构造点、线、面的过程也可以通过移动游标来模拟。游标可以看做是一只画笔，依次经过各个节点来画出几何要素，矢量瓦片记录就是画笔移动的偏移量。

对于多边形的游标有一点需要注意，即多边形闭合时，游标保持在最后一点，并不折回起点。例如下图，ABCD是一个多边形，游标在到达D点后闭合多边形，并不回到起点A。所以开始构造下一个几何要素时，坐标要相对于D点计算偏移量。之所以这么设计，是因为这样可以少存一个点，以减少瓦片存储大小。

{% asset_img cursor.png %}


## `tag`

矢量瓦片会按图层提取出所有的字段和值的唯一值，然后构建索引序列。对于要素的每个属性用一对`tag`字段表示。每对`tag`中，第一个`tag`表示字段的索引号，第二个表示值的索引号。所以，一个要素的`tag`字段数目必定为偶数，并且`tag`字段的值要小于字段和值得唯一值个数。


[1]: /2016/03/04/vector-tile-spec.html
