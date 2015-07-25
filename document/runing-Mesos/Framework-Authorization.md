###Authorization

Mesos 在 0.20.0 版本增加了给 framewrok 的授权支持。

授权允许：

1.  Frameworks 向已经授权的 ***roles*** 注册 ( 重新注册 )。
2.  Frameworks 作为经过授权的 ***users*** 启动 taks / executors 。 
3.  经过授权的 ***principals*** 通过 “/shutdown” HTTP endpoint 来关闭 frameworks。

###ACLs
授权通过访问控制列表 (ACLs) 实现。在上述三点中，都有相应的 ACL(s)，可以通过它来限制访问。 ALCs 可以通过 JSON 格式建立。

每个 ACL 指定一组 ***Subjects*** ，可以在一组 ***Objects*** 上执行 ***action*** 。

目前支持的 ***Actions*** 有：

1. “register_frameworks” : 注册 Frameworks
2. “run_tasks” ：执行 tasks / executors
3. “shutdown_frameworks” ：关闭 Frameworks

目前支持的 ***Subjects*** 有：

1. “principals”
	* Framework principals (用于 “register\_frameworks” 和 “run\_tasks” 动作)
	* Usernames  (用于 “shutdown_frameworks” 动作)

目前支持的 ***Objects*** 有：

1. “roles” ：资源角色其可以被 framework 所注册 (used by “register_frameworks” action)
2. “users” ：Unix 用户以 " user " 模式启动 task / executor。 (used by “run_tasks” action)
3. “framework_principals” ： 框架主体可以通过 HTTP POST 关闭 （ used by “shutdown_frameworks” action ）

注意： ***Subjects*** 和 ***Objects*** 可以用一组字符串或者特殊的值 ( ANY 或者 NONE )

###它是如何工作的

Mesos master 检查 ACLs 来验证一个请求是否被授权。

例如，当一个 framewrok 注册 ( 重新注册 ) 到 Master 时，“register_frameworks”  ACLs 检查如果 framework ( FrameworkInfo.principal ) 被授权，那么它可以接收给定的资源。如果没有被授权， 则不允许 framework 注册 ( 重新注册 )，同时返回一个 ***Error*** 信息 （ 从而中止 scheduler driver ）。

同样，当一个 framework 启动一个 task(s) , “run_tasks” ACLs 会检查 framewrok ( FrameworkInfo.principal ) 是否被授权作为一个 给定的 ***user*** 来执行 task / executor 。如果没有被授权，启动会被拒绝，并且 framwork 会获得 TASK\_LOST。

同理，当 user / principal 试图在 master　上通过　“/shutdown” HTTP endpoint　来关闭一个 framework 时。 “shutdown_frameworks” ACLs 会检查 ***principal*** 是否被授权关闭给定的　framework 。如果没有被授权，shutdown 操作会被拒绝，并且用户会收到一个　***Unauthorized *** (非法的) HTTP response 。

有几个需要注意的重要事项：

1. ACLs 的建立是按顺序匹配的。换句话来说，第一个匹配的 ACL 决定一个请求是否被授权。
2. 当没有指定的 ACLs 匹配给定的请求，那么这个请求是否被授权将取决于 ***ACLs.permissive*** 字段。默认情况下是 " true " 。

###事例

* framework ***foo*** 和 framework ***bar*** 可以以 ***alice*** 的身份执行 tasks 。

```
{
       "run_tasks": [
                      {
                        "principals": { "values": ["foo", "bar"] },
                        "users": { "values": ["alice"] }
                      }
                    ]
     }
```
* 任何 framework 都可以以 ***guest*** 身份执行 tasks 。

```
 {
       "run_tasks": [
                      {
                        "principals": { "type": "ANY" },
                        "users": { "values": ["guest"] }
                      }
                    ]
     }
```

* 不能有任何 framework 以 ***root*** 身份执行任务。

```
{
       "run_tasks": [
                      {
                        "principals": { "type": "NONE" },
                        "users": { "values": ["root"] }
                      }
                    ]
     }
```

* framework ***foo*** 只能以 ***guest*** 身份执行任务，其他身份不可以。

```
{
       "run_tasks": [
                      {
                        "principals": { "values": [ "foo" ] },
                        "users": { "values": ["guest"] }
                      },
                      {
                        "principals": { "values": [ "foo" ] },
                        "users": { "type": "NONE" }
                      }
                    ]
     }
```

* framework ***foo*** 可以注册到 ***analytics*** 和 ***ads*** 

```
  "register_frameworks": [
                                {
                                  "principals": { "values": ["foo"] },
                                  "roles": { "values": ["analytics", "ads"] }
                                }
                              ]
     }
```

* 除了 framework ***foo*** ，其他任何 framewrok 都不能注册到 ***analytics*** 。

```
 {
       "register_frameworks": [
                                {
                                  "principals": { "values": ["foo"] },
                                  "roles": { "values": ["analytics"] }
                                },
                                {
                                  "principals": { "type": "NONE" },
                                  "roles": { "values": ["analytics"] }
                                }
                              ]
     }
```

* 只有 framework ***foo*** 能注册在 ***analytics*** ，其他 framework 不能注册任何 roles 。

```
{
       "permissive" : false,

       "register_frameworks": [
                                {
                                  "principals": { "values": ["foo"] },
                                  "roles": { "values": ["analytics"] }
                                }
                              ]
     }
```

* 只有 ***ops*** principal 可以通过  “/shutdown” HTTP endpoint 关闭任何 frameworks 。

```
{
       "permissive" : false,

       "shutdown_frameworks": [
                                {
                                  "principals": { "values": ["ops"] },
                                  "framework_principals": { "type": "ANY" }
                                }
                              ]
     }
```

###启动授权
作为这个功能的一部分，有一个新的 flag 需要添加到 master。

* ***acls*** 该值可以是 JSON 字符串格式的 ACLs ,或者是包含用于授权的 JSON 格式的 ACL 文件路径。 路径可以是 ‘ file:///path/to/file ’ 或者 ‘ /path/to/file ’ 。 更多的期望格式请参考 mesos.proto 中的 ACLs protobuf。


