# Integration Guide

Practical guide for integrating with Proof of Post, including code examples, best practices, and common integration patterns.

## 🚀 Quick Start Integration

### Basic Setup
```typescript
import { ProofOfPostClient } from '@proofofpost/sdk';

const client = new ProofOfPostClient({
  baseUrl: 'https://api.proofofpost.io',
  walletProvider: 'eternl', // or 'lace', 'vespr'
});

// Connect wallet and authenticate
await client.connect();
const isAuthenticated = await client.authenticate();
```

### Environment Setup
```bash
# Install dependencies
npm install @proofofpost/sdk lucid-cardano

# Environment variables
PROOF_OF_POST_API_URL=https://api.proofofpost.io
CARDANO_NETWORK=mainnet
BLOCKFROST_PROJECT_ID=your_project_id
```

## 🔐 Authentication Integration

### Wallet Connection Flow
```typescript
class WalletAuth {
  private wallet: any;
  private token: string | null = null;

  async connectWallet(walletName: string) {
    try {
      // Check if wallet is available
      if (!window.cardano?.[walletName]) {
        throw new Error(`${walletName} wallet not found`);
      }

      // Enable wallet
      this.wallet = await window.cardano[walletName].enable();
      
      // Get wallet address
      const addresses = await this.wallet.getUsedAddresses();
      const address = addresses[0];

      return { success: true, address };
    } catch (error) {
      console.error('Wallet connection failed:', error);
      return { success: false, error: error.message };
    }
  }

  async authenticate(address: string) {
    try {
      // Get challenge from server
      const challengeResponse = await fetch('/api/auth/challenge', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ address })
      });

      const { challenge } = await challengeResponse.json();

      // Sign challenge with wallet
      const signature = await this.wallet.signData(
        Buffer.from(challenge).toString('hex'),
        Buffer.from(address).toString('hex')
      );

      // Verify signature and get JWT
      const authResponse = await fetch('/api/auth/verify', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ address, signature, challenge })
      });

      const { token } = await authResponse.json();
      this.token = token;

      return { success: true, token };
    } catch (error) {
      console.error('Authentication failed:', error);
      return { success: false, error: error.message };
    }
  }

  getAuthHeaders() {
    return {
      'Authorization': `Bearer ${this.token}`,
      'Content-Type': 'application/json'
    };
  }
}
```

## 📝 Content Management

### Creating Posts
```typescript
class PostManager {
  constructor(private auth: WalletAuth) {}

  async createPost(content: string, options: {
    mediaFiles?: File[];
    isPaidContent?: boolean;
    priceAda?: number;
    signPost?: boolean;
  } = {}) {
    try {
      let mediaUrls: string[] = [];

      // Upload media files if provided
      if (options.mediaFiles?.length) {
        for (const file of options.mediaFiles) {
          const uploadResponse = await this.uploadMedia(file);
          mediaUrls.push(uploadResponse.url);
        }
      }

      // Prepare post data
      const postData = {
        content,
        mediaUrls,
        isPaidContent: options.isPaidContent || false,
        priceLovelace: options.priceAda ? options.priceAda * 1_000_000 : 0
      };

      // Sign post if requested
      if (options.signPost) {
        postData.signature = await this.signPost(postData);
      }

      // Create post
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: this.auth.getAuthHeaders(),
        body: JSON.stringify(postData)
      });

      return await response.json();
    } catch (error) {
      console.error('Post creation failed:', error);
      throw error;
    }
  }

  private async uploadMedia(file: File): Promise<{ url: string; cid: string }> {
    const formData = new FormData();
    formData.append('file', file);

    const response = await fetch('/api/ipfs/upload', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${this.auth.token}` },
      body: formData
    });

    return await response.json();
  }

  private async signPost(postData: any): Promise<string> {
    const message = JSON.stringify(postData);
    return await this.auth.wallet.signData(
      Buffer.from(message).toString('hex'),
      Buffer.from(this.auth.address).toString('hex')
    );
  }
}
```

### Feed Integration
```typescript
class FeedManager {
  constructor(private auth: WalletAuth) {}

  async getFeed(options: {
    limit?: number;
    offset?: number;
    userId?: string;
    includeBoosts?: boolean;
  } = {}) {
    const params = new URLSearchParams({
      limit: (options.limit || 20).toString(),
      offset: (options.offset || 0).toString(),
      includeBoosts: (options.includeBoosts !== false).toString()
    });

    if (options.userId) {
      params.append('userId', options.userId);
    }

    const response = await fetch(`/api/posts?${params}`, {
      headers: this.auth.getAuthHeaders()
    });

    return await response.json();
  }

  async reactToPost(postId: string, reactionType: 'like' | 'rocket' | 'diamond' | 'degen') {
    const response = await fetch(`/api/posts/${postId}/react`, {
      method: 'POST',
      headers: this.auth.getAuthHeaders(),
      body: JSON.stringify({ reactionType })
    });

    return await response.json();
  }

