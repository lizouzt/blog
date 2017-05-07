---
layout:     post
title:      "Excel转json思路探究"
subtitle:   ""
date:       2017-05-08
author:     "ZX"
comments:	true
header-img: "/blog/img/FE/bgexcel.jpeg"
---
# Excel转json的一小步

## 前言
工作中会碰到，运营的同学把数据存储在excel里维护的情况，当需要把这些数据传入后台数据库中时，就需要吧excel转换为后台可接受的数据类型如json，这时就涉及到如何把excel表格的xlsx格式转换成json的问题，下面就来看看前端用js实现的一些思路。

## xlsx是个什么鬼？
xlsx文件是Microsoft Office Excel 2007或者更新版本保存的文件格式，是用新的基于XML的压缩文件格式取代了其目前专有的默认文件格式，在传统的文件名扩展名后面添加了字母x（即：docx取代.doc、.xlsx取代.xls等等），使其占用空间更小。

Excel2007及其以后版本的EXCEL文件其后缀为xlsx，它的原面目是打包压缩的文件，将文件后缀改为rar，解压出来会看到真实的文件内容，其实质为XML文件的一个合集。一般有3个目录及一个文件：
1. _rels
2. docProps
3. xl
  * _rels
  * sharedStrings.xml
  * styles.xml
  * theme
  * workbook.xml
  * worksheets
    * sheet1.xml
4. [Content_Types].xml

### xlsx解压文件分析

1. styles.xml    
存放的是 xlsx 文件里所有的格式/样式定义，主要由 <StyleSheet> 域构成。这些格式/样式大致有 <numFmts> <fonts> <fills> <borders> <cellStyleXfs> <cellXfs> <CellStyles> <dxfs> <tableStyles> <colors> 这些个域定义组成，每个域又是有一定数量的子域组成。

2. workbook.xml  
主要是 <sheets> 和 <definednames> 两个域。<sheets> 说明了各个工作表的名称、内部 sheetId 以及所对应的在压缩包中 worksheets 目录下的 sheet?.xml 文件（通过 r:Id 项指明索引号）。

3. sharedStrings.xml 和 worksheets/sheet1.xml  
主要应该是存放些 xlsx 文件中的数据内容。简略查看之下发现有两类：一类是非 ascii 的如汉字这样的；一类是同样内容多次出现在 xlsx 文件里的。对汉字这样的，即便只是出现过一次，好像也不会直接出现在 worksheets\sheet?.xml 文件中，而是定义在 SharedStrings.xml 中，又 worksheets\sheet?.xml 进行引用。  
对那些同样内容出现在多个单元格里的情形，真如 SharedStrings.xml 其名所意，这样的处理方法会减小文件的尺寸，节省存储空间。  
在这些文件中，所有对其它内容的引用，都是采用 id 或索引序号的方式，而非直接的内容表示；id 或索引序号以 00 为基。  

value对应关系：
![xlsxparse](/blog/img/FE/xlsxparse.png)



## xlsx转化为json的三种思路

### html table标签
#### excel导出html 解析html的table标签
第一种思路excel导出htm，数据存储在table标签中，所以可以解析table标签来得到json。  

导出的htm：
![xlsxhtm](/blog/img/FE/xlsxhtm.png)

解析的js：
![xlsxhtmjs](/blog/img/FE/xlsxhtmjs.png)

#### 优缺点
优点就是用的技术比较基础，不依赖于node，可以通过浏览器级别的写法去处理数据，可读可写。缺点是用法虽然简单但是解析table标签还是比较繁琐的。

有jquery插件，如果使用jquery插件的话会简化很多。但是在页面级的引入插件，页面级的处理逻辑还性能上有损耗。  

因此我们就像还有没有其他更通用更简单的形式。查看excel的导出格式里，有个更加通用的csv。下面来介绍下csv。



### csv
#### 什么是csv？
逗号分隔值（Comma-Separated Values，CSV，有时也称为字符分隔值，因为分隔字符也可以不是逗号），其文件以纯文本形式存储表格数据（数字和文本）。纯文本意味着该文件是一个字符序列，不含必须象二进制数字那样被解读的数据。CSV文件由任意数目的记录组成，记录间以某种换行符分隔；每条记录由字段组成，字段间的分隔符是其它字符或字符串，最常见的是逗号或制表符。通常，所有记录都有完全相同的字段序列。
CSV文件格式的通用标准并不存在，但是在RFC 4180中有基础性的描述。使用的字符编码同样没有被指定，但是7-bit ASCII是最基本的通用编码。  
csv格式：
![xlsxcsv](/blog/img/FE/xlsxcsv.png)

#### 为什么要转为csv？
表格数据文本文件通用，简单。
csv是最早用在简单的数据库里的....格式简单开放性强。总之就是说非常容易被导入pc表格和数据库中。  
虽然它不是一个成文的标准格式，但是基本上被所有的表格处理软件接纳和识别，所以他就相当于是表格数据界的json。它可以实现不同表格文件转化时的桥梁作用。  

