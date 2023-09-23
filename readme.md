# Distributed Services with Go

*by Travis Jeffery*

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

Access Control List using Casbin
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