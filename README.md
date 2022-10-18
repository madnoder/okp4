# okp4
```
NODENAME=<MONIKER HERE>
```
```
OKP4_PORT=60
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export OKP4_CHAIN_ID=okp4-nemeton" >> $HOME/.bash_profile
echo "export OKP4_PORT=${OKP4_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```
```
cd $HOME
git clone https://github.com/okp4/okp4d.git
cd okp4d
make install
```
```
okp4d config chain-id $OKP4_CHAIN_ID
okp4d config keyring-backend test
okp4d config node tcp://localhost:${OKP4_PORT}657
```
```
okp4d init $NODENAME --chain-id $OKP4_CHAIN_ID
```
```
wget -qO $HOME/.okp4d/config/genesis.json "https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton/genesis.json"
```
```
PEERS=f595a1386d5ca2e0d2cd81d3c6372c3bf84bbd16@65.109.31.114:2280,a49302f8999e5a953ebae431c4dde93479e17155@162.19.71.91:26656,dc14197ed45e84ca3afb5428eb04ea3097894d69@88.99.143.105:26656,79d179ea2e1fbdcc0c59a95ab7f1a0c48438a693@65.108.106.131:26706,501ad80236a5ac0d37aafa934c6ec69554ce7205@89.149.218.20:26656,5fbddca54548bf125ee96bb388610fe1206f087f@51.159.66.123:26656,769f74d3bb149216d0ab771d7767bd39585bc027@185.196.21.99:26656,024a57c0bb6d868186b6f627773bf427ec441ab5@65.108.2.41:36656,fff0a8c202befd9459ff93783a0e7756da305fe3@38.242.150.63:16656,2bfd405e8f0f176428e2127f98b5ec53164ae1f0@142.132.149.118:26656,bf5802cfd8688e84ac9a8358a090e99b5b769047@135.181.176.109:53656,dc9a10f2589dd9cb37918ba561e6280a3ba81b76@54.244.24.231:26656,085cf43f463fe477e6198da0108b0ab08c70c8ab@65.108.75.237:6040,803422dc38606dd62017d433e4cbbd65edd6089d@51.15.143.254:26656,b8330b2cb0b6d6d8751341753386afce9472bac7@89.163.208.12:26656

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.okp4d/config/config.toml
```
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OKP4_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${OKP4_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OKP4_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OKP4_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OKP4_PORT}660\"%" $HOME/.okp4d/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OKP4_PORT}317\"%; s%^address = \":8080\"%address = \":${OKP4_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OKP4_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OKP4_PORT}091\"%" $HOME/.okp4d/config/app.toml
```
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.okp4d/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.okp4d/config/app.toml
```
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uknow\"/" $HOME/.okp4d/config/app.toml
```
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.okp4d/config/config.toml
```
```
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d
```
```
sudo tee /etc/systemd/system/okp4d.service > /dev/null <<EOF
[Unit]
Description=okp4d
After=network-online.target

[Service]
User=$USER
ExecStart=$(which okp4d) start --home $HOME/.okp4d
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable okp4d
sudo systemctl restart okp4d && sudo journalctl -u okp4d -f -o cat
```
```
source $HOME/.bash_profile
```
```
okp4d status 2>&1 | jq .SyncInfo
```
```
okp4d keys add $WALLET
```
```
okp4d keys list
```
```
OKP4_WALLET_ADDRESS=$(okp4d keys show $WALLET -a)
OKP4_VALOPER_ADDRESS=$(okp4d keys show $WALLET --bech val -a)
echo 'export OKP4_WALLET_ADDRESS='${OKP4_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export OKP4_VALOPER_ADDRESS='${OKP4_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### faucet
### https://faucet.okp4.network/
```
okp4d query bank balances $OKP4_WALLET_ADDRESS
```
```
okp4d tx staking create-validator \
  --amount 1000000uknow \
  --from $WALLET \
  --commission-max-change-rate "0.10" \
  --commission-max-rate "0.40" \
  --commission-rate "0.17" \
  --min-self-delegation "1" \
  --pubkey  $(okp4d tendermint show-validator) \
  --moniker $NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id $OKP4_CHAIN_ID
```
```
sudo journalctl -u okp4d -f -o cat
```
```
okp4d tx distribution withdraw-rewards $OKP4_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$OKP4_CHAIN_ID
```
```
okp4d tx staking delegate $OKP4_VALOPER_ADDRESS 1000000uknow --from=$WALLET --chain-id=$OKP4_CHAIN_ID
```
