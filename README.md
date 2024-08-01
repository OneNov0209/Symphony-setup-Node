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
git checkout v0.2.1
make install
symphonyd version
```

## Inisialisasi Node

Ganti `NodeName` dengan moniker Anda sendiri.

```bash
symphonyd init NodeName --chain-id=symphony-testnet-2
```

## Unduh Genesis

```bash
wget -O $HOME/.symphonyd/config/genesis.json https://raw.githubusercontent.com/Orchestra-Labs/symphony/main/networks/symphony-testnet-2/genesis.json
```

## Unduh Addrbook

```bash
wget -O $HOME/.symphonyd/config/addrbook.json https://raw.githubusercontent.com/vinjan23/Testnet.Guide/main/Symphony/addrbook.json
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
SNAP_NAME=$(curl -s https://ss-t.symphony.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss-t.symphony.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.symphonyd
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
