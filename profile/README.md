# Swift Microservices

Packages for running Swift services and calling APIs: microservices or a monolith, over gRPC,
HTTP, or both.
They sit on the server ecosystem as it is, grpc-swift 2, Hummingbird, Vapor, PostgresNIO,
swift-service-context, and add the pieces every service needs and none of those frameworks
provides: a transaction boundary that hands a unit of work its repositories, and a caller
proved by a credential and carried with the call. For generated OpenAPI clients, they also
provide shared token authentication and automatic refresh.

Each package is one dependency set, named for what it does, and consumed by tag. A service links
only what it uses: a gRPC service links the interceptors and never Hummingbird; an HTTP monolith
links a middleware and never grpc-swift; a domain target links the two core packages and no
framework at all.

## Packages

| Package | What it gives you | Depends on |
| --- | --- | --- |
| [swift-persistence](https://github.com/swift-microservices/swift-persistence) | `Database<Scope>`: a transaction boundary that hands a unit of work exactly the repositories it may touch | nothing |
| [swift-persistence-postgres](https://github.com/swift-microservices/swift-persistence-postgres) | `PostgresDatabase` over a PostgresNIO pool, with per-transaction settings for row-level security | postgres-nio, swift-service-context |
| [swift-authentication](https://github.com/swift-microservices/swift-authentication) | `Authenticator`, `Principal`, `PrincipalKey`: who is calling, proved by a credential and carried with the call | swift-service-context |
| [swift-authentication-jwt](https://github.com/swift-microservices/swift-authentication-jwt) | `JWTIssuer`, `JWTAuthenticator`: a bearer token as a JSON Web Token | jwt-kit |
| [swift-authentication-x509](https://github.com/swift-microservices/swift-authentication-x509) | `SPIFFEAuthenticator`: a peer by the SPIFFE name in its certificate | swift-certificates |
| [swift-authentication-grpc](https://github.com/swift-microservices/swift-authentication-grpc) | interceptors that bind a bearer token or the peer certificate, and present the token onward | grpc-swift 2 (grpc-swift-2, grpc-swift-nio-transport) |
| [swift-authentication-hummingbird](https://github.com/swift-microservices/swift-authentication-hummingbird) | the bearer middleware for Hummingbird | hummingbird-auth |
| [swift-authentication-vapor](https://github.com/swift-microservices/swift-authentication-vapor) | the bearer middleware for Vapor 4 | vapor |
| [swift-openapi-token-authentication](https://github.com/swift-microservices/swift-openapi-token-authentication) | `AuthenticationSession` and `AuthenticationMiddleware`: shared token authentication, refresh, and a single retry for rejected requests | swift-openapi-runtime, swift-http-types |

## How they fit

**Persistence** is one protocol. A use case declares the scope it needs, a set of repositories,
and takes `any Database` over it; the driver hands the scope to the work inside a transaction.
The Postgres driver applies `PostgresSettings` to each transaction, read from the task's
`ServiceContext`, which is how a caller reaches row-level security policies.

**Authentication** is one shape with two proofs and three transports. An `Authenticator` turns a
credential into an identity, declines with `nil`, or refuses by throwing. jwt and x509 are the
proofs. grpc, hummingbird, and vapor read the credential off the call and bind the result as a
`Principal` in the `ServiceContext` for the length of the call. A service that speaks both
gRPC and HTTP uses two of them with the same authenticator, and the same principal reaches its
handlers either way. Nothing is named by who
presented a credential: a token proves a payload, and whether that is a person or a process is a
claim the application reads.

The server-side authentication and persistence packages meet in `ServiceContext`, the task-local
the server ecosystem already shares, so a caller's identity and its database settings flow
together from the transport to the repository.

**OpenAPI token authentication** handles the client side independently. An `AuthenticationSession`
actor stores credentials and shares in-flight login and refresh work. `AuthenticationMiddleware`
adds bearer tokens and refreshes before retrying an HTTP 401 response once, when the request body
is replayable. Clients provide their API's login and refresh implementation and can observe the
session through `states()`.

## Conventions

- Swift 6.3, strict concurrency, Linux first. Every package builds under the static Linux SDK.
- One package per dependency set. A domain target links `Persistence` and `Authentication`
  without pulling a driver, a token library, or a framework in behind it.
- Consumers pin by tag. Releases are GitHub Releases, created from the semantic-version label
  on each merged pull request.
- Every public declaration has a doc comment, every package has one design article, and every
  behaviour a package promises has a test.

## Contributing

Pull requests are welcome on any package. Keep a change focused, prove new behaviour with a
test, and label the pull request `⚠️ semver/major`, `🆕 semver/minor`, `🔨 semver/patch`, or
`semver/none`. See each repository for package-specific guidance.

See individual package repositories for licensing information.
