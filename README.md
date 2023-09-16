
![image](https://github.com/molla202/Selfchain/assets/91562185/c898d982-4ea4-49d0-bdc2-70a8edd547fb)



![image](https://github.com/molla202/Selfchain/assets/91562185/14802209-1a7e-49f5-92af-1b8b0751cd3b)
* kanala validator rol deyince bota mesaj atıyor.
* sunucuda aşağıdaki kodu yazıyoruz ve node id kısmını alıp mesaj atan bota verıyoruz sonra sunucu ipsini yazıyoruz bunları node eşleşmesi bitince yapıyoruz
```
selfchaind status
```
* id olarak yazıyor.
![image](https://github.com/molla202/Selfchain/assets/91562185/ce062d3a-bd58-47db-ac79-2700677944b9)


https://explorer.nodestake.top/self-testnet

NOT: https://discord.gg/GDe7NDzk
dicorda griyoruz .

# Update Packages
```
sudo apt update && apt upgrade -y
sudo apt install curl git jq lz4 build-essential unzip fail2ban ufw -y
```

# Install Go
```
ver="1.20"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
```
# Install Cosmovisor (OPTIONAL If you are using Cosmovisor)
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

# Download and install binary
```
cd $HOME
wget https://snapshots.indonode.net/selfchain/selfchaind
sudo chmod +x selfchaind
```
```
# Setup Cosmovisor Symlinks
mkdir -p $HOME/.selfchain/cosmovisor/genesis/bin
mv selfchaind $HOME/.selfchain/cosmovisor/genesis/bin/

sudo ln -s $HOME/.selfchain/cosmovisor/genesis $HOME/.selfchain/cosmovisor/current
sudo ln -s $HOME/.selfchain/cosmovisor/current/bin/selfchaind /usr/local/bin/selfchaind


# Set Configuration for your node
selfchaind config chain-id self-dev-1
selfchaind config keyring-backend test
```
```
# Init your node
# You can change "MyNode" to anything you like
selfchaind init MyNode --chain-id self-dev-1
```
```

# Add Genesis File and Addrbook
wget -O $HOME/.selfchain/config/genesis.json  https://raw.githubusercontent.com/hotcrosscom/selfchain-genesis/main/networks/devnet/genesis.json
curl -Ls https://snapshots.indonode.net/selfchain/addrbook.json > $HOME/.selfchain/config/addrbook.json

#Configure Seeds and Peers
SEEDS="94a7baabb2bcc00c7b47cbaa58adf4f433df9599@157.230.119.165:26656,d3b5b6ca39c8c62152abbeac4669816166d96831@165.22.24.236:26656,35f478c534e2d58dc2c4acdf3eb22eeb6f23357f@165.232.125.66:26656"
PEERS="94a7baabb2bcc00c7b47cbaa58adf4f433df9599@157.230.119.165:26656,d3b5b6ca39c8c62152abbeac4669816166d96831@165.22.24.236:26656,35f478c534e2d58dc2c4acdf3eb22eeb6f23357f@165.232.125.66:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.selfchain/config/config.toml

PEERS="$(curl -sS http://157.230.119.165:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.selfchain/config/config.toml


# Set Pruning, Enable Prometheus, Gas Price, and Indexer
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="19"

sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.selfchain/config/app.toml
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.selfchain/config/config.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.selfchain/config/config.toml
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uself\"/" $HOME/.selfchain/config/app.toml
```
### Port değiştirme
```
echo "export SELF_PORT="42"" >> $HOME/.bash_profile
source $HOME/.bash_profile

# app.toml da port değiştiriyoruz çakışmasınlar :D
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${SELF_PORT}317\"%;
s%^address = \":8080\"%address = \":${SELF_PORT}080\"%;
s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${SELF_PORT}090\"%; 
s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${SELF_PORT}091\"%; 
s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${SELF_PORT}545\"%; 
s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${SELF_PORT}546\"%" $HOME/.selfchain/config/app.toml

# config.toml port ayarı
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SELF_PORT}658\"%; 
s%^laddr = \"tcp://0.0.0.0:26657\"%laddr = \"tcp://0.0.0.0:${SELF_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SELF_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SELF_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SELF_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SELF_PORT}660\"%" $HOME/.selfchain/config/config.toml
```
```
# Set Service file
sudo tee /etc/systemd/system/selfchaind.service > /dev/null << EOF
[Unit]
Description=selfchaind testnet node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.selfchain"
Environment="DAEMON_NAME=selfchaind"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable selfchaind
```
```

# Start the Node
sudo systemctl restart selfchaind
sudo journalctl -fu selfchaind -o cat
```
### validator
```
selfchaind tx staking create-validator \
    --amount=1000000000uself \
    --node http://165.232.125.66:26657 \
    --pubkey=$(selfchaind tendermint show-validator) \
    --moniker="moniker-yazınız" \
    --website="https://selfchain.xyz" \
    --details="This is a validator that was added post genesis" \
    --chain-id="self-dev-1" \
    --commission-rate="0.10" \
    --commission-max-rate="0.15" \
    --commission-max-change-rate="0.05" \
    --min-self-delegation="1000000000" \
    --broadcast-mode block \
    --gas="auto" \
    --gas-adjustment="1.2" \
    --gas-prices="0.5uself" \
    --keyring-backend file \
    --keyring-dir $HOME/keys \
    --from="adres-yazınız"
```
### edit validator
```
selfchaind tx staking edit-validator \
    --new-moniker "moniker-yazınız" \
    --from="adres-yazınız" \
    --identity="" \
    --website="" \
    --details="❤️" \
    --security-contact="" \
    --broadcast-mode block \
    --gas="auto" \
    --gas-adjustment="1.2" \
    --gas-prices="0.5uself" \
    --keyring-backend file \
    --keyring-dir $HOME/keys \
    -y
```
### Delege (adres kısmını ve valoper kısmını değiştriniz)
```
selfchaind tx staking delegate valoper-yazınız 28989812716uself\
    --node http://165.232.125.66:26657 \
    --chain-id="self-dev-1" \
    --broadcast-mode block \
    --gas="auto" \
    --gas-adjustment="1.2" \
    --gas-prices="0.5uself" \
    --keyring-backend file \
    --keyring-dir $HOME/keys \
    --from="adres-yazınız"
```
