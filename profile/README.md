# Swift Microservices

Packages for running Swift services and calling APIs: microservices or a monolith, over gRPC,
HTTP, or both.

They build on grpc-swift 2, Hummingbird, Vapor, PostgresNIO, and swift-service-context to provide
transaction boundaries, credential authentication, and caller identity carried with each call.
For generated OpenAPI clients, they also provide shared token authentication and automatic refresh.

Each package has a focused dependency set. A service links only what it uses: gRPC interceptors
for a gRPC service, HTTP middleware for an HTTP application, and the core persistence and
authentication abstractions for domain code.

## Packages

| Package | What it offers | Built on |
| --- | --- | --- |
| [swift-persistence](https://github.com/swift-microservices/swift-persistence) | Transaction boundaries that give each unit of work the repositories it needs | Swift standard library |
| [swift-persistence-postgres](https://github.com/swift-microservices/swift-persistence-postgres) | PostgreSQL transactions with per-transaction settings for row-level security | postgres-nio, swift-service-context |
| [swift-authentication](https://github.com/swift-microservices/swift-authentication) | Credential authentication and caller identity carried with the call | swift-service-context |
| [swift-authentication-jwt](https://github.com/swift-microservices/swift-authentication-jwt) | JWT issuance and verification | jwt-kit |
| [swift-authentication-grpc](https://github.com/swift-microservices/swift-authentication-grpc) | User bearer authentication and token propagation for gRPC | grpc-swift-2 |
| [swift-authentication-hummingbird](https://github.com/swift-microservices/swift-authentication-hummingbird) | Authentication middleware for Hummingbird | hummingbird-auth |
| [swift-authentication-vapor](https://github.com/swift-microservices/swift-authentication-vapor) | Authentication middleware for Vapor | vapor |
| [swift-openapi-token-authentication](https://github.com/swift-microservices/swift-openapi-token-authentication) | Shared token sessions, automatic refresh, and retry for OpenAPI clients | swift-openapi-runtime, swift-http-types |

## How they fit

**Persistence** keeps use cases independent of the database driver. A use case declares the
repository scope it needs, and the driver supplies those repositories inside a transaction.
The PostgreSQL integration carries caller-specific settings into the transaction so database
row-level security policies can enforce tenant isolation.

**Authentication** proves who is calling and carries that identity through the request.
JWT support issues and verifies user tokens; transport integrations authenticate incoming
requests and propagate the original token to downstream user operations. Each receiving service
verifies the token, while the owning use case makes authorization decisions.

**Service security** uses mTLS for service-to-service connections. Backend services verify their
peers with trusted certificates, and gateways expose only the intended public and user operations.
Separate public, user, and internal RPCs keep those boundaries clear: public operations validate
their required credentials or proofs, user operations authenticate the caller, and internal
operations rely on trusted service peers while enforcing domain invariants.

**OpenAPI clients** share authentication state and in-flight login and refresh work across
requests. Middleware adds bearer tokens and can refresh and retry a rejected request once when
its body can be replayed. Applications supply their API's login and refresh implementation.

## Conventions

- Modern Swift, strict concurrency, and Linux support, including the static Linux SDK.
- Focused packages keep database drivers, token libraries, and transport frameworks out of domain code.
- Tagged releases give consumers versioned dependencies.
- Public APIs have documentation, and tests cover the behavior each package promises.

## Skills

[swift-microservices skills](https://github.com/swift-microservices/skills) provide guidance for
architecture, implementation, testing, CI, and deployment of Swift server projects.

## Contributing

Pull requests are welcome on any package. Keep changes focused and cover new behavior with tests.
See each repository for usage, contribution, and licensing details.
