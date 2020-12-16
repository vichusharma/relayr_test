# This set of helm charts deploy the following 

1. A postgress database with a persistant volume of 4GB 
2. A front end with Kanaban interface which will conenct to backend 
3. A back end with Kanban backend which connects to postgress database
4. An ingress which provides the interfaces on 
    a. http://adminer.k8s.com/ and 
    b. http://kanban.k8s.com/ 


This is Deployed on minikube running on Windows 10  using nginx as ingress controller to access the URL's mentioned above you have to  edit C:/Windows/system32/drivers/etc/hosts file to point to your minikube IP. 

All the files are bough together by use of Helm and varaiblizing various deployment,svc, ingress, configmap and pvc yaml files for easier maintance. 

## Pre-requusites : 

1. A Virtualization platform (Hyper-v in my case) on your laptop or desktop 
2. You should have minikube installed which can be done  
3. Enable nginx ingress controller using 
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
  minikube addons enable ingress

## How to Deploy and test: 

1.Clone this repository to local 
2.From the directory where the files are downloaded run the following commands 

### To deploy Postgress run the following command

helm install -f kanban-postgres.yaml postgres ./postgres

Which will create following resources
1. A persistent volume claim 
2. A configuration map 
3. A Deployment of postgress 
4. A service for postgress

**How to Verify**

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get cm                                                                                                                                                                                     NAME              DATA   AGE
postgres-config   3      20h

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get deployment postgres
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
postgres   1/1     1            1           20h

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get svc postgres
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
postgres   ClusterIP   10.100.94.47   <none>        5432/TCP   20h

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get pvc
NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-persistent-volume-claim    Bound    pvc-9f892a4e-5f5f-4ebc-8c3d-e89cc984abfd   4Gi        RWO            standard       20h

### To deploy Kanaban UI run the following command
helm install -f kanban-ui.yaml kanban-ui ./app

Which will create following resources

1. Deployment with name kanaban-ui 
2. Service with name kanban-ui 

**How to Verify** 

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get svc kanban-ui                                                                                                               NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kanban-ui   ClusterIP   10.98.140.117   <none>        8080/TCP   20h
PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get deployment kanban-ui                                                                                                        NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kanban-ui   1/1     1            1           20h

### To deploy Kanaban Backend run the following command

helm install -f kanban-app.yaml kanban-app ./app

Which will create following resources 

1. Deployment with name kaban-app
2. Service with name kanban-app

**How to Verify**

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get deployment kanban-app                                                                                                       NAME         READY   UP-TO-DATE   AVAILABLE   AGE
kanban-app   1/1     1            1           20h
PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get svc kanban-app                                                                                                             NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kanban-app   ClusterIP   10.96.227.101   <none>        8080/TCP   20h
PS C:\Vishnu\Kube_lab\Relayr\Relayr_test>                                                                                                                                                                                                                                                                                           

With this all the services and deployments done the application is avilable on the minikube but you can't access this outside the Kubernetes cluster to do this we need to deploy 
an ingress, as you remember from pre-requisite we installed ngingx ingress controller and enable minikube plugin of ingress. 

### To deploy Ingress to access the application out side 

helm install -f ingress.yaml ingress ./ingress

**Which will create following resources**

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get ingress                                                                                                                     
NAME              CLASS    HOSTS                            ADDRESS         PORTS   AGE
ingress-service   <none>   adminer.k8s.com,kanban.k8s.com   172.17.27.250   80      21h


## All the items which are running now on the minikube 

PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get all -n default                                                                                                                                                                         NAME                              READY   STATUS    RESTARTS   AGE
pod/kanban-app-6c4d97ff87-q7vkb   1/1     Running   2          20h
pod/kanban-ui-7f7c5f974d-hwlf5    1/1     Running   3          20h
pod/postgres-69dc848d8-2ljnc      1/1     Running   1          20h

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kanban-app   ClusterIP   10.96.227.101   <none>        8080/TCP   20h
service/kanban-ui    ClusterIP   10.98.140.117   <none>        8080/TCP   20h
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    20h
service/postgres     ClusterIP   10.100.94.47    <none>        5432/TCP   20h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kanban-app   1/1     1            1           20h
deployment.apps/kanban-ui    1/1     1            1           20h
deployment.apps/postgres     1/1     1            1           20h

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/kanban-app-6c4d97ff87   1         1         1       20h
replicaset.apps/kanban-ui-7f7c5f974d    1         1         1       20h
replicaset.apps/postgres-69dc848d8      1         1         1       20h
PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get all -n ingress-nginx                                                                                                                                                                   NAME                                            READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-54b86f8f7b-8xjgp   1/1     Running   1          21h

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-controller   1/1     1            1           21h

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-controller-54b86f8f7b   1         1         1       21h
PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get pvc                                                                                                                          NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-persistent-volume-claim    Bound    pvc-9f892a4e-5f5f-4ebc-8c3d-e89cc984abfd   4Gi        RWO            standard       21h
PS C:\Vishnu\Kube_lab\Relayr\Relayr_test> kubectl get cm                                                                                                                         NAME              DATA   AGE
postgres-config   3      20h


