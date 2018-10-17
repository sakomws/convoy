# Multi k8s clusters development

Below is described how to facilitate multicluster development in kubernetes.

## Prerequisities

1. Install 13-alpha1 kubectl client binary:
```
chmod +x ./kubectl_client/kubectl; sudo mv ./kubectl_client/kubectl /usr/local/bin/kubectl
```
2. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) depending on your OS:

3. Optionally can use [Skaffold](https://github.com/GoogleContainerTools/skaffold) project, to facilitate continuous development.


## Architecture
The diagram below depicts how enable multi cluster development in k8s:
![Alt text](images/architecture.png?raw=true "OpenGov")

## Getting Started

1. Deploy backend application in remote cluster:
```
./remote_cluster.sh
kubectl apply -f backend.yaml
```

2. Forward the backend service to localhost:
```
kubectl port-forward --address=192.168.99.1 $(kubectl get pod -l type=be-type -o jsonpath='{.items[0].metadata.name}') 2000:5000 &
```
3. Make sure backend service is up:
```
curl http://192.168.99.1:2000/
```

4. Deploy fronted application in minikube cluster:
```
./minikube.sh
kubectl apply -f frontend.yaml
```

5. Forward the frontend service to localhost:
```
kubectl port-forward $(kubectl get pod -l type=fe-type -o jsonpath='{.items[0].metadata.name}') 3000:8080 &
```
6. Make sure frontend service is up and can forward requests to backend:
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

https://github.com/kubernetes/kubernetes/pull/46517/commits/4643c6e95e0a0cf6561554fb3b9a1bc59bcead0c

https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#client-binaries


## Authors

[Opengov](https://opengov.com) Devops team.

## Acknowledgments

Thanks to [Kubehelloworld)](https://github.com/salrashid123/kubehelloworld) project authored by [Sal Rashid](https://github.com/salrashid123).
