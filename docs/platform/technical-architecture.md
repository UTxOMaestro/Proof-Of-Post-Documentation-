# Technical Architecture

Proof of Post is built on a modern, scalable architecture that seamlessly integrates traditional web technologies with Cardano blockchain capabilities. Here's how the platform is constructed:

## Core Technology Stack

### Frontend Architecture
- **Next.js 14** with App Router for modern React development
- **TypeScript** throughout for type safety and developer experience
- **Tailwind CSS** for responsive, utility-first styling
- **Framer Motion** for smooth, performant animations
- **Lucide React** for consistent iconography

### State Management & Data
- **Zustand** with persistence for client-side state
- **SQLite** with Drizzle ORM for efficient data management
- **JWT Authentication** with wallet signature verification
- **Client-side caching** with TTL for optimal performance

### Blockchain Integration
- **Lucid Cardano** for blockchain interactions
- **CIP-30 Wallet Support** for multi-wallet compatibility
- **Blockfrost API** for Cardano network data
- **CIP-8 Message Signing** for cryptographic verification

