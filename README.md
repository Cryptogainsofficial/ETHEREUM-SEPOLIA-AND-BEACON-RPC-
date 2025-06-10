
__REQUIREMENTS__

OS: Ubuntu 20.04 or later
RAM -  8 - 16 GB
CPU -  4 CORE
DISK -  1 TB AND ABOVE


__Step 1. Install Dependecies__

 **Packages:**
```bash 
sudo apt-get update && sudo apt-get upgrade -y
```
   
```bash
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```
__INSTALL DOCKER__
```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker
sudo docker run hello-world

sudo systemctl enable docker
sudo systemctl restart docker
```
__Step 2. Create Directories__
```bash
mkdir -p /root/ethereum/execution
mkdir -p /root/ethereum/consensus
```

__Step 3. Generate the JWT secret:__

``bash
openssl rand -hex 32 > /root/ethereum/jwt.hex
```
__Verify JWT secrets Installed__

```bash
cat /root/ethereum/jwt.hex
```

__Step 4. Configure docker-compose.yml__

```bash
cd ethereum
```
```bash
nano docker-compose.yml
```

__Replace the below  code into your docker-compose.yml file:__
```bash
services:
  geth:
    image: ethereum/client-go:stable
    container_name: geth
    network_mode: host
    restart: unless-stopped
    ports:
      - 30303:30303
      - 30303:30303/udp
      - 8545:8545
      - 8546:8546
      - 8551:8551
    volumes:
      - /root/ethereum/execution:/data
      - /root/ethereum/jwt.hex:/data/jwt.hex
    command:
      - --sepolia
      - --http
      - --http.api=eth,net,web3
      - --http.addr=0.0.0.0
      - --authrpc.addr=0.0.0.0
      - --authrpc.vhosts=*
      - --authrpc.jwtsecret=/data/jwt.hex
      - --authrpc.port=8551
      - --syncmode=snap
      - --datadir=/data
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  prysm:
    image: gcr.io/prysmaticlabs/prysm/beacon-chain
    container_name: prysm
    network_mode: host
    restart: unless-stopped
    volumes:
      - /root/ethereum/consensus:/data
      - /root/ethereum/jwt.hex:/data/jwt.hex
    depends_on:
      - geth
    ports:
      - 4000:4000
      - 3500:3500
    command:
      - --sepolia
      - --accept-terms-of-use
      - --datadir=/data
      - --disable-monitoring
      - --rpc-host=0.0.0.0
      - --execution-endpoint=http://127.0.0.1:8551
      - --jwt-secret=/data/jwt.hex
      - --rpc-port=4000
      - --grpc-gateway-corsdomain=*
      - --grpc-gateway-host=0.0.0.0
      - --grpc-gateway-port=3500
      - --min-sync-peers=3
      - --checkpoint-sync-url=https://checkpoint-sync.sepolia.ethpandaops.io
      - --genesis-beacon-api-url=https://checkpoint-sync.sepolia.ethpandaops.io
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```
__COPY ABOVE CODE RIGHT CLICK TO PASTE IN THE NANO FILE  AND THE CTRL (X) THEN Y THEN HI ENTER TO AVE AND EXIT__

__Step 5. Run Geth & Prysm Nodes__

```bash
docker compose up -d
 ```
```bash
#CHECLK LOGS#
docker compose logs -f
 ```


__U WILL see in the logs, Prysm is synced, but  Geth will take 2 days to be sync depending on your vps or system specs__


__Step 6. Checking If Nodes Are Synced__
```bash
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
 ```
✅Response if  synced:
```bash
{"jsonrpc":"2.0","id":1,"result":false}
```
__🚫Response if still syncing:__
```bash
{"jsonrpc":"2.0","id":1,"result":{"currentBlock":"0x1a2b3c","highestBlock":"0x1a2b4d","startingBlock":"0x0"}}
```

__Beacon Node Prysm__
Response if synced:
```bash
{"data":{"head_slot":"12345","sync_distance":"0","is_syncing":false}}
```
__Response if still syncing:__
```bash
{"data":{"head_slot":"12345","sync_distance":"100","is_syncing":true}}
```

__Step 7. Make Sure  VPS Firewall Is Enabled__

```bash
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw enable
```
__Allow Geth P2P ports:__
```bash
sudo ufw allow 30303/tcp   # Geth P2P
sudo ufw allow 30303/udp   # Geth P2P
```
__Allow ports for local use can only be used on same machine__
```bash
sudo ufw allow from 127.0.0.1 to any port 8545 proto tcp
sudo ufw allow from 127.0.0.1 to any port 3500 proto tcp
```
__Allow ports to use on other machines__
  __Note replace (your vps ip with your actual machine's ip address__
```bash
  sudo ufw allow from <your-vps-ip> to any port 8545 proto tcp
sudo ufw allow from <your-vps-ip> to any port 3500 proto tcp
```
__Reload Firewall to apply changes :__

```bash
sudo ufw reload
```

__HAPPY CODING__
FORK ME IF YOU FIND THIS HELPFUL
