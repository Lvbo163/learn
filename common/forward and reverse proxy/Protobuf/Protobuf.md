[原文地址](https://www.jianshu.com/p/5ea08c6b7031)

#Protobuf简介和使用

**未不明不知不觉 **

## protobuf是什么？
protobuf是google旗下的一款平台无关，语言无关，可扩展的序列化结构数据格式。所以很适合用做数据存储和作为不同应用，不同语言之间相互通信的数据交换格式，只要实现相同的协议格式即同一proto文件被编译成不同的语言版本，加入到各自的工程中去。这样不同语言就可以解析其他语言通过protobuf序列化的数据。目前官网提供了C++,Python,JAVA,GO等语言的支持。

## protobuf 语法定义
要想使用protobuf必须得先定义proto文件。所以得先熟悉protobuf的消息定义的相关语法。下面就来介绍
首先我们先定义一个proto文件，结构如下：

```
message Article {
required int32 article_id=1;
optional string article_excerpt=2;
repeated string article_picture=3;
}
```

上面我们主要定义了一个消息，这个消息包括文章ID，文章摘要，文章图片。下面给出消息定义的相关说明

名称	定义
message	是消息定义的关键字
required	表示这个字段必须的，必须在序列化的时候被赋值。
optional	代表这个字段是可选的，可以为0个或1个但不能大于1个。
repeated	则代表此字段可以被重复任意多次包括0次。
int32和string	字段的类型。后面是我们定义的字段名。
最后的1，2，3则是代表每个字段的一个唯一的编号标签，在同一个消息里不可以重复。这些编号标签用与在消息二进制格式中标识你的字段，并且消息一旦定义就不能更改。需要说明的是标签在1到15范围的采用一个字节进行编码。所以通常将标签1到15用于频繁发生的消息字段。编号标签大小的范围是1到229。此外不能使用protobuf系统预留的编号标签（19000 －19999）。

当然protobuf支持更多的类型，比如bool,double,float，枚举，也可以是其他定义过的消息类型譬如前面的消息Article。支持的基本类型如下：

## 基本数据类型

下面让我们定义一个数据比较多的article.proto文件来再次说明下proto语法的相关内容，起码通过列子可以更直观的感受。

```
syntax = "proto2";
message Article {
  required int32 article_id = 1;
  optional string article_excerpt = 2;
  repeated string article_picture = 3;
  optional int32  article_pagecount = 4 [default = 0];
  enum ArticleType {
    NOVEL = 0;
    PROSE = 1;
    PAPER = 2;
    POETRY = 3;
  }
  optional ArticleType article_type = 5 [default = NOVEL];
  message Author {
    required string name = 1; //作者的名字
    optional string phone = 2;
  }
  optional Author author = 6;
  repeated int32 article_numberofwords = 7 [packed=true];
  reserved  9, 10, 12 to 15;
  extensions 100 to 1000;
}
extend Article {
  optional int32 followers_count = 101;
  optional int32 likes_count= 102;
}
message Other {
  optional string other_info = 1;
  oneof test_oneof {
    string code1 = 2;
    string code2 = 3;
  }
}
```

上面proto文件，我们定义了enum枚举类型，嵌套的消息。甚至对原有的消息进行了扩展，也可以对字段设置默认值。添加注释等
此外reserved关键字主要用于保留相关编号标签，主要是防止在更新proto文件删除了某些字段，而未来的使用者定义新的字段时重新使用了该编号标签。这会引起一些问题在获取老版本的消息时，譬如数据冲突，隐藏的一些bug等。所以一定要用reserved标记这些编号标签以保证不会被使用
当我们需要对消息进行扩展的时候，我们可以用extensions关键字来定义一些编号标签供第三方扩展。这样的好处是不需要修改原来的消息格式。就像上面proto文件，我们用extend关键字来扩展。只要扩展的字段编号标签在extensions定义的范围里。
对于基本数值类型，由于历史原因，不能被protobuf更有效的encode。所以在新的代码中使用packed=true可以更加有效率的encode。注意packed只能用于repeated 数值类型的字段。不能用于string类型的字段。
在消息Other中我们看到定义了一个oneof关键字。这个关键字作用比较有意思。当你设置了oneof里某个成员值时，它会自动清除掉oneof里的其他成员，也就是说同一时刻oneof里只有一个成员有效。这常用于你有许多optional字段时但同一时刻只能使用其中一个，就可以用oneof来加强这种效果。但需要注意的是oneof里的字段不能用required，optional，repeted关键字
一般在我们的项目中肯定会有很多消息类型。我们总不能都定义在一个文件中。当一个proto文件需要另一个proto文件的时候，我们可以通过import导入，就像下面这样：

```
import "article.proto";
message Book {
//定义消息体
}
```
protobuf也提供了包的定义，只要在文件开头定义package关键字即可。主要是为了防止命名冲突，不过对于Python语言在编译的时候会忽略包名。

```
package "foo.bar"；
message  Book {
//定义消息体
}
```

很多时候我们会修改更新我们定义的proto文件，如果不遵守一定规则的话，修改的后proto文件可能会引发许多异常。在官网上对更新proto有以下几点要求

不能改变已有的任何编号标签。
只能添加optional和repeated的字段。这样旧代码能够解析新的消息，只是那些新添加的字段会被忽略。但是序列化的时候还是会包含哪些新字段。而新代码无论是旧消息还是新消息都可以解析。
非required的字段可以被删除，但是编号标签不可以再次被使用，应该把它标记到reserved中去
非required可以被转换为扩展字段，只要字段类型和编号标签保持一致
5.相互兼容的类型，可以从一个类型修改为另一个类型，譬如int32的字段可以修改为int64
ptotobuf语法相对比较简单，一般都能很快熟悉上手。这里只是粗浅的介绍下，更多详细内容可以参考https://developers.google.com/protocol-buffers/docs/proto。
proto文件编译
现在我们有了proto文件，需要把它编译成我们需要的语言，这里以python为例。通过以下命令生成我们需要的python代码，你会发现目录多了一个article_pb2.py的文件。

 protoc -I=.  --python_out=.  article.proto
-I 指定搜索proto文件的目录，这里指定为当前目录。-I 也可以写成 –proto_path
–python_out 会将生成的python代码文件放到等号后面指定的目录，这里也指定当前目录。如果需要生成其他语言的代码譬如java换成–java_out即可。这里提供一个官网提供的模版，如下

protoc --proto_path=_IMPORT_PATH_ --cpp_out=_DST_DIR_ --java_out=_DST_DIR_ --python_out=_DST_DIR_ _path/to/file_.proto
最后指定我们要编译的proto文件。
现在我们有了编译后的article_pb2.py，加入到我们的项目中去该怎么用呢？这个时候就需要用到google提供的protobuf python API。 下面我们通过例子来简单介绍下API的使用
protobuf python api的使用
直接贴代码来看，详细的说明都在注释里。主要的SerializeToString和ParseFromString2个方法。一个序列化，一个反序列化。

```python
# -*- coding: utf-8 -*-
 
import Article_pb2
from google.protobuf import json_format
from google.protobuf import text_format
 
article = Article_pb2.Article()
article.article_id = 1  # 必须赋值，不然在序列化得时候会报异常
article.article_excerpt = "文章简介"
article.article_type = 2
 
# 内嵌消息操作
author = article.author
author.name = "oliver"
author.phone = "11111111111"
 
# repeated类型的字段添加
article_picture = article.article_picture
article_picture.append("1.jpg")
article_picture.append("2.jpg")
 
article.Extensions[Article_pb2.followers_count] = 30  # 给扩展得字段赋值
 
print article.IsInitialized()  # 检查required字段是否全部被赋值
"""
 输出True
"""
 
print article.ListFields()  # 列出所有字段得一个元组列表
 
article_binary = article.SerializeToString()  # 序列化API
# article.SerializePartialToString()  # 也可以序列化消息，只不过它不会检查required是否被设置，也就是说可以序列化required字段没有被赋值的情况
 
with open("article.binary.txt", "wb+") as f:  # 保存到文件
    f.write(article_binary)
 
# 反序列化API ParseFromString 此外将ParseFromString换成MergeFromString这个接口来反序列化也可以
another_article = Article_pb2.Article()
another_article.ParseFromString(article_binary)
print(another_article)
"""
article_id: 1
article_excerpt: "\346\226\207\347\253\240\347\256\200\344\273\213"
article_picture: "1.jpg"
article_picture: "2.jpg"
article_type: PAPER
author {
  name: "oliver"
  phone: "11111111111"
}
[followers_count]: 30
 
"""
 
 
# 消息与json相互转化, 通过json_format的MessageToJson这个API
article_json = json_format.MessageToJson(article)
print(article_json)
"""
{
  "followersCount": 30,
  "author": {
    "phone": "11111111111",
    "name": "oliver"
  },
  "articleExcerpt": "\u6587\u7ae0\u7b80\u4ecb",
  "articleId": 1,
  "articleType": "PAPER",
  "articlePicture": [
    "1.jpg",
    "2.jpg"
  ]
}
"""
 
# 消息之间互相复制，主要用到CopyFrom 和MergeFrom 2个API
copy_article = Article_pb2.Article()
copy_article.CopyFrom(article)
print(copy_article)
 
"""
注意运行以下2行注释代的码需要把 “article.Extensions[Article_pb2.followers_count] = 30”这行代码注释掉。
猜想extension是对原消息得扩展。并不完全属于Article。譬如执行一下代码会报article没有followers_count这个属性
 
article.followers_count = 30
 
google.protobuf.json_format.ParseError: Message type "Article" has no field named "followersCount".
 
所以将json转换为消息类型的时候， 扩展的类型无处安放。
"""
# article_init = json_format.Parse(article_json, article)
#
# print(article_init)
 
print text_format.MessageToString(another_article)
 
# oneof操作,会发现当执行 oneof.code2 = "code2"之后，输出的结果中没有code1.自动被清除了。
oneof = Article_pb2.Other()
oneof.code1 = "code1"
print(oneof)
"""
code1: "code1"
"""
oneof.code2 = "code2"
print(oneof)
"""
code2: "code2"
"""
# 删除指定字段的数据
copy_article.ClearField("author")
# 删除所有数据
copy_article.Clear()

```
以上主要是通过python来操作protobuf序列化的数据，我们也可以将序列化后的数据通过网络发给其他应用。通过protobuf序列化的数据体量更小，传递效率相比于XML，JSON效率会更高。其他应用也不需要是python，可以是java，c++。只要实现了相同的proto协议，就可以解析发送过来的序列化数据。
