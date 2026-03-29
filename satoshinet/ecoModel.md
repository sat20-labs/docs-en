Economic Model of SatoshiNet
====

The income sources of SatoshiNet are primarily divided into two parts: transaction packaging fees (network fees) and earnings from liquidity pool fund inflows and outflows. The SatoshiNet Foundation takes a commission from both income sources, starting at 20% in the early stages and gradually decreasing as the network develops.

#### 1. Basic Assumptions
To better consider the revenue sources for both platform operators and liquidity pool participants, the following parameters are used:
- **Transaction packaging fee**: Fixed at 10 satoshis per transaction.
- **Daily transaction volume**:
    - **Guidance period**: Starting from zero and growing to 1,000 transactions per day, stabilizing the network and preparing for more projects to join.
    - **Development period**: Transaction volume grows from 1,000 to 10,000 transactions per day, with at most 10 active or large token projects joining.
    - **Mature period**: Transaction volume grows from 10,000 to 1 million transactions per day, with at most 100 active tokens traded.
    - **Explosive period**: Over 1 million transactions per day, with the most popular BTC ecosystem tokens being traded on SatoshiNet.

#### 2. Mining Nodes
Mining nodes gain the right to mine by staking **BTC** tokens.
- **Mining node requirements**: The number of nodes required changes during each development phase.
- **Staking amount** is temporarily set at 1 BTC tokens.

**Mining Revenue by Phase**:
- **Guidance Period**: 2–3 nodes, no focus on economic profit.
- **Development Period**: 10 nodes. With daily transactions under 10,000, each node will mine 720 blocks per day. With 1 transactions per block, each block earns 10 satoshis, yielding daily earnings of 7,200 satoshis, and yearly earnings of about 0.02628 BTC ($2,628 USD).
- **Mature Period**: 100 nodes. Assuming 100,000 transactions, each mining machine will still earn 0.02628 BTC annually. If transaction volume increases to 1 million, each node's revenue will increase to 0.2628 BTC. This could be a crucial dividing line: while a significant amount of liquidity is locked up in mining machines, revenue could steadily increase.
- **Explosive Period**: SatoshiNet's daily transaction volume exceeds 1 million, potentially reaching 30 million. Assuming an average of 5 million transactions per day during the explosive growth phase.

#### 3. Liquidity Pools
The liquidity pool provides liquidity in and out of the SatoshiNet. In principle, profit sharing is based on the proportion of funds in the pool, and the foundation receives a commission on the pool's income.

#### 4. Revenue Distribution Model
**Principles**:
- Those who provide services receive the profits.
- The foundation takes a percentage of the revenue for ecosystem development. The revenue share varies across different periods.
    - **Guidance Period**: No commission taken.
    - **Development Period**: 5% commission.
    - **Mature Period**: 3% commission.
    - **Explosive Period**: 1% commission, with a minimum guarantee of 1%.

##### **Liquidity Pool Participant Earnings**
- In the **Explosive Period**, with a daily transaction volume of 5 million transactions, 2% of the funds are expected to flow in and out of SatoshiNet. Funds entering the network are free of charge, but exiting the network incurs a service fee. The fixed service fee is 3,000 satoshis + 0.8% of the amount being withdrawn.
- Assuming an average of 10,000 transactions per day, each worth 100,000 satoshis, a withdrawal transaction will incur 3,000 satoshis of fixed service fees + 8,000 satoshis of fund fees, totaling 11,000 satoshis. This generates a daily cost of approximately 1.1 BTC. Of course, there are about 0.6 BTC in the adjustment channel and the cost of cleaning up various scattered utxo, but there is also a profit of 0.5 BTC.
- At this stage, with a liquidity pool size of 1,000 BTC, the annual revenue could be 182.5 BTC ($18.2 million), with an annual yield of **18.2%**. If a participant stakes $10,000, their annual earnings would be approximately **$1,820**.

##### **Mining Node Earnings**
- Mining nodes must pay a portion of their earnings to the foundation, up to 5%. Additionally, regular mining nodes need to pay core nodes a data service fee, up to 10%.
- The remaining earnings belong to the node operators themselves.

**Explosive Period Example** (5 million daily transactions and 100 nodes):
- **Revenue per node**: 5,0000 transactions per day × 10 satoshis per transaction = 50,0000 satoshis/day.
- **Annual earnings per node**: 50,0000 satoshis/day × 365 = 1.825 BTC/year.
    - Foundation share (5%) = 0.09125 BTC
    - Core node share (10%) = 0.1825 BTC
    - Node operator share (85%) = 1.55125 BTC

**Node Costs**:
- staking 1 BTC.
- Hardware and network costs per node are expected to be less than 0.03 BTC per year.

**Node ROI**:  
- In the mature period, the return on investment for a mining node would be approximately **15%**.

#### 5. Dynamic Adjustment Mechanism
To balance platform earnings and participant attractiveness while adapting to market development, the following dynamic adjustments can be implemented:
1. **Foundation’s Revenue Share**: Starting at 5%, gradually decreasing to a stable range of 1%.
2. **SatoshiNet Network Fee**: Currently 10 satoshis, but it could decrease as BTC prices rise and transaction volumes increase. The fee may eventually drop to 1 satoshi per transaction.

---

**Note**: This is a proposal and should be reviewed and approved by the foundation before implementation.