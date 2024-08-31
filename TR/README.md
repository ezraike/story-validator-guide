# Stort Protocol Validator Node Rehberi

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

### Story Kurulumu
```bash
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.9.11-2a25df1.tar.gz
tar -xzvf story-linux-amd64-0.9.11-2a25df1.tar.gz
sudo cp story-linux-amd64-0.9.11-2a25df1/story $HOME/go/bin/story
story version
```

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

### Daha hızlı senkron olması için PEER kullanabilirsiniz
```bash
PEERS="2f372238bf86835e8ad68c0db12351833c40e8ad@story-testnet-peer.itrocket.net:26656,00c495396dfee53a31476d7619d1cc252b9a47b9@89.58.62.213:26656,800bd9a3bb37a07d5c57c42a5de72d7ab370cfd1@100.42.189.22:26656,7af6ffb7b85695dec024163cf2140769bf17c1ec@65.21.210.147:26656,6a1b35d7c8deae3f6b0588855300af1dfa8ebd17@49.12.172.31:13656,8aa160aa30dd61a66230411f7bd9a5538d73023d@38.242.155.150:26656,8e33fb7dfa20e61bf743cdea89f8ca909946a189@65.108.232.134:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml

sudo systemctl stop story
sudo systemctl stop story-geth
sudo systemctl start story-geth
sudo systemctl start story
```

## Adım 2: Validator Kurma

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
#### Aşağıdki yol dizinine sunucunuza SFTP ile bağlanıp erişin ve privte keyinizi alın

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