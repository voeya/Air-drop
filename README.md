
-------------------------------------------------
cd $HOME
ver="1.18.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile

---------------------------------------------------
go version

--------------------------------------
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app
git checkout v0.6.0
make install

-------------------------------------------------
celestia-appd version

-------------------------------------------
git clone https://github.com/celestiaorg/networks

-------------------------------
cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git

--------------------------
CELESTIA_NODENAME="MY_NODE" 
CELESTIA_WALLET="MY_WALLET"
CELESTIA_CHAIN="mamaki"

ط§ط·ظ„ط§ط¹ط§طھ  ط¯ط§ط®ظ„ "" ط±ظˆ ظ¾ط§ع© ع©ظ†غŒط¯(ظپظ‚ط· ط®ط· ط§ظˆظ„ ظˆ ط¯ظˆظ…) ظˆ ط§ط·ظ„ط§ط¹ط§طھ ط®ظˆط¯طھظˆظ† ط±ظˆ ظ‚ط±ط§ط± ط¨ط¯غŒط¯
ط¨ظ‡ ط¹ظ†ظˆط§ظ† ظ…ط«ط§ظ„ :

CELESTIA_NODENAME="amir" 
CELESTIA_WALLET="amir8372"
CELESTIA_CHAIN="mamaki"

----------------------------------------------------------
echo 'export CELESTIA_CHAIN='$CELESTIA_CHAIN >> $HOME/.bash_profile
echo 'export CELESTIA_NODENAME='${CELESTIA_NODENAME} >> $HOME/.bash_profile
echo 'export CELESTIA_WALLET='${CELESTIA_WALLET} >> $HOME/.bash_profile
source $HOME/.bash_profile

------------------------------------------------
celestia-appd init $CELESTIA_NODENAME --chain-id $CELESTIA_CHAIN

-------------------------------------
cp $HOME/networks/mamaki/genesis.json $HOME/.celestia-app/config/

---------------------------------------
sed -i 's/mode = \"full\"/mode = \"validator\"/g' $HOME/.celestia-app/config/config.toml

---------------------------------------------
BOOTSTRAP_PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/bootstrap-peers.txt | tr -d '\n')

--------------------------------------------------
echo $BOOTSTRAP_PEERS

------------------------------------------------
sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$BOOTSTRAP_PEERS\"/" $HOME/.celestia-app/config/config.toml

-----------------------------------------------------------
sed -i 's/timeout-commit = ".*/timeout-commit = "25s"/g' $HOME/.celestia-app/config/config.toml
sed -i 's/peer-gossip-sleep-duration *=.*/peer-gossip-sleep-duration = "2ms"/g' $HOME/.celestia-app/config/config.toml

-----------------------------------------------------------------
max_num_inbound_peers=40 
max_num_outbound_peers=10 
max_connections=50

----------------------------------------------------------------
sed -i -e "s/^use-legacy *=.*/use-legacy = false/;\
s/^max-num-inbound-peers *=.*/max-num-inbound-peers = $max_num_inbound_peers/;\
s/^max-num-outbound-peers *=.*/max-num-outbound-peers = $max_num_outbound_peers/;\
s/^max-connections *=.*/max-connections = $max_connections/" $HOME/.celestia-app/config/config.toml

------------------------------------------------
pruning_keep_recent="100" 
pruning_interval="10"

------------------------------------------------------

sed -i -e "s/^pruning *=.*/pruningآ = \"custom\"/;\
s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/;\
s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml

--------------------------------------------

sed -i 's/snapshot-interval *=.*/snapshot-interval = 0/' $HOME/.celestia-app/config/app.toml

---------------------------------------------------
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app

---------------------------------------------------

celestia-appd config chain-id $CELESTIA_CHAIN
celestia-appd config keyring-backend test

-------------------------------------------------------
tee $HOME/celestia-appd.service > /dev/null <<EOF
[Unit]
  Description=celestia-appd Cosmos daemon
  After=network-online.target
[Service]
  User=$USER
  ExecStart=$(which celestia-appd) start
  Restart=on-failure
  RestartSec=3
  LimitNOFILE=65535
[Install]
  WantedBy=multi-user.target
EOF

---------------------------------------------------
sudo mv $HOME/celestia-appd.service /etc/systemd/system/

----------------------------------------------------
sudo systemctl enable celestia-appd
sudo systemctl daemon-reload
sudo systemctl restart celestia-appd && journalctl -u celestia-appd -f -o cat

------------------------------------------------------------
Press CTRL+C to stop the logs

-------------------------------------------------------------

sudo systemctl stop celestia-appd

--------------------------------------------------------------

cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/آ | \
    egrep -o ">mamaki.*tar" | tr -d ">")
wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME}آ | tar xf - \
    -C ~/.celestia-app/data/

