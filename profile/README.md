# Swift Microservices

Packages for running Swift services and calling APIs: microservices or a monolith, over gRPC,
HTTP, or both.

They build on grpc-swift 2, Hummingbird, Vapor, PostgresNIO, and swift-operation-context to provide
transaction boundaries, credential authentication, and caller identity carried with each call.
For API clients, they also provide token authentication with automatic refresh, including a
middleware for generated OpenAPI clients.

Each package has a focused dependency set. A service links only what it uses: gRPC interceptors
for a gRPC service, HTTP middleware for an HTTP application, and the core persistence and
authentication abstractions for domain code.

## Packages

| Package | What it offers | Built on |
| --- | --- | --- |
| [swift-operation-context](https://github.com/swift-microservices/swift-operation-context) | Typed, task-scoped propagation of identity, credentials, and other operation values | Swift standard library |
| [swift-persistence](https://github.com/swift-microservices/swift-persistence) | Transaction boundaries that give each unit of work the repositories it needs | Swift standard library |
| [swift-persistence-postgres](https://github.com/swift-microservices/swift-persistence-postgres) | PostgreSQL transactions with per-transaction settings for row-level security | postgres-nio, swift-operation-context |
| [swift-authentication](https://github.com/swift-microservices/swift-authentication) | Credential authentication and caller identity carried with the call | swift-operation-context |
| [swift-authentication-jwt](https://github.com/swift-microservices/swift-authentication-jwt) | JWT issuance and verification | jwt-kit |
| [swift-authentication-grpc](https://github.com/swift-microservices/swift-authentication-grpc) | User bearer authentication and token propagation for gRPC | grpc-swift-2 |
| [swift-authentication-hummingbird](https://github.com/swift-microservices/swift-authentication-hummingbird) | Authentication middleware for Hummingbird | hummingbird-auth |
| [swift-authentication-vapor](https://github.com/swift-microservices/swift-authentication-vapor) | Authentication middleware for Vapor | vapor |
| [swift-token-authentication](https://github.com/swift-microservices/swift-token-authentication) | Shared token sessions, automatic refresh, and retry for API clients, with an OpenAPI client middleware | Swift standard library; swift-openapi-runtime and swift-http-types with its `OpenAPIRuntime` trait |

## How they fit

**Operation context** provides a shared container for library-owned, type-safe values. Scoped
bindings propagate through Swift structured concurrency, allowing child tasks to inherit an
operation's context. Libraries define their own keys and remain responsible for authentication,
authorization, and applying database settings.

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

**API clients** share authentication state and in-flight sign-in and refresh work across
requests. Applications supply their API's sign-in and, optionally, refresh calls. For generated
OpenAPI clients, a middleware adds bearer tokens and can refresh and retry a rejected request once
when its body can be replayed.

## Building a backend

[swift-microservices-architect](https://github.com/swift-microservices/swift-microservices-architect-skill)
is an agent skill that teaches coding agents to design, build, and review Swift server backends on
these packages: gRPC services behind a Hummingbird gateway, one Swift package per service whose
core imports no driver or transport, Postgres row-level security, mTLS between processes, and
durable Temporal workflows. Ask for a plan, a service, an endpoint, or a review, and the agent
answers with the same architecture, names, and code style every time. Its README covers
installation.

## Building an app

[swiftui-architect](https://github.com/swift-microservices/swiftui-architect-skill) is an agent
skill that teaches coding agents to design, build, and review modular SwiftUI apps: one Swift
package per feature, split into domain, UI, and live-service targets whose boundaries the compiler
enforces, `@Observable` view models under Swift 6 strict concurrency, and native `NavigationStack`
navigation between features that never import each other. Ask for a feature, a screen, a link
between two features, a move into packages, or a review, and the agent answers with the same
structure, names, and shapes every time. Its README covers installation.

## CI and delivery

Every repository's CI calls [ci](https://github.com/swift-microservices/ci), pinned by commit, and
nothing else. Its workflows run [swiftlang/github-workflows](https://github.com/swiftlang/github-workflows)
for soundness and tests, and add what those do not: a PostgreSQL service or private dependencies
for the tests, a check that products link neither Foundation's full library nor ICU where their
dependencies allow, and releases versioned from pull request labels. For a service, ci builds one
image per commit and deploys it to Dokploy, to staging on every push to `main` and to production by
hand for a commit staging ran.

## Conventions

- Modern Swift, strict concurrency, and Linux support, including the static Linux SDK.
- Focused packages keep database drivers, token libraries, and transport frameworks out of domain code.
- Every pull request carries one semver label, and releases follow from them, so consumers depend
  on versions.
- CI tests every package on Swift 6.3, 6.4, and the nightly snapshots, on x86_64 and aarch64, with
  every workflow and action pinned by commit.
- Public APIs have documentation, and tests cover the behavior each package promises.

## Contributing

Pull requests are welcome on any package. Keep changes focused and cover new behavior with tests.
See each repository for usage, contribution, and licensing details.
