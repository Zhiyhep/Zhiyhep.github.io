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

## 折线图与曲线图

首先在 QtCreator 中创建一个 Qt Widgets Application 工程，在 .pro 文件中添加如下语句：
{% highlight cpp lineno %}
QT += charts
{% endhighlight %}
然后在 mainwindow.cpp 文件中包含 QtChart 头文件：
{% highlight cpp lineno %}
#include <QtCharts/QtCharts>
{% endhighlight %}
接下来在 mainwindow 构造函数中创建折线图对象。
{% highlight cpp lineno %}
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
chartView->setRenderHint(QPainter:: Antialiasing); 

// 最后在主窗口显示图表
QMainWindow window; 
window.setCentralWidget(chartView); 
window.resize(400, 300); 
window.show(); 
{% endhighlight %}
<center>
<img width = '60%' src ="/images/QtCharts/linechart.png"/>
</center>
曲线图完全类似：
{% highlight cpp %}
QSplineSeries *series = new QSplineSeries(); 
series->setName("spline"); 
series->append(0, 6); 
series->append(2, 4); 
series->append(3, 8); 
series->append(7, 4); 
series->append(10, 5); 
*series << QPointF(11, 1) << QPointF(13, 3) << QPointF(17, 6) << QPointF(18, 3) << QPointF(20, 2); 
QChart *chart = new QChart(); 
chart->legend()->hide(); 
chart->addSeries(series); 
chart->setTitle("Simple spline chart example"); 
chart->createDefaultAxes(); 
chart->axes(Qt:: Vertical).first()->setRange(0, 10); 
QChartView *chartView = new QChartView(chart); 
chartView->setRenderHint(QPainter:: Antialiasing); 
w.setCentralWidget(chartView); 
w.resize(400, 300); 
w.show(); 
{% endhighlight %}

<center>
<img width = '60%' src ="/images/QtCharts/splinechart.png"/>
</center>

## 面积图

{% highlight cpp %}
// 首先创建两个 QLineSeries 作为上下边界
QLineSeries *series0 = new QLineSeries(); 
QLineSeries *series1 = new QLineSeries();
*series0 << QPointF(1, 5) << QPointF(3, 7) << QPointF(7, 6) << QPointF(9, 7) << QPointF(12, 6)
            << QPointF(16, 7) << QPointF(18, 5);
*series1 << QPointF(1, 3) << QPointF(3, 4) << QPointF(7, 3) << QPointF(8, 2) << QPointF(12, 3)
            << QPointF(16, 4) << QPointF(18, 3);
// 创建 QAreaSeries
QAreaSeries *series = new QAreaSeries(series0, series1);
series->setName("Batman"); // 系列名称
QPen pen(0x059605);  //指定边界样式（颜色，宽度等）
pen.setWidth(3);
series->setPen(pen);
// 指定填充样式
QLinearGradient gradient(QPointF(0,0), QPointF(1,1));
gradient.setColorAt(0.0, Qt::red);
gradient.setColorAt(1.0, Qt::blue);
gradient.setCoordinateMode(QGradient::ObjectBoundingMode);
series->setBrush(gradient);
// 创建 QChart 指定标题、坐标轴、图例等
QChart *chart = new QChart();
chart->addSeries(series);
chart->setTitle("Simple areachart example");
chart->createDefaultAxes();
chart->axes(Qt::Horizontal).first()->setRange(0,20);
chart->axes(Qt::Vertical).first()->setRange(0,10);
QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);
w.setCentralWidget(chartView);
w.resize(400, 300);
w.show();
{% endhighlight %}

<center>
<img width = '60%' src ="/images/QtCharts/areachart.png"/>
</center>

## 散点图

{% highlight cpp %}
// 创建 QScatterSeries 对象
QScatterSeries *series0 = new QScatterSeries();
// 设置 series 名称， marker 形状和大小
series0->setName("scatter1");
series0->setMarkerShape(QScatterSeries::MarkerShapeCircle);
series0->setMarkerSize(15.0);

QScatterSeries *series1 = new QScatterSeries();
series1->setName("scatter2");
series1->setMarkerShape(QScatterSeries::MarkerShapeRectangle);
series1->setMarkerSize(20.0);

