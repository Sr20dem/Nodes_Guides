# State-Sync for Nois

Chain **nois-testnet-003**

Stop your node and reset:

```bash
sudo systemctl stop noisd && noisd tendermint unsafe-reset-all --home $HOME/.noisd --keep-addr-book
```

Set RPC:

```bash
RPC="http://130.255.170.151:36657"
```
Set variables `$LATEST_HEIGHT, $BLOCK_HEIGHT, $TRUST_HASH`:

```bash
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
```
Check that the values are received:

```bash
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
If the output is similar to this, then everything is fine and you can proceed to the next step

```bash
786996 784996 22B9F77F85F7A177736209AA81558CB85889C15E774DFFA17B0FE3B01F01CC44
```
Configure persistent peers:
```bash
peers="6e128ad6151f5be3d38c9ac5a960d4d0448169af@130.255.170.151:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.noisd/config/config.toml
```

Start your node:

```bash
sudo systemctl restart noisd && sudo journalctl -u noisd -f --no-hostname -o cat
```