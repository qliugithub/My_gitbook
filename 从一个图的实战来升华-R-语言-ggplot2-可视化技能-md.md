---
title: 从一个图的实战来升华 R 语言 ggplot2 可视化技能.md
date: 2019-08-18 00:39:06
categories: R 可视化
tags: 可视化
---

## 首先我是这是我日常逛 **twitter** 看到的，然后我又是一个搬运工，
![](https://upload-images.jianshu.io/upload_images/6223615-0610c655bb4f2b40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **放最前面的 `链接来源` ：**
  -  `twitter 链接：` [https://twitter.com/FournierJohanie/status/1161454327296339968](https://twitter.com/FournierJohanie/status/1161454327296339968)
  -  `代码链接：` [‘bullet graph’ ou graphique à puce](https://johaniefournier.com/2019/08/13/tyt2019w33-bullet-graph-ou-graphique-a-puce/)

## 涉及的函数
- `readr::read_csv`
- `dplyr::glimpse`：适合用来查看函数的类信息，**没接触过**
- `mutate() + filter() + select()`：简洁方便创建新的数据
- `geom_bar()`：柱状图
- `geom_segment`：自由化画直线条，想画哪里画哪里，这里用来填充图中的蓝色柱子
- `geom_errorbar()`：添加误差线，指定 y 值头到尾即可，这里用来绘制柱子上面的那一条横线
- `geom_point()`：点图，这里用来添加柱子上面的点
- `coord_flip()`：将 xy 轴互换
- `scale_y_continuous(breaks = seq(0, 80, 10), limits = c(0, 80))`：定义 y 轴刻度尺内容（ 即图中展示的 x 轴数字）
- `expand_limits()`：单向扩展阈值，也可以用来指定 xy 轴的范围，这里给后面要添加箭头留白（最上面的那部分空白就是这个函数引起的）
- `theme()`：画板控制，各种参数，具体见正文或者谷歌搜索关键字 **ggplot2 theme**
- `labs()`：可以用来修改坐标轴以及标题、副标题等文本信息，这里通过 **`" "`** 将内容设置为空
- `geom_curve`：作用与 `geom_segment()` 相似，只是前者用来画直线，而这里用来绘制曲线，参数 `arrow` 为箭头
- `annotate()`：可以自由在画板上面添加文本注释信息，想在哪里添加就在哪里添加

## 成品图
![](https://upload-images.jianshu.io/upload_images/6223615-e587175f52513a03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **好了接下来就是我复制粘贴的时间了。**

### 一、读取数据
- 作者这里为了方便我们大家重现或者说学习此代码（我猜的哈），就把数据放在 `github` 上面。
- 使用 `readr` 包的 `read_csv()` 函数读取文件 
```{r}
emperors <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-08-13/emperors.csv")

# 运行后会出现的结果，我们可以很清楚的看到每一列的信息
Parsed with column specification:
cols(
  index = col_double(),
  name = col_character(),
  name_full = col_character(),
  birth = col_date(format = ""),
  death = col_date(format = ""),
  birth_cty = col_character(),
  birth_prv = col_character(),
  rise = col_character(),
  reign_start = col_date(format = ""),
  reign_end = col_date(format = ""),
  cause = col_character(),
  killer = col_character(),
  dynasty = col_character(),
  era = col_character(),
  notes = col_character(),
  verif_who = col_character()
)

dplyr::glimpse(emperors, width = 100)
# 说实话，刚开始看到这个我是不知道在做啥的，然后谷歌，发现这是让我们更直观的去了解我们自己的数据
# width 参数控制输出总字符的宽度
# 可以很清楚的看到有 16 列（变量） 和 68 行
Observations: 68
Variables: 16
$ index       <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, ...
$ name        <chr> "Augustus", "Tiberius", "Caligula", "Claudius", "Nero", "Galba", "Otho", "V...
$ name_full   <chr> "IMPERATOR CAESAR DIVI FILIVS AVGVSTVS", "TIBERIVS CAESAR DIVI AVGVSTI FILI...
$ birth       <date> 0062-09-23, 0041-11-16, 0012-08-31, 0009-08-01, 0037-12-15, 0002-12-24, 00...
$ death       <date> 0014-08-19, 0037-03-16, 0041-01-24, 0054-10-13, 0068-06-09, 0069-01-15, 00...
$ birth_cty   <chr> "Rome", "Rome", "Antitum", "Lugdunum", "Antitum", "Terracina", "Terentinum"...
$ birth_prv   <chr> "Italia", "Italia", "Italia", "Gallia Lugdunensis", "Italia", "Italia", "It...
$ rise        <chr> "Birthright", "Birthright", "Birthright", "Birthright", "Birthright", "Seiz...
$ reign_start <date> 0026-01-16, 0014-09-18, 0037-03-18, 0041-01-25, 0054-10-13, 0068-06-08, 00...
$ reign_end   <date> 0014-08-19, 0037-03-16, 0041-01-24, 0054-10-13, 0068-06-09, 0069-01-15, 00...
$ cause       <chr> "Assassination", "Assassination", "Assassination", "Assassination", "Suicid...
$ killer      <chr> "Wife", "Other Emperor", "Senate", "Wife", "Senate", "Other Emperor", "Othe...
$ dynasty     <chr> "Julio-Claudian", "Julio-Claudian", "Julio-Claudian", "Julio-Claudian", "Ju...
$ era         <chr> "Principate", "Principate", "Principate", "Principate", "Principate", "Prin...
$ notes       <chr> "birth, reign.start are BCE. Assign negative for correct ISO 8601 dates. Ca...
$ verif_who   <chr> "Reddit user zonination", "Reddit user zonination", "Reddit user zonination...

# 再看看 head 函数， 可以看到横列是反的
head(emperors)
# A tibble: 6 x 16
  index name  name_full birth      death      birth_cty birth_prv rise  reign_start reign_end  cause killer dynasty
  <dbl> <chr> <chr>     <date>     <date>     <chr>     <chr>     <chr> <date>      <date>     <chr> <chr>  <chr>  
1     1 Augu~ IMPERATO~ 0062-09-23 0014-08-19 Rome      Italia    Birt~ 0026-01-16  0014-08-19 Assa~ Wife   Julio-~
2     2 Tibe~ TIBERIVS~ 0041-11-16 0037-03-16 Rome      Italia    Birt~ 0014-09-18  0037-03-16 Assa~ Other~ Julio-~
3     3 Cali~ GAIVS IV~ 0012-08-31 0041-01-24 Antitum   Italia    Birt~ 0037-03-18  0041-01-24 Assa~ Senate Julio-~
4     4 Clau~ TIBERIVS~ 0009-08-01 0054-10-13 Lugdunum  Gallia L~ Birt~ 0041-01-25  0054-10-13 Assa~ Wife   Julio-~
5     5 Nero  NERO CLA~ 0037-12-15 0068-06-09 Antitum   Italia    Birt~ 0054-10-13  0068-06-09 Suic~ Senate Julio-~
6     6 Galba SERVIVS ~ 0002-12-24 0069-01-15 Terracina Italia    Seiz~ 0068-06-08  0069-01-15 Assa~ Other~ Flavian
# ... with 3 more variables: era <chr>, notes <chr>, verif_who <chr>
```
### 二、加载后续所要用的包
```{r}
library(dplyr) 
library(tidyverse)
library(lubridate)  # year() 函数要用
library(ggplot2)
```
### 三、数据处理
- 插句题外话，这里的 `=` 对齐，可以用 [remedy](https://thinkr-open.github.io/remedy/) 实现，操作很骚，对于 **`代码整洁和 Markdown`** 特别方便

- 不断的利用 `mutate()` 函数创建新得变量，这里没啥解释的。
- `filter()`: 各种条件筛选
- `select()`：选择要输出的列
```{r}
data <- emperors %>%
  mutate(annee_naiss = year(birth)) %>%
  mutate(annee_mort  = year(death)) %>%
  mutate(annee_deb   = year(reign_start)) %>%
  mutate(annee_fin   = year(reign_end)) %>%
  mutate(age_mort    = abs(annee_mort - annee_naiss)) %>%
  mutate(age_deb     = abs(annee_deb - annee_naiss)) %>%
  mutate(age_fin     = abs(annee_fin - annee_naiss)) %>%
  mutate(duree       = abs(age_fin - age_deb)) %>%
  mutate(remove      = ifelse(age_deb == age_mort, 'retirer', NA)) %>%
  filter(!age_mort %in% NA, !age_deb %in% NA, !age_fin %in% NA,
         !age_mort %in% 4, !remove %in% "retirer") %>%
  select(name, age_deb, age_fin, age_mort, duree)

# 简单查看一下数据
dplyr::glimpse(data, width = 100)
Observations: 51
Variables: 5
$ name     <chr> "Augustus", "Caligula", "Claudius", "Nero", "Galba", "Vespasian", "Titus", "Do...
$ age_deb  <dbl> 36, 25, 32, 17, 66, 60, 40, 30, 66, 45, 41, 52, 40, 31, 16, 48, 10, 20, 52, 15...
$ age_fin  <dbl> 48, 29, 45, 31, 67, 70, 42, 45, 68, 64, 62, 75, 59, 39, 31, 66, 29, 22, 53, 19...
$ age_mort <dbl> 48, 29, 45, 31, 67, 70, 42, 45, 68, 64, 62, 75, 59, 39, 31, 66, 29, 22, 53, 19...
$ duree    <dbl> 12, 4, 13, 14, 1, 10, 2, 15, 2, 19, 21, 23, 19, 8, 15, 18, 19, 2, 1, 4, 13, 3,..

# 还是比较喜欢 head 来展示
head(data)
# A tibble: 6 x 5
  name      age_deb age_fin age_mort duree
  <chr>       <dbl>   <dbl>    <dbl> <dbl>
1 Augustus       36      48       48    12
2 Caligula       25      29       29     4
3 Claudius       32      45       45    13
4 Nero           17      31       31    14
5 Galba          66      67       67     1
6 Vespasian      60      70       70    10
```

### 四、画图（重头戏）
> 为了说明可视化不断修整过程，我将分开展示
#### 1、 `geom_bar()` 函数绘制 柱状图
- **1、** 就是一个简单的柱状图
> 参数说明：
> -  `stat = identity` : 绘图函数 `stat` 的参数，用来对样本进行统计，默认为 `identity`，表示一个 x 对应一个 y，即横坐标 x 在数据中对应的 y 值；同时可以是 `bin` 表示对一个 x 对应落在 x 里面的数，即统计频数，官方说明书 [geom_bar.html](https://ggplot2.tidyverse.org/reference/geom_bar.html)
>     - `stat` 函数有 `stat_bin()` 、 `stat_count()` 、`stat_density()`、`stat_bin_2d()`、`stat_bind_hex()`、`stat_density_2d()`、`stat_ellipse()`、`stat_contour()`、`stat_summary_hex()`、`stat_summary_2d()`、`stat_boxplot()`、`stat_ydensity()`、`stat_ecdf()`、 `stat_quantile()`、`stat_smooth()`、  `stat_qq()`、`stat_summary(fun.data = "mean_cl_boot")`、` stat_summary_bin(fun.y = "mean", geom = "bar")`、`stat_unique()` 等，**最重要的是还可以自定函数 `stat_function(aes(x = -3:3), n = 99, fun = dnorm, args = list(sd=0.5))`**，详情见 [ggplot2 cheat sheet](https://www.rstudio.com/resources/cheatsheets/)，有时间要好好看下每一个对应的功能。
> -  `position = stack`: 用 `Cheat sheet` 里面内容展示，一目了然。`stack` 表示堆积，`dodge` 表示分开，`fill` 表示百分比填充，`jitter` 表示散点图抖动，`nudge` 表示注释信息远离点。
> ![](https://upload-images.jianshu.io/upload_images/6223615-78e0626182d6e4c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```{r}
gg <- ggplot(data, aes(x = reorder(name, -age_mort), y = age_mort))  
gg <- gg + 
  geom_bar(stat          = "identity", position = "stack", width = 0.65, 
           fill          = "#6D7C83", alpha = 0.4)  
```
![](https://upload-images.jianshu.io/upload_images/6223615-004a2e74bd10b668.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2、`geom_segment()` 函数来画直线
- **2、** 通过 `geom_segment()` 函数来画直线, 因为这里是表示柱子，可以理解为线条，主要要指定 x 对应的那一根柱子，然后再指定纵向即 Y 轴的起始 `age_deb` 和 终止 `age_fin` 坐标即可，`size = 2.3` 制定柱子的宽度，不宜太大，然后涂上颜色。
> 至于这个函数式做什么的呢？就是你提供一个四边形的区间，你就可以画出一个四边形，详情可以参考 **`杜雨`**老师写的一篇超级棒的素材 [ggplot2 都有哪些使用不多但是却异常强大的图层函数](https://zhuanlan.zhihu.com/p/37072513)
> 引用其中一句话 `geom_segment 通常用于制作直线段图，路径图、放射线图等，思路也很简单，只需要指定每一条线段的起点坐标、终点坐标即可。即分别制定 x,y,xend,yend`
>   -  后面会涉及一个函数 `geom_curve()` 用来画弧线，用官方 [geom_segment](https://ggplot2.tidyverse.org/reference/geom_segment.html) 一张图说明
> ![](https://upload-images.jianshu.io/upload_images/6223615-b07974999cf30cb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```{r}
gg <- gg + 
  geom_segment(aes(y    = age_deb, x = name,
                   yend = age_fin, xend = name),
               color    = "#175676", size = 2.3, alpha = 0.8)  # size = 2.3 制定柱子的宽度，不宜太大
```
![](https://upload-images.jianshu.io/upload_images/6223615-949981762eb495c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3、`geom_errorbar()` 加上误差线
-  **3、** `geom_errorbar()` 加上误差线，这里起始并不是真正的加上误差线，给我感觉就是在每个柱子最上放划一道横线。用上面的函数 `geom_segment()` 也可以做到。
```{r}
gg <- gg + 
  geom_errorbar(aes(y    = age_mort, x = name, 
                    ymin = age_mort, ymax = age_mort),
                color    = "black", width = 0.85)  
```
![](https://upload-images.jianshu.io/upload_images/6223615-0b3e329ad34d5f6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4、 `geom_point()` 函数加上散点图
-  **4、**  通过 `geom_point()` 函数加上散点图（ `仔细看柱子顶端中间多了一个点` ），并且通过 ` coord_flip() ` 函数将 X 和 Y 轴进行交换
```{r}
gg <- gg + geom_point(aes(name, age_mort), 
                      colour = "black", size = 1)  # 我将原文 0.75 改成了 1 ，这样点的效果好点
gg <- gg + coord_flip()  
```
![](https://upload-images.jianshu.io/upload_images/6223615-b41db029f17dd9cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
0)

#### 5、 `scale_y_continuous()` 函数调整 y 坐标轴刻度尺的内容和范围
-  **5、**  `scale_y_continuous()` 函数调整 y 坐标轴刻度尺的内容和范围，注意因为我们是通过函数 `coord_flip()` 将 xy 轴交换了，但是修改参数的时候，仍然要对应之前的坐标轴。
   -  `scale_y_continuous()` 函数，指定 y 轴刻度尺标签，`breaks` 展示需要的内容，`limits` 指定 y 轴的范围。
```{r}
gg <- gg + scale_y_continuous(breaks = seq(0, 80, 10), 
                              limits = c(0, 80)) 

seq(0, 80, 10)
[1]  0 10 20 30 40 50 60 70 80
```
![](https://upload-images.jianshu.io/upload_images/6223615-d37a0ea6766579a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6、`expand_limits()` 函数可以用来单向扩展阈值
-  **6、**  `expand_limits()` 函数可以用来单向扩展阈值，也可以用来指定 xy 轴的范围，这里给后面要添加箭头留白。
```{r}
gg <- gg + expand_limits(x = c(0, 56)) # 没咋理解这一步加不加有啥区别，先操作，再回来解释
```
![](https://upload-images.jianshu.io/upload_images/6223615-eb291c7f12aa0fc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7、 `theme()` 函数来调整主题
-  **7、** 通过 `theme()` 函数来调整主题，决定哪些显示哪些不显示
  -  [theme 官方参数详解](https://ggplot2.tidyverse.org/reference/theme.html)
  -  [ggplot2主题设置](http://www.rpubs.com/lihaoyi/156592)
  -  `theme()` 函数参数详解，来源：[ggplot2 学习笔记系列之主题（theme）设置](https://www.jianshu.com/p/3ec860e87832)
  -  **这个应该是中文相对详细的：** [ggplot2 作图详解 7（完）：主题（theme）设置](https://blog.csdn.net/u014801157/article/details/24372531)

  -  搜关键词 **ggplot2 theme** ， 一堆供你参考
![](https://upload-images.jianshu.io/upload_images/6223615-474d2388c21dba36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 记住几个主要的吧
```{markdown}
参数	设置内容	继承自
line	所有线属性	 
rect	所有矩形区域属性	 
text	所有文本相关属性	 
title	所有标题属性	 
axis.title	坐标轴标题	
axis.title.x	x 轴属性	axis.title
axis.title.y	y 轴属性	axis.title
axis.text	坐标轴刻度标签属性	 	 
axis.ticks	坐标轴刻度线	 
axis.ticks.length	刻度线长度	 
axis.ticks.margin	刻度线和刻度标签之间的间距	 
axis.line	坐标轴线		 
legend.background	图例背景	
legend.margin	图例边界	 
legend.key	图例符号	 
legend.key.size	图例符号大小	 
legend.key.height	图例符号高度	 
legend.key.width	图例符号宽度	 
legend.text	图例文字标签	 
legend.text.align	图例文字标签对齐方式	0 为左齐，1 为右齐
legend.title	图例标题	text
legend.title.align	图例标题对齐方式	 
legend.position	图例位置	left, right, bottom, top, 两数字向量
legend.direction	图例排列方向	"horizontal" or "vertical"
legend.justification	居中方式	center 或两数字向量
legend.box	多图例的排列方式	"horizontal" or "vertical"
legend.box.just	多图例居中方式	 
panel.background	绘图区背景	
panel.border	绘图区边框	
panel.margin	分面绘图区之间的边距	 
panel.grid	绘图区网格线	
panel.grid.major	主网格线	 
panel.grid.minor	次网格线	  	 
plot.background	整个图形的背景	 
plot.title	图形标题	 
plot.margin	图形边距	top, right, bottom, left
strip.background	分面标签背景
strip.text	分面标签文本		 
```
```{r}
gg <- gg +  theme(panel.border       = element_blank(), # 绘图区边框
                  panel.background   = element_blank(), # 绘图区背景，这里会变成纯白，没有灰色背景
                  plot.background    = element_blank(), # 整个图形的背景
                  panel.grid.major.x = element_line(size = 0.2,linetype = "dotted", color = "#6D7C83"), # 垂直 x 轴的主网格线的类型、粗细、以及颜色
                  panel.grid.major.y = element_blank(), # 同上，只不过这里选择为空，不显示这条线，其实有点多余，本身就没有这里
                  panel.grid.minor   = element_blank(), # 次网格线
                  axis.line.x        = element_blank(), # 坐标轴线 x
                  axis.line.y        = element_blank(), # 坐标轴线 y
                  axis.ticks.y       = element_blank(), # 坐标轴 y 刻度线
                  axis.ticks.x       = element_blank())  # 坐标轴 x 刻度线
```
![](https://upload-images.jianshu.io/upload_images/6223615-364e897867d89b68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 8、 `labs()` 函数来修改所有标签内容
-  **8、** 通过 `labs()` 函数来修改所有标签内容
```{r}
gg <- gg + labs(title  = " ",
              subtitle = "",
              y        = " ",
              x        = " ")
```
![](https://upload-images.jianshu.io/upload_images/6223615-6749fbb02d8195ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 9、 `theme()` 函数来修改所有坐标轴刻度尺内容
-  **9、** 通过 `theme()` 函数来修改所有坐标轴刻度尺内容，可以看到 xy 轴刻度尺的字体和颜色都改变了。
```{r}
gg <- gg + theme(plot.title    = element_blank(),
               plot.subtitle   = element_blank(),
               axis.title.y    = element_blank(),
               axis.title.x    = element_blank(),
               axis.text.y     = element_text(hjust = 1, vjust = 0.5, size = 12,
                                              color = "#6D7C83", face = "bold"),
               axis.text.x     = element_text(hjust = 0.5, vjust = 0, size = 12,
                                              color = "#6D7C83", face = "bold")
```
![](https://upload-images.jianshu.io/upload_images/6223615-7614fdaf45b69388.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 10、`geom_curve()` 函数用来画**弧线**，来实现图中的**箭头标志**
-  **10、** 通过 `geom_curve()` 函数用来画**弧线**，来实现图中的**箭头标志**，这里我们可以看出来前面的函数 `expand_limits()` 是为了给这里的箭头留白。
```{r}
# 制备箭头的坐标
arrows <- tibble(    
  x1 = c(50, 16, 53.5, 53.5, 53.5),    
  x2 = c(49, 15,   51,   51,   51),    
  y1 = c(35, 70,    5,   25,   40),    
  y2 = c(22, 61,    0,   13,   19)  
) 

# 添加箭头
gg <- gg +    
  geom_curve(data                  = arrows, 
                         aes(x     = x1, y = y1, xend = x2, yend = y2),
                         arrow     = arrow(length = unit(0.1, "inch")),
                         size      = 0.3, 
                         color     = "#6D7C83",
                         curvature = -0.3) 
```
![](https://upload-images.jianshu.io/upload_images/6223615-62591c232d4acaf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 11、`annotate()` 函数在图上加一点文本注释
-  **11、** 使用 `annotate()` 函数在图上加一点文本注释，从这里可以发现作者全文的颜色都是统一的。
```{r}
gg <- gg + 
  annotate(geom  = "text", x = 50, y = 35, label = "Le plus jeune à\ndevenir Empereur", 
           color = "#6D7C83", size=3, hjust = 0, vjust = 0.5, fontface = "bold")
```
![](https://upload-images.jianshu.io/upload_images/6223615-027a2e0f780f0ecf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 12、`annotate()` 函数在图上其它几处加一点文本注释
-  **12、**   使用 `annotate()` 函数在图上其它几处加文本注释
```{r}
gg <- gg +
  annotate(geom          = "text", x = 18, y = 70, label = "Son reigne\na pris fin\navant\nson décès", 
           color         = "#6D7C83", size = 3, hjust = 0.5, vjust = 0.5, fontface = "bold")  

gg <- gg + 
  annotate(geom          = "text", x = 54,y = 5, label = "Naissance", 
           color         = "#6D7C83", size = 3, hjust = 0.5,vjust = 0.5, fontface = "bold")  

gg <- gg + annotate(geom = "text", x = 55, y = 25, label = "Début du\nreigne",
                  color  = "#6D7C83", size = 3, hjust = 0.5,vjust = 0.8, fontface = "bold")

gg <- gg + annotate(geom = "text", x = 54,y = 40, label = "Décès",
                  color  = "#6D7C83", size = 3, hjust = 0.5,vjust = 0.5, fontface = "bold")
```
![](https://upload-images.jianshu.io/upload_images/6223615-e587175f52513a03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 我的膝盖现在还疼，为啥？我跪着学完的。