# Swift Microservices

Packages for building gRPC microservices in Swift with Hummingbird, Vapor, PostgresNIO, and
grpc-swift. Each package is one dependency set, named for what it does, and consumed by tag.

## Packages

| Package | What it gives you | Depends on |
| --- | --- | --- |
| [swift-persistence](https://github.com/swift-microservices/swift-persistence) | `Database<Scope>`: a transaction boundary that hands a unit of work exactly the repositories it may touch | nothing |
| [swift-persistence-postgres](https://github.com/swift-microservices/swift-persistence-postgres) | `PostgresDatabase` over a PostgresNIO pool, with per-transaction settings for row-level security | postgres-nio, swift-service-context |
| [swift-authentication](https://github.com/swift-microservices/swift-authentication) | `Authenticator`, `Principal`, `PrincipalKey`: who is calling, proved by a credential and carried with the call | swift-service-context |
| [swift-authentication-jwt](https://github.com/swift-microservices/swift-authentication-jwt) | `JWTIssuer`, `JWTAuthenticator`: a bearer token as a JSON Web Token | jwt-kit |
| [swift-authentication-x509](https://github.com/swift-microservices/swift-authentication-x509) | `SPIFFEAuthenticator`: a peer by the SPIFFE name in its certificate | swift-certificates |
| [swift-authentication-grpc](https://github.com/swift-microservices/swift-authentication-grpc) | interceptors that bind a bearer token or the peer certificate, and present the token onward | grpc-swift-2, grpc-swift-nio-transport |
| [swift-authentication-hummingbird](https://github.com/swift-microservices/swift-authentication-hummingbird) | the bearer middleware for Hummingbird | hummingbird-auth |
| [swift-authentication-vapor](https://github.com/swift-microservices/swift-authentication-vapor) | the bearer middleware for Vapor 4 | vapor |

## How they fit

**Persistence** is one protocol. A use case declares the scope it needs, a set of repositories,
and takes `any Database` over it; the driver hands the scope to the work inside a transaction.
The Postgres driver applies `PostgresSettings` to each transaction, read from the task's
`ServiceContext`, which is how a caller reaches row-level security policies.

**Authentication** is one shape with two proofs and three transports. An `Authenticator` turns a
credential into an identity, declines with `nil`, or refuses by throwing. jwt and x509 are the
proofs. grpc, hummingbird, and vapor read the credential off the call and bind the result as a
`Principal` in the `ServiceContext` for the length of the call. Nothing is named by who
presented a credential: a token proves a payload, and whether that is a person or a process is a
claim the application reads.

Everything meets in `ServiceContext`, the task-local the server ecosystem already shares, so a
caller's identity and its database settings flow together from the transport to the repository.

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
`semver/none`. Each repository's `AGENTS.md` states what belongs in it and what does not.

All packages are MIT licensed.
