
# Use case 03

This use case shows how  to facilitate local frontend development with backend run in remote cluster.

### Prerequisities

1. Install latest [Kubectl client](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

2. Install latest [Telepresence](https://www.telepresence.io/reference/install).

3. Install latest [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).


### Architecture
The diagram below depicts how to enable multi cluster development in k8s:

![Alt text](images/telepresence_use_case_03.png?raw=true "OpenGov")


### Getting Started

Sample application that shows how to swap service in remote cluster with service running in local host.

#### Remote Cluster or Minikube

1. Deploy backend and frontend applications in remote cluster:
```
kubectl apply -f backend.yaml,frontend.yaml
```

2. Make sure frontend-*, backend-* pods and services are up and running:
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

1. Swap frontend service in remote cluster to forward requests to local server:
```
telepresence --swap-deployment frontend --namespace default  --run-shell
python3 -m http.server 8000
```

OR:

```
telepresence --swap-deployment frontend --namespace default --expose 8000 \
--run python3 -m http.server 8000
```

2. Make sure a new frontend deployment created in place of existing one:
```
kubectl get pods,svc -n default
```
```
NAME                                                         READY     STATUS    RESTARTS   AGE
backend-59fd8d4bb7-69bxc                                     1/1       Running   0          4d
frontend-a59bdbf638fe4d36904aed7b7d4c8771-55988dd464-xfnsg   1/1       Running   0          24s
```
3.  In another terminal, make sure can reach frontend service running in remote cluster which forwards requests to local server:
```
curl http://frontend-srv.default:8000/
```
```
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

4. Make sure requests appear in local server logs:
```
python3 -m http.server 8000
```
```
T: Forwarding remote port 8000 to local port 8000.

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 - - [02/Nov/2018 13:45:40] "GET / HTTP/1.1" 200 -
```
Note: Exiting telepresence will restore frontend service running in remote cluster.

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
