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
ver="1.18.5" && \
cd $HOME && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
Install Source Protocol binary

```bash
git clone -b testnet https://github.com/Source-Protocol-Cosmos/ && \source.git
cd ~/source && \
make install
```

Check version, must be 1.0.0

```bash
sourced version --long | head
```

Initialize the node, replace "moniker" with your name

```bash
sourced init <moniker> --chain-id=sourcechain-testnet
```

Add wallet

```bash
sourced keys add <walletName>
```

Download genesis file

```bash
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourcechain-testnet/genesis.json > ~/.source/config/genesis.json
```

Setup seeds and peers

```bash
PEERS="9d16b552697cdce3c8b4f23de53708533d99bc59@165.232.144.133:26656,d565dd0cb92fa4b830662eb8babe1dcdc340c321@44.234.26.62:26656,2dbc3e6d52e5eb9357aec5cf493718f6078ffaad@144.76.224.246:36656"
SEEDS="6ca675f9d949d5c9afc8849adf7b39bc7fccf74f@164.92.98.17:26656"
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









