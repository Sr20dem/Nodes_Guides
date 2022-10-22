# State-Sync for Kyve

Chain **Korellia**

Stop your node and reset:

```bash
sudo systemctl stop chaind && chaind tendermint unsafe-reset-all --home $HOME/.kyve
```

Set RPC:

```bash
SNAP_RPC1="65.108.126.46:28657" 
SNAP_RPC2="65.108.57.92:26657"
SNAP_RPC3="95.216.157.18:26657" 
```
Set variables `$LATEST_HEIGHT, $BLOCK_HEIGHT, $TRUST_HASH`:

```bash
LATEST_HEIGHT=$(curl -s $SNAP_RPC1/block | jq -r .result.block.header.height); \
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
peers="e9a2567ee5fda3f98fce7b0e4342ef1e85c9fed9@141.95.124.151:20056,287c511b6ca0c6a5853cf52021bbbabfee6c7219@65.108.126.46:28656,d040c94305f0b421df815d5375201b34f8cae999@65.108.57.92:26656,2823ef5801b138802d076bf3e0478ec9be4e7bde@95.216.157.18:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.kyve/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC1,$SNAP_RPC2,$SNAP_RPC3\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.kyve/config/config.toml
```

Start your node:

```bash
sudo systemctl restart chaind && sudo journalctl -u chaind -f --no-hostname -o cat
```