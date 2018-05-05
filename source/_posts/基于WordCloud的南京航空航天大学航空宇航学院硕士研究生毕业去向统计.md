---
title: 基于WordCloud的南京航空航天大学航空宇航学院硕士研究生毕业去向统计
date: 2018-03-22 09:41:38
tags:
categories: Notes
---
# Dependecy:
+ word_cloud:

	`pip install wordcloud`

+ jieba:

	`pip install jieba`

# 统计结果
> 图片字体大小对应所占比重大小

> 样本为300位2018届毕业的硕士研究生，统计数据基于学院发布的报到证信息

> 我院2018届待毕业研究生结构为：280位博士研究生（不同年份入学，18年毕业），355位硕士研究生(基本为同一年份入学)，其余55位同学去向未定

> 统计的硕士研究生包含硕士中毕业后转博的同学，统计中不计及280位博士研究生

## 个人观点总结
+ 毕业后主要去向（按多少排序）：
  + 升学
  + 航空工业
  + 中船重工&中船工业
  + 机电
  + 能源（远景、金风）
  + 航天
  + CS
  + 中电
  + 算上直升机所，去四大主机所的同学相对很少
  + 没有去BAT的，甚至没有去华为的。少量去中兴的，IT和CS行业招人门槛在提高
  + 工作多为制造业，少量CS，还有几个有想法的去做房地产
  + 地域上，去向以江浙沪为主; 帝都对南航毕业生的吸引力远不及魔都（对应航天和航空）.
  
## 去向单位
![](/images/基于WordCloud的南京航空航天大学航空宇航学院硕士研究生毕业去向统计/Institution.png)
## 去向地域
![](/images/基于WordCloud的南京航空航天大学航空宇航学院硕士研究生毕业去向统计/Location.png)

# Python代码
```python
# -*- coding: utf-8 -*-
#wordcloud生成中文词云

from wordcloud import WordCloud, ImageColorGenerator
import codecs
import jieba
#import jieba.analyse as analyse
from scipy.misc import imread
import os
from os import path
import matplotlib.pyplot as plt
from PIL import Image, ImageDraw, ImageFont

# 绘制词云
def draw_wordcloud():
    #读入一个txt文件
    comment_text = open('Institution.txt','r').read()
    #结巴分词，生成字符串，如果不通过分词，无法直接生成正确的中文词云
    jieba.load_userdict('Institution.txt')	
    cut_text = " ".join(jieba.cut(comment_text))
    d = path.dirname(__file__) # 当前文件文件夹所在目录
    color_mask = imread("Cloud.png") # 读取背景图片
    cloud = WordCloud(
        #设置字体，不指定就会出现乱码
        font_path="SIMHEI.TTF",
        #font_path=path.join(d,'simsun.ttc'),
        #设置背景色
        background_color='white',
        #词云形状
        mask=color_mask,
        #允许最大词汇
        max_words=2000,
        #最大号字体
        max_font_size=200,
	min_font_size=35,
        width=1920,
        height=1080,
	prefer_horizontal=0.96,
        margin=0
    )
    word_cloud = cloud.generate(cut_text) # 产生词云
    image_colors = ImageColorGenerator(color_mask)
    word_cloud.to_file("Institution.png") #保存图片
    #  显示词云图片

    #plt.imshow(word_cloud,interpolation="bilinear")
    #plt.imshow(word_cloud.recolor(color_func=image_colors))
    #plt.axis('off')
    #plt.show()
    
if __name__ == '__main__':

    draw_wordcloud()
```