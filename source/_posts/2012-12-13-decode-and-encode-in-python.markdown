---
layout: post
title: "Decode and Encode in Python"
date: 2012-12-13 15:09
comments: true
categories: [Programming, Python]
---

本文主要针对2.7.x版本，对3.x版本可能不适用。

# 什么是Decode, Encode

Python内部使用Unicode对所有的字符编码。对于Python而言，Decode指的是将其它编码转化为Unicode，Encode指把Unicode转化为其它编码。

本文只针对utf-8与Unicode的转化，gbk, gb2312就不添乱了。不过可以简单说一句，Python识别源文件的编码主要根据两个内容：

  * 源码中的coding，如：

```
# -*- coding: utf-8 -*-
```

  * 源码文件的实际编码


# decode, encode, unicode函数

在Python console输入以下内容：

```
ustring = u'我们'
ustring                       # u'\u6211\u4eec'
ustring.encode('utf-8')       # '\xe6\x88\x91\xe4\xbb\xac'

string = '我们'
string                        # '\xe6\x88\x91\xe4\xbb\xac'
string.decode('utf-8')        # u'\u6211\u4eec'
unicode(string, 'utf-8')      # u'\u6211\u4eec'
```

以上的内容是Mac下的结果，在Linux应该也没有问题，但不保证在Windows下也会产生相同的结果。原因还是以上两条，而Mac和Linux的shell编码是utf-8的，Windows就不清楚了。

从以上的代码可以得出：

  * unicode和decode的作用是一样的，都是把其它编码转化成Unicode；

  * 字符前加'u'表示Unicode编码，不加任何东西则是默认编码；

  * encode和decode确实是按照之前提到的那样工作；



# 读写文件时的编码转化


继续输入以下代码：

```
open('unicode.txt', 'wb').write(ustring)                   # UnicodeDecodeError
open('utf-8.txt', 'wb').write(ustring.encode('utf-8'))
open('utf-8.txt', 'rb').read()                             # '\xe6\x88\x91\xe4\xbb\xac'
open('utf-8.txt', 'rb').read().decode('utf-8')             # u'\u6211\u4eec'
```

第一条语句是报错的，也就是说Python的内部编码字符串是不能直接写到文件的，应该先encode到一种编码之后再写入到文件。

read调用读取的是原始内容，不经过任何编码，所以需要在程序中自己处理编码问题。比如要把gbk转化成utf-8，则这样调用：

```
ustring = open('gbk.txt', 'rb').read().decode('gbk')
open('utf-8.txt', 'wb').write(ustring.encode('utf-8'))
```



# 读写json时的编码转化

## json.dump

输入以下代码：

```
import json

address = [{'name': '杨xx', 'country': '中国'}, {'name': '卢xx', 'country': '美国'}]
json.dump(address, open('address.json', 'wb'), indent=2)
```

然后打开address.json文件，看到的内容如下：

```
[
  {
    "country": "\u4e2d\u56fd", 
    "name": "\u6768xx"
  }, 
  {
    "country": "\u7f8e\u56fd", 
    "name": "\u5362xx"
  }
]
```

之所以看到这样的结果，是因为json模块内部对address转化成字串的时候，把字串从其它编码解码为Unicode（其实json.dump还有一个参数encoding，默认为'utf-8'，这里没有给出），然后又对Unicode进行了转义（这里write没有报UnicodeDecodeError，是因为json内部把Unicode二进制转化成了'\u4e2d'这样的串，这就是所谓的转义），最后write转义后的字符。

如果这种情况下想要等到正常的中文串，可以这样做：

```
json.dump(address, open('address2.json', 'wb'), indent=2, ensure_ascii=False)
```

设置ensure_ascii=False禁止json进行解码和转义，又因为address的编码就是'utf-8'的，所以write的时候不会报错。

但是如果遇到这种情况：

```
address = [{u'name': u'杨xx', u'country': u'中国'},
           {u'name': u'卢xx', u'country': u'美国'}]
```

直接调用：

```
json.dump(address, open('address.json', 'wb'), indent=2)
```

没有悬念，会得到如上address.json的结果。需要指出的是这次json在内部判断出字符是Unicode编码，所以省略了解编的步骤，只进行转义。

如果想得到中文，按照以上的经验应该输入:

```
json.dump(address, open('address2.json', 'wb'), indent=2, ensure_ascii=False)
```

好吧，出错了，又是UnicodeEncodeError。简单分析一下可以得到结论，ensure_ascii=False保证不会进行解码（当然这里不需要解码）和转义，这样json输出给write的就直接是Unicode字符，之前已经提到过了，write是不能直接写Unicode的。

那么如何让json输出正常的中文呢？一种比较容易想到的就是把整个address的字符全部用utf-8编码之后再像写address2.json那样输出。但是这种方法代价比较大，另一种比较好的方式是：

```
jsonunicodestr = json.dumps(address, indent=2, ensure_ascii=False)
open('address2.json', 'wb').write(jsonunicodestr.encode('utf-8'))
```

第一句把address转化成一个字符串，而且不进行转义，所以整个jsonunicodestr是Unicode字串。然后把字串编码成utf-8写入文件。

当然也可以合二为一：

```
import codecs

json.dump(address, codecs.open('address2.json', 'wb', 'utf-8'), indent=2, ensure_ascii=False)
```

## json.load

无论是从address.json还是address2.json，调用json.load的时候得到的都是Unicode编码的字符。



# 关于Python编码处理几点想法
  * 任何时候使用utf-8；
  * 怎么进，怎么出，尽量不涉及编码问题；
  * json模块是个意外，经过json的所有字符都会解码成Unicode；
