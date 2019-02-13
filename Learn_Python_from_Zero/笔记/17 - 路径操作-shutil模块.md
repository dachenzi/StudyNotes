# 1 路径操作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用Python操作文件系统时,少不了会对路径进行切换,对目录的遍历,以及获取文件的绝对路径的一系列的操作,Python内置了相关的模块完成对应的功能,其中:
- 3.4 以前使用os.path模块
- 3.4 开始使用pathlib模块
## 1.1 os.path模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;os.path是os模块中的一个比较重要的用来拼接、判断路径的主要方法，它主要有如下方法：
```python
os.path.abspath('dir/file'):获取dir/file的绝对路径
os.path.split('path'):把路径分割为目录和文件名组成的元组格式，不管path是否存在
os.dirname('path')：获取文件的父目录名称，不管path是否存在
os.basename('path'):获取文件的名称，不管path是否存在
os.path.exists('path'):判断path是否存在,return bool
os.path.isabs('path')：判断path是否是从根开始,return bool
Linux下：从/开始    Windows下从C,D,E盘开始
os.path.isfile('path'):判断path是否是一个文件
os.path.isdir('path')：判断path是否是一个目录
os.path.join('path1','path2','path3')：把path1和path2及path3进行组合，但如果path2中包含了根路径，那么就会舍弃path1,从path2开始组合
os.path.getatime('path'):获取文件的atime时间，返回时间戳
os.path.getmtime('path')：获取文件的mtime时间，返回时间戳
os.path.getsize(filename)： 获取文件的大小，单位是字节
```