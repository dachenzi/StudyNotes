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
#### 全局方法及判断方法
- `cwd()`: 返回当前工作目录
- `home()`: 返回当前家目录
- `is_dir()`: 是否是目录，是目录且存在，则返回True
- `is_file()`: 是否是普通文件，是文件且存在，则返回True
- `is_symlink()`: 是否是阮链接
- `is_socket()`: 是否是socket文件
- `is_block_device()`: 是否是块设备
- `is_char_device()`: 是否是字符设备
- `is_absolute()`: 是否是绝对路径
```python
In [34]: p = Path('/etc','sysconfig')                                            

In [35]: p2 = Path('/etc','hosts')                                               

In [36]: p3 = Path('/etc','rc.d','rc3.d','S10network')                           

In [37]: p.is_dir()                                                              
Out[37]: True

In [38]: p1.is_file()                                                            
Out[38]: False

In [39]: p2.is_file()                                                            
Out[39]: True

In [44]: p3.is_symlink()                                                         
Out[44]: True

In [45]: p.is_absolute()                                                         
Out[45]: True 
```
- `resolve()`: 返回当前Path对象的绝对路径。如果是软连接，则直接被解析
- `absolute()`: 获取Path对象的绝对路径
```python
In [28]: p3 = Path('hosts')                                                      

In [29]: p3                                                                      
Out[29]: PosixPath('hosts')

In [30]: p3.resolve()          # 软链接的真正路径                                                    
Out[30]: PosixPath('/etc/hosts')

In [31]: p3.absolute()         # 软链接的绝对路径                                                 
Out[31]: PosixPath('/home/python/py368/hosts')
```
- `exists()`: 文件或者目录是否存在
- `rmdir()`: 删除空目录(没有提供目录为空的方法)
- `touch(mode=0o666，exist_ok=False)`: 创建一个文件
    * `mode`: 文件的属性，默认为666
    * `exist_ok`: 在3.5版本加入，False时，路径存在，抛出FileExistsError;True时，异常将被忽略
```python
In [47]: p = Path('/tmp','hello.py')                                                                                                         
In [49]: p.exists()                                                              
Out[49]: False

In [50]: p.touch(mode=0o666,exist_ok=False)                                      

In [51]: p.exists()                                                              
Out[51]: True

In [52]: p.touch(mode=0o666,exist_ok=False)                                      
---------------------------------------------------------------------------
FileExistsError                           Traceback (most recent call last)
...
FileExistsError: [Errno 17] File exists: '/tmp/hello.py'

In [53]:  
```
- as_uri(): 将路径返回成URI
```python
In [56]: p2.as_uri()                                                             
Out[56]: 'file:///etc/hosts'
```
- `mkdir(mode=0o777,parents=False,exist_ok=False)`: 创建一个目录
    * `parents`：是否创建父目录，True等同于`mkdir -p`, False时，父目录不存在曝出FileNotFoundError
    * `exist_ok`: 在3.5版本加入，False时，路径存在，抛出FileExistsError;True时，异常将被忽略
