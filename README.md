# Mina ağına "Block Producer" ya da "Snark Worker" olarak bağlanma  
# Docker  
(Bu kurulum bir user üzerinde yapılabilir. Başlamak için önce user oluşturun.)
1. Paketleri güncelleyin  
   `sudo apt update`  
2. Peer listesini indirin  
   `wget -O ~/peers.txt https://raw.githubusercontent.com/MinaProtocol/mina/encore-peers/automation/terraform/testnets/encore/peers.txt`  
3. .coda-config isimli bir dizin oluşturun  
   `mkdir $HOME/.coda-config`  
4. Private ve public key dosyalarını oluşturmak için keys isimli bir dizin oluşturun ve içine girin  
    `mkdir keys`  
    `cd keys`  
5. nano metin düzenleyici programını indirin  
   `sudo apt-get install nano`  
6. my-wallet isimli bir dosya oluşturun  
    `sudo nano my-wallet`  
7. {"box_primitive": ile başlayan özel anahtarı bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
8. my-wallet.pub isimli bir dosya oluşturun  
    `sudo nano my-wallet.pub`  
9. public keyi bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
10. Kök dizine dönün  
    `cd ~ `  
11. keys dizini ve my-wallet dosyasını yetkilendirin  
    `sudo chmod 700 ~/keys`  
    `sudo chmod 600 ~/keys/my-wallet`  
12. Docker kurun ve yetkilendirin  
    `sudo apt-get install curl apt-transport-https ca-certificates software-properties-common`  
    `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`  
    `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`  
    `sudo apt update`  
    `apt-cache policy docker-ce`  
    `sudo apt install docker-ce`  
    `sudo chmod 666 /var/run/docker.sock` 
13. Yapmak istediğiniz işleme göre bunlardan birini seçin (Block Producer ya da Snark Worker)  
    a-) Block Producer docker image için (tüm satırları tek seferde yapıştırın)  
    ```
    docker run --name mina -d \
    -p 8302:8302 \
    --restart=always \
    --mount "type=bind,source=`pwd`/keys,dst=/keys,readonly" \
    --mount "type=bind,source=`pwd`/.coda-config,dst=/root/.coda-config" \
    --mount type=bind,source="`pwd`/peers.txt,dst=/root/peers.txt",readonly \
    -e CODA_PRIVKEY_PASS="<PRIVKEY_PASS>" \
    minaprotocol/mina-daemon-baked:0.3.3-3ef8663-encore-3b5824a \
    daemon \
    -peer-list-file /root/peers.txt \
    -block-producer-key /keys/my-wallet \
    -file-log-level Info \
    -log-level Info \
    -super-catchup
    ```  
    b-) Snark Worker docker image için (tüm satırları tek seferde yapıştırın)  
    ```
    docker run --name mina -d \
    -p 8302:8302 \
    --restart always \
    --mount "type=bind,source=`pwd`/keys,dst=/keys,readonly" \
    --mount "type=bind,source=`pwd`/.coda-config,dst=/root/.coda-config" \
    --mount type=bind,source="`pwd`/peers.txt,dst=/root/peers.txt",readonly \
    -e CODA_PRIVKEY_PASS="<PRIVKEY_PASS>" \
    minaprotocol/mina-daemon-baked:0.3.3-3ef8663-encore-3b5824a \
    daemon \
    -peer-list-file /root/peers.txt \
    -run-snark-worker "<PUBLIC_KEY>" \
    -snark-worker-fee "0.1" \
    -file-log-level Info \
    -log-level Info \
    -super-catchup \
    -work-selection seq
    ```
14. Oluşturulan mina container içine girin  
    `docker exec -it mina bash`  
15. Networke bağlanma durumunuzu kontrol edin (sıradaki maddeden devam etmek için catchup ya da sync durumuna gelmesini bekleyin)  
    `coda client status`  
16. nano metin düzenleyici programını bu container içine de indirin  
    `apt-get install nano`  
17. Private ve public key dosyalarını bu container içinde de oluşturmak için keys isimli bir dizin oluşturun ve içine girin   
    `mkdir keys`  
    `cd keys`  
18. my-wallet isimli bir dosya oluşturun  
    `nano my-wallet`  
