# gRUSTpcWEB

A full-stack demo showcasing a **Rust**-based gRPC service with **gRPC-Web** proxying and a **Vue.js** frontend client. The pipeline includes:

- **Rust gRPC Service** (`src/`): Implements a `ProductService` gRPC API using `tonic`.
- **Envoy Proxy** (`envoy.yaml`): Exposes gRPC-Web endpoints for browser consumption.
- **Vue.js SPA** (`vue/`): A web application consuming the gRPC-Web API via `grpc-web` client.
- **Protocol Buffers** (`proto/`): Shared service definitions for gRPC and client codegen.
- **Docker**: Multi-stage `Dockerfile` builds the Rust server and Vue frontend, orchestrates with Envoy for gRPC-Web.
- **build.rs**: Automates Protobuf codegen for Rust.

---

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [Architecture](#architecture)
4. [Prerequisites](#prerequisites)
5. [Project Structure](#project-structure)
6. [Installation & Build](#installation--build)
7. [Configuration](#configuration)
8. [Running the Application](#running-the-application)
9. [API Reference](#api-reference)
10. [Envoy Proxy](#envoy-proxy)
11. [Docker Setup](#docker-setup)
12. [Contributing](#contributing)
13. [License](#license)

---

## Overview

`gRUSTpcWEB` demonstrates end-to-end gRPC-Web integration:

1. **Backend**: Rust service using [`tonic`](https://docs.rs/tonic/) to serve gRPC RPCs.
2. **Proxy**: Envoy handles gRPC-Web translation for browser clients.
3. **Frontend**: Vue.js SPA uses [`grpc-web`](https://github.com/grpc/grpc-web) to call RPCs from the browser.

## Features

- 🔨 **Rust gRPC**: High-performance, type-safe APIs in Rust.
- 🌐 **gRPC-Web**: Browser-friendly RPC via Envoy.
- 📦 **Vue.js SPA**: Modern UI with live RPC data.
- 📜 **Protobuf Schemas**: Single source of truth for service contracts.
- 🐳 **Dockerized**: Containerized backend, proxy, and frontend.
- ⚙️ **Automated Codegen**: `build.rs` for Rust stubs.

## Architecture

```text
+-----------+    gRPC    +---------------+    gRPC-Web    +------------+
|           | ----------> | Rust Server  | -------------> | Vue Client |
| Frontend  |             | (tonic gRPC) |                | (grpc-web) |
| (Browser) | <----------> | Envoy Proxy  | <------------> |            |
+-----------+    HTTP    +---------------+   HTTP/2 + Web    +------------+
```

- The **Vue** app communicates over HTTP/1.1 to Envoy.
- **Envoy** translates gRPC-Web to gRPC HTTP/2 for the Rust server.

## Prerequisites

- **Rust** (stable toolchain)
- **Node.js** (>= 14) and npm/pnpm
- **Protobuf** compiler (`protoc`)
- **Envoy** (if running locally)
- **Docker** & **Docker Compose** (optional)

## Project Structure

```
├── proto/           # .proto files defining ProductService
├── src/             # Rust gRPC server (Cargo.toml, build.rs)
├── vue/             # Vue.js frontend
│   ├── package.json
│   └── src/         # Vue components and gRPC client setup
├── envoy.yaml       # Envoy config for gRPC-Web proxy
├── Dockerfile       # Multi-stage build for server & client
├── .gitignore
└── README.md        # This documentation
```

## Installation & Build

### 1. Clone the repository

```bash
git clone https://github.com/nicktretyakov/gRUSTpcWEB.git
cd gRUSTpcWEB
```

### 2. Rust Codegen & Build

```bash
# Generate Rust gRPC code from proto
cargo build --release
```

This runs `build.rs`, which invokes `protoc` to generate Rust server stubs in `src/generated/`.

### 3. Frontend Install

```bash
cd vue
npm install        # or pnpm install
npm run build      # builds static assets
cd ..
```

## Configuration

- **Envoy**: Adjust ports and upstream clusters in `envoy.yaml`.
- **Server**: Configure gRPC listen address in `src/main.rs` or via env vars.
- **Vue**: Update RPC endpoint in `vue/src/grpc-client.ts` if needed.

## Running the Application

### Locally (without Docker)

1. **Start Envoy**
   ```bash
   envoy -c envoy.yaml
   ```
2. **Run Rust Server**
   ```bash
   cargo run --release
   ```
3. **Serve Vue App**
   ```bash
   cd vue
   npm run serve
   ```
4. Visit `http://localhost:8080` (or configured port) to access the SPA.

### With Docker Compose

```bash
docker-compose up --build
```

This spins up:
- **grpc-server**: Rust service
- **envoy**: gRPC-Web proxy
- **vue-frontend**: Static site served via nginx

## API Reference

### gRPC Service: `ProductService`

```proto
service ProductService {
  rpc ListProducts(Empty) returns (stream Product);
  rpc GetProduct(ProductRequest) returns (Product);
  rpc CreateProduct(Product) returns (Product);
  rpc UpdateProduct(Product) returns (Product);
  rpc DeleteProduct(ProductRequest) returns (Empty);
}
```

- **ListProducts**: Server streaming products
- **CRUD**: Standard create, read, update, delete

### Frontend Calls (grpc-web)

```js
import { ProductServiceClient } from '../generated/pc_grpc_web_pb';
const client = new ProductServiceClient('http://localhost:8080', null, null);

client.listProducts(new Empty(), {})
  .on('data', product => { console.log(product.toObject()); });
```

## Envoy Proxy

Key settings in `envoy.yaml`:

- **Listeners**: Accepts HTTP/1.1 on port 8080
- **Filters**: `grpc_web`, `router`
- **Clusters**: Points to `grpc-server:50051`

## Docker Setup

Use the provided `Dockerfile` and `docker-compose.yml`:

```yaml
env:
  - ENV=production
services:
  grpc-server:
    build:
      context: .
      target: grpc-server
  envoy:
    image: envoyproxy/envoy:v1.21.1
    volumes:
      - ./envoy.yaml:/etc/envoy/envoy.yaml
  vue-frontend:
    build:
      context: .
      target: vue-frontend
```

## Contributing

Contributions welcome!

1. Fork the repo  
2. Create branch: `feature/your-feature`  
3. Commit changes & add tests  
4. Submit a pull request

Ensure Rust code passes `cargo fmt` and `cargo clippy`, and Vue code passes linting.

## License

MIT License © 2025 Nick Tretyakov

