# State-Sync for Hypersign. Jagrat testnet

Stop your node and reset:

```bash
sudo systemctl stop hid-noded && hid-noded tendermint unsafe-reset-all --home $HOME/.hid-node
```

Set RPC:

```bash
RPC="https://rpc-hypersign.sr20de.xyz:443"
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
You can view the list of peers by running this command in a terminal:
```bash
curl -sS https://rpc-hypersign.sr20de.xyz/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)","}'
```



Configure persistent peers:
```bash
peers="f1e8d741d7437d62c15337e5f7475e139119cf8b@65.108.229.233:31656,
3f658dc173540479ba70f58f27d60fa9de83378a@195.201.165.123:21056,
52eee2c34150d621312087e49f118969472ba55f@149.102.137.192:26656,
9cdb081f41dc897111a8e8eddcae22e7384b8cb2@167.86.125.117:26656,
e7bb31c8fdd8d26a739bfd87cdf3ba7a8f90406e@65.21.145.228:31656,
776785ba52f350e10c0eaba22731e0891edb07fc@154.12.236.152:26656,
91089c0911b59f59fe2ec79fdae017f9beefbbfd@65.108.101.158:26656,
22d2b3587e2ce6ae750c189b12461e7315d08ae4@167.235.151.119:26656,
ec5127072c252f7246fb66f7e7762423a23ff6bd@154.12.228.93:31656,
15d2f1bc2bfaa143388465ea115c59e5ce6e77dc@65.109.39.223:26656,
70f00c612c1d681a04244749a56f3a35e9be1420@65.108.194.40:28765,
97387c1024c0b7c5478899d598937bdaf3871cb1@213.133.102.206:21066,
3990d5a402ca8f9e53441b02e22f4558c5c85fc5@65.108.44.149:27756,
5cd888a5c37474ca778277cfd9dee7d24fe96094@95.217.214.107:26656,
1de2abae74a4c5fd7d96d9869ef02187f81498f0@134.209.238.66:26656,
8e4938aa6561695326f61f432ea2b2a53a428205@95.217.118.96:27161,
4aa182ce191cd089929544fe0612d33a02a2cde9@46.17.250.145:26656,
5b4482bfe02384184470070c3d3a4465cf0c18d4@144.91.82.61:31656,
aa8c0064e866dc57b341a389006df8925a0718fe@5.161.55.130:31656,
4d618f2e15c35f92e9ce18113a350fdfed678165@135.181.24.128:27656,
de1f980cc59bdb2457202768d4b4d964d783789e@167.235.21.165:36656,
2641ddcf28d8adf448edb573de1efba0b6971d9e@178.154.222.128:26656,
84408be4e3f13dcd976568d6370e1c50e9eb614d@185.252.232.110:46656,
1dae68f061204fe2c10e9476239c0333258889e7@65.109.31.114:2460,
cf94099349980f9593a3f0362c85fe7c6eda8b14@8.219.48.59:26656,
fd8a8905a404d4169e9a9ed7f4b034079e6a13ab@65.108.77.250:46151,
ac25bdc230944cc20f03913a8dae881c9b5f9c18@3.239.45.125:26656,
020f85954af998b22590835af3b5a7524393c161@144.91.120.124:26656,
2eb8a0e9e8b32e0890a8ecde766e1ab80126fddf@135.181.5.47:21056"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.hid-node/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.hid-node/config/config.toml
```

Start your node:

```bash
sudo systemctl restart hid-noded && sudo journalctl -u hid-noded -f --no-hostname -o cat
```
