# Elys

### Elys node Installation Instructions.

[Official documentation](https://elys-network.gitbook.io/)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```


### Download and build binaries
```
cd && rm -rf elys
git clone https://github.com/elys-network/elys
cd elys
git checkout v0.32.0
ROCKSDB=1 LD_LIBRARY_PATH=/usr/local/lib make install
```

# Config and init app
```
elysd config chain-id elystestnet-1
elysd config keyring-backend test
elysd config node tcp://localhost:22057
elysd init "your moniker" --chain-id elystestnet-1
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/elys-testnet/genesis.json > $HOME/.elys/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/elys-testnet/addrbook.json > $HOME/.elys/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "cdf9ae8529aa00e6e6703b28f3dcfdd37e07b27c@37.187.154.66:26656,86987eeff225699e67a6543de3622b8a986cce28@91.183.62.162:26656,ae22b82b1dc34fa0b1a64854168692310f562136@198.27.74.140:26656,61284a4d71cd3a33771640b42f40b2afda389a1e@5.101.138.254:26656,ae7191b2b922c6a59456588c3a262df518b0d130@elys-testnet-seed.itrocket.net:54656,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,609c64cc50fb4ebbe7cae3347545d3950ea2c018@65.108.195.29:23656,0977dd5475e303c99b66eaacab53c8cc28e49b05@elys-testnet-peer.itrocket.net:38656"|' $HOME/.elys/config/config.toml
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uelys,0.01ibc/2180E84E20F5679FCC760D8C165B60F42065DEF7F46A72B447CFF1B7DC6C0A65,0.01ibc/E2D2F6ADCC68AA3384B2F5DFACCA437923D137C14E86FB8A10207CF3BED0C8D4"|' $HOME/.elys/config/app.toml
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.elys/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/elysd.service > /dev/null << EOF
[Unit]
Description=Elys Network node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which elysd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable elysd.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/elys-testnet/elys-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.elys"
```

# enable and start service
```
sudo systemctl start elysd.service
sudo journalctl -u elysd.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
elysd keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
elysd keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
evmosd status 2>&1 | jq .SyncInfo
```

### We receive tokens from the tap in the [discord](https://discord.gg/elysnetwork)
```
The faucet is not working yet, so we are waiting or asking for tokens in the chat.
```

# before creating a validator, you need to fund your wallet and check balance
```
elysd q bank balances $(elysd keys show wallet -a) 
```
# Create validator
```
elysd tx staking create-validator \
--amount=1000000uelys \
--pubkey=$(elysd tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=elystestnet-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.01uelys \
--gas-adjustment=1.5 \
--gas=auto \
-y
```

### Update
```
No update

Current network:elystestnet-1
Current version:v0.32.0
```

### Useful commands

Check balance
```
elysd q bank balances $(elysd keys show wallet -a) 
```

CHECK SERVICE LOGS
```
sudo journalctl -u elysd -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart elysd
```

GET VALIDATOR INFO
```
elysd status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
elysd tx staking delegate $(elysd keys show wallet --bech val -a) 1000000uelys --from wallet --chain-id elystestnet-1 --gas-prices 0.01uelys --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop elysd && sudo systemctl disable elysd && sudo rm /etc/systemd/system/elysd.service && sudo systemctl daemon-reload && rm -rf $HOME/.elys && rm -rf elys && sudo rm -rf $(which elysd) 
```
