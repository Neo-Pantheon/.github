# Neo Pantheon Protocol

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

## Architecture

Neo Pantheon consists of three core smart contracts:

1. **NEOX Token**: The platform's native ERC20 token used for all transactions
2. **AgentNFT**: ERC721 tokens representing ownership of AI agents
3. **AgentRegistry**: Central registry for agent creation, management and usage

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

The AgentNFT contract represents ownership of AI agents as NFTs. Each agent can have multiple NFT copies (up to 12), with revenue being distributed equally among all NFT holders of a particular agent.

Key features:
- ERC721 token with enumeration support
- Tracks agent metadata and usage
- Manages revenue distribution and claims
- Custom token URI support for marketplaces and applications

#### Core Functions

1. **createAgent**: Creates a new NFT representing an agent
   - Called by the AgentRegistry when a new agent is created
   - Mints tokens directly to the creator's address
   - Records all metadata including agent type, name, and symbol
   
2. **distributeRevenue**: Distributes revenue to all NFT holders of an agent
   - Called automatically when fees are collected
   - Splits revenue equally among all NFT tokens of an agent
   - Updates unclaimedRevenue for each token
   
3. **claimRevenue**: Allows NFT owners to claim their accumulated revenue
   - Transfers NEOX tokens directly to the owner's wallet
   - Resets unclaimedRevenue and updates totalRevenueClaimed
   - Can only be called by the token owner

4. **recordUsage**: Tracks usage statistics for agents
   - Updates totalUsageCount for analytics
   - Emits events for off-chain tracking

#### Usage Tracking and Analytics

The AgentNFT contract provides rich data for agent owners:
- Total usage count per agent
- Total revenue claimed for each token
- Unclaimed revenue available per token
- Historical usage through event logs

### AgentRegistry Contract

The AgentRegistry contract serves as the central hub for creating and using agents. It manages agent metadata, tier access, and fee collection. All interactions with agents flow through this registry, making it the core coordination layer of the Neo Pantheon ecosystem.

Key features:
- Agent creation with configurable properties
- Tiered access system (Basic, Advanced, Premium)
- Usage tracking and fee collection
- Revenue distribution to NFT holders

#### Core Functions

1. **createAgent**: The gateway for creators to mint new AI agents
   - Requires 25,000 NEOX tokens
   - Configures agent name, symbol, type, personality, and model settings
   - Creates 1-12 NFTs representing ownership
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

4. **getAgentDetails/getAgentConfig**: Query functions for retrieving agent metadata
   - Returns all public agent information
   - Provides personality and model settings for AI configuration

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

#### Creator Dashboard Features

The registry provides special functions for creators:
- **getCreatorAgents**: Lists all agents created by an address
- Auto-tracking of usage statistics
- Access to detailed configuration data
- Management of agent availability

## Key Features

### For Creators

1. **Agent Creation**: Create custom AI agents with unique personalities and capabilities
2. **NFT Monetization**: Earn ongoing revenue from agent usage through NFT ownership
3. **Fractional Ownership**: Create multiple NFTs (up to 12) for each agent to enable fractional ownership
4. **Automatic Revenue**: Automatically receive 70% of all fees related to your agents

### For Users

1. **Tiered Access**: Purchase different tiers of access to agents based on usage needs
   - **Basic**: Entry-level access with limited features
   - **Advanced**: Enhanced access with more capabilities
   - **Premium**: Full access to all agent features and capabilities
2. **Diverse Agent Types**: Access different types of specialized agents:
   - **Builder**: Focused on creation and development
   - **Researcher**: Specialized in information gathering and analysis
   - **Socialite**: Designed for communication and social interaction
3. **Ownership Opportunity**: Purchase agent NFTs to earn passive income from other users

## Token Economics

The NEOX token is central to the Neo Pantheon ecosystem:

