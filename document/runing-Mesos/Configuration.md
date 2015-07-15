#mesos 配置向导
标签： 配置

mesos master和slave可以通过命令行参数或环境变量来操作一系列的配置选项。一系列的可用选项可以通过运行mesos-master –help或者mesos-slave –help来查看。每个选项可以通过两种方式设置：

 - 通过使用 –-option_name=value将配置选项传递给应用，即可以通过直接指定值或指定一个内部包含值得文件(--opthon_name=file://文件路径).该路径对应当下工作目录可以为绝对或相对路径。
 - 通过设定系统变量  MESOS_OPTION_NAME(选项名称带有对应的MESOS_前缀名)

配置值通常首先通过环境变量接着通过命令行来发现
**重要的配置**
如果你需要特定复杂的需求，当配置Mesos的时候请参考./configure --help 。另外，本文档列举的近视这些选项的一个子集。你可以通过运行命令带有--help标记来找到一个完整的信息源包含你的mesos版本支持哪个标记，例如 mesos-master --help。

**Master和Slave的配置选项**

下面的这些选项可以同时被master和slave所支持。

     标记                       解释
    --ip=VALUE                 监听的地址
    --firewall_rules=VALUE     该值可以为JSON类型字符的rules或一个文件路径包含JSON类型
                               rules在终端防火墙中被使用。路径可以为以下任何形式 
                               file://文件路径 或 /文件路径。
                               期望的类型信息请参考flags.proto中的防火墙信息。
                               比如：
                               {
                                 "disabled_endpoints" : {
                                    "paths" : [
                                      "/files/browse.json",
                                      "/slave(0)/stats.json",
                                         ]
                                     }
                               }
    --[no-]help                输出帮助信息(默认:否)
    --log_dir=VALUE            输出日志文件的地址(没有默认，如果不指定就什么都不写，不会影响输出到stderr)
    --logbufsecs=VALUE         缓冲日志的时间单位 默认：0秒
    --loggin_level=VALUE       输入日志的级别 默认INFO
    --port=VALUE               监听端口(master默认5050,slave默认5051)
    --[no-]quiet               禁用输出日志到sterr (默认:否)
    --[no-]version             显示版本并且退出(默认：否)



**Master配置选项**

必选标记

        标记                        解释   
        --quorum=VALUE              副本的仲裁数量的大小取决于使用'replicated_log'的基本注册表。                      其非常重要的设定此值为大多数的管理节点.比如，quorum > (管理节点总量)/2,如果在独立模式下运行不需要该设置(non-HA)。
        --work_dir=VALUE            注册表中存储持久性信息的地址。
        --zk=VALUE                  zookeeper URL(用于从主节点中选举出来一个首脑)可能为以下之一：
                                    zk://host1:port1,host2:port2,.../path
                                    zk://username:password@host1:port1,host2:port2,.../path
                                    file:///path/to/file
                                    注意：如果在独立模式下运行就不需要该设置(non-HA)。
可选标记：

        标记                        解释 
        --acls=VALUE               值是一个JSON格式的字符类型ACLs，请记住你也可以使用file:///文件路径或/文件路径参数值格式来将该JSON写入文件，针对期望的格式请参考mesos.proto中的ACLs protobuf
                                   JSON文件例子：
                                   {
                                        "register_frameworks": [
                                        {
                                    "principals": { "type": "ANY" },
                                    "roles": { "values": ["a"] }
                                    }
                                        ],
                                    "run_tasks": [
                                        {
                                    "principals": { "values": ["a", "b"] },
                                    "users": { "values": ["c"] }
                                        }
                                    ],
                                "shutdown_frameworks": [
                                    {
                                    "principals": { "values": ["a", "b"] },
                                    "framework_principals": {  "values":                           ["c"]}
                                            }
                                        ]
                                    }
    --allocation_interval=VALUE 性能(批量)分配之间多长时间用来等待(比如，500ms,1sec等).(默认值：1secs) 
    --allocator=VALUE           针对框架做资源分配的分配器。使用默认的HierarchicalDRF分配器或通过使用--modules来加载备用的分配器模块。(默认：HierarchicalDRF)
    --[no-]authenticate         如果身份验证设置为'true'，只有经过身份验证的框架才被允许被注册。如果设置为'false'未经身份验证的框架也能允许被注册。(默认：false)
    --[no-]authenticate_slaves  如果设置为'true' 通过身份验证的slave才能被注册，如果设置为'false'未经身份验证的slaves也能被注册。(默认：false)
    --authenticators=VALUE      验证器被实现用于当框架或slave进行身份验证时，可以使用默认的crammd5或者通过使用--modules来加载备用的验证器模块。(默认:crammd5)
    --cluster=VALUE             显示在webui上的可被人读取的集群名称。
    --credentials=VALUE         一个包含一系列凭据列表的文件的路径，每行包含由空格分割的'principal' 和 'secret'，或是包含凭证的JSON格式文件的路径，路径格式必须为file:///文件路径或/文件路径之一。
                                json文件样例：
                                {
                                    "credentials": [
                                    {
                                        "principal": "sherman",
                                        "secret": "kitesurf"
                                    }
                                        ]
                                }
                                文本文件样例：
                                username secret
    --external_log_file=VALUE    指定外部的管理日志文件。该文件将暴露在webui和HTTP api中，当用stderr记录日志时很有用，否则mesos日志文件是未知的。

--framework_sorter=VALUE 给定用户的工作框架之间资源分配的策略。选项和user_allocator相同。默认：drf。

--hooks=VALUE 钩模块以逗号分割的列表安装在住节点内。

--hostname=VALUE 主节点通知zookeeper里面的主机名。如果没有设置将从绑定的IP地址解析。

--[no-]log_auto_initialize 是否自动初始化注册日志，如果这是为false，必须在第一次使用时手动初始化。默认：true。

--modules 加载模块列表并用于内部的子系统。使用—modules=filepath指定包含JSON格式文件的模块列表。可以使用file:///path/to/file 或者/path/to/file指定JSON格式的文件。用—modules=”{...}”指定模块列表。

--offer_timeout=VALUE 从工作框架提议撤销之前的持续时间。有助于当工作框架坚持提议，或者不小心丢了提议时的公平。

--rate_limits=VALUE 值可以是一个JSON格式的字符串或者是一个文件路径包含JSON格式的文件用户框架的速率限制。文件路径可以是file:///path/to/file或者/path/to/file。

--recovery_slave_removal_limit=VALUE 对于故障转移，限制百分比从节点冲注册表中删除和关机后重新登记超时时间间隔。如果超出限制主节点将删除从节点。这可以用来为生产环境提供安全保障。

生产环境希望master故障切换，顶多一定比例的从节点会永久失败例如由于机架失败。设定此限制将确保需要人介入从节点意外普遍故障发生在集群中。值[0%-100%]。默认:100%.

--slave_removal_rate_limit=VALUE 最大速率在从节点被删除从主节点健康检查失败时。默认情况下从节点将被删除只要健康检查失败。值的形式是‘从节点数量’/‘持续时间’。

--registry=VALUE 注册表持久性策略；可用选项是 'replicated_log', 'in_memory'，进行测试。默认：replicated_log。

--registry_fetch_timeout=VALUE 持续时间等待从注册表获取数据后其中操作被认为失败。默认：1mins

--registry_store_timeout=VALUE 持续时间等待将数据存储到注册表视为失败。默认：5secs

--[no-]registry_strict master是否将会采取行动持续信息基础存储到注册表中。

--roles=VALUE 逗号分割的分配角色列表，在这个集群可能属于工作框架。

--[no-]root_submissions 可以root提交框架吗。默认：true

--slave_reregister_timeout=VALUE 超时时间内所有的从节点预计将重新注册时，选举出一个新的master。如果超时没有重新注册上从节点将从注册表里面删除并关闭如果他们试图与主服务器通信。

--user_sorter=VALUE 用户之间资源分配策略，可能是dominant_resource_fairness (drf)。默认：drf

--webui_dir=VALUE 网页文件目录，默认：/usr/local/share/mesos/webui

--weights=VALUE 逗号分割的role/weight列表，成对表单'role=weight,role=weight'。 weights是用来表示形式的优先级。

--whitelist=VALUE 一个文件名包含从节点作为通知的列表。这个文件是个监测，并且定期重新读取刷新从节点名单。默认：None。

--zk_session_timeout=VALUE zookeeper session超时。



从节点选项

必须项：

--master=VALUE 从节点链接到主节点的地址，有三种方式1.--master=masterip1:port,masterip2:port 2.--master=zk://host1:port1,host2:port2,.../path --master=zk://username:password@host1:port1,host2:port2,.../path 3. file:///path/to/file



可选项

--attributes=VALUE 机器的属性：rack:2或者rack:2;u:1

--authenticatee=VALUE 用于主节点身份验证，默认crammd5，或者用—modules加载备用模块。默认：crammd5。

--[no-]cgroups_enable_cfs 通过限制CFS带宽来限制CPU资源. 默认: defult

--cgroups_hierarchy=VALUE cgroups的路径根位置. 默认: /sys/fs/cgroup

--[no-]cgroups_limit_swap 用于内存和swap的限制, 默认: false, 只限制内存

--cgroups_root=VALUE 根cgroup的命名. 默认: mesos

--container_disk_watch_interval=VALUE 用于查询容器中磁盘配额时间间隔. 被用于posix/disk的时间间隔, 默认: 15秒

--containerizer_path=VALUE 当外部隔离机制被激活时(--isolation=external), 外部容器被执行的路径 

--containerizers=VALUE 用逗号把一组容器隔开, 以达到对容器的实现. 包括mesos, external, docker在Linux中. 默认: mesos

--credential=VALUE 一行包含"principal"和"secret"由空格隔开的文本路径. 或是包含一条凭证的JSON格式文件的路径. 路径的格式是file://path/to/file. 也可以使用路径file:///path/to/file, 从文件中读取值. 
                                  json文件样例：
                                  {
                                      "principal": "username",
                                      "secret": "secret"
                                  }

--default_container_image=VALUE 正在使用外部容器机制时, 如果没有指定任务, 默认的容器镜像被使用. 

--default_container_info=VALUE JSON格式的容器信息将会列出一些没有指定的容器信息的任何执行信息. 
                                  例子:
                                  {
                                      "type": "MESOS",
                                      "volumes": [
                                      {
                                          "host_path": "./.private/tmp",
                                          "container_path": "/tmp",
                                          "mode": "RW"
                                       }
                                          ]
                                  }

--default_role=VALUE 

--disk_watch_interval=VALUE 硬盘使用情况的周期性检查间隔. 默认: 1分钟

--docker=VALUE docker容器中docker执行文件的绝对路径. 默认: docker

--docker_remove_delay=VALUE 删除容器之前的等待时间. 默认: 6小时

--[no-]docker_kill_orphans 杀掉一个孤立的容器. 当你发布多个slave在同一个OS, 为了避免





 






