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