19. {"box_primitive": ile başlayan özel anahtarı bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
20. my-wallet.pub isimli bir dosya oluşturun  
    `nano my-wallet.pub`  
21. public keyi bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
22. Kök dizine dönün  
    `cd ~ `  
23. keys dizini ve my-wallet dosyasını yetkilendirin  
    `chmod 700 ~/keys`  
    `chmod 600 ~/keys/my-wallet`  
24. Oluşturulan anahtar çiftini coda accounts içine import edin  
    `coda accounts import -privkey-path ~/keys/my-wallet`  
25. Sık kullanılan bilgileri değişkenlere atayacağınız .mina-env dosyasını oluşturun  
    `nano .mina-env`  
26. public key değişkenini oluşturduktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
    `MINA_PUBLIC_KEY=<PUBLIC_KEY>`  
27. Bu değişkenleri kullanabilmek için .mina-env dosyasını kaynak olarak belirleyin  
    `source .mina-env`  
28. Import edilen hesap üzerinde işlem yapabilmek için hesabın kilidini açın  
    `coda accounts unlock -public-key $MINA_PUBLIC_KEY`  
29. Hesabınızın bakiyesini kontrol edin  
    `coda accounts list`  
# Mina gönderme  
  (Burada receiver alanındaki adres 1. görev için gereken echo service'e ait adres. Farklı gönderimlerde alıcı adresini buraya yazın.)
  ```
  coda client send-payment \
    -amount 1 \
    -receiver B62qndJi5mnRoBZ8SAYDM1oR2SgAk5WpZC8hGpJUZ4e64kDHGbFMeLJ \
    -fee 0.1 \
    -sender $MINA_PUBLIC_KEY
  ```   
# Blok Producer çalıştırma  
  `coda client set-staking -public-key $MINA_PUBLIC_KEY`   
# Snark Worker çalıştırma  
  `coda client set-snark-work-fee 0.1`  
  `coda client set-snark-worker -address $MINA_PUBLIC_KEY`   
# Docker Image güncelleme  
1. Çalışan nodu durdurun  
   `docker exec -it mina coda client stop-daemon`  
2. Mina containeri durdurun  
   `docker stop mina`  
3. Mina containeri silin  
   `docker container rm mina`  
4. Docker imageleri listeleyin  
   `docker images`  
5. IMAGE ID ile Mina image silin  
   `docker rmi <IMAGE-ID>`  
6. .coda-config dosyasını silin  
   `sudo rm -rf .coda-config`  
7. peers.txt dosyasını silin  
   `sudo rm -R peers.txt`  
8. "Mina ağına bağlanma" başlığındaki;  
   2 ve 3. adımları uygulayın,  
   4-12 arasını atlayın,  
   13. işlemden itibaren yeni versiyon ile uygulamaya devam edin.   
# Yeni Keypair oluşturma  
1. Keypair oluşturma paketini indirin  
   `sudo apt-get install mina-generate-keypair`  
2. Keypair oluşturun  
   `mkdir ~/keys`  
   `chmod 700 ~/keys`  
   `mina-generate-keypair -privkey-path ~/keys/my-wallet`  
   `chmod 600 ~/keys/my-wallet`  
   
# Daemon  
`systemctl --user stop mina`  
`sudo rm -R .mina-config`  
`sudo rm -R peers.txt`  
`sudo apt-get remove -y mina-testnet-postake-medium-curves`  
`echo "deb [trusted=yes] http://packages.o1test.net release main" | sudo tee /etc/apt/sources.list.d/mina.list`  
`sudo apt-get update`  
`sudo apt-get install -y curl unzip mina-testnet-postake-medium-curves=1.0.0-fd39808`  
`sudo nano peers.txt`  
copy `https://storage.googleapis.com/seed-lists/finalfinal3_seeds.txt` content to `peers.txt`  
`mina daemon --generate-genesis-proof true --peer-list-url https://storage.googleapis.com/seed-lists/finalfinal3_seeds.txt`  
after bootstrap `Ctrl-C`  
`sudo nano .mina-env`  
  `CODA_PRIVKEY_PASS="private key password"`  
  `EXTRA_FLAGS=" --file-log-level Debug --coinbase-receiver <PUBLIC-KEY>"`  
`source .mina-env`  
`systemctl --user enable mina`  
`systemctl --user start mina`  
`systemctl --user status mina`  
`journalctl --user-unit mina -f`  
`mina client status`  

# Daemon restart
`systemctl --user restart mina`  
