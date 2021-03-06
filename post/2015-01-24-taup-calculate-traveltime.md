---
title: 用 taup_time 计算震相走时及射线信息
date: 2015-01-24
author: SeisMan
categories:
  - 地震学软件
tags:
  - 射线
  - TauP
mathjax: true
slug: taup-calculate-traveltime
---

`taup_time` 是 TauP 提供的一个工具，其主要用于计算震相理论走时，除此之外，
还提供了一些与射线有关的信息。

<!--more-->

## 基本用法

计算震源深度为 300km，震中距为 60 度时，P、S、PcP 等震相的走时信息:

    taup_time -mod prem -ph P,S,PcP,ScS,PKiKP -deg 60 -h 300

其中：

-   `-mod` 指定速度模型为 TauP 内置的 prem 模型；
-   `-ph` 后跟一系列震相名；
-   `-h` 震源深度；
-   `-deg` 后接震中距，单位为度；

在震中距比较小的情况下，用千米表示震中距更方便，可以使用 `-km` 选项。比如
计算一个浅源地震在 100 km 处常见 P 波震相的走时信息:

    taup_time -mod prem -ph ttp+ -h 2 -km 100

当然，也可以通过给定地震位置以及台站位置:

    taup_time -mod prem -ph P,PcP -h 100 -sta 20 130 -evt 40 150

## 输出信息

以第一个例子的输出为例:

    $ taup  taup_time -mod prem -ph P,S,PcP,ScS,PKiKP -deg 60 -h 100

    Model: prem
    Distance   Depth   Phase   Travel    Ray Param  Takeoff  Incident  Purist    Purist
      (deg)     (km)   Name    Time (s)  p (s/deg)   (deg)    (deg)   Distance   Name
    -----------------------------------------------------------------------------------
       60.00   100.0   P        595.39     6.819     30.16    20.83    60.00   = P
       60.00   100.0   PcP      640.10     3.999     17.14    12.04    60.00   = PcP
       60.00   100.0   PKiKP   1017.57     1.242      5.25     3.71    60.00   = PKiKP
       60.00   100.0   S       1081.25    12.786     31.42    21.59    60.00   = S
       60.00   100.0   ScS     1176.81     7.448     17.68    12.38    60.00   = ScS

输出共 9 列，从左到右分别为：

1.  震中距（度）；
2.  震源深度（km）；
3.  震相名；
4.  震相走时（秒）；
5.  射线参数（秒 / 度）；

    $$p = r \frac{\sin \theta}{v_s} = 6371 km \frac{\sin 20.83}{5.8 km/s}
      = 390.5423 s/radian$$
    $$= 390.5423 * 2\pi/360 s/deg = 6.817 s/deg$$

6.  出射角（deg）：即射线与垂直向下方向的夹角。若夹角在 0 到 90 度内，则取正值；若夹角在 90 度到 180 度内，则先减去 90，再取负值。因而，一般情况下，P 波出射者为正值，而 p 出射角为负值；
6.  出射角（deg）：即射线与垂直向下方向的夹角，取值范围为0到180度。

7.  入射角（deg）：射线在台站（位于地表）处入射时与垂直向上方向的夹角。应该是恒正的。
8.  Purist Distance 和 Purist Name 不知为何物；

**老版本的TauP对出射角的定义稍有不同：
若夹角在 0 到 90 度内，则取正值；若夹角在 90 度到 180 度内，则先减去 90，再取负值。
因而，一般情况下，P 波出射者为正值，而 p 出射角为负值。**

## 信息提取

上面介绍的输出中包含了很多信息，想要在脚本中提取出想要的信息有些麻烦。所以
`taup_time` 提供了一些选项，使得在脚本中计算走时什么的更加方便。

### time

该选项使得输出中只包含走时信息:

    $ taup  taup_time -mod prem -ph P,S,PcP -deg 60 -h 100 --time
    595.3896 640.09875 1081.2472

需要注意的是，输出中走时不是按照 `-ph` 选项中震相的顺序进行排序的，而是按照走时递增的顺序排序。比如这里，S 震相放在震相列表的第二位，而走时 1081.2472 却放在第三位。

### rayp

该选项使得输出中只包含射线参数信息:

    $ taup  taup_time -mod prem -ph P,S,PcP -deg 60 -h 100 --rayp
    6.8185554 3.9990747 12.785722

这里，射线参数的输出是按照震相进行排序的。

### time 和 rayp

这两个选项是不能一起用的，也就是说不能同时输出震相走时和射线参数信息。当两个选项同时使用时，会以后出现的选项为准。

当同时需要走时和射线参数时，只能多次执行命令了。

### rel

该选项用于计算震相的相对走时，似乎有 bug，无法正常工作。
