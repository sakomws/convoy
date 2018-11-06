
# Use case 02

This use case shows how  to facilitate local development with services running in remote cluster.

### Prerequisities

1. Install latest [Kubectl client](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

2. Install latest [Docker](https://docs.docker.com/).

3. Install latest [Telepresence](https://www.telepresence.io/reference/install).

4. Install latest [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

5. Install latest [Go language](https://golang.org/doc/install).

### Architecture 
The diagram below depicts how to enable multi cluster development in k8s:

![Alt text](images/telepresence_use_case_02.png?raw=true "OpenGov")


### Getting Started

Sample application that shows how to create new service as a container in remote cluster and forward to service in localhost.

#### Remote Cluster or Minikube

1. Deploy backend and frontend applications in remote cluster:
```
kubectl apply -f backend.yaml,frontend.yaml 
```

2. Make sure fe-*, be-* pods and services are up and running :
```
kubectl get pods,svc 
```
```
NAME                            READY   STATUS    RESTARTS   AGE
pod/backend-5cbcdfd67b-x2sbs         1/1     Running   0          23m
pod/frontend-7b77896f9b-5rnhc   1/1     Running   0          102s


NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/be-srv   ClusterIP   100.68.217.68   <none>        80/TCP    23m
service/fe-srv   ClusterIP   100.70.86.99    <none>        80/TCP    4m10s
```

#### Localhost

1. Create new service in remote cluster and forward to local server:
```
telepresence --new-deployment hello-world --namespace default --expose 9000
python3 helloworld.py

```

2. Make sure a new helloworld deployment created:
```
kubectl get pods,svc -n default
```
```
NAME                                                       READY   STATUS    RESTARTS   AGE
hello-world-5b976dc979-zk7qv   1/1       Running   0          9s
```

Note: To stop telepresence, please type 'exit' in terminal, which will automatically delete telepresence deployment from remote cluster.

3. In another terminal, make sure can reach from backend services running in remote cluster to frontend service in localhost: 
```
curl http://frontend-srv.default:8000/
```
```
curl http://frontend-srv.convox:8000/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".git/">.git/</a></li>
<li><a href="backend/">backend/</a></li>
<li><a href="backend.yaml">backend.yaml</a></li>
<li><a href="frontend/">frontend/</a></li>
<li><a href="frontend.yaml">frontend.yaml</a></li>
<li><a href="helloworld.py">helloworld.py</a></li>
<li><a href="images/">images/</a></li>
<li><a href="portforwarding.md">portforwarding.md</a></li>
<li><a href="README.md">README.md</a></li>
<li><a href="telepresence_use_case_01.md">telepresence_use_case_01.md</a></li>
<li><a href="telepresence_use_case_02.md">telepresence_use_case_02.md</a></li>
<li><a href="telepresence_use_case_03.md">telepresence_use_case_03.md</a></li>
<li><a href="telepresence_use_case_04.md">telepresence_use_case_04.md</a></li>
</ul>
<hr>
</body>
</html>
```

```
python3 helloworld.py
```
```
127.0.0.1 - - [02/Nov/2018 13:17:17] "GET / HTTP/1.1" 200 -
```


#### Uninstall

1. Uninstall backend application in remote cluster:
```
kubectl delete -f backend.yaml,frontend.yaml
```
2. To stop Telepresence, type 'exit' or CTRL-D in terminal telepresence is active.

### References

[Kubehelloworld](https://github.com/salrashid123/kubehelloworld) project authored by [Sal Rashid](https://github.com/salrashid123).

### Authors

[Opengov](https://opengov.com) Devops team.
