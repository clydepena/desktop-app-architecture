# Assets

Networking uses far fewer content assets than rendering or audio. What it uses is mostly **configuration and
schemas**: files that describe protocols, replication rules and endpoints, created by programmers and designers and
loaded like other assets ([Resources](../07-resources.md)).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Platform and transport** | **endpoint lists** | servers, regions, relay and rendezvous addresses |
| | **transport settings** | timeouts, channel setup, maximum packet size, congestion settings |
| **Messages** | **protocol schemas** | message definitions in an interface definition language, or in the data description |
| | **protocol version tables** | which versions are compatible with which |
| **Sessions** | **session and lobby rules** | maximum players, roles, reconnect grace period, heartbeat and timeout values |
| | **matchmaking rules** | skill ranges, regions, how long to search before widening |
| **Replication and RPC** | **replication settings per type** | which fields replicate, update rates, priorities, relevancy rules, quantization ranges |
| | **RPC lists** | which remote calls exist, who may call them, on which channel |
| **Prediction and timing** | **simulation settings** | tick rate, interpolation delay, prediction and correction tuning |
| **Request/response services** | **service descriptions** | endpoints, API versions, retry policies, cache rules |
| **Tools** | **network simulation profiles** | "mobile 3G", "bad Wi-Fi", "intercontinental": latency, jitter, loss, bandwidth |
| | **recorded sessions** | captured message streams, for replaying bugs |

## Source and runtime forms

- **Schemas** are written in a readable definition language and **compiled** into encoders and decoders (generated
  code or compact tables) as part of the build. Client and server must be built from the same schemas, or from
  compatible versions.
- **Replication and session settings** are editable descriptions (JSON, an editor panel) packed into compact data
  for the build.
- **Endpoint lists** often differ per environment (development, testing, production): the same asset structure,
  different files chosen by the build or a setting.

## Not assets: secrets

Certificates, private keys, API keys and passwords are **secrets**, not assets:

- they are provisioned per environment and stored in secure storage or a secrets manager
- they are never committed to the project, never packed into the client, never logged
- a client that needs to prove who it is gets a **token** at runtime ([Security](08-security.md))

## Assets vs settings vs runtime state

- **Assets** describe the protocol and the rules: schemas, replication settings, endpoints. Saved with the project,
  the same for everyone.
- **Settings** are the user's choices: a preferred region, a bandwidth limit, a custom server address
  ([Settings](../04-settings.md)).
- **Runtime state** is what is happening now: connections, sessions, replicated objects, pending requests. Never
  saved as an asset; a game save stores domain state, not network state.
