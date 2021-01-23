

Ridgeline 图(脊线图)，(有时称为Joyplot)可以同时显示几个组的数值分布情况，分布可以使用直方图或密度图来表示，它们都与相同的水平尺度对齐，并略有重叠。常常被用来可视化随时间或空间变化的多个分布/直方图变化。



## 安装包ggridges

```
# Cran安装
install.packages("ggridges")
# github上安装
library(devtools)
install_github("clauswilke/ggridges")
```



## 绘图

### 基础图形

主要利用**geom_density_ridges**函数

```R
# library
library(ggridges)
library(ggplot2)

# Diamonds dataset is provided by R natively
#head(diamonds)

# basic example
ggplot(diamonds, aes(x = price, y = cut, fill = cut)) +
  geom_density_ridges(alpha = 0.5) +
  theme_ridges() + 
  theme(legend.position = "none")
```

![image-20201114153458772](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114153458772.png)

另外，有时候我们也可分面展示不同范围的脊线图

```
diamonds$label= ifelse(diamonds$price >= 10000,'great','bad')
ggplot(diamonds, aes(x = price, y = cut, fill = cut)) +
  geom_density_ridges() +
  theme_ridges() + 
  theme(legend.position = "none")+
  facet_wrap(~label)
```

![image-20201114163433512](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114163433512.png)



### 增加统计信息

这里利用到**stat_density_ridges**函数，可以在图形上增加代表统计信息的线段， 比如增加上 、下四分位数线(Q1 Q3)和中位数线 Q2 。

```
ggplot(diamonds, aes(x = price, y = cut, fill = cut)) +
  stat_density_ridges(quantile_lines = TRUE)+
  theme_ridges() + 
  theme(legend.position = "none")
  

# 当然，我们也可自定义分位数线 比如2.5% 和 60%的线
ggplot(diamonds, aes(x = price, y = cut, fill = cut)) +
  stat_density_ridges(quantile_lines = TRUE, quantiles = c(0.025, 0.600), alpha = 0.7)+
  theme_ridges() + 
  theme(legend.position = "none")
```

![image-20201114164505984](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114164505984.png)

将上述代码中fill的映射值改为**factor(stat(quantile))**，可以将不同颜色映射在不同的分区,并自定义颜色，如：

```R
ggplot(diamonds, aes(x = price, y = cut, fill = factor(stat(quantile)))) +
  stat_density_ridges(
    geom = "density_ridges_gradient", calc_ecdf = TRUE,
    quantiles = 4, quantile_lines = TRUE
  ) +theme_ridges() + 
  scale_fill_manual(
    name = "Probability", values = c("#FF0000A0", "#A0A0A0A0", "#0000FFA0",'gold'),
    labels = c("(0, 0.025]", "(0.025, 0.050]","(0.050,0.075]", "(0.075, 1]")
  )
```

![image-20201114171642384](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114171642384.png)



### 散点图显示

参考了[1](https://www.datanovia.com/en/blog/elegant-visualization-of-density-distribution-in-r-using-ridgeline/ "da")，代码中设置**jittered_points = TRUE**来实现散点的绘制，无论是在**stat_density_ridges**还是在**geom_density_ridges**。

点可以有以下几种选择方式：

- position = 'sina',在基线和山脊线之间的山脊线图中随机分布点。 这是默认选项。
- position= 'jitter', 随机抖动山脊线图中的点。 点随机上下移动和/或左右移动。
- position = 'raincloud': 在山脊线图下方创建随机抖动点的云。

```
# 增加散点图
A <- ggplot(iris, aes(x = Sepal.Length, y = Species)) +
  geom_density_ridges(jittered_points = TRUE)

# 控制点位置
# position = "raincloud"
B <- ggplot(iris, aes(x = Sepal.Length, y = Species)) +
  geom_density_ridges(
    jittered_points = TRUE, position = "raincloud",
    alpha = 0.7, scale = 0.9
  )

# position = "points_jitter"
C <- ggplot(iris, aes(x = Sepal.Length, y = Species)) +
  geom_density_ridges(
    jittered_points = TRUE, position = "points_jitter",
    alpha = 0.7, scale = 0.9
  )

# 增加边际线
D <- ggplot(iris, aes(x = Sepal.Length, y = Species)) +
  geom_density_ridges(
    jittered_points = TRUE,
    position = position_points_jitter(width = 0.05, height = 0),
    point_shape = '|', point_size = 3, point_alpha = 1, alpha = 0.7,
  )

library(patchwork)
(A + B)/(C + D)+ plot_annotation(tag_levels = 'A')
```

![image-20201114214550509](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114214550509.png)

自定义散点的样式、颜色

```R
ggplot(iris, aes(x = Sepal.Length, y = Species, fill = Species)) +
  geom_density_ridges(
    aes(point_color = Species, point_fill = Species, point_shape = Species),
    alpha = .2, point_alpha = 1, jittered_points = TRUE
  ) +
  scale_point_color_hue(l = 40) +
  scale_discrete_manual(aesthetics = "point_shape", values = c(21, 22, 23))
```

![image-20201114215333079](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114215333079.png)



### 其它(渐变色)

除了上述比较单一的色彩,还可使用此包中**geom_density_ridges_gradient**函数添加渐变色，拿Example数据为例，可以通过`?geom_density_ridges_gradient`进行查看更多的控制参数，

```
# library
library(ggridges)
library(ggplot2)
library(viridis)
library(hrbrthemes)

# Plot
ggplot(lincoln_weather, aes(x = `Mean Temperature [F]`, y = `Month`, fill = ..x..)) +
  geom_density_ridges_gradient(scale = 3, rel_min_height = 0.01) +
  scale_fill_viridis(name = "Temp. [F]", option = "C") +
  labs(title = 'Temperatures in Lincoln NE in 2016') +
  theme_ipsum() +
  theme(
    legend.position="none",
    panel.spacing = unit(0.1, "lines"),
    strip.text.x = element_text(size = 8)
  )

## geom_density_ridges_gradient参数很多，可以?详细查看
 geom_density_ridges_gradient(
  mapping = NULL,
  data = NULL,
  stat = "density_ridges",
  position = "points_sina",
  panel_scaling = TRUE,
  na.rm = TRUE,
  gradient_lwd = 0.5,
  show.legend = NA,
  inherit.aes = TRUE,
  ...
)

```

![image-20201114155034265](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114155034265.png)



### 直方图显示

除了上述的密度图，只需添加**stat="binline"**参数即可。

```
ggplot(diamonds, aes(x = price, y = cut, fill = cut)) +
  geom_density_ridges(alpha = 0.5, stat="binline", bins=20) +
  theme_ridges() + 
  theme(legend.position = "none")
```

![image-20201114154328015](https://gitee.com/kai_kai_he/PicGo/raw/master/imgage2/image-20201114154328015.png)









