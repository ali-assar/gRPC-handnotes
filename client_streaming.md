# Client Streaming gRPC in Go

Client streaming RPCs are a type of RPC where the client writes a sequence of messages and sends them to the server. Once the client has finished writing the messages, it waits for the server to read them and return its response. Here's how you can implement a client streaming gRPC service in Go:

<div style="text-align:center">
    <img src="./images/client-stream.png" alt="client streaming">
</div>

## Step 1: Define the Service

First, you need to define the service in your `.proto` file. Here's an example:

```protobuf
syntax = "proto3";

package tutorial;

message NumberRequest {
  int32 num = 1;
}

message NumberResponse {
  int32 sum = 1;
}

service NumberService {
  rpc SumNumbers (stream NumberRequest) returns (NumberResponse);
}
```

In this example, the `NumberService` has a client streaming RPC method `SumNumbers` that takes a stream of `NumberRequest` messages and returns a `NumberResponse`.

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
  "io"
  "log"
  "net"

  pb "path/to/your/package"

  "google.golang.org/grpc"
)

type server struct{}

func (s *server) SumNumbers(stream pb.NumberService_SumNumbersServer) error {
  sum := 0
  for {
    req, err := stream.Recv()
    if err == io.EOF {
      return stream.SendAndClose(&pb.NumberResponse{Sum: int32(sum)})
    }
    if err != nil {
      return err
    }
    sum += int(req.Num)
  }
}

func main() {
  lis, err := net.Listen("tcp", ":50051")
  if err != nil {
    log.Fatalf("failed to listen: %v", err)
  }
  s := grpc.NewServer()
  pb.RegisterNumberServiceServer(s, &server{})
  if err := s.Serve(lis); err != nil {
    log.Fatalf("failed to serve: %v", err)
  }
}
```

In this example, the `SumNumbers` method receives a stream of numbers from the client and sends back their sum.

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
  c := pb.NewNumberServiceClient(conn)

  ctx, cancel := context.WithTimeout(context.Background(), time.Second)
  defer cancel()
  stream, err := c.SumNumbers(ctx)
  if err != nil {
    log.Fatalf("could not sum numbers: %v", err)
  }
  for i := 1; i <= 10; i++ {
    if err := stream.Send(&pb.NumberRequest{Num: int32(i)}); err != nil {
      log.Fatalf("could not send: %v", err)
    }
  }
  res, err := stream.CloseAndRecv()
  if err != nil {
    log.Fatalf("could not receive: %v", err)
  }
  log.Printf("Sum: %d", res.GetSum())
}
```

In this example, the client sends a stream of numbers from 1 to 10 to the server and prints the sum.
