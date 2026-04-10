# BTCPay Server Decred Plugin

A [BTCPay Server](https://github.com/btcpayserver/btcpayserver) plugin that enables receiving payments via [Decred](https://decred.org/).

The plugin talks directly to `dcrd` and `dcrwallet` via JSON-RPC. No NBitcoin or NBXplorer dependency required.

## Deployment

If you already run BTCPay Server via [btcpayserver-docker](https://github.com/btcpayserver/btcpayserver-docker):

1. SSH into your BTCPay Server host.

2. Enable Decred and regenerate the docker-compose file:

   ```bash
   cd "$BTCPAY_BASE_DIRECTORY/btcpayserver-docker"
   export BTCPAYGEN_CRYPTO2=dcr
   . btcpay-setup.sh -i
   ```

   This starts `dcrd` and `dcrwallet` containers alongside your existing stack. The blockchain will take some time to sync.

3. To enable sending from the wallet (withdrawals), add the wallet passphrase to the `.env` file and restart:

   ```bash
   cd "$BTCPAY_BASE_DIRECTORY/btcpayserver-docker"
   echo "BTCPAY_DCR_WALLET_PASSPHRASE=your-wallet-passphrase" >> .env
   . btcpay-setup.sh -i
   ```

   This unlocks dcrwallet at startup so the Send page can create transactions. Without this, receiving payments still works but sending is disabled.

4. Install the plugin from your BTCPay Server admin panel under **Server Settings > Plugins**. Search for "Decred" and click Install.

5. Go to **Store Settings > Decred** to verify the connection status. Once `dcrd` is synced and `dcrwallet` is connected, the store is ready to accept DCR.

6. Create an invoice and select Decred as the payment method. To withdraw funds, use the Send button on the Decred settings page.

### Manual deployment

If you are not using btcpayserver-docker, you need:

- BTCPay Server >= 2.1.0
- A running `dcrd` node with `--txindex` enabled
- A running `dcrwallet` instance connected to the node

Set these environment variables on your BTCPay Server instance:

| Variable | Description | Example |
|---|---|---|
| `BTCPAY_DCR_DAEMON_URI` | dcrd RPC endpoint | `http://dcrd:9109` |
| `BTCPAY_DCR_WALLET_DAEMON_URI` | dcrwallet RPC endpoint | `http://dcrwallet:9110` |
| `BTCPAY_DCR_DAEMON_USERNAME` | RPC username | `btcpay` |
| `BTCPAY_DCR_DAEMON_PASSWORD` | RPC password | `btcpay` |

Then install the plugin DLL manually by placing it in BTCPay Server's `Plugins` directory and restarting.

## Using dcrctl

The Docker containers include `dcrctl` for direct wallet and node management. Run commands via `docker exec`:

```bash
# Shorthand for running dcrctl against dcrwallet
alias dcrctl-wallet="docker exec btcpayserver_dcrwallet dcrctl --wallet --rpcuser=btcpay --rpcpass=btcpay --notls"

# Shorthand for running dcrctl against dcrd
alias dcrctl-node="docker exec btcpayserver_dcrd dcrctl --rpcuser=btcpay --rpcpass=btcpay --notls"
```

Common commands:

```bash
# Check wallet balance
dcrctl-wallet getbalance

# Get a new receiving address
dcrctl-wallet getnewaddress

# List recent transactions
dcrctl-wallet listtransactions

# Rescan the blockchain (e.g. after importing keys or if transactions are missing)
dcrctl-wallet rescanwallet

# Check node sync status
dcrctl-node getblockchaininfo

# Get node connection info
dcrctl-node getpeerinfo
```

## Docker image

The Docker image containing `dcrd`, `dcrwallet`, and `dcrctl` is published to GHCR. The `docker-fragment/` directory contains the compose fragments for btcpayserver-docker.

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
