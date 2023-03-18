![source](source.jpg)

## Source links:

- [Website](https://www.sourceprotocol.io/)
- [Discord](https://discord.gg/T8H35WAWAZ)
- [Twitter](https://twitter.com/sourceprotocol_)

Explorers:
- [Sourceprotocol](https://explorer.testnet.sourceprotocol.io/source)
- [Nodeist](https://exp.nodeist.net/Source/staking)
- [Stake-take](https://explorer.stake-take.com/source/)

## Installation

Install required packages

```bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential git make ncdu htop screen unzip bc fail2ban htop -y
```

Install Go language

```bash
ver="1.20.2" && \
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
Install Source Protocol binary (01.08.22)

```bash
git clone -b testnet https://github.com/Source-Protocol-Cosmos/source.git
cd ~/source
make install
```

Check version, must be 1.0.0

```bash
sourced version --long | head

#version: v1.0.0-2-ge06b810
#commit: e06b810e842e57ec8f5432c9cdd57883a69b3cee

```

Initialize the node, replace "moniker" with your name

```bash
sourced init <moniker> --chain-id=sourcechain-testnet
```

Add wallet

```bash
sourced keys add <walletName>

#or recover wallet from seed phrase

sourced keys add <walletName> --recover
```

Download genesis file

```bash
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourcechain-testnet/genesis.json > ~/.source/config/genesis.json

#check sha256sum
sha256sum $HOME/.source/config/genesis.json
#2bf556b50a2094f252e0aac75c8018a9d6c0a77ba64ce39811945087f6a5165d genesis.json
```

Setup seeds and peers

```bash
PEERS="8b75c926d4060560dbbead7d8b0300b7b411ff9b@5.252.193.133:26656,49b025c08193c8846956423ac80504b0bab2b777@185.182.187.239:26656,5755422056c55063f76e4dd0c4245904640ec34b@135.181.149.90:26656,cac254555deea35a70c821abd7f3e7db47a46d55@65.109.92.241:20056,b99c46a83e72280ccdb81994fd60b9b1cc74b1ab@84.21.171.142:26656,4ede26dd5fbb87bd9dba462fe2c3c3e39e15c8f2@207.180.224.128:46656,bdf9b6ad38b803358e7fd99f35b14795ebcd8144@190.2.155.67:29656,503ec9be5c5542700b7f93d65dfc68371d38e6e9@16.163.74.176:26656,b02e2bd359623aeee2d4fad94d37af8b064508f6@167.235.224.141:26656,f2936d8f0ae99b9fa99d179f746faacc9c41a5c3@65.108.158.181:26656,03d324b03078e3bd38c7c7550988362d11106ce4@135.181.198.246:26656,46ae715de3bcf284ff997b841e6e82f279e3654f@154.26.153.179:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.source/config/config.toml
```
Set up the minimum gas price

```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0usource\"/;" ~/.source/config/app.toml
```

Download addrbook

```bash
wget -O $HOME/.source/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Source/addrbook.json"
```
Create a service file

```bash
sudo tee /etc/systemd/system/sourced.service > /dev/null <<EOF
[Unit]
Description=source
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sourced) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Start node

```bash
sudo systemctl daemon-reload && \
sudo systemctl enable sourced && \
sudo systemctl restart sourced && \
sudo journalctl -u sourced -f -o cat
```









