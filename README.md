# 👛 Solana Multi-Wallet Generator

> Generate and manage up to 100 Solana wallets with military-grade encryption

[![Wallets](https://img.shields.io/badge/Wallets-100%20Max-blue?style=for-the-badge)](https://obsidianbundler.com)
[![Encryption](https://img.shields.io/badge/Encryption-AES--256-green?style=for-the-badge)](https://obsidianbundler.com)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

## 🔐 Why Multiple Wallets?

Single-wallet trading is a liability in 2025:

- **Privacy**: Every transaction is public and traceable
- **Copy Trading**: Successful wallets get watched and copied
- **Analytics**: Tools like BubbleMaps expose your holdings
- **Front-running**: Known wallets get targeted by bots

Multi-wallet strategies solve these problems.

## Features

```
┌─────────────────────────────────────────────────────────┐
│              MULTI-WALLET GENERATOR                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ✅ Generate 1-100 wallets instantly                    │
│  ✅ AES-256-GCM encryption for private keys             │
│  ✅ PBKDF2 key derivation (100,000 iterations)          │
│  ✅ Export keys anytime (you own your keys)             │
│  ✅ Hard disperse for invisible funding                 │
│  ✅ Bulk balance checking                               │
│  ✅ Coordinated transactions                            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Quick Start

```typescript
import { WalletGenerator, WalletManager } from '@obsidian/wallet-tools';

// Generate wallets
const generator = new WalletGenerator({
  encryption: {
    algorithm: 'aes-256-gcm',
    keyDerivation: 'pbkdf2',
    iterations: 100000
  }
});

// Create 20 new wallets
const wallets = await generator.generate(20);

console.log(`Generated ${wallets.length} wallets`);
wallets.forEach((w, i) => {
  console.log(`Wallet ${i + 1}: ${w.publicKey}`);
});

// Export private keys (encrypted)
const exported = await generator.export(wallets, 'my-password');
```

## Encryption Details

We use the same encryption standards as financial institutions:

```typescript
interface EncryptionConfig {
  algorithm: 'aes-256-gcm';     // Military-grade encryption
  keyDerivation: 'pbkdf2';      // Password-Based Key Derivation
  iterations: 100000;            // Brute-force resistant
  saltLength: 32;                // Random salt per key
  ivLength: 16;                  // Random IV per encryption
}
```

### How It Works

```
Your Password
     │
     ▼
┌─────────────────┐
│    PBKDF2       │  100,000 iterations
│  Key Derivation │  + random salt
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   AES-256-GCM   │  + random IV
│   Encryption    │  + auth tag
└────────┬────────┘
         │
         ▼
  Encrypted Private Key
  (Safe to store)
```

## Hard Disperse: Invisible Funding

The biggest tell in multi-wallet setups is the funding trail. If you send SOL from wallet A to wallets B, C, D - they're now linked.

**Hard Disperse breaks this link:**

```typescript
import { HardDisperse } from '@obsidian/wallet-tools';

const disperser = new HardDisperse();

// Fund wallets invisibly
await disperser.disperse({
  source: mainWallet,
  targets: [wallet1, wallet2, wallet3],
  amountPerWallet: 0.5, // SOL
  
  // Each transfer routes through a temporary intermediate
  // that's closed after use - NO DIRECT LINK
});
```

```
WITHOUT Hard Disperse (VISIBLE):
Main Wallet ──► Wallet 1
Main Wallet ──► Wallet 2
Main Wallet ──► Wallet 3
(All clearly connected on BubbleMaps)

WITH Hard Disperse (INVISIBLE):
Main Wallet ──► Temp A ──► Wallet 1 (Temp A closed)
Main Wallet ──► Temp B ──► Wallet 2 (Temp B closed)
Main Wallet ──► Temp C ──► Wallet 3 (Temp C closed)
(No visible connection between wallets)
```

## Wallet Manager

```typescript
import { WalletManager } from '@obsidian/wallet-tools';

const manager = new WalletManager(wallets);

// Check all balances
const balances = await manager.getBalances();
console.log('Total SOL:', balances.total);

// Transfer from one wallet to another
await manager.transfer({
  from: wallet1,
  to: wallet2,
  amount: 1.5 // SOL
});

// Collect all SOL to one wallet
await manager.collectAll({
  destination: mainWallet,
  leaveRent: true // Leave 0.002 SOL for rent
});

// Execute coordinated buys
await manager.coordinatedBuy({
  wallets: [wallet1, wallet2, wallet3],
  tokenMint: 'TOKEN_ADDRESS',
  amountPerWallet: 0.1, // SOL
  useJito: true // Same-block execution
});
```

## Security Best Practices

```typescript
// ✅ DO: Use strong passwords
const password = 'MyStr0ng!P@ssw0rd#2025';

// ✅ DO: Export and backup keys
const backup = await generator.export(wallets, password);
fs.writeFileSync('backup.enc', backup);

// ✅ DO: Store backups securely
// - Hardware wallet
// - Encrypted USB drive
// - Password manager

// ❌ DON'T: Store unencrypted keys
// ❌ DON'T: Share your password
// ❌ DON'T: Use weak passwords
// ❌ DON'T: Keep all funds in hot wallets
```

## Use Cases

### Token Launching

```typescript
// Generate launch wallets
const launchWallets = await generator.generate(30);

// Fund invisibly
await disperser.hardDisperse({
  source: mainWallet,
  targets: launchWallets,
  amountPerWallet: 0.2
});

// Warm up wallets
await warmup.warmupBatch(launchWallets, { intensity: 'medium' });

// Launch with bundle
await bundleLauncher.launch({
  token: tokenConfig,
  devWallet: launchWallets[0],
  sniperWallets: launchWallets.slice(1, 10)
});
```

### Stealth Accumulation

```typescript
// Accumulate token across multiple wallets
// Looks like many small buyers, not one whale

for (const wallet of wallets) {
  await jupiter.swap({
    wallet,
    inputMint: SOL,
    outputMint: TARGET_TOKEN,
    amount: randomBetween(0.05, 0.15), // Varying amounts
    delay: randomBetween(30, 120) // Random delays
  });
}
```

### Private Selling

```typescript
// Sell from multiple wallets over time
// Doesn't trigger "whale selling" alerts

for (const wallet of walletsWithToken) {
  await jupiter.swap({
    wallet,
    inputMint: TARGET_TOKEN,
    outputMint: SOL,
    amount: await getBalance(wallet, TARGET_TOKEN) * 0.1, // 10%
    delay: randomBetween(60, 300)
  });
}
```

## Production Solution

For a complete wallet management platform with UI:

### 👉 [Obsidian Platform](https://obsidianbundler.com)

- ✅ Generate up to 100 wallets
- ✅ AES-256 encryption
- ✅ Hard disperse built-in
- ✅ Wallet warmup
- ✅ Coordinated trading
- ✅ Export anytime
- ✅ Free tier (10 wallets)

## Resources

- 💬 [Telegram Community](https://t.me/obsidianbundler)
- 🐦 [Twitter](https://x.com/obsidianbundler)
- 🌐 [Platform](https://obsidianbundler.com)

## Disclaimer

Educational code. Cryptocurrency involves risk. Secure your keys properly.

---

⭐ **Star this repo** if it helped you!

👛 **Manage wallets like a pro:** [obsidianbundler.com](https://obsidianbundler.com)
