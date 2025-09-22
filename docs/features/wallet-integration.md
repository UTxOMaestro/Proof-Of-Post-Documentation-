# Wallet Integration

Proof of Post's wallet integration is the foundation that enables true decentralized social media. Instead of traditional username/password systems, the platform uses Cardano wallets for authentication, identity, and payments.

## Supported Wallets

### CIP-30 Compatible Wallets
Proof of Post supports these major Cardano wallets that implement the CIP-30 standard:

- **[Eternl](https://eternl.io/)** - Feature-rich wallet with advanced capabilities
- **[Lace](https://www.lace.io/)** - IOG's official light wallet
- **[Vespr](https://vespr.xyz/)** - Community-focused wallet

### Browser Extension Detection
The platform automatically detects installed wallets and presents connection options:

```typescript
// Automatic wallet detection
const availableWallets = Object.keys(window.cardano || {})
  .filter(key => window.cardano[key]?.isEnabled !== undefined);
```

## Authentication Process

### Wallet Connection Flow

1. **Wallet Detection**: Platform scans for available CIP-30 wallets
2. **User Selection**: Choose your preferred wallet from available options
3. **Permission Request**: Wallet asks for connection permission
4. **Address Retrieval**: Platform gets your wallet's receive address
5. **Challenge Generation**: Server creates a unique signing challenge
6. **Message Signing**: Wallet signs the challenge using CIP-8 standard
7. **Verification**: Server verifies the signature against your address
8. **Session Creation**: JWT token issued for authenticated sessions

### Security Features
- **Challenge-Response Authentication**: Prevents replay attacks
- **Signature Verification**: Cryptographic proof of wallet ownership
- **Session Binding**: JWT tokens tied to specific wallet addresses
- **Automatic Expiry**: Sessions expire for security
- **Multi-Device Support**: Connect the same wallet on multiple devices

## Identity Management

### Dual Identity System

#### Platform Usernames (`@username`)
- **User-chosen handles** for easy identification
- **Availability checking** to ensure uniqueness
- **Profile customization** with bios and images
- **Social graph building** through follower relationships

#### ADA Handles (`$handle`)
- **On-chain verification** through Blockfrost API
- **Automatic detection** when wallets connect
- **Verified badge display** for confirmed handles
- **Handle-based profile URLs** (e.g., `/profile/$myhandle`)

### Profile Information
Your wallet connection automatically populates:
- **Wallet Address**: Your primary Cardano address
- **ADA Balance**: Current ADA holdings display
- **Asset Holdings**: All tokens in your wallet
- **ADA Handle**: If you own one, it's automatically verified

## Financial Integration

### Balance Display *(Coming Soon)*
- **Real-time ADA balance** shown in your profile
- **Asset portfolio** display for all Cardano tokens
- **USD value conversion** for better understanding
- **Balance history** tracking over time

### Transaction Capabilities
- **Tip Payments**: Send ADA or tokens to creators
- **Content Purchases**: Buy access to paid media
- **Transaction Signing**: Secure approval for all payments
- **Payment Verification**: Blockchain confirmation of transactions

### Multi-Asset Support
- **Native Tokens**: Support for all Cardano native assets
- **NFT Display**: Show your NFT collection in profiles *(Coming Soon)*
- **Token Tipping**: Tip creators with any supported token
- **Asset Metadata**: Automatic token information lookup

## Security & Privacy

### Cryptographic Security
- **CIP-8 Message Signing**: Industry standard for wallet authentication
- **Address Verification**: Prevent wallet address spoofing
- **Signature Validation**: Every signed message is cryptographically verified
- **Session Security**: JWT tokens with wallet address binding

### Privacy Protection
- **Optional Address Display**: Choose whether to show your wallet address
- **Transaction Privacy**: Only you can see your full transaction history
- **Selective Sharing**: Control what wallet information is public
- **No Seed Phrase Storage**: Platform never accesses your private keys

### Post Signing
- **Optional Post Signatures**: Cryptographically sign your posts
- **Verification Badges**: Signed posts show verification indicators
- **Authenticity Proof**: Immutable proof of post authorship
- **Anti-Impersonation**: Prevent fake posts attributed to your wallet

## Session Management

### Persistent Sessions
- **Auto-Reconnection**: Automatic wallet reconnection when available
- **Session Restoration**: Restore authenticated state after browser restart
- **Multi-Tab Support**: Maintain session across browser tabs
- **Background Sync**: Keep wallet state synchronized

### Session Controls
- **Manual Disconnect**: Explicitly disconnect wallet when desired
- **Session Timeout**: Automatic expiry for security
- **Refresh Tokens**: Seamless session renewal

## Technical Implementation

### CIP-30 Integration
```typescript
// Wallet connection example
const connectWallet = async (walletName: string) => {
  try {
    const wallet = await window.cardano[walletName].enable();
    const api = await wallet.getApi();
    
    // Get wallet address
    const addresses = await api.getUsedAddresses();
    const address = addresses[0];
    
    // Request authentication challenge
    const challenge = await fetch('/api/auth/challenge', {
      method: 'POST',
      body: JSON.stringify({ address })
    });
    
    // Sign challenge
    const signature = await api.signData(
      Buffer.from(challenge).toString('hex'),
      Buffer.from(address).toString('hex')
    );
    
    // Verify and authenticate
    const auth = await fetch('/api/auth/verify', {
      method: 'POST',
      body: JSON.stringify({ address, signature, challenge })
    });
    
    return auth.json();
  } catch (error) {
    console.error('Wallet connection failed:', error);
  }
};
```

### Error Handling
- **Connection Failures**: Graceful handling of wallet connection issues
- **Network Errors**: Retry mechanisms for network problems
- **User Cancellation**: Proper handling when users cancel operations
- **Wallet Not Found**: Clear messaging for missing wallets

## Advanced Features

### Wallet Switching
- **Multi-Wallet Support**: Switch between different connected wallets
- **Wallet Verification**: Verify ownership of multiple addresses
- **Primary Wallet**: Designate main wallet for transactions

### Hardware Wallet Support *(Coming Soon)*
- **Ledger Integration**: Full support for Ledger hardware wallets
- **Trezor Compatibility**: Works with Trezor devices
- **Enhanced Security**: Hardware wallet signing for maximum security
- **Transaction Approval**: Clear transaction details for approval

### Developer Tools *(Internal)*
- **Wallet State Debugging**: Tools for developers to debug wallet integration
- **Transaction Simulation**: Test transactions without spending real ADA
- **Network Selection**: Switch between mainnet and testnet
- **API Monitoring**: Track wallet API calls and responses

---

The wallet integration makes Proof of Post truly decentralized by eliminating traditional authentication while providing enhanced security and direct financial capabilities.

Next: Explore the [Token Ecosystem](token-ecosystem.md) to see how Cardano tokens are integrated throughout the platform.
