# 0G AI Alignment Node - Setup Guide v1.0.0
Quick and easy setup for 0G AI Alignment Node with systemd service.

## Requirements
- Ubuntu/Debian VPS
- 1 CPU, 64MB RAM, 10GB disk
- Node license NFT (buy from: https://claim.0gfoundation.ai)
- Wallet private key (that holds the NFT)

## Installation

### 1. Setup Directory
```bash
cd ~
mkdir 0g-alignment-node
cd 0g-alignment-node
```

### 2. Download and Extract Node
```bash
wget https://github.com/0gfoundation/alignment-node-release/releases/download/v1.0.0/alignment-node.tar.gz
tar -xzf alignment-node.tar.gz
mv alignment-node/0g-alignment-node ./
chmod +x 0g-alignment-node
rm -rf alignment-node alignment-node.tar.gz
```

### 3. Configure Node
**Choose your port** (default: 8080, example: 42069):
```bash
export MY_PORT=42069  # Change this to any port you want
```

Create `.env` file:
```bash
cat > .env << EOF
ZG_ALIGNMENT_NODE_LOG_LEVEL=info
ZG_ALIGNMENT_NODE_SERVICE_IP=http://0.0.0.0:$MY_PORT
ZG_ALIGNMENT_NODE_SERVICE_PRIVATEKEY=YOUR_PRIVATE_KEY_WITHOUT_0X
EOF
```

Create `config.toml`:
```bash
cat > config.toml << EOF
ZG_ALIGNMENT_NODE_LOG_LEVEL="info"
ZG_ALIGNMENT_NODE_SERVICE_IP="http://0.0.0.0:$MY_PORT"
ZG_ALIGNMENT_NODE_SERVICE_PRIVATEKEY="YOUR_PRIVATE_KEY_WITHOUT_0X"
EOF
```

**IMPORTANT**: Replace `YOUR_PRIVATE_KEY_WITHOUT_0X` with your actual private key (no 0x prefix)

### 4. Open Your Port
```bash
sudo ufw allow $MY_PORT/tcp
sudo ufw allow 22/tcp
sudo ufw --force enable
```

### 5. Register Operator
Load environment and register your NFT(s):

**For Single NFT:**
```bash
source .env

./0g-alignment-node registerOperator \
  --key YOUR_PRIVATE_KEY_WITHOUT_0X \
  --token-id YOUR_NFT_TOKEN_ID \
  --commission 10 \
  --chain-id 42161 \
  --rpc https://arb1.arbitrum.io/rpc \
  --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac \
  --mainnet
```

**For Multiple NFTs (run separately for each):**
```bash
# Register NFT 1
./0g-alignment-node registerOperator \
  --key YOUR_PRIVATE_KEY_WITHOUT_0X \
  --token-id NFT_ID_1 \
  --commission 10 \
  --chain-id 42161 \
  --rpc https://arb1.arbitrum.io/rpc \
  --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac \
  --mainnet

# Wait for confirmation, then register NFT 2
./0g-alignment-node registerOperator \
  --key YOUR_PRIVATE_KEY_WITHOUT_0X \
  --token-id NFT_ID_2 \
  --commission 10 \
  --chain-id 42161 \
  --rpc https://arb1.arbitrum.io/rpc \
  --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac \
  --mainnet

# Continue for each NFT...
```

**Note:** Each NFT must be registered separately. Wait for transaction confirmation between registrations.

### 6. Approve NFTs for Delegation
After registering, approve your NFTs for delegation:

**For Single NFT:**
```bash
./0g-alignment-node approve --mainnet \
  --key YOUR_PRIVATE_KEY_WITHOUT_0X \
  --chain-id 42161 \
  --rpc https://arb1.arbitrum.io/rpc \
  --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac \
  --destNode YOUR_NODE_ADDRESS \
  --tokenIds YOUR_NFT_TOKEN_ID
```

**For Multiple NFTs (all at once):**
```bash
./0g-alignment-node approve --mainnet \
  --key YOUR_PRIVATE_KEY_WITHOUT_0X \
  --chain-id 42161 \
  --rpc https://arb1.arbitrum.io/rpc \
  --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac \
  --destNode YOUR_NODE_ADDRESS \
  --tokenIds ID1,ID2,ID3,ID4,ID5
```

**Parameters:**
- `YOUR_PRIVATE_KEY_WITHOUT_0X`: Your wallet private key (no 0x prefix)
- `YOUR_NODE_ADDRESS`: Your node address (shown in logs as "Verified identity result")
- `YOUR_NFT_TOKEN_ID`: Your NFT ID(s) from Arbiscan
- `commission`: Your commission rate (10 = 10%)

### 7. Create Service
```bash
sudo tee /etc/systemd/system/0g-alignment-node.service << EOF
[Unit]
Description=0G AI Alignment Node
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/0g-alignment-node
EnvironmentFile=/root/0g-alignment-node/.env
ExecStart=/root/0g-alignment-node/0g-alignment-node start --mainnet
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

### 8. Start Node
```bash
sudo systemctl daemon-reload
sudo systemctl enable 0g-alignment-node
sudo systemctl start 0g-alignment-node
```

### 9. Check Status
```bash
# Check if running
sudo systemctl status 0g-alignment-node

# View logs
sudo journalctl -u 0g-alignment-node -f

# Your node address will show in logs as "Verified identity result"
```

## Update Binary
When a new version is released:

```bash
cd ~/0g-alignment-node && \
sudo systemctl stop 0g-alignment-node && \
mv 0g-alignment-node 0g-alignment-node.backup && \
wget https://github.com/0gfoundation/alignment-node-release/releases/download/v1.0.0/alignment-node.tar.gz && \
tar -xzf alignment-node.tar.gz && \
mv alignment-node/0g-alignment-node ./ && \
chmod +x 0g-alignment-node && \
rm -rf alignment-node alignment-node.tar.gz && \
sudo systemctl restart 0g-alignment-node && \
sudo systemctl status 0g-alignment-node
```

## Delegation Portal
1. Go to: https://claim.0gfoundation.ai/delegation
2. Connect wallet (use Arbitrum Network)
3. Enter your node wallet address
4. Click "Delegate"

## Network Configuration

### Add Arbitrum to MetaMask
- **Network Name**: Arbitrum One
- **RPC**: https://arb1.arbitrum.io/rpc
- **Chain ID**: 42161
- **Symbol**: ETH
- **Explorer**: https://arbiscan.io

## Useful Commands
```bash
# Stop node
sudo systemctl stop 0g-alignment-node

# Restart node
sudo systemctl restart 0g-alignment-node

# View logs
sudo journalctl -u 0g-alignment-node -f

# View last 50 log lines
sudo journalctl -u 0g-alignment-node -n 50

# Check port status
sudo ss -tulpn | grep YOUR_PORT

# Check service status
sudo systemctl status 0g-alignment-node
```

## Important Notes
- **Register First**: Always register operator before starting the node service
- **Each NFT Separately**: Register each NFT individually (no bulk registration)
- **Approve Can Be Bulk**: Approve supports multiple NFTs in one command
- **400 Error**: If you see "Failed to send request status_code=400" in logs, this is normal
- **Gas Fees**: Have enough ETH on Arbitrum for registration transactions
- **Commission**: Set between 0-100 (10 = 10% commission)

## ⚠️ Security Warning
- **Private Key Visibility**: Your private key appears in logs when debug mode is enabled
- **Use `info` log level** for production (not `debug`)
- **Secure your VPS**: Use strong passwords, SSH keys, and firewall rules
- **Never share logs publicly** without removing sensitive information
- **Use dedicated wallet**: Only keep NFT + gas fees, not your main funds
- **Backup safely**: Keep `.env` file backup in secure location, not on public services

## Troubleshooting

**Port Already in Use**
```bash
# Check what's using the port
sudo lsof -i :YOUR_PORT
# Choose a different port and update .env and config.toml
```

**Node Not Starting**
```bash
# Check logs for errors
sudo journalctl -u 0g-alignment-node -n 100
```

**Registration Failed**
- Ensure NFT is in your wallet
- Check sufficient ETH for gas on Arbitrum
- Verify correct NFT token ID from Arbiscan

**Check Port is Open**
```bash
sudo ufw status | grep YOUR_PORT
```

---
That's it! Your node will run 24/7 automatically.
