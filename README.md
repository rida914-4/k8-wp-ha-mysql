
# Highly scalable/available Wordpress website backed by Stateful Kubernetes Cluster

We need to create a highly scalable and available wordpress website which has multiple replica wordpress pods and MySQL pods. We are going to use stateful replication of MySQL slaves which clone from the master one by one in order to scale and ease the master pod’s burden. We are going to create a single node based wordpress deployment and then replicate the pods by scaling. This will ensure that a pod goes down or one part of the website crashes, the kubernetes cluster would automatically reboot the pad and run as per the configureation


There is a stateful controller which controls the pods, the services, looks up ConfigMaps to configure these entities.We create a configMap file which is going to override the default mysql my.cnf file. We want our master and slave mysql clusters to follow the configurations specified by us. 

We deploy persistence volumes which are individual for the mysql slave clones. We also create a claim through which we can access the data using the name and the read/write permissions. 

Horizontal Pod Autoscaler (HPA) is used to replicate the replicas. We have created two autoscalers for each of the deployment i.e wordpress and MySQL. 
 
##### Setup Kubernetes

Install minikube offline on a linux academy server
https://kubernetes.io/docs/tasks/tools/install-minikube

Run minikube on a single cluster
``` sh
minikube addons enable metrics-server
kubectl get apiservices
minikube start
```

Create a kustomization file. Generate a 64 character password and add it inside for MySQL authentication. Also specify the configuration files for the custom cluster and apply the files.
``` sh
kubectl apply -k ./
```
Access the NodePort 30060 specified in the wordpress configuration which allows us to access the instance. The website can be accessed by concatenating the minikube ip and the port. We can directly get the URL as well.Get WP URL
``` sh
minikube service wordpress --url
``` 
##### Cleanup 

``` sh
 kubectl get pods --no-headers=true --all-namespaces |sed -r 's/(\S+)\s+(\S+).*/kubectl --namespace \1 delete pod \2/e'

kubectl delete deployment,svc,pods,pvc,rc,rs  --all
``` 
##### Debugging
Check each component on each node/node/pod
```sh
Kubectl cluster-info
Kubectl get componentstatus/nodes/pods
kubectl get deployment,svc,pods,pvc,rc,rs
```

##### Issues
The wordpress pod has insufficient CPU error. We debugged the problem by using this command.

```sh
 kubectl top pod wordpress-588d54b55b-bdpc5 --namespace=default
```
We need to change the CPU limits in the wordpress deployment file and make sure that we are not specifying a CPU request that is too big for our nodes. As we can see the pod has moved past the error and is now creating containers.

```sh
https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/
```


