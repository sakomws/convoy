
# Use case 01

This use case shows how  to facilitate local frontend development with backend run in remote cluster without any user impact.

### Prerequisities

1. Install latest [Kubectl client](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

2. Install latest [Docker](https://docs.docker.com/).

3. Install latest [Telepresence](https://www.telepresence.io/reference/install).

4. Install latest [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

5. Install latest [Go language](https://golang.org/doc/install).

### Architecture 
The diagram below depicts how to enable multi cluster development in k8s:

![Alt text](images/telepresence_use_case_01.png?raw=true "OpenGov")


### Getting Started

Sample application that shows how to connect a frontend in development with a production backend run in remote cluster.

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

1. Start telepresence:
```
telepresence --namespace default --run-shell
```

2. Make sure a new telepresence deployment created:
```
kubectl get pods,svc -n default
```
```
NAME                                                       READY   STATUS    RESTARTS   AGE
pod/backend-59fd8d4bb7-4kwsp                               1/1     Running   0          42m
pod/frontend-69cf9fd844-mbktv                              1/1     Running   0          8m19s
pod/telepresence-1540939716-464121-8623-74f495799b-c2bsg   1/1     Running   0          30s
```

Note: To stop telepresence, please type 'exit' in terminal, which will automatically delete telepresence deployment from remote cluster.

3. In another terminal, make sure can reach frontend and backend services running in remote cluster: 
```
curl http://frontend-srv.default.svc.cluster.local:8000
```
```
curl http://frontend-srv.default.svc.cluster.local:8000
lookup _backend._http.backend-srv.default.svc.cluster.local on 100.64.0.10:53: no such host
SRV CNAME:
Pod Name: frontend-69cf9fd844-mbktv
Pod Namespace: default
USER_VAR:

Kubenertes environment variables
BACKEND_SRV_SERVICE_HOST = 100.66.71.4
BACKEND_SRV_SERVICE_PORT = 80
BE_SRV_SERVICE_HOST = backend-srv.default.svc.cluster.local
BE_SRV_SERVICE_PORT = 80
FRONTEND_SRV_SERVICE_HOST = 100.70.233.71
FRONTEND_SRV_SERVICE_PORT = 8000
KUBERNETES_SERVICE_HOST = 100.64.0.1
KUBERNETES_SERVICE_PORT = 443

Found ENV lookup backend ip: backend-srv.default.svc.cluster.local port: 80
ENV Lookup Response from backend
BACKEND Response
Found DNS lookup backend ip:  port:
```

```
curl http://backend-srv.default.svc.cluster.local
```
```
BACKEND Response
```

4. Make sure dev-frontend service running in localhost can reach backend service running in remote cluster:

a) Go to frontend application directory and compile it:
```
go build -o frontend .
```

b) Set backend environment variables used by frontend application:
```
export BE_SRV_SERVICE_HOST=backend-srv.default.svc.cluster.local
export BE_SRV_SERVICE_PORT=80
```  

c) Run compiled frontend binary:
```
./frontend

```

d) Make sure can reach dev-frontend service which in turn can reach backend service in remote cluster:
```
curl http://localhost:8000
```
```
lookup _backend._http.backend-srv.convox.svc.cluster.local on 8.8.8.8:53: no such host
SRV CNAME:
Pod Name:
Pod Namespace:
USER_VAR:

Kubenertes environment variables
BE_SRV_SERVICE_HOST = backend-srv.convox.svc.cluster.local

Found ENV lookup backend ip: backend-srv.convox.svc.cluster.local port:
ENV Lookup Response from backend
BACKEND Response
Found DNS lookup backend ip:  port:
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
