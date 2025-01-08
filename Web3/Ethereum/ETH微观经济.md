在以太坊中，存在一个**铸造**和**销毁**机制的平衡，这样可以有效控制以太坊的供应量，防止通货膨胀或通货紧缩，同时促进ETH的价值保持稳定。这个平衡的关键是：  
- **铸造过程**：指通过出块奖励为验证者铸造新的ETH。  
- **销毁过程**：通过交易中 `baseFee` 的销毁来减少ETH的总供应量。

# 铸造过程：新区块奖励

在以太坊的**PoS（权益证明）**模式下，每当一个新区块被生产时，系统会给予验证者一定数量的ETH作为奖励。这些奖励是通过**以太坊网络**的共识机制铸造出来的，是**凭空创造**的。

## 奖励计算公式
以太坊2.0的验证者奖励包括了基础奖励（由区块生产产生）和根据**`baseFee`** 的销毁以及网络活动（例如gas价格）带来的额外奖励。

### 基础奖励（Base Reward）
   每个验证者在每个 Epoch 中的基础奖励基于质押总量和总验证者数量计算。  

   **公式：**  
   
$$\text{Base Reward} = \frac{\text{Effective Balance} \times R}{\text{Total Balance}}$$

   - **Effective Balance**: 验证者的有效质押余额（最多 32 ETH，过多部分不计入）。  
   - **R**: 奖励因子，与网络的总质押量（`Total Balance`）相关。  
   - **Total Balance**: 网络中所有验证者的质押总量。

   **R 的计算**：
   
$$R = \frac{\text{Base Reward Per Increment}}{\sqrt{\text{Total Active Balance}}}$$

   - **Base Reward Per Increment**: 固定参数，当前为 64（具体值可能随协议更新变化）。  
   - **Total Active Balance**: 所有验证者的活跃质押余额总和。  

   **简化理解**：总质押越高，每个验证者的奖励越低；总质押越低，每个验证者的奖励越高。


### 出块奖励（Proposer Reward）
   出块的验证者会获得额外的奖励，通常来自网络中的交易小费（`Tip`）。  
   **公式：**
   
   $$\text{Proposer Reward} = \frac{1}{8} \times \text{Total Attestation Rewards}$$
   
   - **Total Attestation Rewards**: 当前区块中其他验证者的总奖励。

### 小费（Tips）
   验证者还可以通过出块收取交易小费，作为额外的激励。  
   **公式：**
   
$$\text{Tips} = \text{maxPriorityFeePerGas} \times \text{GasUsed}$$

   - 小费是交易者直接支付给验证者的费用，主要用于提升交易优先级。
  
## 奖励的分配

**出块奖励**：主要分配给验证者。具体的奖励数量和验证者质押的ETH数量有关。  

**验证者奖励（年化）**：网络中的所有验证者按质押ETH的比例来分享总的奖励池，基于**年化利率**进行分配。

1. **基础奖励分配**  
   验证者参与网络共识（如投票或提议新区块）时，按比例分配基础奖励。未按要求参与时将减少奖励。

2. **附加奖励分配**  
   验证者根据其行为（如投票是否及时、正确）获得额外奖励。  
   **行为奖励类型：**
   - **同步贡献奖励**：如果验证者正确参与区块提议和投票。  
   - **罚没机制**：恶意行为（如双签名）导致奖励减少或直接罚没部分质押。

3. **动态调整年化收益率**  
   - 质押总量（网络参与率）影响验证者奖励：  
	$$\text{Yearly Yield} \propto \frac{1}{\sqrt{\text{Total Stake}}}$$
   - 如果质押率较低，验证者的年化收益率上升，激励更多人质押。

**触发铸造的条件**：
- 每生产一个新区块，系统会根据当前的奖励机制铸造出新的ETH，并将其奖励给验证者。  
- 该过程的触发条件就是区块的产生与验证，且与链上交易的数量、复杂度等因素无关。

## **举例计算**  
假设：
- 网络总质押量：10,000,000 ETH  
- 验证者质押：32 ETH  
- 基础奖励因子：64  

