# coda worker

1. Öncelikle bu [video](https://www.youtube.com/watch?v=iSrtjxOsC6A&feature=youtu.be)'nun ilk 3 dakikasında anlatılan şekilde çalışma ortamını hazırlayın ve yeni bir ssh(ssh-1) penceresi açın
2. Daha önce coda kurulduysa temizlemek için  
`sudo apt-get remove coda-testnet-postake-medium-curves`  
`sudo apt-get remove coda-kademlia`  
3. Coda kurulumu için  
`echo "deb [trusted=yes] http://packages.o1test.net unstable main" | sudo tee /etc/apt/sources.list.d/coda.list`  
`sudo apt-get update`  
`sudo apt-get install -t unstable coda-testnet-postake-medium-curves=0.0.12-beta+406048-feature-bump-genesis-timestamp-3e9b174-PV48525e92`  
22, 8302 ve 8303 portlarını açın  
4. Coda'yı ilk kez çalıştırıp eşlere bağlanmak için coda.service isimli scripti oluşturun  
`cd /lib/systemd/system`  
`sudo nano coda.service`  
Aşağıdaki kodu script içine yapıştırdıktan sonra Ctrl+O ile kaydedin ve Ctrl+X ile dosyayı kapatın  
```
Description=coda-daemon

[Service]
Type=simple
Restart=always
RestartSec=5s
ExecStart=/usr/local/bin/coda daemon \
    -external-port 8302 \
    -peer $SEED1 \
    -peer $SEED2
[Install]
WantedBy=multi-user.target
```  
5. Servisi başlatın  
`cd`  
`sudo systemctl enable coda`  
`sudo systemctl start coda`  
6. Yeni bir ssh(ssh-2) penceresi açın ve aşağıdaki komut ile coda'yı takip edin  
`sudo tail -f /var/log/syslog`  
7. ssh-1 içinde durumu kontrol edin  
`coda client status`  
8. Sync status: Synced durumuna gelmesini bekleyin
9. Daha önce public key oluşturuldu ve private key yedeği alındı ise;
  keys isimli bir dizin oluşturun  
`mkdir -m 700 keys`  
`cd keys`  
`sudo nano my-wallet` private keyi yapıştırıp kaydedin  
`sudo nano my-wallet.pub` public keyi yapıştırıp kaydedin  
10. İlk defa public key oluşturacak ise;  
`sudo apt-get install coda-generate-keypair-phase3`  
`coda-generate-keypair-phase3 -privkey-path keys/my-wallet` komut sonunda verilen public keyi saklayın  
`cat keys/my-wallet` private keyi saklayın  
11. keys dizini içinde oluşturulan keyleri accounts içine tanımlayın  
`sudo passwd` root için şifre oluşturun ve onaylayın  
`su` root hesabına geçin  
`cd`  
`chmod -R 700 /home/{user}/keys` keys dizini için yetkiyi 700 olarak belirleyin (user yerine kullanıcınız)  
`coda accounts import -privkey-path /home/{user}/keys/my-wallet` private keyi import edin (user yerine kullanıcınız)  
`coda accounts list` public keyi bu listede görüyor olmalısınız  
`coda accounts unlock -public-key <KEY>` hesabın kilidini açın (KEY yerine public keyiniz)  
12. ssh1'i kapatın ve yeni bir ssh penceresi açın. Snark worker ve staker olarak devam etmek için phyton kurun ve coda_restart.py dosyasını oluşturun   
`sudo apt-get install python`  
`sudo apt-get install python-psutil`  
`mkdir coda_python`  
`sudo chmod +x coda_python`  
`cd coda_python`  
`sudo nano coda_restart.py`  
Aşağıdaki kodu script içine yapıştırdıktan sonra Ctrl+O ile kaydedin ve Ctrl+X ile dosyayı kapatın (user yerine kullanıcınız, KEY yerine public keyiniz ve WALLET-PASSWORD yerine cüzdan şifrenizi yazın) 
```
#!/usr/bin/python

import psutil
import subprocess
import os
import time

while True:
        def checkIfProcessRunning(Coda):
            '''
            Check if there is any running process that contains the given name processName.
            '''
            #Iterate over the all the running process
            for proc in psutil.process_iter():
                try:
                    # Check if process name contains the given name string.
                    if Coda.lower() in proc.name().lower():
                        return True
                except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):
                    pass
            return False;

        if checkIfProcessRunning('coda'):
                print('Coda is running')
                time.sleep(5)
        else:
                os.system('export main_key=<KEY>')
                print('coda is down attempting restart')
                os.system('CODA_PRIVKEY_PASS=<WALLET-PASSWORD> coda daemon -peer $SEED1 -peer $SEED2 -run-snark-worker <KEY> -snark-worker-fee 1 -block-producer-key /home/{user}/keys/my-wallet')
                time.sleep(5)
```
13. Ana dizine geçin ve coda_restart.py scriptini çalıştıracak coda_restart.service dosyasını oluşturun  
`cd`  
`cd /lib/systemd/system`  
`sudo nano coda_restart.service`  
Aşağıdaki kodu script içine yapıştırdıktan sonra Ctrl+O ile kaydedin ve Ctrl+X ile dosyayı kapatın (user yerine kullanıcınız)
```
[Unit]
Description=Coda Restart Service
After=multi-user.target
Conflicts=getty@tty1.service

[Service]
Type=simple
ExecStart=/usr/bin/python /home/{user}/coda_python/coda_restart.py
StandardInput=tty-force

[Install]
WantedBy=multi-user.target
```
14. Önce çalışmakta olan servisi durdurun. Ardından snark worker ve staker olarak hazırlanan servisi çalıştırın  
`cd`  
`sudo systemctl stop coda`  
`sudo systemctl enable coda_restart.service`  
`sudo systemctl start coda_restart.service`  
15. ssh-1 içinde durumu kontrol edin  
`coda client status`  
16. Sync status: Synced durumuna gelmesini bekleyin. 
17. Coda bakiyenizi kontrol etmek için  
`coda client get-balance -public-key <KEY>`
18. Coda göndermek için (KEY yerine public keyiniz ve RECEIVER-KEY yerine alıcı public keyini yazın)  
`coda client send-payment -amount 1 -receiver <RECEIVER-KEY> -fee 1 -sender <KEY>`
