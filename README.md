# Step-By-Step Guide on setting up and running an Ethereum validator node using Google Cloud Infrastructure

An Ethereum node validator is a component within the Ethereum network that participates in the Ethereum 2.0 consensus mechanism, specifically in the Proof of Stake (PoS) protocol. Validators play a crucial role in validating transactions, creating new blocks, and securing the network by staking a certain amount of cryptocurrency (in this case, ETH) as collateral.

Here's a breakdown of the key concepts:

1. **Ethereum 2.0 and Proof of Stake (PoS):** Ethereum 2.0 is an upgrade to the Ethereum network aimed at improving scalability, security, and sustainability. One of the major changes in Ethereum 2.0 is the shift from the current Proof of Work (PoW) consensus mechanism to Proof of Stake (PoS). PoS relies on validators to create and validate new blocks, rather than miners as in PoW.

2. **Validator Nodes:** A validator node is a participant in the Ethereum 2.0 network that helps maintain the blockchain by proposing and validating new blocks. Validators are responsible for attesting to the validity of transactions and blocks. Unlike traditional miners, validators do not solve complex mathematical puzzles; instead, they are chosen to validate transactions based on the amount of cryptocurrency they stake.

3. **Staking and Collateral:** To become a validator, individuals or entities must stake a certain amount of ETH as collateral. This collateral acts as a security deposit to ensure that validators follow the rules and do not attempt to validate fraudulent transactions. If a validator behaves maliciously or goes offline, they risk losing a portion of their staked ETH.

4. **Attestations and Block Proposal:** Validators participate in attesting to the validity of blocks by signing messages indicating their agreement with the proposed block. These attestations contribute to reaching consensus and finalizing blocks. Validators also take turns proposing new blocks to add to the blockchain. The PoS protocol ensures that validators are selected to propose blocks and validate transactions based on their staked amount and participation.

5. **Rewards and Penalties:** Validators are rewarded with additional ETH for their active participation and accurate validation of transactions. However, if a validator behaves dishonestly, goes offline frequently, or attempts to manipulate the system, they can be penalized by losing a portion of their staked ETH.

6. **Network Security and Decentralization:** Validator nodes contribute to the overall security and decentralization of the Ethereum network. A distributed network of validators helps prevent centralization of power and reduces the energy consumption associated with traditional PoW mining.

The following provides you with a step by step guide on implementing and maintaining a secure Ethereum Staking Node using the currently recommended software versions.

It is divided into three parts: 
1. Installation
2. Monitoring with Grafana and Prometheus
3. Maintenance

## Part 1: Installation

Installation is a 5 step process. From start to finish, time to complete these steps can take up to a few hours. Fully syncing your the node can take a day or two. Here are the following steps;
- Step 1: Prerequisites
- Step 2: Configuring Node
- Step 3: Installing execution client
- Step 4: Installing consensus client
- Step 5: Installing Validator

### Step 1: Prerequisites

- Acquire some hardware (laptop, desktop, server) or rent a VPS (cloud server): You need to run a node to stake.
- Sync an execution layer client
- Sync a consensus layer client
- Generate your validator keys and import them into your validator client
- Monitor and maintain your node
A Ethereum node consists of the Execution Layer and a Consensus Layer.

**Diagrammatic Explanation:**

![Pasted image 20230808224201](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/ddac1cc9-6df8-42fd-9348-9fe78f7fb924)

