<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 CSV文件](#1-csv文件)
    - [1.1 手动生成一个csv文件](#11-手动生成一个csv文件)
    - [1.2 cvs模块](#12-cvs模块)
        - [1.2.1 reader方法](#121-reader方法)
        - [1.2.2 writer方法](#122-writer方法)
- [2 ini文件处理](#2-ini文件处理)
    - [2.1 configparser模块](#21-configparser模块)
    - [2.2 常用方法](#22-常用方法)
        - [2.2.1 读取配置配件](#221-读取配置配件)
        - [2.2.2 section操作](#222-section操作)
        - [2.2.3 option操作](#223-option操作)
        - [2.2.4 获取value](#224-获取value)
        - [2.2.5 设置value](#225-设置value)
        - [2.2.6 保存修改后的配置文件](#226-保存修改后的配置文件)
    - [2.3 字典的访问方式](#23-字典的访问方式)

<!-- /TOC -->
# 1 CSV文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;逗号分隔值（Comma-Separated Values，CSV，有时也称为字符分隔值，因为分隔字符也可以不是逗号），其文件以纯文本形式存储表格数据（数字和文本）。
```csv
# 下面都是csv文件的内容格式
1,2,3,4,5

1,2,"1,3"

1,2,"a,""f"
```
- CSV 是一个被行分隔符、列分隔符划分成行和列的文本文件。
- CSV 不指定字符编码。

它有以下规范：
1. 行分隔符为\r\n，最后一行可以没有换行符
2. 列分隔符常为逗号或者制表符。每一行称为一条记录record
3. 字段可以使用双引号括起来，也可以不使用。如果字段中出现了双引号、逗号、换行符必须使用双引号括起来。
4. 如果字段的值是双引号，需要额外使用一个个双引号，表示一个转义。
5. 表头可选，和字段列对齐就行了。

## 1.1 手动生成一个csv文件
只需要打开一个文件，按照CSV的格式，编写内容，然后写入文件中去即可。
```python
from pathlib import Path

p = Path(r'E:\Python - base - code\chapter06文件\test.csv')
p.touch()
csv_body = '''\
no,name,age,comment
1,daxin,20,"I Like Cat"
2,tom,30,"I Like Movice"
3,ben,40,"I""m super man    # "" 表示"
4xyz"
'''

with open(p,'a+') as f:
    f.write(csv_body)
```

## 1.2 cvs模块
Python提供的csv模块用于处理csv文件，主要通过`读(reader)`和`写(writer)`两种方式：

### 1.2.1 reader方法
csv模块的reader方法，用于读模式打开一个csv文件，返回一个`reader对象`，是一个`行迭代器`。它的基本格式为：
```python
csv.reader(iterable, dialect='excel',**fmtparams)
```
- `dialect`：表示使用excel的方言(标准)去读取
- `**fmtparams`: 对excel的规范进行自定义设置

可以设置的参数有：
1. delimiter：列分隔符(逗号)
2. lineterminator：行分隔符\r\n
3. quotechar：字段的引用符号，缺省为 " 双引号
4. 双引号的处理(`一般使用doublequote=True`，按照默认的来)
    - doublequote 双引号的处理，默认为True。如果碰到数据中有双引号，而quotechar也是双引号，doublequote=True时，则使用2个双引号表示，False表示使用转义字符将作为双引号的前缀
    - escapechar 一个转义字符，默认为None
    - writer = csv.writer(f, doublequote=False, escapechar='@') 遇到双引号，则必须提供转义字符
5. quoting 指定双引号的规则
    - QUOTE_ALL 所有字段
    - `QUOTE_MINIMAL`特殊字符字段，Excel方言使用该规则
    - QUOTE_NONNUMERIC非数字字段
    - QUOTE_NONE都不使用引号。

> 建议都是用默认规则
```python
from pathlib import Path
import csv

p = Path(r'E:\Python - base - code\chapter06文件\test.csv')
with open(p, 'r') as f:
    reader = csv.reader(f)
    print(next(reader))  # ['no', 'name', 'age', 'comment']  逗号分隔好的一行列表
    for line in reader:
        print(line)

# ['no', 'name', 'age', 'comment']
# ['1', 'daxin', '20', 'I Like Cat']
# ['2', 'tom', '30', 'I Like Movice']
# ['3', 'ben', '40', 'I"m super man\n4xyz']
```

### 1.2.2 writer方法
csv模块的writer方法，用于写模式打开一个csv文件，返回一个`DictWriter对象`。它的基本格式为：
```python
writer(fileobj, dialect='excel', *args, **kwargs)：
```
它包含两个主要的方法：
- `writerow(row)`：写一行
- `writerows(rows)`：写多行

注意，这里的row需要是一个列表

```python
from pathlib import Path
import csv

p = Path(r'E:\Python - base - code\chapter06文件\test.csv')

rows = [
    ['1', 'daxin', '20', 'I Like Cat'],
    ['2', 'tom', '30', 'I Like Movice'],
    ['3', 'ben', '40', 'I"m super man\n4xyz']
]

with open(p, 'a+') as f:
    writer = csv.writer(f)
    writer.writerow(rows[0])  # 写一行
    writer.writerows(rows)    # 写多行
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;观察生成好的csv文件，发现每一行，后面都会多一行空行，这是因为在写的时候把一行文本的结尾\r\n转换成两个回车写入的，所以官方建议我们在打开文件的时候，添加`newline=''`参数，来组织替换。读取时也可以使用newline参数(不会影响原来的读取效果)
```python
with open(p, 'a+', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(rows[0])  # 写一行
    writer.writerows(rows)    # 写多行
```

# 2 ini文件处理
作为配置文件，ini文件格式很流行。著名的配置文件还有：json,yaml,xml等。下面是一个ini格式的文件(mariadb就是用的是这种格式的配置文件)
```ini
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

```
其中：
- `中括号`里面的部分称为`section`，译作节、区、段。
- 每一个section内，都是key=value形成的键值对，`key`称为`option`选项。


## 2.1 configparser模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;configparser模块的ConfigParser类就是用来操作ini文件的。可以将section当做key，section存储着键值对组成的字典，可以把ini配置文件当做一个嵌套的字典。默认使用的是有序字典。  


> 注意：在Python中的configparser库中，DEFAULT是缺省section的名字，必须大写，必须存在。当其他section中不存在某个option中时，会在DEFAULT中寻找对应的option，如果不存在则提示异常或者调用fallback。


ConfigParser对象中有很多方法，其中与读取配置文件，判断配置相关的方法有：
- `read`: 读取一个ini配置文件
- `sections`：返回一个包含所有章节的列表
- `has_sections`：判断章节是否存在
- `items`：以元祖的形式返回所有的选项
- `options`：返回一个包含章节下所有选项的列表
- `has_option`：判读某个选项是否存在
- `get`、`getboolean`、`getinit`、`getfloat`：获取选项的值

ConfigParser对象也提供了许多方法便于我们修改配置文件：
- `remove_section`：删除一个章节
- `add_section`：添加一个章节
- `remote_option`：删除一个选项
- `set`：添加一个选项(__写入的所有值，都是字符串__)
- `write`：将ConfigParser对象中保存的数据保存的文件中去

## 2.2 常用方法

### 2.2.1 读取配置配件 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解析一个配置文件，需要先创建一个ConfigParser对象，创建ConfigParser对象时有多个参数，其中比较重要的是`allow_no_value`，allow_no_value默认取值为`False`，表示在配置文件中是否允许有没有选项的值的情况。默认情况下每个选项都应该有一个值，但是在一些特殊的应用选项下，选项存在即为真，不存在即为假，比如MySQL的配置文件skip-external-locking。所以如果需要解析这样的参数，那么就需要在实例化的时候添加allow_no_value 为True
```python
config.read(filenames, encoding=None,allow_no_value=False)：
```
读取ini文件，可以是单个文件，也可以是文件列表。可以指定文件编码。

### 2.2.2 section操作
- sections() 返回section列表。缺省section不包括在内。
- add_section(section_name) 增加一个section。
- has_section(section_name) 判断section是否存在
- remove_section(section)：移除section及其所有option

### 2.2.3 option操作
- options(section) 返回section的所有option，会追加缺省section的option
- has_option(section, option) 判断section是否存在这个option
- items(section， raw=False, vars=None)：没有section，则返回所有section名字及其对象，如果指定section，则返回这个指定的section的键值对组成二元组。
- remove_option(section, option)：移除section下的option。

### 2.2.4 获取value
- get(section, option, *, raw=False, vars=None[, fallback]):从指定的段的选项上取值，如果找到返回，如果没有找到就去找DEFAULT段有没有，如果还没有，则看fallback是否指定，没有则直接抛出异常，否则返回fallback。
- getint(section, option, *, raw=False, vars=None[, fallback])：同get方式，结果会使用int函数转换
- getfloat(section, option, *, raw=False, vars=None[, fallback])：同get方式，结果会使用float函数转换
- getboolean(section, option, *, raw=False, vars=None[, fallback])：同get方式，结果会使用bool函数转换

### 2.2.5 设置value
- set(section, option, value)：section存在的情况下，写入option=value，要求option、value必须是字符串。

### 2.2.6 保存修改后的配置文件
- write(fileobject, space_around_delimiters=True)：将当前config的所有内容写入fileobject中，一般open函数使用w模式。

## 2.3 字典的访问方式
对Configparser对象的操作还可以使用字典的方式：
```python
from configparser import ConfigParser
from pathlib import Path

p = Path('mysql.ini')

config = ConfigParser()
config.read(p)

config['daxin'] = {'name':'daxin','age':20}  # 新建一个section
print(config['daxin']['name'])  # daxin
```
> 在Configparser内部，其实使用的就是一个OrderDict存储的













