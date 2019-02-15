# 1 路径操作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用Python操作文件系统时,少不了会对路径进行切换,对目录的遍历,以及获取文件的绝对路径的一系列的操作,Python内置了相关的模块完成对应的功能,其中:
- 3.4 以前使用os.path模块
- 3.4 开始使用pathlib模块
## 1.1 os.path模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;os.path是os模块中的一个比较重要的用来拼接、判断路径的主要方法，它主要有如下方法：
```python
os.path.abspath('dir/file')    获取dir/file的绝对路径
os.path.split('path')          把路径分割为目录和文件名组成的元组格式，不管path是否存在
os.dirname('path')             获取文件的父目录名称，不管path是否存在
os.basename('path')            获取文件的名称，不管path是否存在
os.path.exists('path')         判断path是否存在,return bool
os.path.isabs('path')          判断path是否是从根开始,return bool
os.path.isfile('path')         判断path是否是一个文件
os.path.isdir('path')          判断path是否是一个目录
os.path.join('path1','path2','path3')：把path1和path2及path3进行组合，但如果path2中包含了根路径，那么就会舍弃path1,从path2开始组合
os.path.getatime('path')       获取文件的atime时间，返回时间戳
os.path.getmtime('path')       获取文件的mtime时间，返回时间戳
os.path.getsize(filename)      获取文件的大小，单位是字节
```
> Linux下：从/开始,Windows下从C,D,E盘开始
```python
In [1]: import os                                                            

In [2]: os.path.join('/etc','sysconfig','network-scripts')                   
Out[2]: '/etc/sysconfig/network-scripts'

In [3]: os.path.join('/etc','/sysconfig','network-scripts')                  
Out[3]: '/sysconfig/network-scripts'

In [8]: p = os.path.join('/etc','sysconfig','network-scripts')               

In [9]: p                                                                    
Out[9]: '/etc/sysconfig/network-scripts'

In [10]: type(p)                                                             
Out[10]: str

In [12]: os.path.exists(p)                                                   
Out[12]: True

In [13]: os.path.split(p)                                                    
Out[13]: ('/etc/sysconfig', 'network-scripts')

In [14]: os.path.abspath('.')                                                
Out[14]: '/home/python/py368'

In [16]: os.path.dirname(p)                                                  
Out[16]: '/etc/sysconfig'

In [17]: os.path.basename(p)                                                 
Out[17]: 'network-scripts'

>>> os.path.splitdrive('c:/etc/sysconfig/network-script')   # windows 
('c:', '/etc/sysconfig/network-script')
```
> \_\_file\_\_：变量比较特殊，存放的是当前的Python文件的名称，我们可以使用os.path.abspath(__file__)来获取当前python文件的绝对路径，然后进行打包或者进行相对调用。
## 1.2 pathlib模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.4以后建议使用pathlib模块，它提供Path对象来对路径进行操作，还包括了目录和文件。
> 在使用时需要实现导入： from pathlib import Path
### 1.2.1 目录操作
下面来说一下日常的目录相关操作
#### 初始化(一个路径对象)
通过构建一个Path对象来对路径进行初始化
```python

In [19]: from pathlib import Path                                            
In [20]: p = Path()  # 当前目录                                              
In [21]: p1 = Path('a','b','c')   # 当前目录下的a/b/c                                            
In [22]: p2 = Path('/etc','sysconfig','network-scripts')  # /etc/sysconfig/network-scripts

In [23]: p                                                                   
Out[23]: PosixPath('.')

In [24]: p1                                                                  
Out[24]: PosixPath('a/b/c')

In [25]: p2                                                                  
Out[25]: PosixPath('/etc/sysconfig/network-scripts')

```
#### 路径拼接和分解
`/`: Path对象支持使用`/`来进行路径的拼接，拼接规则应遵循：
- Path对象 / Path对象
- Path对象 / 字符串 或者 字符串 / Path对象
```python
In [30]: p2 / 'ifcfg-eth0'                                                   
Out[30]: PosixPath('/etc/sysconfig/network-scripts/ifcfg-eth0')

In [35]: p2 / p1                                                             
Out[35]: PosixPath('/etc/sysconfig/network-scripts/a/b/c')

In [37]: '/root' / p2                                                        
Out[37]: PosixPath('/etc/sysconfig/network-scripts')
```
需要注意的是：
1. 拼接的顺序和路径的顺序是一致的
1. p1 是相对路径，而p2是从根开始，如果p1/p2 那么返回的将会是p2
2. 如果当 字符串 / Path对象这种格式时，如果Path对象是一个绝对路径，那么将无法进行拼接  

