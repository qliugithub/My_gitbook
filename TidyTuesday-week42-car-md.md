---
title: TidyTuesday-week42-car.md
date: 2019-10-16 15:32:01
categories: R 可视化
tags: 可视化
---

## 参考链接：
[https://github.com/gkaramanis/tidytuesday/blob/master/week-42/big-epa-cars.R](https://github.com/gkaramanis/tidytuesday/blob/master/week-42/big-epa-cars.R)
![](https://upload-images.jianshu.io/upload_images/6223615-bfa112e165dd61d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#######################################################################
## 学习到的技能（需要掌握的函数）
#######################################################################
- `pivot_longer()` 函数: 宽数据变长数据。
- `forcats包` 对于处理 `factor` 相关操作非常方便，绘图排序经常用到
- `row_number()` 函数：通用排名，并列的名次结果按先后顺序不一样，靠前出现的元素排名在前


## 画图思路
> 数据清洗就不说了，一如既往的结合 `filter() + group_by() + summarize() + ungroup() + mutate()` 等常用函数

### 数据清洗
> `提一下:` 在只看到这里数据清洗的视化并未理解为啥要产生 `lane` 这个变量，直到，看了后面可视化 `geom_image(data = city_race, aes(x = lane, y = city_mpg_median, image = car), size= .05, asp = 0.6)` 后，才发现原来是给图中的小车子标识位置，一种公路分两列，又一次跪拜。
```{r}
# 没有这些包的先自行安装。
library(tidyverse)
library(here)
library(ggimage)

big_epa_cars <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-10-15/big_epa_cars.csv")

city_race <- big_epa_cars %>%
  # pivot_longer() 函数将宽数据变为长数据，参数意思都是字面意思
  pivot_longer(cols = c("city08", "cityA08"), names_to = "city_fuel", values_to = "city_mpg") %>%
  filter(city_mpg > 0) %>% # 筛选 city_mpg 数值大于 1 的行
  group_by(make) %>% # 按照 make 列变量进行分组
  summarize(city_mpg_median = median(city_mpg)) %>% 
  # 然后分别求出 make 列对应的每一组变量的 city_mpg 的平局值，这里其实就可以运用到我们的将候选基因分 bin 啊，然后在求出每个 bin 对应的候选基因的值的平均值、最大值、最小值等等。
  ungroup() %>% # 取消分组
  mutate(make = fct_reorder(make, city_mpg_median)) %>%
  # 将 make 对应的因子即变量按照 city_map_median 重新进行排序
  # 详细见 R|数据处理|因子型数据
  # https://ask.hellobi.com/blog/R_shequ/15408

  top_n(10, city_mpg_median) %>% # 取 city_mpg_median 前十个
  arrange(city_mpg_median) %>% # 按照 city_mpg_median 大小进行重新排序
  mutate(lane = row_number() %% 2 * 2) 
  # row_number 通用排名，并列的名次结果按先后顺序不一样，靠前出现的元素排名在前

highway_race <- big_epa_cars %>%
  pivot_longer(cols = c("highway08", "highwayA08"), names_to = "highway_fuel", values_to = "highway_mpg") %>%
  filter(highway_mpg > 0) %>%
  group_by(make) %>%
  summarize(highway_mpg_median = median(highway_mpg)) %>%
  ungroup() %>%
  mutate(make = fct_reorder(make, highway_mpg_median)) %>%
  top_n(10, highway_mpg_median) %>%
  arrange(highway_mpg_median) %>%
  mutate(lane = row_number() %% 2 * 2 + 13) %>%
  group_by(highway_mpg_median) %>% 
  mutate(make = paste0(make, collapse = ", ")) 
```
## 产生的数据格式是这样的
![](https://upload-images.jianshu.io/upload_images/6223615-ad24b8ec02b5a247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 导入图中的小车 `png` 图片
- `注意：` 这里先将图片保存在自己当前环境的路径下
```{r}
# Icon by mynamepong, flaticon.com
# here() 函数上一次介绍过就不介绍了，就是当前路径
car = here::here("car.png")
```
## 绘图
> `大致思路：` 用 `geom_rect()` 绘制一个大的灰色背景幕布 → `geom_rect()` 绘制城市公路，哈哈（就是途中的黑色）→ 再绘制公路中间的虚线（白色虚线）→ 同城市公路绘制高速公路 → 通过 `geom_image()` 把小车图标加上去，哈哈（）萌死我了 → 使用 `geom_text()` 加上车子类型注释，注意其中的参数 `hjust` 参数一左一右 → 使用 `geom_segment` 加上刻度尺，这里太机智了，我的妈欸（分了左右，而不是在一条线上）→ `xlim()` 函数标注 x 轴的范围 → `labs()` 加上标题啊、小标题啊、脚注啊等等 → 主题优化
> **`总之逻辑特别清晰，一步紧跟一步，绘图代码部分，一步一步运行就好，简单粗暴易理解，真的是学习的好案例`**。（ **作者脑洞真的大，也可能是我脑子太小了** ）
```{r}
ggplot() +
  # city background 
  geom_rect(aes(xmin = -2, ymin = 0, xmax = 7, ymax = 105), fill = "grey60") +
  # city street
  geom_rect(aes(xmin = -0.75, ymin = 0, xmax = 2.75, ymax = 105), fill = "grey20") +
  # city street line
  geom_segment(aes(x = 1, y = 0, xend = 1, yend = 105), size = 2, color = "grey90", linetype = 2) +
  # city label
  geom_text(aes(x = -1.6, y = 1, label = "CITY"), size = 8, color = "grey90", alpha = 0.4, family = "IBM Plex Sans Medium", angle = 90, hjust = "left") +
  
  # highway background
  geom_rect(aes(xmin = 8, ymin = 0, xmax = 17, ymax = 105), fill = "brown4") +
  # highway 
  geom_rect(aes(xmin = 12.25, ymin = 0, xmax = 15.75, ymax = 105), fill = "grey20") +
  # highway road line
  geom_segment(aes(x = 14, y = 0, xend = 14, yend = 105), size = 2, color = "grey90", linetype = "longdash") +
  # highway label
  geom_text(aes(x = 16.6, y = 1, label = "HIGHWAY"), size = 8, color = "grey90", alpha = 0.4, family = "IBM Plex Sans Medium", angle = 90, hjust = "left") +
  
  # city cars
  geom_image(data = city_race, aes(x = lane, y = city_mpg_median, image = car), size= .05, asp = 0.6) +
  # city cars make
  geom_text(data = city_race, aes(x = 6.6, y = city_mpg_median, label = make), hjust = "right", family = "IBM Plex Sans Condensed", size = 6, color = "white") +
  geom_segment(data = city_race, aes(x = 6.7, y = city_mpg_median, xend = 7, yend = city_mpg_median), color = "white") +
  
  # highway cars
  geom_image(data = highway_race, aes(x = lane, y = highway_mpg_median, image = car), size= .05, asp = 0.6) +
  # highway cars make
  geom_text(data = highway_race, aes(x = 8.4, y = highway_mpg_median, label = make), hjust = "left", family = "IBM Plex Sans Condensed", size = 6, color = "white", check_overlap = TRUE) +
  geom_segment(data = highway_race, aes(x = 8, y = highway_mpg_median, xend = 8.3, yend = highway_mpg_median), color = "white") +
  
  # y axis between roads
  geom_text(aes(x = 7.5, y = seq(0, 100, 10)), label = seq(0, 100, 10), family = "IBM Plex Mono Light", size = 6, color = "black") +
  geom_text(aes(x = 7.5, y = seq(5, 95, 10)), label = seq(5, 95, 10), family = "IBM Plex Mono Light", size = 6, color = "grey50") +
  geom_text(aes(x = 7.5, y = -3, label = "MPG/MPGe"), family = "IBM Plex Mono Light", size = 6, color = "black") +
  
  # title and theme
  xlim(-2, 17) +
  labs(
    title = "Tesla is the leading car brand in energy efficiency",
    subtitle = "Top 10 most energy efficient brands in city and highway driving.\nRanking is based on calculated median MPG and MPGe of all\nmodels made by every car manufacturer since 1984.",
    caption = "Source: EPA | Graphic: Georgios Karamanis\nCar icon by mynamepong, flaticon.com"
  ) +
  theme_void(base_family = "IBM Plex Sans") +
  theme(
    plot.margin = margin(20, 20, 20, 20),
    plot.title = element_text(family = "IBM Plex Serif Bold", size = 28, margin = margin(10, 0, 0, 0)),
    plot.subtitle = element_text(size = 24, margin = margin(10, 0, 0, 40)),
    plot.caption = element_text(size = 12, color = "grey60", margin = margin(0, 40, 0, 0)),
    
  ) +
  ggsave(
    here::here(paste0("big-epa-cars-", format(Sys.time(), "%Y%m%d_%H%M%S"), ".png")),
    height = 19, width = 12, dpi = 320
  ) # 出图到保存一步到位
```
![](https://upload-images.jianshu.io/upload_images/6223615-886d2cb6abaaa5cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
