# 0G AI Alignment Node - Simple Setup Guide

Quick and easy setup for 0G AI Alignment Node with systemd service.

## Requirements

- Ubuntu/Debian VPS
- 1 CPU, 64MB RAM, 10GB disk
- Node license NFT
- Wallet private key

## Installation

### 1. Setup Directory

```bash
cd ~
mkdir 0g-alignment-node
cd 0g-alignment-node
```

### 2. Download Node

```bash
wget https://github.com/0gfoundation/alignment-node-release/releases/download/v1.0.0/0g-alignment-node
chmod +x 0g-alignment-node
```

### 3. Configure Node

⚠️ **SECURITY WARNING**: 
- Your private key will be visible in logs and process lists
- Never share logs or screenshots that might contain your private key
- Consider using a dedicated wallet for node operations only

**Choose your port** (default: 8080, example: 42069):
```bash
export MY_PORT=42069  # Change this to any port you want
```

Create `.env` file:
```bash
cat > .env << EOF
ZG_ALIGNMENT_NODE_LOG_LEVEL=info
ZG_ALIGNMENT_NODE_SERVICE_PORT=$MY_PORT
ZG_ALIGNMENT_NODE_SERVICE_PRIVATEKEY=YOUR_PRIVATE_KEY_WITHOUT_0X
EOF

# Secure the file
chmod 600 .env
```

**IMPORTANT**: 
- Replace `YOUR_PRIVATE_KEY_WITHOUT_0X` with your actual private key (no 0x prefix)
- Keep this file secure and never commit to git

Create `config.yaml`:
```bash
cat > config.yaml << EOF
service:
  port: $MY_PORT
log:
  level: info
EOF
```

### 4. Open Your Port

```bash
sudo ufw allow $MY_PORT/tcp
sudo ufw allow 22/tcp
sudo ufw --force enable
```

### 5. Create Service

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

### 6. Start Node

```bash
sudo systemctl daemon-reload
sudo systemctl enable 0g-alignment-node
sudo systemctl start 0g-alignment-node
```

### 7. Check Status

```bash
# Check if running
sudo systemctl status 0g-alignment-node

# View logs
sudo journalctl -u 0g-alignment-node -f
```

## Port Information

The node uses the port you specify in `.env` file:
- **Default port**: 8080
- **Custom port**: Set any port you want (e.g., 42069, 30333, 50000)
- **Port usage**: Node listens on this port for consensus communication
- **Important**: Make sure your chosen port is not already in use

To check if a port is available:
```bash
sudo ss -tulpn | grep YOUR_PORT_NUMBER
```

## Delegation

1. Go to: https://claim.0gfoundation.ai/delegation
2. Connect wallet (use 0G Mainnet)
3. Enter your wallet address
4. Click "Delegate"

## Add 0G Network to MetaMask

- **Network Name**: 0G Mainnet
- **RPC**: https://evmrpc.0g.ai
- **Chain ID**: 16661
- **Symbol**: 0G

## Commands

```bash
# Stop node
sudo systemctl stop 0g-alignment-node

# Restart node
sudo systemctl restart 0g-alignment-node

# View logs
sudo journalctl -u 0g-alignment-node -f

# Check your port
sudo ss -tulpn | grep $(grep SERVICE_PORT .env | cut -d'=' -f2)
```

## Troubleshooting



**Private Key Security Warning**:
- Systemd logs may show your private key in error messages
- Never share logs publicly without checking for sensitive data
- Use `journalctl -u 0g-alignment-node -n 10 | grep -v PRIVATEKEY` for safer log viewing

**Port already in use**: Choose a different port number and update `.env` and `config.yaml`

**Check if port is open**:
```bash
sudo ufw status | grep YOUR_PORT
```

---

That's it! Your node will run 24/7 automatically on your chosen port.
