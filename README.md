# Mina ağına "Block Producer" ya da "Snark Worker" olarak bağlanma  
# Docker  
(Bu kurulum bir user üzerinde yapılabilir. Başlamak için önce user oluşturun.)
1. Paketleri güncelleyin  
   `sudo apt update`  
2. Peer listesini indirin  
   `wget -O ~/peers.txt https://storage.googleapis.com/mina-seed-lists/mainnet_seeds.txt`  
3. .mina-config isimli bir dizin oluşturun  
   `mkdir $HOME/.mina-config`  
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
    --mount "type=bind,source=`pwd`/.mina-config,dst=/root/.mina-config" \  
    -e CODA_PRIVKEY_PASS="<PRIVKEY_PASS>" \  
    minaprotocol/mina-daemon:1.2.0-fe51f1e-mainnet \  
    daemon \  
    --block-producer-key /keys/my-wallet \  
    --peer-list-url https://storage.googleapis.com/mina-seed-lists/mainnet_seeds.txt  
    ```  
    b-) Snark Worker docker image için (tüm satırları tek seferde yapıştırın)  
    ```  
    docker run --name mina -d \  
    -p 8302:8302 \  
    --restart=always \  
    --mount "type=bind,source=`pwd`/keys,dst=/keys,readonly" \  
    --mount "type=bind,source=`pwd`/.mina-config,dst=/root/.mina-config" \  
    -e CODA_PRIVKEY_PASS="<PRIVKEY_PASS>" \  
    minaprotocol/mina-daemon:1.2.0-fe51f1e-mainnet \  
    daemon \  
    --run-snark-worker "<PUBLIC_KEY>" \
    --snark-worker-fee "0.1" \
    --peer-list-url https://storage.googleapis.com/mina-seed-lists/mainnet_seeds.txt \  
    --work-selection seq
    ```
14. Oluşturulan mina container içine girin  
    `docker exec -it mina bash`  
15. Networke bağlanma durumunuzu kontrol edin (sıradaki maddeden devam etmek için catchup ya da sync durumuna gelmesini bekleyin)  
    `mina client status`  
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
    `mina accounts import -privkey-path ~/keys/my-wallet`  
25. Sık kullanılan bilgileri değişkenlere atayacağınız .mina-env dosyasını oluşturun  
    `nano .mina-env`  
26. public key değişkenini oluşturduktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
    `MINA_PUBLIC_KEY=<PUBLIC_KEY>`  
27. Bu değişkenleri kullanabilmek için .mina-env dosyasını kaynak olarak belirleyin  
    `source .mina-env`  
28. Import edilen hesap üzerinde işlem yapabilmek için hesabın kilidini açın  
    `mina accounts unlock -public-key $MINA_PUBLIC_KEY`  
29. Hesabınızın bakiyesini kontrol edin  
    `mina accounts list`  
# Mina gönderme  
  (Burada receiver alanındaki adres 1. görev için gereken echo service'e ait adres. Farklı gönderimlerde alıcı adresini buraya yazın.)
  ```
  mina client send-payment \
    -amount 1 \
    -receiver B62qndJi5mnRoBZ8SAYDM1oR2SgAk5WpZC8hGpJUZ4e64kDHGbFMeLJ \
    -fee 0.1 \
    -sender $MINA_PUBLIC_KEY
  ```   
# Blok Producer çalıştırma  
  `mina client set-staking -public-key $MINA_PUBLIC_KEY`   
# Snark Worker çalıştırma  
  `mina client set-snark-work-fee 0.1`  
  `mina client set-snark-worker -address $MINA_PUBLIC_KEY`   
# Docker Image güncelleme  
1. Çalışan nodu durdurun  
   `docker exec -it mina mina client stop-daemon`  
2. Mina containeri durdurun  
   `docker stop mina`  
3. Mina containeri silin  
   `docker container rm mina`  
4. Docker imageleri listeleyin  
   `docker images`  
5. IMAGE ID ile Mina image silin  
   `docker rmi <IMAGE-ID>`  
6. .coda-config dosyasını silin  
   `sudo rm -rf .mina-config`  
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
`echo "deb [trusted=yes] http://packages.o1test.net stretch stable" | sudo tee /etc/apt/sources.list.d/mina.list`  
`sudo apt-get update`  
`sudo apt-get install -y mina-mainnet=1.2.0-fe51f1e`  
`wget -O ~/peers.txt https://storage.googleapis.com/mina-seed-lists/mainnet_seeds.txt`  
`mina daemon --generate-genesis-proof true --peer-list-url https://storage.googleapis.com/mina-seed-lists/mainnet_seeds.txt`  
after bootstrap `Ctrl-C`  
`sudo nano .mina-env`  
  ```
  EXTRA_FLAGS=" --block-producer-key /home/<USER-NAME>/keys/my-wallet \  
   --uptime-submitter-key /home/<USER-NAME>/keys/my-wallet \  
   --uptime-url http://34.134.227.208/v1/submit \  
   --coinbase-receiver <PUBLIC-KEY-PATH> \  (rewardların toplanacağı adres)  
   --limited-graphql-port 3095 \  (sidecar çalıştırıyorsanız gerekli)  
   (bp ile birlikte snark da çalışacak ise aşağıdaki 3 parametre eklenmeli)  
   --run-snark-worker <PUBLIC-KEY> \ (snark ödüllerinin toplanacağı adres)  
   --snark-worker-fee 0.0009 \ (snark fee)  
   --work-selection seq" (snark aktif)  
  UPTIME_PRIVKEY_PASS="<PRIVATE-KEY-PASSWORD>"  
  MINA_PRIVKEY_PASS="<PRIVATE-KEY-PASSWORD>"  
  CODA_PRIVKEY_PASS="<PRIVATE-KEY-PASSWORD>"  
  LOG_LEVEL=Info  
  FILE_LOG_LEVEL=Debug  
   ```  
`source .mina-env`  
`systemctl --user enable mina`  
`systemctl --user start mina`  
`systemctl --user restart mina`  
`systemctl --user status mina`  
`journalctl --user-unit mina -f`  
`mina client status`  

# Daemon update (config ve peerları silmeden)  
`systemctl --user stop mina`  
`sudo apt-get remove -y mina-testnet-postake-medium-curves`  
`echo "deb [trusted=yes] http://packages.o1test.net stretch stable" | sudo tee /etc/apt/sources.list.d/mina.list`  
`sudo apt-get update`  
`sudo apt-get install -y mina-mainnet=1.2.0-fe51f1e`  
`source .mina-env`  
`systemctl --user reload mina`  
`systemctl --user status mina`  
`journalctl --user-unit mina -f`  
`mina client status`  

# Daemon restart
`systemctl --user restart mina`  
