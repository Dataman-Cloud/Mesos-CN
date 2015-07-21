##Mesos Modules
Mesos module 是 Mesos 从版本 0.21.0 开始支持的实验性功能。
###免责声明
- 请自行程度开发和使用 Mesos Module 的风险
- 如有和 Module 有关问题，请发邮件到 modules@mesos.apache.org。发送到 dev 邮件组的 module 有关问题也会被转发到这个邮箱。
###什么是 Mesos Modules
Mesos module 提供了一种方法来轻松扩展 Mesos 的内部机能，这就是通过创建和按需加载共享库（shared libraries）。通过 Module 可以定制 Mesos 来支撑不同的应用场景，无需重新编译/链接整个工程。 Module 可以把外部依赖放到单独的库中，从而精简 Mesos 的核心。 Module 还可以让开发者很容易地尝试新功能。例如，编写一个包含特定编程环境虚拟机( Lua, Python, … )的 allocator，可以用这些编程语言开发资源分配算法，而不必把这些环境的依赖库打到 Mesos 的主工程中。最后，Module 为第三方提供一种简单的方法扩展 Mesos，而不必了解 Mesos 所有内部细节。

###调用 Mesos Modules

在 Mesos master, slave 以及 tests 中，可以通过命令行使用 ***--modules*** 标志位来指定要加载的模块列表。
- 通过 ***--modules=filepath*** 指定包含模块列表的文本文件（JSON格式）。***filepath*** 的格式可以是‘ file:///path/to/file ’ 或者 ‘ /path/to/file ’。
- 直接在命令行使用 ***--modules="{...}"*** 指定的模块列表。

#### JSON 格式举例

1. 加载包含 ***org_apache_mesos_bar*** 和 ***org_apache_mesos_baz*** 两个模块的 ***libfoo.so*** 库文件
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

2. 通过库文件 ***foo*** 来加载模块  ***org_apache_mesos_bar***， 同时将参数 X=Y 传递给命令行。例子的第二部分加载了 ***org_apache_mesos_baz*** 模块，但不包含任何参数。
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

3.通过命令行中加载 module：
```
 --modules='{"libraries":[{"file":"/path/to/libfoo.so", "modules":[{"name":"org_apache_mesos_bar"}]}]}'
```
####库名称

在 "file" 或者 "path" 这两个参数中，加载库的时候至少要指定一个。"file" 参数可以是文件名称(如"libfoo.so")，相对路径(如" myLibs/libfoo.so ")，或者文件的绝对路径(如" /home/mesos/lib/libfoo.so ")。
另外一个参数 "name" 指的是某个库的名称(如 "foo")。如果指定了参数 "name"，它会自动被匹配到当前操作系统的扩展名(如，"foo" 在 Linux 上扩展为 libfoo.so，在 OS X 则变为 "libfoo.dylib")。

如果加载库的时候没有指定 "file" 参数，会在标准的库文件搜索或者定义搜索路径的环境变量 ***LD\_LIBRARY\_PATH*** (OS X 则为 ***DYLD\_LIBRARY\_PATH***)。

如果 "file" 和 "name" 同时指定了，则 "name" 会被忽略。

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
