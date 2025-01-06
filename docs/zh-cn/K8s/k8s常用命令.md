
###### 如何查找非 running 状态的 Pod 呢？
    kubectl get pods -A --field-selector=status.phase!=Running | grep -v Complete

###### 如何查找 running 状态的 Pod 呢？
    kubectl get pods -A --field-selector=status.phase=Running | grep -v Complete

###### 批量设置Deployf副本数
    kubectl get deploy -n "命名空间" | awk '{if (NR>1){print $1}}'| xargs -I {} kubectl scale deploy {}  --replicas=1 -n "命名空间"

###### 批量删掉pod
    kubectl get pods -n "命名空间" | awk '{print $1}' | xargs kubectl delete pod -n "命名空间"

###### 批量删除job
    kubectl get pods -n "命名空间"  | grep OutOfcpu | awk '{print $1}' | xargs kubectl delete pod -n "命名空间" 

###### Terminating可使用kubectl中的强制删除命令
    kubectl delete pod "pod名称" -n "命名空间"--force --grace-period=0

###### 强制删除NAMESPACE
    kubectl delete namespace NAMESPACENAME --force --grace-period=0


###### 获取 cpu并排序
    kubectl top pod -A | sort --reverse --key 3 --numeric

###### 获取 memory并排序
    kubectl top pod -A | sort --reverse --key 4 --numeric

##### **基础命令create，delete，get，run，expose，set，explain，edit** 

> `create` 命令：根据文件或者输入来创建资源创建Deployment和Service资源

    kubectl create -f demo-deployment.yaml
    kubectl create -f demo-service.yaml
> `get` 命令 ：获得资源信息
```
    查看所有的资源信息
    kubectl get all
    kubectl get --all-namespaces
    查看pod列表
    kubectl get pod
    显示pod节点的标签信息
    kubectl get pod --show-labels
    根据指定标签匹配到具体的pod
    kubectl get pods -l app=example
    查看node节点列表
    kubectl get node 
    显示node节点的标签信息
    kubectl get node --show-labels
    查看pod详细信息，也就是可以查看pod具体运行在哪个节点上（ip地址信息）
    kubectl get pod -o wide
    查看服务的详细信息，显示了服务名称，类型，集群ip，端口，时间等信息
    kubectl get svc
    kubectl get svc -n kube-system
    查看命名空间
    $ kubectl get ns
    $ kubectl get namespaces
     查看所有pod所属的命名空间
    $ kubectl get pod --all-namespaces
    查看所有pod所属的命名空间并且查看都在哪些节点上运行
    $ kubectl get pod --all-namespaces  -o wide
    查看目前所有的replica set，显示了所有的pod的副本数，以及他们的可用数量以及状态等信息
    $ kubectl get rs
    查看已经部署了的所有应用，可以看到容器，以及容器所用的镜像，标签等信息
    $ kubectl get deploy -o wide
    $ kubectl get deployments -o wide
```

>    `run` 命令：在集群中创建并运行一个或多个容器镜像
```
    基本语法run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...]
    # 示例，运行一个名称为nginx，副本数为3，标签为app=example，镜像为nginx:1.10，端口为80的容器实例
    kubectl run nginx --replicas=3 --labels="app=example" --image=nginx:1.10 --port=80
    其他用法参见：http://docs.kubernetes.org.cn/468.html

    # 示例，运行一个名称为nginx，副本数为3，标签为app=example，镜像为nginx:1.10，端口为80的容器实例，并绑定到k8s-node1上
    $ kubectl run nginx --image=nginx:1.10 --replicas=3 --labels="app=example" --port=80 --overrides='{"apiVersion":"apps/v1","spec":{"template":{"spec":{"nodeSelector":{"kubernetes.io/hostname":"k8s-node1"}}}}}'
```

>    `expose` 命令:创建一个nginx服务并且暴露端口让外界可以访问
```
    $ kubectl expose deployment nginx --port=88 --type=NodePort --target-port=80 --name=nginx-service

    更多expose详细用法参见：http://docs.kubernetes.org.cn/475.html
```
>    `set` 命令: 配置应用的一些特定资源，也可以修改应用已有的资源

