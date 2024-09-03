# Story Protocol Validator Node Rehberi

![image](https://github.com/user-attachments/assets/ceb92713-eb1c-4c95-98f1-ce4f969a4a7e)

## Story Protocol Nedir?

Story, fikri mülkiyet (Intellectual Property veya IP) olarak adlandırılan değerli varlıkları blok zinciri dünyasına entegre ediyor. Fikri mülkiyet, şarkılar, resimler, icatlar, oyun karakterleri gibi yaratıcı çalışmaları ve fikirleri kapsar. Amaç, bu eserleri dijital olarak “token” adı verilen varlıklara dönüştürmek ve böylece daha kolay alınıp satılabilmelerini ve kullanılabilmelerini sağlamaktır.

* [Twitter / X](https://x.com/StoryProtocol)
* [Website](https://www.storyprotocol.xyz/)
* [Discord](https://discord.gg/storyprotocol)
* [Docs](https://docs.story.foundation/docs/what-is-story)
* [GitHub](https://github.com/storyprotocol)
* [Explorer](https://explorer.story.foundation/)

## Sistem Gereksinimleri

| Donanım    | Tavsiye Edilen |
|------------|----------------|
| CPU        | 4 Cores        |
| RAM        | 8 GB           |
| Disk       | 200 GB         |
| Bandwith   | 10 MBit/s      |

## 1. Adım: Sistem Güncellemeri ve Yükleme İşlemleri

### Gerekli paketleri güncelleyelim
```bash
sudo apt update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 -y
mkdir -p $HOME/go/bin
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Story-Geth Kurulumu

```bash
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
tar -xzvf geth-linux-amd64-0.9.2-ea9f0d2.tar.gz
sudo cp geth-linux-amd64-0.9.2-ea9f0d2/geth $HOME/go/bin/story-geth
story-geth version
```
![image](https://github.com/user-attachments/assets/6981c9f1-27ff-441e-aca9-d1537a7a0a8f)

### Story Kurulumu
```bash
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
story version
```
![image](https://github.com/user-attachments/assets/10f44c14-292a-482c-aef8-f06a201e21a3)

### Moniker Oluşturma
#### Moniker adınızı belirleyin
```bash
story init --network iliad --moniker <moniker-isim>
```

### Story-Geth için Servis Kurulumu
```bash
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

### Story için Servis Kurulumu
```bash
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

### Story-geth Servisini Başlatın
```bash
sudo systemctl daemon-reload && \
sudo systemctl start story-geth && \
sudo systemctl enable story-geth && \
sudo systemctl status story-geth
```

### Story Servisini Başlatın
```bash
sudo systemctl daemon-reload && \
sudo systemctl start story && \
sudo systemctl enable story && \
sudo systemctl status story
```

### Log Kontrolü
```bash
sudo journalctl -u story-geth -f -o cat
```

### Senkronizasyon Kontrolü
```bash
curl localhost:26657/status | jq
```
#### catching_up çıktısı “false” olmadan validator oluşturma aşamasına geçmeyin!

### Daha hızlı senkron olması için PEER ve SNAPSHOT kullanabilirsiniz
```bash
PEERS="a320f8a15892bddd7b5502527e0d11c5b5b9d0e3@69.67.150.107:29931,2f372238bf86835e8ad68c0db12351833c40e8ad@story-testnet-rpc.itrocket.net:26656,d4c5dcfbec11d80399bcf18d83a157259ca3efc7@138.201.200.100:26656,15c7e2b630c04ee11b2c3cfbfb1ede0379df9407@52.74.117.64:26656,359e4420e63db005d8e39c490ad1c1c329a68df3@3.222.216.118:26656,eeb7d2096a887f8ff8fdde2695c394fcf5a19273@194.238.30.192:36656,0b512c9a4421c0259813aaa05c865f82365fa7c0@3.1.137.11:26656,f4d96bf0dc67a05a48287ca2c821bc8e1d2b2023@63.35.134.129:26656,5e4f9ce2d20f2d3ef7f5c92796b1b954384cbfe1@34.234.176.168:26656,371ee318d105b0239b3997c287068ccbbcd46a91@3.248.113.42:26656,df40eee673df8eb3eddd10b359539f0e86ecaee3@207.244.230.111:36656,c82d2b5fe79e3159768a77f25eee4f22e3841f56@3.209.222.59:26656,960278d079a111b44c207dca7c2ffac640b477d1@44.223.234.211:26656,1708afbf73e2fbbb5a943aa2d97c976bf8e0d25c@52.9.183.131:26656,1cceccb08bae25a0f91fe85b0ca562fa791f47aa@184.169.154.204:26656,8876a2351818d73c73d97dcf53333e6b7a58c114@3.225.157.207:26656,0a2bc2ce69b7292deaf0b6a33af76bf9b2e25ec6@88.198.46.55:41656,6bb4ed28b08a186fc1373cfc2e96b83165c1e882@162.55.245.254:33656,a2fe3dfd6396212e8b4210708e878de99307843c@54.209.160.71:26656,aac5871efa351872789eef15c2da7a55a68abdad@88.218.226.79:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml

sudo systemctl restart story
```

### SNAPSHOT Devam Edelim (kodları tek tek girelim)
```bash
sudo apt-get install wget lz4 aria2 pv -y

aria2c -x 16 -s 16 https://vps6.josephtran.xyz/Story/Geth_snapshot.lz4 -o Geth_snapshot.lz4
aria2c -x 16 -s 16 https://vps6.josephtran.xyz/Story/Story_snapshot.lz4 -o Story_snapshot.lz4

sudo systemctl stop story-geth
sudo systemctl stop story

mv $HOME/.story/story/data/priv_validator_state.json $HOME/.story/priv_validator_state.json.backup

sudo rm -rf $HOME/.story/geth/iliad/geth/chaindata
sudo rm -rf $HOME/.story/story/data

sudo mkdir -p $HOME/.story/geth/iliad/geth/chaindata
lz4 -d Geth_snapshot.lz4 | pv | sudo tar xv -C $HOME/.story/geth/iliad/geth/

sudo mkdir -p $HOME/.story/story/data
lz4 -d Story_snapshot.lz4 | pv | sudo tar xv -C $HOME/.story/story/

mv $HOME/.story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json

sudo systemctl start story-geth
sudo systemctl start story
```

## Adım 2: Validator Kurma

#### Catching Up durumu False olana kadar Validator kurulumuna geçmeyin!!

### Bir cüzdan oluşturma
```bash
story validator export --export-evm-key --evm-key-path .env
story validator export
story validator export --export-evm-key 
```
#### Size verilen cüzdan adreslerini kaydedin!
![image](https://github.com/user-attachments/assets/8779935b-368f-4541-a504-b653f644bc9f)

### Faucet
* [Faucet 1](https://faucet.story.foundation/)
* [Faucet 2](https://thirdweb.com/story-iliad-testnet)
* [Faucet 3](https://faucet.quicknode.com/story)

### Private Key'inizi öğrenin
#### Aşağıdaki yol dizinine sunucunuza SFTP ile bağlanıp erişin ve private keyinizi alın

##### /root/.story/story/story/config/private_key.txt

### Validator Oluşturma
```bash
story validator create --stake 1000000000000000000
```
#### Size verilen URL adresini ve TX Hashini saklayın!
![image](https://github.com/user-attachments/assets/fac5c11e-119a-4f18-8fc5-d9fcc329e7e1)


### Kendinize token delege edin
#### Aşağıdaki kodu size verilen cüzdan bilgilerine göre düzenleyin
```bash
story validator stake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1000000000000000000 \
   --private-key YOUR-PRIVATE-KEY
```
#### Self Delegation bölümünde bir hata var, minimum miktarın 1024'e çıktığı söyleniyor, bu nedenle kendinize delege edemeyebilirsiniz. Endişelenmeyin.

### Validator adresinizi öğrenin
```bash
cd $HOME/.story/story/config
cat priv_validator_key.json | grep address
```
![image](https://github.com/user-attachments/assets/1bf2f40e-33cc-4923-8729-5cd23fc5be74)

### Bu adresi tarayıcınızda aratın
```bash
https://testnet.story.explorers.guru/validator/<validator_address>
```
#### Son olarak validator URL'nizi, validator explorer linkini ve oluşturduğunuz cüzdanın EVM adresini yetkilileri de etiketleyerek “dev-chat” kanalına gönderin. <3

![image](https://github.com/user-attachments/assets/631bb70d-9a50-40bb-ac4c-ed9931d8b8d6)

### Validator Linkim: https://testnet.story.explorers.guru/validator/657B5A2BDDBD14FD207D1B318C634A29D2319AEB

### Bir sorun olursa bana ulaşabileceğiniz iletişim adreslerim: 

#### Discord: ezraike
#### Twitter: ezraike
#### Telegram: ezraike
#### E-mail: lifedefiedturkish@gmail.com
