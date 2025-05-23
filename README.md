# ⚡️fulmine

[![Go Version](https://img.shields.io/badge/Go-1.23.1-blue.svg)](https://golang.org/doc/go1.23)
[![GitHub Release](https://img.shields.io/github/v/release/ArkLabsHQ/fulmine)](https://github.com/ArkLabsHQ/fulmine/releases/latest)
[![License](https://img.shields.io/github/license/ArkLabsHQ/fulmine)](https://github.com/ArkLabsHQ/fulmine/blob/main/LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/ArkLabsHQ/fulmine)](https://github.com/ArkLabsHQ/fulmine/stargazers)
[![GitHub Issues](https://img.shields.io/github/issues/ArkLabsHQ/fulmine)](https://github.com/ArkLabsHQ/fulmine/issues)

![fulmine-og-v2](https://github.com/user-attachments/assets/8d59879d-727b-4aa7-8a9f-4d696406c6cf)


Fulmine is a Bitcoin wallet daemon that integrates Ark protocol's batched transaction model with Lightning Network infrastructure, enabling routing nodes, service providers and payment hubs to optimize channel liquidity while minimizing on-chain fees, without compromising on self-custody.

## 🚀 Usage

### 🐳 Using Docker (Recommended)

The easiest way to run fulmine is using Docker. Make sure you have [Docker](https://docs.docker.com/get-docker/) installed on your machine.

```bash
docker run -d \
  --name fulmine \
  -p 7000:7000 \
  -p 7001:7001 \
  -v fulmine-data:/app/data \
  ghcr.io/arklabshq/fulmine:latest
```

Once the container is running, you can access the web UI at [http://localhost:7001](http://localhost:7001).

To view logs:

```bash
docker logs -f fulmine
```

To stop the container:

```bash
docker stop fulmine
```

To update to the latest version:

```bash
docker pull ghcr.io/arklabshq/fulmine:latest
docker stop fulmine && docker rm fulmine
docker run -d \
  --name fulmine \
  -p 7000:7000 \
  -p 7001:7001 \
  -v fulmine-data:/app/data \
  ghcr.io/arklabshq/fulmine:latest
```

### 💻 Using the Binary

Alternatively, you can download the latest release from the [releases page](https://github.com/ArkLabsHQ/fulmine/releases) for your platform. After downloading:

1. Extract the binary
2. Make it executable (on Linux/macOS): `chmod +x fulmine`
3. Run the binary: `./fulmine`

### 🔧 Environment Variables

The following environment variables can be configured:

| Variable | Description | Default |
|----------|-------------|---------|
| `FULMINE_DATADIR` | Directory to store wallet data | `/app/data` in Docker, `~/.fulmine` otherwise |
| `FULMINE_HTTP_PORT` | HTTP port for the web UI and REST API | `7001` |
| `FULMINE_GRPC_PORT` | gRPC port for service communication | `7000` |
| `FULMINE_ARK_SERVER` | URL of the Ark server to connect to | It pre-fills with the default Ark server |
| `FULMINE_ESPLORA_URL` | URL of the Esplora server to connect to | It pre-fills with the default Esplora server |
| `FULMINE_UNLOCKER_TYPE` | Type of unlocker to use for auto-unlock (`file` or `env`) | Not set by default (no auto-unlock) |
| `FULMINE_UNLOCKER_FILE_PATH` | Path to the file containing the wallet password (when using `file` unlocker) | Not set by default |
| `FULMINE_UNLOCKER_PASSWORD` | Password string to use for unlocking (when using `env` unlocker) | Not set by default |
| `FULMINE_BOLTZ_URL` | URL of the custom Boltz backend to connect to for swaps | Not set by default |
| `FULMINE_BOLTZ_WS_URL` | URL of the custom Boltz WebSocket backend to connect to for swap events | Not set by default |

When using Docker, you can set these variables using the `-e` flag:

```bash
docker run -d \
  --name fulmine \
  -p 7001:7001 \
  -e FULMINE_HTTP_PORT=7001 \
  -e FULMINE_ARK_SERVER="https://server.example.com" \
  -e FULMINE_ESPLORA_URL="https://mempool.space/api" \
  -e FULMINE_UNLOCKER_TYPE="file" \
  -e FULMINE_UNLOCKER_FILE_PATH="/app/password.txt" \
  -v fulmine-data:/app/data \
  -v /path/to/password.txt:/app/password.txt \
  ghcr.io/arklabshq/fulmine:latest
```

### 🔑 Auto-Unlock Feature

Fulmine supports automatic wallet unlocking on startup, which is useful for unattended operation or when running as a service. Two methods are available:

1. **File-based unlocker**: Reads the wallet password from a file
   ```
   FULMINE_UNLOCKER_TYPE=file
   FULMINE_UNLOCKER_FILE_PATH=/path/to/password/file
   ```

2. **Environment-based unlocker**: Uses a password directly from an environment variable
   ```
   FULMINE_UNLOCKER_TYPE=env
   FULMINE_UNLOCKER_PASSWORD=your_wallet_password
   ```

⚠️ **Security Warning**: When using the auto-unlock feature, ensure your password is stored securely:
- For file-based unlocking, use appropriate file permissions (chmod 600)
- For environment-based unlocking, be cautious about environment variable visibility
- Consider using Docker secrets or similar tools in production environments

## 📚 API Documentation

### ⚠️ Security Notice

**IMPORTANT**: The REST API and gRPC interfaces are currently **not protected** by authentication. This is a known limitation and is being tracked in [issue #98](https://github.com/ArkLabsHQ/fulmine/issues/98). 

**DO NOT** expose these interfaces over the public internet until authentication is implemented. The interfaces should only be accessed from trusted networks or localhost.

While the wallet seed is encrypted using AES-256 with a password that the user set, the API endpoints themselves are not protected.

### 🔌 API Interfaces

fulmine provides three main interfaces:

1. **Web UI** - Available at [http://localhost:7001](http://localhost:7001) by default
2. **REST API** - Available at [http://localhost:7001/api](http://localhost:7001/api)
3. **gRPC Service** - Available at `localhost:7000`

### 💰 Wallet Service

1. Generate Seed

   ```sh
   curl -X GET http://localhost:7001/api/v1/wallet/genseed
   ```

2. Create Wallet

   Password must:
   - Be 8 chars or longer
   - Have at least one number
   - Have at least one special char

   Private key supported formats:
   - 64 chars hexadecimal
   - Nostr nsec (NIP-19)
  
   ```sh
   curl -X POST http://localhost:7001/api/v1/wallet/create \
        -H "Content-Type: application/json" \
        -d '{"private_key": <hex or nsec>, "password": <strong password>, "server_url": "https://server.example.com"}'
   ```

3. Unlock Wallet

   ```sh
   curl -X POST http://localhost:7001/api/v1/wallet/unlock \
        -H "Content-Type: application/json" \
        -d '{"password": <strong password>}'
   ```

4. Lock Wallet

   ```sh
   curl -X POST http://localhost:7001/api/v1/wallet/lock \
        -H "Content-Type: application/json"
   ```

5. Get Wallet Status

   ```sh
   curl -X GET http://localhost:7001/api/v1/wallet/status
   ```

### ⚡ Service API

1. Get Address

   ```sh
   curl -X GET http://localhost:7001/api/v1/address
   ```

2. Get Balance

   ```sh
   curl -X GET http://localhost:7001/api/v1/balance
   ```

3. Send funds offchain

   ```sh
   curl -X POST http://localhost:7001/api/v1/send/offchain \
        -H "Content-Type: application/json" \
        -d '{"address": <ark address>, "amount": <in sats>}'
   ```

4. Send funds onchain

   ```sh
   curl -X POST http://localhost:7001/api/v1/send/onchain \
        -H "Content-Type: application/json" \
        -d '{"address": <bitcoin address>, "amount": <in sats>}'
   ```

5. Settle transactions, Renew VTXOs or swap boarding UTXOs for VTXOs

   ```sh
   curl -X GET http://localhost:7001/api/v1/settle
   ```

6. Get transaction history

   ```sh
   curl -X GET http://localhost:7001/api/v1/transactions
   ```

7. Refund VHTLC Without Receiver
   Refunds a VHTLC output without requiring the receiver's cooperation. Useful for reclaiming funds after timeout if the receiver is unavailable.

   ```sh
   curl -X POST http://localhost:7001/api/v1/vhtlc/refundWithoutReceiver \
        -H "Content-Type: application/json" \
        -d '{"preimage_hash": "<hex preimage hash>"}'
   ```
   - Replace `<hex preimage hash>` with the actual preimage hash for the VHTLC you wish to refund.
   - Returns: `{ "redeem_txid": "<txid>" }` on success.

Note: Replace `http://localhost:7001` with the appropriate host and port where your fulmine is running. Also, ensure to replace placeholder values (like `strong password`, `ark_address`, etc.) with actual values when making requests.

For more detailed information about request and response structures, please refer to the proto files in the `api-spec/protobuf/fulmine/v1/` directory.

## 👨‍💻 Development

To get started with fulmine development you need Go `1.23.1` or higher and Node.js `18.17.1` or higher.

```bash
git clone https://github.com/ArkLabsHQ/fulmine.git
cd fulmine
go mod download
make run
```

Now navigate to [http://localhost:7001/](http://localhost:7001/) to see the dashboard.

### Testing

Run all unit tests:
```bash
make test
```

Run VHTLC-specific unit tests:
```bash
make test-vhtlc
```

Run integration tests:
```bash
make build-test-env
make up-test-env
make setup-test-env
make integrationtest
make down-test-env
```

## 🤝 Contributing

We welcome contributions to fulmine! Here's how you can help:

1. **Fork the repository** and create your branch from `main`
2. **Install dependencies**: `go mod download`
3. **Make your changes** and ensure tests pass: `make test`4. **Run the linter** to ensure code quality: `make lint`
4. **Submit a pull request**

For major changes, please open an issue first to discuss what you would like to change.

### 🛠️ Development Commands

The Makefile contains several useful commands for development:

- `make run`: Run in development mode
- `make build`: Build the binary for your platform
- `make test`: Run unit tests
- `make lint`: Lint the codebase
- `make proto`: Generate protobuf stubs (requires Docker)

## Support

If you encounter any issues or have questions, please file an issue on our [GitHub Issues](https://github.com/ArkLabsHQ/fulmine/issues) page.

## Security

We take the security of Ark seriously. If you discover a security vulnerability, we appreciate your responsible disclosure.

Currently, we do not have an official bug bounty program. However, we value the efforts of security researchers and will consider offering appropriate compensation for significant, [responsibly disclosed vulnerabilities](./SECURITY.md).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
