# 简介

## 什么是elasticsearch

一个使用Java开发的开源的高扩展的分布式全文检索引擎



## elasticsearch和solr的对比

不考虑建索引的同时进行搜索，单纯的对已有数据进行搜索时，solr速度更快，当建立索引时，搜索效率下降，因为solr建立索引会产生io阻塞，这时 Elasticsearch具有明显的优势。

数据量越大，elasticsearch相比较于solr的优势越明显

综上所述，Solr的架构不适合实时搜索的应用。

## elasticsearch和关系型数据库的对比

| 关系型数据库     | Elasticsearch       |
| ---------------- | ------------------- |
| 数据库(database) | 索引(indices)       |
| 表(tables)       | 类型(types)         |
| 行(rows)         | 文档(documents)(id) |
| 字段(columns)    | 字段(fields)        |



elasticsearch(集群)中可以包含多个索引(数据库)，每个索引中可以包含多个类型(表)， 每个类型下又包含多个文档(行)，每个文档中又包含多个字段(列)。

**物理设计（分片）**： elasticsearch 在后台把每个索引划分成多个分片，每分分片可以在集群中的不同服务器间迁移

**逻辑设计**： 一个索引类型中，包含多个文档，比如说文档1，文档2。 当我们索引一篇文档时，可以通过这样的 顺序找到它: 索引 ▷ 类型 ▷ 文档ID ，通过这个组合我们就能索引到某个具体的文档。 注意:这里的ID不必是整数，实际上它是个字符串。



## 倒排索引

elasticsearch使用的是一种称为倒排索引的结构，采用Lucene倒排索作为底层。这种结构适用于快速的 全文搜索， 一个索引由文档中所有不重复的列表构成，**对于每一个词，都有一个包含它的文档列表。**
例如，现在有两个文档， 每个文档包含如下内容：

Study every day, good good up to forever # 文档1包含的内容 To forever, study every day, good good up # 文档2包含的内容

为了创建倒排索引，我们首先要将每个文档拆分成独立的词(或称为词条或者tokens)， 然后创建一个包含所有不重复的词条的排序列表，然后列出每个词条出现在哪个文档 :
<img src="/Users/liqunzhang/IdeaProjects/elasticsearch-study/img.png" alt="img" style="zoom: 67%;" />

现在，我们试图搜索 to forever，只需要查看包含每个词条的文档
<img src="/Users/liqunzhang/IdeaProjects/elasticsearch-study/img_1.png" alt="img_1" style="zoom:67%;" />

两个文档都匹配，但是第一个文档比第二个匹配程度更高。如果没有别的条件，现在，这两个包含关键 字的文档都将返回。 再来看一个示例，比如我们通过博客标签来搜索博客文章。那么倒排索引列表就是这样的一个结构 :
<img src="/Users/liqunzhang/IdeaProjects/elasticsearch-study/img_2.png" alt="img_2" style="zoom:67%;" />

**倒排索引的精髓就是：在于把每个词都搞出一个包含这个词的文档列表 （搞出个文档列表，把表中显示的是包含这个词的文档）**

# ik分词器

## ik_smart和ik_max_word



![image-20210606155724253](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210606155724253.png)

![image-20210606155905717](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210606155905717.png)

```properties
GET _analyze
{
"analyzer":"ik_smart",
"text":["中国共产党好。"]
}
```

![img_3](/Users/liqunzhang/IdeaProjects/elasticsearch-study/img_3.png)

```properties
GET _analyze
{
"analyzer":"ik_max_word",
"text":["中国共产党好。"]
}
```

![img_4](/Users/liqunzhang/IdeaProjects/elasticsearch-study/img_4.png)

## 自定义词库

1. 进入elasticsearch/plugins/ik/config目录
2. 新建一个my.dic文件，编辑内容
3. 修改IKAnalyzer.cfg.xml（在ik/config目录下）

```xml

<properties>
    <comment>IK Analyzer 扩展配置</comment> <!-- 用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict">myword.dic</entry> <!-- 用户可以在这里配置自己的扩展停止词字典 -->
    <entry key="ext_stopwords"></entry>
</properties>
```

# Rest风格
## 基本Rest命令说明：
<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210606113546334.png" alt="image-20210606113546334" style="zoom:50%;" />



## 基础测试



1. 不指定字段数据类型

```properties
PUT mytest/user/1
{
  "name": "张立群",
  "age": 18
}
```

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114224833521.png" alt="image-20210114224833521" style="zoom:50%;" />

输入 GET mytest/user/1,会给我们查询对应的结果

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114231138176.png" alt="image-20210114231138176" style="zoom:50%;" />

PUT test1/type1/1 ： 索引test1相当于关系型数据库的库，类型type1就相当于表 ，1 代表数据中的主键 id





输入GET mytest，发现数据类型已经自动帮我们设置好了字段类型

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114231225771.png" alt="image-20210114231225771" style="zoom:33%;" />