QScatterSeries *series2 = new QScatterSeries();
series2->setName("scatter3");
series2->setMarkerShape(QScatterSeries::MarkerShapeRectangle);
series2->setMarkerSize(30.0);

series0->append(0, 6);
series0->append(2, 4);
series0->append(3, 8);
series0->append(7, 4);
series0->append(10, 5);

*series1 << QPointF(1, 1) << QPointF(3, 3) << QPointF(7, 6) << QPointF(8, 3) << QPointF(10, 2);
*series2 << QPointF(1, 5) << QPointF(4, 6) << QPointF(6, 3) << QPointF(9, 5);

// 为 series2 创建图片 marker
QPainterPath starPath;
starPath.moveTo(28, 15);
for (int i = 1; i < 5; ++i) {
    starPath.lineTo(14 + 14 * qCos(0.8 * i * M_PI),
                    15 + 14 * qSin(0.8 * i * M_PI));
}
starPath.closeSubpath();

QImage star(30, 30, QImage::Format_ARGB32);
star.fill(Qt::transparent);

QPainter painter(&star);
painter.setRenderHint(QPainter::Antialiasing);
painter.setPen(QRgb(0xf6a625));
painter.setBrush(painter.pen().color());
painter.drawPath(starPath);

series2->setBrush(star);
series2->setPen(QColor(Qt::transparent));

QChart *chart = new QChart();
chart->addSeries(series0);
chart->addSeries(series1);
chart->addSeries(series2);
chart->setTitle("Simple scatterchart example");
chart->createDefaultAxes();
chart->setDropShadowEnabled(false);
chart->legend()->setMarkerShape(QLegend::MarkerShapeFromSeries);
QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);
w.setCentralWidget(chartView);
w.resize(400, 300);
w.show();
{% endhighlight %}

<center>
<img width = '60%' src ="/images/QtCharts/scatterchart.png"/>
</center>

## 柱状图

### 竖直柱状图
{% highlight cpp %}
// 每个 BarSet 代表柱状图中的一组数据
// 比如要展示 n 个时期的直方图， 则每个 BarSet
// 代表直方图中的一项，需要添加对应 n 个时期的 n 组数据 
QBarSet *set0 = new QBarSet("Jane");
QBarSet *set1 = new QBarSet("John");
QBarSet *set2 = new QBarSet("Axel");
QBarSet *set3 = new QBarSet("Mary");
QBarSet *set4 = new QBarSet("Samantha");

*set0 << 1 << 2 << 3 << 4 << 5 << 6;
*set1 << 5 << 0 << 0 << 4 << 0 << 7;
*set2 << 3 << 5 << 8 << 13 << 8 << 5;
*set3 << 5 << 6 << 7 << 3 << 4 << 5;
*set4 << 9 << 7 << 5 << 3 << 1 << 2;

QBarSeries *series = new QBarSeries();
series->append(set0);
series->append(set1);
series->append(set2);
series->append(set3);
series->append(set4);

QChart *chart = new QChart();
chart->addSeries(series);
chart->setTitle("Simple barchart example");
// AnimationOption 代表图片刷新时的动画
chart->setAnimationOptions(QChart::SeriesAnimations);

// 为柱状图设置文字横坐标轴和数字竖坐标轴
QStringList categories;
categories << "Jan" << "Feb" << "Mar" << "Apr" << "May" << "Jun";
QBarCategoryAxis *axisX = new QBarCategoryAxis();
axisX->append(categories);
chart->addAxis(axisX, Qt::AlignBottom);
series->attachAxis(axisX);

QValueAxis *axisY = new QValueAxis();
axisY->setRange(0,15);
chart->addAxis(axisY, Qt::AlignLeft);
series->attachAxis(axisY);

// 设置图例
chart->legend()->setVisible(true);
chart->legend()->setAlignment(Qt::AlignBottom);

QChartView *chartView = new QChartView(chart);
chartView->setRenderHint(QPainter::Antialiasing);
w.setCentralWidget(chartView);
w.resize(420, 300);
w.show();
{% endhighlight %}
<center>
<img width = '60%' src ="/images/QtCharts/barchart.png"/>
</center>

