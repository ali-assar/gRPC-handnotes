# Deadlines 

Deadlines allow gRPC clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with the error `DEADLINE_EXCEEDED`. This is an important mechanism to ensure that services don't hang indefinitely and resources can be freed appropriately. Here's how you can use deadlines in gRPC with Go:

## Server Side

On the server side, you can check if a context is done to see if the client has cancelled the request or if the deadline has been exceeded:

```go
func (s *server) SomeMethod(ctx context.Context, in *pb.SomeRequest) (*pb.SomeResponse, error) {
  for {
    select {
    case <-ctx.Done():
      if ctx.Err() == context.Canceled {
        // The client cancelled the request.
        return nil, status.Error(codes.Canceled, "The client cancelled the request.")
      }
      // The deadline was exceeded.
      return nil, status.Error(codes.DeadlineExceeded, "The deadline was exceeded.")
    default:
      // Continue processing the request.
    }
    // ...
  }
}
```

In this example, the server checks if the context is done in a loop and returns an error if the client cancelled the request or if the deadline was exceeded.

## Client Side

On the client side, you can set a deadline on the context when making a request:

```go
ctx, cancel := context.WithTimeout(context.Background(), time.Second)
defer cancel()

res, err := c.SomeMethod(ctx, &pb.SomeRequest{})
if err != nil {
  if status.Code(err) == codes.DeadlineExceeded {
    log.Printf("Deadline exceeded: %v", err)
  } else {
    log.Printf("Unexpected error: %v", err)
  }
  return
}
// ...
```

In this example, the client sets a deadline of one second on the context. If the server doesn't respond within one second, the request is cancelled and the client receives a `DEADLINE_EXCEEDED` error.
