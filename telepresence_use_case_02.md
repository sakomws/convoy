
# Use case 02

This use case shows how  to facilitate local development with services running in remote cluster.

### Prerequisities

1. Install latest [Kubectl client](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

2. Install latest [Telepresence](https://www.telepresence.io/reference/install).

3. Install latest [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

### Architecture
The diagram below depicts how to enable multi cluster development in k8s:

![Alt text](images/telepresence_use_case_02.png?raw=true "OpenGov")


### Getting Started

Sample application that shows how to create new service as a container in remote cluster and forward requests to service in localhost.

#### Localhost

1. Create new service in remote cluster and forward to local server:
```
telepresence --new-deployment hello-world --namespace default --expose 9000
python3 helloworld.py
```

2. Make sure a new hello-world deployment created:
```
kubectl get pods,svc -n default
```
```
NAME                                                       READY   STATUS    RESTARTS   AGE
hello-world-5b976dc979-zk7qv   1/1       Running   0          9s
```

Note: To stop telepresence, please type 'exit' in terminal, which will automatically delete telepresence deployment from remote cluster.

3. In another terminal, make sure can reach hello-world service running in remote cluster which  forwards requests to local server:
```
curl http://hello-world.default:9000/
```
```
Hello, world!
```
4. Make sure requests appear in local server logs:
```
python3 helloworld.py
```
```
127.0.0.1 - - [02/Nov/2018 13:17:17] "GET / HTTP/1.1" 200 -
```


#### Uninstall

1. To stop Telepresence, type 'exit' or CTRL-D in terminal telepresence is active.

### Authors

[Opengov](https://opengov.com) Devops team.