使用 kubectl set --help查看，它的子命令，env，image，resources，selector，serviceaccount，subject。
语法：resources (-f FILENAME | TYPE NAME) [--limits=LIMITS & --requests=REQUESTS]
set 命令详情参见： http://docs.kubernetes.org.cn/669.html
```
kubectl set resources 命令:这个命令用于设置资源的一些范围限制。

    资源对象中的Pod可以指定计算资源需求（CPU-单位m、内存-单位Mi），即使用的最小资源请求（Requests），限制（Limits）的最大资源需求，Pod将保证使用在设置的资源数量范围。
对于每个Pod资源，如果指定了Limits（限制）值，并省略了Requests（请求），则Requests默认为Limits的值。可用资源对象包括(支持大小写)：replicationcontroller、deployment、daemonset、job、replicaset。
将deployment的nginx容器cpu限制为“200m”，将内存设置为“512Mi”$ kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi
# 设置所有nginx容器中 Requests和Limits
$ kubectl set resources deployment nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi
# 删除nginx中容器的计算资源值
$ kubectl set resources deployment nginx --limits=cpu=0,memory=0 --requests=cpu=0,memory=0
```
```
    kubectl set selector 命令:设置资源的 selector（选择器）。如果在调用"set selector"命令之前已经存在选择器，则新创建的选择器将覆盖原来的选择器。
        selector必须以字母或数字开头，最多包含63个字符，可使用：字母、数字、连字符" - " 、点"."和下划线" _ "。如果指定了--resource-version，则更新将使用此资源版本，否则将使用现有的资源版本。
    注意：目前selector命令只能用于Service对象。
    语法：selector (-f FILENAME | TYPE NAME) EXPRESSIONS [--resource-version=version]
```
```
    kubectl set image 命令:用于更新现有资源的容器镜像。
    可用资源对象包括：pod (po)、replicationcontroller (rc)、deployment (deploy)、daemonset (ds)、job、replicaset (rs)。
    语法：image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N
    # 将deployment中的nginx容器镜像设置为“nginx：1.9.1”
    $ kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1

    # 所有deployment和rc的nginx容器镜像更新为“nginx：1.9.1”
    $ kubectl set image deployments,rc nginx=nginx:1.9.1 --all

    # 将daemonset abc的所有容器镜像更新为“nginx：1.9.1”
    $ kubectl set image daemonset abc *=nginx:1.9.1

    # 从本地文件中更新nginx容器镜像
    $ kubectl set image -f path/to/file.yaml nginx=nginx:1.9.1 --local -o yaml
```

