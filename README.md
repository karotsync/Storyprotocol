Manual Installation
Official Documentation
Recommended Hardware: 6 Cores, 16GB RAM, 400GB of storage (NVME), 100 Mb/s

**install dependencies, if needed**
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**install go, if needed**
```
cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

**set vars**
```
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export STORY_CHAIN_ID="odyssey-0"" >> $HOME/.bash_profile
echo "export STORY_PORT="52"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**download binaries**
```
cd $HOME
wget -O geth https://github.com/piplabs/story-geth/releases/download/v0.10.1/geth-linux-amd64
chmod +x $HOME/geth
mv $HOME/geth ~/go/bin/
[ ! -d "$HOME/.story/story" ] && mkdir -p "$HOME/.story/story"
[ ! -d "$HOME/.story/geth" ] && mkdir -p "$HOME/.story/geth"
```

**install Story**
``
cd $HOME
rm -rf story
git clone https://github.com/piplabs/story
cd story
git checkout v0.13.0
go build -o story ./client 
mv $HOME/story/story $HOME/go/bin/
```

**init story app**
```
story init --moniker test --network odyssey
```

# set seeds and peers
SEEDS="434af9dae402ab9f1c8a8fc15eae2d68b5be3387@story-testnet-seed.itrocket.net:29656"
PEERS="c2a6cc9b3fa468624b2683b54790eb339db45cbf@story-testnet-peer.itrocket.net:26656,5c67635523b37d01f8466a9d8509e75ab0c1208c@152.53.108.188:26656,766f68ac2329a01aa8324750f1947b1499231f3d@149.50.101.203:32656,6a1b35d7c8deae3f6b0588855300af1dfa8ebd17@49.12.172.31:13656,16b98ab14c106c6dbb26d21beb6b77b6e611fe32@135.181.212.253:2010,54a2735f0a96a3a6eec089652c1aa8f899d5769a@195.7.5.165:26656,b8ad3364924728ef0f434102d3f7803fb18c6f90@37.60.236.104:656,7c6a76119716ed6c0c98c8fe13a77e137e6a961a@37.60.246.162:26656,c238dc3f4cc3a298af3c63a5feadc6bb47684a61@100.42.183.110:656,979c7af4049e4bfb592734afb55050ba039cbaa7@141.95.97.1:26656,53a782f8358dfbb8a06e103a2eb949c9f027aca0@144.76.112.58:14656,363b71824f52093037d2b1de69fcb5cebf5d3058@37.60.234.0:656,6e3423d9a8128645d5cad9165e26fa2eac66d150@93.190.138.116:26656,28cd96eed349e843ea015392ca565fd052b7cf9a@84.247.140.178:656,fa294c4091379f84d0fc4a27e6163c956fc08e73@65.108.103.184:26656,45e497fdcdf954567c0ff0f220291cb0221236f0@77.237.234.158:26656,01c3ad391364738fc45ecdb9fc6cd1f7b59f3251@38.242.218.19:26656,5eae68a6accfb78f0f3539d2922b5a68df16263f@65.109.156.123:14656,2806b03d75965227a818a38d59aba1cf62b2f634@37.60.236.178:656,7140598b3ad132261ccbfd1af679284c49035e13@195.26.241.251:26656,17334e0738463a9eac3466147283bf0637208cdb@37.60.234.243:656,ae51a59fdd1addf65bdcb47358da15f684435955@49.12.130.151:26656,8d932e886e4c5c9aac108758df228f01dd1f7ea5@148.72.138.188:26656,bbac56eacc65a4c7db8ae3199d2191c669fe508a@159.69.138.81:56656,75ac7b193e93e928d6c83c273397517cb60603c0@3.142.16.95:26656,8e6fce342f605ffe551cf8d76ca25c51b40a3bc8@141.94.155.97:26656,524cf2b9594cd6cf9d7b419a1cd02b0235fcadd6@84.247.165.71:656,5f3d13f7d0196073ddd229d10369a5e7fed99559@45.159.222.124:26656,89f06acb78296b3abcde19c31a5be31e688a60d0@100.42.183.228:656,86bd5cb6e762f673f1706e5889e039d5406b4b90@195.201.130.235:35616"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml

# download genesis and addrbook
wget -O $HOME/.story/story/config/genesis.json https://server-3.itrocket.net/testnet/story/genesis.json
wget -O $HOME/.story/story/config/addrbook.json  https://server-3.itrocket.net/testnet/story/addrbook.json

# set custom ports in story.toml file
sed -i.bak -e "s%:1317%:${STORY_PORT}317%g;
s%:8551%:${STORY_PORT}551%g" $HOME/.story/story/config/story.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${STORY_PORT}658%g;
s%:26657%:${STORY_PORT}657%g;
s%:26656%:${STORY_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STORY_PORT}656\"%;
s%:26660%:${STORY_PORT}660%g" $HOME/.story/story/config/config.toml

# enable prometheus and disable indexing
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.story/story/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.story/story/config/config.toml

# create geth servie file
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/geth --odyssey --syncmode full --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 0.0.0.0 --http.port ${STORY_PORT}545 --authrpc.port ${STORY_PORT}551 --ws --ws.api eth,web3,net,txpool --ws.addr 0.0.0.0 --ws.port ${STORY_PORT}546
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

# create story service file
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Service
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/.story/story
ExecStart=$(which story) run

Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# download snapshots
# backup priv_validator_state.json
cp $HOME/.story/story/data/priv_validator_state.json $HOME/.story/story/priv_validator_state.json.backup

# remove old data and unpack Story snapshot
rm -rf $HOME/.story/story/data
curl https://server-3.itrocket.net/testnet/story/story_2024-11-27_874712_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/story

# restore priv_validator_state.json
mv $HOME/.story/story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json

# delete geth data and unpack Geth snapshot
rm -rf $HOME/.story/geth/odyssey/geth/chaindata
mkdir -p $HOME/.story/geth/odyssey/geth
curl https://server-3.itrocket.net/testnet/story/geth_story_2024-11-27_874712_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/geth/odyssey/geth

# enable and start geth, story
sudo systemctl daemon-reload
sudo systemctl enable story story-geth
sudo systemctl restart story-geth && sleep 5 && sudo systemctl restart story

# check logs
journalctl -u story -u story-geth -f
Automatic Installation
source <(curl -s https://itrocket.net/api/testnet/story/story-autoinstall/)
Cosmovisor Setup
Install go, if needed:

cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
Install and init Cosmovisor:

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
echo "export DAEMON_NAME="story"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.story/story"" >> $HOME/.bash_profile
source $HOME/.bash_profile
cosmovisor init $(which story)
Create a directory and download the current version of story:

mkdir -p $HOME/.story/story/cosmovisor/upgrades/v0.12.1/bin
wget -O $HOME/.story/story/cosmovisor/upgrades/v0.12.1/bin/story https://github.com/piplabs/story/releases/download/v0.12.1/story-linux-amd64
chmod +x $HOME/.story/story/cosmovisor/upgrades/v0.12.1/bin/story
Update service file:

sudo tee /etc/systemd/system/story.service > /dev/null << EOF
[Unit]
Description=story node service
After=network-online.target

[Service]
User=$USER
Environment="DAEMON_NAME=story"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/data"
ExecStart=$(which cosmovisor) run run
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
Enable and start Story using Cosmovisor:

sudo systemctl daemon-reload
sudo systemctl enable story
sudo systemctl restart story && sudo journalctl -u story -f
Congrats, you are now using Cosmovisor! 🎊
Create validator
View your validator key

story validator export
Export EVM private key

story validator export --export-evm-key
View EVM private key and make a key backup

cat $HOME/.story/story/config/private_key.txt
Use this private key to import your account into a wallet, e.g. Metamask or Phantom. Add the odyssey testnet to your wallet via faucet. Then, copy your 'EVM address' from the wallet and request $IP tokens. Now you can see the balance and make transactions in the wallet app.

Before creating a validator, wait for your node to get fully synced. Once "catching_up" is "false", move on to the next step
curl localhost:$(sed -n '/\[rpc\]/,/laddr/ { /laddr/ {s/.*://; s/".*//; p} }' $HOME/.story/story/config/config.toml)/status | jq
Create validator
story validator create --stake 1500000000000000000000 --moniker $MONIKER --chain-id 1516 --private-key $(cat $HOME/.story/story/config/private_key.txt | grep "PRIVATE_KEY" | awk -F'=' '{print $2}')
Remember to backup your validator priv_key from here:

cat $HOME/.story/story/config/priv_validator_key.json
Firewall rules
Configure firewall rules:

sudo ufw allow 30303/tcp comment geth_p2p_port
sudo ufw allow 26656/tcp comment story_p2p_port
Delete node
sudo systemctl stop story story-geth
sudo systemctl disable story story-geth
rm -rf $HOME/.story
sudo rm /etc/systemd/system/story.service /etc/systemd/system/story-geth.service
sudo systemctl daemon-reload