  async boostPost(postId: string, comment?: string) {
    const response = await fetch(`/api/posts/${postId}/boost`, {
      method: 'POST',
      headers: this.auth.getAuthHeaders(),
      body: JSON.stringify({ comment })
    });

    return await response.json();
  }
}
```

## 💰 Payment Integration

### Tipping System
```typescript
class PaymentManager {
  constructor(private auth: WalletAuth) {}

  async tipPost(postId: string, amountAda: number, message?: string) {
    try {
      // Create payment intent
      const intentResponse = await fetch('/api/payments/intent', {
        method: 'POST',
        headers: this.auth.getAuthHeaders(),
        body: JSON.stringify({
          postId,
          type: 'tip',
          amountLovelace: amountAda * 1_000_000,
          message
        })
      });

      const { paymentId, recipientAddress, amountLovelace } = await intentResponse.json();

      // Build transaction using Lucid
      const lucid = await Lucid.new(
        new Blockfrost('https://cardano-mainnet.blockfrost.io/api/v0', 'your_project_id'),
        'Mainnet'
      );

      lucid.selectWallet(this.auth.wallet);

      const tx = await lucid
        .newTx()
        .payToAddress(recipientAddress, { lovelace: BigInt(amountLovelace) })
        .complete();

      const signedTx = await tx.sign().complete();
      const txHash = await signedTx.submit();

      // Verify payment
      const verifyResponse = await fetch(`/api/payments/${paymentId}/verify`, {
        method: 'POST',
        headers: this.auth.getAuthHeaders(),
        body: JSON.stringify({ transactionHash: txHash })
      });

      return await verifyResponse.json();
    } catch (error) {
      console.error('Tip payment failed:', error);
      throw error;
    }
  }

  async purchasePaidContent(postId: string) {
    try {
      // Get post details to check price
      const postResponse = await fetch(`/api/posts/${postId}`, {
        headers: this.auth.getAuthHeaders()
      });
      const post = await postResponse.json();

      if (!post.isPaidContent) {
        throw new Error('Post is not paid content');
      }

      // Create payment intent
      const intentResponse = await fetch('/api/payments/intent', {
        method: 'POST',
        headers: this.auth.getAuthHeaders(),
        body: JSON.stringify({
          postId,
          type: 'paid_content',
          amountLovelace: post.priceLovelace
        })
      });

      const { paymentId, recipientAddress, amountLovelace } = await intentResponse.json();

      // Process payment (similar to tip flow)
      // ... payment processing code ...

      return { success: true, txHash };
    } catch (error) {
      console.error('Content purchase failed:', error);
      throw error;
    }
  }
}
```

## 🔍 Search & Discovery

### Search Integration
```typescript
class SearchManager {
  constructor(private auth: WalletAuth) {}

  async search(query: string, options: {
    type?: 'all' | 'posts' | 'users' | 'tokens' | 'topics';
    limit?: number;
    offset?: number;
  } = {}) {
    const params = new URLSearchParams({
      q: query,
      type: options.type || 'all',
      limit: (options.limit || 20).toString(),
      offset: (options.offset || 0).toString()
    });

    const response = await fetch(`/api/search?${params}`, {
      headers: this.auth.getAuthHeaders()
    });

    return await response.json();
  }

  async getTrendingTopics(limit = 10) {
    const response = await fetch(`/api/topics/trending?limit=${limit}`, {
      headers: this.auth.getAuthHeaders()
    });

    return await response.json();
  }

  async getTokenInfo(tokenId: string) {
    const response = await fetch(`/api/tokens/${tokenId}`, {
      headers: this.auth.getAuthHeaders()
    });

    return await response.json();
  }
}
```

## 🎨 UI Components

### React Component Examples
```tsx
import React, { useState, useEffect } from 'react';

// Post Component
export const PostCard: React.FC<{ post: Post }> = ({ post }) => {
  const [reactions, setReactions] = useState(post.reactionCounts);

  const handleReaction = async (type: string) => {
    try {
      await postManager.reactToPost(post.id, type);
      setReactions(prev => ({
        ...prev,
        [type]: prev[type] + 1
      }));
    } catch (error) {
      console.error('Reaction failed:', error);
    }
  };

  return (
    <div className="post-card">
      <div className="post-header">
        <img src={post.user.avatarUrl} alt={post.user.username} />
        <span>@{post.user.username}</span>
        {post.user.adaHandle && <span>${post.user.adaHandle}</span>}
      </div>
      
      <div className="post-content">
        {post.content}
        {post.mediaUrls.map(url => (
          <img key={url} src={url} alt="Post media" />
        ))}
      </div>

      <div className="post-actions">
        <button onClick={() => handleReaction('like')}>
          ❤️ {reactions.like}
        </button>
        <button onClick={() => handleReaction('rocket')}>
          🚀 {reactions.rocket}
        </button>
        <button onClick={() => handleReaction('diamond')}>
          💎 {reactions.diamond}
        </button>
      </div>
    </div>
  );
};

