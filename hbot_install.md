# Setting Up Hummingbot with Gateway

This guide provides step-by-step instructions for setting up Hummingbot with Gateway on a Linux system. Follow these steps to install dependencies, clone repositories, configure Hummingbot and Gateway, connect to exchanges, and run a trading script.

## Table of Contents
1. [Install System Packages and Dependencies](#1-install-system-packages-and-dependencies)
2. [Configure Firewalls](#2-configure-firewalls)
3. [Create Directory and Clone Repositories](#3-create-directory-and-clone-repositories)
4. [Prepare Hummingbot](#4-prepare-hummingbot)
   - [Fetch and Checkout Hummingbot Branch](#a-fetch-and-checkout-hummingbot-branch)
   - [Run the Installation Script](#b-run-the-installation-script)
   - [Activate the Hummingbot Environment](#c-activate-the-hummingbot-environment)
   - [Compile Hummingbot Source Code](#d-compile-hummingbot-source-code)
   - [Start Hummingbot](#e-start-hummingbot)
   - [Initialize Hummingbot](#f-initialize-hummingbot)
   - [Set Up Certificates](#g-set-up-certificates-use-a-passphrase-without-special-characters)
5. [Prepare Gateway](#5-prepare-gateway)
   - [Fetch and Checkout Gateway Branch](#a-fetch-and-checkout-gateway-branch)
   - [Install and Build Gateway Dependencies](#b-install-and-build-gateway-dependencies)
   - [Run Gateway Setup Script](#c-run-gateway-setup-script)
   - [Start Gateway with Concealed Passphrase](#d-start-gateway-with-concealed-passphrase)
6. [Connect to Exchanges from Hummingbot](#6-connect-to-exchanges-from-hummingbot)
   - [Create Maya Wallet in gateway](#a-create-maya-wallet-in-gateway-when-it-asks-for-private-key-use-mnemonic-phrase)
   - [Connect Maya Exchange Wrapper](#b-connect-maya-exchange-wrapper)
   - [Connect to Hyperliquid Perpetual Exchange](#c-connect-to-hyperliquid-perpetual-exchange)
   - [Verify Connections](#d-verify-connections)
7. [Run Hummingbot Maya v2 XEMM Script](#7-run-hummingbot-maya-v2-xemm-script)
   - [Create Configuration File](#a-create-configuration-file)
   - [Run Hummingbot](#b-run-hummingbot)

## 1. Install System Packages and Dependencies

The following packages are required:

| Package         | Description                               | Minimum Version |
|-----------------|-------------------------------------------|-----------------|
| build-essential | Essential tools for compiling software    | Latest          |
| Node.js         | JavaScript runtime for Gateway            | 20.x            |
| Miniconda3      | Python environment manager for Hummingbot | Latest          |
| pnpm            | Package manager for Gateway dependencies  | Latest          |

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
sudo npm install -g pnpm
```

## 2. Configure Firewall

## 2.1 The following firewall rules are required:

| Source IP | Destination IP | Port  | Protocol | Purpose                           |
|-----------|----------------|-------|----------|-----------------------------------|
| 10.0.0.10 | Any            | 22    | TCP      | Allow SSH access for management   |
| 127.0.0.1 | 127.0.0.1      | 15888 | TCP      | Allow local Gateway communication |

```bash
sudo ufw allow from 10.0.0.10 to any port 22 proto tcp
sudo ufw allow from 127.0.0.1 to 127.0.0.1 port 15888 proto tcp
sudo ufw enable
```

> **Note**: Replace `10.0.0.10` with the IP address of the system or network allowed to access your server via SSH. Use `0.0.0.0/0` to allow all IPs, but this is less secure.

## 2.2 Enable logging
```bash
sudo ufw logging on
sudo ufw reload
```

**Verify** the firewall configuration to ensure the rules are active:

```bash
sudo ufw status verbose
```
Monitor logging:
```bash
sudo journalctl -u ufw
```


## 3. Create Directory and Clone Repositories

Create an `hbot` directory and clone the repositories:

```bash
mkdir -p hbot
cd hbot
git clone git@github.com:Homeric-Freedom/gateway.git
git clone git@github.com:Homeric-Freedom/hummingbot.git
```

## 4. Prepare Hummingbot

Configure and install Hummingbot.

### 4.1 Fetch and Checkout Hummingbot Branch

```bash
cd hummingbot
git fetch origin dev
git checkout dev
```

### 4.2 Run the Installation Script

```bash
./install
```

### 4.3 Activate the Hummingbot Environment

```bash
source ~/.bashrc
conda activate hummingbot
```

### 4.4 Compile Hummingbot Source Code

```bash
./compile
```

### 4.5. Start Hummingbot

```bash
./start
```

### 4.6. Initialize Hummingbot

Follow the Hummingbot initialization process to create a password.

### 4.7 Set Up Certificates, Using a Passphrase without Special Characters

```bash
gateway generate-certs
```

## 5. Prepare Gateway

Configure and install Gateway.

### 5.1. Fetch and Checkout Gateway Branch

```bash
cd gateway
git fetch origin dev
git checkout dev
```

### 5.2 Install and Build Gateway Dependencies

```bash
pnpm install
pnpm build
```
If there have been updates to the branch, use:

```
pnpm install
pnpm refresh-templates
pnpm build
```
### 5.3 Run Gateway Setup Script

```bash
pnpm run setup
```

### 5.4 Start Gateway with Concealed Passphrase
To start the Gateway server securely without storing the passphrase in cleartext or exposing it in command-line arguments, use a Bash script that prompts for the passphrase interactively, concealing the input, and passes it via an environment variable.

Create `start-gateway.sh` in `hbot` directory:

```bash
#!/bin/bash

# Change to the gateway directory
cd gateway || { echo "Error: Failed to change to gateway directory."; exit 1; }

# Prompt for passphrase securely (input is hidden, not stored in shell history)
read -s -p "Enter Gateway passphrase: " GATEWAY_PASSPHRASE
echo ""

# Validate that a passphrase was entered
if [ -z "$GATEWAY_PASSPHRASE" ]; then
    echo "Error: No passphrase entered. Exiting."
    exit 1
fi

# Export the passphrase as an environment variable for Gateway
export GATEWAY_PASSPHRASE

# Start Gateway
pnpm start

# Clear the environment variable for added security
unset GATEWAY_PASSPHRASE

```

Make the script executable:
```bash
chmod +x start-gateway.sh
```

Run the script to start Gateway:
```bash
./start-gateway.sh
```

## 6. Connect to Exchanges from Hummingbot

### a. Create Maya Wallet in gateway (When It Asks for Private Key Use Mnemonic Phrase) 
 
```bash
gateway connect mayadex
```
- Which mayachain-based network do you want to connect to >>> mainnet
- Do you want to continue to use node url 'None' for mayachain-mainnet? >> Yes
### NOTE: if you get the following then you need to delete your gateway/conf/wallets folder and hummingbot/conf/gateway_connections.json file
- Do you want to connect to mayachain-mainnet with one of your existing wallets on Gateway? (Yes/No) >>> ctrl + c and delete the specified files. 
- Enter your mayachain-mainnet wallet private key >>> {Enter your mnemonic phrase}

### b. Connect Maya Exchange Wrapper

Use the following command and enter your Maya address (Make sure this matches the wallet configured above):

```bash
connect maya
```

> Currently, `connect maya` defaults to the first maya wallet configured in Gateway.

### c. Connect to Hyperliquid Perpetual Exchange

Use the `connect` command with your Hyperliquid credentials:

- Enter your Arbitrum wallet private key.
- When prompted, select `No` for using the vault address.
- Enter your Arbitrum address.

> **Note**: Refer to the Hummingbot CLI entry in KeePass for credentials.

### d. Verify Connections

Check the status of both connections using:

```bash
balance
```

## 7. Run Hummingbot Maya v2 XEMM Script

Run the Maya v2 XEMM trading script with the desired configuration.

### a. Create Configuration File

Create a configuration file at `conf/scripts/maya_v2_xemm.yml` with your desired settings.

```bash
##########################################                                      
###   Multi Leg Arbitrage strategy config   ###                                 
##########################################                                      
                                                                                
template_version: 6                                                             
strategy: multi_leg_arb                                                         
                                                                                
# Minimum percentage profit to trigger trading                                  
min_profit: 1.0                                                                 
                                                                                
# Maximum acceptable percentage loss                                            
max_loss: 2.0                                                                   
                                                                                
# Optional default order amount                                                 
order_amount: 5.0                                                               
                                                                                
# Routes string separated by commas, legs separated by spaces, params by :      
# Expected format: side:connector:pair[:amount]                                 
#                                                                               
# Definitions:                                                                  
#   side      = B (buy) or S (sell)                                             
#   connector = valid exchange or connector name                                
#   pair      = trading pair (e.g SOL-USDC)                                     
#   amount    = optional; only used in the first leg                            
#                                                                               
# Example:                                                                      
#   B:mexc:SOL-USDC:1 S:mexc:SOL-USDT, B:mexc:LINK-ETH:1 S:mexc:LINK-USDC       
routes: S:mayadex/swap:THOR.RUNE-BTC/BTC:40 S:hyperliquid_perpetual:BTC-USD     
  B:hyperliquid_perpetual:RUNE-USD,S:mayadex/swap:BTC/BTC-THOR.RUNE:0.0003      
  S:hyperliquid_perpetual:RUNE-USD B:hyperliquid_perpetual:BTC-USD              
                                                                                
# Optional token aliases mapping (alias=TOKEN), used to normalize token names when
# validating route continuity. Example maps RUNE.THOR to RUNE and BTC to BTC/BTC.
token_aliases: THOR.RUNE=RUNE,BTC=BTC/BTC                      
```

### b. Run Hummingbot  

#### 1. Within Hummingbot    

```bash
start --script maya_v2_xemm.py --conf maya_v2_xemm.yml
````
#### 2. With the Script in terminal

```bash
python bin/hummingbot_quickstart.py --script-conf maya_v2_xemm.yml --config-file-name maya_v2_xemm.py
```
