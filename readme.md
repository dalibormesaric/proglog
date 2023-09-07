# Distributed Services with Go

## Chapter 1. Let's Go

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

## Chapter 2. Define Your Domain Types as Protocol Buffers

https://grpc.io/docs/protoc-installation/

``` bash
sudo apt install protobuf-compiler
```

``` bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

In case of `protoc-gen-go: program not found or is not executable` add to `~/.bashrc`
``` bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
and do `. ~/.bashrc`

To generate
``` bash
protoc api/v1/*.proto --go_out=. --go_opt=paths=source_relative --proto_path=.
```