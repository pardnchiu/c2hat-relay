# C²hat - Cross-Domain Chat

[<img src=https://raw.githubusercontent.com/pardnchiu/pardnchiu/refs/heads/main/image/available-chrome-web-store-seeklogo.png height=40>](https://chromewebstore.google.com/detail/c2hat-cross-domain-chat/chngimmfgmkpninihhljpidnieocmhdn)

## End-to-End Encryption Main Flow

```mermaid
graph TD
  A[Open Extension] --> B[Check Domain<br>Establish WebSocket Connection<br>Join Chatroom<br>Generate SHA-256 Hash]
  B --> C{Check for Stored Key Pair}
  C -->|Not Found| C0[Generate New RSA-2048 Key Pair<br>Export Keys as Base64<br>Generate Encryption Password Using PBKDF2<br>Encrypt Private Key with AES-GCM<br>Save to Chrome Storage]
    C0 --> D1
  C -->|Found| C1[Load Existing Key Pair<br>Generate Password Using Display Name<br>Decrypt Private Key with PBKDF2]
    C1 --> D{Decryption Successful?}
    D -->|Failed| C0
    D -->|Successful| D1[Share Public Key to Server]
      D1 --> E[Show Chat Interface<br>Fetch Online Users] 
      E --> F{Chat Mode}
        F -->|Group Chat| G[Plaintext Message]
        F -->|Private Chat| H[Request Recipient's Public Key]
          H --> I{Public Key Retrieved?}
            I -->|Failed| I0[Cannot Establish Encrypted Connection]
            I -->|Successful| J[Cache Recipient's Public Key<br>Start Encrypted Conversation<br>Encrypt Message with RSA-OAEP]
              J --> K[Send Encrypted Message to Server]
      E --> L[Leave Chatroom]
      L --> M[Clear Server-Side Data]
      M --> N[Retain Local Encrypted Data]
```

## End-to-End Encryption Decryption Flow
```mermaid
graph TD
  A[Receive Encrypted Message] --> B{Message Type}
  B -->|Group Message| C[Display Plaintext Directly]
    C --> D[Save Locally by Domain]
    D --> F[Update Chat History]
  B -->|Private Message| G[Decrypt Using Private Key]
    G --> H{Decryption Successful?}
      H -->|Successful| I[Display Decrypted Message]
        I --> D
      H -->|Failed| J[Show Decryption Failed]
        J --> D
```

## Key Management Flow
- Key Pair Generation: Use `RSA-2048` for asymmetric keys
- Local Private Key Encryption: Use `AES-GCM 256` for secure storage
- Password Derivation: Strengthen password with `PBKDF2`
- Key Persistence: Store keys using `Chrome Storage API`

## Message Encryption Flow
- Sender Encryption: Use recipient's public key for `RSA-OAEP`
- Receiver Decryption: Use private key to decrypt message content
- Group/Private Mode: Automatically determine message type, enforce encryption for private messages

## Privacy Protection Mechanisms
- Zero Server Storage: Server does not store any message content, Cloudflare Worker acts as a relay
- Local Encrypted Storage: Use `Chrome Storage API` for encrypted chat history
- Instant Cleanup: Clear server-side cache after connection termination

## Performance Optimization Features
- Key Caching: Cache recipient's public key after successful connection to reduce redundant requests
- Auto Reconnect: Automatically attempt reconnection if WebSocket disconnects
- Message Queue: Temporarily store messages during network instability

## Special Design
- Domain Isolation: Ensure complete separation of messages across different websites