- **Total Supply**: 10 billion NEOX tokens
- **Use Cases**:
  - Creating new agents (25,000 NEOX)
  - Purchasing tier access (100-2,000 NEOX)
  - Using agents (10 NEOX per use)
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
   const nftCount = 5; // Create 5 NFTs
   const customURI = "https://api.neopantheon.io/metadata/financial-advisor";
   
   // Calculate creation fee (25,000 NEOX)
   const creationFee = ethers.utils.parseEther("25000");
   
   // Approve token spend
   await neoxToken.approve(agentRegistry.address, creationFee);
   
   // Create agent
   const tx = await agentRegistry.createAgent(
     name, symbol, agentType, personality, modelConfig, nftCount, customURI
   );
   const receipt = await tx.wait();
   
   // Extract agentId from event logs
   const event = receipt.events.find(e => e.event === "AgentCreated");
   const agentId = event.args.agentId;
   console.log(`Created agent #${agentId} with ${nftCount} NFTs`);
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

5. **Track Agent Usage**:
   ```javascript
   // Get agent usage statistics
   const usageCount = await agentNFT.totalUsageCount(agentId);
   console.log(`Agent #${agentId} has been used ${usageCount} times`);
   
   // Track usage events
   const filter = agentNFT.filters.AgentUsed(agentId, null);
   const events = await agentNFT.queryFilter(filter, -10000); // Last 10000 blocks
   
   console.log(`Recent usage events: ${events.length}`);
   events.forEach((event, i) => {
     console.log(`${i+1}. Used by: ${event.args.user} at block ${event.blockNumber}`);
   });
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

2. **Check Current Tier Access**:
   ```javascript
   // Get your current tier for a specific agent
   const userAddress = "0x..."; // Your address
   const agentId = 42; // Agent ID
   
   const tier = await agentRegistry.userTiers(userAddress, agentId);
   const tierName = tier == 0 ? "None" : 
                   tier == 1 ? "Basic" :
                   tier == 2 ? "Advanced" : "Premium";
   
   console.log(`Your current access tier for agent #${agentId}: ${tierName}`);
   ```

3. **Purchase Tier Access**:
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

4. **Use an Agent**:
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

5. **Buy an Agent NFT** (through secondary market):
   ```javascript
   // This example assumes a marketplace integration

   // 1. Find available NFTs for a specific agent
   const agentId = 42;
   const tokens = await agentNFT.getAgentTokens(agentId);
   
   console.log(`Agent #${agentId} has ${tokens.length} NFTs in circulation`);
   
   // 2. Check ownership and marketplace listings
   // (integration with marketplaces like OpenSea, Rarible, etc.)
   
   // 3. After purchase, check revenue for your new NFT
   const myTokenId = 123; // ID of the token you purchased
   const unclaimed = await agentNFT.unclaimedRevenue(myTokenId);
   console.log(`Your new NFT has ${ethers.utils.formatEther(unclaimed)} NEOX in unclaimed revenue`);
   
   // 4. Claim available revenue
   if (unclaimed.gt(0)) {
     const tx = await agentNFT.claimRevenue(myTokenId);
     await tx.wait();
     console.log(`Successfully claimed ${ethers.utils.formatEther(unclaimed)} NEOX`);
   }
   ```

## Technical Implementation

### Agent Creation Process

1. Creator pays 25,000 NEOX to create an agent
2. Agent metadata is stored in the registry
3. NFTs (1-12) are minted to the creator
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
│  NEOX Token   │◄──────────►│ AgentRegistry │◄────────────►│   AgentNFT    │
│  (ERC20)      │            │ (Management)  │             │   (ERC721)     │
│               │            │               │             │               │
└───────────────┘            └───────────────┘             └───────────────┘
        ▲                            ▲                             ▲
        │                            │                             │
        │                            │                             │
        ▼                            ▼                             ▼
┌───────────────┐            ┌───────────────┐             ┌───────────────┐
│               │            │               │             │               │
│    Creator    │◄──────────►│     User      │◄────────────►│  NFT Holder   │
│               │            │               │             │               │
└───────────────┘            └───────────────┘             └───────────────┘
```

### Example Workflow: Agent Creation and Usage

1. **Creation Flow**:
   ```
   Creator -> approves NEOX spend -> AgentRegistry.createAgent() ->
     -> AgentRegistry creates agent record
     -> AgentRegistry calls AgentNFT.createAgent() multiple times
     -> AgentNFT mints tokens to Creator
     -> AgentRegistry grants Creator premium tier
   ```

