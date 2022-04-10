---
title: unrealCesium
date: 2022-04-10 21:46:35
tags:
---
## 思路

更新了ureal的地形设计，使用Cesium for Unreal导入真实地形（DEM精度较低）

<!-- more -->

## 流程

1. 下载Cesium for Unreal插件 [Cesium for Unreal plugin page](https://cesium.com/unreal-marketplace/)
2. 在Epic Game导入插件
3. 开启插件选项
4. 使用Cesium World Terrain + Bing Maps Aerial 选项
5. 在CesiumGeoreference中调整经纬度
6. 将准备好的GeoTiff图通过[cesiumlab](https://www.cesiumlab.com/)生成TMS服务
7. 在Cesium World Terrain中添加CesiumTileMapServiceRasterOverlay组件并引入TMS服务
8. 在Material Layer Key选项中调整图层，以免出现错误