```
    explain 命令
    用于显示资源文档信息
    $ kubectl explain rs
```
```
    edit命令
    用于编辑资源信息
    # 编辑Deployment nginx的一些信息
        $ kubectl edit deployment nginx

        # 编辑service类型的nginx的一些信息
        $ kubectl edit service/nginx
```
**设置命令：label，annotate，completion**
```
    label命令
    用于更新（增加、修改或删除）资源上的 label（标签）
    - label 必须以字母或数字开头，可以使用字母、数字、连字符、点和下划线，最长63个字符。
    - 如果--overwrite 为 true，则可以覆盖已有的 label，否则尝试覆盖 label 将会报错。
    - 如果指定了--resource-version，则更新将使用此资源版本，否则将使用现有的资源版本。
    语法：label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
    # 给名为foo的Pod添加label unhealthy=true
    $ kubectl label pods foo unhealthy=true

    # 给名为foo的Pod修改label 为 'status' / value 'unhealthy'，且覆盖现有的value
    $ kubectl label --overwrite pods foo status=unhealthy

    # 给 namespace 中的所有 pod 添加 label
    $ kubectl label  pods --all status=unhealthy

    # 仅当resource-version=1时才更新 名为foo的Pod上的label
    $ kubectl label pods foo status=unhealthy --resource-version=1

    # 删除名为“bar”的label 。（使用“ - ”减号相连）
    $ kubectl label pods foo bar-

    annotate命令
    - 更新一个或多个资源的Annotations信息。也就是注解信息，可以方便的查看做了哪些操作。
    - Annotations由key/value组成。
    - Annotations的目的是存储辅助数据，特别是通过工具和系统扩展操作的数据，更多介绍在这里。
    - 如果--overwrite为true，现有的annotations可以被覆盖，否则试图覆盖annotations将会报错。
    - 如果设置了--resource-version，则更新将使用此resource version，否则将使用原有的resource version。
    语法：annotate [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
    # 更新Pod“foo”，设置annotation “description”的value “my frontend”，如果同一个annotation多次设置，则只使用最后设置的value值
    $ kubectl annotate pods foo description='my frontend'

    # 根据“pod.json”中的type和name更新pod的annotation
    $ kubectl annotate -f pod.json description='my frontend'

    # 更新Pod"foo"，设置annotation“description”的value“my frontend running nginx”，覆盖现有的值
    $ kubectl annotate --overwrite pods foo description='my frontend running nginx'

    # 更新 namespace中的所有pod
    $ kubectl annotate pods --all description='my frontend running nginx'

    # 只有当resource-version为1时，才更新pod 'foo'
    $ kubectl annotate pods foo description='my frontend running nginx' --resource-version=1

    # 通过删除名为“description”的annotations来更新pod 'foo'。
    # 不需要 -overwrite flag。
    $ kubectl annotate pods foo description-

    completion命令
    用于设置 kubectl 命令自动补全
    BASH
    # 在 bash 中设置当前 shell 的自动补全，要先安装 bash-completion 包
    $ source <(kubectl completion bash)

    # 在您的 bash shell 中永久的添加自动补全
    $ echo "source <(kubectl completion bash)" >> ~/.bashrc 

    ZSH
    # 在 zsh 中设置当前 shell 的自动补全
    $ source <(kubectl completion zsh)  

    # 在您的 zsh shell 中永久的添加自动补全
    $ echo "if [ $commands[kubectl] ]; then source <(kubectl completion zsh); fi" >> ~/.zshrc 
```
###### **kubectl 部署命令：rollout，rolling-update，scale，autoscale**
```
    rollout 命令
    用于对资源进行管理,可用资源包括：deployments，daemonsets。
    history（查看历史版本）
    pause（暂停资源）
    resume（恢复暂停资源）
    status（查看资源状态）
    undo（回滚版本）
    # 语法
    $ kubectl rollout SUBCOMMAND

    # 回滚到之前的deployment
    $ kubectl rollout undo deployment/abc

    # 查看daemonet的状态
    $ kubectl rollout status daemonset/foo

    rolling-update命令
    执行指定ReplicationController的滚动更新。
    该命令创建了一个新的RC， 然后一次更新一个pod方式逐步使用新的PodTemplate，最终实现Pod滚动更新，new-controller.json需要与之前RC在相同的namespace下。
    语法：rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] --image=NEW_CONTAINER_IMAGE | -f NEW_CONTROLLER_SPEC)
    # 使用frontend-v2.json中的新RC数据更新frontend-v1的pod
    $ kubectl rolling-update frontend-v1 -f frontend-v2.json

    # 使用JSON数据更新frontend-v1的pod
    $ cat frontend-v2.json | kubectl rolling-update frontend-v1 -f -

    # 其他的一些滚动更新
    $ kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2

    $ kubectl rolling-update frontend --image=image:v2

    $ kubectl rolling-update frontend-v1 frontend-v2 --rollback

    scale命令
    扩容或缩容 Deployment、ReplicaSet、Replication Controller或 Job 中Pod数量
    scale也可以指定多个前提条件，如：当前副本数量或 --resource-version ，进行伸缩比例设置前，系统会先验证前提条件是否成立。这个就是弹性伸缩策略。
    语法：kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME)
    # 将名为foo中的pod副本数设置为3。
    $ kubectl scale --replicas=3 rs/foo
    kubectl scale deploy/nginx --replicas=30

    # 将由“foo.yaml”配置文件中指定的资源对象和名称标识的Pod资源副本设为3
    $ kubectl scale --replicas=3 -f foo.yaml

    # 如果当前副本数为2，则将其扩展至3。
    $ kubectl scale --current-replicas=2 --replicas=3 deployment/mysql

    # 设置多个RC中Pod副本数量
    $ kubectl scale --replicas=5 rc/foo rc/bar rc/baz

    autoscale命令：这个比scale更加强大，也是弹性伸缩策略 ，它是根据流量的多少来自动进行扩展或者缩容。
    指定Deployment、ReplicaSet或ReplicationController，并创建已经定义好资源的自动伸缩器。使用自动伸缩器可以根据需要自动增加或减少系统中部署的pod数量
    语法：kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [flags]
    # 使用 Deployment “foo”设定，使用默认的自动伸缩策略，指定目标CPU使用率，使其Pod数量在2到10之间
    $ kubectl autoscale deployment foo --min=2 --max=10

    # 使用RC“foo”设定，使其Pod的数量介于1和5之间，CPU使用率维持在80％
    $ kubectl autoscale rc foo --max=5 --cpu-percent=80
```
###### **集群管理命令：certificate，cluster-info，top，cordon，uncordon，drain，taint**
```
    certificate命令
    用于证书资源管理，授权等
    [root@master ~]# kubectl certificate --help
    Modify certificate resources.

    Available Commands:
    approve     Approve a certificate signing request
    deny        Deny a certificate signing request

    Usage:
    kubectl certificate SUBCOMMAND [options]

    Use "kubectl <command> --help" for more information about a given command.
    Use "kubectl options" for a list of global command-line options (applies to all commands).

    # 例如，当有node节点要向master请求，那么是需要master节点授权的
    kubectl  certificate approve node-csr-81F5uBehyEyLWco5qavBsxc1GzFcZk3aFM3XW5rT3mw node-csr-Ed0kbFhc_q7qx14H3QpqLIUs0uKo036O2SnFpIheM18

    cluster-info 命令
    显示集群信息
    $ kubectl cluster-info

    top命令
    用于查看资源CPU，内存磁盘等资源的使用率
    kubectl top pod --all-namespaces
    它需要heapster运行才行

    cordon命令：用于标记某个节点不可调度
    kubectl cordon NODE

    uncordon命令：用于标签节点可以调度
    kubectl uncordon NODE

    drain命令： 用于在维护期间排除节点。
    kubectl drain Node --force

    taint命令：用于给某个Node节点设置污点
    语法：$ kubectl taint NODE NAME KEY_1=VAL_1:TAINT_EFFECT_1 ... KEY_N=VAL_N:TAINT_EFFECT_N
    用带有键“专用”和值“特殊用户”的污点更新节点“ foo”，并效果为“ NoSchedule”。＃如果已经存在具有该键和效果的污点，则将其值替换为指定的值。
    kubectl taint nodes foo dedicated=special-user:NoSchedule

    从节点“ foo”中删除带有键“专用”的污点，并在存在的情况下影响“ NoSchedule”。
    kubectl taint nodes foo dedicated:NoSchedule-

    从节点“ foo”中删除键为“专用”的所有污点
    kubectl taint nodes foo dedicated-

    在标签为mylabel = X的节点上添加键为“专用”的污点
    kubectl taint node -l myLabel=X  dedicated=foo:PreferNoSchedule

    向节点“ foo”添加带有键“ bar”且无值的污点
    kubectl taint nodes foo bar:NoSchedule
```
##### **集群故障排查和调试命令：describe，logs，exec，attach，port-**
```    
    describe命令：显示特定资源的详细信息
    语法：kubectl describe TYPE NAME_PREFIX（首先检查是否有精确匹配TYPE和NAME_PREFIX的资源，如果没有，将会输出所有名称以NAME_PREFIX开头的资源详细信息）支持的资源包括但不限于（大小写不限）：pods (po)、services (svc)、 replicationcontrollers (rc)、nodes (no)、events (ev)、componentstatuses (cs)、 limitranges (limits)、persistentvolumes (pv)、persistentvolumeclaims (pvc)、 resourcequotas (quota)和secrets。
    #查看my-nginx pod的详细状态
    kubectl describe po my-nginx

    logs命令：用于在一个pod中打印一个容器的日志，如果pod中只有一个容器，可以省略容器名
    语法：kubectl logs [-f] [-p] POD [-c CONTAINER]
    # 返回仅包含一个容器的pod nginx的日志快照
    $ kubectl logs nginx
    # 返回pod ruby中已经停止的容器web-1的日志快照
    $ kubectl logs -p -c ruby web-1
    # 持续输出pod ruby中的容器web-1的日志
    $ kubectl logs -f -c ruby web-1
    # 仅输出pod nginx中最近的20条日志
    $ kubectl logs --tail=20 nginx
    # 输出pod nginx中最近一小时内产生的所有日志
    $ kubectl logs --since=1h nginx

    参数选项
    -c, --container="": 容器名。
    -f, --follow[=false]: 指定是否持续输出日志（实时日志）。
    --interactive[=true]: 如果为true，当需要时提示用户进行输入。默认为true。
    --limit-bytes=0: 输出日志的最大字节数。默认无限制。
    -p, --previous[=false]: 如果为true，输出pod中曾经运行过，但目前已终止的容器的日志。
    --since=0: 仅返回相对时间范围，如5s、2m或3h，之内的日志。默认返回所有日志。只能同时使用since和since-time中的一种。
    --since-time="": 仅返回指定时间（RFC3339格式）之后的日志。默认返回所有日志。只能同时使用since和since-time中的一种。
    --tail=-1: 要显示的最新的日志条数。默认为-1，显示所有的日志。
    --timestamps[=false]: 在日志中包含时间戳。
    exec命令：进入容器进行交互，在容器中执行命令
    语法：kubectl exec POD [-c CONTAINER] -- COMMAND [args...]
    命令选项：
    - -c, --container="": 容器名。如果未指定，使用pod中的一个容器。
    - -p, --pod="": Pod名。
    - -i, --stdin[=false]: 将控制台输入发送到容器。
    - -t, --tty[=false]: 将标准输入控制台作为容器的控制台输入。
    # 进入nginx容器，执行一些命令操作
    kubectl exec -it nginx-deployment-58d6d6ccb8-lc5fp bash

    attach命令：连接到一个正在运行的容器。
    语法：kubectl attach POD -c CONTAINER
    参数选项：
    - -c, --container="": 容器名。如果省略，则默认选择第一个 pod。
    - -i, --stdin[=false]: 将控制台输入发送到容器。
    - -t, --tty[=false]: 将标准输入控制台作为容器的控制台输入。
    # 获取正在运行中的pod 123456-7890的输出，默认连接到第一个容器
    $ kubectl attach 123456-7890

    # 获取pod 123456-7890中ruby-container的输出
    $ kubectl attach 123456-7890 -c ruby-container

    # 切换到终端模式，将控制台输入发送到pod 123456-7890的ruby-container的“bash”命令，并将其输出到控制台/
    # 错误控制台的信息发送回客户端。
    $ kubectl attach 123456-7890 -c ruby-container -i -t

    cp命令：拷贝文件或者目录到pod容器中
    用于pod和外部的文件交换,类似于docker 的cp，就是将容器中的内容和外部的内容进行交换。
    语法：kubectl cp
    将/tmp/foo_dir本地目录复制到默认名称空间中的远程pod中的/tmp/bar_dir
    kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir

    将/tmp/foo本地文件复制到特定容器中远程pod的/tmp/bar中
    kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container>

    将/tmp/foo本地文件复制到命名空间中远程pod的/ tmp/bar中
    kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar
    
    将/tmp/foo从远程Pod复制到本地/tmp/bar
    kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar
```

