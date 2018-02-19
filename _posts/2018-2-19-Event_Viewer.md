---
layout: post
title: "用 EVE 可视化探测器事例"
date: 2018-2-19 17:12
categories: software
tags: ROOT
excerpt: 利用 EVE 提供的类和方法实现探测器几何，粒子径迹，沉积能量等的可视化。
author: zhiy
mathjax: true
---

* content
{:toc}

探测器事例的可视化可以直观地反映指定事例的特点（几何位置，hits, cluster 等），可以帮助我们进行物理分析以及探测器与模拟程序调试。利用 [EVE](https://pos.sissa.it/070/103/pdf) 提供的框架可以方便地实现探测器几何、粒子径迹、沉积能量、hits、cluster 等重要数据的可视化。说白了，就是让我们可以在任意窗口以任意风格和视角绘制任意几何元素。




## EVE 框架
在 EVE 中，各种要素（点、线、几何体等）放置在 [TEveScene] (https://root.cern/doc/master/classTEveScene.html) 对象中， 每个 Scene 由一个 [TEveViewer](https://root.cern/doc/master/classTEveViewer.html) 对象控制其绘图风格与相机方位。所以， 编写 EVE 可视化程序的思路就是：
1. 创建几何元素，设置其颜色和透明度。
2. 将若干几何元素添加到 Scene 中。
3. 在指定的窗口内创建 Viewer, 设置风格和相机方位，并添加 Scene 。

所有 EVE 元素由一个 [TEveManager](https://root.cern/doc/master/classTEveManager.html) 单例控制， 所以我们只要正确地创建对象就好了。在程序开头创建 TEveManager: `TEveManager::Create` 。

## EVE 几何元素

### 探测器几何
探测器几何文件为 &quot;Detector.gdml&quot;。首先需要将其转化成 `.root` 文件：
```c++
TGeoManager *geo = new TGeoManager();
geo->Import("Detector.gdml");
geo->Export("Detector.root");
```
在 EVE 程序中：
```c++
gGeoManager = gEve->GetGeometry("Detector.root");
TGeoNode* node = gGeoManager->GetCurrentNode();
TEveGeoTopNode* det = new TEveGeoTopNode(gGeoManager,node);
gEve->AddGlobalElement(det);
```
其中 &quot;det&quot; 可以作为探测器几何元素添加到 Scene 中。
还可以为指定的 &quot;Volume&quot; 设置颜色和透明度：
```c++
TGeoVolume* vol = gGeoManager->GetVolume("Volume Name");
TGeoMaterial* mat = gGeoManager->GetMaterial("Material Name");
mat->SetLineColor(kColor);//e.g. kBlue, kGreen
mat->SetTransparency(transparency);
```
其中透明度数值从 0~100 由完全不透明到完全透明。

### BoxSet
[TEveBoxSet](https://root.cern/doc/master/classTEveBoxSet.html) 对象包含了一系列任意尺寸和位置的盒子。共有 5 种 BoxType 可供选择， 一般选择 kBT_AABox， 定义如下：
```c++
```