# CODA WORKER  
`sudo apt update`
`wget -O ~/peers.txt <PEER-LIST-URL>`  
`mkdir $HOME/.coda-config`  
`export CODA_PRIVKEY_PASS=<PRIVATE_KEY_IMPORT_PASSWORD>`  
`mkdir keys`  
`cd keys`  
`sudo nano my-wallet` (past private key, save and exit)  
`sudo nano my-wallet.pub` (past public key, save and exit)  
`cd ~ `  
`chmod 700 ~/keys`  
`sudo chmod 600 ~/keys/my-wallet`   
# Install Docker and running daemon using Docker  
`sudo apt-get install curl apt-transport-https ca-certificates software-properties-common`  
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`  
`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`  
`sudo apt update`  
`apt-cache policy docker-ce`  
`sudo apt install docker-ce`  
`sudo chmod 666 /var/run/docker.sock`   
```
docker run --name mina -d \
-p 8301-8305:8301-8305 \
--restart=always \
--mount "type=bind,source=`pwd`/keys,dst=/keys,readonly" \
--mount "type=bind,source=`pwd`/.coda-config,dst=/root/.coda-config" \
--mount type=bind,source="`pwd`/peers.txt,dst=/root/peers.txt",readonly \
-e CODA_PRIVKEY_PASS="$CODA_PRIVKEY_PASS" \
minaprotocol/mina-daemon-baked:4.1-turbo-pickles-mina4a053a4-auto8a44ae5 \
daemon \
-peer-list-file /root/peers.txt \
-block-producer-key /keys/my-wallet \
-insecure-rest-server \
-file-log-level Debug \
-log-level Info
```  
`docker exec -it mina bash`  
`coda client status` (wait until catchup)  
`apt-get install nano`  
`mkdir keys`  
`cd keys`  
`nano my-wallet` (past private key, save and exit)  
`nano my-wallet.pub` (past public key, save and exit)  
`cd ~ `  
`chmod 700 ~/keys`  
`chmod 600 ~/keys/my-wallet`  
`coda accounts import -privkey-path ~/keys/my-wallet`  
`nano .mina-env`  
```
CODA_PRIVKEY_PASS="<PRIVATE_KEY_IMPORT_PASSWORD>"
CODA_PUBLIC_KEY=<PUBLIC_KEY>
MINA_PUBLIC_KEY=<PUBLIC_KEY>
```  
`source .mina-env`  
`coda accounts unlock -public-key $CODA_PUBLIC_KEY`  
`coda accounts list` (if the balance is 0, go to discord #faucet channel and send `$request <PUBLIC_KEY>` and wait for the balance to increase)   
# SET SNARK WORKER  
`coda client set-snark-work-fee 0.1`  
`coda client set-snark-worker -address $MINA_PUBLIC_KEY`   
# SEND MINA  
```
coda client send-payment \
  -amount 1 \
  -receiver B62qndJi5mnRoBZ8SAYDM1oR2SgAk5WpZC8hGpJUZ4e64kDHGbFMeLJ \
  -fee 0.1 \
  -sender $CODA_PUBLIC_KEY
```  
