---
title: 使用python搭建小型搜索引擎
date: 2018-12-31 13:19:06
tags:
- Python
- Scrapy
- Django
- Haystack
- Whoosh
- Jieba
categories: 
- 项目总结
---

[项目代码链接](https://github.com/HaxiSnake/EXERCISE/tree/master/others/homework/MessageRetrieval/SearchEngine)

# 一、目标
为准备信息检索课程的期末大作业，因此我使用python搭建了一个小型的搜索引擎，其功能是检索[大工新闻网](http://news.dlut.edu.cn/)的新闻。
# 二、原理和工具简介

## 2.1 原理

该搜索引擎的原理是采用scrapy对大工新闻网进行爬虫，提取出文字新闻，并将新闻内容存入数据库A，再利用Django框架搭建一个搜索服务器，在服务器上部署Haystack+Whoosh搜索引擎，使用jieba分词工具来进行中文分词和停用词过滤。通过搜索引擎工具建立索引文件后，在前端完成用户交互界面，实现一个完整的小型搜索引擎。

## 2.2 工具简介

- Scrapy 

    Python开发的一个快速、高层次的屏幕抓取和web抓取框架，用于抓取web站点并从页面中提取结构化的数据。Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试。
    
    在本项目中用来爬取网站数据。

- Django

    Django是一个开放源代码的Web应用框架，由Python写成。采用了MVC的框架模式，即模型M，视图V和控制器C。

    在本项目用来做搜索服务器的框架。

- Haystack+Whoosh

    Haystack是一个Django上的应用，可以用于整合搜索引擎，它只依赖于它自身的代码，利用它可以切换不同的搜索引擎工具。Whoosh是一个索引文本和搜索文本的类库，它可以提供搜索文本的服务。

    在本项目中使用Haystack将Whoosh部署到Django服务器作为搜索引擎后端。

- Jieba

    Jieba是一个python实现的分词库，对中文有着很强大的分词能力。
    
    在本项目用于对新闻进行中文分词和停用词过滤。

# 三、开发环境和运行环境

## 3.1 开发环境

* win 10 
* python 3.6.5  
* scrapy 1.5.1 [安装教程](https://www.cnblogs.com/MC-Curry/p/8503813.html)
* Django 2.1.3 [安装教程](https://docs.djangoproject.com/zh-hans/2.1/howto/windows/)
* Haystack+Whoosh [配置教程](https://django-haystack.readthedocs.io/en/master/tutorial.html)
* Jieba 0.39 [配置教程](https://blog.csdn.net/yx1179109710/article/details/81304036)

## 3.2 运行环境

同开发环境

# 四、系统模块

## 4.1 网络爬虫模块

### 定义用于Scrapy爬虫的数据类型，包括链接、标题和文章内容:

    ```python
    class NewsItem(scrapy.Item):
        url = scrapy.Field() 
        title = scrapy.Field()
        content = scrapy.Field()
    ```
### 编写爬虫策略:

爬虫策略的制定依据于网页源代码的链接形式，由于要爬取的是文字类新闻，所以要跟进与文字类新闻的链接。对于具体的新闻页利用回调函数爬取其链接、标题和内容。而对于新闻列表页，需要跟进下一页的链接。具体规则代码如下:

    ```python
    class NewsSpider(CrawlSpider):
        print("news spider starting")
        name = 'news'
        allowed_domains = ['news.dlut.edu.cn']
        start_urls = ['http://news.dlut.edu.cn/']

        rules = (
            # 对于新闻页链接进行跟进
            Rule(LinkExtractor(allow=("xw/[a-z]+.htm"))),
            # 对于详细新闻页利用parse_item回调函数进行内容爬取
            Rule(LinkExtractor(allow=("info/\d{4,}/\d{3,}\.htm")),callback="parse_item"),
            # 对于新闻列表中的下一页链接进行跟进
            Rule(LinkExtractor(allow=("\d{1,}.htm"),restrict_xpaths="//a[@class='Next']")),
        )

        def parse_item(self, response):
            self.log("Hi, this is a new page! %s"% response.url)
            item = NewsItem()
            item['title'] = response.xpath('/html/head/title/text()').extract()[0]
            item['url'] = response.url
            item['content']=response.xpath("//div[@class='cont-detail fs-small']/p/text()").extract()
            yield item
    ```
### 使用pipline机制将数据保存至数据库当中

    ```python
    class SpiderprojectPipeline(object):
        def process_item(self, item, spider):
            if spider.name == 'news':
                conn = sqlite3.connect('db.sqlite3') 
                cursor = conn.cursor()
                title = item['title']
                url = item['url']
                content_tmp = item['content']
                content=""
                for p in content_tmp:
                    content+=p.strip()
                sql_search = 'select arturl from search_article where arturl=="%s"' % (url) 
                sql = 'insert into articles_article(title, content, arturl) values("%s", "%s", "%s")'%(title, content, url)
                try:
                    #如果当前数据库中不存在该条新闻，则将新闻保存至数据库当中
                    cursor.execute(sql_search)
                    result_search = cursor.fetchone()
                    if result_search is None or result_search[0].strip()=='':
                        cursor.execute(sql)
                        result=cursor.fetchone()
                        conn.commit()
                    cursor.execute(sql)
                    result=cursor.fetchone()
                    conn.commit()
                except Exception as e:
                    print(">>> catch exception !")
                    print(e)
                    conn.rollback()
            return item
    ```
## 4.2 搜索模块

在要进行搜索的应用的models.py文件中建立model类用来表示要进行搜索的新闻文章

    ```python
    class Article(models.Model):
        title = models.CharField(max_length=50)
        arturl = models.CharField(max_length=200)
        content = models.CharField(max_length=1000)
    ```
同时使用django命令`python manage.py makemigrations`和`python manage.py migrate`生成数据库文件，并用爬虫得到的数据库替换生成的数据库。

在django框架中配置Haystack+Whoosh来引入搜索模块

    ```python
    # 配置搜索引擎后端
    HAYSTACK_CONNECTIONS={
        'default':{
            'ENGINE': 'articles.whoosh_cn_backend.WhooshEngine',
            # 索引文件路径
            'PATH': os.path.join(BASE_DIR, 'whoosh_index'),  # 在项目目录下创建文件夹 whoosh_index
        }
    }
    # 当添加、修改、删除数据时，自动生成索引
    HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
    # 每页显示十条搜索结果
    HAYSTACK_SEARCH_RESULTS_PER_PAGE = 10
    ```
在要进行搜索的应用的目录下建立search_indexes.py文件

    ```python
    from haystack import indexes
    from articles.models import Article
    
    class ArticleIndex(indexes.SearchIndex, indexes.Indexable):     #类名必须为需要检索的Model_name+Index，这里需要检索Article，所以创建ArticleIndex
        text = indexes.CharField(document=True, use_template=True)  #创建一个text字段
    
        def get_model(self):          #重载get_model方法，必须要有！
            return Article
    
        def index_queryset(self, using=None):   #重载index_..函数
            """Used when the entire index for model is updated."""
            return self.get_model().objects.all()  
    ```
在articles\templates\search\indexes\articles\下建立article_text.txt文件确定搜索内容

    ```
    {{ object.title }}
    {{ object.content }}
    {{ object.url }}
    ```
## 4.2 预处理模块

使用jieba来进行中文分词，需要在whoosh_cn_backend文件中替换StemmingAnalyzer为ChineseAnalyzer

同时引入停用词表，配置ChineseAnalyzer使其支持停用词过滤

    ```python
    import jieba
    import re
    import os
    # 导入停用词过滤表
    stop_file_dir=os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    STOP_WORDS = frozenset([line.strip() for line in open(os.path.join(stop_file_dir, 'stop.txt'),'r',encoding='gbk').readlines()])

    accepted_chars = re.compile(r"[\u4E00-\u9FD5]+")

    class ChineseTokenizer(Tokenizer):

        def __call__(self, text, **kargs):
            words = jieba.tokenize(text, mode="search")
            token = Token()
            for (w, start_pos, stop_pos) in words:
                if not accepted_chars.match(w) and len(w) <= 1:
                    continue
                token.original = token.text = w
                token.pos = start_pos
                token.startchar = start_pos
                token.endchar = stop_pos
                yield token
        def ChineseAnalyzer(stoplist=STOP_WORDS, minsize=1, stemfn=stem, cachesize=50000):
            return (ChineseTokenizer() | LowercaseFilter() |
                    StopFilter(stoplist=stoplist, minsize=minsize) |
                    StemFilter(stemfn=stemfn, ignore=None, cachesize=cachesize))
    ```
配置完成后使用`python manage.py rebuild_index`建立搜索索引

## 4.4 交互模块

搜索首页html关键代码：

    ```html
    <form method='get' action="/search/" target="_blank">
        <input type="text" name="q">
        <br>
        <input type="submit" value="查询">
    </form>
    ```

首页如下图所示：
{% asset_img query.jpg image %}


查询结果页html关键代码：

``` html
{% load highlight %}
<h3>搜索&nbsp;<b>{{query}}</b>&nbsp;结果如下：</h3>
<ul>
{%for item in page%}
    <li>{{item.object.title|safe}}</li>
    <p>{% highlight item.object.content with query %}</p>
    <a href="{{item.object.arturl}}">{{item.object.arturl}}</a>
{%empty%}
    <li>啥也没找到</li>
{%endfor%}
</ul>
<hr>
{%for pindex in page.paginator.page_range%}
    {%if pindex == page.number%}
        {{pindex}}&nbsp;&nbsp;
    {%else%}
        <a href="?q={{query}}&amp;page={{pindex}}">{{pindex}}</a>&nbsp;&nbsp;
    {%endif%}
{%endfor%}
```

其中使用

``` html
{% load highlight %}
<p>{% highlight item.object.content with query %}</p>
```
进行搜索结果的高亮显示

最终搜索页面如下图所示：

{% asset_img search.jpg image %}

# 五、项目代码

[代码链接](https://github.com/HaxiSnake/EXERCISE/tree/master/homework/MessageRetrieval/SearchEngine)