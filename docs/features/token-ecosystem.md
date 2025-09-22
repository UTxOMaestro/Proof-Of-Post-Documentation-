# Token Ecosystem

Proof of Post seamlessly integrates Cardano's rich token ecosystem, making it easy to discover, discuss, and interact with tokens directly within the social media experience.

---
**Disclaimer**: All features and functionalities described in this documentation are subject to change during development. The final implementation may differ from what is described here.
---

## Token Discovery

### Comprehensive Token Database
The platform maintains an extensive database of Cardano tokens with:
- **Token Metadata**: Names, tickers, descriptions, and images
- **Policy Information**: Policy IDs and asset names
- **Holder Statistics**: Number of holders and distribution
- **Price Data**: Real-time pricing via DexHunter API
- **Trading Volume**: 24h volume and market activity
- **Social Metrics**: Mention frequency and community engagement

### Search & Discovery
- **Token Search**: Find tokens by ticker, name, or policy ID
- **Trending Tokens**: Discover popular tokens based on mentions
- **New Listings**: Recently added tokens to the ecosystem
- **Category Filtering**: Browse by token type (utility, meme, governance)
- **Watchlist**: Follow your favorite tokens for updates

## Token Profiles

### Comprehensive Token Pages
Each token gets a dedicated profile page featuring:

#### Basic Information
- **Token Name & Ticker**: Official project identification
- **Token Logo**: High-resolution project branding
- **Description**: Project overview and use case
- **Total Supply**: Maximum token supply information
- **Circulating Supply**: Currently available tokens

#### Market Data
- **Current Price**: Real-time price in ADA and USD
- **24h Change**: Price movement percentage
- **Market Cap**: Total market valuation
- **Trading Volume**: 24-hour trading activity
- **Price Charts**: Historical price performance

#### Social Integration
- **Mention Feed**: All posts mentioning the token
- **Community Stats**: Follower count and engagement
- **Official Links**: Website, Twitter, Discord, Telegram
- **Holder Analysis**: Top holders and distribution charts

## Token Mentions

### Automatic Token Linking
When users mention tokens in posts using the `$TOKEN` format:
- **Automatic Detection**: Platform recognizes token mentions
- **Clickable Links**: Mentions become links to token profiles
- **Real-time Validation**: Verify token exists in the database
- **Price Display**: Show current price on hover
- **Trending Impact**: Mentions contribute to trending calculations

### Mention Analytics
- **Mention Frequency**: Track how often tokens are discussed
- **Sentiment Analysis**: Positive/negative mention tracking
- **Influential Mentions**: Posts from verified or high-reputation users
- **Mention History**: Historical mention trends and patterns

## Price Integration

### Real-time Price Data
- **DexHunter API Integration**: Reliable price feeds from Cardano DEXs
- **Multiple Price Sources**: Aggregated data for accuracy
- **Price Alerts**: Notify users of significant price movements
- **Historical Charts**: Price history with various timeframes
- **Volume Analysis**: Trading volume trends and patterns

### Price Display Features
- **Inline Prices**: Show current price next to token mentions
- **Price Change Indicators**: Color-coded price movement
- **Portfolio Tracking**: Track value of tokens you hold
- **Watchlist Prices**: Monitor prices of followed tokens
- **Price Notifications**: Alerts for significant price changes

## Token Interactions

### Tipping System
- **Multi-Token Tipping**: Tip creators with any supported token
- **Custom Amounts**: Set specific tip amounts
- **Tip History**: Track tips sent and received
- **Popular Tip Tokens**: See which tokens are commonly used for tips
- **Tip Leaderboards**: Top tippers and recipients

### Trading Integration
- **Swap Widget**: Direct token trading within the platform
- **DEX Integration**: Connect to popular Cardano DEXs
- **Trading Pairs**: Discover available trading pairs
- **Liquidity Information**: Pool depth and trading fees
- **Trade History**: Track your trading activity

## Token Communities

### Community Features
- **Token-Specific Feeds**: Dedicated feeds for each token's mentions
- **Community Moderators**: Verified community representatives
- **Official Announcements**: Highlighted posts from project teams
- **Community Guidelines**: Token-specific posting rules
- **Event Calendar**: Upcoming project milestones and events

### Engagement Tools
- **Token Holders**: Verify token ownership for exclusive content
- **Holder Verification**: Prove token holdings for credibility
- **Community Polls**: Token holder voting on proposals
- **AMA Sessions**: Ask Me Anything with project founders
- **Community Challenges**: Engagement campaigns and contests

## Watchlist & Portfolio

### Personal Watchlist
- **Token Following**: Follow tokens for updates and news
- **Custom Lists**: Create themed token collections
- **Portfolio Tracking**: Monitor your token holdings
- **Performance Metrics**: Track gains/losses over time
- **Rebalancing Alerts**: Notifications for portfolio changes

### Portfolio Features
- **Automatic Detection**: Detect tokens in connected wallets
- **Portfolio Value**: Total portfolio worth in ADA/USD
- **Asset Allocation**: Pie charts showing token distribution
- **Performance History**: Track portfolio performance over time
- **Tax Reporting**: Export data for tax calculations

## Token Alerts

### Price Alerts
- **Price Targets**: Notify when tokens reach specific prices
- **Percentage Changes**: Alerts for significant price movements
- **Volume Spikes**: Notifications for unusual trading activity
- **New Listings**: Alerts when tokens are newly listed
- **Delisting Warnings**: Notifications for tokens being removed

### Social Alerts
- **Mention Spikes**: When tokens are suddenly trending
- **Influencer Posts**: When notable users mention specific tokens
- **Community Milestones**: Holder count or volume achievements
- **Project Updates**: Official announcements from project teams
- **Regulatory News**: Important regulatory developments

## Developer Tools

### Token API
```typescript
// Get token information
const tokenInfo = await fetch('/api/tokens/search?q=ADA');

// Get token price
const priceData = await fetch('/api/tokens/price/ada');

// Get token mentions
const mentions = await fetch('/api/tokens/mentions/ada?limit=50');

// Track token in watchlist
await fetch('/api/tokens/watch', {
  method: 'POST',
  body: JSON.stringify({ tokenId: 'ada' })
});
```

### Integration Features
- **Token Widgets**: Embeddable price and info widgets
- **Price Feeds**: Real-time WebSocket price updates
- **Historical Data**: Access to historical price and volume data
- **Metadata API**: Token information and imagery
- **Social Metrics**: Mention counts and sentiment data

## Visual Integration

### Token Branding
- **Token Logos**: High-quality logos throughout the platform
- **Color Schemes**: Token-specific color theming
- **Brand Guidelines**: Consistent token presentation
- **Icon Sets**: Token icons for various contexts
- **Badge Systems**: Achievement badges for token communities

### UI Components
- **Token Cards**: Standardized token information display
- **Price Tickers**: Scrolling price displays
- **Portfolio Widgets**: Personal holdings visualization
- **Trading Charts**: Interactive price charts
- **Trend Indicators**: Visual trend and momentum indicators

---

The token ecosystem integration makes Proof of Post the premier destination for Cardano token communities, combining social media with comprehensive token information and trading capabilities.

Next: Learn about [Content Monetization](content-monetization.md) to discover how creators can earn directly from their content.
