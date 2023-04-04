<h1 align="center"> THIS IS TUTORIAL FOR GETH - LIGHTHOUSE </h1>



## Installing Prerequisites :

 1. #### Library
 ```
sudo apt update && sudo apt upgrade -y && sudo apt install micro curl tar wget screen clang pkg-config libssl-dev jq unzip build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool software-properties-common -y
```
2. #### Install Geth
```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt update -y
sudo apt install ethereum -y
```
 3. #### Install lighthouse 
 
```
wget https://github.com/sigp/lighthouse/releases/download/v4.0.1/lighthouse-v4.0.1-x86_64-unknown-linux-gnu-portable.tar.gz  
```
```
tar -xzf lighthouse-v4.0.1-x86_64-unknown-linux-gnu-portable.tar.gz
```
```
rm lighthouse-v4.0.1-x86_64-unknown-linux-gnu-portable.tar.gz
mv lighthouse /usr/local/bin
```

## Install Go :
```
ver="1.19" 
cd $HOME  
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
rm "go$ver.linux-amd64.tar.gz"  
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile 
source ~/.bash_profile  
go install github.com/protolambda/eth2-testnet-genesis@latest
go install github.com/protolambda/eth2-val-tools@latest
```
## Initial Setup

 - ####   Create Folder
 ```
cd ~ 
mkdir testnet  
mkdir testnet/data  
mkdir testnet/beacon-1  
mkdir testnet/beacon-2 
mkdir testnet/logs  
cd testnet
```
 -  ####  Download Repository
```
wget https://github.com/gitshock-labs/testnet-list/releases/download/Iteration-70.a/cartenz-iteration-70.a.zip
```
```
unzip cartenz-iteration-70.a.zip
```
```
rm -rf cartenz-iteration-70.a.zip
```

 -  ####  Create Jwt Secret
 ```
 openssl rand -hex 32 | tr -d "\n" > "jwt.hex"
```
 -  ####  Create Geth Account
 ```
 geth account new --datadir "data"
```
 - ####   Write custom genesis block
 ```
 geth --datadir $HOME/testnet/data init $HOME/testnet/execution/genesis.json
```
 

## Running Execution Layer

 -  #### It is recommended to use Screen for running the Execution Layer and Consensus Layer.

```
screen -S cartenz
```
 -  #### Please make sure that ports are open before you start 

```
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 8545/tcp
sudo ufw allow 30303/tcp
sudo ufw allow 5052/tcp 
sudo ufw allow 5053/tcp 
sudo ufw allow 9000/tcp 
sudo ufw allow 30303/udp
```
 -  #### Please edit identity for example "tommy"
```
 nohup geth \ 
 --datadir "$HOME/testnet/data" \ 
 --http --http.api="engine,eth,web3,net,admin" \ 
 --ws --ws.api="engine,eth,web3,net" \ 
 --http.port 8545 \ 
 --http.addr 0.0.0.0 \ 
 --http.corsdomain "*" \ 
 --identity "PUT-YOUR-NAME" \ 
 --networkid=1881 \ 
 --syncmode=full \ 
 --authrpc.jwtsecret="$HOME/testnet/jwt.hex" \ 
 --authrpc.port 8551 \ 
 --bootnodes "enode://0e2b41699b95e8c915f4f5d18962c0d2db35dc22d3abbebbd25fc48221d1039943240ad37a6e9d853c0b4ea45da7b6b5203a7127b5858c946fc040cace8d2d63@147.75.71.217:30303,enode://45b4fff6ab970e1e490deea8a5f960d806522fafdb33c8eaa38bc0ae970efc2256fc5746f0ecfec770af24c44864a3e6772a64f2e9f031f96fd4af7fd0483110@147.75.71.217:30304" \ 
 > $HOME/testnet/logs/geth_1.log &
```
 - #### **output : nohup: ignoring input and redirecting stderr to stdout ;** 
**--Then press ctrl+c**

## Running Consensus Layer :

 - ####  Please edit suggested-fee-recipient and graffiti for example "0xB0F9575xC688Ed5C67f50B3aF0d195664c2EB123" and "TOMMY"

