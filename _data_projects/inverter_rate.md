---
layout: post
title: 逆变器转换效率
description: another project
---

记录一些做关于逆变器数据可视化分析内容。实现目的，通过绘制效率曲线找出相对表现较差的设备，并尝试分析原因。

## 逆变器转换效率曲线说明

逆变器转换效率的特点：

1.  光伏逆变器实际输出功率不同时，转换效率不同
2.  实际输出功率相同，输入电压不同时，转换效率也不同
3.  理论上，每条效率曲线都有一个最大值，即为逆变器的最大转换效率

以下述电站为例进行相关分析。

``` r
station<-"哈尔滨聚合"
month<-7
source("~/R/forecast/app/set/deviceRateCo.R")
```

### 不同年份相同月份效率曲线的差异

通过不同年份相同月份数据可见。第一年数据不稳定，之后转换效率逐年下降。

``` r
g<-ggplot(data,aes(x=load, y=convert,color=device_name)) +
    geom_smooth(method='loess',se = FALSE)+
    ylim(max(data$convert),1)+
    xlim(0,1)+
    theme(legend.position = "bottom")+
    theme(text = element_text(family = "Noto Sans CJK SC")) +
    facet_grid(device_mode_name~month)+
    xlab("负载率") +
    ylab("转换效率")
htmlwidgets::saveWidget(plotly::ggplotly(g),"month_rate.html")
```
<iframe src="{{site.baseurl}}/assets/inverter_rate/month_rate.html" height="400px" width="100%"></iframe>

### 每台设备在不同年份的负荷率分布

目前我绘制的这个图根本看不出来啥。

``` r
g<-ggplot(data,aes(x=load, y=convert,color=device_name)) +
  geom_histogram(aes(y=..density..),fill='white',bins = 10)+
  geom_smooth(method='loess',se = FALSE,color='black')+
  ylim(max(data$convert),2)+
  theme(legend.position = "none")+
  facet_grid(month~device_name)+
  theme(text = element_text(family = "Noto Sans CJK SC")) +
  xlab("负载率") +
  ylab("转换效率")
htmlwidgets::saveWidget(plotly::ggplotly(g),"device_month_rate.html")
```
<iframe src="{{site.baseurl}}/assets/inverter_rate/device_month_rate.html" height="600px" width="100%"></iframe>

## 找到一个统计量来描述逆变器效率低

### 平均值

-   平均数最低的设备是 \#4

``` r
describeBy(data$convert, data$device_name, mat = TRUE) %>%arrange(mean)
```

    ##      item group1 vars     n      mean        sd    median   trimmed        mad
    ## X16     6     #4    1 17601 0.5155850 0.4331975 0.7626667 0.5244108 0.29175423
    ## X17     7     #5    1 17663 0.5184041 0.4353317 0.7777778 0.5279170 0.26983816
    ## X110   10     #8    1 17661 0.5284096 0.4383963 0.8027682 0.5403190 0.23342562
    ## X15     5     #3    1 17524 0.5355032 0.4397539 0.8216553 0.5491341 0.20638619
    ## X14     4     #2    1 17544 0.5522570 0.4462958 0.8678211 0.5695664 0.14402622
    ## X13     3    #11    1 17626 0.5523307 0.4492916 0.8693139 0.5689238 0.15054525
    ## X18     8     #6    1 17369 0.5524820 0.4462132 0.8542274 0.5687468 0.17719771
    ## X19     9     #7    1 17259 0.5738465 0.4517454 0.9027027 0.5955849 0.10346948
    ## X111   11     #9    1 13971 0.7233512 0.3991133 0.9609063 0.7815419 0.02691477
    ## X11     1     #1    1 12348 0.7977831 0.3219225 0.9534521 0.8753891 0.02662938
    ## X12     2    #10    1 13800       Inf       NaN 0.9106579 0.6516487 0.08669594
    ##      min      max    range       skew   kurtosis          se
    ## X16    0 1.016872 1.016872 -0.2527015 -1.8269972 0.003265256
    ## X17    0 1.020748 1.020748 -0.2598152 -1.8302722 0.003275579
    ## X110   0 1.015019 1.015019 -0.2950794 -1.8199139 0.003298824
    ## X15    0 1.025937 1.025937 -0.3209283 -1.8101507 0.003321950
    ## X14    0 1.014673 1.014673 -0.3729492 -1.7939270 0.003369446
    ## X13    0 1.111345 1.111345 -0.3580171 -1.8038661 0.003384164
    ## X18    0 1.030749 1.030749 -0.3596062 -1.7883394 0.003385751
    ## X19    0 1.070070 1.070070 -0.4311367 -1.7532475 0.003438634
    ## X111   0 1.040828 1.040828 -1.1823039 -0.4995617 0.003376622
    ## X11    0 1.054598 1.054598 -1.8770280  1.8239724 0.002897031
    ## X12    0      Inf      Inf        NaN        NaN         NaN

### 中位数

-   中位数最低的设备是 \#4

