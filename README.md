# Symphony Node Setup

Instruksi ini menjelaskan cara mengatur node Symphony pada server Ubuntu.

## Prasyarat

4 Cores, 8G Ram, 160G SSD, Ubuntu 22.04

```bash
sudo apt update
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```

## Instalasi Go

```bash
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.22.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```

## Instalasi Symphony Node

```bash
cd $HOME
rm -rf symphony
git clone https://github.com/Orchestra-Labs/symphony.git
cd symphony
git checkout v0.4.1
make install
symphonyd version
```

## Inisialisasi Node

Ganti `NodeName` dengan moniker Anda sendiri.

```bash
symphonyd init NodeName --chain-id=symphony-testnet-4
```

## Unduh Genesis

```bash
wget -O $HOME/.symphonyd/config/genesis.json https://raw.githubusercontent.com/Orchestra-Labs/symphony/refs/heads/main/networks/symphony-testnet-4/genesis.json
```

## Unduh Addrbook

```bash
wget -O $HOME/.symphonyd/config/addrbook.json https://raw.githubusercontent.com/vinjan23/Testnet.Guide/refs/heads/main/Symphony/addrbook.json
```

## Buat Service

```bash
sudo tee /etc/systemd/system/symphonyd.service > /dev/null <<EOF
[Unit]
Description=symphonyd Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which symphonyd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable symphonyd
```

## Unduh Snapshot (opsional)

```bash
sudo systemctl stop symphonyd
cp $HOME/.symphonyd/data/priv_validator_state.json $HOME/.symphonyd/priv_validator_state.json.backup
rm -rf $HOME/.symphonyd/data
curl -L https://snap-t.vinjan.xyz./symphony/latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.symphonyd
mv $HOME/.symphonyd/priv_validator_state.json.backup $HOME/.symphonyd/data/priv_validator_state.json
sudo systemctl restart symphonyd
sudo journalctl -u symphonyd -f -o cat
```

## Buat Wallet

```bash
symphonyd keys add wallet
```

## Import wallet

```bash
symphonyd keys add wallet --recover
```


## Luncurkan Node

```bash
sudo systemctl restart symphonyd
sudo journalctl -u symphonyd -f -o cat
```

## Check Sync ( If False than go to create validator )

```bash
symphonyd status 2>&1 | jq
```

## Create Validator

```bash
symphonyd tx staking create-validator \
--amount=1000000note \
--moniker="$Your_Validator_Name" \
--identity="" \
--details="" \
--website="" \
--from $WALLET \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.05 \
--min-self-delegation 1 \
--pubkey $(symphonyd tendermint show-validator) \
--chain-id symphony-testnet-4 \
--fees=800note \
-y
```
