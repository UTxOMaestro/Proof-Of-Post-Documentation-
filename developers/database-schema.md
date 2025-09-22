# Database Schema

Complete database schema documentation for Proof of Post, built on SQLite with Drizzle ORM for type safety and performance.

## 🏗️ Schema Overview

The platform uses a relational database design optimized for social media interactions and blockchain integration.

### Core Tables
- **users** - User profiles and authentication
- **posts** - Content and post metadata
- **sessions** - JWT authentication sessions
- **reactions** - User reactions to posts
- **boosts** - Post sharing/retweeting
- **follows** - User follow relationships
- **comments** - Post replies and comments
- **bookmarks** - Saved posts
- **tips** - Token tips sent to posts
- **topics** - Hashtag tracking
- **tokens** - Token metadata cache
- **paid_posts** - Monetized content configuration
- **purchases** - Payment tracking
- **media** - IPFS file metadata

## 👥 Users Table

```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  wallet_address TEXT UNIQUE NOT NULL,
  username TEXT UNIQUE,
  ada_handle TEXT UNIQUE,
  display_name TEXT,
  bio TEXT,
  avatar_url TEXT,
  banner_url TEXT,
  website_url TEXT,
  twitter_handle TEXT,
  discord_handle TEXT,
  telegram_handle TEXT,
  follower_count INTEGER DEFAULT 0,
  following_count INTEGER DEFAULT 0,
  post_count INTEGER DEFAULT 0,
  reputation_score INTEGER DEFAULT 0,
  is_verified BOOLEAN DEFAULT FALSE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_users_wallet_address ON users(wallet_address);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_ada_handle ON users(ada_handle);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### User Fields
- `id` - Unique user identifier (UUID)
- `wallet_address` - Cardano wallet address (unique)
- `username` - Platform username (@username)
- `ada_handle` - On-chain ADA handle ($handle)
- `display_name` - User's display name
- `bio` - User biography (max 280 chars)
- `avatar_url` - Profile picture IPFS URL
- `banner_url` - Profile banner IPFS URL
- `social_links` - External social media links
- `reputation_score` - Community reputation metric
- `is_verified` - Verification status for notable accounts

## 📝 Posts Table

```sql
CREATE TABLE posts (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  content TEXT NOT NULL,
  media_urls TEXT, -- JSON array of IPFS URLs
  signature TEXT, -- Cryptographic signature
  signature_verified BOOLEAN DEFAULT FALSE,
  hashtags TEXT, -- JSON array of hashtags
  mentioned_users TEXT, -- JSON array of user IDs
  mentioned_tokens TEXT, -- JSON array of token IDs
  reaction_counts TEXT DEFAULT '{"like":0,"rocket":0,"diamond":0,"degen":0}',
  boost_count INTEGER DEFAULT 0,
  reply_count INTEGER DEFAULT 0,
  tip_count INTEGER DEFAULT 0,
  tip_total_lovelace INTEGER DEFAULT 0,
  view_count INTEGER DEFAULT 0,
  is_paid_content BOOLEAN DEFAULT FALSE,
  price_lovelace INTEGER DEFAULT 0,
  purchase_count INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_posts_hashtags ON posts(hashtags);
CREATE INDEX idx_posts_is_paid_content ON posts(is_paid_content);
CREATE INDEX idx_posts_signature_verified ON posts(signature_verified);

-- Full-text search index
CREATE VIRTUAL TABLE posts_fts USING fts5(
  content,
  hashtags,
  content='posts',
  content_rowid='rowid'
);
```

### Post Fields
- `content` - Post text content (max 280 chars)
- `media_urls` - JSON array of IPFS media URLs
- `signature` - CIP-8 cryptographic signature
- `hashtags` - Extracted hashtags for indexing
- `mentioned_users/tokens` - Referenced entities
- `reaction_counts` - JSON object with reaction counts
- `is_paid_content` - Whether content requires payment
- `price_lovelace` - Price in lovelace for paid content

## 💡 Reactions Table

```sql
CREATE TABLE reactions (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  post_id TEXT NOT NULL,
  reaction_type TEXT NOT NULL CHECK (reaction_type IN ('like', 'rocket', 'diamond', 'degen')),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  UNIQUE(user_id, post_id, reaction_type)
);

-- Indexes
CREATE INDEX idx_reactions_post_id ON reactions(post_id);
CREATE INDEX idx_reactions_user_id ON reactions(user_id);
CREATE INDEX idx_reactions_type ON reactions(reaction_type);
CREATE INDEX idx_reactions_created_at ON reactions(created_at);
```

## 🔄 Boosts Table

```sql
CREATE TABLE boosts (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  post_id TEXT NOT NULL,
  comment TEXT, -- Optional boost comment
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  UNIQUE(user_id, post_id)
);

-- Indexes
CREATE INDEX idx_boosts_post_id ON boosts(post_id);
CREATE INDEX idx_boosts_user_id ON boosts(user_id);
CREATE INDEX idx_boosts_created_at ON boosts(created_at DESC);
```

## 👥 Follows Table

```sql
CREATE TABLE follows (
  id TEXT PRIMARY KEY,
  follower_id TEXT NOT NULL,
  following_id TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (follower_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (following_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE(follower_id, following_id),
  CHECK(follower_id != following_id)
);

-- Indexes
CREATE INDEX idx_follows_follower_id ON follows(follower_id);
CREATE INDEX idx_follows_following_id ON follows(following_id);
CREATE INDEX idx_follows_created_at ON follows(created_at);
```

## 💬 Comments Table

```sql
CREATE TABLE comments (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  post_id TEXT NOT NULL,
  parent_comment_id TEXT, -- For threaded replies
  content TEXT NOT NULL,
  signature TEXT,
  signature_verified BOOLEAN DEFAULT FALSE,
  reaction_counts TEXT DEFAULT '{"like":0,"rocket":0,"diamond":0,"degen":0}',
  reply_count INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (parent_comment_id) REFERENCES comments(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_comments_parent_id ON comments(parent_comment_id);
CREATE INDEX idx_comments_created_at ON comments(created_at);
```

## 💰 Tips Table

```sql
CREATE TABLE tips (
  id TEXT PRIMARY KEY,
  from_user_id TEXT NOT NULL,
  to_post_id TEXT NOT NULL,
  to_user_id TEXT NOT NULL,
  amount_lovelace INTEGER NOT NULL,
  token_policy_id TEXT DEFAULT '',
  token_asset_name TEXT DEFAULT '',
  token_amount TEXT DEFAULT '0',
  transaction_hash TEXT UNIQUE NOT NULL,
  message TEXT,
  is_anonymous BOOLEAN DEFAULT FALSE,
  verified BOOLEAN DEFAULT FALSE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (from_user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (to_post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (to_user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_tips_to_post_id ON tips(to_post_id);
CREATE INDEX idx_tips_to_user_id ON tips(to_user_id);
CREATE INDEX idx_tips_from_user_id ON tips(from_user_id);
CREATE INDEX idx_tips_transaction_hash ON tips(transaction_hash);
CREATE INDEX idx_tips_created_at ON tips(created_at DESC);
```

## 🏷️ Topics Table

```sql
CREATE TABLE topics (
  id TEXT PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  display_name TEXT NOT NULL,
  description TEXT,
  post_count INTEGER DEFAULT 0,
  follower_count INTEGER DEFAULT 0,
  trending_score REAL DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_topics_name ON topics(name);
CREATE INDEX idx_topics_trending_score ON topics(trending_score DESC);
CREATE INDEX idx_topics_post_count ON topics(post_count DESC);
```

## 🪙 Tokens Table

```sql
CREATE TABLE tokens (
  id TEXT PRIMARY KEY,
  policy_id TEXT NOT NULL,
  asset_name TEXT NOT NULL,
  ticker TEXT,
  name TEXT NOT NULL,
  description TEXT,
  logo_url TEXT,
  website_url TEXT,
  twitter_handle TEXT,
  discord_url TEXT,
  telegram_url TEXT,
  current_price REAL DEFAULT 0,
  price_change_24h REAL DEFAULT 0,
  market_cap REAL DEFAULT 0,
  volume_24h REAL DEFAULT 0,
  holder_count INTEGER DEFAULT 0,
  mention_count INTEGER DEFAULT 0,
  follower_count INTEGER DEFAULT 0,
  total_supply TEXT DEFAULT '0',
  circulating_supply TEXT DEFAULT '0',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(policy_id, asset_name)
);

-- Indexes
CREATE INDEX idx_tokens_ticker ON tokens(ticker);
CREATE INDEX idx_tokens_name ON tokens(name);
CREATE INDEX idx_tokens_policy_id ON tokens(policy_id);
CREATE INDEX idx_tokens_mention_count ON tokens(mention_count DESC);
CREATE INDEX idx_tokens_market_cap ON tokens(market_cap DESC);
```

## 💎 Paid Posts Table

```sql
CREATE TABLE paid_posts (
  id TEXT PRIMARY KEY,
  post_id TEXT UNIQUE NOT NULL,
  price_lovelace INTEGER NOT NULL,
  preview_url TEXT, -- Blurred/watermarked preview
  full_url TEXT NOT NULL, -- Full resolution content
  purchase_count INTEGER DEFAULT 0,
  total_revenue_lovelace INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_paid_posts_post_id ON paid_posts(post_id);
CREATE INDEX idx_paid_posts_price ON paid_posts(price_lovelace);
```

## 🛒 Purchases Table

```sql
CREATE TABLE purchases (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  post_id TEXT NOT NULL,
  paid_post_id TEXT NOT NULL,
  amount_lovelace INTEGER NOT NULL,
  transaction_hash TEXT UNIQUE NOT NULL,
  verified BOOLEAN DEFAULT FALSE,
  access_granted BOOLEAN DEFAULT FALSE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
  FOREIGN KEY (paid_post_id) REFERENCES paid_posts(id) ON DELETE CASCADE,
  UNIQUE(user_id, post_id)
);

-- Indexes
CREATE INDEX idx_purchases_user_id ON purchases(user_id);
CREATE INDEX idx_purchases_post_id ON purchases(post_id);
CREATE INDEX idx_purchases_transaction_hash ON purchases(transaction_hash);
```

## 🔐 Sessions Table

```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  wallet_address TEXT NOT NULL,
  jwt_token_hash TEXT UNIQUE NOT NULL,
  expires_at DATETIME NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  last_used_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  user_agent TEXT,
  ip_address TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_wallet_address ON sessions(wallet_address);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);
CREATE INDEX idx_sessions_jwt_token_hash ON sessions(jwt_token_hash);
```

## 📁 Media Table

```sql
CREATE TABLE media (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  post_id TEXT,
  ipfs_cid TEXT UNIQUE NOT NULL,
  filename TEXT NOT NULL,
  mime_type TEXT NOT NULL,
  file_size INTEGER NOT NULL,
  width INTEGER,
  height INTEGER,
  duration REAL, -- For videos
  alt_text TEXT,
  is_paid_content BOOLEAN DEFAULT FALSE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_media_user_id ON media(user_id);
CREATE INDEX idx_media_post_id ON media(post_id);
CREATE INDEX idx_media_ipfs_cid ON media(ipfs_cid);
CREATE INDEX idx_media_mime_type ON media(mime_type);
```

## 🔔 Notifications Table

```sql
CREATE TABLE notifications (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  from_user_id TEXT,
  post_id TEXT,
  type TEXT NOT NULL CHECK (type IN ('reaction', 'boost', 'comment', 'follow', 'mention', 'tip', 'purchase')),
  title TEXT NOT NULL,
  message TEXT NOT NULL,
  data TEXT, -- JSON data for notification
  is_read BOOLEAN DEFAULT FALSE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (from_user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
CREATE INDEX idx_notifications_type ON notifications(type);
```

## 🔄 Database Triggers

### Update Counters
```sql
-- Update post count when posts are created/deleted
CREATE TRIGGER update_user_post_count_insert
AFTER INSERT ON posts
BEGIN
  UPDATE users SET 
    post_count = post_count + 1,
    updated_at = CURRENT_TIMESTAMP
  WHERE id = NEW.user_id;
END;

-- Update reaction counts on posts
CREATE TRIGGER update_post_reaction_counts
AFTER INSERT ON reactions
BEGIN
  UPDATE posts SET 
    reaction_counts = json_set(
      reaction_counts, 
      '$.' || NEW.reaction_type, 
      json_extract(reaction_counts, '$.' || NEW.reaction_type) + 1
    ),
    updated_at = CURRENT_TIMESTAMP
  WHERE id = NEW.post_id;
END;

-- Update follower counts
CREATE TRIGGER update_follow_counts
AFTER INSERT ON follows
BEGIN
  UPDATE users SET 
    follower_count = follower_count + 1,
    updated_at = CURRENT_TIMESTAMP
  WHERE id = NEW.following_id;
  
  UPDATE users SET 
    following_count = following_count + 1,
    updated_at = CURRENT_TIMESTAMP
  WHERE id = NEW.follower_id;
END;
```

## 📊 Performance Optimization

### Query Optimization
- **Composite indexes** for common query patterns
- **Partial indexes** for filtered queries
- **Full-text search** using SQLite FTS5
- **JSON indexes** for structured data queries

### Maintenance Tasks
```sql
-- Cleanup expired sessions
DELETE FROM sessions WHERE expires_at < CURRENT_TIMESTAMP;

-- Update trending scores for topics
UPDATE topics SET trending_score = (
  SELECT COUNT(*) * 1.0 / (julianday('now') - julianday(MAX(created_at)) + 1)
  FROM posts 
  WHERE hashtags LIKE '%' || topics.name || '%'
  AND created_at > datetime('now', '-7 days')
);

-- Vacuum database periodically
VACUUM;
ANALYZE;
```

Ready to integrate? Check out the [Integration Guide](integration-guide.md) for practical implementation examples.
