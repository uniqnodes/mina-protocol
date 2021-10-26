# Install  

1 `sudo apt-get install -y mina-bp-stats-sidecar=1.2.0-fe51f1e-mainnet`  
2 `apt-get update && apt-get install python3 python3-certifi`  
3 `sudo nano /etc/mina-sidecar.json`  

```
{
  "uploadURL": "https://us-central1-mina-mainnet-303900.cloudfunctions.net/block-producer-stats-ingest/?token=72941420a9595e1f4006e2f3565881b5",
  "nodeURL": "http://127.0.0.1:3095"
}
```  
 4 `sudo nano /etc/systemd/system/mina-bp-stats-sidecar.service`  
``` 
Restart=always
RestartSec=5
```  
5 `sudo nano ~/.mina-env`  
```
CODA_PRIVKEY_PASS="private key password"
EXTRA_FLAGS=" --file-log-level Debug --limited-graphql-port 3095"
```  
6 `sudo systemctl enable mina-bp-stats-sidecar`  
7 `sudo service mina-bp-stats-sidecar start`  
7 `sudo service mina-bp-stats-sidecar restart`  
8 `sudo service mina-bp-stats-sidecar status`  
9 `journalctl -f -u mina-bp-stats-sidecar.service`
