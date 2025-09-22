# API Documentation

Complete reference for Proof of Post's RESTful API, enabling developers to build integrations and applications on top of the platform.

## 🔐 Authentication

### Wallet-Based Authentication
All API requests require JWT authentication obtained through wallet signature verification.

```typescript
// Authentication flow
const authResponse = await fetch('/api/auth/challenge', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ address: walletAddress })
});

const { challenge } = await authResponse.json();

// Sign with wallet (using Lucid or similar)
const signature = await wallet.signMessage(challenge);

const verifyResponse = await fetch('/api/auth/verify', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ address: walletAddress, signature, challenge })
});

const { token } = await verifyResponse.json();
```

### Using JWT Tokens
Include the JWT token in the Authorization header for authenticated requests:

```typescript
const response = await fetch('/api/posts', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});
```

## 📝 Posts API

### Create Post
```http
POST /api/posts
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "content": "Hello Cardano community! #cardano $ada",
  "mediaUrls": ["ipfs://QmHash..."],
  "isPaidContent": false,
  "priceLovelace": 0,
  "signature": "optional_post_signature"
}
```

### Get Posts Feed
```http
GET /api/posts?limit=20&offset=0&userId=optional
Authorization: Bearer <jwt_token>

Response:
{
  "posts": [
    {
      "id": "post_id",
      "userId": "user_id",
      "content": "Post content",
      "mediaUrls": ["ipfs://..."],
      "reactionCounts": {
        "like": 5,
        "rocket": 2,
        "diamond": 1,
        "degen": 0
      },
      "boostCount": 3,
      "replyCount": 7,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "user": {
        "username": "creator",
        "adaHandle": "$handle",
        "avatarUrl": "ipfs://..."
      }
    }
  ],
  "hasMore": true
}
```

### React to Post
```http
POST /api/posts/:postId/react
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "reactionType": "like" | "rocket" | "diamond" | "degen"
}
```

### Boost Post
```http
POST /api/posts/:postId/boost
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "comment": "Optional boost comment"
}
```

## 👥 Users API

### Get User Profile
```http
GET /api/users/:userId
Authorization: Bearer <jwt_token>

Response:
{
  "id": "user_id",
  "walletAddress": "addr1...",
  "username": "username",
  "adaHandle": "$handle",
  "displayName": "Display Name",
  "bio": "User bio",
  "avatarUrl": "ipfs://...",
  "bannerUrl": "ipfs://...",
  "followerCount": 150,
  "followingCount": 75,
  "postCount": 42,
  "isFollowing": true,
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

### Update Profile
```http
PUT /api/profile
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "username": "newusername",
  "displayName": "New Display Name",
  "bio": "Updated bio",
  "avatarUrl": "ipfs://new_avatar",
  "websiteUrl": "https://example.com",
  "twitterHandle": "twitter_handle"
}
```

### Follow/Unfollow User
```http
POST /api/users/:userId/follow
Authorization: Bearer <jwt_token>

DELETE /api/users/:userId/follow
Authorization: Bearer <jwt_token>
```

## 🔍 Search API

### Universal Search
```http
GET /api/search?q=cardano&type=all&limit=20&offset=0
Authorization: Bearer <jwt_token>

Response:
{
  "posts": [...],
  "users": [...],
  "tokens": [...],
  "topics": [...]
}
```

### Search Types
- `all` - Search everything
- `posts` - Posts only
- `users` - Users only  
- `tokens` - Tokens only
- `topics` - Hashtags only

## 🪙 Tokens API

### Get Token Information
```http
GET /api/tokens/:tokenId
Authorization: Bearer <jwt_token>

