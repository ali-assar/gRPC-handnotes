# Introduction to Protocol Buffers

In this section, we will cover the basics of Protocol Buffers (protobufs), a language-agnostic binary data format developed by Google.

## What are Protocol Buffers?

Protocol Buffers (protobuf) are a flexible, efficient, and automated mechanism for serializing structured data. They are more streamlined and faster than XML and offer robust support for schemas.

## Why use Protocol Buffers?

- **Language Agnostic**: Protobuf messages can be generated and read by a variety of programming languages.
- **Efficiency**: Protobuf messages are much smaller and faster to serialize/deserialize than XML and JSON messages.
- **Backwards Compatibility**: Protobuf allows you to add new fields to your message formats without breaking old programs.

## How to Define a Message Type

A protobuf message is defined in a `.proto` file. Here's an example:

```protobuf
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

In this example, `SearchRequest` is the name of the message and `query`, `page_number`, and `result_per_page` are its fields. Each field has a name, a type, and a unique number.

## Compiling Your Protocol Buffers

After defining your message, you can compile it using the Protocol Buffer compiler (`protoc`). This will generate data access classes in your chosen language (in our case, Go).

```bash
protoc --go_out=. *.proto
```

This command will generate a `.pb.go` file with the same name as your `.proto` file. This file contains the Go code for your message.

## Defining RPC Methods

In gRPC, a service is defined in the `.proto` file. It specifies the methods that can be called remotely with their parameters and return types. Here's an example:

```protobuf
syntax = "proto3";

package tutorial;

service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

In this example, `SearchService` is the service and `Search` is the RPC method. `SearchRequest` is the request parameter and `SearchResponse` is the return type.

## Streaming

gRPC supports streaming semantics, where either the client or the server (or both) send a stream of messages on a single RPC call. The three types of streaming are:

- **Server-side streaming**: The client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages.

- **Client-side streaming**: The client writes a sequence of messages and sends them to the server. Once the client has finished writing the messages, it waits for the server to read them all and return its response.

- **Bidirectional streaming**: Both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like.

Here's an example of how to define a server-side streaming RPC:

```protobuf
syntax = "proto3";

package tutorial;

service SearchService {
  rpc Search (SearchRequest) returns (stream SearchResponse);
}
```

In this example, the `Search` RPC method returns a stream of `SearchResponse` messages.

## Go Package Option

When generating Go code from your `.proto` files, you can specify the package name with the `go_package` option:

```protobuf
syntax = "proto3";

option go_package = "github.com/yourusername/yourrepo";

package tutorial;

// rest of your .proto file
```

In this example, the generated Go code will be part of the `github.com/yourusername/yourrepo` package.