// Wallet Connect Component
export const WalletConnector: React.FC = () => {
  const [isConnecting, setIsConnecting] = useState(false);
  const [connectedWallet, setConnectedWallet] = useState<string | null>(null);

  const connectWallet = async (walletName: string) => {
    setIsConnecting(true);
    try {
      const result = await walletAuth.connectWallet(walletName);
      if (result.success) {
        const authResult = await walletAuth.authenticate(result.address);
        if (authResult.success) {
          setConnectedWallet(walletName);
        }
      }
    } catch (error) {
      console.error('Connection failed:', error);
    } finally {
      setIsConnecting(false);
    }
  };

  return (
    <div className="wallet-connector">
      {!connectedWallet ? (
        <div className="wallet-options">
          <button onClick={() => connectWallet('eternl')} disabled={isConnecting}>
            Connect Eternl
          </button>
          <button onClick={() => connectWallet('lace')} disabled={isConnecting}>
            Connect Lace
          </button>
          <button onClick={() => connectWallet('vespr')} disabled={isConnecting}>
            Connect Vespr
          </button>
        </div>
      ) : (
        <div className="wallet-connected">
          Connected: {connectedWallet}
        </div>
      )}
    </div>
  );
};
```

## 🔔 Real-time Updates

### WebSocket Integration
```typescript
class RealtimeManager {
  private ws: WebSocket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;

  connect(token: string) {
    const wsUrl = `wss://api.proofofpost.io/ws?token=${token}`;
    this.ws = new WebSocket(wsUrl);

    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.ws.onclose = () => {
      console.log('WebSocket disconnected');
      this.attemptReconnect(token);
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  private handleMessage(data: any) {
    switch (data.type) {
      case 'new_post':
        this.onNewPost(data.post);
        break;
      case 'reaction':
        this.onReaction(data.postId, data.reaction);
        break;
      case 'tip':
        this.onTip(data.postId, data.tip);
        break;
      case 'notification':
        this.onNotification(data.notification);
        break;
    }
  }

  private attemptReconnect(token: string) {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      setTimeout(() => {
        console.log(`Reconnecting... Attempt ${this.reconnectAttempts}`);
        this.connect(token);
      }, Math.pow(2, this.reconnectAttempts) * 1000);
    }
  }

  // Event handlers
  onNewPost: (post: any) => void = () => {};
  onReaction: (postId: string, reaction: any) => void = () => {};
  onTip: (postId: string, tip: any) => void = () => {};
  onNotification: (notification: any) => void = () => {};
}
```

## 🚨 Error Handling

### Comprehensive Error Handling
```typescript
class ProofOfPostError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode?: number,
    public details?: any
  ) {
    super(message);
    this.name = 'ProofOfPostError';
  }
}

class ErrorHandler {
  static handle(error: any): ProofOfPostError {
    if (error instanceof ProofOfPostError) {
      return error;
    }

    // Network errors
    if (error.name === 'NetworkError' || !navigator.onLine) {
      return new ProofOfPostError(
        'NETWORK_ERROR',
        'Network connection failed. Please check your internet connection.',
        0
      );
    }

    // API errors
    if (error.status) {
      switch (error.status) {
        case 401:
          return new ProofOfPostError('UNAUTHORIZED', 'Authentication required');
        case 403:
          return new ProofOfPostError('FORBIDDEN', 'Insufficient permissions');
        case 404:
          return new ProofOfPostError('NOT_FOUND', 'Resource not found');
        case 429:
          return new ProofOfPostError('RATE_LIMITED', 'Too many requests');
        default:
          return new ProofOfPostError('API_ERROR', error.message || 'API request failed');
      }
    }

    // Wallet errors
    if (error.message?.includes('wallet')) {
      return new ProofOfPostError('WALLET_ERROR', 'Wallet operation failed');
    }

    // Generic error
    return new ProofOfPostError('UNKNOWN_ERROR', error.message || 'An unexpected error occurred');
  }
}
```

## 📊 Best Practices

### Performance Optimization
- **Implement caching** for frequently accessed data
- **Use pagination** for large data sets
- **Optimize images** before uploading to IPFS
- **Batch API requests** when possible
- **Implement retry logic** for failed requests

### Security Guidelines
- **Validate all inputs** before sending to API
- **Never store private keys** in your application
- **Use HTTPS** for all API communications
- **Implement rate limiting** in your client
- **Sanitize user content** to prevent XSS

### User Experience
- **Show loading states** during operations
- **Provide clear error messages** to users
- **Implement offline support** where possible
- **Use optimistic updates** for instant feedback
- **Cache content** for faster loading

Ready to build? Start with the [API Documentation](api-documentation.md) for detailed endpoint information.
