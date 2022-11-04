# State-Sync for Arable bamboo_9000-1 testnet

Stop your node and reset:

```bash
sudo systemctl stop acred && acred tendermint unsafe-reset-all --home $HOME/.acred
```

Set RPC:

```bash
RPC="https://rpc-arable.sr20de.xyz:443"
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
peers="edd42d8ea771458c28d3a7edd267f8b82b1ed45f@95.217.160.249:26656,
b6480b63c60c7974d71413c23b0cf2d37ed9df7f@65.108.134.78:21095,
6f71a4782ad31b24cfcd86f0973708a158ebf05e@65.108.100.214:26656,
cc751411f9e40f8ded410872ea2c73f8261ac0c7@135.181.72.187:11593,
f6c9febf6502e25e4882c622b61fa6f05d4986be@95.217.224.252:25656,
a1900a1eca73c08a2b5718d07d8b649d6e1e0fc9@94.237.27.199:26646,
9c3c083535b948492dd2d9b3ff88f22ee859e51c@176.9.53.27:26666,
ec213c011aa4eaaba2cac641ff7cf4c6034c3ed5@116.202.117.229:26256,
6a0f8aae236aef784d7781265c2e2daf3018367e@202.61.229.238:26656,
2fa06d2723e4440fb54d014e82565bc704e0c827@194.163.172.168:26656,
5fae9791c7a2265a7fbd4d31774fe8bc600fb594@161.97.80.151:26656,
9354fc84bb26b159087b3d065a40b2497b64d591@162.55.1.2:26256,
33a9c1f6d7a978441340905ef6350f6875b53af2@31.7.196.68:26656,
221460c042f6b8314308f4f522b1bdbc15cca1f0@31.7.196.66:26656,
e84f18aff98c4d0632c2e5f696bfa14ac15481e1@93.189.30.114:26656,
6bacc01640421479eef75794919632872e76b69a@65.108.141.152:16656,
bbd7f482a8d4932883d048da67b6f5b880b071ec@65.21.53.39:6656,
213dea614ff4b1ffc6ebbbddb6152ed2d311cc6e@65.108.134.84:26606,
1265e77e6a5c91d3b62dc34e123b6a045b186deb@23.88.66.239:26256,
cfdfcfc4c31983ed8dca9a1438237bf7557c2747@31.7.196.70:26656,
51d9531e1c40e6d261be30c0883ee01aca9824e1@123.193.196.217:36656,
1b015a2d5e98e504f598057adbcda485ebd3bad4@31.7.196.65:26656,
fa0f01b90862ff9490ce96f9e3656723d0db6122@65.108.78.41:46656,
ef6fc69ba60b177221c816a628ace13bb4342230@147.156.57.65:26656,
ffdd3c4f05b6b080907c40f4f36ac95998dd2c9e@65.109.28.177:49466,
483eea73b95fadce8b99b8157d3abd78757160d6@161.97.104.185:26656,
b2812b77bf7d11071a21c4561b7e46608f090238@185.250.148.210:26656,
2a8c4dc560a7b28efa4b9fc93bc3492f99b6fe0c@3.17.187.160:26656,
f4ad618dac1a49603c188581ead1b1366125eb4b@204.27.59.74:26656,
79f234d95580c1b49840137131e65a1ce23667cf@65.108.204.119:26616,
8543680fc9434f48e96ce11569c48083397798d3@89.245.24.69:21256,
6c5460d6f0f96da4f7ef5196553b01e8e320c845@91.223.89.168:26656,
78529083f299e7f2ea5de2699920882f71ed39a6@147.156.57.66:26656,
6a2dfdcb9af59e4fe5842be05a83ef47d647be31@185.252.235.174:26656,
ae6c86f47b509cecb6a7cd1b8ac588d1069df491@62.171.151.100:26656,
1ef0826e6149afa6ed4441a46a1d1ce19f3cf041@141.95.104.246:26656,
2f8ed1460b619797d8127708ac702222ace3f30c@147.156.56.40:26656,
1dd071f79f57ec8b5e293aa3373cce481d94c1b0@147.75.199.11:26656,
18e15814131fbed8d17c7b3bb74825443c89a00d@202.61.207.206:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.acred/config/config.toml
```
Let's make changes to the config.toml:

```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.acred/config/config.toml
```

Start your node:

```bash
sudo systemctl restart acred && sudo journalctl -u acred -f --no-hostname -o cat
```
