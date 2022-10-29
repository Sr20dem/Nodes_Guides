# State-Sync for Tgrade

Stop your node and reset:

```bash
sudo systemctl stop tgrade && tgrade tendermint unsafe-reset-all --home ~/.tgrade
```

Set RPC:

```bash
RPC="https://tgrade-api.sr20de.xyz:443"
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
948537 946537 22B9F77F85F7A177736209AA81558CB85889C15E774DFFA17B0FE3B01F01CC44
```
Configure persistent peers:
```bash
peers="8f73ed46016df93a0ab772c279b2f329b2f38d56@130.255.170.165:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.tgrade/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.tgrade/config/config.toml
```

Start your node:

```bash
sudo systemctl restart tgrade && sudo journalctl -u tgrade -f --no-hostname -o cat
```
