---
title: TidyTuesday_week38_heatmap
date: 2019-09-21 18:05:25
categories: R 可视化
tags: 可视化
---

# 参考链接
## https://github.com/gkaramanis/tidytuesday/tree/master/week-38 
![](https://raw.githubusercontent.com/qliugithub/PicGo/master/TidyTuesday_week38_heatmap.png)


#######################################################################
# 学习到的技能（**需要掌握的函数**）
#######################################################################

- `filter() + mutate() + group_by() + arrange() + drop_na()` 
- `mutate()` 函数中的 `case_when` 来条件判断赋值，可以使用代码冗余复杂变得浅显易懂，也很方便进行修改
- `lag()` 函数表示取向量中的前一个数据
- `lead()` 函数表示取向量中的后一个数据
  - 最后记得先排序后操作，或者在函数中指定 `order_by = xx`
  
- `geom_segment()` 函数  
- `expand` 函数表示范围扩展常数矢量，用于在数据周围添加一些填充，以确保将其放置在距轴一定距离的位置。使用便捷函数 
- `expand_scale()` 函数来生成 `expand` 参数的值。默认值是将 `连续变量` 的比例尺各扩大 **5％**，将 `离散变量` 的比例尺各扩大 **0.6** 个单位。(**翻译官方说明**)
  - `expand_scale(mult = 0, add = 0)`
  - 一句话 `add` 表示图形距离画布左右两边的距离, `mult` 表示图形里画布上下两边的距离

- `rescale()` 函数将数据范围归一化到 **[0, 1]**

- `display.brewer.all()` 就可以看到对应取色方案
- 也可以在网站 [Colormaps](https://gallantlab.github.io/colormaps.html) 进行查看

- `labs()` 要善于利用此函数来加标题等注释

- ` theme_void()` 去除所有内置主题

- `here` 包, 可以快速创建文件夹以及文件

#######################################################################
# 正文
#######################################################################
# 数据清洗
## 导入数据
```{r}
rm(list = ls())

library(tidyverse)
library(here)
library(lemon)
library(RColorBrewer)
library(scales)

park_visits <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-09-17/national_parks.csv")

# 可以看到每一列的数据类型是什么样的。
# 我们可以清楚看到 `year` 那一列是属于字符串类型，后面进行操作时候变为了 `numeric`
# cols(
#   year = col_character(),
#   gnis_id = col_character(),
#   geometry = col_character(),
#   metadata = col_character(),
#   number_of_records = col_double(),
#   parkname = col_character(),
#   region = col_character(),
#   state = col_character(),
#   unit_code = col_character(),
#   unit_name = col_character(),
#   unit_type = col_character(),
#   visitors = col_double()
# )
```


## 整理数据
- 这里充分结合来了 `filter() + mutate() + group_by() + arrange() + drop_na()` 函数一套清洗
- 还有一点需要强调的是要善于利用 `mutate()` 函数中的 `case_when` 来条件判断赋值，可以使用代码冗余复杂变得浅显易懂，也很方便进行修改
- 这里还需要注意的是利用了 `lag()` 函数，起到提取向量中的前一个值，后面会解释。
```{r}
pv_ch <- park_visits %>% 
  distinct(year, unit_name, unit_type, visitors) %>% # 去除重复行，类似base::uniq() 函数, 返回去除重复后的数据
  filter(unit_type == "National Park" & year != "Total") %>% # 筛选数据类型为 `National Park` 和年份为 `非 Total` 的行
  mutate(year = as.numeric(year)) %>% # 将 `year` 列由 `character` 变为 `numercic`
  group_by(unit_name) %>% # 按照 `unit_name` 列变量进行分组
  arrange(year, .by_group = TRUE) %>% # 将 `year` 年份按照分组分别进行排序
  mutate(pct_change = (visitors/lag(visitors) - 1) * 100) %>% # 新建一列 `pct_change` lag() 向量中的 `前一个`  值
  filter(unit_name != "Denali National Preserve") %>% # 将 `Denali National Preserve` 行去除
  drop_na() %>% # 去除空值 `NA`
  mutate( # 按照 `pct_change` 列的大小分组，三组
    pct_change = case_when(
      pct_change > 100 ~ 200,
      pct_change < -100 ~ -200,
      TRUE ~ pct_change # 表示除了上面的范围之内的范围都属于这个范畴
    )
  )
```

> 不得不插一句说明 `lag()` 和 `lead()` 函数

- 官方说明书：https://dplyr.tidyverse.org/reference/lead-lag.html 
- `lag()` 函数表示取向量中的前一个数据
- `lead()` 函数表示取向量中的后一个数据
- 最后记得先排序后操作，或者在函数中指定 `order_by = xx`

```{r}
lag(1:10, 1)
# [1] NA  1  2  3  4  5  6  7  8  9

lag(1:10, 2)
# [1] NA NA  1  2  3  4  5  6  7  8

lead(1:10, 1)
#>  [1]  2  3  4  5  6  7  8  9 10 NA

lead(1:10, 2)
#>  [1]  3  4  5  6  7  8  9 10 NA NA
```

# 可视化这部分思想很重要，
> `geom_segment()` 函数用来绘制区域，即指定四个点的位置, 详情见 [ggplot2 都有哪些使用不多但是却异常强大的图层函数](https://zhuanlan.zhihu.com/p/37072513) 一文，反正我看完是受益匪浅。   
> `expand:` 范围扩展常数矢量，用于在数据周围添加一些填充，以确保将其放置在距轴一定距离的位置。使用便捷函数 

- 借用这里 [How does ggplot scale_continuous expand argument work?](https://stackoverflow.com/questions/44170871/how-does-ggplot-scale-continuous-expand-argument-work?rq=1) 的一个解释:

```{r}
ggplot(mpg, aes(displ, hwy)) +
    geom_point() +
    scale_x_continuous(limits = c(1, 7), expand = c(0.5, 0))
# right most position will be 7 + (7-1) * 0.5 = 10

ggplot(mpg, aes(displ, hwy)) +
    geom_point() +
    scale_x_continuous(limits = c(1, 7), expand = c(0.5, 2))
# right most position will be 7 + (7-1) * 0.5  + 2 = 12
```
> `expand_scale()` 来生成 `expand` 参数的值。默认值是将 `连续变量` 的比例尺各扩大 **5％**，将 `离散变量` 的比例尺各扩大 **0.6** 个单位。(**翻译官方说明**)

- `expand_scale(mult = 0, add = 0)`

- `mult:` 范围扩展因子的向量。如果向量长度为 1，则刻度的下限和上限均会向外扩大。如果向量长度为 2，则下限由 `mult[1]` 扩展，上限由 `mult[2]` 扩展。`add = c(0,1)` 表示下边距离 x 轴零距离，距离顶端 1 倍长度。一般用于 y 轴，即 `scale_y_*`     

- `add:` 可加范围扩展常数的向量。如果向量长度为 1，则通过添加单位向外扩展刻度的上下限。如果向量长度为 2，则下限由 `add[1]` 扩展，上限由 `add[2]` 扩展。 `add = c(0,1)` 表示距离 y 轴零距离，距离右边 1 个单位。一般用于 x 轴，即 `scale_x_*`   

- 一句话 `add` 表示图形距离画布左右两边的距离, `mult` 表示图形里画布上下两边的距离

> `rescale()`   

- 将数据范围归一化到 **[0, 1]**
```{r}
rescale(1:10)
# [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667 0.7777778 0.8888889 1.0000000

rescale(1:11)
# [1] 0.0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1.0
```

> 图中对应的颜色条

- `display.brewer.all()` 就可以看到对应取色方案
- 也可以在网站 [Colormaps](https://gallantlab.github.io/colormaps.html) 进行查看

> `labs()` 要善于利用此函数来加标题等注释

> ` theme_void()` 去除所有内置主题，具体一眼见 [ggplot2 的主题模板](https://dataxujing.github.io/ggplot2%E7%9A%84%E4%B8%BB%E9%A2%98%E6%A8%A1%E6%9D%BF/)   

> 还得提一下的是 `here` 包, 可以快速创建文件夹以及文件

- [说明书](https://github.com/jennybc/here_here)

```{r}
here()
# [1] "F:/temp_desktop/2019-5"

here("one", "two", "awesome.txt")
# [1] "F:/temp_desktop/2019-5/one/two/awesome.txt"
# cat(readLines(here("one", "two", "awesome.txt")))
# 这里需要管理员权限运行才行，由于我这里没有管理员运行，所以后面那部分保存图片是运行不了的
```

```{r}
pv_ch %>%
  ggplot() +
  geom_segment(aes(x = year, xend = year,
                   y = 0, yend = 1, color = pct_change), size = 1) + # 绘制几何图形并填充颜色
  scale_x_continuous(breaks = seq(1910, 2010, 20), # 指定 x 轴标签内容
                     expand = expand_scale(add = c(5, 1))) + # 表示指定离左边的距离为 5, 离右边为 1，当为 c(0, 0) 时候会填充整个坐标轴画布
  scale_y_continuous(expand = c(0.05, 0.25)) + # 表示将 y 轴变为 ymax + (ymax - ymin) * 0.05 + 0.25
  facet_wrap(vars(unit_name), ncol = 3) + # 表示按照 `unit_name` 进行分面，变为三列。
  # `scale_colour_gradientn` N种颜色之间的平滑颜色渐变
  scale_colour_gradientn(limits = c(-200, 200), # 指定颜色填充的范围
                         colors = brewer.pal(n = 7, name = "RdYlGn"), # 选取颜色，对应颜色注释见 `display.brewer.all()`
                         values = rescale(c(-200, -100, -1, 0, 1, 100, 200)), # 对七个刻度进行归一化？，反正就是变成 [0, 1] 
                         labels = c("", "-100%", "0", "100%", "") # 对应的刻度尺进行标签注释
  ) +
  labs( # 标签注释，懒得解释
    title = "National Park Visits 1904–2016",
    subtitle = "Year-over-year percentage change",
    caption = "Source: dataisplural/data.world | Graphic: Georgios Karamanis"
  ) +
  guides(color = guide_colorbar(
    title.position = "top",
    label.position = "top",
    title = NULL,
    barwidth = 20,
    barheight = 0.5
  )) +
  theme_void(base_family = "Times New Roman") +
  theme(
    legend.position = "top",
    legend.title = element_text(size = 20, color = "grey20"),
    legend.margin = margin(0, 0, 20, 0),
    plot.background = element_rect(fill = "grey80", color = NA),
    strip.background = element_rect(fill = "grey80", color = NA),
    strip.text = element_text(family = "Times New Roman", color = "grey30",
                              hjust = 1, vjust = 1),
    plot.title = element_text(size = 28, color = "grey20", family = "Times New Roman"),
    plot.subtitle = element_text(size = 20, color = "grey20"),
    plot.caption = element_text(size = 8, color = "grey30", margin = margin(20, 0, 0, 0)),
    axis.text.x = element_text(family = "Times New Roman", size = 7, color = "grey40"),
    panel.grid.major.x = element_line(color = "grey75"),
    plot.margin = margin(20, 20, 20, 20)
  )  #本来有 +
  
  # save image
#   ggsave(
#     here::hcere("week-38", "figures", "temp", paste0("national-parks", format(Sys.time(), "%Y%m%d_%H%M%S"), # ".png")),
#     width = 18, height = 14, dpi = 320
#   ) # 没有管理员运行 R ，所以我的不行，需要自行设置
```
![](https://raw.githubusercontent.com/qliugithub/PicGo/master/TidyTuesday_week38_heatmap.png)


## 最后这个学了以后可以用来做什么呢？那就看大家自己的想法了。