```
nohup lighthouse beacon \
--eth1 \
--http \
--testnet-dir $HOME/testnet/consensus \
--datadir "$HOME/testnet/beacon-1" \
--http-allow-sync-stalled \
--execution-endpoints http://127.0.0.1:8551 \
--http-port=5052 \
--enr-udp-port=9000 \
--enr-tcp-port=9000 \
--discovery-port=9000 \
--graffiti "PUT-YOUR-NAME" \
--execution-jwt "$HOME/testnet/jwt.hex" \
--suggested-fee-recipient="METAMASK-ADDRESS" \
> $HOME/testnet/logs/beacon_1.log &
```
 - #### **output : nohup: ignoring input and redirecting stderr to stdout ;** 
**--Then press ctrl+c**

## Getting ENR Key :

 - #### SAVE YOUR ENR KEY!

```
curl http://localhost:5052/eth/v1/node/identity | jq .data.enr
```

## Running Other Consensus Layer :

 - #### Please edit only suggested-fee-recipient , code block do the rest , dont worry

```
nohup lighthouse \ 
--testnet-dir $HOME/testnet/consensus \ 
bn \ 
--datadir $HOME/testnet/beacon-2 \ 
--eth1 \ 
--http \ 
--http-allow-sync-stalled \ 
--execution-endpoints "http://127.0.0.1:8551" \ 
--eth1-endpoints "http://127.0.0.1:8545" \ 
--http-address 0.0.0.0` \ `--http-port 5053 \ 
--http-allow-origin="*" \ 
--listen-address 0.0.0.0 \
--enr-udp-port 9001 \ 
--enr-tcp-port 9001 \ 
--port 9001 \ 
--enr-address $(curl -s ifconfig.me) \
--execution-jwt "$HOME/testnet/jwt.hex" \ 
--suggested-fee-recipient="METAMASK-ADDRESS" \ 
--boot-nodes="$(curl -s http://localhost:5052/eth/v1/node/identity | jq -r '.data | .enr'),enr:-MS4QHXShZPtKwtexK2p9yCxMxDwQ-EvdH_VemoxyVyweuaBLOC_8cmOzyx7Gy-q6-X8KGT1d_rhAn_ekXnhpCkA_REHh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBMfxReAmd2k___________gmlkgnY0gmlwhJNLR9mJc2VjcDI1NmsxoQJB10N42nK6rr7Q_NIJNkJFi2uo6itMTOQlPZDcCy09T4hzeW5jbmV0c4gAAAAAAAAAAIN0Y3CCIyiDdWRwgiMo,enr:-MS4QEw_RpORuoXgJ0279QuVLLFAiXevNdYtU7vR8S1CY7X9CS6tceMbaxdIIJYRmHN43ClqHtE2b0H0maSb18cm9D0Hh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBMfxReAmd2k___________gmlkgnY0gmlwhJNLR9mJc2VjcDI1NmsxoQOkQIyCVHLbLjIFMjqNSJEUsbYMe4Tsv9blUWvN6Rsft4hzeW5jbmV0c4gAAAAAAAAAAIN0Y3CCIymDdWRwgiMp,enr:-LS4QExQqM_G3y2CfedjrGEbapN5Vprdy7Iq2gzfylwLW8PQf4Tf82XnQxLg9PbH8QLwsMaoWwYjTo7xHQ4oy4eCn7kBh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBMfxReAmd2k___________gmlkgnY0iXNlY3AyNTZrMaEDec2pARmw1GLJHiXIDaG-6J74gZ1SyDcF_CuVUzRsmX2Ic3luY25ldHMAg3RjcIIjKoN1ZHCCIyo" \
> $HOME/testnet/logs/beacon_2.log &
```
 - #### **output : nohup: ignoring input and redirecting stderr to stdout ;** 
**--Then press ctrl+c**

## Check Logs :

```
tail -f logs/geth_1.log
```
```
tail -f logs/beacon_1.log
```
```
tail -f logs/beacon_2.log
```

 ## Staking Part :
 ```
 git clone https://github.com/gitshock-labs/staking-cli.git
cd staking-cli
git checkout main
sudo apt install python3-pip -y
pip3 install -r requirements.txt
sudo python3 setup.py install
./deposit.sh install
```
```
./deposit.sh new-mnemonic
```

 - #### It will be announced by the team in the upcoming days , waiting for next update
 
 
 ## Delete the whole system !! DELETES EVERYTHING :
```
killall geth
killall lighthouse 
cd ~
rm -rf testnet 
 
```
