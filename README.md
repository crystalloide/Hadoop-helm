# Hadoop : installation lancement et utilisation dans gitpod


[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/crystalloide/Hadoop-helm)

### https://github.com/crystalloide/Hadoop-helm

### https://gitpod.io/workspaces

### https://hub.docker.com/r/apache/hadoop


# Apache Hadoop

Apache Hadoop est un framework qui permet le traitement distribué de grands ensembles de données sur des clusters d'ordinateurs,

à l'aide de modèles de programmation simples. 

Il est conçu pour passer d'un seul serveur à des milliers de machines, chacune offrant un calcul et un stockage locaux. 

Plutôt que de s'appuyer sur du matériel pour offrir une haute disponibilité, la bibliothèque elle-même est conçue pour détecter 

et gérer les pannes au niveau de la couche application, 

fournissant ainsi un service hautement disponible au-dessus d'un cluster d'ordinateurs, 

dont chacun peut être sujet à des pannes.

## Démarrage rapide

Un cluster Hadoop peut être créé en extrayant l'image Docker appropriée et en spécifiant les configurations requises.

Ici on va utiliser un déploiement Kubernetes 


## Lancement de l'environnement de démo : 

Optionnel : si vous utilisez un environnement autre que Gitpod, on va d'abord cloner l'environnement : 

```bash

cd ~
 
rm -Rf ~/Hadoop-helm

git clone https://github.com/crystalloide/Hadoop-helm

cd Hadoop-helm

```


## Etape 1 :

   #### Si besoin : 
   
    minikube delete -p sparkhdfs  

   #### On lance : 
    
    minikube start --profile sparkhdfs --cpus 6 --memory 15360 --driver docker --no-vtx-check

## Etape 2 : 

    minikube ssh --profile sparkhdfs

## Etape 3 : 

    docker pull gradiant/hdfs-datanode 

    docker pull gradiant/hdfs-namenode

    docker pull bitnami/spark:3.3.1


## Etape 4 : 

    kubectl apply -f ~/Hadoop-helm/spark-yaml/hdfs-configmap.yaml

## Etape 5 : 

#### Montage du répertoire "/dataset" :  le ":" sépare le point de montage local (c:\dataset ou /home/user/dataset) de celui en cible "/dataset":

    mkdir ~/dataset
    
    ls ~/

#### minikube mount --profile sparkhdfs /home/user/dataset:/dataset

    minikube mount --profile sparkhdfs ~/dataset:/dataset

    minikube ssh --profile sparkhdfs
    
    ls ~/

#### Dans un autre terminal : 
    minikube ssh --profile sparkhdfs

    ls /

#### On voit bien /dataset qui est monté :-)

#### On sort : 

    exit 


## Etape 6 : 

    kubectl appl -f https://github.com/crystalloide/Hadoop-helm/blob/main/spark-yaml/sparkhdfs.yaml

## Etape 7 : On va regarder le dashboard : 

#### Mais avant on active les métriques : 

    minikube -p sparkhdfs addons enable metrics-server
    
    minikube dashboard --profile sparkhdfs

    
## Etape 8 : 

    kubectl port-forward svc/sparkhdfs-master-namenode 8080:8080 50070:50070


## Etape 9 : 

#### Navigateur web sur l'URL : 

127.0.0.1:53279/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads


## Etape 10 :

#### Navigateur web sur l'URL : 

localhost:50070/dfshealth.html#tab-datanode

localhost:50070/dfshealth.html#tab-overview

#### On voit les deux datanodes notamment


## Etape 11 :

#### Navigateur web sur l'URL : 

localhost:8080

#### On voit l'UI Spark et l'état du cluster Spark

## Etape 12 :

#### Sur la console Kubernetes, on va en mode console (SSH) sur le pod sparkhdfs-master-namenode-0  (= namenode HDFS)

#### Workloads > Pods > sparkhdfs-master-namenode-0 > Shell 

#### Prompt : "bash-4.4$" 

    cd /dataset

#### les données sont déjà disponibles : "1987*"

## Etape 13 :

#### on crée le répertoire cible dans HDFS : 

    hdfs dfs -mkdir /formation 

    hdfs dfs -ls /

## Etape 14 :

#### et on copie les données à traiter : 

    hdfs dfs -put /dataset/1987.csv /formation 

## Etape 15 :

#### Navigateur web sur l'URL : 
localhost:50070/explorer.html#

#### On regarde le répertoire HDFS /formation et son contenu


## Etape 16 :

#### Sur la console Kubernetes, on ouvre une nouvelle console (SSH) sur le pod sparkhdfs-master-namenode-0  (= namenode HDFS)

#### Workloads > Pods > sparkhdfs-master-namenode-0 > Shell 

#### Prompt : "I have no name!@sparkhdfs-master--namenode-0:/opt/bitnami/spark$" 


## Etape 17 :

#### Du code pour lancer des exemples est présent : 

    ls examples/jars/spark-examples_2.12-3.3.1.jar

#### Lancement d'un job spark pour calculer PI : 
    
    bin/spark-submit --class org.apache.spark.examples.SparkPi --master spark://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:7077 \
    --num-executors 2 --total-executor-cores 1 --executor-memory 1g \
    ./examples/jars/spark-examples_2.12-3.3.1.jar 100



