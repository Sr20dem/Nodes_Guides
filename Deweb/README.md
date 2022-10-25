# State-Sync for Deweb Sirius Testnet

Stop your node and reset:

```bash
sudo systemctl stop dewebd && dewebd unsafe-reset-all
```

Set RPC:

```bash
RPC="http://45.67.32.53:26657"
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
34853 32853 22B9F77F85F7A177736209AA81558CB85889C15E774DFFA17B0FE3B01F01CC44
```
Configure persistent peers:
```bash
peers="61da70005efc69aee1392d880aa34532c06adfdc@164.215.102.44:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.deweb/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.deweb/config/config.toml
```

Start your node:

```bash
sudo systemctl restart dewebd && sudo journalctl -u dewebd -f --no-hostname -o cat
```
