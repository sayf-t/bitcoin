# Bitcoin Core Fundamentals: Supplementary Projects

## 🎯 Learning Objectives
These projects will help you:
- Visualize Bitcoin's core concepts
- Understand network growth and mining
- See how blocks and transactions connect
- Grasp economic principles through simulation

## 🚀 Project 1: Bitcoin Network Visualizer

A React/TypeScript application that simulates and visualizes Bitcoin network growth.

### Project Structure
```bash
bitcoin-visualizer/
├── src/
│   ├── components/
│   │   ├── NetworkGraph/
│   │   ├── BlockchainView/
│   │   └── Statistics/
│   ├── services/
│   │   ├── simulation.service.ts
│   │   └── metrics.service.ts
│   └── utils/
│       └── visualization.utils.ts
├── public/
└── package.json
```

### Core Simulation Code
```typescript
// src/services/simulation.service.ts
interface Block {
    height: number;
    hash: string;
    prevHash: string;
    timestamp: number;
    difficulty: number;
    reward: number;
}

export class BitcoinSimulation {
    private blocks: Block[] = [];
    private difficulty = 1;
    private blockReward = 50;
    private halveningInterval = 210000;
    
    constructor() {
        // Create genesis block
        this.blocks.push({
            height: 0,
            hash: '000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f',
            prevHash: '0000000000000000000000000000000000000000000000000000000000000000',
            timestamp: 1231006505,
            difficulty: 1,
            reward: 50
        });
    }
    
    simulateBlocks(numberOfBlocks: number) {
        for (let i = 1; i <= numberOfBlocks; i++) {
            const prevBlock = this.blocks[i - 1];
            
            // Adjust difficulty every 2016 blocks
            if (i % 2016 === 0) {
                this.adjustDifficulty();
            }
            
            // Handle halvening
            if (i % this.halveningInterval === 0) {
                this.blockReward /= 2;
            }
            
            const newBlock = {
                height: i,
                hash: this.calculateHash(prevBlock.hash, i),
                prevHash: prevBlock.hash,
                timestamp: prevBlock.timestamp + 600, // Average 10 minutes
                difficulty: this.difficulty,
                reward: this.blockReward
            };
            
            this.blocks.push(newBlock);
        }
        
        return this.blocks;
    }
    
    private calculateHash(prevHash: string, height: number): string {
        // Simplified hash calculation for simulation
        return crypto
            .createHash('sha256')
            .update(prevHash + height.toString())
            .digest('hex');
    }
    
    private adjustDifficulty() {
        // Simplified difficulty adjustment
        const timeSpan = this.blocks[this.blocks.length - 1].timestamp - 
                        this.blocks[this.blocks.length - 2016].timestamp;
        const targetTimeSpan = 2016 * 600; // 2 weeks in seconds
        
        this.difficulty *= targetTimeSpan / timeSpan;
    }
}
```

### Visualization Component
```typescript
// src/components/BlockchainView/BlockchainGraph.tsx
import React, { useEffect, useRef } from 'react';
import * as d3 from 'd3';

interface BlockchainGraphProps {
    blocks: Block[];
}

export const BlockchainGraph: React.FC<BlockchainGraphProps> = ({ blocks }) => {
    const svgRef = useRef<SVGSVGElement>(null);
    
    useEffect(() => {
        if (!svgRef.current || !blocks.length) return;
        
        const svg = d3.select(svgRef.current);
        const width = 1000;
        const height = 600;
        
        // Clear previous
        svg.selectAll("*").remove();
        
        // Create scales
        const xScale = d3.scaleLinear()
            .domain([0, blocks.length])
            .range([50, width - 50]);
            
        const yScale = d3.scaleLinear()
            .domain([0, d3.max(blocks, d => d.difficulty)!])
            .range([height - 50, 50]);
            
        // Draw difficulty line
        const line = d3.line<Block>()
            .x(d => xScale(d.height))
            .y(d => yScale(d.difficulty));
            
        svg.append("path")
            .datum(blocks)
            .attr("fill", "none")
            .attr("stroke", "steelblue")
            .attr("stroke-width", 1.5)
            .attr("d", line);
            
        // Add block rewards
        svg.selectAll("circle")
            .data(blocks.filter(b => b.height % this.halveningInterval === 0))
            .enter()
            .append("circle")
            .attr("cx", d => xScale(d.height))
            .attr("cy", d => yScale(d.difficulty))
            .attr("r", 5)
            .attr("fill", "red")
            .append("title")
            .text(d => `Block ${d.height}\nReward: ${d.reward} BTC`);
    }, [blocks]);
    
    return (
        <div className="blockchain-graph">
            <svg ref={svgRef} width="1000" height="600" />
        </div>
    );
};
```

