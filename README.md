# skir-swift-client

Runtime library for [Skir](https://skir.build)-generated Swift code.

## Installation

Add the package to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/gepheum/skir-swift-client", from: "0.1.0"),
],
targets: [
    .target(
        name: "MyApp",
        dependencies: [
            .product(name: "SkirClient", package: "skir-swift-client"),
        ]
    ),
]
```

## Usage

### Working with generated structs

The Skir code generator produces a Swift module for each `.skir` file. Import
it alongside `SkirClient` to access serializers, builders, and copy helpers:

```swift
import SkirClient
import user_skir  // generated from user.skir

// Create a value using the generated partial factory.
let user = User_skir.User.partial(id: 42, name: "Ada")

// Copy with modifications.
let renamed = user.copy(name: .set("Ada Lovelace"))

// Serialize to JSON / binary.
let json: String = User_skir.User.serializer.toJson(user)
let bytes: [UInt8] = User_skir.User.serializer.toBytes(user)

// Deserialize.
let decoded = try User_skir.User.serializer.fromJson(json)
```

### Calling a remote service

```swift
let client = try ServiceClient(serviceUrl: "https://api.example.com/myapi")
client.withDefaultHeader("Authorization", "Bearer \(token)")

let response = try await client.invokeRemote(
    MySvc_skir.getUser(),
    request: MySvc_skir.GetUserRequest.partial(id: 42)
)
```

### Implementing a service (server-side)

```swift
struct RequestMeta { let userId: String }

struct ServiceImpl {
    func getUser(_ req: MySvc_skir.GetUserRequest, meta: RequestMeta) async throws
            -> MySvc_skir.GetUserResponse {
        // ... your logic here ...
    }
}

let impl = ServiceImpl()
let service = try Service<RequestMeta>(methods: [
    .init(MySvc_skir.getUser()) { req, meta in try await impl.getUser(req, meta: meta) },
])

// Wire into your HTTP framework (e.g. Vapor):
app.on(.POST, .GET, "myapi") { req async in
    let body = req.method == .GET ? (req.url.query ?? "") : (req.body.string ?? "")
    let meta = RequestMeta(userId: req.headers.first(name: "X-User-Id") ?? "")
    return try await service.handle(body: body, meta: meta)
}
```

## License

MIT
