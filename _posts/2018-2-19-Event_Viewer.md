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
在 EVE 中，各种要素（点、线、几何体等）放置在 [TEveScene](https://root.cern/doc/master/classTEveScene.html) 对象中， 每个 Scene 由一个 [TEveViewer](https://root.cern/doc/master/classTEveViewer.html) 对象控制其绘图风格与相机方位。所以， 编写 EVE 可视化程序的思路就是：
1. 创建几何元素，设置其颜色和透明度。
2. 将若干几何元素添加到 Scene 中。
3. 在指定的窗口内创建 Viewer, 设置风格和相机方位，并添加 Scene 。

所有 EVE 元素由一个 [TEveManager](https://root.cern/doc/master/classTEveManager.html) 单例控制， 所以我们只要正确地创建对象就好了。在程序开头创建 TEveManager: `TEveManager::Create()` 。

## EVE 几何元素

### 探测器几何
探测器几何文件为 &quot;Detector.gdml&quot;。首先需要将其转化成 `.root` 文件：
```cpp
TGeoManager *geo = new TGeoManager();
geo->Import("Detector.gdml");
geo->Export("Detector.root");
```
在 EVE 程序中：
```cpp
gGeoManager = gEve->GetGeometry("Detector.root");
TGeoNode* node = gGeoManager->GetCurrentNode();
TEveGeoTopNode* det = new TEveGeoTopNode(gGeoManager,node);
gEve->AddGlobalElement(det);
```
其中 &quot;det&quot; 可以作为探测器几何元素添加到 Scene 中。
还可以为指定的 &quot;Volume&quot; 设置颜色和透明度：
```cpp
TGeoVolume* vol = gGeoManager->GetVolume("Volume Name");
TGeoMaterial* mat = gGeoManager->GetMaterial("Material Name");
mat->SetLineColor(kColor);//e.g. kBlue, kGreen
mat->SetTransparency(transparency);
```
其中透明度数值从 0~100 由完全不透明到完全透明。

### BoxSet
[TEveBoxSet](https://root.cern/doc/master/classTEveBoxSet.html) 对象包含了一系列任意尺寸和位置的盒子。共有 5 种 BoxType 可供选择， 一般选择 kBT_AABox， 定义如下：
```cpp
TEveRGBAPalette* pal = new TEveRGBAPalette(0,130);
TEveBoxSet* q = new TEveBoxSet("BoxSet");
q->SetPalette(pal);
q->Reset(TEveBoxSet::kBT_AABox, kFALSE, 64);
q->RefitPlex();
```
之后，可以向 BoxSet 中添加盒子：
```cpp
q->AddBox(x,y,z,a,b,c);
q->DigitValue(color);
```
可以利用 BoxSet 来画 hits， 用颜色深浅表示沉积能量大小。

### LineSet
[TEveStraightLineSet](https://root.cern/doc/master/classTEveStraightLineSet.html) 对象包含了一系列任意的空间线段。其定义和添加元素的命令如下：
```cpp
TEveStraightLineSet* ls = new TEveStraightLineSet();
ls->AddLine(x1,y1,z1,x2,y2,z2);
```
可以利用 LineSet 来画粒子径迹。

### PointSet
[TEvePointSet](https://root.cern/doc/master/classTEvePointSet.html) 对象包含了一系列空间点。其定义和添加元素的命令如下：
```cpp
TEvePointSet* ps = new TEvePointSet();
ps->SetNextPoint(x,y,z);
ps->SetMarkerColor(kColor);
ps->SetMarkerSize(1.5);
ps->SetMarkerStyle(kStyle);
```
利用 PointSet 可以标记一些重要的参考点。

### 添加几何元素
在定义好几何元素后，需要将其添加到 TEveScene 对象中：
```cpp
TEveScene *Scene = gEve->SpawnNewScene("Name","Title");
Scene->AddElement(det);
Scene->AddElement(q);
Scene->AddElement(ls);
Scene->AddElement(ps);
```
再将 Scene 添加到 Viewer 中，后面的工作就可以交给 TEveManager 来管理了：
```cpp
TEveViewer *Viewer = gEve->SpawnNewScene("Name","Title");
Viewer->GetGLViewer()->SetCurrentCamera(TGLViewer::kCameraOrthoXOZ);
Viewer->GetGLViewer()->SetStyle(TGLRnrCtx::kOutline);
Viewer->AddScene(Scene);
```

## 多窗口绘图
EVE 中的 Viewer 画在 [TEveWindowSlot](https://root.cern/doc/master/classTEveWindowSlot.html) 对象中。多个 slot 放在一个 [TEveWindowPack](https://root.cern/doc/master/classTEveWindowPack.html) 对象内。创建窗口：
```cpp
TEveWindowSlot* slot = TEveWindow::CreateWindowInTab(gEve->GetBrowser()->GetTabRight());
TEveWindowPack* pack = slot->MakePack();
pack->SetElementName("Multi View");
pack->SetShowTitleBar(kFALSE);
pack->SetHorizontal();
pack->NewSlot()->MakeCurrent();
pack = pack->NewSlot()->MakePack();
pack->NewSlot()->MakeCurrent();
pack = pack->NewSlot()->MakePack();
pack->SetHorizontal();
pack->NewSlot()->MakeCurrent();
pack = pack->NewSlot()->MakePack();
pack->NewSlot()->MakeCurrent();
pack->NewSlot()->MakeCurrent();
```
![](/images/EventViewer/pack.png)
每创建一个 slot，可以用 &quot;MakeCurrent()&quot; 方法将其设置为当前的 slot。然后创建 Viewer 即可将 Viewer 绑定到当前的 slot 上：
```cpp
pack->NewSlot()->MakeCurrent();
TEveViewer Viewer = gEve->SpawnNewViewer("Name","Title");
Viewer->AddScene(Scene);
```
更方便的做法是使用 `$ROOTSYS/tutorial/eve/MultiView.C`。首先将 &quot;MultiView.C&quot;拷贝到工作目录，然后修改其中的 &quot;Import&quot;函数，例如：
```cpp
void ImportGeomRPhi(TEveElement* el)
{
    // fRPhiMgr->ImportElement(el, fRPhiGeomScene);
    fRPhiGeomScene->AddElement(el);
}
```
使用方法为：
```cpp
#include "MultiView.C"
...
gMultiView = new MultiView;
gMultiView->f3DView->GetGLViewer()->SetStyle(TGLRnrCtx::kOutlien);
gMultiView->SetDepth(-10);
gMultiView->ImportGeomRPhi(det);
gMultiView->ImportEventRPhi(q);
gMultiView->ImportEventRPhi(ps);
gMultiView->ImportEventRPhi(ls);
gMultiView->ImportGeomRhoz(det);
gMultiView->ImportEventRhoz(q);
gMultiView->ImportEventRhoz(ps);
gMultiView->ImportEventRhoz(ls);
gMultiView->SetDepth(0);
```
![示例图片](/images/EventViewer/example.png)

## 总结
```cpp
#include "MultiView.C"
...
gEve = TEveManager::Create();
// 添加几何元素
...
// MultiView
...
// Other Stuff
gEve->GetViewers()->SwitchColorSet();
gEve->GetDefaultGLViewer()->SetStyle(TGLRnCtx::kOutline);
gEve->Redraw3D(kTRUE);
```
