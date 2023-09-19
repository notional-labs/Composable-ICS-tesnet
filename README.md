# Composable Testnet Documentation
## Information
- Network information: https://github.com/notional-labs/Composable-ICS-tesnet
- Chain ID: `banksy-testnet-3`
- Genesis: https://raw.githubusercontent.com/notional-labs/Composable-ICS-tesnet/main/genesis.json
- Binary: https://github.com/notional-labs/Composable-ICS-tesnet/raw/main/binaries/v5.0.0/centaurid
- Current version: `v5.0.0`
- Peers: `c0f197bdf6c4a4a16eb9db112d1ec9545336fd43@168.119.91.22:2250,364b8245e72f083b0aa3e0d59b832020b66e9e9d@65.109.80.150:21500`
- Public Notional endpoints: 
    - RPC: `https://rpc-banksy.notional.ventures:443`
    - API: `https://api-banksy.notional.ventures:443`
    - gRPC: `https://grpc-banksy.notional.ventures:443`
- Block Explorer: `https://explorer.nodexcapital.com/banksy-testnet`
## Setup Instruction
**1. Building the binary**

There are two ways of setting up the `centaurid` binary, building from source or installing from binary URL, which is mentioned in the install script in section **2. Joining testnet**.

To build the binary from source, run these commands:
```bash
#mkdir $HOME/go/bin # ignore this command if you already have $HOME/go/bin folder
export PATH=$PATH:$HOME/go/bin
cd $HOME
git clone https://github.com/notional-labs/composable-centauri
cd composable-centauri
git checkout v5.0.0 # Using v5.0.0
make install
centaurid version # v5.0.0
```

**2. Joining testnet**
Here is a full script to install `centaurid` binary and run the node with state sync. This script should be run with administration priviledges by running `sudo script.sh`:
```bash
# script.sh
#mkdir $HOME/go/bin # ignore this command if you already have $HOME/go/bin folder
export PATH=$PATH:$HOME/go/bin
wget https://github.com/notional-labs/composable-centauri/releases/download/v5.0.0/centaurid -O $HOME/go/bin.centaurid
chmod +x $HOME/go/bin.centaurid
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/raw/main/internal/api/libwasmvm.x86_64.so
centaurid version # v5.0.0
centaurid init <moniker> --chain-id banksy-testnet-3
wget https://raw.githubusercontent.com/notional-labs/Composable-ICS-tesnet/main/genesis.json -O $HOME/.banksy/config/genesis.json


# state sync
SNAP_RPC="https://rpc-banksy.notional.ventures:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.banksy/config/config.toml

# run node
centaurid start --p2p.seeds 364b8245e72f083b0aa3e0d59b832020b66e9e9d@65.109.80.150:21500,c0f197bdf6c4a4a16eb9db112d1ec9545336fd43@168.119.91.22:2250
```

**3. Join testnet as a validator**
To join `banksy-testnet-3` as a validator, you should setup a running node as in **step 1** above and wait for it to be fully synced, and then setup validator:
- Create a key:
    ```bash
    centaurid keys add <validator-key> # generate a new key, or use `--recover` to recover an existed key with mnemonic
    ```
- For testnet tokens, head to `testnet-3-faucet` [channel](https://discord.gg/An8nXEGP) on the Composable discord and send the following message with you address included
    ```
    $request <address> composable
    ```
- To see the current amount of the address, use:
    ```
    $balance <address> composable
    ```
- Create validator:
    ```bash
    centaurid tx staking create-validator --amount=1000000000000ppica --moniker="<validator-name>" --chain-id=banksy-testnet-3 --commission-rate="0.05"  --commission-max-change-rate="0.01" --commission-max-rate="0.20" --from=<validator-key> --node=https://rpc-banksy.notional.ventures:443 --gas=auto --min-self-delegation 10 --pubkey=$(centaurid tendermint show-validator)
    ```

- If you want to add validator info:
    ```bash
    centaurid tx staking edit-validator \
        --website="" \ # URL to validator website
        --identity="" \ # keybase.io identity 
        --details="" \ # Additional detail 
        --security-contact="" \ # security email
        --from=<validator-key> \
        --node="https://rpc-banksy.notional.ventures:443"
    ```
