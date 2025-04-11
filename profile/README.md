# Welcome To Neo Pantheon

![neo-pantheon-logo](https://github.com/user-attachments/assets/d8818ee2-c3b0-485e-a2f8-2587cecf5b8e)

## Overview

Neo Pantheon is a decentralized protocol for creating, sharing, and monetizing AI agents through NFT ownership. The platform allows creators to build and deploy custom AI agents with unique personalities and capabilities, while enabling users to access these agents through a tiered subscription model, all powered by the NEOX token economy.

## Table of Contents

- [Architecture](#architecture)
- [Smart Contracts](#smart-contracts)
- [Key Features](#key-features)
- [Token Economics](#token-economics)
- [Getting Started](#getting-started)
  - [For Creators](#for-creators)
  - [For Users](#for-users)
- [Technical Implementation](#technical-implementation)
- [Fee Structure](#fee-structure)
- [Revenue Distribution](#revenue-distribution)
- [Marketplace](#marketplace)
- [Roadmap](#roadmap)
- [Conclusion](#conclusion)
- [License](#license)

## Architecture

Neo Pantheon consists of four core smart contracts:

1. **NEOX Token**: The platform's native ERC20 token used for all transactions
2. **AgentNFT**: ERC721 tokens representing ownership of AI agents
3. **AgentRegistry**: Central registry for agent creation, management and usage
4. **NPMarketplace**: Decentralized marketplace for buying and selling Agent NFTs

These contracts work together to create a complete ecosystem for AI agent creation, ownership, and monetization.

## Smart Contracts

### NEOX Token Contract

The NEOX token serves as the native currency of the Neo Pantheon ecosystem, with a total supply of 10 billion tokens.

```solidity
contract NEOX is ERC20, Ownable {
    uint256 public constant INITIAL_SUPPLY = 10_000_000_000 * 10**18; // 10 billion tokens
    
    constructor() ERC20("Neo Pantheon", "NEOX") Ownable(msg.sender) {
        _mint(msg.sender, INITIAL_SUPPLY);
    }
}
```

### AgentNFT Contract

The AgentNFT contract represents ownership of AI agents as NFTs. Each agent has exactly 12 NFT copies, with revenue being distributed equally among all NFT holders of a particular agent.

Key features:
- ERC721 token with enumeration support
- Access control for role-based permissions
- Tracks agent metadata and usage
- Manages revenue distribution and claims
- Custom token URI support for marketplaces and applications
- Security protection with reentrancy guards

#### Core Functions

1. **createAgent**: Creates a new NFT representing an agent
   - Called by the AgentRegistry when a new agent is created
   - Mints tokens directly to the creator's address
   - Records all metadata including agent type, name, and symbol
   
2. **distributeRevenue**: Distributes revenue to all NFT holders of an agent
   - Called automatically when fees are collected
   - Splits revenue equally among all NFT tokens of an agent
   - Updates unclaimedRevenue for each token
   - Handles remainder tokens when division isn't even
   
3. **claimRevenue**: Allows NFT owners to claim their accumulated revenue
   - Uses SafeERC20 for secure token transfers
   - Protected against reentrancy attacks
   - Transfers NEOX tokens directly to the owner's wallet
   - Resets unclaimedRevenue and updates totalRevenueClaimed
   - Can only be called by the token owner

4. **recordUsage**: Tracks usage statistics for agents
   - Updates totalUsageCount for analytics
   - Emits events for off-chain tracking

#### Security Features

- **Role-Based Access Control**: Only the registry contract can create agents and distribute revenue
- **ReentrancyGuard**: Prevents reentrancy attacks during revenue claiming
- **SafeERC20**: Uses OpenZeppelin's safe transfer methods for token operations
- **Maximum Token Limit**: Prevents gas-limit issues by capping tokens per agent

### AgentRegistry Contract

The AgentRegistry contract serves as the central hub for creating and using agents. It manages agent metadata, tier access, and fee collection. All interactions with agents flow through this registry, making it the core coordination layer of the Neo Pantheon ecosystem.

Key features:
- Agent creation with configurable properties
- Fixed creation of exactly 12 NFTs per agent
- Tiered access system (Basic, Advanced, Premium)
- Usage tracking and fee collection
- Revenue distribution to NFT holders
- Owner controls over fee parameters

#### Core Functions

1. **createAgent**: The gateway for creators to mint new AI agents
   - Requires 25,000 NEOX tokens
   - Configures agent name, symbol, type, personality, and model settings
   - Creates exactly 12 NFTs representing ownership
   - Automatically grants creator Premium tier access
   - Returns the newly created agentId for reference

2. **purchaseTier**: Allows users to buy access to agents at different levels
   - Basic Tier (100 NEOX): Entry-level features
   - Advanced Tier (500 NEOX): Enhanced capabilities
   - Premium Tier (2,000 NEOX): Full functionality
   - Automatically distributes 70% to NFT holders, 30% to platform

3. **useAgent**: The primary interface for interacting with agents
   - Requires appropriate tier access
   - Charges 10 NEOX per usage
   - Records usage statistics for analytics
   - Distributes revenue to NFT holders
   - Emits events for off-chain agent activation

4. **getAgentDetails**: Query function for retrieving complete agent metadata
   - Returns all public agent information including personality and model settings
   - Provides data needed for AI configuration

5. **getCreatorAgents**: Retrieves all agents created by a specific address
   - Useful for creator dashboards and portfolio management

6. **Administrative Functions**: Owner-only functions to adjust fees and revenue distribution
   - setFees: Update creation and usage fees
   - setFeeDistribution: Adjust the percentage split between creators and platform
   - withdrawFees: Allow platform owner to withdraw accumulated fees

#### Agent Structure

Each agent in the registry contains rich metadata:
```solidity
struct Agent {
    string name;         // Human-readable name
    string symbol;       // Short identifier (2-6 chars)
    uint8 agentType;     // Builder(0), Researcher(1), Socialite(2)
    address creator;     // Original creator address
    uint256 createdAt;   // Creation timestamp
    uint256 usageCount;  // Total usage counter
    string personality;  // Personality traits for AI model
    string modelConfig;  // Technical configuration
    bool active;         // Whether agent is currently available
    string customURI;    // Metadata URI for NFT marketplaces
}
```

### NPMarketplace Contract

The NPMarketplace contract enables decentralized trading of Agent NFTs using NEOX tokens. It provides a secure platform for creators to monetize their agents and for users to acquire ownership stakes in popular agents.

Key features:
- List Agent NFTs for sale at custom prices
- Purchase NFTs using NEOX tokens
- Integrated marketplace fee structure (2%)
- Secure NFT and token transfers
- Complete listing management

#### Core Functions

1. **listNFT**: Lists an NFT for sale
   - Verifies ownership of the NFT
   - Checks for proper approval
   - Creates active listing record
   - Emits NFTListed event

2. **purchaseNFT**: Allows users to buy listed NFTs
   - Non-reentrant function for security
   - Verifies buyer has sufficient NEOX tokens
   - Automatically splits payment between seller and marketplace fee
   - Transfers NFT to buyer
   - Emits NFTPurchased event

3. **delistNFT**: Allows sellers to remove NFTs from sale
   - Verifies caller is the original seller
   - Removes listing data
   - Emits NFTDelisted event

4. **getListing**: Query function to view current listing details
   - Returns complete listing information

#### Security Features

- **ReentrancyGuard**: Prevents reentrancy attacks during purchases
- **Ownership Verification**: Ensures only legitimate owners can list NFTs
- **ERC721Receiver**: Properly handles direct NFT transfers to the contract
- **Secure Token Transfers**: Verifies balance and allowance before transactions

## Key Features

### For Creators

1. **Agent Creation**: Create custom AI agents with unique personalities and capabilities
2. **NFT Monetization**: Earn ongoing revenue from agent usage through NFT ownership
3. **Fractional Ownership**: Create 12 NFTs for each agent to enable fractional ownership
4. **Automatic Revenue**: Automatically receive 70% of all fees related to your agents
5. **Marketplace Integration**: Sell Agent NFTs through the integrated marketplace

### For Users

1. **Tiered Access**: Purchase different tiers of access to agents based on usage needs
   - **Basic**: Entry-level access with limited features (100 NEOX)
   - **Advanced**: Enhanced access with more capabilities (500 NEOX)
   - **Premium**: Full access to all agent features (2,000 NEOX)
2. **Diverse Agent Types**: Access different types of specialized agents:
   - **Builder**: Focused on creation and development
   - **Researcher**: Specialized in information gathering and analysis
   - **Socialite**: Designed for communication and social interaction
3. **Ownership Opportunity**: Purchase agent NFTs to earn passive income from other users
4. **Secondary Market**: Buy and sell Agent NFTs through the decentralized marketplace

## Token Economics

The NEOX token is central to the Neo Pantheon ecosystem:

- **Total Supply**: 10 billion NEOX tokens
- **Use Cases**:
  - Creating new agents (25,000 NEOX)
  - Purchasing tier access (100-2,000 NEOX)
  - Using agents (10 NEOX per use)
  - Trading Agent NFTs on the marketplace
  - Earning revenue as an NFT holder

## Getting Started

### For Creators

1. **Create Your Agent**:
   ```javascript
   // Example parameters
   const name = "FinancialAdvisor";
   const symbol = "FADV";
   const agentType = 1; // Researcher
   const personality = "Professional, knowledgeable, and friendly financial advisor with expertise in investment strategies, retirement planning, and wealth management. Presents information in a clear, actionable manner with appropriate risk disclosures.";
   const modelConfig = "gpt-4-turbo, financial-focus, temperature=0.7, max_tokens=8192";
   const customURI = "https://api.neopantheon.io/metadata/financial-advisor";
   
   // Calculate creation fee (25,000 NEOX)
   const creationFee = ethers.utils.parseEther("25000");
   
   // Approve token spend
   await neoxToken.approve(agentRegistry.address, creationFee);
   
   // Create agent (automatically creates 12 NFTs)
   const tx = await agentRegistry.createAgent(
     name, symbol, agentType, personality, modelConfig, customURI
   );
   const receipt = await tx.wait();
   
   // Extract agentId from event logs
   const event = receipt.events.find(e => e.event === "AgentCreated");
   const agentId = event.args.agentId;
   console.log(`Created agent #${agentId} with 12 NFTs`);
   ```

2. **Check Your NFTs**:
   ```javascript
   // Get your agent's NFT tokens
   const tokens = await agentNFT.getAgentTokens(agentId);
   console.log(`You own ${tokens.length} NFTs for agent #${agentId}`);
   
   // Get details for each token
   for (const tokenId of tokens) {
     const info = await agentNFT.agentInfo(tokenId);
     console.log(`Token #${tokenId}:`);
     console.log(`- Name: ${info.name}`);
     console.log(`- Symbol: ${info.symbol}`);
     console.log(`- Type: ${info.agentType === 0 ? "Builder" : info.agentType === 1 ? "Researcher" : "Socialite"}`);
   }
   ```

3. **Monitor Revenue**:
   ```javascript
   // Get your agent's NFT tokens
   const tokens = await agentNFT.getAgentTokens(agentId);
   
   // Check unclaimed revenue for each token
   let totalUnclaimed = ethers.BigNumber.from(0);
   let totalClaimed = ethers.BigNumber.from(0);
   
   for (const tokenId of tokens) {
     const unclaimed = await agentNFT.unclaimedRevenue(tokenId);
     const claimed = await agentNFT.totalRevenueClaimed(tokenId);
     totalUnclaimed = totalUnclaimed.add(unclaimed);
     totalClaimed = totalClaimed.add(claimed);
     
     console.log(`Token #${tokenId}:`);
     console.log(`- Unclaimed: ${ethers.utils.formatEther(unclaimed)} NEOX`);
     console.log(`- Total Claimed: ${ethers.utils.formatEther(claimed)} NEOX`);
   }
   
   console.log(`Total Unclaimed: ${ethers.utils.formatEther(totalUnclaimed)} NEOX`);
   console.log(`Total Claimed: ${ethers.utils.formatEther(totalClaimed)} NEOX`);
   ```

4. **Claim Revenue**:
   ```javascript
   // Claim revenue for all your tokens
   for (const tokenId of tokens) {
     const unclaimed = await agentNFT.unclaimedRevenue(tokenId);
     
     if (unclaimed.gt(0)) {
       console.log(`Claiming ${ethers.utils.formatEther(unclaimed)} NEOX from token #${tokenId}`);
       const tx = await agentNFT.claimRevenue(tokenId);
       await tx.wait();
       console.log(`Successfully claimed revenue from token #${tokenId}`);
     }
   }
   ```

5. **List NFTs on the Marketplace**:
   ```javascript
   // Set NFT approval for marketplace
   await agentNFT.approve(marketplace.address, tokenId);
   
   // Set price (e.g., 15,000 NEOX)
   const price = ethers.utils.parseEther("15000");
   
   // List NFT for sale
   const tx = await marketplace.listNFT(agentNFT.address, tokenId, price);
   await tx.wait();
   
   console.log(`Successfully listed token #${tokenId} for ${ethers.utils.formatEther(price)} NEOX`);
   ```

### For Users

1. **Discover Available Agents**:
   ```javascript
   // Get the total number of agents
   const agentCount = await agentRegistry.agent.methods._agentIdCounter().call();
   
   // Query details for recent agents
   const recentAgents = [];
   const maxDisplay = 10;
   const startIdx = Math.max(0, agentCount - maxDisplay);
   
   for (let i = startIdx; i < agentCount; i++) {
     try {
       const details = await agentRegistry.getAgentDetails(i);
       recentAgents.push({
         id: i,
         name: details.name,
         symbol: details.symbol,
         type: details.agentType === 0 ? "Builder" : 
               details.agentType === 1 ? "Researcher" : "Socialite",
         creator: details.creator,
         usageCount: details.usageCount.toString(),
         active: details.active
       });
     } catch (error) {
       console.log(`Error fetching agent #${i}: ${error.message}`);
     }
   }
   
   console.table(recentAgents);
   ```

2. **Purchase Tier Access**:
   ```javascript
   // Get tier costs
   const basicCost = ethers.utils.parseEther("100");    // 100 NEOX
   const advancedCost = ethers.utils.parseEther("500"); // 500 NEOX
   const premiumCost = ethers.utils.parseEther("2000"); // 2000 NEOX
   
   // Select tier (1=Basic, 2=Advanced, 3=Premium)
   const selectedTier = 2; // Advanced
   const tierCost = selectedTier === 1 ? basicCost : 
                   selectedTier === 2 ? advancedCost : premiumCost;
   
   // Approve token spend
   await neoxToken.approve(agentRegistry.address, tierCost);
   
   // Purchase tier access
   const tx = await agentRegistry.purchaseTier(agentId, selectedTier);
   await tx.wait();
   
   console.log(`Successfully purchased ${tierName} tier access to agent #${agentId}`);
   ```

3. **Use an Agent**:
   ```javascript
   // Calculate usage fee (10 NEOX)
   const usageFee = ethers.utils.parseEther("10");
   
   // Approve token spend
   await neoxToken.approve(agentRegistry.address, usageFee);
   
   // Use the agent
   const tx = await agentRegistry.useAgent(agentId);
   await tx.wait();
   
   console.log(`Successfully used agent #${agentId}`);
   
   // The application will now connect to the agent for the user session
   // Example connection to Neo Pantheon API:
   const apiResponse = await fetch(`https://api.neopantheon.io/v1/connect`, {
     method: 'POST',
     headers: {
       'Content-Type': 'application/json',
     },
     body: JSON.stringify({
       agentId,
       userAddress,
       txHash: tx.hash
     })
   });
   
   const { sessionId } = await apiResponse.json();
   console.log(`Connected to agent with session ID: ${sessionId}`);
   ```

4. **Buy an Agent NFT from the Marketplace**:
   ```javascript
   // Get marketplace listings for a specific agent
   const agentId = 42;
   const tokens = await agentNFT.getAgentTokens(agentId);
   
   // Check for active listings
   for (const tokenId of tokens) {
     try {
       const listing = await marketplace.getListing(agentNFT.address, tokenId);
       
       if (listing.isActive) {
         console.log(`Token #${tokenId} is listed for ${ethers.utils.formatEther(listing.price)} NEOX`);
         console.log(`Seller: ${listing.seller}`);
       }
     } catch (error) {
       console.log(`Error checking listing for token #${tokenId}: ${error.message}`);
     }
   }
   
   // Purchase a listed NFT
   const tokenToBuy = 123; // Token ID you want to buy
   const listing = await marketplace.getListing(agentNFT.address, tokenToBuy);
   
   // Approve NEOX tokens for the purchase
   await neoxToken.approve(marketplace.address, listing.price);
   
   // Buy the NFT
   const tx = await marketplace.purchaseNFT(agentNFT.address, tokenToBuy);
   await tx.wait();
   
   console.log(`Successfully purchased token #${tokenToBuy} for ${ethers.utils.formatEther(listing.price)} NEOX`);
   ```

## Technical Implementation

### Agent Creation Process

1. Creator pays 25,000 NEOX to create an agent
2. Agent metadata is stored in the registry
3. Exactly 12 NFTs are minted to the creator
4. Creator automatically receives Premium tier access

### Revenue Flow

1. Users pay NEOX for tier access and agent usage
2. 70% of fees are distributed equally among all NFT holders of that agent
3. 30% of fees go to the platform
4. NFT holders can claim their revenue at any time

### Contract Interaction Diagram

```
┌───────────────┐            ┌───────────────┐             ┌───────────────┐
│               │            │               │             │               │
│  NEOX Token   │◄──────────►│ AgentRegistry │◄───────────►│   AgentNFT    │
│  (ERC20)      │            │ (Management)  │             │   (ERC721)    │
│               │            │               │             │               │
└───────────────┘            └───────────────┘             └───────────────┘
        ▲                            ▲                             ▲
        │                            │                             │
        │                            │                             │
        ▼                            ▼                             ▼
┌───────────────┐            ┌───────────────┐             ┌───────────────┐
│               │            │               │             │               │
│    Creator    │◄──────────►│     User      │◄───────────►│  NFT Holder   │
│               │            │               │             │               │
└───────────────┘            └───────────────┘             └───────────────┘
                                                                  ▲
                                                                  │
                                                                  │
                                                                  ▼
                                                          ┌───────────────┐
                                                          │               │
                                                          │ NPMarketplace │
                                                          │               │
                                                          └───────────────┘
```

### Agent Types

The protocol supports three types of specialized agents:

1. **Builder (Type 0)**:
   - Focused on creation, development, and technical tasks
   - Examples: Code generation, app development, technical problem-solving

2. **Researcher (Type 1)**:
   - Specialized in information gathering, analysis, and knowledge work
   - Examples: Data analysis, market research, content curation

3. **Socialite (Type 2)**:
   - Designed for communication, engagement, and social interaction
   - Examples: Customer service, content creation, community management

## Fee Structure

| Action | Fee | Distribution |
|--------|-----|--------------|
| Agent Creation | 25,000 NEOX | 100% to platform |
| Basic Tier | 100 NEOX | 70% to creators, 30% to platform |
| Advanced Tier | 500 NEOX | 70% to creators, 30% to platform |
| Premium Tier | 2,000 NEOX | 70% to creators, 30% to platform |
| Agent Usage | 10 NEOX per use | 70% to creators, 30% to platform |
| NFT Marketplace Sale | Sale Price | 98% to seller, 2% to platform |

## Revenue Distribution

When fees are collected from tier purchases and agent usage:

1. 70% is distributed equally among all NFT holders of that agent
2. 30% goes to the platform treasury
3. NFT holders must manually claim their revenue
4. Remainder tokens from uneven divisions go to the first token

For example, if an agent has 12 NFTs and generates 1,000 NEOX in usage fees:
- 700 NEOX (70%) is distributed to NFT holders (58.33 NEOX each, with 0.04 NEOX remainder to first token)
- 300 NEOX (30%) goes to the platform

## Marketplace

The integrated NFT marketplace allows for secure trading of Agent NFTs:

1. **Listing Process**:
   - Seller approves the marketplace contract to handle their NFT
   - Seller lists the NFT with a desired price in NEOX tokens
   - Listing is published on-chain and available to all users

2. **Purchase Process**:
   - Buyer approves NEOX token spending
   - Buyer purchases the NFT through marketplace contract
   - 98% of payment goes to seller, 2% to marketplace fee
   - NFT is automatically transferred to buyer

3. **Benefits**:
   - Secure escrow mechanism
   - Transparent fee structure
   - Automated revenue distribution
   - Direct integration with Agent usage system

## Roadmap

- **Q2 2025**: Launch of AI Agent Marketplace
- **Q3 2025**: Introduction of Agent Collaboration Framework
- **Q4 2025**: DAO Governance for Protocol Parameters
- **Q1 2026**: Cross-Chain Integration

## Conclusion

Neo Pantheon represents a new paradigm for AI agent ownership and monetization. By tokenizing AI agents as NFTs and creating a sustainable economic model, the protocol enables creators to earn ongoing revenue while users gain access to a diverse ecosystem of specialized AI capabilities.

## License

MIT License
