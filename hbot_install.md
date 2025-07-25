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

Ensure your system has the necessary packages and tools installed.

- `build-essential`
- Node.js 20+
- Miniconda3
- `pnpm` package manager

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
sudo npm install -g pnpm
```

## 2. Configure Firewalls
Ensure that the firewall allows traffic on the necessary ports for Hummingbot and Gateway.
```bash
sudo ufw allow from 10.0.0.10 to any port 22 proto tcp
sudo ufw allow from 127.0.0.1 to 127.0.0.1 port 15888 proto tcp
sudo ufw enable
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

### a. Fetch and Checkout Hummingbot Branch

```bash
cd hummingbot
git fetch origin pull/95/head:pr-95
git checkout pr-95
git reset --hard origin/pr-95
```

If new code was pushed:  

```bash
git branch -D pr-95
git fetch origin pull/95/head:pr-95
git checkout pr-95
```

### b. Run the Installation Script

```bash
./install
```

### c. Activate the Hummingbot Environment

```bash
source ~/.bashrc
conda activate hummingbot
```

### d. Compile Hummingbot Source Code

```bash
./compile
```

### e. Start Hummingbot

```bash
./start
```

### f. Initialize Hummingbot

Follow the Hummingbot initialization process to create a password.

### g. Set Up Certificates (use a passphrase without special characters)

```bash
gateway generate-certs
```

## 5. Prepare Gateway

Configure and install Gateway.

### a. Fetch and Checkout Gateway Branch

```bash
cd gateway
git fetch origin pull/33/head:pr-33
git checkout pr-33
```
If there have been updates to the PR branch, use:
```bash
git branch -D pr-33
git fetch origin pull/33/head:pr-33
pnpm build
```

### b. Install and Build Gateway Dependencies

Install dependencies defined in `requirements.txt`.

```bash
pnpm install
pnpm build
```

### c. Run Gateway Setup Script

```bash
pnpm run setup
```

### d. Start Gateway with Concealed Passphrase
To start the Gateway server securely without storing the passphrase in cleartext or exposing it in command-line arguments, use a Bash script that prompts for the passphrase interactively, concealing the input, and passes it via an environment variable.

#### 1. Create `start-gateway.sh` in `hbot` directory:

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
#### 2. Make the script executable:
```bash
chmod +x start-gateway.sh
```
#### 3. Run the script to start Gateway:
```bash
./start-gateway.sh
```

## 6. Connect to Exchanges from Hummingbot

### a. Create Maya Wallet in gateway (When It Asks for Private Key Use Mnemonic Phrase 
 
```bash
gateway connect mayadex
```
- Which mayachain-based network do you want to connect to >>> mainnet
- Do you want to continue to use node url 'https://midgard.mayachain.info' for mayachain-mainnet? >> Yes
- Do you want to connect to mayachain-mainnet with one of your existing wallets on Gateway? (Yes/No) >>> No
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
markets: {}
candles_config: []
controllers_config: []
script_file_name: maya_v2_xemm.py
maker_connector: maya
maker_trading_pairs:
# triangular pairs
#- ETH-BTC
#- ETH-WBTC
#- RUNE-BTC
#- RUNE-ETH
#- RUNE-WBTC
# ETH/stable pairs
- BTC-sUSDC
- BTC-sUSDT
- ETH-sUSDC
- ETH-sUSDT
- RUNE-sUSDC
- RUNE-sUSDT
#- WBTC-sUSDC
#- WBTC-sUSDT
# ARB/stable pairs
#- BTC-USDT
#- ETH-USDT
#- RUNE-USDT
#- WBTC-USDT

taker_connector: hyperliquid_perpetual
taker_trading_pairs:
# triangular pairs
#- ETH-USD
#- ETH-USD
#- RUNE-USD
#- RUNE-USD
#- RUNE-USD
# ETH/stable pairs
- BTC-USD
- BTC-USD
- ETH-USD
- ETH-USD
- RUNE-USD
- RUNE-USD
#- BTC-USD
#- BTC-USD
# ARB/stable pairs
#- BTC-USD
#- ETH-USD
#- RUNE-USD
#- BTC-USD

target_profitability: 0.006
min_profitability: 0.0025
max_profitability: 0.0098
order_amount_quote: 220
```

### b. Run Hummingbot  

#### 1. Within Hummingbot    

```bash
start --script maya_v2_xemm.yml --conf maya_v2_xemm.py
````
#### 2. With the Script in terminal

```bash
python bin/hummingbot_quickstart.py --script-conf maya_v2_xemm.yml --config-file-name maya_v2_xemm.py
```
