---
layout: post
title: "QtCharts 简介"
date: 2019-8-12 20:47
categories: 编程
tags: Qt QtChart
excerpt: 简述 QtCharts 模块的使用方法。
author: zhiy
mathjax: true
---

* content

{:toc}

Qt Charts 模块是 Qt 提供的二维数据可视化工具。利用 QChartView 类可以方便地添加各种类型的二维数据可视化图表，其中不同类型的图表各自用一个 QAbstractSeries 类表示。

## 折线图

首先在 QtCreator 中创建一个 Qt Widgets Application 工程，在 .pro 文件中添加如下语句：

{% highlight ruby lineno %}
QT += charts
{% endhighlight %}

然后在 mainwindow.cpp 文件中包含 QtChart 头文件：

{% highlight ruby lineno %}
#include <QtCharts/QtCharts>
{% endhighlight %}

接下来在 mainwindow 构造函数中创建折线图对象。

{% highlight ruby lineno %}
// 创建 LineSeries 对象，并添加数据
QLineSeries *series = new QLineSeries();
series->append(0, 6);
series->append(2, 4);
series->append(3, 8);
series->append(7, 4);
series->append(10, 5);
*series << QPointF(11, 1) << QPointF(13, 3) << QPointF(17, 6) << QPointF(18, 3) << QPointF(20, 2);

// 创建 QChart 对象， 并设置图例、坐标轴、标题
QChart *chart = new QChart();
chart->addSeries(series);
chart->legend()->hide();
chart->createDefaultAxes();
chart->setTitle("Simple line chart example");

// 创建 QChartView 对象， 并设置渲染、主题等
QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);

// 最后在主窗口显示图表
QMainWindow window;
window.setCentralWidget(chartView);
window.resize(400, 300);
window.show();
{% endhighlight %}
![](/images/QtCharts/linechart.png)
