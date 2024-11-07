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
    
    minikube start --profile sparkhdfs --cpus 6 --memory 15360 --driver virtualbox --no-vtx-check

## Etape 2 : 

    minikube ssh --profile sparkhdfs

## Etape 3 : 

    docker pull gradiant/hdfs-datanode 

    docker pull gradiant/hdfs-namenode

    docker pull bitnami/spark:3.3.1


## Etape 4 : 

    kubectl appl -f https://github.com/crystalloide/Hadoop-helm/blob/main/spark-yaml/hdfs-configmap.yaml

## Etape 5 : 

## Montage du répertoire "/dataset" :  le ":" sépare le point de montage local (c:\dataset ou /home/user/dataset) de celui en cible "/dataset":
## minikube mount --profile sparkhdfs /home/user/dataset:/dataset
    minikube mount --profile sparkhdfs ~/dataset:/dataset

    minikube ssh --profile sparkhdfs
    
    ls ~/


## Etape 6 : 

    kubectl appl -f https://github.com/crystalloide/Hadoop-helm/blob/main/spark-yaml/sparkhdfs.yaml

## Etape 7 : 

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

#### Lancement d'un job spark : 

    bin/spark-submit --master spark://sparkhdfs-master-namenode-0.sparkhdfs-master-namenode.default.svc.cluster.local:7077 --num-executors 2 --total-executor-cores 1 --executor-memory 1g \
    --class test.spark.arrivalDelay2008 6  /dataset/sparkexample.jar sparkhdfs-master-namenode


## Etape 18 :

#### On va sur l'UI Spark et on regarde "Running Applications"

#### Navigateur web sur l'URL : 

localhost:8080



# Fin du TP
