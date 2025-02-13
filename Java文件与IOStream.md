# File

- File对象就表示一个路径，可以是文件的路径、也可以是文件夹的路径
- 这个路径可以是存在的，也允许是不存在的

| 方法名称                                 | 说明                                               |
| ---------------------------------------- | -------------------------------------------------- |
| public File(String pathname)             | 根据文件路径创建文件对象                           |
| public File(String parent, String child) | 根据文件父路径字符串和子路径名字符串创建文件对象   |
| public File(File parent, String child)   | 根据父路径对应文件对象和子路径名字符串创建文件对象 |



## 绝对路径与相对路径

- 绝对路径是带盘符的
- 相对路径是不带盘符的，默认到当前项目下找



## File的常见成员方法

| 方法名称                                       | 说明                                       |
| ---------------------------------------------- | ------------------------------------------ |
| public static File[] listRoots()               | 列出可用的文件系统根                       |
| public String[] list()                         | 获取当前该路径下所有内容                   |
| public String[] list(FilenameFilter filter)    | 利用文件名过滤器获取当前该路径下的所有内容 |
| public File[] listFiles()                      | 获取当前该路径下所有内容                   |
| public File[] listFiles(FileFilter filter)     | 利用文件名过滤器获取当前该路径下所有内容   |
| public File[] listFiles(FilenameFilter filter) | 利用文件名过滤器获取当前该路径下所有内容   |

| 方法名称                        | 说明                                 |
| ------------------------------- | ------------------------------------ |
| public boolean isDirectory()    | 判断此路径名表示的File是否为文件夹   |
| public boolean isFile()         | 判断此路径名表示的File是否为文件     |
| public boolean exists()         | 判断此路径名表示的File是否存在       |
| public long length()            | 返回文件的大小（字节数量）           |
| public String getAbsoultePath() | 返回文件的绝对路径                   |
| public String getPath()         | 返回定义文件时使用的路径             |
| public String getName()         | 返回文件的名称，带后缀               |
| public long lastModified()      | 返回文件的最后修改时间（时间毫秒值） |

| 方法名称                       | 说明                 |
| ------------------------------ | -------------------- |
| public boolean createNewFile() | 创建一个新的空文件夹 |
| public boolean mkdir()         | 创建单级文件夹       |
| public boolean mkdirs()        | 创建多级文件夹       |
| public boolean delete()        | 删除文件、空文件夹   |

delete方法默认只能删除文件和空文件夹，而且直接删除不走回收站。



# IOStream

IO流：用于读写文件中的数据（可以读写文件，或网络中的数据）

流的方向：

- 输入流：读取
- 输出流：写出

操作文件类型：

- 字节流：所有类型的文件
  - 输入流：InputStream
  - 输出流：OutputStream

- 字符流：纯文本文件