### Statistics Component
```typescript
// src/components/Statistics/NetworkStats.tsx
export const NetworkStats: React.FC<{ blocks: Block[] }> = ({ blocks }) => {
    const totalBitcoin = blocks.reduce((sum, block) => sum + block.reward, 0);
    const latestDifficulty = blocks[blocks.length - 1].difficulty;
    
    return (
        <div className="network-stats">
            <h3>Network Statistics</h3>
            <div className="stat-grid">
                <div className="stat-item">
                    <label>Total Blocks</label>
                    <value>{blocks.length}</value>
                </div>
                <div className="stat-item">
                    <label>Bitcoin Mined</label>
                    <value>{totalBitcoin.toFixed(2)} BTC</value>
                </div>
                <div className="stat-item">
                    <label>Current Difficulty</label>
                    <value>{latestDifficulty.toExponential(2)}</value>
                </div>
                <div className="stat-item">
                    <label>Current Reward</label>
                    <value>{blocks[blocks.length - 1].reward} BTC</value>
                </div>
            </div>
        </div>
    );
};
```

## 🚀 Project 2: Transaction Flow Simulator

A Node.js application that simulates transaction propagation through the network.

```typescript
// transaction-simulator/src/NetworkNode.ts
class NetworkNode {
    private peers: NetworkNode[] = [];
    private mempool: Transaction[] = [];
    private propagationDelay: number;
    
    constructor(id: string, delay: number = 100) {
        this.id = id;
        this.propagationDelay = delay;
    }
    
    async broadcastTransaction(tx: Transaction) {
        this.mempool.push(tx);
        
        // Simulate network propagation
        await Promise.all(this.peers.map(async peer => {
            await new Promise(resolve => 
                setTimeout(resolve, this.propagationDelay));
            peer.receiveTransaction(tx, this);
        }));
    }
    
    async receiveTransaction(tx: Transaction, from: NetworkNode) {
        if (!this.mempool.find(t => t.id === tx.id)) {
            this.mempool.push(tx);
            
            // Propagate to other peers
            const otherPeers = this.peers.filter(p => p !== from);
            await Promise.all(otherPeers.map(async peer => {
                await new Promise(resolve => 
                    setTimeout(resolve, this.propagationDelay));
                peer.receiveTransaction(tx, this);
            }));
        }
    }
}
```

## 🚀 Project 3: Mining Reward Calculator

A React application that visualizes Bitcoin's monetary policy.

```typescript
// mining-calculator/src/components/RewardCalculator.tsx
const RewardCalculator: React.FC = () => {
    const [currentBlock, setCurrentBlock] = useState(0);
    const [totalSupply, setTotalSupply] = useState(0);
    
    const calculateRewards = (blockHeight: number) => {
        let reward = 50;
        let supply = 0;
        
        for (let i = 0; i <= blockHeight; i++) {
            if (i > 0 && i % 210000 === 0) {
                reward /= 2;
            }
            supply += reward;
        }
        
        return { reward, supply };
    };
    
    useEffect(() => {
        const { reward, supply } = calculateRewards(currentBlock);
        setTotalSupply(supply);
    }, [currentBlock]);
    
    return (
        <div className="reward-calculator">
            <div className="controls">
                <label>Block Height</label>
                <input 
                    type="range" 
                    min="0" 
                    max="6930000" 
                    value={currentBlock}
                    onChange={e => setCurrentBlock(Number(e.target.value))}
                />
                <span>{currentBlock}</span>
            </div>
            
            <div className="stats">
                <div>Total Supply: {totalSupply.toFixed(2)} BTC</div>
                <div>Percent Mined: {((totalSupply / 21000000) * 100).toFixed(2)}%</div>
                <div>Years Since Genesis: {(currentBlock * 10 / (60 * 24 * 365)).toFixed(2)}</div>
            </div>
        </div>
    );
};
```

## 🎯 Learning Outcomes
Through these projects, students will:
1. Visualize blockchain growth and network effects
2. Understand Bitcoin's monetary policy through simulation
3. See how transactions propagate through the network
4. Grasp the relationship between difficulty, time, and block rewards

## 📚 Additional Project Ideas

1. **Block Explorer Lite**
   - View block headers
   - Track chain reorganizations
   - Visualize merkle trees

2. **Difficulty Calculator**
   - Simulate difficulty adjustments
   - Visualize hashrate changes
   - Show network security metrics

3. **UTXO Visualizer**
   - Track UTXO set growth
   - Visualize address balances
   - Show transaction graphs

## 🔧 Setup Instructions

1. Clone the repository
2. Install dependencies:
```bash
npm install
# or
yarn install
```

3. Start the development server:
```bash
npm run dev
# or
yarn dev
```

## 🎯 Exercises

1. Modify the simulation parameters to see different scenarios
2. Add more network statistics
3. Implement different visualization styles
4. Add real-time data feeds

Remember: These projects are meant to illustrate concepts. They use simplified models but maintain accuracy in demonstrating Bitcoin's core principles. 