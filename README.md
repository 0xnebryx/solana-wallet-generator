# solana-wallet-generator

> Generate and manage many encrypted Solana wallets safely — AES-256-GCM + PBKDF2 patterns.

Reference patterns for safely creating, storing, and using many Solana keypairs at scale. The focus is on making sure private-key material is encrypted at rest with industry-standard primitives.

---

## What "safe" means here

A naive multi-wallet generator stores keypairs in plaintext JSON. Two problems:

1. Anyone with filesystem access reads every private key
2. If the file ends up in a backup, container snapshot, log, or git commit — full compromise

The pattern below uses AES-256-GCM authenticated encryption + PBKDF2 key derivation so the data at rest is useless without the user's master password.

---

## Crypto choices

| Concern                | Choice                              | Why                                        |
|------------------------|-------------------------------------|--------------------------------------------|
| Symmetric cipher       | **AES-256-GCM**                     | Authenticated, fast, hardware-accelerated  |
| Key derivation         | **PBKDF2-SHA-256, 100K iterations** | Standard, slow enough to resist brute      |
| Salt                   | 16 random bytes per ciphertext      | Defeats rainbow tables                     |
| IV / nonce             | 12 random bytes per ciphertext      | GCM requirement, never reuse               |
| Auth tag               | 16 bytes (GCM default)              | Detects tampering                          |

```ts
// pseudo-code shape
import { createCipheriv, randomBytes, pbkdf2Sync } from 'crypto';

function encrypt(plaintext: string, password: string): string {
  const salt = randomBytes(16);
  const iv = randomBytes(12);
  const key = pbkdf2Sync(password, salt, 100_000, 32, 'sha256');
  const cipher = createCipheriv('aes-256-gcm', key, iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return [salt, iv, tag, encrypted].map(b => b.toString('base64')).join(':');
}
```

`decrypt()` reverses, including `decipher.setAuthTag(tag)` so tampered ciphertexts throw on `final()`.

---

## Storage layout

For a fleet of N wallets, store each as:

```json
{
  "id": "uuid-here",
  "publicKey": "GxK...",       // safe to log / index
  "encryptedSecret": "...",    // AES-GCM ciphertext of base58 secret
  "createdAt": "2026-06-09T00:00:00Z",
  "role": "BUNDLE | FOLLOW | TRADE | MAIN"
}
```

Public keys can live in a regular DB index. Encrypted secrets ride alongside but are never logged, never sent to the frontend, never copied to backups un-encrypted.

---

## Operational rules

1. **Master password lives only in the user's head** — never in env vars, config files, or transit. The backend derives the encryption key per-request.
2. **Failed decrypts are silent at the API layer** — never echo "wrong password" with stack traces.
3. **Rotate encryption schema if compromised** — store an `encVersion` field so future migrations are clean.
4. **Memory hygiene** — overwrite buffers after use (`buffer.fill(0)`) so decrypted keys don't linger in heap dumps.
5. **No key export without explicit auth challenge** — wallet export should require fresh password proof, not just session token.

---

## Footguns

- **PBKDF2 100K iterations** is the *minimum*. For higher security needs, bump to 600K (OWASP 2026 recommendation) or migrate to Argon2id.
- **GCM nonce reuse is catastrophic** — if you reuse a nonce for the same key, GCM is broken. Always re-randomize.
- **Don't roll your own crypto** — use the Node `crypto` module or `@noble/ciphers`, not custom XOR + SHA.
- **Backup the encrypted blob safely** — losing the encrypted DB AND the master password = losing all funds. Plan for one or the other being recoverable.
- **Solana keypairs have specific shapes** — `Keypair.fromSecretKey(bytes)` expects 64 bytes (32 secret + 32 public). Storing only the 32-byte secret + deriving public on load saves space.

## Reading list

- [Node.js `crypto` module docs](https://nodejs.org/api/crypto.html)
- [OWASP password storage cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Argon2 vs PBKDF2](https://github.com/p-h-c/phc-winner-argon2)
- [`@noble/ciphers`](https://github.com/paulmillr/noble-ciphers)

## License

MIT
