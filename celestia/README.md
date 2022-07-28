# Celestia Specific Notes

## Install & Initialize Celestia App (from Control Machine)

```
git clone https://github.com/Throbac-Labs/cosmos-validators.git
cd cosmos-validators
# edit inventory.sample

cp inventory.sample inventory
ansible-playbook main.yml -e "target=celestia_main"
```

## Test Celestia App (from Target machine)

```
sudo tail -f /var/log/syslog
sudo journalctl -fu celestia.service
curl -s localhost:16657/status | jq .result | jq .sync_info
curl -s localhost:16657/status
# may need to run sudo systemctl restart systemd-journald.service
```

## Create Wallet & Delegate Stake

```
celestia-appd config keyring-backend test
celestia-appd keys add validator
celestia-appd keys list
```
Go to discord #faucet
$request celestia1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Check Balance & Delegate
```
celestia-appd q bank balances celestia1mr80fa6q7pewnpw7f6gq55khnmndrqw604zppa --node http://localhost:16657
export VALIDATOR_WALLET="your-wallet-address"
celestia-appd keys show $VALIDATOR_WALLET --bech val -a
# delegate to your validator
celestia-appd tx staking delegate \ 
    celestiavaloper1mr80fa6q7pewnpw7f6gq55khnmndrqw622qchm 1000000utia \ 
    --from=$VALIDATOR_WALLET --chain-id=mamaki --node http://localhost:16657
```

# Deploy Celestia Bridge Node

Multi step process

## Install Celestia Node
```
cd $HOME 
rm -rf celestia-node 
git clone https://github.com/celestiaorg/celestia-node.git 
cd celestia-node/ 
git checkout tags/v0.3.0-rc2 
make install
```
## Initialize Bridge Node

Once your celestia-app is synced
```
celestia bridge init --core.remote http://localhost:16657 --core.grpc http://localhost:9090
```

## Start Bridge Node

Create Celestia Bridge systemd file:
```
sudo nano /etc/systemd/system/celestia-bridge.service
[Unit]
Description=celestia-bridge Cosmos daemon
After=network-online.target

[Service]
User=ubuntu 
ExecStart=/home/ubuntu/go/bin/celestia bridge start 
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
If the file was created successfully you will be able to see its content:
```
cat /etc/systemd/system/celestia-bridge.service
```
Enable and start celestia-bridge daemon:
```
sudo systemctl enable celestia-bridge 
sudo systemctl start celestia-bridge
sudo journalctl -u celestia-bridge.service -f
```
## Connect Validator
```
export MONIKER="your_moniker"
export VALIDATOR_WALLET="validator"
```
```
celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=$MONIKER \
    --chain-id=mamaki \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=$VALIDATOR_WALLET \
    --keyring-backend=test
```

#### Potential Tweaks

config.toml edits
```
max-connections = 90
use-legacy = false
```


















