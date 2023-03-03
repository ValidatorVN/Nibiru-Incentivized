# Nibiru-Incentivized

Guide Install Nibiru:

Cấu hình đề xuất
* 4CPU
* 16GB RAM
* 300GB of disk space (SSD)

1/ Bộ cài đặt:

    sudo apt update && sudo apt upgrade --yes
    
Tải file cài đặt gốc dự án:

    curl -s https://get.nibiru.fi/@v0.19.2! | bash
    
Thay moniker = tên bạn muốn đặt

    nibid init <moniker> --chain-id=nibiru-itn-1 --home $HOME/.nibid
    
2/ Câu lệnh tạo ví:

    nibid keys add wallet
    
 Câu lệnh khôi phục ví bằng 24 kí tự: 
 
    nibid keys add wallet --recover

Lưu thông tin Validator:

    cat $HOME/.nibid/config/priv_validator_key.json
    
3/ Thêm data cho node:

    NETWORK=nibiru-itn-1
    curl -s https://networks.itn.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
    
    NETWORK=nibiru-itn-1
    sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/seeds)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
    
    NETWORK=nibiru-itn-1
    sed -i 's|enable =.*|enable = true|g' $HOME/.nibid/config/config.toml
    sed -i 's|rpc_servers =.*|rpc_servers = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/rpc_servers)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's|trust_height =.*|trust_height = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_height)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's|trust_hash =.*|trust_hash = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_hash)'"|g' $HOME/.nibid/config/config.toml

4/ Cài đặt Cosmosvisor:

    apt install golang-go -y
    apt install make -y
    
    
Cài đặt cosmosvisor bản mới nhất:

    git clone https://github.com/cosmos/cosmos-sdk
    cd cosmos-sdk
    git checkout v0.42.7
    make cosmovisor
    cd $HOME
    
    export DAEMON_NAME=nibid
    export DAEMON_HOME=$HOME/.nibid
    mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
    mkdir -p $DAEMON_HOME/cosmovisor/upgrades
    cp $(which nibid) $DAEMON_HOME/cosmovisor/genesis/bin
    cd $HOME
    
4/ Tạo hệ thống:

    sudo tee /etc/systemd/system/cosmovisor-nibiru.service<<EOF
    [Unit]
    Description=Cosmovisor for Nibiru Node
    Requires=network-online.target
    After=network-online.target

    [Service]
    Type=exec
    User=root
    Group=root
    ExecStart=/home/root/go/bin/cosmovisor run start --home /home/root/.nibid
    Restart=on-failure
    RestartSec=3
    Environment="DAEMON_NAME=nibid"
    Environment="DAEMON_HOME=/home/root/.nibid"
    Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
    Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
    Environment="DAEMON_LOG_BUFFER_SIZE=512"
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF
    
5/ Chạy hệ thống & kiểm tra logs:

    sudo systemctl daemon-reload
    sudo systemctl enable nibid
    sudo systemctl start nibid

    sudo journalctl -u nibid -f --no-hostname -o cat
    
6/ Kiểm tra trạng thái Sync:

    nibid status 2>&1 | jq .SyncInfo.catching_up
    
7/ Tạo validator: thoả mãn 2 điều kiện đã faucet & node đã sync xong. Thay chữ Hero -> tên bạn muốn đặt:

Trỏ về folder nibi:

    cd .nibid

Chạy lệnh tạo validator:

    nibid tx staking create-validator \
    --amount=1000000unibi \
    --pubkey=$(nibid tendermint show-validator) \
    --moniker="Node & Validator VietNam" \
    --identity=1342DBE69C23B662 \
    --details="https://t.me/NodeValidatorVietNam" \
    --chain-id=nibiru-itn-1 \
    --commission-rate=0.10 \
    --commission-max-rate=0.20 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1 \
    --from=wallet \
    --keyring-backend test \
    --gas-prices=0.1unibi \
    --gas-adjustment=1.5 \
    --gas=auto \
    -y
    
  Chúc bạn thành công, thả tim thả Sao cho mình nhé!
    
