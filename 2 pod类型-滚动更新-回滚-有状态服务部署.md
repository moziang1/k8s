# 基础概念--pod



# 1  pod 类型



```mysql
#pod分为两类  : 官方没有定义，由前辈定义的
1. 自主式pod   #当自主容器死亡时，无法进行恢复
2. 控制器管理的pod
```

## 1  自主式pod 

当我们开启一个pod后，它会运行第一个容器

```mysql
pause  
```

意义：

```mysql
#只要是有pod这个容器就会被启动
#一个pod中的容器是>=1

#如果我预定义了2个容器，那么他们会通用pause的网络站，存储卷

#因为他们共享同一个网络站，所以容器的端口是不能冲突的
#因为是再同一个网络战中，也会挂载同一个存储卷（valumo）
（其实我个人理解，就是一个pod 就相当于一台服务器，你里面的容器的网络都是使用的同一块网卡、硬盘，不分什么你我。 就是端口占用的时候注意别冲突就行）
#如果端口冲突，则会无限重启，或起不来
```



## 2 控制器管理的pod



### 1. pod控制器类型

```mysql
1. replication controller & replicase &  deployment
	  > HPA(horizontalpodautoscale)
	  
2. Statefullset
3. DaemonSet
4. Job,Cornjob
```



#### 1.1  replication controller & replicase &  deployment

```mysql
1. RC(replication controller)  #期望副本数量为指定数量，当pod失效时，它在资源条件允许的情况下会自动去生产新的pod,而且多出来的容器也会被自动回收

2. RC(replicase)  #rc和rs基本功能一样，比rs多一个支持集合式的selector

#我们创建pod时会给pod打标签
#比如php容器 表明名
#app - apache1
#php - php1
#当我要删除容器或pod时，我就可以使用，当php等于php1什么的。
#RS比RC更有效，在新版本中官方抛弃RC专用RS


3. deployment   #虽然RS可以独立使用，但一般啊hi是简易使用deployment来'自动管理' RS，这样就不需要单项和其他机制的不兼容问题
(比如RS不支持rolling-update(滚动更新)，但deployment支持)

```



#### 1.2 #滚动更新的含义

```mysql
 #我们要修改两个pod的版本
 #滚动更新


#1 当我运行了两个pod

  pod1        pod2 
(v1 开启)   (v2 开启)
   |            |


 #2 先多一个新的pod
 
  pod1        pod2         pod3
(v1 开启)   (v2 开启)     (v2 开启)
   |            |            |
  #3 然后关闭一个旧pod（v2)
 
   pod1        pod3
(v1 开启)   (v2 开启)
   |            |

  #4 在开启一个新的pod

  pod1        pod3      pod4
(v1 开启)   (v2 开启)   (v1 开启)
   |            |         |
   
   #5 关闭旧pod(v1)
     pod3      pod4
  (v2 开启)   (v1 开启)
       |         |
       
       
       
#rs不支持这个机制，但deployment支持这个
#但是我们要知道的不是说的deployment比RS好，就不用RS了,deployment本身是不支持pod的创建的

#它是根据RS来达到一个pod的创建的
#创建过程

		      deployement   #它会自动创建一个RS
		           |
		---------------------      
		|                    |
	    RS                  ...
	     | #RS创建pod
 ------------
 |      |     |
pod1  pod2  pod3


#有一天我们需要滚动更新了
#那么它会创建一个新的RS

		      deployement   
		           |
		---------------------      
		|                    |
	    RS                  RS-1  #新建的RS
	     |                    |
 --------------           --------------
 |      |     |           |      |     |
pod1  pod2  pod3         

#他会陆续把RS上的pod关闭，然后再RS-1上开启新的pod
#如下

#滚动更新1
		      deployement   
		           |
		---------------------      
		|                    |
	    RS                  RS-1  
	     |                    |
 --------------           --------------
 |      |     |           |      |     |
[pod1] pod2  pod3        #pod1                      #RS的pod1关闭,再RS-1上创建一个新的pod1
      
#滚动更新2
      		      deployement   
		           |
		---------------------      
		|                    |
	    RS                  RS-1  
	     |                    |
 --------------           --------------
 |      |     |           |      |     |
[pod1] [pod2]  pod3     #pod1    pod2          #RS的pod2关闭,再RS-1上创建一个新的pod2

#滚动更新3
      		      deployement   
		           |
		---------------------      
		|                    |
	    RS                  RS-1  
	     |                    |
 --------------           --------------
 |      |     |           |      |     |
[pod1] [pod2] [pod3]     #pod1   pod2  pod3      #RS的pod3关闭,再RS-1上创建一个新的pod3 
      
```



#### 1.3 #回滚

```mysql
#当我们发现更新的版本有问题时可以进行回滚操作

      		      deployement   
		           |
		---------------------      
		|                    |
	    RS                  RS-1  
	     |                    |
 --------------           --------------
 |      |     |           |      |     |
[pod1] [pod2]'[pod3]'     pod1   pod2  pod3 


#running-auto
#为什么可以回滚，因为deployement 的旧RS创建的pod并不是删除pod，而是停用, 我们回滚，就是开启旧的RS上的pod
```





#### 1.4 HPA (pod水平自动扩展） 

```mysql
  #HPA (Horizontal Pod Autoscaling)
  仅使用于deployment和RS,在v1版本中仅支持根据pod的cpu利用率扩容，在v1alpha,支持根据内存和用户自定义的metric扩容
  
  
  #1 我们先定义了一个RS
         deployement   
              |
    ---------------------      
    |                    |
    RS    #定义RS 
  ------
  |    |
 pod1 pod2 #运行了俩POD
 
 
 
 #2  然后基于这个RS去定义了一个HPA
          deployement   
              |
    ---------------------      
    |                    
    RS-------------HPA #定义一个HPA
  ------
  |    |
 pod1 pod2 
 
 
 
  #3  给HPA定义一些规则
          deployement   
              |
    ---------------------      
    |                    
    RS-------------HPA #定义一个HPA      
  ------          #cpu > 80   
  |    |          #MAX 10
 pod1 pod2        #min 2
 
 #当cpu利用率大于80的情况下才会去扩建容器
 #当cpu利用不足时，会自动回收pod
 #最大可用扩容到10个pod(RS中)
 #最小有2个pod
```







## 2 StatefullSet(部署有状态服务如：mysql)

```mysql
#对应deployments和replicasets是为无状态服务而设计)

#docker 主要面对的是(无状态服务)
#没有对应的存储去实时保留，或者摘出来之后，放回去还能用
#无状态服务: apache,nginx
#有状态服务: mysql  monidb



#StatefullSet是为了解决有状态服务的问题


1. 稳定的持久化存储，即pod重新调度后，还是能访问到相同的持久化数据,基于PVC来实现
	#数据持久化
2. 稳定的网络标志，即pod重新调度后器podname(pod名称)和hostname(主机名称)基于HS(headless Service)(即没有Cluster IP的service)来实现
	#pod名，主机名不变
3. 有序部署,有序扩展, 即pod是有顺序的，在部署或扩展的时候要一句定义的顺序，依次依次进行
(级从0到N+1，在下一个pod运行之前所有之前的pod都是Running和Ready状态)基于init containers实现
	#有序部署（扩展，回收）
	#在大型环境中是有指定的pod启动顺序的
	# nginx  (3)
	# php    (2)
	# mysql  (1)

4. 有序收缩，有序删除(即从N-1到0)
```