### 水平柱状图 

只需要将 *QBarSeries* 改为 *QHorizontalBarSeries* 即可。

<center>
<img width = '60%' src ="/images/QtCharts/horizontalbarchart.png"/>
</center>

### 堆叠柱状图

将 *QBarSeries* 改为 *QStackedBarSeries* 或 *QHorizontalStackedBarSeries* 即可。
<center>
<img width = '60%' src ="/images/QtCharts/stackedbarchart.png"/>
</center>

## 饼状图

{% highlight cpp%}
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    QChartView *chartView = new QChartView();
    chartView->setRenderHint(QPainter::Antialiasing);
    QChart *chart = chartView->chart();
    chart->legend()->setVisible(false);
    chart->setTitle("Nested donuts demo");
    chart->setAnimationOptions(QChart::AllAnimations);

    qreal minSize = 0.1;
    qreal maxSize = 0.9;
    int donutCount = 5; // 5 个饼图

    for(int i = 0; i < donutCount; i++){
        QPieSeries *donut = new QPieSeries();
        int sliceCount = 3 + QRandomGenerator::global()->bounded(5);
        for(int j = 0; j < sliceCount; j++){
            qreal value = 100 + QRandomGenerator::global()->bounded(100);
            QPieSlice *slice = new QPieSlice(QString("%1").arg(value),value);
            slice->setLabelVisible(true);
            slice->setLabelColor(Qt::white);
            slice->setLabelPosition(QPieSlice::LabelInsideTangential);
            connect(slice, &QPieSlice::hovered, this, &MainWindow::explodeSlice);
            donut->append(slice);
            donut->setHoleSize(minSize+i*(maxSize-minSize)/donutCount);
            donut->setPieSize(minSize + (i + 1) * (maxSize - minSize) / donutCount);
        }
        m_donuts.append(donut);
        chartView->chart()->addSeries(donut);
    }

    QGridLayout *mainLayout = new QGridLayout;
    mainLayout->addWidget(chartView,1,1);
    setLayout(mainLayout);

    updateTimer = new QTimer(this);
    connect(updateTimer, &QTimer::timeout, this, &MainWindow::updateRotation);
    updateTimer->start(1250);

    setCentralWidget(chartView);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::updateRotation()
  {
      for (int i = 0; i < m_donuts.count(); i++) {
          QPieSeries *donut = m_donuts.at(i);
          qreal phaseShift =  -50 + QRandomGenerator::global()->bounded(100);
          donut->setPieStartAngle(donut->pieStartAngle() + phaseShift);
          donut->setPieEndAngle(donut->pieEndAngle() + phaseShift);
      }
  }

void MainWindow::explodeSlice(bool exploded)
  {
      QPieSlice *slice = qobject_cast<QPieSlice *>(sender());
      if (exploded) {
          updateTimer->stop();
          qreal sliceStartAngle = slice->startAngle();
          qreal sliceEndAngle = slice->startAngle() + slice->angleSpan();

          QPieSeries *donut = slice->series();
          qreal seriesIndex = m_donuts.indexOf(donut);
          for (int i = seriesIndex + 1; i < m_donuts.count(); i++) {
              m_donuts.at(i)->setPieStartAngle(sliceEndAngle);
              m_donuts.at(i)->setPieEndAngle(360 + sliceStartAngle);
          }
      } else {
          for (int i = 0; i < m_donuts.count(); i++) {
              m_donuts.at(i)->setPieStartAngle(0);
              m_donuts.at(i)->setPieEndAngle(360);
          }
          updateTimer->start();
      }
      slice->setExploded(exploded);
  }
{% endhighlight %}

<center>
<img width = '90%' src ="/images/QtCharts/piechart.gif"/>
</center>

## 坐标轴类型

* *QValueAxis* 数值坐标轴
* *QCategoryAxis* 分类坐标轴
* *QBarCategoryAxis* 柱状图分类坐标轴
* *QDateTimeAxis* 日期坐标轴
* *QLogValueAxis* 对数坐标轴 


