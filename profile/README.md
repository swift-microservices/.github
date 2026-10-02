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
| [swift-authentication-grpc](https://github.com/swift-microservices/swift-authentication-grpc) | user bearer authentication and propagation on user RPC descriptors | grpc-swift-2 |
| [swift-authentication-hummingbird](https://github.com/swift-microservices/swift-authentication-hummingbird) | the bearer middleware for Hummingbird | hummingbird-auth |
| [swift-authentication-vapor](https://github.com/swift-microservices/swift-authentication-vapor) | the bearer middleware for Vapor 4 | vapor |
| [swift-openapi-token-authentication](https://github.com/swift-microservices/swift-openapi-token-authentication) | `AuthenticationSession` and `AuthenticationMiddleware`: shared token authentication, refresh, and a single retry for rejected requests | swift-openapi-runtime, swift-http-types |

## How they fit

**Persistence** is one protocol. A use case declares the scope it needs, a set of repositories,
and takes `any Database` over it; the driver hands the scope to the work inside a transaction.
The Postgres driver applies `PostgresSettings` to each transaction, read from the task's
`ServiceContext`, which is how a caller reaches row-level security policies.

**Security** uses mTLS for service-to-service connections, including gateway upstreams,
worker clients, and Temporal. Backend listeners require client certificates from explicit CA
roots; clients verify server certificates and destination hostnames. Keep listeners private
and gateway routes limited to their intended public and user operations.

JWTs authenticate users making these calls. `Authenticator.authenticate(_:)` returns a concrete
identity or throws, and the transport binds a `Principal` in `ServiceContext`. Each receiving
service verifies the original JWT. Apply bearer authentication and propagation to user RPC
descriptors, and authorize users in the owning use case. User-scoped database settings preserve
tenant isolation on this pathway.

Use separate public, user, and internal protobuf descriptors. Public operations validate their
required credentials or proofs. Every peer admitted by an internal listener's CA trust can call
its internal RPCs; those operations accept business input and enforce domain invariants.
Workers call their own Core operations locally and other services through internal RPCs.

**Certificate lifecycle** combines deployment-managed renewal with a primed
`TimedCertificateReloader` from `NIOCertificateReloading` running alongside transports in `ServiceGroup`. New handshakes use
refreshed material; connection age and graceful draining bound existing sessions. Monitor
renewal and expiry, and rotate trust roots with overlap and a tested transport rebuild or
restart. See the [gRPC certificate renewal guide](https://github.com/swift-microservices/swift-authentication-grpc/blob/main/Sources/AuthenticationGRPC/Documentation.docc/Articles/MutualTLSAndCertificateRenewal.md).

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

## Foundation

We follow [Swift Foundation's direction](https://forums.swift.org/t/swift-foundation-now-available/73530)
toward modern Swift APIs and focused modules. Use the standard library when it suffices, and
import and link `FoundationEssentials` when Foundation types are needed. Never introduce legacy
Foundation APIs in new or changed code: use modern format styles and parse strategies instead
of formatter-based date decoding or printf-style formatting. ISO 8601 styles are available in
Essentials; localized formatting may require `FoundationInternationalization` and ICU.

Check the latest compatible dependency releases, APIs, and trait defaults before choosing
versions. Some libraries retain a default-enabled `FullFoundation` trait for compatibility
with users of legacy APIs. Hummingbird 2.27.0 and swift-openapi-runtime 1.12.1 are examples:
use `traits: []` when no optional features are needed, or explicitly select only needed traits
such as Hummingbird's `ConfigurationSupport`. Defaults and trait names vary, and another
dependency can enable the trait again, so verify the complete resolved graph.

An application may still link full Foundation through widely used server libraries. At our
2026-09-27 audit, Vapor 4.122.2 and PostgresNIO 1.33.1 still required it. Recheck upstream
releases rather than treating that as permanent; keep our code on modern Essentials APIs
even while that requirement remains. Libraries that can avoid full Foundation enforce it
with a separate Linux linking CI check; static SDK success alone does not prove its absence.
See the skills' [dependency and trait guidance](https://github.com/swift-microservices/skills/blob/main/skills/building-swift-services/references/service-package.md#foundation-dependencies-and-traits)
and [API policy](https://github.com/swift-microservices/skills/blob/main/skills/building-swift-services/references/swift-style.md#foundation-and-modern-apis).

## Contributing

Pull requests are welcome on any package. Keep a change focused, prove new behaviour with a
test, and label the pull request `⚠️ semver/major`, `🆕 semver/minor`, `🔨 semver/patch`, or
`semver/none`. See each repository for package-specific guidance.

See individual package repositories for licensing information.