- `iterdir()`: 迭代当前目录，不递归。
```python
In [74]: for x in p4.parents[0].iterdir(): 
    ...:     if x.is_dir(): 
    ...:         flag = False 
    ...:         for _ in x.iterdir(): 
    ...:             flag = True 
    ...:             break 
    ...:         print('dir: {} , is {}'.format(x,'not empty ' if flag else 'empt
    ...: y' )) 
    ...:     elif x.is_file(): 
    ...:         print('{} is a file'.format(x)) 
    ...:     else: 
    ...:         print('other file')
```
> 判断文件类型，当文件为目录时，判断其是否为空目录。
#### 通配符
- `glob(partten)`: 在`目录下`通配给定的格式
- `rglob(partten)`: 在`目录下`递归通配给定的格式(递归目录)  
- `match(partten)`: 模式匹配(对当前Path对象进行匹配)，成功返回True
支持的通配符与Linux下相同：
- ？ 表示一个字符
- \* 表示任意个字符
- [abc]或[a-z] 表示区间内的任意一个字符
```python
In [84]: p4                                                                      
Out[84]: PosixPath('/etc/sysconfig/network-scripts')

In [85]: list(p4.glob('ifu?-*'))                                                 
Out[85]: 
[PosixPath('/etc/sysconfig/network-scripts/ifup-aliases'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-bnep'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-eth'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-ippp'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-ipv6'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-isdn'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-plip'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-plusb'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-post'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-ppp'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-routes'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-sit'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-tunnel'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-wireless'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-ib'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-Team'),
 PosixPath('/etc/sysconfig/network-scripts/ifup-TeamPort')]

In [87]: p4.match('/etc/*/network-script?')                                      
Out[87]: True
```
#### 目录属性
- `stat()`: 查看目录的详细信息，相当于stat命令
- `lstat()`: 如果是符号链接，则显示符号链接本身的文件信息
```python
In [88]: p4.stat()                                                               
Out[88]: os.stat_result(st_mode=16877, st_ino=67533402, st_dev=2050, st_nlink=2, st_uid=0, st_gid=0, st_size=4096, st_atime=1550229289, st_mtime=1545830238, st_ctime=1545830238)
```
### 1.2.2 文件操作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Path对象同样提供了打开文件的函数，功能类似于内建函数open。返回一个文件对象。当我们创建一个Path对象时，这个文件已经被打开，当我们写入数据时，文件不存在会新建，重名或者是目录，会有相应的异常提示，它的语法是
```python
Path.open(mode='r',buffering=-1,encoding=None,errors=None,newline=None)
```
例：
```python
In [115]: p5                                                                     
Out[115]: PosixPath('/tmp/123')

In [116]: p = p5.open(mode='r')                                                  

In [117]: p                                                                      
Out[117]: <_io.TextIOWrapper name='/tmp/123' mode='r' encoding='UTF-8'>

In [118]: p.read()                                                               
Out[118]: '123'

In [119]: p5.read_text()     # 不存在时报异常，存在则直接打开并读取                                                      
Out[119]: '123'

```
3.5以后新增加的函数方法：
- `Path.read_bytes()`: 以'rb'方式读取路径对应文件，并返回二进制流。
- `Path.read_text()`: 以'rt'方式读取路径文件， 并返回文件。__无视指针__
- `Path.write_bytes()`: 以'wb'方式写入数据到路径对应文件中。
- `Path.write_text()`: 以'wt'方式写入数据到路径对应文件中。
## 1.3 os 模块
os模块的常用方法：
```python
os.getcwd():        获取当前路径
os.chdir()：        切换当前目录，当路径中存在\的时候，由于是转意的意思，那么就需要对\进行转意，那么路径就是c:\\User,或者在目录前面加r，表示后面的字符串不进行解释
os.curdir():        获取当前目录名
os.pardir():        获取上级目录名
os.mkdir('dir'):    创建目录，注意只能创建一级目录
os.makedirs('dir_path'):创建多级目录
os.rmdir('dir'):    删除一个目录
os.removedir('dir_path')：删除多级目录（目录为空的话）
os.listdir('dir'):  显示目录下的所有文件,默认为当前目录,返回的结果为list
os.remove('file'):  删除一个文件
os.rename('old_name','new_name'):修改文件名称
os.stat('file/dir')：获取文件/目录的stat信息(调用的是系统的stat)
os.sep:             返回当前操作系统的路径分隔符(Windows下：\\ , Linux下:/）
os.linesep:         返回当前操作系统的换行符（Windows下:\r\n  ,Linux下:\n）
os.pathsep:         返回当前操作系统环境变量分隔符（Windows下是; ,Linux下是:）
os.name:            返回当前系统的类型（nt 表示Windows,  posix表示Linux）
os.system('Commmand'):执行命令
os.environ:         获取系统环境变量，使用字典存储
os.path.abspath('dir/file'):获取dir/file的绝对路径
os.path.split('path'):把路径分割为目录和文件名组成的元组格式，不管path是否存在
os.dirname('path')：获取文件的父目录名称，不管path是否存在
os.basename('path'):获取文件的名称，不管path是否存在
```
> os.stat(follow_symlinks=True)，返回源文件本身信息，False时，显示链接文件的信息,对于软连接本身，还可以使用os.lstat方法
```python
In [133]: os.lstat('hosts')                                                      
Out[133]: os.stat_result(st_mode=41471, st_ino=2083428, st_dev=2050, st_nlink=1, st_uid=1001, st_gid=1001, st_size=10, st_atime=1550259162, st_mtime=1550259161, st_ctime=1550259161)

In [134]: os.stat('hosts')                                                       
Out[134]: os.stat_result(st_mode=33188, st_ino=67245317, st_dev=2050, st_nlink=1, st_uid=0, st_gid=0, st_size=158, st_atime=1550229294, st_mtime=1370615492, st_ctime=1545666279)

In [136]: os.stat('hosts',follow_symlinks=False)      # 等同于os.lstat()                            
Out[136]: os.stat_result(st_mode=41471, st_ino=2083428, st_dev=2050, st_nlink=1, st_uid=1001, st_gid=1001, st_size=10, st_atime=1550259162, st_mtime=1550259161, st_ctime=1550259161)

In [137]:    
```
# 2 shutil模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据前面所学的知识，我们如果想要进行文件拷贝，需要先打开两个文件对象对象，源文件读取内容，写入到目标文件中去。 这种方式虽然完成了文件的拷贝，但是却丢失了文件的属性信息，比如属组、权限等，因为我们根本没有进行复制。所以，python提供了一个用于高级文件操作的库，它的名字就叫做shutil。
## 2.1 copy复制
- `shutil.copyfileobj(fsrc,fdes,length)`: 将文件内容拷贝到另一个文件中，可以只拷贝部分内容，需要我们自行打开文件对象进行copy,length表示buffer的大小，需要注意的是fdes必须可写
```python
>>> import os,shutil
>>> os.system('ls')
1.txt
>>> shutil.copyfileobj(open('1.txt'),open('2.txt','w'))
>>> os.system('ls')
1.txt  2.txt
>>> 
```
- `shutil.copyfile(fsrc,fdes)`: 复制文件，我们只需要传入文件名称即可进行复制，不用自行预先打开，等于创建一个新的文件，把老文件写入到新文件中然后关闭，新创建的文件权限和属主等信息遵循操作系统规定(本质上还是调用copyfileobj)
```python
>>> shutil.copyfile('1.txt','3.txt')
>>> os.system('ls')
1.txt  2.txt  3.txt
```
- `shutil.copymode(src,des)`: 复制文件权限，既把src文件的权限复制给 des文件，只改变权限，不改变其他比如属组，内容等(des文件必须存在)
```python
>>> os.system('ls -l')
total 12
-rwxrwxrwx 1 root root 6 Mar  9 18:35 1.txt
-rw-r--r-- 1 root root 6 Mar  9 18:36 2.txt
-rw-r--r-- 1 root root 6 Mar  9 18:38 3.txt
>>> shutil.copymode('1.txt','2.txt')
>>> os.system('ls -l')
total 12
-rwxrwxrwx 1 root root 6 Mar  9 18:35 1.txt
-rwxrwxrwx 1 root root 6 Mar  9 18:36 2.txt
-rw-r--r-- 1 root root 6 Mar  9 18:38 3.txt
>>> 
```
- `shutil.copystat(src,des)`: 复制文件的权限，还包括，atime，mtime，flags等信息，不改变文件内容（des需存在）
```python
>>> os.system('stat 1.txt')
  File: `1.txt'
  Size: 6             Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 926326      Links: 1