## Etape 18 :

#### On va sur l'UI Spark et on regarde "Running Applications"

#### Navigateur web sur l'URL : 

localhost:8080


## Etape 19 :

#### Log de notre application : 
	I have no name!@sparkhdfs-master-namenode-0:/opt/bitnami/spark$ bin/spark-submit --class org.apache.spark.examples.SparkPi --master spark://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:7077 --num-executors 2 --total-executor-cores 1 --executor-memory 1g ./examples/jars/spark-examples_2.12-3.3.1.jar 100
	24/11/07 17:20:35 INFO SparkContext: Running Spark version 3.3.1
	24/11/07 17:20:36 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
	24/11/07 17:20:36 INFO ResourceUtils: ==============================================================
	24/11/07 17:20:36 INFO ResourceUtils: No custom resources configured for spark.driver.
	24/11/07 17:20:36 INFO ResourceUtils: ==============================================================
	24/11/07 17:20:36 INFO SparkContext: Submitted application: Spark Pi
	24/11/07 17:20:36 INFO ResourceProfile: Default ResourceProfile created, executor resources: Map(cores -> name: cores, amount: 1, script: , vendor: , memory -> name: memory, amount: 1024, script: , vendor: , offHeap -> name: offHeap, amount: 0, script: , vendor: ), task resources: Map(cpus -> name: cpus, amount: 1.0)
	24/11/07 17:20:36 INFO ResourceProfile: Limiting resource is cpu
	24/11/07 17:20:36 INFO ResourceProfileManager: Added ResourceProfile id: 0
	24/11/07 17:20:36 INFO SecurityManager: Changing view acls to: spark
	24/11/07 17:20:36 INFO SecurityManager: Changing modify acls to: spark
	24/11/07 17:20:36 INFO SecurityManager: Changing view acls groups to: 
	24/11/07 17:20:36 INFO SecurityManager: Changing modify acls groups to: 
	24/11/07 17:20:36 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(spark); groups with view permissions: Set(); users  with modify permissions: Set(spark); groups with modify permissions: Set()
	24/11/07 17:20:37 INFO Utils: Successfully started service 'sparkDriver' on port 39231.
	24/11/07 17:20:37 INFO SparkEnv: Registering MapOutputTracker
	24/11/07 17:20:37 INFO SparkEnv: Registering BlockManagerMaster
	24/11/07 17:20:37 INFO BlockManagerMasterEndpoint: Using org.apache.spark.storage.DefaultTopologyMapper for getting topology information
	24/11/07 17:20:37 INFO BlockManagerMasterEndpoint: BlockManagerMasterEndpoint up
	24/11/07 17:20:37 INFO SparkEnv: Registering BlockManagerMasterHeartbeat
	24/11/07 17:20:37 INFO DiskBlockManager: Created local directory at /tmp/blockmgr-bc3a2d79-0328-4520-bb81-048206836390
	24/11/07 17:20:37 INFO MemoryStore: MemoryStore started with capacity 366.3 MiB
	24/11/07 17:20:37 INFO SparkEnv: Registering OutputCommitCoordinator
	24/11/07 17:20:38 INFO Utils: Successfully started service 'SparkUI' on port 4040.
	24/11/07 17:20:38 INFO SparkContext: Added JAR file:/opt/bitnami/spark/examples/jars/spark-examples_2.12-3.3.1.jar at spark://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:39231/jars/spark-examples_2.12-3.3.1.jar with timestamp 1731000035829
	24/11/07 17:20:38 INFO StandaloneAppClient$ClientEndpoint: Connecting to master spark://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:7077...
	24/11/07 17:20:38 INFO TransportClientFactory: Successfully created connection to sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local/10.244.0.3:7077 after 87 ms (0 ms spent in bootstraps)
	24/11/07 17:20:38 INFO StandaloneSchedulerBackend: Connected to Spark cluster with app ID app-20241107172038-0002
	24/11/07 17:20:38 INFO StandaloneAppClient$ClientEndpoint: Executor added: app-20241107172038-0002/0 on worker-20241107161028-10.244.0.8-34389 (10.244.0.8:34389) with 1 core(s)
	24/11/07 17:20:38 INFO StandaloneSchedulerBackend: Granted executor ID app-20241107172038-0002/0 on hostPort 10.244.0.8:34389 with 1 core(s), 1024.0 MiB RAM
	24/11/07 17:20:38 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 36313.
	24/11/07 17:20:38 INFO NettyBlockTransferService: Server created on sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:36313
	24/11/07 17:20:38 INFO BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
	24/11/07 17:20:38 INFO StandaloneAppClient$ClientEndpoint: Executor updated: app-20241107172038-0002/0 is now RUNNING
	24/11/07 17:20:38 INFO BlockManagerMaster: Registering BlockManager BlockManagerId(driver, sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local, 36313, None)
	24/11/07 17:20:38 INFO BlockManagerMasterEndpoint: Registering block manager sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:36313 with 366.3 MiB RAM, BlockManagerId(driver, sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local, 36313, None)
	24/11/07 17:20:38 INFO BlockManagerMaster: Registered BlockManager BlockManagerId(driver, sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local, 36313, None)
	24/11/07 17:20:38 INFO BlockManager: Initialized BlockManager: BlockManagerId(driver, sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local, 36313, None)
	24/11/07 17:20:39 INFO StandaloneSchedulerBackend: SchedulerBackend is ready for scheduling beginning after reached minRegisteredResourcesRatio: 0.0
	24/11/07 17:20:40 INFO SparkContext: Starting job: reduce at SparkPi.scala:38
	24/11/07 17:20:40 INFO DAGScheduler: Got job 0 (reduce at SparkPi.scala:38) with 100 output partitions
	24/11/07 17:20:40 INFO DAGScheduler: Final stage: ResultStage 0 (reduce at SparkPi.scala:38)
	24/11/07 17:20:40 INFO DAGScheduler: Parents of final stage: List()
	24/11/07 17:20:40 INFO DAGScheduler: Missing parents: List()
	24/11/07 17:20:40 INFO DAGScheduler: Submitting ResultStage 0 (MapPartitionsRDD[1] at map at SparkPi.scala:34), which has no missing parents
	24/11/07 17:20:40 INFO MemoryStore: Block broadcast_0 stored as values in memory (estimated size 4.0 KiB, free 366.3 MiB)
	24/11/07 17:20:40 INFO MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 2.3 KiB, free 366.3 MiB)
	24/11/07 17:20:40 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:36313 (size: 2.3 KiB, free: 366.3 MiB)
	24/11/07 17:20:40 INFO SparkContext: Created broadcast 0 from broadcast at DAGScheduler.scala:1513
	24/11/07 17:20:40 INFO DAGScheduler: Submitting 100 missing tasks from ResultStage 0 (MapPartitionsRDD[1] at map at SparkPi.scala:34) (first 15 tasks are for partitions Vector(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14))
	24/11/07 17:20:40 INFO TaskSchedulerImpl: Adding task set 0.0 with 100 tasks resource profile 0
	24/11/07 17:20:43 INFO CoarseGrainedSchedulerBackend$DriverEndpoint: Registered executor NettyRpcEndpointRef(spark-client://Executor) (10.244.0.8:36896) with ID 0,  ResourceProfileId 0
	24/11/07 17:20:44 INFO BlockManagerMasterEndpoint: Registering block manager 10.244.0.8:36795 with 366.3 MiB RAM, BlockManagerId(0, 10.244.0.8, 36795, None)
	24/11/07 17:20:44 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0) (10.244.0.8, executor 0, partition 0, PROCESS_LOCAL, 4582 bytes) taskResourceAssignments Map()
	24/11/07 17:20:45 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on 10.244.0.8:36795 (size: 2.3 KiB, free: 366.3 MiB)
	24/11/07 17:20:46 INFO TaskSetManager: Starting task 1.0 in stage 0.0 (TID 1) (10.244.0.8, executor 0, partition 1, PROCESS_LOCAL, 4582 bytes) taskResourceAssignments Map()
	...
	...
	...
	24/11/07 17:20:49 INFO DAGScheduler: Job 0 is finished. Cancelling potential speculative or zombie tasks for this job
	24/11/07 17:20:49 INFO TaskSchedulerImpl: Killing all running tasks in stage 0: Stage finished
	24/11/07 17:20:49 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 9.489825 s
	Pi is roughly 3.1421839142183914
	24/11/07 17:20:49 INFO SparkUI: Stopped Spark web UI at http://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:4040
	24/11/07 17:20:49 INFO StandaloneSchedulerBackend: Shutting down all executors
	24/11/07 17:20:49 INFO CoarseGrainedSchedulerBackend$DriverEndpoint: Asking each executor to shut down
	24/11/07 17:20:49 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
	24/11/07 17:20:49 INFO MemoryStore: MemoryStore cleared
	24/11/07 17:20:49 INFO BlockManager: BlockManager stopped
	24/11/07 17:20:49 INFO BlockManagerMaster: BlockManagerMaster stopped
	24/11/07 17:20:49 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
	24/11/07 17:20:49 INFO SparkContext: Successfully stopped SparkContext
	24/11/07 17:20:49 INFO ShutdownHookManager: Shutdown hook called
	24/11/07 17:20:49 INFO ShutdownHookManager: Deleting directory /tmp/spark-874b60c5-a6b4-4869-b92e-7a4e449d89de
	24/11/07 17:20:49 INFO ShutdownHookManager: Deleting directory /tmp/spark-6eff1b3e-97c1-42de-9fc8-117f0b9570d6



#### Autre exemples : 

    bin/spark-submit --master spark://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:7077 --num-executors 2 --total-executor-cores 1 --executor-memory 1g \
    --class test.spark.arrivalDelay2008 6  /dataset/sparkexample.jar sparkhdfs-master-namenode

#### 
    
    https://spark.apache.org/examples.html

####

    https://spark.apache.org/docs/latest/

#### 

    ./bin/spark-submit examples/src/main/python/pi.py 10


# Fin du TP