转化csvtojson的node代码：
![xlsxcsvjs](/blog/img/FE/xlsxcsvjs.png)

导出csv时的编码问题  
因为node不是用模块的话是不支持gbk等中文编码的，所以我们需要导出csv是utf8的编码，直接excel导出貌似不行，查了资料之后，发现mac上的Number软件可以打开excel并且也可以导出csv文件，最重要的是可以指定编码类型。所以就用Number导出utf8的csv，这样文件就准备好了，然后在node里操作。
csv是文本，读出来的也是文本，那么csv转json其实就是文本之间的格式转换了。

导出csv文件转化的优缺点：简单，通用，还是要导一次有点麻烦，如果有合并单元格的话就会有内容为空的格子。


### node模块xlsx
#### 最直接的方法，使用第三方node模块解析
[sheetjs](http://sheetjs.com/)  
js-xlsx: 目前 Github 上 star 数量最多的处理 Excel 的库，支持解析多种格式表格XLSX / XLSM / XLSB / XLS / CSV，解析采用纯js实现，写入需要依赖nodejs或者FileSaver.js实现生成写入Excel，可以生成子表Excel，功能强大，但上手难度稍大。不提供基础设置Excel表格api例单元格宽度，文档有些乱，不适合快速上手；  
可读可写表格文件，纯js实现符合官方标准和测试标准，强调稳健性，跨表格文件格式兼容，并且兼容ES3和老IE（提供了Polyfills和shim垫片插件），可在node里使用也可在浏览器端使用。    

支持格式：
![xlsxnode](/blog/img/FE/formats.png)
![xlsxnode](/blog/img/FE/legend.png)

注意使用这个模块功能时尽量不要用node的读取流操作。
因为常见的表格格式（XLS, XLSX/M, XLSB, ODS）都是ZIP或CFB的压缩文件。
这两种格式都不会讲目录结构声明放在文件的开头，ZIP文件将中央目录记录放置在逻辑文件的末尾，而CFB文件可以将FAT结构放置在文件的任何位置！
因此，如果要正确处理这些格式，流式传输功能必须在开始之前缓冲整个文件。
因为要缓冲整个文件，这样就违背了流式读取的初衷，所以模块不提供流式读取文件的功能。

在使用这个库之前，先介绍库中的一些概念。
workbook 对象，指的是整份 Excel 文档。我们在使用 js-xlsx 读取 Excel 文档之后就会获得 workbook 对象。
worksheet 对象，指的是 Excel 文档中的表。我们知道一份 Excel 文档中可以包含很多张表，而每张表对应的就是 worksheet 对象。
cell 对象，指的就是 worksheet 中的单元格，一个单元格就是一个 cell 对象。  
它们的关系如下：

```
// workbook
{
  SheetNames: ['sheet1', 'sheet2'],
  Sheets: {
      // worksheet
      'sheet1': {
          // cell
          'A1': { ... },
          // cell
          'A2': { ... },
          ...
      },
      // worksheet
      'sheet2': {
          // cell
          'A1': { ... },
          // cell
          'A2': { ... },
          ...
      }
  }
}
```
基本用法  
1. 用 XLSX.read 读取获取到的 Excel 数据，返回 workbook
2. 用 XLSX.readFile 打开 Excel 文件，返回 workbook
3. 用 workbook.SheetNames 获取表名
4. 用 workbook.Sheets[xxx] 通过表名获取表格
5. 用 worksheet[address]操作单元格
6. 用XLSX.utils.sheet_to_json针对单个表获取表格数据转换为json格式
7. 用XLSX.writeFile(wb, 'output.xlsx')生成新的 Excel 文件

使用代码：
![xlsxnodejs](/blog/img/FE/xlsxnodejs.png)

优缺点：
功能强大，支持多种格式兼容多个版本js。直接引入xlsx文件转换，一步到位。可以根据数据导出xlsx文件。依赖node解析，浏览器端使用需要额外的依赖。

## 总结
这三种思路都可以实现excel向json的转化，前两种都是可以自己实现但是不方便，可以用来了解原理，最后的xlsx模块功能已经很强大了，生产中可以使用这种成熟的模块提高效率。

## 参考文献
1. xlsx文件数据结构解析 By: Datainner
2. xlsx简单分析 By: zara
3. table表数据转Json格式 By: 佚名
4. Node 读写 Excel 文件探究实践 By: 高大师
5. 在 Node.js 中利用 js-xlsx 处理 Excel 文件 By: scarlex
6. Parse XLSX with Node and create json By: stackoverflow
7. Comma-Separated Values By: wikipedia
8. https://github.com/SheetJS/js-xlsx By: SheetJS
9. https://sheetjs.gitbooks.io/docs/#xlsx By: SheetJS
10. http://sds.sourceforge.net/doc/csf.html By: Stig E Sandø
