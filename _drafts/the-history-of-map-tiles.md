---
layout: theme:post
title: "地图瓦片的历史"
date: 2016-01-01T13:03:58+08:00
---

地图瓦片是在线地图的基础构件，类比于瓦片铺成屋顶，地图瓦片拼接成用户地图。


地图瓦片是网络地图的基础构件。将网络地图类比于房屋的话，那么地图瓦片就是砌房子的砖块。一张完整的网络地图通常很大，而用户所感兴趣的部分往往只是地图上一小块区域。为了节省数据传输，需要将完整的地图切割成小块的地图瓦片。用户在浏览地图的时候，只传输感兴趣的瓦片到浏览器进行拼接。

2005年，Google公司推出了地图服务Google Maps，自此开始，地图出现在每个人的手机、电脑、平板设备上。老百姓才开始熟悉地图、使用地图。Google Maps所采用的瓦片技术，使得地图能够出现在各种终端设备上。
现在各大在线地图所采用的是栅格瓦片，本质就是图片。地图服务器将地图渲染后切割成一块块256*256大小的png/jpg图片，用户在浏览地图时，将窗口范围内的瓦片传输到浏览器进行拼接。
我们知道地理数据有两种组织格式：栅格和矢量，为什么Google Maps选择了栅格格式作为瓦片的组织方式呢？这是出于兼容性的考虑，地图要在线传输到浏览器上显示，浏览器天生就支持图片文件。如果地图瓦片采用栅格格式，意味着不需要对浏览器做任何改动就能支持在线地图。
此后，Google Maps又推出了调用地图服务的API，使得第三方应用可以接入，扩大了地图应用范围。成了在线地图的金标准，其他各大地图厂商基本上都采用相同的技术搭建地图服务。Google使地图进入了千家万户，渗透到了生活的方方面面。Google Maps使地理信息技术从象牙塔进入工程实用阶段，大众也开始意识到地理信息的重要性。所以，当前地理位置服务相关产业的蓬勃发现，很大一部分要归功于Google的贡献。
前面说了很多关于Google Maps的贡献，现在回到地图瓦片。Google Maps采用的栅格地图瓦片技术，使得人们在各种各样的设备上可以浏览地图。然而从2005年推出Google地图，到现在已经有10年了。这10年间技术宅不断演进，网速在增快、浏览器的处理能力增强、用户需求多种多样，原先设计的应用场景已经大大改善。栅格地图瓦片，在这波大潮中渐渐显露出种种缺陷。