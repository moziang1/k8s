

## YAML的应用

```
# yaml格式的pod定义文件完整内容：
apiVersion: v1       #必选，版本号，例如v1
kind: Pod       #必选，Pod             //定义创建pod的类型
metadata:       #必选，元数据          //关于pod的信息信息数据
  name: string       #必选，Pod名称       
  namespace: string    #必选，Pod所属的命名空间
  labels:      #自定义标签
    - name: string     #自定义标签名字
  annotations:       #自定义注释列表
    - name: string
    
spec:         #必选，Pod中容器的详细定义（为这个创建的pod设置详细信息
  containers:      #必选，Pod中容器列表
  - name: string     #必选，容器名称
    image: string    #必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent] #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
    command: [string]    #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]     #容器的启动命令参数列表
    workingDir: string     #容器的工作目录
    volumeMounts:    #挂载到容器内部的存储卷配置
    - name: string     #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string    #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean    #是否为只读模式
    ports:       #需要暴露的端口库号列表
    - name: string     #端口号名称
      containerPort: int   #容器需要监听的端口号
      hostPort: int    #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string     #端口协议，支持TCP和UDP，默认TCP
    env:       #容器运行前需设置的环境变量列表
    - name: string     #环境变量名称
      value: string    #环境变量的值
    resources:       #资源限制和请求的设置
      limits:      #资源限制的设置
        cpu: string    #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string     #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:      #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string     #内存清楚，容器启动的初始可用数量
    livenessProbe:     #对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:      #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0  #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0   #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged:false
    restartPolicy: [Always | Never | OnFailure]#Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
    nodeSelector: obeject  #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:    #Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork:false      #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:       #在该pod上定义共享存储卷列表
    - name: string     #共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}     #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string     #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string     #Pod所在宿主机的目录，将被用于同期中mount的目录
      secret:      #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string  
        items:     
        - key: string
          path: string
      configMap:     #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
          path: string
```



## 案例

vim nginx.yml

```
apiVersion: extensions/v1beta1          //使用版本v1
kind: Deployment                    //指定pod模式为期望（群集中一直保持特定的副本数量）
metadata:                             //设置pod的信息
  name: nginx-deployment2             //设置pod的名称为nginx-deployment2
spec:                     //定义这个pod的信息
  replicas: 2                  //设置副本数量为2（期望值为2）
  template:                    //设置pod的模板
    metadata:                      //设置pod模板的数据信息
      labels:                             //设置pod模板数据信息中的标签
        app: web_server                        //定义pod模板数据信息中标签的名称为web_server
    spec:                        //和metadata平级，设置pod模板的信息
      containers:                    //定义pod模板中副本的详细信息
      - name: nginx                      //定义一个name为 nginx
        images: nginx                    //给nginx副本设置一个镜像
```





如何卸载

查看

```
kubectl get pod -o wide
```

![image-20191126202000053](image/image-20191126202000053.png)

卸载手动建立的

```
kubectl delete deployment nginx-deployment                //nginx-deployment是我们要卸载的副本的名称，会卸载掉所有同名的副本
```



### 当要删除用yml文件配置出来的如何卸载

```
kubectl delete -f nginx.yml
```

这样就会将用这个yml文件部署的nginx都卸载掉





## 关闭驱离主节点(主节点默认情况下就是驱离的，我们使他能够工作)

```
kubectl taint  node k8smaster node-role.kuberetes.io/master-
```



## 驱离

```
kubectl taint  node k8smaster node-role.kuberetes.io/master="":NoSchedule
```

#查看服务节点

```
kubectl get pod -o wide
```

