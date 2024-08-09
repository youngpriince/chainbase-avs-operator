# Chainbase-avs-operator

![image](https://github.com/user-attachments/assets/d971db5b-020d-4ee3-aaf6-1b95392d523d)
Chainbase is the world’s largest omnichain data network designed to integrate all blockchain data into a unified ecosystem, providing an open and transparent data interoperability layer for the AI era.


## How to setup an AVS operator
## System Requirements
Based on their [official docs](https://blog.chainbase.com/how-to-setup-an-avs-operator#h-hardware)

![image](https://github.com/user-attachments/assets/f9a0ed5e-acbe-4de3-afe7-5442c9f69f15)

## Install dependencies
```console
# Update & Install Packages
sudo apt update & sudo apt upgrade -y
sudo apt install ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev curl git wget make jq build-essential pkg-config lsb-release libssl-dev libreadline-dev libffi-dev gcc screen unzip lz4 -y

# Install Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
docker version

# Install Docker-Compose
VER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4)

curl -L "https://github.com/docker/compose/releases/download/"$VER"/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version

# Docker Permission to user
sudo groupadd docker
sudo usermod -aG docker $USER

# Install Go
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

## Install Eigenlayer CLI
```console
curl -sSfL https://raw.githubusercontent.com/layr-labs/eigenlayer-cli/master/scripts/install.sh | sh -s

export PATH=$PATH:~/bin

eigenlayer --version
```

## Clone Chainbase AVS repo
```console
git clone https://github.com/chainbase-labs/chainbase-avs-setup

cd chainbase-avs-setup/holesky
```

## Wallet
create a new EVM wallet and fund it with at least 1 ETH faucet on the holesky chain.
you can obtain ETH faucet [here](https://cloud.google.com/application/web3/faucet/ethereum/holesky)

> Replace `PRIVATEKEY` with yours
```console
eigenlayer operator keys import --key-type ecdsa opr PRIVATEKEY
```
> Enter a password and press Enter

This is a prompt asking you to enter a password to encrypt the ECDSA private key.

![image](https://github.com/user-attachments/assets/2e928612-2152-41a2-8339-e986b74e3a6c)

`Ctrl + c then Enter then Enter` to save

## Config & Register Operator
```console
eigenlayer operator config create
```
* `operator address`: Your Eigenlayer ETH address
* `earnings address`: press `Enter`
* `ETH rpc url`: https://ethereum-holesky-rpc.publicnode.com
* `network`: holesky
* `signer type`: local_keystore
* `ecdsa key path:`: /root/.eigenlayer/operator_keys/opr.ecdsa.key.json

The command will create two files: operator.yaml and metadata.json.

edit metadata.json `nano metadata.json` and set your operator name, logo image public URL, website, and Twitter account. Upload the file to a publicly accessible location.
* `logo`: Must be raw link to your operator favorite logo
* Logo only supports `.png` file less than 1MB in size
* Create a repositry in github and upload `.png` there
* To get your logo raw link: **1)** Navigate to the file in your repository, **2)** Click on the file name to view its contents, **3)** In the address bar of your browser, **4)** replace `blob` with `raw` in the URL, **5)** Press Enter to load the raw content view, **6)** Copy the updated URL from the address bar and paste in front of `logo` field in `metadata.json`
* My example `metadata.json` file in github: https://github.com/youngpriince/chainbase-metadata/main/metadata.json (Don't use this! Create yours)

edit `operator.yaml` 
```console
nano operator.yaml
```
Add your metadata.json raw url in github in front of `metadata-url`

![image](https://github.com/user-attachments/assets/fd845525-86e1-4a36-ba20-fbe1aff29ee5)

## Register operator
```console
eigenlayer operator register operator.yaml

# Check status
eigenlayer operator status operator.yaml
```
![image](https://github.com/user-attachments/assets/05638da7-16ae-4c5f-a55b-64710be75b52)

## Config Chainbase AVS
**Create .env file**
```console
# Delete old files
rm -rf .env

# Open Edit menu
nano .env
```

**Paste below codes in it**
* `NODE_ECDSA_KEY_PASSWORD`: Replace `***123ABCabc123***` with your Eigenlayer password
* Optional: You can Change #TODO lines if needed but it should be okay by default
```
# Chainbase AVS Image
MAIN_SERVICE_IMAGE=repository.chainbase.com/network/chainbase-node:testnet-v0.1.7
FLINK_TASKMANAGER_IMAGE=flink:latest
FLINK_JOBMANAGER_IMAGE=flink:latest
PROMETHEUS_IMAGE=prom/prometheus:latest

MAIN_SERVICE_NAME=chainbase-node
FLINK_TASKMANAGER_NAME=flink-taskmanager
FLINK_JOBMANAGER_NAME=flink-jobmanager
PROMETHEUS_NAME=prometheus

# FLINK CONFIG
FLINK_CONNECT_ADDRESS=flink-jobmanager
FLINK_JOBMANAGER_PORT=8081
NODE_PROMETHEUS_PORT=9091
PROMETHEUS_CONFIG_PATH=./prometheus.yml

# Chainbase AVS mounted locations
NODE_APP_PORT=8080
NODE_ECDSA_KEY_FILE=/app/operator_keys/ecdsa_key.json
NODE_LOG_DIR=/app/logs

# Node logs configs
NODE_LOG_LEVEL=debug
NODE_LOG_FORMAT=text

# Metrics specific configs
NODE_ENABLE_METRICS=true
NODE_METRICS_PORT=9092

# holesky smart contracts
AVS_CONTRACT_ADDRESS=0x5E78eFF26480A75E06cCdABe88Eb522D4D8e1C9d
AVS_DIR_CONTRACT_ADDRESS=0x055733000064333CaDDbC92763c58BF0192fFeBf

###############################################################################
####### TODO: Operators please update below values for your node ##############
###############################################################################
# TODO: Operators need to point this to a working chain rpc
NODE_CHAIN_RPC=https://rpc.ankr.com/eth_holesky
NODE_CHAIN_ID=17000

# TODO: Operators need to update this to their own paths
USER_HOME=$HOME
EIGENLAYER_HOME=${USER_HOME}/.eigenlayer
CHAINBASE_AVS_HOME=${EIGENLAYER_HOME}/chainbase/holesky

NODE_LOG_PATH_HOST=${CHAINBASE_AVS_HOME}/logs

# TODO: Operators need to update this to their own keys
NODE_ECDSA_KEY_FILE_HOST=${EIGENLAYER_HOME}/operator_keys/opr.ecdsa.key.json

# TODO: Operators need to add password to decrypt the above keys
# If you have some special characters in password, make sure to use single quotes
NODE_ECDSA_KEY_PASSWORD=***eigenlayer password***
```

**Create `docker-compose.yml` file**
```console
# Remove old file
rm -rf docker-compose.yml

# Open edit menu
nano docker-compose.yml
```

**Paste below codes in it and save**
* You can change ports if any of them are in use: 8081, 9091, 8080, 9092
* If you want to change `8081` to `35081` then just change the port on the left side like: `35081:8081`
```
services:
  prometheus:
    image: ${PROMETHEUS_IMAGE}
    container_name: ${PROMETHEUS_NAME}
    env_file:
      - .env
    volumes:
      - "${PROMETHEUS_CONFIG_PATH}:/etc/prometheus/prometheus.yml"
    command: 
      - "--enable-feature=expand-external-labels"
      - "--config.file=/etc/prometheus/prometheus.yml"
    ports:
      - "9091:9090"
    networks:
      - chainbase
    restart: unless-stopped

  flink-jobmanager:
    image: ${FLINK_JOBMANAGER_IMAGE}
    container_name: ${FLINK_JOBMANAGER_NAME}
    env_file:
      - .env
    ports:
      - "8081:8081"
    command: jobmanager
    networks:
      - chainbase
    restart: unless-stopped

  flink-taskmanager:
    image: ${FLINK_JOBMANAGER_IMAGE}
    container_name: ${FLINK_TASKMANAGER_NAME}
    env_file:
      - .env
    depends_on:
      - flink-jobmanager
    command: taskmanager
    networks:
      - chainbase
    restart: unless-stopped

  chainbase-node:
    image: ${MAIN_SERVICE_IMAGE}
    container_name: ${MAIN_SERVICE_NAME}
    command: ["run"]
    env_file:
      - .env
    ports:
      - "8080:8080"
      - "9092:9092"
    volumes:
      - "${NODE_ECDSA_KEY_FILE_HOST:-./opr.ecdsa.key.json}:${NODE_ECDSA_KEY_FILE}"
      - "${NODE_LOG_PATH_HOST}:${NODE_LOG_DIR}:rw"
    depends_on:
      - prometheus
      - flink-jobmanager
      - flink-taskmanager
    networks:
      - chainbase
    restart: unless-stopped

networks:
  chainbase:
    driver: bridge
```

**Create folders for docker**
```console
source .env && mkdir -pv ${EIGENLAYER_HOME} ${CHAINBASE_AVS_HOME} ${NODE_LOG_PATH_HOST}
```

**Give permissions to bash script**
```console
chmod +x ./chainbase-avs.sh
```

**update prometheus.yml**
* Replace ${YOUR_OPERATOR_NAME} with your operator address
```console
nano prometheus.yml
```

![image](https://github.com/user-attachments/assets/0cbf8a41-289a-4108-9259-6ae5325c3191)

## Run Chainbase AVS
Register AVS
```console
./chainbase-avs.sh register
```
![image](https://github.com/user-attachments/assets/3934719c-2da0-4ab1-a032-394007ceed4c)


Run AVS
```
./chainbase-avs.sh run
```
![image](https://github.com/user-attachments/assets/174032d6-4949-4568-b758-a2b8379e561e)


## Check AVS health
**Check chainbase-node logs**
```console
docker logs -f chainbase-node
```
![image](https://github.com/user-attachments/assets/2eb1e4c3-4c80-46e7-ac26-aea404d712c0)


**Get your AVS link**
```console
export PATH=$PATH:~/bin

eigenlayer operator status operator.yaml
```
![image](https://github.com/user-attachments/assets/f8a03ca0-e5cc-4e53-813e-f480781e2ec9)


**Check Operator Health**
```console
# If your port is 8080
curl -i localhost:8080/eigen/node/health
```
![image](https://github.com/user-attachments/assets/7ae0e9e7-05d5-482d-8038-18476efd118c)


Check docker containers
* You must have 4 new docker containers
```console
docker ps
```
![image](https://github.com/user-attachments/assets/ff72d128-b3c2-40c9-a18e-9059f7646d22)



## Fill the form
https://forms.gle/w9h8Su87kEnDwRMA7

## Post your Operator address in the Operator Support channel
https://discord.gg/chainbase

![Screenshot (110)](https://github.com/user-attachments/assets/7ae7a98c-c57a-4d4d-854d-59c4b47e4b74)