(Reference : [CoinCashew](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet#2-signup-to-be-a-validator-at-the-launchpad))



![Pasted image 20230808224245](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/8c2c0bae-8604-4505-9be8-1e3795282157)

Representation of Execution / Consensus / Validator (Reference : [CoinCashew](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet#2-signup-to-be-a-validator-at-the-launchpad))

#### **Minimum Node Setup Requirements**

- **Operating system:** 64-bit Linux (i.e. Ubuntu 22.04.1 LTS Server or Desktop)
- **Processor:** Dual core CPU, Intel Core i5–760 or AMD FX-8100 or better
- **Memory:** 16GB RAM
- **Storage:** 2TB SSD
- **Internet:** Stable broadband internet connection with speeds at least 5 Mbps upload and download.
- **Internet Data Plan**: At least 2 TB per month.
- **Power:** Reliable electrical power. Mitigate with an Uninterrupted Power Supply
- **ETH balance:** at least 32 ETH and some ETH for deposit transaction fees
- **Wallet**: Metamask installed

#### **Setup Ubuntu and Metamask**

With your local or remote node, now you need to install an Operating System. This guide is designed for Ubuntu 22.04.1 LTS.

- To install **Ubuntu Server or Desktop**, refer to this [guide](https://docs.ethstaker.cc/ethstaker-knowledge-base/tutorials/installing-linux)
- When the time comes to make your validator's 32ETH deposit(s), you'll need a wallet to transfer funds to the beacon chain deposit contract. To install Metamask, refer to this [guide](https://www.coincashew.com/wallets/browser-wallets/metamask-ethereum)

#### Ethereum Node Validator Overview
A staking node validator hosts three main components in two layers: consensus layer consists of a consensus client, also known as a validator client with a beacon chain client. The execution layer consists of a execution client, formerly a eth1 node. They are:
**Validator client** - Responsible for producing new blocks and attestations in the beacon chain and shard chains.
**Consensus client** - Responsible for managing the state of the beacon chain, validator shuffling, and more.
**Execution client** - Supplies incoming validator deposits from the eth mainnet chain to the beacon chain client.


### Step 2: Configuring Node

**Logging to the node**
**Using Ubuntu Server**: Begin by connecting with your SSH client.
```bash
ssh username@staking.node.ip.address
```

**Using Ubuntu Desktop**: You're likely in-front of your **local** node. Simply open a terminal window from anywhere by typing Ctrl+Alt+T.

**Updating the node**
Ensure all the latest packages, tools and patches are installed first, then reboot.
```bash
sudo apt-get update -y && sudo apt dist-upgrade -y
sudo apt-get install git ufw curl ccze -y
sudo apt-get autoremove
sudo apt-get autoclean
sudo reboot
```

#### Security Configuration
A. **Create a non-root user with sudo privileges**
Create a new user called `ethereum`
```bash
sudo useradd -m -s /bin/bash ethereum
```

Set the password for ethereum user
```bash
sudo passwd ethereum
```

Add ethereum to the sudo group
```bash
sudo usermod -aG sudo ethereum
```

Log out and log back in as this new user.

**Using Ubuntu Server**: Use the following commands.
```bash
exit
ssh ethereum@staking.node.ip.address
```

**Using Ubuntu Desktop**: Log out can be found in the top right corner under the Power Icon. Click the `ethereum` user account and enter password.

B. Creating a new SSH key
Create a new SSH key pair on **your client machine (i.e. local laptop)**. Run this on **your client machine,** not remote node. 
```bash
ssh-keygen -t ed25519 -C "name@email.com"
```

You'll see this next:
`Generating public/private ed25519 key pair.
`Enter file in which to save the key (/home/<myUserName>/.ssh/id_ed25519):`

Here you're asked to type a file name in which to save the SSH private key. If you press enter, you can use the default file name `id_ed25519`

Next, you're prompted to enter a passphrase.
```
Enter passphrase (empty for no passphrase):
```

A **passphrase** adds an extra layer of protection to your SSH private key. Everytime you connect via SSH to your remote node, enter this passphrase to unlock your SSH private key.

Passphrase is highly recommended! Do not leave this empty for no passphrase.

💡 Do not forget or lose your passphrase. Save this to a password manager.

**Location**: Your SSH key pair is stored in your home directory under `~/.ssh`

**File name:** If your default keyname is`id_ed25519`, then
- your **private SSH key** is `id_ed25519`
- your **public SSH key** is `id_ed25519.pub`

🔥 **IMPORTANT:** Make multiple backup copies of your **private SSH key file** to external storage, such as a USB backup key, for recovery purposes. Also backup your **passphrase**!

Verify the contents of your private SSH key file before moving on.
```bash
cat ~/.ssh/id_ed25519
```

It should look similar to this example.
```bash
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBAblzWLb7/0o62FZf9YjLPCV4qFhbqiSH3TBvZXBiYNgAAAJCWunkulrp5
LgAAAAtzc2gtZWQyNTUxOQAAACBAblzWLb7/0o62FZf9YjLPCV4qFhbqiSH3TBvZXBiYNg
AAAEAxT+yCmifGWgbFnkauf0HyOAJANhYY5EElEX8fI+M4B0BuXNYtvv/SjrYVl/1iMs8J
XioWFuqJIfdMG9lcGJg2AAAACWV0aDJAZXRoMgECAwQ=
-----END OPENSSH PRIVATE KEY-----
```

**Transferring the SSH Public Key to Remote node**
a. Transferring with ssh-copy-id
Works with Linux or MacOS. 
```bash
ssh-copy-id -i ~/.ssh/id_ed25519 ethereum@staking.node.ip.address
```

b. Copying the key manually (For Windows)
Open a command prompt (Windows Key + R, then `cmd`, finally press enter).
```cmd
type %USERPROFILE%\.ssh\id_ed25519.pub
```

The output will look similar to the following:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAoc78lv+XDh2znunKXUF/9zBNJrM4Nh67yut9RN14SX name@email.com
```

Copy into your clipboard this output, also known as your public SSH key.
On your **remote node**, run the following:
```bash
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```

First, a directory called **.ssh** is created, then `Nano` is a text editor for editing a special file called **authorized_keys**
With nano opening the authorized_keys file, right-click your mouse to paste your public SSH key into this file.
To exit and save, press `Ctrl` + `X`, then `Y`, then `Enter`.

Verify your public SSH key was properly pasted into the file.
```bash
cat ~/.ssh/authorized_keys
```

#### Disabling Password Authentication
Disabling root login and password based login

With SSH key authentication enabled, there's still the possibility to connect to your remote node with login and password, a much less secure and brute force-able attack vector.

a. Login via ssh with your new ethereum user
```bash
ssh ethereum@staking.node.ip.address
```

b. Edit the ssh configuration file
```bash
sudo nano /etc/ssh/sshd_config
```

c. Locate **PubkeyAuthentication** and update to yes. 
```bash
PubkeyAuthentication yes
```

d. Locate **PasswordAuthentication** and update to no. 
```bash
PasswordAuthentication no
```

5. Locate **PermitRootLogin** and update to prohibit-password. 
```bash
PermitRootLogin prohibit-password
```

6. Locate **PermitEmptyPasswords** and update to no. 
```bash
PermitEmptyPassword no
```

To exit and save, press `Ctrl` + `X`, then `Y`, then `Enter`.

7. Validate the syntax of your new SSH configuration.
```bash
sudo sshd -t
```

If no errors with the syntax validation, restart the SSH process.
```bash
sudo systemctl restart sshd
```

Verify the login still works.
```bash
ssh ethereum@staking.node.ip.address
```

**Optional**: Make logging in easier by updating your local ssh config.

To simplify the ssh command needed to log in to your server, consider updating on your local client machine the `$HOME/myUserName/.ssh/config` file:
```bash
Host ethereum-server
	User ethereum
	HostName <staking.node.ip.address>
	Port 22
```
This will allow you to log in with `ssh ethereum-server` rather than needing to pass through all ssh parameters explicitly.

#### Synchronizing time with Chrony

**Note**: Because the consensus client relies on accurate times to perform attestations and produce blocks, your node's time must be accurate to real NTP time within 0.5 seconds.

Chrony is an implementation of the Network Time Protocol and helps to keep your computer's time synchronized with NTP.
To install chrony:
```bash
sudo apt-get install chrony -y
```

To see the source of synchronization data:
```bash
chronyc sources
```

To view the current status of chrony:
```bash
chronyc tracking
```

#### Setting Timezone
To pick your timezone run the following command:
```bash
sudo dpkg-reconfigure tzdata
```

Find your region using the simple text-based GUI.
In the event that you are using national system like India's `IST` select:
```
Asia/Kolkata
```

This will be appropriate for all locales in the country (`IST`, `GMT+0530`).

### Network Configuration
The standard UFW - Uncomplicated firewall can be used to control network access to your node and protect against unwelcome intruders.

**Configure UFW Defaults**
By default, deny all incoming and outgoing traffic.
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**Configure SSH Port 22**
If your node is remote in the cloud, or at home but on a different headless server, you will need to enable SSH port 22 in order to connect.
##### Allow ssh access for remote node
```bash
sudo ufw allow 22/tcp comment 'Allow SSH port'
```

**Note**: If your node is local at home and you have **keyboard access** to it, it's good practice to deny SSH port 22.

##### Deny ssh access for local node
```bash
sudo ufw deny 22/tcp comment 'Deny SSH port'
```

 **Allow Execution Client Port 30303**
Peering on port 30303, execution clients use this port for communication with other network peers.
```bash
sudo ufw allow 30303 comment 'Allow execution client port'
```

#### Allow Consensus Client port
Consensus clients generally use port 9000 for communication with other network peers. Using tcp port 13000 and udp port 12000, Prysm uses a slightly different configuration. In this guide, we will be using Prysm throughout.
**Prysm**:
```bash
sudo ufw allow 13000/tcp comment 'Allow consensus client port'
sudo ufw allow 12000/udp comment 'Allow consensus client port'
```

##### Enable firewall
Finally, enable the firewall and review the configuration.
```bash
sudo ufw enable
sudo ufw status numbered
```

#### **Install Fail2ban**
Fail2ban is an intrusion-prevention system that monitors log files and searches for particular patterns that correspond to a failed login attempt. If a certain number of failed logins are detected from a specific IP address (within a specified amount of time), fail2ban blocks access from that IP address.
**To install fail2ban:**
```bash
sudo apt-get install fail2ban -y
```

Edit a config file that monitors SSH logins.
```bash
sudo nano /etc/fail2ban/jail.local
```

Add the following lines to the bottom of the file.
```bash
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

To exit and save, press `Ctrl` + `X`, then `Y`, then`Enter`.

Restart fail2ban for settings to take effect.
```bash
sudo systemctl restart fail2ban
```

### Step 3: Installing the execution client

For the purpose of this guide we will be using Nethermind as our execution client. However there are other available execution clients like Geth, Besu and Erigon. 
**Execution Client Diversity** - To strengthen Ethereum's resilience against potential attacks or consensus bugs, it's best practice to run a minority client in order to increase client diversity. Check out the latest distribution of execution clients here: [Client Diversity](https://clientdiversity.org/)

![Pasted image 20230808235705](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/f57d3609-1b23-4ba6-b22b-88a6df1ddf8f)

Client Diversity (Reference : [CoinCashew](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet#2-signup-to-be-a-validator-at-the-launchpad))

**Nethermind** is built on .NET core. Nethermind makes integration with existing infrastructures simple, without losing sight of stability, reliability, data integrity, and security.

**Installation and Configuration:**
a. Create a service user for the execution service, create data directory and assign ownership.
```bash
sudo adduser --system --no-create-home --group execution
sudo mkdir -p /var/lib/nethermind
sudo chown -R execution:execution /var/lib/nethermind
```

b. Install dependencies.
```bash
sudo apt install curl libsnappy-dev libc6-dev jq libc6 unzip -y
```

c. Download binaries
Run the following to automatically download the latest linux release, un-zip and cleanup.
```bash
RELEASE_URL="https://api.github.com/repos/NethermindEth/nethermind/releases/latest"
BINARIES_URL="$(curl -s $RELEASE_URL | jq -r ".assets[] | select(.name) | .browser_download_url" | grep linux-x64)"
echo Downloading URL: $BINARIES_URL
cd $HOME
wget -O nethermind.zip $BINARIES_URL
unzip -o nethermind.zip -d $HOME/nethermind
rm nethermind.zip
```
Install the binaries.
```bash
sudo mv $HOME/nethermind /usr/local/bin/nethermind
```

d. Setup and Configure systemd
Create a **systemd unit file** to define your `execution.service` configuration.
```bash
sudo nano /etc/systemd/system/execution.service
```

Paste the following configuration into the file.
```bash
[Unit]
Description=Nethermind Execution Layer Client service for Mainnet
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com
[Service]
Type=simple
User=execution
Group=execution
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
WorkingDirectory=/var/lib/nethermind
Environment="DOTNET_BUNDLE_EXTRACT_BASE_DIR=/var/lib/nethermind"
ExecStart=/usr/local/bin/nethermind/Nethermind.Runner \
--config mainnet \
--datadir="/var/lib/nethermind" \
--Metrics.Enabled true \
--Metrics.ExposePort 6060 \
--Metrics.IntervalSeconds 10000 \
--Sync.SnapSync true \
--Sync.AncientBodiesBarrier 11052984 \
--Sync.AncientReceiptsBarrier 11052984 \
--JsonRpc.JwtSecretFile /secrets/jwtsecret
[Install]
WantedBy=multi-user.target
```

To exit and save, press `Ctrl` + `X`, then `Y`, then `Enter`.
Run the following to enable auto-start at boot time.
```bash
sudo systemctl daemon-reload
sudo systemctl enable execution
```

Finally, start your execution layer client and check it's status.
```bash
sudo systemctl start execution
sudo systemctl status execution
```

Press `Ctrl` + `C` to exit the status.

e. Other execution client command
To view logs
```bash
sudo journalctl -fu execution | ccze
```
To stop the client
```bash
sudo systemctl stop execution
```
To start the client
```bash
sudo systemctl start execution
```
To view status
```bash
sudo systemctl status execution
```

**Note:** If you're checking the logs and see any warnings or errors, please be patient as these will normally resolve once both your execution and consensus clients are fully synced to the Ethereum network.

### Step 4: Installing a consensus client
For the purpose of this guide, we will be using Prysm as our consensus client. Other client exists, even more stable than Prysm like Lodestar, Teku, Nimbus and Lighthouse. When choosing the consensus  clients for your Ethereum validator node, several factors come into play, including performance, stability, features, and community support. 
1. **Prysm:**
- **Tradeoffs:**
  - **Performance:** Prysm is known for its relatively efficient resource utilization and low memory footprint, making it suitable for setups with limited resources.
  - **Stability:** It has a stable and mature codebase, and its development team has been actively working on improving performance and fixing issues.
  - **Ease of Use:** Prysm offers user-friendly command-line tools and a graphical user interface (GUI) for easier management and monitoring.
  - **Community:** Prysm has a strong and active community, providing support and contributing to the project's development.

1. **Teku:**
- **Tradeoffs:**
  - **Performance:** Teku is built using the Java programming language, which may result in higher memory usage compared to other clients. It might require more resources to run efficiently.
  - **Features:** Teku is designed with enterprise and institutional users in mind, offering features like monitoring and management tools suitable for larger setups.
  - **Stability:** While Teku is backed by ConsenSys and PegaSys, it might not be as mature as some other clients due to being a relatively newer entrant to the Ethereum 2.0 client landscape.

**Rationale and Choice:**

Based on the provided hardware requirements, which include a modern multi-core processor, a minimum of 8 GB of RAM, a fast SSD with at least 256 GB of storage, and a stable high-speed internet connection, both Prysm and Teku are viable options for running an Ethereum validator node. However, considering the hardware specifications and tradeoffs mentioned, the following choice is suggested:

- **Prysm:** Given Prysm's efficient resource utilization, lower memory footprint, and established stability, it aligns well with the specified hardware requirements. Its active community and user-friendly tools make it a solid choice, especially for those looking to optimize performance on hardware with moderate resources.

Ultimately, the choice between Prysm and Teku depends on your specific preferences, technical expertise, and whether you prioritize resource efficiency and stability (Prysm) or enterprise-oriented features (Teku) for your Ethereum validator node setup. Whichever client you choose, it's important to follow the official documentation and best practices to ensure a secure and successful deployment.


![Pasted image 20230809001951](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/5c89b3ac-8542-460a-a86a-6b6fa54da207)

Consensus / Execution Diversity (Reference : [CoinCashew](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet#2-signup-to-be-a-validator-at-the-launchpad))

# Prysm

## Overview

[Prysm](https://github.com/prysmaticlabs/prysm) is a Go implementation of Ethereum protocol with a focus on usability, security, and reliability. Prysm is developed by [Prysmatic Labs](https://prysmaticlabs.com), a company with the sole focus on the development of their client. Prysm is written in Go and released under a GPL-3.0 license.


#### Official Links

| Subject       | Links                                                                                              |
| ------------- | -------------------------------------------------------------------------------------------------- |
| Releases      | [https://github.com/prysmaticlabs/prysm/releases](https://github.com/prysmaticlabs/prysm/releases) |
| Documentation | [https://docs.prylabs.network](https://docs.prylabs.network/)                                      |
| Website       | [https://prysmaticlabs.com](https://prysmaticlabs.com/)                                            |

### 1. Initial configuration

Create a service user for the consensus service, create data directory and assign ownership.

```bash
sudo adduser --system --no-create-home --group consensus
sudo mkdir -p /var/lib/prysm/beacon
sudo chown -R consensus:consensus /var/lib/prysm/beacon
```

Install dependencies.

```bash
sudo apt install curl jq git -y
```

### 2. Install Binaries

* Downloading binaries is often faster and more convenient.
* Building from source code can offer better compatibility and is more aligned with the spirit of FOSS (free open source software).

<details>

<summary>Option 1 - Download binaries</summary>

Run the following to automatically download the latest binaries.

```bash
cd $HOME
prysm_version=$(curl -f -s https://prysmaticlabs.com/releases/latest)
file_beacon=beacon-chain-${prysm_version}-linux-amd64
file_validator=validator-${prysm_version}-linux-amd64
curl -f -L "https://prysmaticlabs.com/releases/${file_beacon}" -o beacon-chain
curl -f -L "https://prysmaticlabs.com/releases/${file_validator}" -o validator
chmod +x beacon-chain validator
```

Install the binaries.

<pre class="language-bash"><code class="lang-bash"><strong>sudo mv beacon-chain validator /usr/local/bin
</strong></code></pre>

</details>

<details>

<summary>Option 2 - Build from source code</summary>

Install Bazel

```bash
wget -O bazel https://github.com/bazelbuild/bazelisk/releases/download/v1.16.0/bazelisk-linux-amd64
chmod +x bazel
sudo mv bazel /usr/local/bin
```

Build the binaries.

```bash
mkdir -p ~/git
cd ~/git
git clone https://github.com/prysmaticlabs/prysm.git
cd prysm
git fetch --tags
RELEASETAG=$(curl -s https://api.github.com/repos/prysmaticlabs/prysm/releases/latest | jq -r .tag_name)
git checkout tags/$RELEASETAG
bazel build //cmd/beacon-chain:beacon-chain --config=release
bazel build //cmd/validator:validator --config=release
```

Verify Prysm was built properly by displaying the help menu.

```shell
bazel run //beacon-chain -- --help
```

Install the binaries.

```shell
sudo cp -a $HOME/git/prysm /usr/local/bin/prysm
```

</details>

### **3. Setup and configure systemd**

Create a **systemd unit file** to define your `consensus.service` configuration.

```bash
sudo nano /etc/systemd/system/consensus.service
```

Depending on whether you're downloading binaries or building from source, choose the appropriate option. Paste the following configuration into the file.

<details>

<summary>Option 1 - Configuration for binaries</summary>

<pre class="language-bash"><code class="lang-bash"><strong>[Unit]
</strong>Description=Prysm Consensus Layer Client service for Mainnet
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=consensus
Group=consensus
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
ExecStart=/usr/local/bin/beacon-chain \
  --mainnet \
  --datadir=/var/lib/prysm/beacon \
  --checkpoint-sync-url=https://beaconstate.info \
  --genesis-beacon-api-url=https://beaconstate.info \
  --execution-endpoint=http://localhost:8551 \
  --jwt-secret=/secrets/jwtsecret \
  --accept-terms-of-use=true \
  --suggested-fee-recipient=&#x3C;0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>

[Install]
WantedBy=multi-user.target
</code></pre>

</details>

<details>

<summary>Option 2 - Configuration for building from source</summary>

<pre class="language-shell"><code class="lang-shell"><strong>[Unit]
</strong>Description=Prysm Consensus Layer Client service for Mainnet
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=consensus
Group=consensus
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
WorkingDirectory=/usr/local/bin/prysm
ExecStart=bazel run //cmd/beacon-chain --config=release -- \
  --mainnet \
  --datadir=/var/lib/prysm/beacon \
  --checkpoint-sync-url=https://beaconstate.info \
  --genesis-beacon-api-url=https://beaconstate.info \
  --execution-endpoint=http://localhost:8551 \
  --jwt-secret=/secrets/jwtsecret \
  --accept-terms-of-use=true \
  --suggested-fee-recipient=&#x3C;0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>

[Install]
WantedBy=multi-user.target
</code></pre>

</details>

* Replace**`<0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>`** with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable.
* **Not staking?** If you only want a full node, delete the whole lines beginning with

```
--suggested-fee-recipient
```

To exit and save, press `Ctrl` + `X`, then `Y`, then `Enter`.

Run the following to enable auto-start at boot time.

```bash
sudo systemctl daemon-reload
sudo systemctl enable consensus
```

Finally, start your consensus layer client and check it's status.

```bash
sudo systemctl start consensus
sudo systemctl status consensus
```

Press `Ctrl` + `C` to exit the status.

Check your logs to confirm that the consensus clients are up and syncing.

```bash
sudo journalctl -fu consensus | ccze
```

**Example of Synced Consensus Client Logs**

```bash
"Peer summary" activePeers=69 inbound=0 outbound=69 prefix=p2p
"Synced new block" block=0xb5ccb2f85... epoch=1837 finalizedEpoch=1838 finalizedRoot=0x1dce0... prefix=blockchain slot=21338 "Finished applying state transition" attestations=128 payloadHash=0x000000000000 prefix=blockchain slot=2138 syncBitsCount=213 txCount=0"terminal difficulty has not been reached yet" latestDifficulty=10000000 prefix=powchain terminalDifficulty=10000000
```

### 4. Helpful consensus client commands

{% tabs %}
{% tab title="View Logs" %}
```bash
sudo journalctl -fu consensus | ccze
```

**Example of Synced Prysm Consensus Client Logs**

```bash
time="2023-02-02 11:21:00" level=info msg="Peer summary" activePeers=35 inbound=10 outbound=25 prefix=p2p
time="2023-02-02 11:21:00" level=info msg="Synced new block" block=0xd9ddeza1289... epoch=11795 finalizedEpoch=111794 finalizedRoot=0x462e3275... prefix=blockchain slot=31205
time="2023-02-02 11:21:00" level=info msg="Finished applying state transition" attestations=64 payloadHash=0x000000000000 prefix=blockchain slot=31205 syncBitsCount=209 txCount=0
```

Stop Client
```bash
sudo systemctl stop consensus
```


Start Client
```bash
sudo systemctl start consensus
```

View Status of client
```bash
sudo systemctl status consensus
```


Now that your consensus client is configured and started, you have a full node.

Proceed to the next step on setting up your validator client, which turns a full node into a staking node.



### Step 5: Installing the Validator

To complete installation of a full node with validator, complete the following steps:
a. Setting up Validator Keys
b. Installing Validator
c. Next Steps

#### Setting up Validator Keys
A. Install dependencies, the Ethereum Foundation deposit tool and generate your two sets of key pairs.

🚩
Each validator will have two sets of key pairs. A **signing key** and a **withdrawal key.** These keys are derived from a single mnemonic phrase. 
You will also set your [ETH Withdrawal Address](https://notes.ethereum.org/@launchpad/withdrawals-faq#Q-What-are-the-two-types-of-withdrawals), preferably from your Ledger or Trezor hardware wallet.
🛑🛑**DO NOT USE AN EXCHANGE ADDRESS AS WITHDRAWAL ADDRESS.**🛑🛑

🛑🛑**Double check your work as this is permanent once set!**🛑🛑

**The Withdrawal Address is:**
- where your ETH is returned upon "voluntary exiting a validator", or also known as full withdrawal.
- where you receive partial withdrawals, which is where any excess balance above 32 ETH is periodically scraped and made available for use.

You have the choice of using the [Wagyu GUI](https://github.com/stake-house/wagyu-installer), downloading the pre-built [Ethereum staking deposit tool](https://github.com/ethereum/staking-deposit-cli) or building it from source. For the purpose of this guide, we will be downloading the pre-built Ethereum staking and building it from source

Option A: **Downloading the pre-built Ethereum staking:**
a. Download staking-deposit-cli
```bash
cd $HOME
wget https://github.com/ethereum/staking-deposit-cli/releases/download/v2.5.0/staking_deposit-cli-d7b5304-linux-amd64.tar.gz
```

b. Verify the SHA256 Checksum matches the checksum on the [releases page](https://github.com/ethereum/staking-deposit-cli/releases).
```bash
echo "3f51859d78ad47a3e258470f5a5caf03d19ed1d4307d517325b7bb8f6fcde6ef *staking_deposit-cli-d7b5304-linux-amd64.tar.gz" | shasum -a 256 --check
```

Example valid output:
```
> staking_deposit-cli-d7b5304-linux-amd64.tar.gz: OK
```

**Only proceed if the sha256 check passes with **OK**!**

c. Extract the archive.
```bash
tar -xvf staking_deposit-cli-d7b5304-linux-amd64.tar.gz
mv staking_deposit-cli-d7b5304-linux-amd64 staking-deposit-cli
rm staking_deposit-cli-d7b5304-linux-amd64.tar.gz
cd staking-deposit-cli
```

Make a new mnemonic and replace `<ETH_ADDRESS_FROM_IDEALLY_HARDWARE_WALLET>` with your [ethereum withdrawal address](https://notes.ethereum.org/@launchpad/withdrawals-faq#Q-If-I-used---eth1_withdrawal_address-when-making-my-initial-deposit-which-type-of-withdrawal-credentials-do-I-have), ideally from a Trezor, Ledger or comparable hardware wallet.

🛑🛑**DO NOT USE AN EXCHANGE ADDRESS AS WITHDRAWAL ADDRESS.**🛑🛑

🛑🛑**Double check your work as this is permanent once set!**🛑🛑

```bash
./deposit new-mnemonic --chain mainnet --eth1_withdrawal_address <ETH_ADDRESS_FROM_IDEALLY_HARDWARE_WALLET>
```

Option B: **Build from source code**
a. Install dependencies.
```bash
sudo apt update
sudo apt install python3-pip git -y
```

b. Download source code and install.
```bash
cd $HOME
git clone https://github.com/ethereum/staking-deposit-cli
cd staking-deposit-cli
sudo ./deposit.sh install
```

c. Make a new mnemonic and replace `<ETH_ADDRESS_FROM_IDEALLY_HARDWARE_WALLET>` with your [ethereum withdrawal address](https://notes.ethereum.org/@launchpad/withdrawals-faq#Q-If-I-used---eth1_withdrawal_address-when-making-my-initial-deposit-which-type-of-withdrawal-credentials-do-I-have), ideally from a Trezor, Ledger or comparable hardware wallet.

🛑🛑**DO NOT USE AN EXCHANGE ADDRESS AS WITHDRAWAL ADDRESS.**🛑🛑

🛑🛑**Double check your work as this is permanent once set!**🛑🛑

```bash
./deposit.sh new-mnemonic --chain mainnet --eth1_withdrawal_address <ETH_ADDRESS_FROM_IDEALLY_HARDWARE_WALLET>
```

B. If using **staking-deposit-cli**, follow the prompts and pick a **KEYSTORE password**. This password encrypts your keystore files. Write down your mnemonic and keep this safe and **offline**.

>**Caution**: Only deposit the 32 ETH per validator if you are confident your execution client and consensus client will be fully synched and ready to perform validator duties. You can return later to launchpad with your deposit-data to finish the next steps.

C. Follow the steps at [Ethereum-Launchpad](https://launchpad.ethereum.org) while skipping over the steps you already just completed. Study the eth2 phase 0 overview material. Understanding eth staking is the key to success!

>🐳**Batch Depositing Tip**: If you have many deposits to make for many validators, consider using [Abyss.finance's eth2depositor tool.](https://abyss.finance/eth2depositor) This greatly improves the deposit experience as multiple deposits can be batched into one transaction, thereby saving gas fees and saving your fingers by minimizing Metamask clicking.

Source: [https://twitter.com/AbyssFinance/status/1379732382044069888](https://twitter.com/AbyssFinance/status/1379732382044069888)

D. Back on the launchpad website, upload your`deposit_data-#########.json` found in the `validator_keys` directory.

E. Connect to the launchpad with your Metamask wallet, review and accept terms

F. Confirm the transaction(s). There's one deposit transaction of 32 ETH for each validator.

> For instance, if you want to run 3 validators you will need to have (32 x 3) = 96 ETH plus some extra to cover the gas fees.

>Tip for **Ledger Nano Hardware wallet** users: If you encounter difficulty making the deposit transaction, enable blind signing and contract data.

>Your transaction is sending and depositing your ETH to the [official ETH2 deposit contract address.](https://blog.ethereum.org/2020/11/04/eth2-quick-update-no-19/)**Check**, _double-check_, **_triple-check_** that the official Eth2 deposit contract address is correct.[`0x00000000219ab540356cBB839Cbe05303d7705Fa`](https://etherscan.io/address/0x00000000219ab540356cbb839cbe05303d7705fa)

>🚩 **Critical Crypto Reminder:** Keep your mnemonic, keep your ETH.
 a. Write down your mnemonic seed **offline**. _Not email. Not cloud._
 b. Multiple copies are better. Best stored in a [_metal seed._](https://jlopp.github.io/metal-bitcoin-storage-reviews/)
 c. Make **offline backups**, such as to a USB key, of your **`validator_keys`** directory.


### Installing Validator

For the purpose of this guide, we will be using Prysm for our validator:

a. Create a service user for the validator service, as this improves security, then create data directories.
```bash
sudo adduser --system --no-create-home --group validator
sudo mkdir -p /var/lib/prysm/validators
```

Storing your **keystore password** in a text file is required so that Prysm can decrypt and load your validators automatically.

b. Create a file to store your **keystore password**. Type your password in this file.
```bash
sudo nano /var/lib/prysm/validators/password.txt
```

To exit and save, press `Ctrl` + `X`, then `Y`, then `Enter`.

c. Confirm that your **keystore password** is correct.
```bash
sudo cat /var/lib/prysm/validators/password.txt
```

Import your validator keys by importing your **keystore file**. When asked to create a new wallet password, enter your **keystore password**. When prompted for the imported accounts password, enter your **keystore password** again.

For Binaries:
```bash
sudo /usr/local/bin/validator accounts import \
  --accept-terms-of-use \
  --mainnet \
  --wallet-dir=/var/lib/prysm/validators \
  --keys-dir=$HOME/staking-deposit-cli/validator_keys
```
OR
Built from source:
```bash
cd /usr/local/bin/prysm
sudo bazel run //validator:validator -- accounts import \
  --accept-terms-of-use \
  --mainnet \
  --wallet-dir=/var/lib/prysm/validators \
  --keys-dir=$HOME/staking-deposit-cli/validator_keys
```

🛑 **Do not import your validator keys into multiple validator clients and run them at the same time, or you might get slashed. If moving validators to a new setup or different validator client, ensure deletion of the previous validator keys before continuing.**

d. Verify that your keystore file was imported successfully.
For Binaries
```bash
sudo /usr/local/bin/validator accounts list \
  --wallet-dir=/var/lib/prysm/validators \
  --mainnet
```
OR
Built from source
```bash
cd /usr/local/bin/prysm
sudo bazel run //validator:validator -- accounts list \
  --wallet-dir=/var/lib/prysm/validator \
  --mainnet
```

Once successful, you will be shown your **validator's public key**. For example:
```bash
Showing 2 validator accounts
View the eth1 deposit transaction data for your accounts by running `validator accounts list --show-deposit-data`
Account 0 | gently-learning-chamois
[validating public key] 0x95d39860a0d6ea3b92cba78069d21f3a987988f3b8417b14f0945353d79ed9e338bbe6e9d63d487abc044a710ce34866
Account 1 | presumably-powerful-lynx
[validating public key] 0x82b225f66476962b161ed015786df00a0b7b28231915e6d09e81ba8d5c4ae8502b6d5337e3bf101ad72741dc69f0a7cf
```


e. Setup ownership permissions, including hardening the access to this directory.
```bash
sudo chown -R validator:validator /var/lib/prysm/validators
sudo chmod 700 /var/lib/prysm/validators
```

f. Create a **systemd unit file** to define your `validator.service` configuration.
```bash
sudo nano /etc/systemd/system/validator.service
```

Depending on whether you're downloading binaries or building from source, choose the appropriate option. Paste the following configuration into the file.

**Configuration for binaries
```
[Unit]
Description=Prysm Validator Client service for mainnet
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=validator
Group=validator
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
ExecStart=/usr/local/bin/validator \
  --mainnet \
  --accept-terms-of-use \
  --datadir=/var/lib/prysm/validators \
  --beacon-rpc-provider=localhost:4000 \
  --wallet-dir=/var/lib/prysm/validators \
  --wallet-password-file=/var/lib/prysm/validators/password.txt \
  --graffiti "" \
  --suggested-fee-recipient=<0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>

[Install]
WantedBy=multi-user.target
```

**Configuration for building from source**
```
[Unit]
Description=Prysm validator Client service for mainnet
Wants=network-online.target
After=network-online.target
Documentation=https://www.coincashew.com

[Service]
Type=simple
User=validator
Group=validator
Restart=on-failure
RestartSec=3
KillSignal=SIGINT
TimeoutStopSec=900
WorkingDirectory=/usr/local/bin/prysm
ExecStart=bazel run //cmd/validator --config=release -- \
  --mainnet \
  --accept-terms-of-use \
  --datadir=/var/lib/prysm/validators \
  --beacon-rpc-provider=localhost:4000 \
  --wallet-dir=/var/lib/prysm/validators \
  --wallet-password-file=/var/lib/prysm/validators/password.txt \
  --graffiti "" \
  --suggested-fee-recipient=<0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>

[Install]
WantedBy=multi-user.target
```

- Replace`<0x_CHANGE_THIS_TO_MY_ETH_FEE_RECIPIENT_ADDRESS>` with your own Ethereum address that you control. Tips are sent to this address and are immediately spendable.
- If you wish to customize a graffiti message that is included when you produce a block, add your message between the double quotes after `--graffiti`.

g. To exit and save, press `Ctrl` + `X`, then `Y`, then `Enter`.

h. Run the following to enable auto-start at boot time.
```bash
sudo systemctl daemon-reload
sudo systemctl enable validator
```

i. Finally, start your validator client and check it's status.
```bash
sudo systemctl start validator
sudo systemctl status validator
```

g. Check your logs to confirm that the validator clients are up and functioning.
```bash
sudo journalctl -fu validator | ccze
```

For example when using 2 validators, logs will show the following:
```
level=info msg="Validating for public key" prefix=validator publicKey=0x95d39860a0d6
level=info msg="Validating for public key" prefix=validator publicKey=0x82b225f66476
```

Press `Ctrl` + `C` to exit the logs.

**Example of Synced Prysm Validator Client Logs**

- Once the validator is active and proceeded through the validator activation queue, attestation messages will appear indicating successful attestations.
- Notice the key words "`INFO validator: Submitted new attestations`".
```
[2022-11-21 1:21:21]  INFO validator: Submitted new attestations AggregatorIndices=[12412] AttesterIndices=[73613] BeaconBlockRoot=0xca3213f1a3 CommitteeIndex=12 Slot=12422 SourceEpoch=12318 SourceRoot=0xd9ddeza1289 TargetEpoch=121231 TargetRoot=0xff313419acaa1
```

You've successfully setup your Ethereum Node Validator

#### Note: 
- **Sync Timeline:** Syncing the consensus client is instantaneous with checkpoint sync but the execution client can take up to a day. On nodes with fast NVME drives and gigabit internet, expect your node to be fully synced in a few hours.
- **Patience required**: If you're checking the logs and see any warnings or errors, be patient as these will normally resolve once both your execution and consensus clients are fully synced to the Ethereum network.
- These are the steps to know if you've been fully synced:
a. Check your execution client's logs and compare the block number against the most recent block on [Etherscan.io](https://etherscan.io/)
- To check EL logs; enter: `journalctl -fu execution`
b. Check your consensus client's logs and compare the slot number against the most recent slot on [Beaconcha.in](https://beaconcha.in/)
- To check CL logs; enter: `journalctl -fu consensus`
## Part 2: Monitoring Your Validator with Prometheus and Grafana

Prometheus is a monitoring platform that collects metrics from monitored targets by scraping metrics HTTP endpoints on these targets. [Official Prometheus documentation here.](https://prometheus.io/docs/introduction/overview/) Grafana is a dashboard used to visualize the collected data.

#### Installation

a. Install prometheus and prometheus node exporter.
```bash
sudo apt-get install -y prometheus prometheus-node-exporter
```

b. Install grafana
```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" > grafana.list
sudo mv grafana.list /etc/apt/sources.list.d/grafana.list
sudo apt-get update && sudo apt-get install -y grafana
```

c. Enable services so they start automatically.
```bash
sudo systemctl enable grafana-server.service prometheus.service prometheus-node-exporter.service
```

d. Create the **prometheus.yml** config file. Simply copy and paste.
```bash
cat > $HOME/prometheus.yml << EOF
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
   - job_name: 'node_exporter'
     static_configs:
       - targets: ['localhost:9100']
   - job_name: 'validator'
     static_configs:
       - targets: ['localhost:8081']
   - job_name: 'beacon node'
     static_configs:
       - targets: ['localhost:8080']
   - job_name: 'slasher'
     static_configs:
       - targets: ['localhost:8082']
EOF
```

e. Setup prometheus for your execution client. Start by editing **prometheus.yml**
```bash
nano $HOME/prometheus.yml
```

f. Append the applicable job snippet for your execution client to the end of **prometheus.yml**. Save the file.

🛑 **Spacing matters**. Ensure all `job_name` snippets are in alignment.
Snippet for Nethermind:
```bash
   - job_name: 'nethermind'
     static_configs:
       - targets: ['localhost:6060']
```

g. Move it to `/etc/prometheus/prometheus.yml`
```bash
sudo mv $HOME/prometheus.yml /etc/prometheus/prometheus.yml
```

h. Update file permissions.
```bash
sudo chmod 644 /etc/prometheus/prometheus.yml
```

i. Finally, restart the services.
```bash
sudo systemctl restart grafana-server.service prometheus.service prometheus-node-exporter.service
```

g. Verify that the services are running properly:
```bash
sudo systemctl status grafana-server.service prometheus.service prometheus-node-exporter.service
```

#### Grafana Security: **SSH Tunnels**
🛑
Do not expose Grafana (port 3000) to the public internet as this invites a new attack surface! A secure solution would be to access Grafana through a ssh tunnel.

Example of how to create a ssh tunnel in Linux or MacOS:
```bash
ssh -N -v <user>@<staking.node.ip.address> -L 3000:localhost:3000
```

Example of how to create a ssh tunnel in Windows with [Putty](https://putty.org/):

Navigate to Connection > SSH > Tunnels > Enter Source Port `3000` > Enter Destination `localhost:3000` > Click Add

![](https://1280931729-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-M5KYnWuA6dS_nKYsmfV-887967055%2Fuploads%2FRdvcICpA2KMdGo5KSoFJ%2Fimage.png?alt=media&token=47098d5b-bb3d-4021-a225-08aad68d5f15)

Putty dashboard (Reference : [CoinCashew](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet#2-signup-to-be-a-validator-at-the-launchpad))
Now you can access Grafana on your local machine by pointing a web browser to [Localhost:3000](http://localhost:3000/)

#### Setting up Grafana Dashboards

- Open [http://localhost:3000](http://localhost:3000) or http://<your validator's ip address>:3000 in your web browser.
- Login with **admin** / **admin**
- Change password
- Click the **configuration gear** icon, then **Add data Source**
- Select **Prometheus**
- Set **Name** to **"Prometheus**"
- Set **URL** to [http://localhost:9090](http://localhost:9090)
- Click **Save & Test**
- **Download and save** your consensus client's json file. For the purpose of this guide; we would use Prysm; however, other consensus clients json files are available [Prysm](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/less_10_validators.json)
- **Download and save** your execution client's json file. For the purpose of this guide, we would use Nethermind; however, other execution clients json files are available [Nethermind](https://raw.githubusercontent.com/NethermindEth/metrics-infrastructure/master/grafana/provisioning/dashboards/nethermind.json)
- **Download and save** a [node-exporter dashboard](https://grafana.com/api/dashboards/11074/revisions/9/download) for general system monitoring
- Click **Create +** icon > **Import**
- Add the consensus client dashboard via **Upload JSON file**
- If needed, select Prometheus as **Data Source**.
- Click the **Import** button.
- Repeat steps 12-15 for the execution client dashboard.
- Repeat steps 12-15 for the node-exporter dashboard.

**Troubleshooting common Grafana issues**
**Symptom**: Your dashboard is missing some data.
**Solution**_:_ Ensure that the execution or consensus client has enabled the appropriate metrics flag.
- For Nethermind: Nethermind.Runner --Metrics.Enabled true
- For Erigon: erigon --metrics
- For Lighthouse beacon-node: lighthouse bn --validator-monitor-auto
- Nimbus: nimbus_beacon_node --metrics --metrics-port=8008
- Teku: --metrics-enabled=true --metrics-port=8008
- Lodestar beacon-node: lodestar beacon --metrics true
- Geth: geth --http --metrics --pprof
- Besu: besu --metrics-enabled=true

#### Example of Grafana Dashboards for Prysm consensus client

![Pasted image 20230809033952](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/c38040cb-daac-4b64-9972-65e211766216)

Reference: [Github](https://github.com/GuillaumeMiralles/prysm-grafana-dashboard)

![Pasted image 20230809034049](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/d86d408e-006c-474b-883d-20e109a771dc)

Reference: [Github](https://github.com/metanull-operator/eth2-grafana/)

JSON download link: [Github](https://github.com/metanull-operator/eth2-grafana/raw/master/eth2-grafana-dashboard-single-source.json)

#### Example of Grafana Dashboards for Nethermind execution client

![Pasted image 20230809034323](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/3d921bdc-eebd-4297-b854-1c119d60bee3)

Reference: [Github](https://github.com/NethermindEth/metrics-infrastructure)

#### Example of Node-Exporter Dashboard
**General system monitoring**
Includes: CPU, memory, disk IO, network, temperature and other monitoring metrics.

![Pasted image 20230809034514](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/36f7c67f-b5c4-471b-93d2-19913a370939)

Reference: [Grafana Node Exporter Dashboard](https://grafana.com/grafana/dashboards/11074)


![Pasted image 20230809034534](https://github.com/kingsmandralph/Ethereum-Node-Validator/assets/95085212/2038f66b-afa3-416e-a34a-8b4a34caa1ae)
Reference: [Grafana Node Exporter Dashboard](https://grafana.com/grafana/dashboards/11074)

#### Setting up Alert Notifications
Setup alerts to get notified if your validators go offline and get notified of problems with your validators via email notifications
- Visit [Beaconcha.in](https://beaconcha.in)
- Sign up for an account.
- Verify your **email**
- Search for your **validator's public address**
- Add validators to your watchlist by clicking the **bookmark symbol**.

### Monitoring with Uptime Check by Google Cloud
Here's how to setup a no-cost monitoring service called Uptime Check by Google:
- Visit [cloud.google.com](https://cloud.google.com)
- Search for **Monitoring** in the search field.
- Click **Select a Project to Start Monitoring**.
- Click **New Project.***
- **Name your project and click Create.**
- From the notifications menu, select your new project.
- On the right column, there's a Monitoring Card. Click **Go to Monitoring**.
- On the left menu, click **Uptime checks** and then **CREATE UPTIME CHECK.**
- Type in a title i.e. **_Prysm node_**
- Select protocol as **_TCP_**
- Enter your public IP address and port number. i.e. ip=**7.55.6.3** and port=**30303**
- Select your desired frequency to check i.e. **5 minutes.**
- Choose the region closest to you to check from. Click Next.
- Create a Notification Channel. Click **Manage Notification Channels.**
- Choose your desired settings. Pick from any or all of Slack, Webhook, Email or SMS.
- Go back to Create Uptime Check window.
- Within the notifications field, click the refresh button to load your new notification channels.
- Select desired notifications.
- Click **TEST** to verify your notifications are setup correctly.
- Click **CREATE** to finish.

## Part 3: Maintenance

## Updating the execution client
When a new release is cut, you will want to update to the latest stable release. The following shows you how to update your execution client.
Always review the **release notes** before updating. There may be changes requiring your attention.
- [Nethermind](https://github.com/NethermindEth/nethermind/releases)
- [Besu](https://github.com/hyperledger/besu/releases)
- [Geth](https://github.com/ethereum/go-ethereum/releases)
- [Erigon](https://github.com/ledgerwatch/erigon/releases)

**Step 1: Select your execution client.**
For the purpose of this guide, we're still going with Nethermind
Download binaries:
a. Run the following to automatically download the latest linux release, un-zip and cleanup.
```bash
RELEASE_URL="https://api.github.com/repos/NethermindEth/nethermind/releases/latest"
BINARIES_URL="$(curl -s $RELEASE_URL | jq -r ".assets[] | select(.name) | .browser_download_url" | grep linux-x64)"
echo Downloading URL: $BINARIES_URL
cd $HOME
wget -O nethermind.zip $BINARIES_URL
unzip -o nethermind.zip -d $HOME/nethermind
rm nethermind.zip
```

b. Stop the services.
```bash
sudo systemctl stop execution
```

c. Remove old binaries, install new binaries and restart the services.
```bash
sudo rm -rf /usr/local/bin/nethermind
sudo mv $HOME/nethermind /usr/local/bin/nethermind
sudo systemctl start execution
```


**Step 2: Verify services and logs are working properly**
a. Verify services status
```bash
sudo systemctl status execution
```

b. Check logs
```bash
sudo journalctl -fu execution
```

**Step 3: Optional - Verify your validator's attestations on public block explorer**
a. Visit [Beaconcha.in](https://beaconcha.in) or [Beaconscan.com](https://beaconscan.com)
b. Enter your validator's pubkey into the search bar and look for successful attestations.

## Updating the consensus client
When a new release is cut, you will want to update to the latest stable release. The following shows you how to update your beacon chain and validator.
Always review the **release notes** before updating. There may be changes requiring your attention.

- [Lighthouse](https://github.com/sigp/lighthouse/releases)
- [Lodestar](https://github.com/ChainSafe/lodestar/releases)
- [Teku](https://github.com/ConsenSys/teku/releases)
- [Nimbus](https://github.com/status-im/nimbus-eth2/releases)
- [Prysm](https://github.com/prysmaticlabs/prysm/releases)

**Step 1: Select your consensus client.**
For the purpose of this guide; we are using Prysm for the consensus client
Option 1: Download binaries
a. Run the following to automatically download the latest binaries.
```bash
cd $HOME
prysm_version=$(curl -f -s https://prysmaticlabs.com/releases/latest)
file_beacon=beacon-chain-${prysm_version}-linux-amd64
file_validator=validator-${prysm_version}-linux-amd64
curl -f -L "https://prysmaticlabs.com/releases/${file_beacon}" -o beacon-chain
curl -f -L "https://prysmaticlabs.com/releases/${file_validator}" -o validator
chmod +x beacon-chain validator
```

b. Stop the services.
```bash
sudo systemctl stop consensus validator
```

c. Remove old binaries, install new binaries and restart the services.
```bash
sudo rm /usr/local/bin/beacon-chain
sudo rm /usr/local/bin/validator
sudo mv beacon-chain validator /usr/local/bin
sudo systemctl start consensus validator
```

Option 2: Build from source code
a. Install Bazel
```bash
wget -O bazel https://github.com/bazelbuild/bazelisk/releases/download/v1.16.0/bazelisk-linux-amd64
chmod +x bazel
sudo mv bazel /usr/local/bin
```

b. Pull the latest source code and build the binaries.
```bash
cd $HOME/git/prysm
git fetch --tags
RELEASETAG=$(curl -s https://api.github.com/repos/prysmaticlabs/prysm/releases/latest | jq -r .tag_name)
git checkout tags/$RELEASETAG
bazel build //cmd/beacon-chain:beacon-chain --config=release
bazel build //cmd/validator:validator --config=release
```

c. Verify Prysm was built properly by displaying the help menu.
```bash
bazel run //beacon-chain -- --help
```

d. Stop the services.
```bash
sudo systemctl stop consensus validator
```

e. Remove old binaries, install new binaries and restart the services.
```bash
sudo rm -rf /usr/local/bin/prysm
sudo cp -a $HOME/git/prysm /usr/local/bin/prysm
sudo systemctl start consensus validator
```

**Step 2: Verify services and logs are working properly**
a. Verify services status
```bash
# Verify services status
sudo systemctl status consensus validator
```

b. Check logs
```bash
# Check logs
sudo journalctl -fu consensus
sudo journalctl -fu validator
```

Step 3: Optional - Verify your validator's attestations on public block explorer
a. Visit [Beaconcha.in](https://beaconcha.in) or [Beaconscan](https://beaconscan.com)
b. Enter your validator's pubkey into the search bar and look for successful attestations.

## Backups Checklist: Critical Staking Node Data
#### 🔥 **Critical Crypto Reminder:** **Keep your mnemonics, keep your ETH.**

- **Withdrawal Wallet Seed Phrase**: Ideally this is secured by a hardware wallet. Most important piece of data and represents your ETH!
- **Validator Key Mnemonic**: Write this down **offline**. Not email. Not cloud.
- **Validator_keys directory**: Contains key
- store files and deposit_data.
- **Password for keystore files:** Required when installing validators.

**Suggestions**:
- Create multiple copies. _Best stored in a_ [_metal seed._](https://jlopp.github.io/metal-bitcoin-storage-reviews/)
- Make **offline backups**, such as to a USB key, of your **`validator_keys`** directory.

### 🚀🚀 For more maintenance tips, click this [link](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-ii-maintenance/updating-consensus-client)

## References
a. [How to install and run an ETH 2.0 Validator Node](https://medium.com/simplystaking/setting-up-an-eth-2-0-validator-node-simply-staking-40b5f96a9e8d)
b. [Prysm](https://prysmaticlabs.com/)
c. [Prysm v1.0 vs Prysm Pyrmont](https://discourse.dappnode.io/t/prysm-v1-0-vs-prysm-pyrmont-question/494/20?page=2)
d. [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
e. [Grafana Documentation](https://grafana.com/docs/grafana/latest/getting-started/getting-started-prometheus/)
f. [Setup a validator on eth2 mainnet](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet#2-signup-to-be-a-validator-at-the-launchpad)

#### Link to Substack Article: [Substack Article](https://open.substack.com/pub/ekeneraphael/p/step-by-step-guide-on-setting-up?r=1rgtae&utm_campaign=post&utm_medium=web)
#### *Hope this guide helped you a lot..... *
