# State-Sync for KYVE beta

Chain **beta**

Stop your node and reset:

```bash
sudo systemctl stop chaind && chaind tendermint unsafe-reset-all --home $HOME/.kyve
```

Set RPC:

```bash:
RPC="https://kyve-beta-api.sr20de.xyz:443"
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
729060 727060 8AD7A8AE02EDDC1157AE94E01585B7F6C1BFFFC332716DDA7FCBB1F9752C3F03
```
Configure persistent peers:
```bash
peers="67a2cec5e11f59dca3df5d33e0dcd1983dbb9244@62.171.144.51:21656,410bf0cb2cdb9a6e159c14b9d01531b9ecb1edd4@3.70.26.46:26656,e5b22ff8c98015a522da49f35240df2e190e9a99@77.83.92.238:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.kyve/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.kyve/config/config.toml
```

Start your node:

```bash
sudo systemctl restart chaind && sudo journalctl -u chaind -f --no-hostname -o cat
```