1. **基础奖励**  
$$
   \text{Base Reward} = \frac{32 \times 64}{\sqrt{10,000,000}} \approx 0.64 \, \text{ETH / Epoch}
$$
   每 6.4 分钟获得约 0.64 ETH。

2. **年化收益率**  
$$
   \text{Yearly Yield} = 0.64 \times 365 \times 24 \times 60 / 6.4 \approx 8.76 \% \, \text{年化收益率}
$$

---

# 销毁过程：`baseFee` 的销毁

为了防止ETH的通货膨胀，**EIP-1559** 引入了销毁机制，旨在通过**销毁`baseFee`** 来减少ETH的总供应量，从而对抗铸造过程带来的通货膨胀。

## `baseFee` 销毁机制

`baseFee`是由网络中的区块大小决定的基础费用，动态调整，以保持网络的区块容量稳定。这个费用的**一部分会被销毁**，而不是奖励给验证者。  在以太坊中，`baseFee` 是交易费用的一部分，定义了在一个区块中执行交易所需支付的最低费用。在 **EIP-1559** 提案中，`baseFee` 引入了一个**动态调整机制**，该机制根据网络的负载和拥堵情况自动调整 `baseFee` 的值。这个调节机制的设计目标是优化交易费用的透明性和可预测性，并避免交易费用波动过大。

###  1. `baseFee` 机制的基本概念

`baseFee` 是每笔交易的最低费用，并且是由网络自动调节的，基于以下几个因素：
- 网络当前的 **负载**（即当前区块的使用情况）。
- 网络 **目标的区块大小**（即区块目标的 gas 限额）。

EIP-1559 改变了传统的以太坊费用模型，使得交易费用变得更加**可预测**，并且将一部分费用 **销毁**，而不是支付给矿工。`baseFee` 作为这部分费用的核心部分，其动态调整机制起到了关键作用。

### 2. `baseFee` 的动态调节原理

`baseFee` 是根据上一块区块的拥堵情况进行调整的。调整的规则如下：

- **目标区块大小**：以太坊的目标区块大小是 **15,000,000 gas**（以太坊网络上的一个理想值）。区块的实际大小会决定 `baseFee` 的调整。
- **如果区块的 gas 使用量高于目标（即区块接近 15,000,000 gas）**，`baseFee` 会**增加**，以此来降低交易的需求，并避免网络拥堵。
- **如果区块的 gas 使用量低于目标（即区块未满）**，`baseFee` 会**减少**，以鼓励更多交易的提交，提升区块的利用率。

具体的调整规则是基于区块内的 gas 使用量（`gasUsed`）与目标值（`gasTarget`）之间的差异来计算：

- 每个区块的 `baseFee` 会根据以下公式调整：

$$
  \text{new baseFee} = \text{old baseFee} + \text{adjustment}
$$

  其中，`adjustment` 是根据以下条件计算的：

  - **如果区块的 gas 使用量高于目标**，`baseFee` 会增加。增加的幅度为 `baseFee` 的 **1/8**。
  - **如果区块的 gas 使用量低于目标**，`baseFee` 会减少。减少的幅度同样是 `baseFee` 的 **1/8**。

  即：

$$
  \text{adjustment} = \text{baseFee} \times \frac{1}{8} \times \left( \frac{\text{gasUsed}}{\text{gasTarget}} - 1 \right)
$$

  这个公式确保了 `baseFee` 逐渐适应区块的实际使用情况，避免了过大的波动。

- **调整的上限和下限**：
  - `baseFee` 的调整是逐步的，每个区块的 `baseFee` 相对于上一块最多只能增加或减少 `1/8`。
  - 这意味着即使网络负载突然增加或减少，`baseFee` 也不会立即发生剧烈变化，避免了费用过高或过低的极端情况。

### 3. `baseFee` 、 `maxFeePerGas` 、`gasLimit`和 `maxPriorityFeePerGas` 的关系

在以太坊的**EIP-1559** 交易费用模型中，**`maxFeePerGas`**, **`maxPriorityFeePerGas`**, **`baseFee`**, 和 **`gas limit`** 是关键参数，它们共同决定了用户支付的总交易费用以及矿工/验证者的收入。

