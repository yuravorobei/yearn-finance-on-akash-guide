# Deployment guide for Yearn.Finance on Akash DeCloud
![title picture](https://github.com/yuravorobei/iearn-finance-akash-guide/blob/main/media/header.png)

  In this guide for the "Akashian Challenge: Phase 3 Start Guide" I'll show you how to deploy the [Yearn.Finance web-app](https://docs.yearn.finance/) on the [Akash Decentralized Cloud (DeCloud)](https://akash.network/).  
  **Yearn.Finance** is a suite of products in Decentralized Finance (DeFi) that provides lending aggregation, yield generation, and insurance on the Ethereum blockchain. The protocol is maintained by various independent developers and is governed by YFI holders.  
  **Akash** is a marketplace for cloud compute resources which is designed to reduce waste, thereby cutting costs for consumers and increasing revenue for providers. Akash DeCloud is a faster, better, and lower cost cloud built for DeFi, decentralized projects, and high growth companies, providing unprecedented scale, flexibility, and price performance. 10x lower in cost, Akash serverless computing platform is compatible with all cloud providers and all applications that run on the cloud.

This guide is divided into three main parts: dockerizing, installing and preparing Akash, deploying. All actions I perform on a Ubuntu 18.04 operating system. And so we go.

# 1. Dockerizing
The Akash works with a Docker images, so the first thing we need to do to deploy is to get the Docker image of the Yearn.Finance application. If you already have the image or know how to get it, you can skip this step.

### 1.1 Getting the source code
First, you need to get the source code of the Yearn.Finance application on your computer. To do this, clone the [Yearn.Finance GitHub repository](https://github.com/iearn-finance/iearn-finance.git):
  ```bash
  $ git clone https://github.com/iearn-finance/iearn-finance.git
  ```

### 1.2 Creating a server.js
The YearnFinance application is written in Node.JS, so to run it, create a server.js file that defines a web app using the Express.js framework:
  
  ```bash
  $ cd iearn-finance
  $ nano server.js
  ```

And put the following code in it:
  ```javascript
  const express = require('express');
  const path = require('path');
  const app = express();

  const port = 8080;
  const host = '0.0.0.0';

  app.use(express.static(path.join(__dirname, 'build')));

  app.get('/', function (req, res) {
    res.sendFile(path.join(__dirname, 'build', 'index.html'));
  });

  app.listen(port, host);
  ```
Here I've configured web app to work on port `8080`, but in fact you can specify any port here, because later on the deployment stage we can forward the ports using the port mapping rules.

### 1.3 Creating a Dockerfile
Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Create the Dockerfile:
  ```bash
  $ nano Dockerfile
  ```
And put the following code in it:
  ```dockerfile
  # Latest (at the time of writing) LTS version of node available from the Docker Hub
  FROM node:15

  # Working directory for our application
  WORKDIR /usr/src/app

  # Bundle app source
  COPY . .

  # Install app dependencies
  RUN npm install --only=production

  # Builds the app for production to the build folder
  RUN npm run build

  # expose the port that the app uses in the container, the one that we specified in the server.js file
  EXPOSE 8080

  # start node server
  CMD [ "node", "server.js" ]
  ```


### 1.4 Building the Docker image
To build the image run the following command. Here instead of `<your_username>` you must specify your username, and instead of `iearn-finance`, you can use any name.:
  ```bash
  $ docker build -t <your_username>/iearn-finance .
  ```
Waiting for the completion of the function.

### 1.5 Publishing the image
To transfer the created image to the cloud server, you need to share it. To do this I will push the image to the [Docker Hub](https://hub.docker.com/). Log into the Docker Hub from the command line. Replace `<your_hub_username>` with your Docker Hub usesrname.
  ```bash
  $ docker login --username=<your_hub_username>
  ```
Find the image ID using `docker images`:
  ```
  $ docker images

  REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
  yura/iearn-finance   latest              7900fe35b502        32 seconds ago      1.58GB
  ```
My image ID is `7900fe35b502`. Then tag your image with the following command. Replace <image_id> with your image ID and replace `<your_hub_username>` with your Docker Hub usesrname. Any name can be used instead of `iearn-finance`.
  ```
  $ docker tag <image_id> <your_hub_username>/iearn-finance
  ```
Push your image to the Docker Hub repository with the following command:
  ```
  $ docker push <your_hub_username>/iearn-finance
  ```
Waiting for loading.
Dockerization done!



# 2. Installing and preparing Akash
To interact with the Akash system, you need to install it and create a wallet. I will follow the official [Akash Documentation guides](https://docs.akash.network/v/master/).

### 2.1 Choosing a Network
At any given time, there are a number of different active Akash networks running, each with a different akash version, chain-id, seed hosts, etc… Three networks  are currently available: mainnet, testnet and edgenet. In this guide for the "Akashian Challenge: Phase 3 Start Guide" i'll use the `edgenet` network.

So let’s define the required shell variables:
  ```bash
  $ AKASH_NET="https://raw.githubusercontent.com/ovrclk/net/master/edgenet"
  $ AKASH_VERSION="$(curl -s "$AKASH_NET/version.txt")"
  $ AKASH_CHAIN_ID="$(curl -s "$AKASH_NET/chain-id.txt")"
  $ AKASH_NODE="$(curl -s "$AKASH_NET/rpc-nodes.txt" | shuf -n 1)"
  ```

### 2.2 Installing the Akash client
There are several ways to install Akash: from the [release page](https://github.com/ovrclk/akash/releases), from the source or via `godownloader`. As for me, the easiest and the most convenient way is to download via `godownloader`:
  ```bash
  $ curl https://raw.githubusercontent.com/ovrclk/akash/master/godownloader.sh | sh -s -- "$AKASH_VERSION"
  ```
And add the akash binaries to your shell PATH. Keep in mind that this method will only work in this terminal instance and after restarting your computer or terminal this PATH addition will be removed. You can read how to set your PATH permanently [on this page](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux-unix) if you want:
  ```bash
  $ export PATH="$PATH:./bin"
  ```


### 2.3 Wallet setting
Let's define additional shell variables. `KEY_NAME` value of your choice, I uses a value of “crypto”:
  ```bash
  $ KEY_NAME="crypto"
  $ KEYRING_BACKEND="test"
  ```
Derive a new key locally:
  ```bash
  $ akash --keyring-backend "$KEYRING_BACKEND" keys add "$KEY_NAME"
  ```

You'll see a response similar to below:
  ```
  - name: crypto
    type: local
    address: akash1kfd3adu7sgcu2vxd3ucc3pehmrautp25kxenn5
    pubkey: akashpub1addwnpepqvc7x8r2dyula25ucxtawrt39henydttzddvrw6xz5gvcee4arfp7ppe6md
    mnemonic: ""
    threshold: 0
    pubkeys: []


  **Important** write this mnemonic phrase in a safe place.
  It is the only way to recover your account if you ever forget your password.

  share mammal walnut direct plug drive cruise real pencil random regret chunk use live entire gloom donate require autumn bid brown impact scrap slab
  ```
In the above example, your new Akash address is `akash1kfd3adu7sgcu2vxd3ucc3pehmrautp25kxenn5`, define a new shell variable with it:
  ```bash
  $ ACCOUNT_ADDRESS="akash1kfd3adu7sgcu2vxd3ucc3pehmrautp25kxenn5"
  ```

### 2.4 Funding your account
In this guide I use edgenet Akash network. Non-mainnet networks will often times have a "faucet" running - a server that will send tokens to your account. You can see the faucet url by running:
  ```bash
  $ curl "$AKASH_NET/faucet-url.txt"

  https://akash.vitwit.com/faucet
  ```
Go to the resulting URL and enter your account address; you should see tokens in your account shortly.
Check your account balance with:
  ```bash
  $ akash --node "$AKASH_NODE" query bank balances "$ACCOUNT_ADDRESS"

  balances: 
  - amount: "100000000" 
   denom: uakt 
  pagination: 
   next_key: null 
   total: "0"
  ```
As you can see, the balance is non-zero and contains 100000000 uakt.
Okay, now you're ready to deploy the application.



# 3. Deploying
You should have all the shell variables defined in the previous step.


### 3.1 Creating a deployment configuration
For configuration in Akash uses [Stack Definition Language (SDL)](https://docs.akash.network/v/master/documentation/sdl). Deployment services, datacenters, pricing, etc.. are described by a YAML configuration file. These files may end in .yml or .yaml.
Create deploy.yml configuration file with following content.  
In the `services.images` field replace value `<your_hub_username>/iearn-finance` with your Docker image name.  
The `services.expose.port` field means the container port to expose. In the first section of the guide I specified port `8080`.  
The `services.expose.as` field is the port number to expose the container port as specified. Port `80` is the standard port for web servers HTTP protocol.  
The `resources.storage.size` filed is the required hard disk space for the application. The size of my Docker image is 1.58GB, so I rounded it up with a margin to 2GB and specified it in the configuration as `2Gi`.

  ```yaml
  $ cat > deploy.yml <<EOF
  ---
  version: "2.0"

  services:
    web:
      image: <your_hub_username>/iearn-finance
      expose:
        - port: 8080
          as: 80
          to:
            - global: true

  profiles:
    compute:
      web:
        resources:
          cpu:
            units: 0.1
          memory:
            size: 512Mi
          storage:
            size: 2Gi
    placement:
      westcoast:    
        pricing:
          web: 
            denom: uakt
            amount: 9000

  deployment:
    web:
      westcoast:
        profile: web
        count: 1
  EOF
  ```


### 3.2 Creating the deployment
In this step, you post your deployment, the Akash marketplace matches you with a provider via auction. To deploy on Akash, run:
  ```bash
  $ akash tx deployment create deploy.yml --from $KEY_NAME \
    --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID \
    --keyring-backend $KEYRING_BACKEND -y
  ```
Make sure there are no errors in the command response. The error information will be in the `raw_log` responce field, you can also check the transaction status by transaction hash on the [explorer](https://testnet.akash.aneka.io/).


### 3.3 Wait for your lease
You can check the status of your lease by running:
  ```
  $ akash query market lease list --owner $ACCOUNT_ADDRESS \
    --node $AKASH_NODE --state active
  ```
  You should see a response similar to:
  ```
  - lease_id:
      dseq: "123991"
      gseq: 1
      oseq: 1
      owner: akash1kfd3adu7sgcu2vxd3ucc3pehmrautp25kxenn5
      provider: akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7
    price:
      amount: "718"
      denom: uakt
    state: active
  pagination:
    next_key: null
    total: "0"
  ```

From this response we can extract some new required for future referencing shell variables:
  ```bash
  $ PROVIDER="akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7"
  $ DSEQ="123991"
  $ OSEQ="1"
  $ GSEQ="1"
  ```


### 3.4 Uploading manifest
Upload the manifest using the values from above step:
  ```bash
  $ akash provider send-manifest deploy.yml \
    --node $AKASH_NODE --dseq $DSEQ \
    --oseq $OSEQ --gseq $GSEQ \
    --owner $ACCOUNT_ADDRESS --provider $PROVIDER
  ```
Your image is deployed, once you uploaded the manifest. You can retrieve the access details by running the below:
  ```bash
  $ akash provider lease-status \
    --node $AKASH_NODE --dseq $DSEQ \
    --oseq $OSEQ --gseq $GSEQ \
    --provider $PROVIDER --owner $ACCOUNT_ADDRESS
  ```
You should see a response similar to:
  ```
  {
    "services": {
      "web": {
        "name": "web",
        "available": 1,
        "total": 1,
        "uris": [
          "vbr0pmrhftbmddsanj9ssftb60.provider2.akashdev.net"
        ],
        "observed-generation": 0,
        "replicas": 0,
        "updated-replicas": 0,
        "ready-replicas": 0,
        "available-replicas": 0
      }
    },
    "forwarded-ports": {}
  }
  ```
You can access the application by visiting the hostnames mapped to your deployment. In above example, its http://vbr0pmrhftbmddsanj9ssftb60.provider2.akashdev.net. Following the address, make sure that the application works:
![result1](https://github.com/yuravorobei/iearn-finance-akash-guide/blob/main/media/deployed_1.png)
![result2](https://github.com/yuravorobei/iearn-finance-akash-guide/blob/main/media/deployed_2.png)


### 3.5 Close your deployment

When you are done with your application, close the deployment. This will deprovision your container and stop the token transfer. Close deployment using deployment by creating a deployment-close transaction:
  ```shell
  $ akash tx deployment close --node $AKASH_NODE \
    --chain-id $AKASH_CHAIN_ID --dseq $DSEQ \
    --owner $ACCOUNT_ADDRESS --from $KEY_NAME \
    --keyring-backend $KEYRING_BACKEND -y
  ```

Additionally, you can also query the market to check if your lease is closed:
  ```bash
  $ akash query market lease list --owner $ACCOUNT_ADDRESS --node $AKASH_NODE
  ```
You should see a response similar to:
  ```
  - lease_id:
      dseq: "123991"
      gseq: 1
      oseq: 1
      owner: akash1kfd3adu7sgcu2vxd3ucc3pehmrautp25kxenn5
      provider: akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7
    price:
      amount: "718"
      denom: uakt
    state: closed
  pagination:
    next_key: null
    total: "0"
  ```
As you can notice from the above, you lease will be marked `closed`.