`parts属性`: 将Path对象按照当前操作系统的分隔符进行分割返回一个元组
```python
In [39]: p2.parts                                                            
Out[39]: ('/', 'etc', 'sysconfig', 'network-scripts')
```
`joinpath(*other)`: 在Path对象中使用当前操作系统的路径分隔符分割并追加多个字符串。 
```python
In [43]: p2.joinpath('/etc')                                                 
Out[43]: PosixPath('/etc')

In [44]: p2.joinpath('etc')                                                  
Out[44]: PosixPath('/etc/sysconfig/network-scripts/etc')

In [50]: p2.joinpath('/etc','/proc')                                        
Out[50]: PosixPath('/proc')
```
需要注意的是：
1. 如果添加的的路径都为/开始，那么最后添加的路径将会覆盖前面所有添加的路径字符串
#### 获取路径
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Path返回的是一个路径对象，那么如何才可以只打印路径的字符串格式呢，我们可以通过`str(Path对象)`进行转换，当需要bytes格式时，也可以使用`bytes(Path对象)`转换。
```python
In [51]: bytes(p2)                                                           
Out[51]: b'/etc/sysconfig/network-scripts'

In [52]: str(p2)                                                             
Out[52]: '/etc/sysconfig/network-scripts'
```
#### 父目录
`parent`: 当前目录的逻辑父目录  
`parents`: 所以父目录的序列，索引0时为当前目录的父目录，依次类推
```python
In [53]: p2.parent                                                           
Out[53]: PosixPath('/etc/sysconfig')

In [54]: p2.parent.parent                                                    
Out[54]: PosixPath('/etc')

In [55]: p2.parent.parent.parent    # 链式操作                                         
Out[55]: PosixPath('/')

In [57]: p2.parents                                                          
Out[57]: <PosixPath.parents>   # 可迭代对象

In [58]: list(p2.parents)                                                    
Out[58]: [PosixPath('/etc/sysconfig'), PosixPath('/etc'), PosixPath('/')]

In [59]:     
```
> parent属性，看似支持类js的链式操作，主要还是因为每次使用parent属性时，返回的还是一个Path对象，所以才可以一直parent下去。
#### 目录的组成部分
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在一个目录的绝对路径中我们可能会单独使用目录的名称、目录的后缀名等等，Path对象提供了专门的属性及方法便于获取或者对它们进行修改。
- `name`: 目录的最后一个部分
- `suffix`：目录中最后一个部分的扩展名
- `stem`：目录中最后一个部分，不包含后缀名
- `suffixes`: 多个后缀名形成的列表
```python
In [1]: from pathlib import Path                                             

In [2]: p = Path('/tmp/mysql.tar.gz')                                        

In [3]: p.name                                                               
Out[3]: 'mysql.tar.gz'

In [4]: p.suffix                                                             
Out[4]: '.gz'

In [5]: p.stem                                                               
Out[5]: 'mysql.tar'

In [6]: p.suffixes                                                           
Out[6]: ['.tar', '.gz']
```
- `with_suffix(suffix)`: 有扩展名则替换，无则补充扩展名(注意后缀名要加点)
- `with_name(name)`：替换目录最后一个部分并返回一个新的路径
```python
In [9]: p                                                                    
Out[9]: PosixPath('/tmp/mysql.tar.gz')

In [10]: p.with_suffix('.abc')                                               
Out[10]: PosixPath('/tmp/mysql.tar.abc')

In [11]: p1 = Path('/tmp/nginx')                                             

In [12]: p1.with_suffix('.abc')                                              
Out[12]: PosixPath('/tmp/nginx.abc')

In [15]: p                                                                   
Out[15]: PosixPath('/tmp/mysql.tar.gz')

In [16]: p.with_name('nginx.tar.gz')                                         
Out[16]: PosixPath('/tmp/nginx.tar.gz')
 

```