#### 定义与含义

- **`maxFeePerGas`**：
  - 用户愿意为交易支付的每单位`gas`的最大费用（总上限）。
  - 这是用户设定的费用上限，用于限制支付的最高费用。
  - 单位：Gwei。

- **`maxPriorityFeePerGas`**：
  - 用户希望额外支付给矿工/验证者的每单位`gas`的小费（奖励）。
  - 这个值是用户主动提供的，通常用于鼓励矿工/验证者优先处理自己的交易。
  - 单位：Gwei。
  - 如果设置为0，则交易没有小费。

- **`baseFee`**：
  - 每单位`gas`的基础费用，由网络动态调整，决定交易执行所需的最小费用。
  - 由协议规定，所有交易的`baseFee`部分都会被销毁，而不是分配给矿工/验证者。
  - 单位：Gwei。
  - **动态调整规则**：
    - 如果区块的实际`gas`使用量超过目标（区块`gas limit`的一半），则`baseFee`会增加。
    - 如果使用量低于目标，则`baseFee`会减少。

- **`gas limit`**：
  - 交易中可以消耗的最大`gas`数量，用户设定的上限。
  - 确保用户不会为超出需求的计算支付费用。
  - 区块级的`gas limit`决定了整个区块内最多可消耗的`gas`数量。

#### 计算逻辑与关系

**用户实际支付的费用（effective gas fee）**
用户为每单位`gas`实际支付的费用可以表示为：
$$
\text{Effective Gas Fee} = \text{baseFee} + \min(\text{maxPriorityFeePerGas}, (\text{maxFeePerGas} - \text{baseFee}))
$$

- **解释**：
  - **`baseFee`**：这是网络最低要求的费用，每笔交易都必须支付。
  - **小费部分**：用户支付的实际小费是`maxPriorityFeePerGas`与`maxFeePerGas - baseFee`的较小值。
  - 如果`maxFeePerGas`不足以覆盖`baseFee`，交易将被拒绝。

**总交易费用（total transaction fee）**
用户支付的总费用计算为：
$$
\text{Total Transaction Fee} = \text{Effective Gas Fee} \times \text{Gas Used}
$$

- **`Gas Used`**：是交易实际消耗的`gas`量，通常小于或等于用户设置的`gas limit`。
- **`baseFee` 与 `maxFeePerGas` 的关系**：
  - 如果用户设定的`maxFeePerGas`小于当前的`baseFee`，交易无法提交。
  - 用户支付的实际费用不会超过`maxFeePerGas`。

- **`maxPriorityFeePerGas` 与 `maxFeePerGas` 的关系**：
  - `maxPriorityFeePerGas` 决定用户愿意支付的小费，实际的小费不能超过`maxFeePerGas - baseFee`。
  - 如果`baseFee`接近`maxFeePerGas`，小费部分会自动减少。

- **`gas limit` 的作用**：
  - 用户设定的`gas limit`决定了交易的最大`gas`消耗。
  - 用户支付的总费用与实际消耗的`gas`数量相关，未使用的`gas`会退还。

#### 举例说明

**假设参数如下：**
- `maxFeePerGas = 50 Gwei`
- `maxPriorityFeePerGas = 10 Gwei`
- `baseFee = 30 Gwei`
- `gas limit = 21,000`

**计算步骤：**
1. **有效单价（Effective Gas Fee）**：
$$
   \text{Effective Gas Fee} = \text{baseFee} + \min(\text{maxPriorityFeePerGas}, (\text{maxFeePerGas} - \text{baseFee}))
$$
$$
   \text{Effective Gas Fee} = 30 + \min(10, (50 - 30)) = 30 + 10 = 40 \, \text{Gwei}
$$

2. **总费用（Total Transaction Fee）**：
$$
   \text{Total Transaction Fee} = \text{Effective Gas Fee} \times \text{Gas Used}
$$
   如果实际使用的`gas`为21,000：
$$
   \text{Total Transaction Fee} = 40 \times 21,000 = 840,000 \, \text{Gwei} = 0.00084 \, \text{ETH}
