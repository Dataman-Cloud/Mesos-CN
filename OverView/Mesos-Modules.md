##Mesos Modules
Mesos module 在 Mesos 0.21.0版本中被引用。
###什么是 Mesos Modules
Mesos module 提供了一种通过创建和按需使用共享库来轻松扩展 Mesos 内部工作。 Module 可以定制 Mesos 而无需通过重新编译/链接每个特定的实例。 Module 可以把外部依赖放到单独的库中，从而精简 Mesos 核心。 Module 还可以非常容易的尝试新的功能。例如，假设加载配置包含一个虚拟机( Lua, Python, … )，可以尝试编写新的配置脚本，而不强迫这些以来关系进入到项目中。最后，Module 提供一种简单的方法为第三方轻松扩展 Mesos ，而不必知道具体的内部细节。
###调用  Mesos Modules
命令行使用 ***--modules*** 可用于 Mesos master, slave 以及 tests 以指定模块列表被加载并提供给内部子系统。

使用 ***--modules=filepath*** 通过包含指定模块列表的JSON格式字符串文件。***filepath*** 格式可以是‘ file:///path/to/file ’ 或者 ‘ /path/to/file ’。

使用 ***--modules="{...}"*** 指定的模块内部名单。

#### JSON strings 例子:

1. 加载一个包含 ***org_apache_mesos_bar*** 和 ***org_apache_mesos_baz*** 两个模块的 ***libfoo.so*** 库文件
```
 {
   "libraries": [
     {
       "file": "/path/to/libfoo.so",
       "modules": [
         {
           "name": "org_apache_mesos_bar",
         },
         {
           "name": "org_apache_mesos_baz"
         }
       ]
     }
   ]
 }
```

2. 通过 ***foo*** 库文件指定模块  ***org_apache_mesos_bar*** 的 key/valuse 参数为 X/Y  ( ***org_apache_mesos_baz*** 模块不包含任何参数加载)：
```
 {
   "libraries": [
     {
       "name": "foo",
       "modules": [
         {
           "name": "org_apache_mesos_bar"
           "parameters": [
             {
               "key": "X",
               "value": "Y",
             }
           ]
         },
         {
           "name": "org_apache_mesos_baz"
         }
       ]
     }
   ]
 }
```

3.在命令行中指定：
```
 --modules='{"libraries":[{"file":"/path/to/libfoo.so", "modules":[{"name":"org_apache_mesos_bar"}]}]}'
```
####库名称

每个库，至少有一个" file "或者" path "在参数中被指定。" file "参数可以是文件名称(如"  libfoo.so ")，相对路径(如" myLibs/libfoo.so ")，或者绝对路径(如" /home/mesos/lib/libfoo.so ")。
参数" name "指的是一个库名称(如" foo ")。如果指定了" name ",它会自动将匹配当前平台的扩展名(如，" foo "在 Linux 上扩展为    libfoo.so ,在 OS X 则为" libfoo.dylib ")。

如果库没有指定" file "参数，该库会搜索标准库的路径或目录指向的环境变量 ***LD\_LIBRARY\_PATH*** (OS X 则为 ***DYLD\_LIBRARY\_PATH***)。

如果" file "和" name "同时指定了，则" name "会被忽略。

###支持哪些类型的模块？
下面是目前可用的各种模块的种类。

####分配器( Allocator )

Mesos master 的分配器会定期将资源分配给某个framework。
要加载一个自定义的分配器到 Mesos master 中，需要如下：

- 采用它的 Mesos master 清单中的 ***--modules*** 配置。

- 通过 ***--allocator*** 标志将其声明成分配器

例如，下面的命令将通过 ***ExternalAllocatorModule*** 运行 Mesos master：
```
./bin/mesos-master.sh --work_dir=m/work --modules="file://<modules-including-allocator>.json" --allocator=ExternalAllocatorModule
```

####Anonymous
Anonymous 模块不接收任何回调，只是与它们的父进程共存。
Anonymous 模块不需要任何特定的选择器( flags )，他们会立即实例化通过 ***--modules*** 标志在 Mesos master 或者 slave。

####Hook
待翻译

####隔离(Isolator)
Isolator 模块可以与专门的隔离与监控功能交互。

###编写 Mesos modules

####A HelloWorld module
下面代码片段描述了" TestModule "中的" org_apache_mesos_bar "模块的实现：

```
#include <iostream>
#include "test_module.hpp"

class TestModuleImpl : public TestModule
{
public:
  TestModuleImpl()
  {
    std::cout << "HelloWorld!" << std::endl;
  }

  virtual int foo(char a, long b)
  {
    return a + b;
  }

  virtual int bar(float a, double b)
  {
    return a * b;
  }
};

static TestModule* create()
{
    return new TestModule();
}

static bool compatible()
{
  return true;
}

// Declares a module named 'org_apache_mesos_TestModule' of
// 'TestModule' kind.
// Mesos core binds the module instance pointer as needed.
// The compatible() hook is provided by the module for compatibility checks.
// The create() hook returns an object of type 'TestModule'.
mesos::modules::Module<TestModule> org_apache_mesos_TestModule(
    MESOS_MODULE_API_VERSION,
    MESOS_VERSION,
    "Apache Mesos",
    "modules@mesos.apache.org",
    "This is a test module.",
    compatible,
    create);
```

####构建模块

下面假设 Mesos 安装在标准位置，即 Mesos 动态库和头文件是可用的。
```
G ++ -lmesos -fpic -o test_module.o test_module.cpp
$ GCC -shared -o libtest_module.so test_module.o
```
####测试模块

下面的命令将测试 Mesos 与 org_apache_mesos_TestCpuIsolator 隔离模块：
```
./bin/mesos-tests.sh --modules="/home/kapil/mesos/isolator-module/modules.json" --isolation="org_apache_mesos_TestCpuIsolator"
``` 

####模块的命名规则
每个模块的名称都应该是唯一的。在 Json String中有相同模块名字会引起进程的异常终止。

###附录：
####JSON格式：

```
{
  "type":"object",
  "required":false,
  "properties":{
    "libraries":{
      "type":"array",
      "required":false,
      "items":{
        "type":"object",
        "required":false,
        "properties":{
          "file":{
            "type":"string",
            "required":false
          },
          "name":{
            "type":"string",
            "required":false
          },
          "modules":{
            "type":"array",
            "required":false,
            "items":{
              "type":"object",
              "required":false,
              "properties":{
                "name":{
                  "type":"string",
                  "required":true
                },
                "parameters":{
                  "type":"array",
                  "required":false,
                  "items":{
                    "type":"object",
                    "required":false,
                    "properties":{
                      "key":{
                        "type":"string",
                        "required":true
                      },
                      "value":{
                        "type":"string",
                        "required":true
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```