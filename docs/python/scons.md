### 基本使用
SConstruct文件就功能而言相当于Makefile文件，就内容而言则是Python脚本，scons读入它时，会把脚本的命令都执行一遍，但具体编译过程则有scons自己相机而定。
``` python
Program('hello.c')
#前为目标，后为源文件
Program('new_hello','hello_c')  
#多个源文件
Program(['prog.c','file1.c','file2.c']) 
#指定目标文件，多个源文件
Program(‘program'，['prog.c','file1.c','file2.c'])   
Program('program‘，Glob('*.c'))
#多个原文件更清晰的写法
Program('program',Split('main.c file.c file2.c'))  
src_files=Split("""main.c file1.c file2.c""")
Program(target='program',source=src_files)
```

### 编译多个目标文件就多次调用Program()

### 多个程序共享源文件
``` python
eg.1  
       Program(Split('foo.c common1.c common2.c'))
       Program('bar', Split('bar1.c bar2.c common1.c common2.c'))
eg.2    
       common = ['common1.c', 'common2.c']
       foo_files = ['foo.c'] + common
       bar_files = ['bar1.c', 'bar2.c'] + common
       Program('foo', foo_files)
       Program('bar', bar_files)
```
### 库编译和链接
#### 库编译
    Library('foo', ['f1.c', 'f2.c', 'f3.c'])
#####  你甚至可以在文件List里混用源文件和.o文件
    Library('foo', ['f1.c', 'f2.o', 'f3.c', 'f4.o'])
#####  显式编译静态库
    StaticLibrary('foo', ['f1.c', 'f2.c', 'f3.c'])
##### 编译动态库
    SharedLibrary('foo', ['f1.c', 'f2.c', 'f3.c'])
#### 库链接
    Library('foo', ['f1.c', 'f2.c', 'f3.c'])
        Program('prog.c', LIBS=['foo', 'bar'], LIBPATH='.')
    不用指明库的前缀和后缀，如lib,.a等。
#### 找到库：$LIBPATH变量
      Program('prog.c', LIBS = 'm',
                        LIBPATH = ['/usr/lib', '/usr/local/lib'])
    Unix上     LIBPATH = '/usr/lib:/usr/local/lib'
    Windows上  LIBPATH = 'C:\\lib;D:\\lib'

### 节点对象（nodes）
 SCons把所有知道的文件和目录表示为节点。
#### builder方法的返回值都是节点对象List,这一点很重要，在python中List+List还是List，下面可以看到应用。
      hello_list = Object('hello.c', CCFLAGS='-DHELLO')
      goodbye_list = Object('goodbye.c', CCFLAGS='-DGOODBYE')
      Program(hello_list + goodbye_list)
    这里如果不使用变量的话，就不能跨平台使用了，因为不同系统object扩展名是不同的

#### 显示创建文件和目录节点
      hello_c = File('hello.c')
      Program(hello_c)
      classes = Dir('classes')
      Java(classes, 'src')
    一般，你不用显式调用他们，因为builder方法会自动处理文件名字符串并转换为节点的。
    当你不知道目标是文件还是目录时：
    xyzzy = Entry('xyzzy')

#### 打印节点文件名
      hello_c = File('hello.c')
      Program(hello_c)

      classes = Dir('classes')
      Java(classes, 'src')

      object_list = Object('hello.c')
      program_list = Program(object_list)
      print "The object file is:", object_list[0]
      print "The program file is:", program_list[0]
#### 把节点文件名当作字符串来使用
      imp

ort os.path
      program_list = Program('hello.c')
      program_name = str(program_list[0])
      if not os.path.exists(program_name):
          print program_name, "does not exist!"
    此例判断了文件是否存在


### 依赖关系
#### Decider函数
确定文件是否被修改
#### 默认使用MD5检查源文件内容而不是时间戳，用touch不算修改过
      显示调用
    Program('hello.c')
        Decider('MD5')
#### 使用时间戳
        Program('hello.c')
        Decider('timestamp-newer')
      与下面语句等价
        Program('hello.c')
        Decider('make')
      更详细的检查
        Program('hello.c')
        Decider('timestamp-match')
#### 两者同时使用
        Program('hello.c')
        Decider('MD5-timestamp')
#### $CPPPATH 隐含依赖关系
       Program('hello.c', CPPPATH = ['include', '/home/project/inc'])
    会在编译时加 -I

#### 缓冲隐式依赖关系
     % scons -Q --implicit-cache hello  运行选项
    SetOption('implicit_cache', 1)      写在脚本里
    这样下次就不用从头开始找依赖关系了
         % scons -Q --implicit-deps-changed hello
    再打开

#### 显式依赖关系： Depends函数
    当Scons没有找到关系时
       hello = Program('hello.c')
       Depends(hello, 'other_file')
####    总是编译 
      hello = Program('hello.c')
      AlwaysBuild(hello)

### 环境
#### 创建环境
#####
     imp ort os

         env = Environment(CC = 'gcc',
                           CCFLAGS = '-O2')

         env.Program('foo.c')
 % scons -Q
##### 获取环境变量
         env = Environment()
         print "CC is:", env['CC']
         % scons -Q
         CC is: cc
         scons: `.' is up to date.
         env = Environment(FOO = 'foo', BAR = 'bar')
         dict = env.Dictionary()
         for key in ['OBJSUFFIX', 'LIBSUFFIX', 'PROGSUFFIX']:
             print "key = %s, value = %s" % (key, dict[key])
      

This SConstruct file will print the specified dictionary items for us on POSIX systems as follows:

         % scons -Q
         key = OBJSUFFIX, value = .o
         key = LIBSUFFIX, value = .a
         key = PROGSUFFIX, value = 
         scons: `.' is up to date.
### Install安装文件至其他文件夹
     env = Environment()
     hello = env.Program('hello.c')
     env.Install('/usr/bin', hello)
#### 在目录里安装多个文件
       env = Environment()
       hello = env.Program('hello.c')
       goodbye = env.Program('goodbye.c')
       env.Install('/usr/bin', [hello, goodbye])
       env.Alias('install', '/usr/bin')
### 一个文件不同存放
       env = Environment()
       hello = env.Program('hello.c')
       env.InstallAs('/usr/bin/hello-new', hello)
       env.Alias('install', '/usr/bin')

       % scons -Q install
       cc -o hello.o -c hello.c
       cc -o hello hello.o
       Install file: "hello" as "/usr/bin/hello-new"

### 平台独立的文件系统操作
#### 复制
        Command("file.out", "file.in", Copy("$TARGET", "$SOURCE"))
     或者
        Command("file.out", [], Copy("$TARGET", "file.in"))

### 多层目录构建
Sconconstruct在最顶层
#### SConscript文件可以层层包含
      SConscript(['drivers/SConscript',
                  'parser/SConscript',
                  'utilities/SConscript'])
在drivers下：
      SConscript(['display/SConscript',
                  'mouse/SConscript'])
#### 对文件SConscript的相对目录
    SConscript(['prog1/SConscript',
                  'prog2/SConscript'])
