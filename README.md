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

## Crypto choices (general)

For symmetric-key encryption at rest, AES-256-GCM is the standard choice — authenticated, hardware-accelerated, and broadly supported. Pair it with a memory-hard or slow KDF (Argon2id is best of breed; PBKDF2 is acceptable if iteration count is current with OWASP guidance). Always use a fresh random salt and a fresh random nonce per ciphertext; never reuse a nonce under the same key.

The exact serialization format (how you encode salt/IV/tag/ciphertext) is an implementation detail, but the rules are universal:

- Salt must be random per encryption operation
- Nonce/IV must be random per encryption operation (reuse breaks GCM completely)
- Auth tag must be verified on decrypt — tampered ciphertexts should throw
- Iteration count for the KDF must keep pace with hardware — what was strong in 2020 is weak in 2026

---

## Storage layout (conceptual)

Separate the public-safe metadata (pubkey, id, timestamps, optional role tag) from the encrypted secret material. The metadata can live in a regular DB index. The encrypted secrets ride alongside but are never logged, never sent to the frontend, never copied to backups un-encrypted.

---

## Operational rules

1. **Master password lives only in the user's head** — never in env vars, config files, or transit. The backend derives the encryption key per-request.
2. **Failed decrypts are silent at the API layer** — never echo "wrong password" with stack traces.
3. **Rotate encryption schema if compromised** — store an `encVersion` field so future migrations are clean.
4. **Memory hygiene** — overwrite buffers after use (`buffer.fill(0)`) so decrypted keys don't linger in heap dumps.
5. **No key export without explicit auth challenge** — wallet export should require fresh password proof, not just session token.

---

## Footguns

- **PBKDF2 iteration counts must keep pace with hardware**. Check the current OWASP recommendation; what was strong years ago is weak now. For higher-security applications, prefer Argon2id.
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
