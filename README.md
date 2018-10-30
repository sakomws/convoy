# Multi k8s clusters development

### Motivation

What is the best approach to proxy traffic from developer laptop to services running in remote cluster:

- [ ] Little or no configuration change in application side
- [ ] No proxy maintenance
- [ ] Faster development process

### Current Development Work Flow

Below is usual software development process for applications running inside containers and deployed in kubernetes cluster:

1. Change source code
2. Build a Docker image
3. Push the Docker image to a Docker registry
4. Redeploy application in kubernetes to use new image
5. Wait for the image to download and start service


As we can see, for any small changes in application code, we need to reiterate the cycle which is time-consuming and not convenient. 

### Using Port-forwarding 

Using Port-forwarding feature in Kubernetes, deploy frontend in minikube cluster, and backend in remote cluster.
For detailed sample application use case, please follow [tutorial](portforwarding.md).

#### Advantages

* Only Minikube install in developer machine
* No any install in remote cluster
* Isolated from developer host OS
* Little or no difference in environments applications run
* Seamless proxy between Minikube and remote cluster
* Can forward traffic to pod, deployment, replica-set and service

#### Disadvantages

* Not stable, frequent connection lost to pod
* When the pod restarts, the tunnel breaks
* Load on host OS, due to Minikube running
* Each backend service needs to be port-forwarded
* UDP not supported, only TCP

### Using Telepresence

Open source project developed by Datawire and contributed to CNCF.  Telepresence works by building a two-way network proxy (bootstrapped using kubectl port-forward).

Telepresence works by running code locally, as a normal local process, and then forwarding requests to/from the Kubernetes cluster.

There are 3 proxying methods in Telepresence:

a) VPN - Uses a program called sshuttle to open a VPN-like connection to the Kubernetes cluster. Works best with Go language, limitations are:
* Cannot work on top of other VPNs
* Can run only 1 telepresence connection per machine
* Cloud resources like AWS RDS will not be routed automatically. Need to specify the hosts manually using --also-proxy, e.g. --also-proxy mydatabase.somewhere.vpc.aws.amazon.com to route traffic to that host via the Kubernetes cluster
* FQDN like services yourservice.default.svc.cluster.local won't resolve correctly on Linux
* Service endpoints like yourservice and yourservice.default will resolve correctly

b) Inject-TCP - By default this method is used. Injects shared library into the subprocess and  can run more than one telepresence connection, not works with:\
* statically linked libraries
* suid binaries in telepresence shell
* custom DNS resolvers that parse "/etc/resolv.conf" and do DNS lookups themselves

c) Docker - ideal for container-native development. New proxy container will start, and then call docker run with arguments passed to --docker-run to start a container that will have its networking proxied. All networking is proxied:
* Outgoing to Kubernetes
* Outgoing to cloud resources like AWS RDS added manually with --also-proxy
* Incoming connections to ports specified with --expose
* Volumes and environment variables from the remote Deployment are also available in the container


Below are 5 possible use cases with Telepresence:

1. Run dev-frontend as a service in localhost and frontend, backend as containers in remote cluster without any user impact:
For detailed sample application use case, please follow tutorial in [tutorial](telepresence_use_case_01.md).

![Alt text](images/telepresence_use_case_01.png?raw=true "OpenGov")

2. Create new service as a container in remote cluster and forward to service in localhost:
For detailed sample application use case, please follow tutorial in [tutorial](uc2.md).

![Alt text](images/telepresence_use_case_01.png?raw=true "OpenGov")

3. Swap service as a container in remote cluster with service in localhost:
For detailed sample application use case, please follow tutorial in [tutorial](uc3.md).

![Alt text](images/telepresence_use_case_03.png?raw=true "OpenGov")

4. Swap service as a container in remote cluster with service as a container in localhost:
For detailed sample application use case, please follow tutorial in [tutorial](uc4.md).

![Alt text](images/telepresence_use_case_04.png?raw=true "OpenGov")

5. Swap service_A in remote cluster with service in localhost_A and service_B in remote cluster with service in localhost_B :
For detailed sample application use case, please follow tutorial in [tutorial](uc5.md).


#### Advantages

* Fast local development
* Simple local setup
* Full access to other services in the remote cluster
* Full access to Kubernetes environment variables, secrets, and ConfigMap
* Full access to local service from remote services
* Do live coding/debugging in a remote Kubernetes cluster

#### Disadvantages

* Additional cost for cloud resources
* Shared environment
* Configure proxy or VPN can be complex
* UDP not supported, only TCP

#### Recommendation

Telepresence is preferred way for development. It is stable, cross-platform, works with any program and transparent to application. 

#### References

[Kubehelloworld](https://github.com/salrashid123/kubehelloworld) project authored by Sal Rashid.


#### Authors

[Opengov](https://opengov.com) Devops team.
