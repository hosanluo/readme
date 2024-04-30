## 原版需求
1. #### [从1688抓取对应产品的链接，筛选条件](#1688数据匹配)：
    - 非定制供应商
    - 有验厂报告的供应商
    - 价格有竞争力的前几名供应商
    - 总抓取数量为10条

2. #### [销售价格与采购价格之间的可行性分析，在利润值内的显示为推荐](#销售价格与采购价格之间的可行性分析在利润值内的显示为推荐)

3. #### [与现有的已上架的产品进行对比，重复款显示SKU在备注上面](#匹配己方系统的产品-排重)

4. #### [将以下类别筛选去掉：婴儿、儿童类，机顶盒，激光类，医疗类，书籍类， DVD刻录机， 会员卡，木制品，电视机，鞋类，服装类，手饰类。(Amir负责开发)](#竞品产品爬虫相关)

5. #### [去掉AMAZON.UK， AMAZON.COM站点以及属于，增加以下站点的爬取 (Amir负责开发)](#竞品产品爬虫相关)
    - https://www.kaufland.de/
    - https://www.cdiscount.com/
    - https://www.worten.pt/
    - https://www.mediamarkt.de/
    - ~~https://www.elgiganten.se/~~

6. #### [Potential抓取信息增加：(Amir负责开发)](#竞关品产品爬虫相)
    - 多张图片
    - 产品描述（产品的功能，尺寸，重量）
    - 产品上架时间
    - 销量信息


## 对应上方需求点的相关开发以及拓展:
### 1688数据匹配
INF和AYD都有实现1688产品匹配的功能. 不过现在以AYD的为主. 
目前是用竞品产品里面的图片通过接口匹配1688的产品数据. 对于竞品产品旧数据处理, 需求方表示只需要处理2023-01-01之后的产品. 所以目前2023-01-01之后竞品产品都匹配了1688的产品(无图片的除外).  
在AYD这边如果看到竞品的旧数据在匹配1688产品上面出现多个竞品产品匹配到同一批1688的产品时, 可以点击详情里面的重新匹配. 这个问题是在INF做批量自动匹配的时候逻辑上面的bug导致的错误数据.  
匹配alibaba的产品数据是基于竞品产品基础上实现的. 一个竞品产品要匹配20条alibaba的产品. 由于旧数据的已经匹配好了. 之后每一次的竞品数量爬取也只是每次几百条. 所以1688产品匹配的脚本可以每天一次. 目前定时任务还没开启.  
<font color="red">注: 竞品产品的爬取之前是一个月两次, 数据量不大所以1688的脚本可以设定为一天一次. 但是需求方要求竞品产品的爬取后续要改成一天一次, 所以对应匹配1688产品的定时任务时间设定需要考虑如何设置.</font>
```
command:
app/Console/Commands/Competitor/PotentialProductMatchAlibabaProductCommand.php

model:
app/Models/Purchase/PotentialProductAlibabaMatches.php

controller:
app/Http/Controllers/Purchase/CompetitorController.php

service:
app/Services/Scrapy/ScraperApiService.php
app/Services/Scrapy/AlibabaService.php
app/Services/Purchase/CompetitorService.php
```

### 销售价格与采购价格之间的可行性分析，在利润值内的显示为推荐
重量以及包装规格是计算价格必要条件, 由于采集的1688数据缺失这些数据. 所以这个功能暂时开发不了.  
之前有和需求方讨论过, 后期可以在1688数据列表上方提供几个输入框, 分别填入重量, 长宽高然后自动计算出来.

### 匹配己方系统的产品, 排重.
第一版: 利用chatGPT分析竞品产品的title获取关键字, 然后匹配己方系统里面的产品title(已投入使用. 由于竞品产品来源的title语义定义方面的不可控因素, 所以这种匹配非常不精准)  
现在在进行开发的版本: 从产品图片入手. 利用google gemini分析系统产品图片提取图片的特征并储存. 当想查看某一个竞品产品是否存在相同的在运营时, 会即时将此产品的图像信息请求INF接口. 获取特征, 然后根据特征在已生成的图片特征表进行查询.   
此版本需要对己方系统产品的图片初始化, 全部提取特征并保存. 目前已有脚本在运行, 不过速度比较慢. 按当前时间2024-04-30来统计. 总共有9W多张图片需要提取特征, 脚本运行结束时间未明.
前端对于此版本的功能已完成, 不过暂未发布上线, 之前是想着等图片特征初始化之后再发布, 不过由于提取特征的过程比较久且未明. 所以功能可以先发布.  
<font color=red>注: 图片提取特征脚本目前只是对旧数据的初始化. 新数据的还未做对应处理. 可以考虑将此脚本设置为一天一次. 或者是在新增图片时进行特征提取.</font>
```
INF接口:
从title分析关键字  
app/Http/Controllers/Api/CompetitorController.php:59
提取图片特征  
app/Http/Controllers/Api/CompetitorController.php:92
检测是否属于过滤分类的覆盖范围  
app/Http/Controllers/Api/CompetitorController.php:154

AYD:
command  
app/Console/Commands/Competitor/CheckPotentialProductCategoryCommand.php
app/Console/Commands/GenerateImageFeatureCommand.php

目前图片特征生成的脚本正在运行. 
运行 ps aux | grep artisan 可以看到 GenerateImageFeatureCommand 的进程.
也可以在每天的 log 里面 grep GenerateImageFeatureCommand 看到运行到哪一个图片了.
```

## 竞品产品爬虫相关
需求的4,5,6点属于Amir负责. 计划是等Amir在INF完成后交给Amy测试, 没问题就迁移到AYD.  
以下是与2024-04-29第一次测试新的四个站点竞品爬取发现的issues:  
1. 部分站点的产品没有实现能爬取到多张图片
2. 缺失描述部分, 产品上架时间, 销量信息
3. category过滤的条件是什么?  

关于旧的站点issues:
1. cdon.se 于 2023-10-01 开始就没有新的数据
2. fyndiq.se 于 2021-12-16 开始就没有新的数据
3. elgiganten.se 和 amazon.fr 貌似从来都没有爬取过.
