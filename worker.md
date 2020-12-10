   NOT: <> parantezleri bulunan alanlara, parantez olmadan kendi bilgilerinizi yazın.  
   Ör. MINA_PRIVKEY_PASS=<PRIVATE KEY OLUŞTURURKEN SEÇTİĞİNİZ ŞİFRE> ---> MINA_PRIVKEY_PASS=123456789   
# Mina ağına bağlanma  
1. Yeni kullanıcı ekleyin  
   (istenilen şifre ve diğer bilgileri girin)  
   `sudo adduser <KULLANICI ADINIZ>`  
2. Kullanıcıya root yetkisi verin  
   `visudo`  
   "root ALL=(ALL:ALL) ALL" bulunan satırın altına  
   `<KULLANICI ADINIZ> ALL=(ALL:ALL) ALL` satırı ekledikten sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
3. Yeni kullanıcı ile yeni bir SSH penceresi açın  
4. Sistemi güncelleyin  
   `sudo apt update`  
5. Programa kabul edildiğinize dair gönderilen emaildeki "wget" ile başlayan komutu girin  
   `wget -O ~/peers.txt <PEER-LIST-URL>`  
6. .coda-config isimli bir dizin oluşturun  
   `mkdir $HOME/.coda-config`  
7. Private key oluştururken seçtiğiniz şifreyi bir değişkene atayın  
   `export MINA_PRIVKEY_PASS=<PRIVATE KEY OLUŞTURURKEN SEÇTİĞİNİZ ŞİFRE>`  
8. Private ve public key dosyalarını oluşturmak için keys isimli bir dizin oluşturun ve içine girin  
   `mkdir keys`  
   `cd keys`  
9. nano metin düzenleyici programını indirin  
   `sudo apt-get install nano`  
10. my-wallet isimli bir dosya oluşturun  
    `sudo nano my-wallet`  
11. {"box_primitive": ile başlayan özel anahtarı bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
12. my-wallet.pub isimli bir dosya oluşturun  
    `sudo nano my-wallet.pub`  
13. public keyi bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
14. Kök dizine dönün  
    `cd ~ `  
15. keys dizini ve my-wallet dosyasını yetkilendirin  
    `sudo chmod 700 ~/keys`  
    `sudo chmod 600 ~/keys/my-wallet`  
16. Docker kurun ve yetkilendirin  
    `sudo apt-get install curl apt-transport-https ca-certificates software-properties-common`  
    `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`  
    `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`  
    `sudo apt update`  
    `apt-cache policy docker-ce`  
    `sudo apt install docker-ce`  
    `sudo chmod 666 /var/run/docker.sock` 
17. mina docker image'i oluşturun (tüm satırları tek seferde yapıştırın)  
    ```
    docker run --name mina -d \
    -p 8301-8305:8301-8305 \
    --restart=always \
    --mount "type=bind,source=`pwd`/keys,dst=/keys,readonly" \
    --mount "type=bind,source=`pwd`/.coda-config,dst=/root/.coda-config" \
    --mount type=bind,source="`pwd`/peers.txt,dst=/root/peers.txt",readonly \
    -e CODA_PRIVKEY_PASS="$MINA_PRIVKEY_PASS" \
    minaprotocol/mina-daemon-baked:4.1-turbo-pickles-mina880882e-autoa026dd9 \
    daemon \
    -peer-list-file /root/peers.txt \
    -block-producer-key /keys/my-wallet \
    -insecure-rest-server \
    -file-log-level Debug \
    -log-level Info
    ```  
18. Oluşturulan mina container içine girin  
    `docker exec -it mina bash`  
19. Networke bağlanma durumunuzu kontrol edin (sıradaki maddeden devam etmek için catchup ya da sync durumuna gelmesini bekleyin)  
    `coda client status`  
20. nano metin düzenleyici programını bu container içine de indirin  
    `apt-get install nano`  
21. Private ve public key dosyalarını bu container içinde de oluşturmak için keys isimli bir dizin oluşturun ve içine girin   
    `mkdir keys`  
    `cd keys`  
22. my-wallet isimli bir dosya oluşturun  
    `nano my-wallet`  
23. {"box_primitive": ile başlayan özel anahtarı bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
24. my-wallet.pub isimli bir dosya oluşturun  
    `nano my-wallet.pub`  
25. public keyi bu dosyanın içine yapıştırdıktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
26. Kök dizine dönün  
    `cd ~ `  
27. keys dizini ve my-wallet dosyasını yetkilendirin  
    `chmod 700 ~/keys`  
    `chmod 600 ~/keys/my-wallet`  
28. Oluşturulan anahtar çiftini coda accounts içine import edin  
    `coda accounts import -privkey-path ~/keys/my-wallet`  
29. Sık kullanılan bilgileri değişkenlere atayacağınız .mina-env dosyasını oluşturun  
    `nano .mina-env`  
30. public key değişkenini oluşturduktan sonra CTRL+O ve Enter ile belgeyi kaydedin ve CTRL+X ile kapatın  
    `MINA_PUBLIC_KEY=<PUBLIC_KEY>`  
31. Bu değişkenleri kullanabilmek için .mina-env dosyasını kaynak olarak belirleyin  
    `source .mina-env`  
32. Import edilen hesap üzerinde işlem yapabilmek için hesabın kilidini açın  
    `coda accounts unlock -public-key $MINA_PUBLIC_KEY`  
33. Hesabınızın bakiyesini kontrol edin  
    `coda accounts list`  
34. Bakiye 0 ise discord #faucet kanalına `$request <PUBLIC_KEY>` komutunu girerek hesabınıza bakiye isteyin   
# Mina gönderme  
  (Burada receiver alanındaki adres 1. görev için gereken echo service'e ait adres. Farklı gönderimlerde alıcı adresini buraya yazın.)
  ```
  coda client send-payment \
    -amount 1 \
    -receiver B62qndJi5mnRoBZ8SAYDM1oR2SgAk5WpZC8hGpJUZ4e64kDHGbFMeLJ \
    -fee 0.1 \
    -sender $MINA_PUBLIC_KEY
  ```   
# Snark Worker çalıştırma  
  (Snark üretmeye başlamadan öncelikle blok bulma görevlerinin tamamlanması tavsiye ediliyor)  
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
   5, 6 ve 7. adımları uygulayın,  
   8 ile 16 arasını atlayın,  
   17. işlemden itibaren yeni versiyon ile (4.1-turbo-pickles-XXX-XXX) uygulamaya devam edin.   
# Yeni Keypair oluşturma  
1. Keypair oluşturma aracını indirin ve kurun  
   `echo "deb [trusted=yes] http://packages.o1test.net unstable main" | sudo tee /etc/apt/sources.list.d/coda.list`  
   `sudo apt-get update`  
   `sudo apt-get install libjemalloc-dev`  
   `sudo apt-get install mina-generate-keypair=0.0.16-beta7-20bce37`  
2. Keypair oluşturun  
   `mina-generate-keypair -privkey-path keys/my-wallet`  
3. Dizini ve my-wallet dosyasını yetkilendirin  
   `sudo chmod 700 ~/keys`  
   `sudo chmod 600 ~/keys/my-wallet`
