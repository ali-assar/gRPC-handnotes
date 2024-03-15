# Unary gRPC in Go

Unary RPCs are the simplest type of RPC where the client sends a single request and gets a single response back. Here's how you can implement a unary gRPC service in Go:

## Step 1: Define the Service

First, you need to define the service in your `.proto` file. Here's an example:

```protobuf
syntax = "proto3";

package tutorial;

message StringRequest {
  string data = 1;
}

message StringResponse {
  string data = 1;
}

service StringService {
  rpc ToUpper (StringRequest) returns (StringResponse);
}
```

In this example, the `StringService` has a unary RPC method `ToUpper` that takes a `StringRequest` and returns a `StringResponse`.

## Step 2: Generate Go Code

After defining your service, you can generate the Go code using the `protoc` compiler:

```bash
protoc --go_out=plugins=grpc:. *.proto
```

This command generates a `.pb.go` file with the same name as your `.proto` file. This file contains the Go code for your service and messages.

## Step 3: Implement the Server

Next, you need to implement the server. Here's an example:

```go
package main

import (
  "context"
  "log"
  "net"

  pb "path/to/your/package"

  "google.golang.org/grpc"
)

type server struct{}

func (s *server) ToUpper(ctx context.Context, in *pb.StringRequest) (*pb.StringResponse, error) {
  return &pb.StringResponse{Data: strings.ToUpper(in.Data)}, nil
}

func main() {
  lis, err := net.Listen("tcp", ":50051")
  if err != nil {
    log.Fatalf("failed to listen: %v", err)
  }
  s := grpc.NewServer()
  pb.RegisterStringServiceServer(s, &server{})
  if err := s.Serve(lis); err != nil {
    log.Fatalf("failed to serve: %v", err)
  }
}
```

In this example, the `ToUpper` method converts the input string to upper case.

## Step 4: Implement the Client

Finally, you need to implement the client. Here's an example:

```go
package main

import (
  "context"
  "log"
  "os"
  "time"

  pb "path/to/your/package"

  "google.golang.org/grpc"
)

func main() {
  conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure(), grpc.WithBlock())
  if err != nil {
    log.Fatalf("did not connect: %v", err)
  }
  defer conn.Close()
  c := pb.NewStringServiceClient(conn)

  ctx, cancel := context.WithTimeout(context.Background(), time.Second)
  defer cancel()
  r, err := c.ToUpper(ctx, &pb.StringRequest{Data: "hello"})
  if err != nil {
    log.Fatalf("could not greet: %v", err)
  }
  log.Printf("Greeting: %s", r.GetData())
}
```

In this example, the client sends a "hello" string to the server and prints the response.

That's it! You have successfully implemented a unary gRPC service in Go. In the next sections, we will cover more advanced gRPC concepts.
