# Nibiru-Incentivized

Guide Install Nibiru:

Cấu hình đề xuất
* 4CPU
* 16GB RAM
* 300GB of disk space (SSD)

1/ Bộ cài đặt:

    sudo apt update && sudo apt upgrade -y
    
Cài đặt Package:

    apt install make jq lz4 -y
    
Tải file cài đặt gốc dự án:

    curl -s https://get.nibiru.fi/@v0.19.2! | bash
    
2/ Thêm data cho node:

Thay moniker = tên bạn muốn đặt

    nibid init <moniker> --chain-id=nibiru-itn-1 --home $HOME/.nibid
    
Thêm thông tin Network    
    
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
    
Tải bản snapshot:

    nibid config keyring-backend test
    nibid config chain-id nibiru-itn-1
    nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
    
    SNAP_NAME=$(curl -s https://snapshots2-testnet.nodejumper.io/nibiru-testnet/info.json | jq -r .fileName)
    curl "https://snapshots2-testnet.nodejumper.io/nibiru-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C $HOME/.nibid
    
    
3/ Tạo hệ thống:

    sudo tee /etc/systemd/system/nibid.service > /dev/null << EOF
    [Unit]
    Description=Nibiru Node
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which nibid) start
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=10000
    [Install]
    WantedBy=multi-user.target
    EOF
    
4/ Chạy hệ thống & kiểm tra logs:

    sudo systemctl daemon-reload
    sudo systemctl enable nibid
    sudo systemctl start nibid

    sudo journalctl -u nibid -f --no-hostname -o cat
    
5/ Câu lệnh tạo ví:
    
Tạo ví:

    nibid keys add wallet
    
 Câu lệnh khôi phục ví bằng 24 kí tự: 
 
    nibid keys add wallet --recover

Lưu thông tin Validator:

    cat $HOME/.nibid/config/priv_validator_key.json
    
    
6/ Kiểm tra trạng thái Sync:

    nibid status 2>&1 | jq .SyncInfo.catching_up
    
 Kiểm tra số block đã sync hiện tại:
    
    nibid status 2>&1 | jq .SyncInfo.latest_block_height
    
7/ Tạo validator: thoả mãn 2 điều kiện đã faucet & node đã sync xong. Thay chữ moniker= -> tên bạn muốn đặt:

Trỏ về folder nibi:

    cd 

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
    
 Lệnh unjail:
 
    nibid tx slashing unjail --from wallet --chain-id nibiru-itn-1 --gas-prices 0.1unibi --gas-adjustment 1.5 --gas auto -y 
 
  Chúc bạn thành công, thả tim thả Sao cho mình nhé!
    