Access: (0777/-rwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2017-03-09 18:36:59.223738919 +0800
Modify: 2017-03-09 18:35:23.148738381 +0800
Change: 2017-03-09 18:39:59.061738605 +0800

>>> os.system('stat 3.txt')
  File: `3.txt'
  Size: 6             Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 940237      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2017-03-09 18:39:42.214738376 +0800
Modify: 2017-03-09 18:38:13.862738316 +0800
Change: 2017-03-09 18:38:13.862738316 +0800

>>> shutil.copystat('1.txt','3.txt')
>>> os.system('stat 3.txt')
  File: `3.txt'
  Size: 6             Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 940237      Links: 1
Access: (0777/-rwxrwxrwx)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2017-03-09 18:36:59.223738000 +0800
Modify: 2017-03-09 18:35:23.148738000 +0800
Change: 2017-03-09 18:44:33.286738354 +0800
>>> 
```
- `shutil.copy(src,des)`: 复制文件的同时复制权限信息,等同于执行了如下命令:
    1. __shutil.copyfile(src,dest,follow_symlinks=True)__ 
    2. __shutil.copymode(src,dest,follow_symlinks=True)__
- `shutil.copy2(src,des)`: 比copy对了全部原数据，但需要平台支持,等同于执行了如下命令:
    1. __shutil.copyfile(src,dest,follow_symlinks=True)__ 
    2. __shutil.copystat(src,dest,follow_symlinks=True)__
- `shutil.copytree(src,dest,symlinks=False,ignore=None,copy_function=copy2,ignore_dangling_symlinks=False)`: 递归复制文件，类似于copy -r，默认使用copy2
    - src必须存在，dest必须不存在。
> ignore = func, 提供一个callable(src,namnes) --> ignoted_names。提供一个函数，它会被调用。src是原目录，names是原目录下的文件列表(os.listdir(src))，返回值是要被过滤的文件名的set类型数据
```python
In [146]: def func(src,names): 
     ...:     ig = filter(lambda x: not x.endswith('conf'),names) 
     ...:     return set(ig)
     
In [164]: os.listdir('old')                                                      
Out[164]: 
['123.txt',
 '456.txt',
 'asound.conf',
 'brltty.conf',
 'chrony.conf',
 'dleyna-server-service.conf',
 'dnsmasq.conf',
 'dracut.conf',
 'e2fsck.conf',
 'fprintd.conf',
 'fuse.conf',
 'GeoIP.conf',
 'host.conf']

In [161]: shutil.copytree('old','new',ignore=func)                               
Out[161]: 'new'

In [163]: os.listdir('new')                                                      
Out[163]: ['123.txt', '456.txt']


```
> 

= shutil.ignore_patterns('*py')
复制代码
>>> os.system('ls -l')
total 4
drwxr-xr-x 2 root root 4096 Mar  9 18:46 test

>>> shutil.copytree('test','test1')
>>> os.system('ls -l')
total 8
drwxr-xr-x 2 root root 4096 Mar  9 18:46 test
drwxr-xr-x 2 root root 4096 Mar  9 18:46 test1

>>> os.system('ls -l test1')
total 12
-rwxrwxrwx 1 root root 6 Mar  9 18:35 1.txt
-rwxrwxrwx 1 root root 6 Mar  9 18:36 2.txt
-rwxrwxrwx 1 root root 6 Mar  9 18:35 3.txt
>>> 
复制代码
shutil.rmtree(des)  递归的删除文件，类似于　rm -rf


复制代码
>>> os.system('ls -l')
total 8
drwxr-xr-x 2 root root 4096 Mar  9 18:46 test
drwxr-xr-x 2 root root 4096 Mar  9 18:46 test1

>>> shutil.rmtree('test1')
>>> os.system('ls -l')
total 4
drwxr-xr-x 2 root root 4096 Mar  9 18:46 test
>>> 
复制代码
shutil.move(src,des)  移动文件/目录，类似于mv 命令

shutil.make_archive('data_bak','gztar',roots_dir = '/data' )  压缩，data_bak 可以写为绝对路径


复制代码
import tarfile
t = tarfile.open('data_bak.tar.gz') #打开我呢间
t.extractall('/tmp') #解压缩到某个路径
t.close()

#tar打包
t = tarfile.open('data_bak.tar','w')
t.add(r'/tmp/1.txt') -->　#会把程序的路径也进行打包
t.close()

#tar打包
t =tarfile.open('data_bak.tar.gz')
t.add('/tmp/1.txt',arcname='1.txt_bak') # 打包的时候把 /tmp/1.txt　打包成 1.txt_bak