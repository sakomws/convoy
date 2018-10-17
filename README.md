# Multi k8s clusters development

Below is described how to facilitate application development in running in kubernetes.

## Prerequisities

1. Install 1.13 release of kubectl client:
```
chmod +x ./kubectl_client/kubectl; sudo mv ./kubectl_client/kubectl /usr/local/bin/kubectl
```
2. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) depending on your OS:

3. Optionally can use [Skaffold](https://github.com/GoogleContainerTools/skaffold) project, to facilitate continuous development.

## Architecture
The diagram below depicts how enable multi cluster development in k8s:
![Alt text](images/architecture.png?raw=true "OpenGov")

## Getting Started

Sample application that shows how to connect a frontend system (fe) with a backend (be) in different clusters.

### Remote Cluster

1. Deploy backend application in remote cluster:
```
./remote_cluster.sh
kubectl apply -f backend.yaml
```

2. Make sure be-* pod and service are up and running :
```
kubectl get pods,svc
```
```
NAME                          READY   STATUS    RESTARTS   AGE
pod/be-rc-czn64               1/1     Running   1          7d5h

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/be-srv       ClusterIP   100.70.181.149   <none>        5000/TCP   7d5h
```

3. Forward the backend service to localhost:
```
kubectl port-forward --address=192.168.99.1 $(kubectl get pod -l type=be-type -o jsonpath='{.items[0].metadata.name}') 2000:5000 &
```
4. Check backend service via browser or curl:
```
curl http://192.168.99.1:2000/
```

### Minikube

1. Deploy fronted application in minikube cluster:
```
./minikube.sh
kubectl apply -f frontend.yaml
```
2. Make sure fe-* pod and service are up and running :
```
kubectl get pods,svc
```
```
NAME                                            READY   STATUS    RESTARTS   AGE
pod/fe-rc-dlghf                                 1/1     Running   1          11h

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/fe-srv       ClusterIP   10.97.237.77     <none>        8080/TCP       11h
```

3. Forward the frontend service to localhost:
```
kubectl port-forward $(kubectl get pod -l type=fe-type -o jsonpath='{.items[0].metadata.name}') 3000:8080 &
```
4. Check frontend service is up and can forward requests to backend:
```
curl http://localhost:3000/
```

### Uninstall

1. Remove any kubectl port-forward processes that may still be running:
```
killall kubectl
```

2. Uninstall backend application in remote cluster:
```
./remote_cluster.sh
kubectl delete -f backend.yaml
```

3. Deploy frontend application in minikube cluster:
```
./minikube.sh
kubectl delete -f frontend.yaml
```

## References

[PR46517](https://github.com/kubernetes/kubernetes/pull/46517/commits/4643c6e95e0a0cf6561554fb3b9a1bc59bcead0c) to enable binding port-forwarding to different IP in localhost.

[Kubectl Client 1.13](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#client-binaries) artifact repository.

[Kubehelloworld Sample application](https://github.com/salrashid123/kubehelloworld) project authored by [Sal Rashid](https://github.com/salrashid123).

## Authors

[Opengov](https://opengov.com) Devops team.