2. **User Access Flow**:
   ```
   User -> approves NEOX spend -> AgentRegistry.purchaseTier() ->
     -> AgentRegistry transfers NEOX tokens
     -> AgentRegistry calls AgentNFT.distributeRevenue()
     -> AgentNFT updates unclaimedRevenue for all tokens
     -> AgentRegistry grants User tier access
   ```

3. **Usage Flow**:
   ```
   User -> approves NEOX spend -> AgentRegistry.useAgent() ->
     -> AgentRegistry verifies tier access
     -> AgentRegistry transfers NEOX tokens
     -> AgentRegistry calls AgentNFT.recordUsage()
     -> AgentRegistry calls AgentNFT.distributeRevenue()
     -> AgentNFT updates unclaimedRevenue for all tokens
     -> Off-chain systems activate AI agent for user
   ```

4. **Revenue Claim Flow**:
   ```
   NFT Holder -> AgentNFT.claimRevenue() ->
     -> AgentNFT verifies token ownership
     -> AgentNFT transfers NEOX tokens to holder
     -> AgentNFT updates tracking metrics
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

### AgentNFT Distribution Mechanism

The AgentNFT contract implements a sophisticated revenue distribution mechanism that ensures fair and transparent revenue sharing:

```solidity
function distributeRevenue(uint256 agentId, uint256 amount) external {
    require(msg.sender == registry || msg.sender == owner(), "Unauthorized");
    require(amount > 0, "Zero amount");
    
    uint256[] storage tokens = agentTokens[agentId];
    require(tokens.length > 0, "No tokens for agent");
    
    uint256 amountPerToken = amount / tokens.length;
    require(amountPerToken > 0, "Amount too small");
    
    for (uint256 i = 0; i < tokens.length; i++) {
        unclaimedRevenue[tokens[i]] += amountPerToken;
    }
    
    emit RevenueDistributed(agentId, amount);
}
```

This function is called automatically when:
1. A user purchases tier access to an agent
2. A user calls the `useAgent` function to interact with an agent

The total revenue is split equally among all NFT tokens for that agent, ensuring proportional distribution based on ownership share. For example, if an agent has 5 NFTs and generates 700 NEOX in creator revenue, each NFT would receive 140 NEOX.

### Revenue Claiming Process

NFT owners must manually claim their revenue using the `claimRevenue` function:

```solidity
function claimRevenue(uint256 tokenId) external {
    require(ownerOf(tokenId) == msg.sender, "Not token owner");
    require(unclaimedRevenue[tokenId] > 0, "No revenue to claim");
    
    uint256 amount = unclaimedRevenue[tokenId];
    unclaimedRevenue[tokenId] = 0;
    totalRevenueClaimed[tokenId] += amount;
    
    neoxToken.transfer(msg.sender, amount);
    
    emit RevenueClaimed(tokenId, msg.sender, amount);
}
```

This function:
1. Verifies the caller is the legitimate NFT owner
2. Checks if there is unclaimed revenue available
3. Updates internal accounting records
4. Transfers NEOX tokens directly to the owner's wallet
5. Emits an event for transparency and tracking

## Revenue Distribution

When fees are collected from tier purchases and agent usage:

1. 70% is distributed equally among all NFT holders of that agent
2. 30% goes to the platform treasury
3. NFT holders must manually claim their revenue

For example, if an agent has 5 NFTs and generates 1,000 NEOX in usage fees:
- 700 NEOX (70%) is distributed to NFT holders (140 NEOX each)
- 300 NEOX (30%) goes to the platform

## Roadmap

- **Q2 2025**: Launch of AI Agent Marketplace
- **Q3 2025**: Introduction of Agent Collaboration Framework
- **Q4 2025**: DAO Governance for Protocol Parameters
- **Q1 2026**: Cross-Chain Integration

## Conclusion

Neo Pantheon represents a new paradigm for AI agent ownership and monetization. By tokenizing AI agents as NFTs and creating a sustainable economic model, the protocol enables creators to earn ongoing revenue while users gain access to a diverse ecosystem of specialized AI capabilities.

## License

MIT License