pod 按照内存排序

        kubectl top pods  -A | sort --reverse --key 4 --numeric
pod 按照cpu排序

        kubectl top pod -A | sort --reverse --key 3 --numeric
查看 node的总的request和limit值

        kubectl get nodes --no-headers | awk '{print $1}' | xargs -I {} sh -c "echo {} ; kubectl describe node {} | grep Allocated -A 5 | grep -ve Event -ve Allocated -ve percent -ve --;"
所有pod按照内存请求排序

        kubectl get pods --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,POD:.metadata.name,MEMORY_REQUEST:.spec.containers[*].resources.requests.memory" | sort -t ' ' -k3 -h
按照node节点分组，将节点pod按照请求内存排序

        kubectl get pods --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,NODE:.spec.nodeName,POD:.metadata.name,MEMORY_REQUEST:.spec.containers[*].resourc
        zes.requests.memory" | sort -k2,2 -k4,4h
按照所有pod cpu请求排序

        kubectl get pods --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,POD:.metadata.name,CPU_REQUEST:.spec.containers[*].resources.requests.cpu" --sort-by=.spec.containe
        rs[0].resources.requests.cpu

按照node节点分组，将节点pod按照请求cpu排序

        kubectl get pods --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,POD:.metadata.name,CPU_REQUEST:.spec.containers[*].resources.requests.cpu,NODE:.spec.nodeName" | sort -k4,4h

按照node节点分组，将节点pod按照内存 limit 排序

        kubectl get pods --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,NODE:.spec.nodeName,POD:.metadata.name,MEMORY_LIMIT:.spec.containers[*].resources.limits.memory" | sort -k2,2 -k4,4 | column -t

按照node节点分组，将节点pod按照cpu limit 排序


获取前一个容器的日志：

        kubectl logs goods-center-web-6db555bcb9-6g7rt --previous --tail=4000 -n=saas-prod

平滑删除pod

        kubectl -n pre delete pod factory-process-web-f-86b9b6d89-bnrjr  --grace-period=45