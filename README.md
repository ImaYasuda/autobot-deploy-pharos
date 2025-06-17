# Pharos Multi-Wallet Bot

🚀 Auto deploy TX menggunakan banyak wallet di Pharos testnet.

## Persiapan

1. **Buat folder baru dan masuk ke dalamnya**
    ```bash
    mkdir pharos-multiwallet-bot && cd pharos-multiwallet-bot
    ```

2. **Inisialisasi npm**
    ```bash
    npm init -y
    ```

3. **Install dependensi**
    ```bash
    npm install ethers@5 dotenv
    ```

4. **Buat file .gitignore**
    ```bash
    echo ".env\nnode_modules/" > .gitignore
    ```

5. **Buat contoh file .env**
    ```bash
    cat <<EOF > .env.example
    RPC_URL=https://testnet.dplabs-internal.com
    PRIVATE_KEYS=0xabc...,0xdef...
    TO_ADDRESS=0x178f55FaA0845Ae2e6348d53B9Ff3E869916b939
    TX_DATA=0x775c300c
    GAS_LIMIT=342010
    GAS_PRICE=1200000000
    VALUE=146000000000000
    EOF
    ```

6. **Copy .env.example ke .env lalu isi dengan data kamu**
    ```bash
    cp .env.example .env
    # Edit file .env dan masukkan private key serta parameter lainnya
    ```

7. **Buat file autobot.js dengan kode berikut**
    ```js
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
    ```

8. **Jalankan bot**
    ```bash
    node autobot.js
    ```

---

cara no2 
# Pharos Autobot Deploy

🔁 Auto deploy transaction using multiple wallets on the Pharos testnet.

## 🧪 Features

- 🧠 Support unlimited private keys
- 🔁 Sends multiple transactions per wallet (default 10)
- 🕒 Includes delay to avoid nonce/gas errors

## ⚙️ How to Use

```bash
# 1. Clone the repo
git clone https://github.com/ImaYasuda/autobot-deploy-pharos.git
cd autobot-deploy-pharos

# 2. Install dependencies
npm install

# 3. Copy and edit environment config
cp .env.example .env
# then edit `.env` to fill in your private keys and other values

# 4. Run the bot
node autobot.js



