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

如果加载库的时候没有指定 "file" 参数，会在系统默认的或者环境变量 ***LD\_LIBRARY\_PATH*** (OS X 则为 ***DYLD\_LIBRARY\_PATH***) 指定的搜索路径中寻找。

如果同时指定了 "file" 和 "name" 参数，则 "name" 会被忽略。

###支持哪些类型的模块？
下面是目前可用的各种模块的种类。

####Allocator（分配器）

Mesos master 的 allocator 会定期将资源分配给某个framework。Allocator module 支持用户自己定义资源分配的算法（目前处于测试阶段），例如抢先超售算法（oversubscription with preemption），这个算法是 mesos 默认的 Hierarchical Dominant Resource Fairness allocator 无法支持的。

请按照如下步骤来加载一个自定义的分配器到 Mesos master 中：

- 将这个 module 列在 Mesos master 的 ***--modules*** 配置列表中。

- 使用 ***--allocator*** 标志位将其声明成 allocator

例如，使用下面的命令将在启动 mesos master 时加载 ***ExternalAllocatorModule*** 
```
./bin/mesos-master.sh --work_dir=m/work --modules="file://<modules-including-allocator>.json" --allocator=ExternalAllocatorModule
```

####Anonymous（匿名模块）
Anonymous module（匿名模块）不接收任何回调，只是与它们的父进程共存。这种 module 既不实现具体功能，也不代替任何现有的 module。例如不像 decorator module （装饰模块？），它不会直接向 mesos core 提供任何数据。

当使用 ***--modules*** 标志位（指定 anonymous module）启动 Mesos master 或 slave 的时候，不需要任何特定的 selector 标志位，anonymous module 会立即实例化。

####Authentication（身份认证模块）
通过使用 Authenticatee 和 Authenticator modules，第三方可以很快捷地开发和部署新的身份认证手段。例如通过 PAM (LDAP, MySQL, NIS, UNIX) 进行身份认证。

####Hook（钩子模块）
类似 Apache web 服务器里面的 Hook，hooks allows module writers to tie into internal components which may not be suitable to be abstracted entirely behind modules but rather lets them define actions on so-called hooks.

The available hooks API is defined in mesos/hook.hpp and for each hook defines the insertion point and available context. An example of this context is the task information which is passed to masterLaunchTaskHook.

Some hooks take in an object (e.g. TaskInfo) and return all or part of that object (e.g. task labels), so that the hook can modify or replace the contents in-flight. These hooks are referred to as decorators.

In order to enable decorator modules to remove metadata (environment variables or labels), the effect of the return value for decorator hooks changed in Mesos 0.23.0.

The Result return values before and after Mesos 0.23.0 means:

State	Before (0.22.x)	After (0.23.0+)
Error	Error is propagated to the call-site	No change
None	The result of the decorator is not applied	No change
Some	The result of the decorator is appended	The result of the decorator overwrites the final labels/environment object
To load a hook into Mesos, you need to

introduce it to Mesos by listing it in the --modules configuration,

select it as a hook module via the --hooks flag.

For example, the following command will run the Mesos slave with the TestTaskHook hook:

./bin/mesos-slave.sh --master=<IP>:<PORT> --modules="file://<path-to-modules-config>.json" --hooks=TestTaskHook


####隔离(Isolator)
Isolator 模块可以试验专门的隔离和监控功能，例如针对 GPGPU 或网络资源的第三方的资源隔离机制。

###编写 Mesos module

####A HelloWorld module
下面的代码片段描述了名为 "TestModule" 中的 "org_apache_mesos_bar" 模块的实现方法：

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

假设 Mesos 安装在默认位置，即 Mesos 动态库和头文件是可用的。
```
G ++ -lmesos -fpic -o test_module.o test_module.cpp
$ GCC -shared -o libtest_module.so test_module.o
```
####测试自行定义的 module
一方面我们可以使用 **–modules** 标志位来测试特定模块，另一方面也可以跑完整的 mesos 的测试用例。下面的命令将测试 mesos 加上 提供资源隔离功能的 org_apache_mesos_TestCpuIsolator 模块：
```
./bin/mesos-tests.sh --modules="/home/kapil/mesos/isolator-module/modules.json" --isolation="org_apache_mesos_TestCpuIsolator"
``` 

####Module 的命名规则
每个 module 的名称都应该是唯一的。在 JSON 文件中出现重复的 module 名称话，启动过程会异常退出。我们鼓励大家使用 Java 的包命名的规则来给 module 起名(http://docs.oracle.com/javase/tutorial/java/package/namingpkgs.html)。 

例如：
```
Module Name  Module Domain name  Module Symbol Name
foobar	      mesos.apache.org	   org_apache_mesos_foobar
barBaz	      example.com	        com_example_barBaz
```
长话短说：所有字母小写，将 domain name 逆序 - 用下划线分隔. - 不要简单地把 module 功能种类的名子作为这个 module 的名子 - 同一个组织下的模块仍然需要不同的名子。

####Module 版本号定义规则和向上兼容
在载入一个 module 之前，首先要将包含这个 module 代码的动态库载入 mesos。这个工作会在 mesos 启动的初期进行。添加新的 module 之后，Mesos 开发者不需要手动修改 mesos 的启动代码。开发者需要编写一个列表来说明 module 的某个版本和 mesos 各版本之间的兼容关系。这个列表位于配置文件 src/module/manager.cpp 中。这个列表包括了 module 的所有版本，以及和对应 mesos 版本的兼容关系。Module 的开发者要负责维护并且更新这个列表，保证版本兼容性的正确。Given that module implementation for older Mesos versions can still be written in the future, this may be impossible to tell and so in doubt it is best to just bump the required module version to the current Mesos version. But if one can be reasonably sure, assuming cooperative module developers, that a certain kind of module will continue to function across several Mesos versions, the table provides an easy way to specify this.

Module 和 mesos 的版本号大小之间必须遵从如下关系，保证 module 能够顺利加载：
kind 版本号 （小于等于） Library 版本号 （小于等于） Mesos 版本号

Mesos	  kind version Library	 是否兼容	  说明
0.18.0	 0.18.0	      0.18.0	  Yes	
0.29.0	 0.18.0	      0.18.0	  yes	
0.29.0	 0.18.0	      0.21.0	  yes	
0.18.0	 0.18.0	      0.29.0	  NO	        Library compiled against a newer Mesos release.
0.29.0	 0.21.0	      0.18.0	  NO	        Module/Library older than the kind version supported by Mesos.
0.29.0	 0.29.0	      0.18.0	  NO	        Module/Library older than the kind version supported by Mesos.

###Mesos module API 的变化
Modules API 可能造成兼容的问题的变化历史如下：

####Version 2
Added support for module-specific command-line parameters.
Changed function signature for create().

####Version 1
modules API 的初始版本。


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
