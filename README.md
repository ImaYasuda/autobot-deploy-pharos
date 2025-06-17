# autobot-deploy-pharos
only for tesnet


# Buat folder baru
mkdir pharos-multiwallet-bot && cd pharos-multiwallet-bot

# Inisialisasi npm
npm init -y

# Install dependensi
npm install ethers@5 dotenv

# Buat file .gitignore
echo ".env\nnode_modules/" > .gitignore

# Buat contoh .env
cat <<EOF > .env.example
RPC_URL=https://testnet.dplabs-internal.com
PRIVATE_KEYS=0xabc...,0xdef...
TO_ADDRESS=0x178f55FaA0845Ae2e6348d53B9Ff3E869916b939
TX_DATA=0x775c300c
GAS_LIMIT=342010
GAS_PRICE=1200000000
VALUE=146000000000000
EOF

# Buat autobot.js
cat <<'EOF' > autobot.js
require('dotenv').config();
const { ethers } = require('ethers');

const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
const TO = process.env.TO_ADDRESS;
const DATA = process.env.TX_DATA;
const GAS_LIMIT = parseInt(process.env.GAS_LIMIT);
const GAS_PRICE = ethers.BigNumber.from(process.env.GAS_PRICE);
const VALUE = ethers.BigNumber.from(process.env.VALUE);
const keys = process.env.PRIVATE_KEYS.split(',');

const TX_PER_WALLET = 10;

async function sendTx(wallet, index, txIndex) {
  try {
    const tx = {
      to: TO,
      data: DATA,
      gasLimit: GAS_LIMIT,
      gasPrice: GAS_PRICE,
      value: VALUE,
    };
    const sent = await wallet.sendTransaction(tx);
    console.log(`✅ Wallet[${index + 1}] TX[${txIndex + 1}] → ${sent.hash}`);
    await sent.wait();
  } catch (err) {
    console.error(`❌ Wallet[${index + 1}] TX[${txIndex + 1}] ERROR: ${err.message}`);
  }
}

async function deploy(wallet, index) {
  for (let j = 0; j < TX_PER_WALLET; j++) {
    await sendTx(wallet, index, j);
    await new Promise(res => setTimeout(res, 2000));
  }
}

async function main() {
  for (let i = 0; i < keys.length; i++) {
    const wallet = new ethers.Wallet(keys[i].trim(), provider);
    console.log(`🚀 Mulai Wallet ${i + 1}: ${wallet.address}`);
    await deploy(wallet, i);
  }
}

main();
EOF

# Buat README.md
cat <<EOF > README.md
# Pharos Multi-Wallet Bot

🚀 Auto deploy TX using multiple wallets on Pharos testnet.

## Setup

```bash
npm install
cp .env.example .env
# edit .env and fill in your private keys and values
node autobot.js
