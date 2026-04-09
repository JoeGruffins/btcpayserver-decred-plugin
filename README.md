# BTCPay Server Decred Plugin

A [BTCPay Server](https://github.com/btcpayserver/btcpayserver) plugin that enables receiving payments via [Decred](https://decred.org/).

The plugin talks directly to `dcrd` and `dcrwallet` via JSON-RPC. No NBitcoin or NBXplorer dependency required.

## Requirements

- BTCPay Server >= 2.1.0
- A running `dcrd` node
- A running `dcrwallet` instance connected to the node

## Configuration

Set the following environment variables on your BTCPay Server instance:

| Variable | Description | Example |
|---|---|---|
| `BTCPAY_DCR_DAEMON_URI` | dcrd RPC endpoint | `http://dcrd:9109` |
| `BTCPAY_DCR_WALLET_DAEMON_URI` | dcrwallet RPC endpoint | `http://dcrwallet:9110` |
| `BTCPAY_DCR_DAEMON_USERNAME` | RPC username | `btcpay` |
| `BTCPAY_DCR_DAEMON_PASSWORD` | RPC password | `btcpay` |

## Docker

A Docker image containing `dcrd`, `dcrwallet`, and `dcrctl` is published to GHCR. See the `docker-fragment/` directory for the `btcpayserver-docker` compose fragment.

## Development

### Building

```bash
git clone --recursive https://github.com/user/btcpayserver-decred-plugin.git
cd btcpayserver-decred-plugin
dotnet build Plugins/Decred/BTCPayServer.Plugins.Decred.csproj
```

If you get `NETSDK1226: Prune Package data not found` errors, apply this workaround before building:

```bash
echo '<Project><PropertyGroup><AllowMissingPrunePackageData>true</AllowMissingPrunePackageData></PropertyGroup></Project>' > submodules/btcpayserver/Directory.Build.props
```

### Testing with the dcrdex simnet harness

```bash
# Start the harness (requires dcrd, dcrwallet, dcrctl in PATH)
cd /path/to/dcrdex/dex/testing/dcr && bash harness.sh

# In another terminal, run the integration tests
cd btcpayserver-decred-plugin
dotnet run --project Tests/HarnessTest.csproj
```

For full end-to-end testing (running BTCPayServer with the plugin, creating invoices, and verifying payments), see [docs/e2e-testing.md](docs/e2e-testing.md).

## License

[MIT](LICENSE.md)
