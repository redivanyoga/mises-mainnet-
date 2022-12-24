 # AUTO INSTALLER WITH STATESYNC EASY TO USE BROOHHH !!! 
 
 #Spesifikasi minimum
 - RAM 8 
 - OS ubuntu 20
 - CPU 4 
 - SSD 1TB or high
 - Bandwitch up to 100MB/s 
 

```
wget https://raw.githubusercontent.com/redivanyoga/mises-mainnet-/main/install.sh && chmod +x install.sh && ./install.sh
```

# STATE SYNC
 *Stop dulu 
 ```
 sudo systemctl stop misestmd
 ```
 
 * Jalankan 
    ```
    misestmd unsafe-reset-all
    ```
 * EKSEKUSI
``` 
SNAP_RPC="https://e1.mises.site:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.misestm/config/config.toml

sudo systemctl restart misestmd && sudo journalctl -u misestmd -f --no-hostname -o cat
```

# Chek chunk bila masuk 10 cunk brarti jalan
```
sudo journalctl -fu misestmd -o cat | grep chunk
```

### Informasi node

   * cek sync node
```
misestmd status 2>&1 | jq .SyncInfo
```
   * cek log node
```
journalctl -fu misestmd -o cat
```
   * cek node info
```
misestmd status 2>&1 | jq .NodeInfo
```
   * cek validator info
```
misestmd status 2>&1 | jq .ValidatorInfo
```
  * cek node id
```
misestmd tendermint show-node-id
```

### Membuat wallet
   * wallet baru
```
misestmd keys add $WALLET
```
   * recover wallet
```
misestmd keys add $WALLET --recover
```
   * list wallet
```
misestmd keys list
```
   * hapus wallet
```
misestmd keys delete $WALLET
```
### Simpan informasi wallet
```
MISES_WALLET_ADDRESS=$(misestmd keys show $WALLET -a)
MISES_VALOPER_ADDRESS=$(misestmd keys show $WALLET --bech val -a)
echo 'export MISES_WALLET_ADDRESS='${MISES_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export MISES_VALOPER_ADDRESS='${MISES_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Membuat validator
 * cek balance
```
misestmd query bank balances $MISES_WALLET_ADDRESS
```
 * membuat validator
```
misestmd tx staking create-validator \
  --amount 1000000umis \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(misestmd tendermint show-validator) \
  --moniker $NODENAME \
  --fees 250umis \
  --chain-id $MISES_CHAIN_ID
```
 * edit validator
```
misestmd tx staking edit-validator \
  --moniker="nama-node" \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$MISES_CHAIN_ID \
  --fees 250umis \
  --from=$WALLET
```
 ° unjail validator
```
misestmd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$MISES_CHAIN_ID \
  --fees=250umis
```
### Voting
```
misestmd tx gov vote 1 yes --from $WALLET --chain-id=$MISES_CHAIN_ID
```
### Delegasi dan Rewards
  * delegasi
```
misestmd tx staking delegate $MISES_VALOPER_ADDRESS 1000000umis --from=$WALLET --chain-id=MISES_CHAIN_ID --fees=250umis
```
  * withdraw reward
```
misestmd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$MISES_CHAIN_ID --fees=250umis
```
  * withdraw reward beserta komisi
```
misestmd tx distribution withdraw-rewards $MISES_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$MISES_CHAIN_ID
```

### Stop Node

```
sudo systemctl stop misestmd
```

### Restart Node

```
sudo systemctl restart misestmd
```

### Hapus node
```
sudo systemctl stop misestmd && \
sudo systemctl disable misestmd && \
rm /etc/systemd/system/misestmd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf mises-tm && \
rm -rf mises.sh && \
rm -rf .misestm && \
rm -rf $(which misestmd)
```
