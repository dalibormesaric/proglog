# Distributed Services with Go

*by Travis Jeffery*

This is written in September 2023 using Go 1.21.0 with Visual Studio Code on Ubuntu 22.04 running in Windows 10 WSL2.

### Related blog post

https://developerschallenges.com/2023/10/02/book-review-—-distributed-services-with-go-by-travis-jeffery/

## Part I — Get Started

### Chapter 1. Let's Go

``` bash
curl -X POST localhost:8080 -d '{"record":{"value":"TGV0J3MgR28gIzEK"}}'
curl -X POST localhost:8080 -d '{"record":{"value":"TGV0J3MgR28gIzIK"}}'
curl -X POST localhost:8080 -d '{"record":{"value":"TGV0J3MgR28gIzMK"}}'
```

``` bash
echo TGV0J3MgR28gIzEK | base64 -d
```

``` bash
curl -X GET localhost:8080 -d '{"offset":0}'
curl -X GET localhost:8080 -d '{"offset":1}'
curl -X GET localhost:8080 -d '{"offset":2}'
```

### Chapter 2. Structure Data with Protocol Buffers

We need to install `protoc` compiler that compiles protobuf `.proto` files. Read more [here](https://grpc.io/docs/protoc-installation/).
``` bash
sudo apt install protobuf-compiler
```

We also need language-specific runtime to compile protobuf files to `.go` files, in our case for Go.
``` bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

In case of `protoc-gen-go: program not found or is not executable` add to `~/.bashrc`
``` bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
and do `source ~/.bashrc`

This is how we compile `.proto` files to `.go` files.
``` bash
protoc api/v1/*.proto --go_out=. --go_opt=paths=source_relative --proto_path=.
```

To execute `Makefile`, we need to install `make` program.
``` bash
sudo apt install make
```

### Chapter 3. Write a Log Package

Logs, somethimes know as write-ahead logs, transaction logs or commit logs, are at the heart of many different distributed systems.

Logs are use to improve data integrity.
What is log? What is segment? What is index? What is store?...

## Part II — Network

### Chapter 4. Serve Requests with gRPC

To run modified makefile

https://grpc.io/docs/languages/go/quickstart/

go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2

To resolve imports

go get -u google.golang.org/grpc



sudo apt install gcc


export CGO_ENABLED=1 // if needed
make test

### Chapter 5. Secure Your Services

1. Encrypt data in-flight to protect against man-in-the-middle attacks
2. Authenticate to identify clients
3. Authorize to determine the permissions of the identified clients

We are using mutual TLS authentication and list-based authorization to control whether a client is allowed to read from or write to (or both) to log.

CFSSL is CloudFlare's toolkit for signing, verifying and bunding TLS certificates.
``` bash
go install github.com/cloudflare/cfssl/cmd/cfssl
go install github.com/cloudflare/cfssl/cmd/cfssljson
```

Access Control List (ACL) using Casbin
``` bash
go get github.com/casbin/casbin/v2
```

### Chapter 6. Observe Your Systems

Observability? Three types of telemetry data - metrics, structured logs and traces.

cd internal/server/
go test -v -debug=true

cat /tmp/metrics-*.log
cat /tmp/traces-*.log

## Part III — Distribute

### Chapter 7. Server-to-Server Service Discovery

https://www.serf.io/

Membership

Replicator

Agent

### Chapter 8. Coordinate Your Services with Consensus

Consensus algorithms are tools used to agree on shared state even in the face of failures.

Raft is used for leader election and replication. https://github.com/hashicorp/raft
https://raft.github.io/
1. Leader Election
    - Term - tells other servers how authoritative and current this server is.
2. Log Replication

...

In order to serve both gRPC and Raft connections on the same port we need Multiplexing: https://github.com/soheilhy/cmux

### Chapter 9. Discover Server and Load Balance from the Client

Three Load-Balancing Strategies:
  - Server proxying
  - External load balancing
  - Client-side balancing

Resolver - fetches list of servers.

Picker
  - handles the RPC balancing logic based on servers discovered by resolver
  - can route RPCs based on information about RPC, client and server
  - returns ErrNoSubConnAvailable until resolver has discovered servers and updated picker's state
    - this instructs gRPC to block client's RPCs

## Part IV — Deploy

### Chapter 10. Deploy Applications with Kubernetes Locally

`kubectl` is available thanks to [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)

To build a `cli` application to serve as an entry point in `Docker`, we are using [Cobra](https://github.com/spf13/cobra).

To enable container registry on `microk8s`
```
microk8s enable registry
```

Add `"insecure-registries": ["192.168.17.102:32000"]` to Docker Desktop > Settings > Docker Engine

To push docker image to `microk8s` (replace microk8s.local with ip address)
```
docker tag github.com/dalibormesaric/proglog:0.0.1 microk8s.local:32000/github.com/dalibormesaric/proglog:0.0.1
docker push microk8s.local:32000/github.com/dalibormesaric/proglog:0.0.1
```

To intall `helm`
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```


```
POD_NAME=$(kubectl get pod --selector=app.kubernetes.io/name=nginx --template '{{index .items 0 "metadata" "name" }}')
SERVICE_IP=$(kubectl get svc --namespace default my-nginx --template "{{ .spec.clusterIP }}")
kubectl exec $POD_NAME curl $SERVICE_IP
```

curl is not available as a command in the nginx image, so I just did the following instead of the command suggested in the book:
```
kubectl exec $POD_NAME -- cat index.html
```

Kubernetes probes:
  - Liveness - if it fails, container restarts
  - Readiness - if it fails, container stops receiving traffic
  - Startup - only when this succeedes, kubernetes starts probing liveness and readiness

helm template proglog deploy/proglog

helm install proglog deploy/proglog

kubectl describe statefulset proglog



[ERROR] raft: failed to commit logs: error=EOF

https://forum.devtalk.com/t/distributed-services-with-go-unable-to-pass-readiness-liveness-checks-page-210-215/22354

### Chapter 11. Deploy Applications with Kubernetes to the Cloud

Since I already decided to use microk8s cluster in the previous chapter, I'll skip the whole GKE setup and continue working against my cluster.

Metacontroller is a Kubernetes add-on.
https://metacontroller.github.io/metacontroller/

  - DecoratorController - adds a loadbalancer service for each pod in our service's StatefulSet

To define the Metacontroller Helm chart:
``` bash
cd deploy
helm create metacontroller
rm metacontroller/templates/**/*.yaml metacontroller/templates/*.yaml metacontroller/templates/NOTES.txt metacontroller/values.yaml
curl https://raw.githubusercontent.com/metacontroller/metacontroller/v1.4.2/manifests/production/metacontroller.yaml > metacontroller/templates/metacontroller.yaml
curl https://raw.githubusercontent.com/metacontroller/metacontroller/v1.4.2/manifests/production/metacontroller-rbac.yaml > metacontroller/templates/metacontroller-rbac.yaml
curl https://raw.githubusercontent.com/metacontroller/metacontroller/v1.4.2/manifests/production/metacontroller-crds-v1.yaml > metacontroller/templates/metacontroller-crds-v1.yaml
```

To install the Metacontroller chart:
``` bash
kubectl create namespace metacontroller
helm install metacontroller deploy/metacontroller
```

``` bash
helm install proglog deploy/proglog --set service.lb=true
kubectl get services -w -A
```