``` r
describeBy(data$convert, data$device_name, mat = TRUE) %>%arrange(median)
```

    ##      item group1 vars     n      mean        sd    median   trimmed        mad
    ## X16     6     #4    1 17601 0.5155850 0.4331975 0.7626667 0.5244108 0.29175423
    ## X17     7     #5    1 17663 0.5184041 0.4353317 0.7777778 0.5279170 0.26983816
    ## X110   10     #8    1 17661 0.5284096 0.4383963 0.8027682 0.5403190 0.23342562
    ## X15     5     #3    1 17524 0.5355032 0.4397539 0.8216553 0.5491341 0.20638619
    ## X18     8     #6    1 17369 0.5524820 0.4462132 0.8542274 0.5687468 0.17719771
    ## X14     4     #2    1 17544 0.5522570 0.4462958 0.8678211 0.5695664 0.14402622
    ## X13     3    #11    1 17626 0.5523307 0.4492916 0.8693139 0.5689238 0.15054525
    ## X19     9     #7    1 17259 0.5738465 0.4517454 0.9027027 0.5955849 0.10346948
    ## X12     2    #10    1 13800       Inf       NaN 0.9106579 0.6516487 0.08669594
    ## X11     1     #1    1 12348 0.7977831 0.3219225 0.9534521 0.8753891 0.02662938
    ## X111   11     #9    1 13971 0.7233512 0.3991133 0.9609063 0.7815419 0.02691477
    ##      min      max    range       skew   kurtosis          se
    ## X16    0 1.016872 1.016872 -0.2527015 -1.8269972 0.003265256
    ## X17    0 1.020748 1.020748 -0.2598152 -1.8302722 0.003275579
    ## X110   0 1.015019 1.015019 -0.2950794 -1.8199139 0.003298824
    ## X15    0 1.025937 1.025937 -0.3209283 -1.8101507 0.003321950
    ## X18    0 1.030749 1.030749 -0.3596062 -1.7883394 0.003385751
    ## X14    0 1.014673 1.014673 -0.3729492 -1.7939270 0.003369446
    ## X13    0 1.111345 1.111345 -0.3580171 -1.8038661 0.003384164
    ## X19    0 1.070070 1.070070 -0.4311367 -1.7532475 0.003438634
    ## X12    0      Inf      Inf        NaN        NaN         NaN
    ## X11    0 1.054598 1.054598 -1.8770280  1.8239724 0.002897031
    ## X111   0 1.040828 1.040828 -1.1823039 -0.4995617 0.003376622

### 中国效率

测试电站所在北京区域为三类资源区域，中国效率公式计算为：
η=0.02\*η5%+0.05\*η10%+0.09\*η20%+0.2\*η30%+0.34\*η50%+0.28\*η75%+0.02\*η100%

``` r
#定义不同负载区间
b_load<-c(0,0.075,0.15,0.25,0.40,0.62,0.875,1)
#计算不同负载区间，转换效率的平均值
mean_rate<-data%>%filter(!is.infinite(convert))%>%group_by(device_name)%>%summarise(mean_rate=matrixStats::binMeans(convert,x=load,bx=b_load),.groups = "keep")
mean_rate<-aggregate(mean_rate~device_name,mean_rate,c)
#定义不同负载区间转换效率的权重
w_load<-c(0.02,0.05,0.09,0.20,0.34,0.28,0.02)
rate<-as.data.frame(colSums(t(mean_rate[,-1])*w_load)%>%cbind(mean_rate[,1]))
names(rate)<-c('rate','device_name')
arrange(rate)
```

    ##                 rate device_name
    ## 1  0.951642239679568          #1
    ## 2  0.938219250961868         #10
    ## 3  0.940321210380126         #11
    ## 4  0.935304182096692          #2
    ## 5  0.925760623561969          #3
    ## 6  0.914183350723838          #4
    ## 7  0.916847104490992          #5
    ## 8  0.940235363345565          #6
    ## 9  0.948407495609555          #7
    ## 10 0.922638442747333          #8
    ## 11 0.960128131776301          #9

``` r
min_avg<-rate%>%arrange(rate)
```

-   按照中国效率公式计算，效率最低的设备是 \#4

### 拟合曲线的效率

``` r
res<-data%>%filter(!(is.na(convert)|is.infinite(convert)|is.na(load)|is.infinite(load)))%>%group_by(device_name)%>%mutate(fit=fitted(loess(convert~load)))%>%select(device_name,load,fit)

#定义不同负载区间
b_load<-c(0,0.075,0.15,0.25,0.40,0.62,0.875,1)
#计算不同负载区间，转换效率的平均值
mean_rate<-res%>%group_by(device_name)%>%summarise(mean_rate=matrixStats::binMeans(fit,x=load,bx=b_load),.groups = "keep")
mean_rate<-aggregate(mean_rate~device_name,mean_rate,c)
#定义不同负载区间转换效率的权重
w_load<-c(0.02,0.05,0.09,0.20,0.34,0.28,0.02)
rate<-as.data.frame(colSums(t(mean_rate[,-1])*w_load)%>%cbind(mean_rate[,1]))
names(rate)<-c('rate','device_name')
arrange(rate)
```

    ##                 rate device_name
    ## 1  0.961103056890467          #1
    ## 2  0.943453800751061         #10
    ## 3  0.944533188936648         #11
    ## 4  0.938890144921392          #2
    ## 5  0.931398596853765          #3
    ## 6  0.920226464608453          #4
    ## 7  0.922471371612847          #5
    ## 8  0.944350781968247          #6
    ## 9  0.950446685385407          #7
    ## 10 0.928142593120974          #8
    ## 11 0.969918844206735          #9

``` r
min_avg<-rate%>%arrange(rate)
```

-   按照拟合曲线，效率最低的设备是 \#4

通过平均值、中位数、行业中国效率、拟合曲线效率结算结果表现最差的设备是一致的。
