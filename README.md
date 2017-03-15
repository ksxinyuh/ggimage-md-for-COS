# ggimage：ggplot2中愉快地使用图片

> 作者简介：余光创，香港大学公共卫生学院，生物信息学博士生。
>
> 博客：<https://guangchuangyu.github.io>， 公众号：biobabble


## 图上嵌图片


Base plot可以在做图的时候嵌入图片，使用的是`graphics::rasterImage`：

```r
imgurl <- "http://phylopic.org/assets/images/submissions/295cd9f7-eef2-441e-ba7e-40c772ca7611.256.png"
library(EBImage)
x <- readImage(imgurl)
plot(1, type = "n", xlab = "", ylab = "", xlim = c(0, 8), ylim = c(0, 8))
rasterImage(x, 2, 2, 6, 4)
```

![](figures/raster.png)



如果我们搜索"ggplot2 image"，会找到类似于下面这样的帖子/博文：

+ [r - Inserting an image to ggplot2 - Stack Overflow](https://stackoverflow.com/questions/9917049/inserting-an-image-to-ggplot2)
+ [Add a background png image to ggplot2 | R-bloggers](https://www.r-bloggers.com/add-a-background-png-image-to-ggplot2/)

也就是说通过程序员秘笈，搜索，我们用ggplot2同样也可以做到。

这里我们需要用到`annotation_custom(rasterGrob)`来把图片加到ggplot2图形中，这和base plot是一模一样的。


```r
library(grid)
library(ggplot2)

p <- ggplot(d = data.frame(x = c(0, 8), y = c(0, 8)), aes(x, y)) + geom_blank()
p + annotation_custom(rasterGrob(x), 2, 6, 2, 4)
```



如果要使用图片来打点画一个散点图，我们就需要`for`循环，对每一个点进行操作，这显然是low level的操作，而`ggplot2`是一个高抽象的画图系统，我们希望能够使用`ggplot2`的语法。

`ggimage`就是来实现这样一个功能，它只是一个简单的包，允许我们在ggplot2中把离散性变量映射到不同的图片来画图。


![](figures/Screenshot1.png)


实现这个功能的想法已经酝酿很久了，在`ggtree`的开发中，我实现了`phylopic`函数来使用Phylopic数据库的图片注释进化树，也实现了`subview`函数在图上嵌入小图。用图片来注释进化树在进化分析上还是很常见的，特别是在一些分类学的研究中，需要把一些分类学特征在进化树上展示出来，而像我们做病毒，也经常会把一些图片放在进化树上来展示病毒的宿主信息。

`ggtree`和可视化有关的函数分两类，一类是加注释的图层，另一类是可视化操作树（比如像旋转、合并分支）。操作树的都是普通函数，而加注释的都是`geom`图层，除了`subview`和`phylopic`，这种所谓逼死处女座的存在，我早就想改成了`geom_subview`和`geom_phylopic`了。这也是为什么我要写`ggimage`的原因了。

## 安装

`ggimage`依赖于`EBImage`来读图片，这是个Bioconductor包，所以我们需要额外的动作来安装它。

#### 方法1：通过biocLite

```r
source("https://www.bioconductor.org/biocLite.R")
biocLite("ggimage")
```

### 方法2：不通过biocLite

```r
setRepositories(ind=1:2)
install.packages("ggimage")
```

方法1是按照Bioconductor的方法，或者你也可以用`setRepositories`把Bioconductor软件仓库加进来，这样`install.packages`也可以搜索到它的包。



## 实例分析

据我所知目前支持使用图片的R包有`CatterPlots`, `rphylopic`, `emoGG`,
`ggflags`这几个，都是为特定的目的而实现的，都有其特定的应用场景，这里我将用`ggimage`来演示在这些场景中的应用实例。

__`CatterPlots`__这个包只可以应用于base plot中，通过预设的几个猫图（R对象，随包载入）来画散点图。最近[RevolutionAnalytics 有博文](http://blog.revolutionanalytics.com/2017/02/catterplots-plots-with-cats.html)介绍。`ggplot2`没有相应画猫的包。我们可以使用`ggimage`来画，而且不用限制于`CatterPlots`预设的几个图形。


```r
library(ggplot2)
library(ggimage)

mytheme <- theme_minimal() +
    theme(axis.title = element_blank())
theme_set(mytheme)

x <- seq(-2*pi, 2*pi, length.out = 30)
d <- data.frame(x = x, y = sin(x))

img <- "http://www.belleamibengals.com/bengal_cat_2.png"
ggplot(d, aes(x, y)) + geom_image(image = img, size = .1)
```

![](figures/ggimage_CatterPlots.png)

`CatterPlots`实现的方式就是上面谈到的`rasterImage`内部使用了循环。__`rphylopic`__同时支持base plot和`ggplot2`，也是一样的实现方式，不过`rphylopic`内部没有使用循环，一次只能加一个图，它使用的图来自于[phylopic](http://phylopic.org/)数据库。


我们用`ggimage`同样可以使用`phylopic`图片：

```r
img <- "http://phylopic.org/assets/images/submissions/500bd7c6-71c1-4b86-8e54-55f72ad1beca.128.png"
ggplot(d, aes(x, y)) + geom_image(image = img, size = .1)
```

![](figures/ggimage_rphylopic.png)

> 图中是`翼足目`动物。


__`emoGG`__是专门来画`emoji`的，如果要画`emoji`的话，我推荐我写的`emojifont`包，在轩哥的[`showtext`基础](https://cos.name/2014/01/showtext-interesting-fonts-and-graphs/)上，把`emoji`当做普通字体一样操作，更方便。

这个包提供了`geom_emoji`图层，虽然一次可以画出散点，但因为不支持`aes`映射，而`ggimage`支持映射，下面的例子中我们做了一个简单的回归分析，如果残差<0.5则用笑脸，`>0.5`则用哭脸来表示。

```r
set.seed(123)
iris2 <- iris[sample(1:nrow(iris), 30),]
model <- lm(Petal.Length ~ Sepal.Length, data=iris2)
iris2$fitted <- predict(model)

p <- ggplot(iris2, aes(x = Sepal.Length, y = Petal.Length)) +
  geom_linerange(aes(ymin = fitted, ymax = Petal.Length),
                 colour = "purple") +
  geom_abline(intercept = model$coefficients[1],
              slope = model$coefficients[2])

baseurl <- "https://twemoji.maxcdn.com/72x72/"
emoji <- paste0(baseurl, c("1f600", "1f622"), ".png")

p + geom_image(aes(image = emoji[(abs(Petal.Length-fitted) > 0.5) + 1]))
```

![](figures/emoji_residual2.png)

如果要用`emoGG`来做的话，则需要自己切数据分两次来进行：

```r
p + geom_emoji(data=subset(iris2, (Petal.Length-fitted)<0.5), emoji="1f600") + geom_emoji(data=subset(iris2, (Petal.Length-fitted)>0.5), emoji="1f622")
```

这里我们只分两类(残差是否大于0.5)，所以需要加两次，试想我们有个分类变量，有多种可能的取值，则我们需要分多次切数据加图层，`CatterPlots`、`rphylopic`和`emoGG`都有这个问题，这也是`aes`映射之于`ggplot2`的重要和强大之处，它让我们可以在更高的抽像水平思考，

__`ggflags`__是支持`aes`映射的，只不过它只能用来画国旗而已。这里我也用`ggimage`来展示做图时加入国旗。


```r
library(rvest)
library(dplyr)

url <- "http://www.nbcolympics.com/medals"

medals <- read_html(url) %>%
    html_nodes("table") %>%
    html_table() %>% .[[1]]

library(countrycode)
library(tidyr)

medals <- medals %>%
    mutate(code = countrycode(Country, "country.name", "iso2c")) %>% gather(medal, count, Gold:Bronze) %>% filter(Total >= 10)

head(medals)
```

|Country       | Total|code |medal | count|
|:-------------|-----:|:----|:-----|-----:|
|Russia        |    33|RU   |Gold  |    13|
|United States |    28|US   |Gold  |     9|
|Norway        |    26|NO   |Gold  |    11|
|Canada        |    25|CA   |Gold  |    10|
|Netherlands   |    24|NL   |Gold  |     8|
|Germany       |    19|DE   |Gold  |     8|


首先我们从网站上爬回来2016年各个国家的奥林匹克奖牌数，画出柱状图，并在`xlab`国家名边上用`ggimage`画上国旗：

```r
baseurl <- "https://behdad.github.io/region-flags/png/"
flags <- paste0(baseurl, medals$code, ".png")
names(flags) <- medals$code

p <- ggplot(medals, aes(Country, count)) + geom_col(aes(fill=medal), width=.8)

p+geom_image(y = -2, aes(image = flags[code])) + coord_flip() + expand_limits(y=-2)  + scale_fill_manual(values = c("Gold" = "gold", "Bronze" = "#cd7f32","Silver" = "#C0C0C0"))
```

![](figures/olympics_2016.png)

## `ggimage`

前面我们介绍了`ggimage`在一些场景的应用实例，虽然有专门的包针对这些应用场景，但`ggimage`在这些领域中的表现要比大多数的包要好。但`ggimage`的使用并不限于这些。这里将展示一些有趣的例子。

#### 用R图标来画R形状

```r
x <- c(2,2,2,2,2,3,3,3.5,3.5,4)
y <- c(2,3,4,5,6,4,6,3,5,2)
d <- data.frame(x=x, y=y)

img <- system.file("img", "Rlogo.png", package="png")
ggplot(d, aes(x, y)) + geom_image(image=img, size=.1) +
  xlim(0,6) + ylim(0,7)
```

![](figures/R.png)

#### 嵌套式绘图

这里我要展示的是非常有名的bubble plot，但bubble不是圆圈，而是使用
`ggplot2`画的饼图，我先把饼图保存起来，再用`ggimage`拿来画，饼图的大小
与人口总数正相关。这个例子可以应用到很多场景中去，比如一个时间序列的曲线，你要用统计图在某些时间点上展示相关的信息，比如你要在地图上加个某些地方的相关统计信息（如果要在地图上画饼图，可以使用我写的[scatterpie](https://cran.r-project.org/package=scatterpie)包）。

```r
require(dplyr)
require(tidyr)
require(ggplot2)
require(gtable)
require(ggtree)

crime <- read.csv("http://datasets.flowingdata.com/crimeRatesByState2005.tsv", header=TRUE, sep="\t", stringsAsFactors=F)

plot_pie <- function(i) {
    df <- gather(crime[i,], type, value, murder:motor_vehicle_theft)
    ggplot(df, aes(x=1, value,fill=type)) +
        geom_col() + coord_polar(theta = 'y') +
        ggtitle(crime[i, "state"]) +
        theme_void() + theme_transparent() +
        theme(legend.position = "none",
              plot.title = element_text(size=rel(10), hjust=0.5))
}

pies <- sapply(1:nrow(crime), function(i) {
    outfile <- paste0("crime_", i, ".png")
    plot_pie(i) + ggsave(outfile, bg="transparent")
    outfile
})

radius <- sqrt(crime$population / pi)
crime$radius <- 0.2*radius/max(radius)
crime$pie <- pies

leg1 <- gtable_filter(ggplot_gtable(ggplot_build(plot_pie(1) + theme(legend.position="right"))), "guide-box")

p <- ggplot(crime, aes(murder, Robbery)) + geom_image(aes(image=pie, size=I(radius)))
subview(p, leg1, x=8.8, y=50)
```

![](figures/crime.png)

我们还可以每次只打一个州的数据，制作成动图。

```r
plot_crime <- function(i) {
    p <- ggplot(crime, aes(murder, Robbery)) + geom_blank() + geom_image(data=crime[i,], aes(image=pie, size=I(radius)))
    p <- subview(p, leg1, x=8.8, y=50)
    p
    o = paste0(i, ".png")
    ggsave(o, p, width=12, height=12)
    o
}

require(magick)
require(purrr)
order(crime$murder, decreasing=F) %>% map(plot_crime) %>% map(image_read) %>% image_join() %>% image_animate(fps=3) %>% image_write("crime.gif")
```

![](figures/crime.gif)

`ggtree`的`subview`函数可以图上嵌图，并不需要保存为图片，但对于`ggplot2`来讲，保存图片也是有好处的，因为`ggplot2`画图，点线是在数据空间上，随着我们保存图片的大小是按比例缩小或放大的，但文字是在像素空间上，和画图空间并不相关。所以当我们嵌图时缩小了画图窗口之后，字体会显得格外大，微调起来也比较繁琐，这时候保存为合适尺寸的图片，再用`ggimage`来加上去，显然就轻松得多。


#### 其它来自R社区的例子

SAS博客对`M&M`巧克力的[颜色分布做了分析](http://blogs.sas.com/content/iml/2017/02/20/proportion-of-colors-mandms.html)，通过simulation估计不同颜色的置信区间。这个[分析被翻译成R](http://rpubs.com/hrbrmstr/mms)，并产生下图：

![](figures/mm.png)

其中垂直片段|是真实值，水平片段当然就是置信空间了，而估计值用了`ggimage`来画不同颜色的巧克力。


另一个例子是[迪斯尼电影主人公名字的流行程度](https://rpubs.com/bhaskarvk/disney):

![](figures/Screenshot3.png)


`ggimage`是通用的包，所以可以被应用于不同的领域/场景中，起码可以让我们画出更好玩的图出来，后续我有时间的话，会写一个`draw_key_image`的函数，实现使用图片来当legend key的功能，也会把`ggtree::subview`和`ggtree::phylopic`这些也搬过来。


最后祝大家玩得开心！不要把图画得太有魔性哦:)

![](figures/Screenshot2.png)

> 感谢大为、太云和雪宁的校稿，特别是大为提出很多修改意见以及给出了用R画R的例子。


## References

+ <https://stackoverflow.com/questions/9917049/inserting-an-image-to-ggplot2>
+ <https://www.r-bloggers.com/add-a-background-png-image-to-ggplot2/>
+ <https://github.com/GuangchuangYu/ggimage>
+ <https://github.com/Gibbsdavidl/CatterPlots>
+ <https://github.com/sckott/rphylopic>
+ <https://github.com/baptiste/ggflags>
+ <http://blog.revolutionanalytics.com/2017/02/catterplots-plots-with-cats.html>
+ <http://blogs.sas.com/content/iml/2017/02/20/proportion-of-colors-mandms.html>
+ <http://rpubs.com/hrbrmstr/mms>
+ <https://rpubs.com/bhaskarvk/disney>
+ <https://cran.r-project.org/package=scatterpie>
