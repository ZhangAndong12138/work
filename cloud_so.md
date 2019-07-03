# 工会智搜(cloud_so)交接

## 一. 工会新闻爬虫程序
### 1. 源码位置
> svn://118.190.105.76/SDnewsinfo/

相关代码在src/lm目录下，其中
- `fond_xxx`为基金项目爬虫，与工会项目无瓜。
- `sddazhongribao`、`sdgonghuixinshikong`、`sdgongrenribao`、`sdqiluwanbao`、`shandonggongrenbao`、`shandongshenggonghui`为项目前期要求爬取各网站历史数据的代码，现已废弃不用。
- `SDNesinfoforPostgre`为爬取数据后保存至**Postgresql**数据库的代码，目前测试环境数据保存至MySQL，正式环境数据先保存至部署在互联网上的MySQL后，再同步至政务外网的~~瀚高~~**Postgresql**数据库，所以也没有实际作用。
- `SDNewsinfo`为工会新闻的相关爬取程序，分为*Setp1*和*Step2*。
### 2. 数据来源
- 大众日报
- 大众网
- 工人日报
- 齐鲁网
- 齐鲁晚报
- 工会新时空
- 山东工人报
- 山东省总工会官网
> [ 标题带有“**工会**”的数据 ]
### 3. 步骤
1. *Step1*：访问新闻列表，获取标题、链接、发布媒体、时间等信息
2. *Step2*：访问新闻内容，获取新闻来源(即发布媒体)、阅读量(如有)、新闻内容
### 4. 结果保存位置
- 开发/测试环境 
> mysql://118.190.105.76/cloud_so
- 正式环境需要联系浪潮相关人员
### 5. 定时执行任务
- 测试环境，使用**crontab**命令
> 输入命令：`crontab -e`
> -- --
> 在末尾添加一行：`0 23 * * * /opt/stepone.sh`
> 
>（第一个0代表每小时的的0分，第二个23代表每天的23时，这条定时任务指定每天的23时0分自动执行/opt/stepone.sh）
> -- --
> 在末尾添加一行：`5 23 * * * /opt/steptwo.sh`
>
>（第一个5代表每小时的的5分，第二个23代表每天的23时，这条定时任务指定每天的23时5分自动执行/opt/steptwo.sh）
- 正式环境，由浪潮相关人员处理

## 二. ElasticSearch  
### 1. Elastic Stack版本
> ElasticSearch - 6.5.1

> Logstash - 6.5.1
### 2. ElasticSearch上的索引
- cloud_so_doc 工会智搜的文档文件，主要来自于政研室提供
- ~~cloud_so_member 工会人员信息，由于一员一档功能被隐藏，而且正式环境身份证加密无法破解，所以正式环境没用到~~
- cloud_so_news 工会相关新闻，来源于爬虫程序
- index_category_metadata 索引元数据，目前包括cloud_so_doc和cloud_so_news
- sensitive_words 敏感词，用于过滤不和谐的搜索关键字
- sxkj_hot_words 热词，最近搜索热度较高的词

### 3.Logstash导入数据
- 从`Postgresql`数据库导入到`ElasticSearch`
- 配置文件
1. `pipelines.yml`：`logstash`的管道配置文件，一般仅修改`pipeline.id`和`path.config`即可，其中`path.config`为conf配置文件的路径。
2. `xxx.conf`：数据导入的配置文件，注释很详细，按注释修改即可。
3. `xxx.sql`：SQL文件，路径在`xxx.conf`中的`statement_filepath`项进行配置，主要目的是通过SQL的**Alias**将数据库的字段名转化为希望在`ElasticSearch`中希望看到的名称。
- 导入数据的命令
1. 按单个`conf`文件的配置导入数据
> ./bin/logstash -f /config/xxx.conf
2. 多个`conf`文件批量导入数据
> ./bin/logstash -e



## 三. 工会智搜项目
### 1. 源码位置
> svn://118.190.105.76/cloud_so
### 2. 项目内部ElasticSearch客户端
> 正在使用：Java High Level REST Client
> 
>https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.6/java-rest-high.html
>
>备选：TransportClient

### 3. 搜索流程
1. 
