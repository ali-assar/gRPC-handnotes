## Server Side

On the server side, you can return errors from your methods:

```go
import (
  "google.golang.org/grpc/codes"
  "google.golang.org/grpc/status"
)

func (s *server) SomeMethod(ctx context.Context, in *pb.SomeRequest) (*pb.SomeResponse, error) {
  if someErrorCondition {
    return nil, status.Errorf(codes.InvalidArgument, "Invalid argument: %v", someArgument)
  }
  // ...
}
```

In this example, if `someErrorCondition` is true, the method returns an `InvalidArgument` error with a description.

## Client Side

On the client side, you can check if an error is returned:

```go
import (
  "google.golang.org/grpc/codes"
  "google.golang.org/grpc/status"
)

res, err := c.SomeMethod(ctx, &pb.SomeRequest{})
if err != nil {
  if status.Code(err) == codes.InvalidArgument {
    log.Printf("Invalid argument: %v", err)
  } else {
    log.Printf("Unexpected error: %v", err)
  }
  return
}
// ...
```

In this example, the client checks if the error code is `InvalidArgument` and logs a different message accordingly.

## Error Codes

gRPC provides a list of standard error codes that you can use to represent different types of errors:

- `OK`: The operation completed successfully.
- `Cancelled`: The operation was cancelled.
- `Unknown`: An unknown error occurred.
- `InvalidArgument`: The client specified an invalid argument.
- `DeadlineExceeded`: The deadline expired before the operation could complete.
- `NotFound`: The requested entity was not found.
- `AlreadyExists`: The entity that the client attempted to create already exists.
- `PermissionDenied`: The client does not have sufficient permission.
- `ResourceExhausted`: The system resources have been exhausted.
- `FailedPrecondition`: The operation was rejected because the system is not in a state required for the operation's execution.
- `Aborted`: The operation was aborted.
- `OutOfRange`: The operation was attempted past the valid range.
- `Unimplemented`: The method has not been implemented.
- `Internal`: Internal errors.
- `Unavailable`: The service is currently unavailable.
- `DataLoss`: Unrecoverable data loss or corruption.

You can find more information about these error codes in the [gRPC documentation](https://grpc.github.io/grpc/core/md_doc_statuscodes.html).