------------------------------------------------------------------
ظ…ظ†طھط¸ط± ط¨ظ…ط§ظ†غŒط¯ طھط§ ع¯ط±ظ‡ ط´ظ…ط§ ط§ع©ط«ط± ط¨ظ„ظˆع© ظ‡ط§ ط±ط§ ط§ط² ط²ظ†ط¬غŒط±ظ‡ ط¨ظ„ظˆع©غŒ Celestia ط¯ط§ظ†ظ„ظˆط¯ ع©ظ†ط¯

--------------------------------------------------
sudo systemctl restart celestia-appd && journalctl -u celestia-appd -f -o cat

-------------------------------------------------
CTRL+C ط±ط§ ظپط´ط§ط± ط¯ظ‡غŒط¯ طھط§ ظ„ط§ع¯ ظ‡ط§ ظ…طھظˆظ‚ظپ ط´ظˆظ†ط¯

---------------------------------
ظ‚ط¨ظ„ ط§ط² ط§ط¯ط§ظ…ظ‡طŒ ط¨ط§ ع©ط¯ ط²غŒط± ظ…ط·ظ…ط¦ظ† ط´ظˆغŒط¯ ع©ظ‡ ط¨ظ‡ ط·ظˆط± ع©ط§ظ…ظ„ ط¨ط§ ط´ط¨ع©ظ‡ ظ‡ظ…ع¯ط§ظ… ط´ط¯ظ‡ ط§غŒط¯ 

curl -s localhost:26657/status | grep block_height
------------------------------------------------
ط¨ط§غŒط¯ ظ…ظ†طھط¸ط± ط¨ظ…ط§ظ†غŒط¯ طھط§ ع¯ط±ظ‡ Validator ط´ظ…ط§ ظ‡ظ…ظ‡ ط¨ظ„ظˆع©â€Œظ‡ط§ ط±ط§ ط§ط² ط²ظ†ط¬غŒط±ظ‡ ط¨ظ„ظˆع©غŒ ط³ظ„ط³طھغŒط§ ط¯ط§ظ†ظ„ظˆط¯ ع©ظ†ط¯

---------------------------------------------------

celestia-appd keys add $CELESTIA_WALLET

ط§ط·ظ„ط§ط¹ط§طھ ط¯ط±غŒط§ظپطھغŒ ط§غŒظ† ط¨ط®ط´ ط±ظˆ طھظˆغŒ ظ†ظˆطھ ظ¾ط¯ ط¨ط±ط§غŒ ط®ظˆط¯طھظˆظ† ط°ط®غŒط±ظ‡ ع©ظ†غŒط¯

-----------------------------------
CELESTIA_ADDR=$(celestia-appd keys show $CELESTIA_WALLET -a) 
echo $CELESTIA_ADDR 
echo 'export CELESTIA_ADDR='${CELESTIA_ADDR} >> $HOME/.bash_prof

------------------------------------------------
CELESTIA_VALOPER=$(celestia-appd keys show $CELESTIA_WALLET --bech val -a) 
echo $CELESTIA_VALOPER 
echo 'export CELESTIA_VALOPER='${CELESTIA_VALOPER} >> $HOME/.bash_profile 
source $HOME/.bash_profile

--------------------------------------------
ط¨ط§ ظ„غŒظ†ع© https://discord.gg/celestiacommunityآ ظˆط§ط±ط¯ ط¯غŒط³ع©ظˆط±ط¯ ظ¾ط±ظˆعکظ‡ ط¨ط´غŒط¯ ظˆ ط¨ظ‡ ط¨ط®ط´ mamaki faucet  ط¨ط±غŒط¯ ظˆ ط¨ظ‡ ط´ع©ظ„ ط²غŒط± ط§ط¯ط±ط³ ط®ظˆط¯طھظˆظ† ط±ظˆ ظˆط§ط±ط¯ ع©ظ†غŒط¯ ظˆ ط¯ط±ط®ظˆط§ط³طھ ظپط§ط³طھ ط´ط¨ع©ظ‡ ط±ظˆ ط¨ط¯غŒط¯
$request celestia1a35035fu83jkeeqz00d3jmt3k5wu5x3lyvn6qp

------------------------------------- 
ط¨ط§ ع©ط¯ ط²غŒط± ظ‡ظ… ظ…غŒطھظˆظ†غŒط¯ ظ…ظˆط¬ظˆط¯غŒ ظˆظ„طھ ط®ظˆط¯طھظˆظ† ط±ظˆ ط¨ط¨ظ†غŒط¯  ع©ظ‡ ط¨ظ‡ ط¬ط§غŒ ط§غŒع©ط³ ظ‡ط§ ط§ط¯ط±ط³ ط®ظˆط¯طھظˆظ† ط±ظˆ ظˆط§ط±ط¯ ع©ظ†غŒط¯ 

celestia-appd q bank balances xxxxxxxxxxxxx
