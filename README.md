## Lava Testnet Node Installation Guide

### Introduction
This guide provides instructions for manually installing a node and joining the Lava Testnet. Note that this guide does not use the "Cosmovisor" tool, so you will need to upgrade your node incrementally as described in the upgrades section.

### Prerequisites
1. **Ensure hardware requirements are met.**
2. **Install required packages and dependencies.**

### Install Required Packages
Run the following commands to update your system and install dependencies. If you encounter permission errors, try running the commands with `sudo`.

```bash
sudo apt update
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Create a temporary directory for the installation:

```bash
temp_dir=$(mktemp -d) && cd $temp_dir
```

### Go Installation
Set up and install Go:

```bash
go_package_url="https://go.dev/dl/go1.20.5.linux-amd64.tar.gz"
go_package_file_name=${go_package_url##*\/}

# Download the Go archive
wget -q $go_package_url

# Extract the archive
sudo tar -C /usr/local -xzf $go_package_file_name

# Set up environment variables
echo "export PATH=\$PATH:/usr/local/go/bin" >> ~/.profile
echo "export PATH=\$PATH:\$(go env GOPATH)/bin" >> ~/.profile
source ~/.profile
```

Verify Go installation:

```bash
go version
go env GOPATH
echo $PATH
```

### Set Up a Local Node
1. **Download Configuration Files**

```bash
git clone https://github.com/lavanet/lava-config.git
cd lava-config/testnet-2
source setup_config/setup_config.sh
```

2. **Copy `lavad` Configuration Files**

```bash
echo "Lava config file path: $lava_config_folder"
mkdir -p $lavad_home_folder
mkdir -p $lava_config_folder
cp default_lavad_config_files/* $lava_config_folder
```

3. **Add Genesis File**

```bash
cp genesis_json/genesis.json $lava_config_folder/genesis.json
```

### Join the Lava Testnet
1. **Install the `lavad` Binary**

```bash
lavad_binary_path="$HOME/go/bin/"
mkdir -p $lavad_binary_path
wget -O ./lavad "https://github.com/lavanet/lava/releases/download/v0.21.1.2/lavad-v0.21.1.2-linux-amd64"
chmod +x lavad
cp lavad /usr/local/bin
```

Ensure the binary is accessible in your PATH:

```bash
lavad --help
```

2. **Create and Start `lavad` Service**

Create a `systemd` service file:

```bash
echo "[Unit]
Description=Lava Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lavad) start --home=$lavad_home_folder --p2p.seeds $seed_node
Restart=always
RestartSec=180
LimitNOFILE=infinity
LimitNPROC=infinity
[Install]
WantedBy=multi-user.target" > lavad.service

sudo mv lavad.service /lib/systemd/system/lavad.service
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable lavad.service
sudo systemctl restart systemd-journald
sudo systemctl start lavad
```

Check the service status:

```bash
sudo systemctl status lavad
sudo journalctl -u lavad -f
```

### Upgrades
When an upgrade is required, follow these steps:

1. **Stop the Current `lavad` Process**

```bash
sudo systemctl stop lavad
```

2. **Download and Replace the Binary**

```bash
temp_dir=$(mktemp -d) && cd $temp_dir
required_upgrade_name="v0.21.1.2" # Change this to the required version
upgrade_binary_url="https://github.com/lavanet/lava/releases/download/$required_upgrade_name/lavad-$required_upgrade_name-linux-amd64"

wget "$upgrade_binary_url" -q -O $temp_dir/lavad
chmod +x $temp_dir/lavad
sudo cp $temp_dir/lavad $(which lavad)
```

3. **Restart the `lavad` Service**

```bash
sudo systemctl start lavad
```

4. **Verify the Node Sync Status**

```bash
lavad status | jq .SyncInfo.catching_up
sudo journalctl -u lavad -f
```

### Conclusion
Your node should now be up and running on the Lava Testnet. If you encounter any issues or have questions, feel free to seek support!
