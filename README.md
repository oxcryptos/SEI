# Sei  

### Cистемные требования:
Оффициальные: 4 CPU 32GB RAM 1TB SSD

Тестированые: 4 CPU 8GB RAM 200GB SSD

---
### Установка и обновление до 1.0.3beta (10.06.22)

### [Обновление до 1.0.4beta (22.06.22)](https://github.com/0xCryptos/SEI/blob/main/Upd%201.0.3b%20-%3E%201.0.4b.md) 
### [Обновление до 1.0.5beta (30.06.22)](https://github.com/0xCryptos/SEI/blob/main/Upd%201.0.4b%20-%3E%201.0.5b.md)




## Установка

**Название вашего валидатора и кошелька:**

меняем на свое

    NODENAME=НАЗВАНИЕ_ВАЛИДАТОРА
    WALLET=НАЗВАНИЕ_КОШЕЛЬКА

**Задаем переменные**:

    echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
    echo "export WALLET=$WALLET" >> $HOME/.bash_profile
    echo "export CHAIN_ID=sei-testnet-2" >> $HOME/.bash_profile
    source $HOME/.bash_profile

**Обновляем все что есть**

    sudo apt update && sudo apt upgrade -y

**Ставим необходимые зависимости**

    sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y

**Ставим GO**

    ver="1.18.1"
    cd $HOME
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
    rm "go$ver.linux-amd64.tar.gz"
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
    source ~/.bash_profile
    go version

**Компилим бинарник**

    cd $HOME
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
    git checkout 1.0.2beta
    make install
    
    
    seid version --long | head
    # version: 1.0.2beta 
    # commit: f556c64de7b9056d280f65f742f826b0c656a521

**Добавляем значения в конфиг**

    seid config chain-id $CHAIN_ID
    seid config keyring-backend file

**Инициализируем**

    seid init $NODENAME --chain-id $CHAIN_ID

**Качаем genesis и addrbook**

    wget -O $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/genesis.json"
	wget -O $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/addrbook.json"
**Проверим генезис**

	sha256sum ~/.sei/config/genesis.json
	# aec481191276a4c5ada2c3b86ac6c8aad0cea5c4aa6440314470a2217520e2cc
**Удостоверяемся что состояние валидатора на 0**

	cd && cat .sei/data/priv_validator_state.json

> {
>  
>  "height": "0",  
> "round": 0,  
> "step": 0  
>  }

Если нет, то вводим 

	seid tendermint unsafe-reset-all --home $HOME/.sei
	wget -O $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/addrbook.json"
**Cтавим минимальный газ**

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025usei\"/;" ~/.sei/config/app.toml

**Ставим сиды и пиры**

    external_address=$(wget -qO- eth0.me)
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sei/config/config.toml 
	peers=""
	sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sei/config/config.toml
	seeds=""
	sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sei/config/config.toml

**Настраиваем прунинг**

    pruning="custom"
    pruning_keep_recent="100"
    pruning_keep_every="0"
    pruning_interval="10"
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml

**Создаем сервис**

    sudo tee /etc/systemd/system/seid.service > /dev/null <<EOF
    [Unit] 
    Description=seid 
    After=network-online.target 

	[Service] 
	User=$USER 
	ExecStart=$(which seid) start 
	Restart=on-failure 
	RestartSec=3 
	LimitNOFILE=65535 

	[Install] 
	WantedBy=multi-user.target 
	EOF

**Запускаем ноду**

    sudo systemctl daemon-reload
    sudo systemctl enable seid
    sudo systemctl restart seid
    source $HOME/.bash_profile

**Посмотреть логи**

    journalctl -u seid -f -o cat
**и синхронизацию**

    seid status 2>&1 | jq .SyncInfo
---
***Пока ждем синхронизации разбираемся с кошельками***
### Операции с кошельками

**Создаем кошелек**

    seid keys add $WALLET
**Или восстанавливаем из сид фразы**

    seid keys add $WALLET --recover
**Добавляем наш адрес в переменные**

    WALLET_ADDRESS=$(seid keys show $WALLET -a)
**И VALOPER адрес тоже**
    
    VALOPER_ADDRESS=$(seid keys show $WALLET --bech val -a)
**Загружаем переменные**
    
    echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
    echo 'export VALOPER_ADDRESS='${VALOPER_ADDRESS} >> $HOME/.bash_profile
    source $HOME/.bash_profile
---
**Ждем синхронизации до 153759 высоты после чего обновляемся до 1.0.3beta**

	sudo systemctl stop seid && \ 
	sudo rm -rf $HOME/sei-chain && \ 
	git clone https://github.com/sei-protocol/sei-chain.git && \ 
	cd sei-chain && \ git checkout 1.0.3beta && \ 
	make install
	 
	seid version --long | head 

> version: 1.0.3beta

	sudo systemctl restart seid && journalctl -u seid -f -o cat

**И снова ждем синхронизации пока команда ** 

	seid status 2>&1 | jq ."SyncInfo"."catching_up"
не покажет 

> false

**Чтобы посмотреть кошельки используем**

    seid keys list

**Посмотреть баланс**

    seid query bank balances $WALLET_ADDRESS
**Сделать трансфер**

    seid tx bank send <FROM WALLET ADDRESS> <TO WALLET ADDRESS> 900000usei --chain-id=$CHAIN_ID  --fees=500usei
    


    
### Операции с валидатором
**Создаем валидатора**

    seid tx staking create-validator \
      --amount 900000usei \
      --from $WALLET \
      --commission-max-change-rate "0.1" \
      --commission-max-rate "0.2" \
      --commission-rate "0.07" \
      --min-self-delegation "1" \
      --pubkey  $(seid tendermint show-validator) \
      --moniker $NODENAME \
      --chain-id $CHAIN_ID
**Редактировать инфо валидатора**

    seid tx staking edit-validator \
    --moniker=$NODENAME \
    --identity=860918966CCB92D7 \  #вставить своё
    --website="http://t.me/oxcryptos" \ #вставить своё
    --details="не отличаюсь внимательностью" \ #вставить своё
    --chain-id=$CHAIN_ID \
    --from=$WALLET

**Освободить валидатора**

    seid tx slashing unjail \
      --broadcast-mode=block \
      --from=$WALLET \
      --chain-id=$CHAIN_ID \
      --gas=auto
      
### Делегирование, ределигирование, клайм

**Делегировать**


    seid tx staking delegate $VALOPER_ADDRESS 900000usei --from=$WALLET --chain-id=$CHAIN_ID  --fees=500usei

**Ределегировать другому валидатору**

    seid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000usei --from=$WALLET --chain-id=$CHAIN_ID --gas=auto

**Заклаймить  реварды**

    seid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID --gas=auto

**Заклаймить реварды с комиссией**

    seid tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CHAIN_ID

### Инфа о ноде
**Синхронизация**

    seid status 2>&1 | jq .SyncInfo
**Валидатор**

    seid status 2>&1 | jq .ValidatorInfo
**Нода**

    seid status 2>&1 | jq .NodeInfo
**ID ноды**

    seid tendermint show-node-id

**Удалить ноду**

	sudo systemctl stop seid && \
	sudo systemctl disable seid && \
	rm /etc/systemd/system/seid.service && \
	sudo systemctl daemon-reload && \
	cd $HOME && \
	rm -rf .sei .sei-chain sei-chain && \
	rm -rf $(which seid)