$$

3. **销毁的ETH**：
   - 销毁部分仅包含`baseFee`：

     $$\text{Burned Fee} = \text{baseFee} \times \text{Gas Used}$$
     $$\text{Burned Fee} = 30 \times 21,000 = 630,000 \, \text{Gwei} = 0.00063 \, \text{ETH}$$

4. **验证者收益（小费部分）**：
   - 小费是`maxPriorityFeePerGas`：
     $$\text{Tip} = 10 \times 21,000 = 210,000 \, \text{Gwei} = 0.00021 \, \text{ETH}$$

### 4. 销毁的机制与 `baseFee`

在 EIP-1559 机制中，`baseFee` 不会直接支付给矿工，而是被**销毁**（burned），从而减少 ETH 的总供应量。

- 销毁的 ETH 数量是根据每个区块的 `baseFee` 和区块的 gas 使用量（`gasUsed`）来决定的。
- 销毁的 ETH 量等于每个交易的 `baseFee` 与该交易消耗的 gas 量的乘积：
  
$$
  \text{burned ETH} = \text{baseFee} \times \text{gasUsed}
$$

  随着网络交易量的增加，销毁的 ETH 数量也会增多，从而可能导致 ETH 的供应量减少，产生通货紧缩的效果。

**销毁过程**：每当发生交易时，系统会根据交易中的`baseFee`来销毁一定数量的ETH。这意味着**`baseFee`** 部分的ETH会永久消失，从而减少市场上的ETH总供应量。

**`baseFee` 的销毁机制会被触发的情况**：
- 当用户发起交易时，交易费用中包含的 **`baseFee`** 会被销毁，而不是奖励给矿工或验证者。  
- `baseFee` 会随着网络的拥堵情况动态调整，当区块满时，`baseFee` 增加；反之，当区块空闲时，`baseFee` 会减少。  
- 销毁的`baseFee` 每个区块的具体数量是通过算法自动计算的，并且每个区块的`baseFee`会根据网络的拥堵状况自动变化。

## 触发销毁的过程

- 触发销毁的主要过程是用户发起交易并提交到网络中。这时，系统会根据交易的`gasPrice`（包括`baseFee`和`maxPriorityFeePerGas`）来计算销毁的ETH数量。
- 除了交易之外，某些智能合约的操作也会触发`baseFee`的销毁。例如，执行合约调用时，合约内部的交易（如ERC-20代币转账）也会根据相应的`baseFee`销毁ETH。


---

# 良性循环：铸造和销毁的平衡

**铸造与销毁形成平衡**：
- **铸造过程**：每个新区块奖励一定数量的ETH给验证者，增加市场的ETH供应量。
- **销毁过程**：通过销毁`baseFee`，减少市场的ETH供应量，避免过度膨胀。

**如何保持ETH供应量稳定？**
- 在一个健康的以太坊网络中，**铸造和销毁机制会共同作用**，实现供需平衡，防止过度的通货膨胀。
- **销毁机制对供应量的影响**：通过销毁`baseFee`，ETH的供应量会得到控制。具体销毁多少ETH，取决于网络的交易量和拥堵状况。
- **网络需求与奖励**：交易量越高，`baseFee` 就越高，销毁的ETH就越多，这可以有效减缓通货膨胀的速度。
- **验证者奖励**：验证者通过出块得到的奖励是新增的ETH，在网络正常运行时，这部分奖励与销毁的ETH保持平衡，避免ETH的数量过快增加。

**合适的销毁与铸造机制**：
- **区块奖励与销毁的关系**：销毁`baseFee`的ETH，有时可能会比新增奖励的ETH更多，这就可能导致长期内ETH的总供应量减少，进而形成某种程度的**通货紧缩**。
- 在交易需求高、`baseFee` 较高的情况下，销毁的ETH数量可能大于验证者奖励的ETH，从而实现一个**负增长**或稳定的供应量。
- 在交易需求低、`baseFee` 较低的情况下，新增的ETH数量可能高于销毁量，导致ETH的供应量逐渐增加。