2. 指定字段数据类型
   - 字符串类型: text 、 keyword
   - 数值类型: long, integer, short, byte, double, flfloat, half_flfloat, scaled_flfloat
   - 日期类型: date
   - 布尔值类型: boolean
   - 二进制类型: binary

```properties
PUT mytest2
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      }
    }
  }
}
```



返回结果：<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114230012984.png" alt="image-20210114230012984" style="zoom:50%;" />



输入GET mytest2

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114230930432.png" alt="image-20210114230930432" style="zoom: 33%;" />



之后如我我们不按要求定义类型就会出现报错！

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114230211718.png" alt="image-20210114230211718" style="zoom:50%;" />

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114230220193.png" alt="image-20210114230220193" style="zoom:33%;" />



3. GET _cat/indices?v 获取elasticsearch的索引情况

   <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114231603273.png" alt="image-20210114231603273" style="zoom:33%;" />



# elasticsearch的增删改查

## 新增数据PUT

插入三条数据

```properties
PUT person/user/1
{
  "name": "张立群",
  "age" : 18,
  "tags" :["技术宅","IT男","务实"]
}

PUT person/user/2
{
  "name": "苏伊",
  "age" : 16,
  "tags" :["美女","温暖","善解人意"]
}

PUT person/user/3
{
  "name": "珈碧",
  "age" : 13,
  "tags" :["可爱","温柔","善良"]
}
```

**注意：** 如果数据不存在put是添加数据，如果数据存在则是修改数据，并把version加1

查看数据

在elasticsearch客户端查看数据

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114232705803.png" alt="image-20210114232705803" style="zoom:50%;" />



## 删除数据DELETE

举例：

```properties
DELETE person/user/3
```

删除person索引中的user类型中的3号文档



## 修改数据 POST

put也可以更新数据，上述已声明

使用post命令后面要有_update，而且不要忘了"doc"字段！

```properties
POST person/user/2/_update
{
  "doc": {
    "name": "女友苏伊",
    "tags": "我女朋友！她是完美的！"
  }
}
```



<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114233354898.png" alt="image-20210114233354898" style="zoom:50%;" />



## 查询数据GET

普通查询不再赘述，上面已经提过



### 1.条件查询

GET person/user/_search?q=name:"张"

查询name中有“张”的数据

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210114235652882.png" alt="image-20210114235652882" style="zoom:50%;" />

其中的value参数表示数据的数量



### 2.构建查询

#### 查询部分

```properties
GET person/user/_search
{
  "query": {
    "match": {
      "name": "张立群"
    }
  }
}
```



<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210115100302218.png" alt="image-20210115100302218" style="zoom: 33%;" />

#### 查询全部

```properties
GET person/user/_search
{
  "query": {
    "match_all": {}
  }
}
```

返回全部的所有字段结果，就像mysql中的select * 一样！



如果需求是只查询部分字段？

这里只查出age

```properties
GET person/user/_search
{
  "query": {
    "match_all": {}
  },
  "_source": ["age"]
}
```

如上例所示，在查询中，通过 _source 来控制仅返回 age 属性。



### 3.排序查询

正序查询  依照age正序

```properties
GET person/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}
```



倒序：asc



### 4.分页查询

```properties
GET person/user/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "from": 0     #从第一个开始查询
  , "size": 1   #每页显示一个数据
}
```



### 5.must查询

```properties
GET person/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "张立群"
          }
        },
        {
          "match": {
            "age": "17"
          }
        }
      ]
    }
  }
}
```

使用must查询，match中的两个条件都要符合才能查询出来



### 6.should查询

只要满足其中一个就能查询

```properties
GET person/user/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "name": "张立群"
          }
        },
        {
          "match": {
            "age": "13"
          }
        }
      ]
    }
  }
}
```



### 7.mustnot查询

```properties
GET person/user/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "age": "13"
          }
        }
      ]
    }
  }
}
```

查询年龄不是13的



### 8.filter过滤查询

```properties
GET person/user/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "name": "张立群"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "age": {
              "lt": 20,
              "gte": 10,
              "lte": 20
            }
          }
        }
      ]
    }
  }
}
```

查询姓名不是张立群且年龄小于20岁的



### 9.高亮显示

```properties
GET person/user/_search
{
 "query": {
   "match": {
     "name": "张立群"
   }
 },
 "highlight": {
   "fields": {
     "name": {}
   }
 }
}
```

高亮显示name

上面的高亮是系统给我们设定的，也可以自定义高亮

```properties
GET person/user/_search
{
 "query": {
   "match": {
     "name": "张立群"
   }
 },
 "highlight": {
   "pre_tags": "<b class='key' style='color:red'>",
   "post_tags": "</b>", 
   "fields": {
     "name": {}
   }
 }
}
```

输出结果：<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210115105509566.png" alt="image-20210115105509566" style="zoom: 33%;" />

**注意**：自定义标签中的值不能用花括号，否则报错！



# 使用Java操作ES



搭建Idea项目，配置maven环境，修改对应自己版本的ES依赖

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210115110050278.png" alt="image-20210115110050278" style="zoom: 33%;" />

