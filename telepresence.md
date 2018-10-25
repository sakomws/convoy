
# Multi k8s clusters development using Telepresence

Below is described how to facilitate application development in running in kubernetes.

Currently, the development process is as below:

![Alt text](images/asis.png?raw=true "OpenGov")

1. Change code
2. Build a Docker image
3. Push the Docker image to a Docker registry in the cloud
4. Update the Kubernetes cluster to use new image
5. Wait for the image to download

In order to optimize process, we are going to present Telepresence. It works by building a two-way network proxy (bootstrapped using kubectl port-forward)
There are 3 proxying methods can be used:

![Alt text](images/telepresence.png?raw=true "OpenGov")

1. VPN - works best with GO, not other VPNs
2. Inject-TCP - inject shared lib to process, not works with statically linked
3. Docker - ideal for container-native development
 
## Prerequisities

1. Install [kubectl client](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

2. Install [Docker](https://docs.docker.com/) depending on your OS.


## Getting Started

Sample application that shows how to connect a frontend system (fe) running in localhost with a backend (be) in remote cluster.

### Remote Cluster

1. Deploy backend and frontend applications in remote cluster:
```
./remote_cluster.sh
kubectl apply -f backend.yaml,frontend.yaml -n convox
```

2. Make sure fe-*, be-* pods and services are up and running :
```
kubectl get pods,svc -n convox
```
```
NAME                            READY   STATUS    RESTARTS   AGE
pod/backend-5cbcdfd67b-x2sbs         1/1     Running   0          23m
pod/frontend-7b77896f9b-5rnhc   1/1     Running   0          102s

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/be-srv   ClusterIP   100.68.217.68   <none>        80/TCP    23m
service/fe-srv   ClusterIP   100.70.86.99    <none>        80/TCP    4m10s
```

### Localhost

1. Build and push frontend Docker image:
```
docker build -t sakomws/frontend fe/
docker push sakomws/frontend
```

2. Start telepresence in convox namespace:
```
telepresence --namespace convox --run-shell
```

3. Make sure can reach frontend and backend: 
```
curl http://frontend-srv
curl http://backend-srv
```

4. Swap the remote frontend service with local one:

a) To docker container:
```
telepresence --swap-deployment frontend --namespace convox  --docker-run -v $(pwd):/service  -p 80:80 --rm -it sakomws/frontend:latest
```

b) To local server:
```
telepresence --swap-deployment frontend --namespace convox  --run-shell
python3 -m http.server 8000 
```

OR: 

```
telepresence --swap-deployment frontend --namespace convox --expose 80 \
--run python3 -m http.server 8000 
```

5. Create new service in remote cluster and forward to local server:
```
telepresence --new-deployment hello-world --namespace convox --expose 9000
python3 helloworld.py

```

### Uninstall

1. Uninstall backend application in remote cluster:
```
./remote_cluster.sh
kubectl delete -f backend.yaml,frontend.yaml -n convox
```


## References

[Kubehelloworld Sample application](https://github.com/salrashid123/kubehelloworld) project authored by [Sal Rashid](https://github.com/salrashid123).

## Authors

[Opengov](https://opengov.com) Devops team.
