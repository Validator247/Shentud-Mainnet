# Shentud-Mainnet

Update your server

    sudo apt update && sudo apt upgrade -y

Install packages

    sudo apt install git curl wget tar lz4 unzip jq build-essential pkg-config clang bsdmainutils make ncdu -y

Install Go

    cd $HOME
    version="1.20.4"
    wget "https://golang.org/dl/go$version.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz"
    rm "go$version.linux-amd64.tar.gz"
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile
    go version

Install App and build

    cd $HOME
    git clone https://github.com/certikfoundation/shentu.git 
    cd shentu 
    git checkout v2.9.0 
    make install

check version

    shentud version

Set-variables  

    SHENTU_MONIKER="Replace_AVIAONE_by_your_name"

Initialize the node

    shentud init $SHENTU_MONIKER --chain-id shentu-2.2

Set minimum gas price

    sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uctk\"|" $HOME/.shentud/config/app.toml

Download genesis

    wget -O $HOME/.shentud/config/genesis.json "https://services.shentu-2.2.shentu.aviaone.com/genesis.json"

Download addrbook

    wget -O $HOME/.shentud/config/addrbook.json "https://services.shentu-2.2.shentu.aviaone.com/addrbook.json"

Add Seeds

    SEEDS="258f523c96efde50d5fe0a9faeea8a3e83be22ca@seed.shentu- 
    2.2.shentu.aviaone.com:10270,bc9bbcae77a09b41417f597965f6fcbb8b280892@52.71.99.85:26656,fd2944af442b18dab4ce50d8e001816a38490d56@54.158.108.97:26656,3edd4e16b791218b623f883d04f8aa5c3ff2cca6@shentu-seed.panthea.eu:36656"
    sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.shentud/config/config.toml

Create service

    sudo tee /etc/systemd/system/shentud.service > /dev/null <<EOF
    [Unit] 
    Description=SHENTU\n 
    After=network.target 
    [Service] 
    Type=simple
    User=$USER
    ExecStart=$(which shentud) start 
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF

next

    sudo systemctl enable shentud 
    sudo systemctl daemon-reload

SNAPSHOT

    sudo systemctl stop shentud 
    cp $HOME/.shentud/data/priv_validator_state.json $HOME/.shentud/priv_validator_state.json.backup
    shentud tendermint unsafe-reset-all --home $HOME/.shentud --keep-addr-book
    wget -c https://services.shentu-2.2.shentu.aviaone.com/shentu-2.2_2024-01-07.tar.gz -O - | tar -xz -C $HOME/.shentud
    mv $HOME/.shentud/priv_validator_state.json.backup $HOME/.shentud/data/priv_validator_state.json
    sudo systemctl start shentud && sudo journalctl -u shentud -f --no-hostname -o cat

Check Syn

    shentud status 2>&1 | jq .SyncInfo

Checklogs

    journalctl -u shentud -f --no-hostname -o cat

Create Wallet

    shentud keys add name_wallet

Recover Wallet

    shentud keys add name_wallet --recover

Create validator

    shentud tx staking create-validator \
     --amount=1000000uctk \
     --pubkey=$(shentud tendermint show-validator) \
     --moniker="YOUR_NICKNAME" \
     --chain-id=shentu-2.2 \
     --commission-rate=0.05 \
     --commission-max-rate=0.2 \
     --commission-max-change-rate=0.02 \
     --min-self-delegation=1 \
     --website="https://your-website.com" \
     --identity="keyBASE_id" \
     --details="This is will be display in the blockchain explorer. Write here something about you"\
     --gas-prices=0.1uctk \
     --gas-adjustment=1.5 \
     --gas=auto \
     --from=WRITE_HERE_YOUR_WALLET_ADDRESS

  Edit Validator

      shentud tx staking edit-validator \
     --chain-id=shentu-2.2 \
     --commission-rate=0.04 \
     --from=name_wallet \
     --gas-prices=0.1uctk \
     --gas-adjustment=1.5 \
     --gas=auto \
     -y


 #Congratulations - DONE        
