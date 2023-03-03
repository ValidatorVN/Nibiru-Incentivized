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
    
Thay moniker name = tên bạn muốn đặt

    nibid init <moniker-name> --chain-id=nibiru-itn-1 --home $HOME/.nibid
    
2/ Câu lệnh tạo ví:

    nibid keys add wallet
    
 Câu lệnh khôi phục ví bằng 24 kí tự: 
 
    nibid keys add wallet --recover

Lưu thông tin Validator:

    cat $HOME/.nibid/config/priv_validator_key.json
    
3/ Thêm data cho node:

    NETWORK=nibiru-itn-1
    curl -s https://networks.itn.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
    
    curl -s https://rpc.itn-1.nibiru.fi/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
    
    NETWORK=nibiru-itn-1
    sed -i 's|enable =.*|enable = true|g' $HOME/.nibid/config/config.toml
    sed -i 's|rpc_servers =.*|rpc_servers = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/rpc_servers)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's|trust_height =.*|trust_height = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_height)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's|trust_hash =.*|trust_hash = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_hash)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.nibid/config/app.toml
    sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.nibid/config/app.toml
    sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.nibid/config/app.toml
    sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.nibid/config/app.toml
    sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/seeds)'"|g' $HOME/.nibid/config/config.toml
    sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001unibi"|g' $HOME/.nibid/config/app.toml
    sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.nibid/config/config.toml
    
4/ Tạo hệ thống:

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

5/ Tải snapshot:

    apt install lz4 -y
    
    SNAP_NAME=$(curl -s https://snapshots2-testnet.nodejumper.io/nibiru-testnet/info.json | jq -r .fileName)
    curl "https://snapshots2-testnet.nodejumper.io/nibiru-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C $HOME/.nibid
    
6/ Chạy hệ thống & kiểm tra logs:

    sudo systemctl daemon-reload
    sudo systemctl enable nibid
    sudo systemctl start nibid

    sudo journalctl -u nibid -f --no-hostname -o cat
    
7/ Kiểm tra trạng thái Sync:

    nibid status 2>&1 | jq .SyncInfo.catching_up
    
8/ Tạo validator: thoả mãn 2 điều kiện đã faucet & node đã sync xong. Thay chữ Hero -> tên bạn muốn đặt:

Trỏ về folder nibiru:

    cd nibiru

Chạy lệnh tạo validator:

    nibid tx staking create-validator \
    --amount=1000000unibi \
    --pubkey=$(nibid tendermint show-validator) \
    --moniker="Hero-Node & Validator VietNam" \
    --identity=1342DBE69C23B662 \
    --details="https://t.me/NodeValidatorVietNam" \
    --chain-id=nibiru-itn-1 \
    --commission-rate=0.10 \
    --commission-max-rate=0.20 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1 \
    --from=wallet \
    --keyring-backend test
    --gas-prices=0.1unibi \
    --gas-adjustment=1.5 \
    --gas=auto \
    -y
    
  Chúc bạn thành công, thả tim thả Sao cho mình nhé!
    