Response:
{
  "id": "ada",
  "policyId": "",
  "assetName": "",
  "ticker": "ADA",
  "name": "Cardano",
  "description": "Cardano native token",
  "logoUrl": "ipfs://...",
  "websiteUrl": "https://cardano.org",
  "currentPrice": 0.45,
  "priceChange24h": 5.2,
  "marketCap": 15000000000,
  "volume24h": 250000000,
  "holderCount": 1000000
}
```

### Search Tokens
```http
GET /api/tokens/search?q=ada&limit=10
Authorization: Bearer <jwt_token>
```

### Get Token Mentions
```http
GET /api/tokens/:tokenId/mentions?limit=20&offset=0
Authorization: Bearer <jwt_token>
```

## 💰 Payments API

### Create Payment Intent
```http
POST /api/payments/intent
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "postId": "post_id",
  "type": "paid_content" | "tip",
  "amountLovelace": 5000000,
  "recipientAddress": "addr1..."
}

Response:
{
  "paymentId": "payment_id",
  "recipientAddress": "addr1...",
  "amountLovelace": 5000000,
  "expiresAt": "2024-01-01T01:00:00.000Z"
}
```

### Verify Payment
```http
POST /api/payments/:paymentId/verify
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "transactionHash": "tx_hash"
}
```

### Get Payment History
```http
GET /api/payments/history?limit=20&offset=0
Authorization: Bearer <jwt_token>
```

## 📁 Media API

### Upload to IPFS
```http
POST /api/ipfs/upload
Content-Type: multipart/form-data
Authorization: Bearer <jwt_token>

Form data:
- file: <binary_file_data>

Response:
{
  "cid": "QmHash...",
  "url": "https://ipfs.io/ipfs/QmHash...",
  "size": 1024000
}
```

## 📊 Analytics API

### Get User Analytics
```http
GET /api/analytics/user
Authorization: Bearer <jwt_token>

Response:
{
  "totalPosts": 150,
  "totalReactions": 1250,
  "totalTipsReceived": 45.5,
  "followerGrowth": [
    { "date": "2024-01-01", "count": 100 },
    { "date": "2024-01-02", "count": 105 }
  ],
  "topPosts": [...]
}
```

## 🔔 Notifications API

### Get Notifications
```http
GET /api/notifications?limit=20&offset=0&unreadOnly=false
Authorization: Bearer <jwt_token>

Response:
{
  "notifications": [
    {
      "id": "notif_id",
      "type": "reaction" | "follow" | "mention" | "tip" | "boost",
      "fromUser": { "username": "user", "avatarUrl": "..." },
      "postId": "optional_post_id",
      "message": "liked your post",
      "isRead": false,
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "unreadCount": 5
}
```

### Mark Notifications Read
```http
PUT /api/notifications/read
Content-Type: application/json
Authorization: Bearer <jwt_token>

{
  "notificationIds": ["id1", "id2", "id3"]
}
```

## 🚨 Error Handling

### Standard Error Response
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request parameters are invalid",
    "details": {
      "field": "content",
      "issue": "Content cannot be empty"
    }
  }
}
```

### Common Error Codes
- `UNAUTHORIZED` - Invalid or missing JWT token
- `FORBIDDEN` - Insufficient permissions
- `NOT_FOUND` - Resource not found
- `VALIDATION_ERROR` - Request validation failed
- `RATE_LIMITED` - Too many requests
- `BLOCKCHAIN_ERROR` - Blockchain interaction failed

## 📈 Rate Limits

- **Authenticated requests**: 1000 requests per hour
- **Post creation**: 10 posts per minute
- **Search requests**: 60 requests per minute
- **File uploads**: 5 uploads per minute

Rate limit headers included in responses:
- `X-RateLimit-Limit`: Request limit per window
- `X-RateLimit-Remaining`: Remaining requests
- `X-RateLimit-Reset`: Window reset time

## 🔗 Webhooks (Coming Soon)

Subscribe to real-time events for your applications:
- New posts from followed users
- Reactions on your posts
- New followers
- Tips received
- Mentions

Ready to integrate? Check out the [Integration Guide](integration-guide.md) for practical examples.
