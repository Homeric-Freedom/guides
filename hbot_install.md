# Setting Up Hummingbot with Gateway

This guide provides step-by-step instructions for setting up Hummingbot with Gateway on a Linux system. Follow these steps to install dependencies, clone repositories, configure Hummingbot and Gateway, connect to exchanges, and run a trading script.

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

Set up an `hbot` directory for Hummingbot and Gateway, and clone their repositories.

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

## 6. Connect to Exchanges

### a. Connect to Maya

Use the following command and enter any Maya address (the address is not critical for this step):

```bash
connect maya
```

### b. Connect to Hyperliquid Perpetual Exchange

Use the `connect` command with your Hyperliquid credentials:

- Enter your Arbitrum wallet private key.
- When prompted, select `No` for using the vault address.
- Enter your Arbitrum address.

> **Note**: Refer to the Hummingbot CLI entry in KeePass for credentials.

### c. Verify Connections

Check the status of both connections using:

```bash
balance
```

## 7. Run Hummingbot Maya v2 XEMM Script

Run the Maya v2 XEMM trading script with the desired configuration.

### a. Create Configuration File

Create a configuration file at `conf/scripts/maya_v2_xemm.yml` with your desired settings.

### b. create start-hbot.sh script in hummingbot's parent directory

``` bash
#!/bin/bash

STARTUP_COMMAND="python bin/hummingbot_quickstart.py --script-conf maya_v2_xemm.yml --config-file-name maya_v2_xemm.py"
SECRET_ENV_VARIABLE_NAME="CONFIG_PASSWORD"
TARGET_DIRECTORY="hummingbot"

# Change to the target directory
cd "$TARGET_DIRECTORY" || { echo "Error: Failed to change to $TARGET_DIRECTORY directory."; exit 1; }

# Prompt for secret securely (input is hidden, not stored in shell history or a file)
read -s -p "Enter secret for $SECRET_ENV_VARIABLE_NAME: " SECRET_VALUE
echo ""

# Validate that a secret was entered
if [ -z "$SECRET_VALUE" ]; then
    echo "Error: No secret entered. Exiting."
    exit 1
fi

# Export the secret as an environment variable
export $SECRET_ENV_VARIABLE_NAME="$SECRET_VALUE"

# Execute the startup command with variable substitution
eval "$STARTUP_COMMAND"

# Clear the environment variable for added security
unset $SECRET_ENV_VARIABLE_NAME
```

### c. Execute the script:
```bash
bash start-hbot.sh
```
