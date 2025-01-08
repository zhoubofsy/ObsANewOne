使用Go的官方以太坊实现[go-ethereum](https://github.com/ethereum/go-ethereum)来和以太坊区块链进行交互。Go-ethereum，也被简称为Geth，是最流行的以太坊客户端。因为它是用Go开发的，当使用Golang开发应用程序时，Geth提供了读写区块链的一切功能。

# 环境

- go version go1.23.0 darwin/arm64
- github.com/ethereum/go-ethereum v1.14.12
- github.com/gin-gonic/gin v1.10.0

# 代码示例

完整代码请见：https://github.com/zhoubofsy/web3_golang/tree/main/gin

## Balance查看

*account.go*
```go
type OpAccount struct {
  client *blockchain.Client
}

func (op *OpAccount) GetBalance(accountID string, blkNum *big.Int) (*big.Int, error) {
  if accountID == "" {
    return nil, ErrAccountIDRequired
  }
  // 实现获取余额
  return op.client.Eth.BalanceAt(context.Background(), common.HexToAddress(accountID), blkNum)
}

func (op *OpAccount) GetPendingBalance(accountID string) (*big.Int, error) {
  if accountID == "" {
    return nil, ErrAccountIDRequired
  }
  return op.client.Eth.PendingBalanceAt(context.Background(), common.HexToAddress(accountID))
}
```

## Block查询 

*block.go*
```go
type OpBlock struct {
  client *blockchain.Client
}

func NewOpBlock(client *blockchain.Client) *OpBlock {
  return &OpBlock{
    client: client,
  }
}

func (op *OpBlock) GetBlockNumber() (uint64, error) {
  return op.client.Eth.BlockNumber(context.Background())
}

func (op *OpBlock) GetBlockInfo(number uint64) (BlockInfo, error) {
  blkInfo, err := op.client.Eth.BlockByNumber(context.Background(), big.NewInt(int64(number)))
  if err != nil {
    return BlockInfo{}, err
  }
  return BlockInfo{
    Hash:       blkInfo.Hash().Hex(),
    Height:     blkInfo.Number().Uint64(),
    Timestamp:  blkInfo.Time(),
    Difficulty: blkInfo.Difficulty().Uint64(),
    Nonce:      blkInfo.Nonce(),
    Miner:      blkInfo.Coinbase().Hex(),
    TransCount: uint64(len(blkInfo.Transactions())),
  }, err
}

func (op *OpBlock) ListBlocks(from, to uint64) ([]BlockInfo, error) {
  blkMaxNum, err := op.GetBlockNumber()
  if err != nil {
    return nil, err
  }
  start := max(from, 0)
  end := min(to, blkMaxNum)

  blocks := make([]BlockInfo, 0)
  for i := start; i <= end; i++ {
    bi, err := op.GetBlockInfo(i)
    if err != nil {
      continue
    }
    blocks = append(blocks, bi)
  }
  return blocks, nil
}
```

## 交易查询

*trans.go*
```go
type OpTrans struct {
  client *blockchain.Client
}

func NewOpTrans(bcClient *blockchain.Client) *OpTrans {
  return &OpTrans{client: bcClient}
}

func (op *OpTrans) Transfer(to string, value uint64) (string, error) {
  // 1. 使用私钥生成 ECDSA 密钥对
  privateKey, err := crypto.HexToECDSA("ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80")
  if err != nil {
    return "", err
  }
  // 2. 从私钥中获取公钥
  pubKey, ok := privateKey.Public().(*ecdsa.PublicKey)
  if !ok {
    return "", errors.New("failed to get public key")
  }
  // 3. 从公钥中获取From地址
  fromAddress := crypto.PubkeyToAddress(*pubKey)
  toAddress := common.HexToAddress(to)

  // 4. 获取当前账户(From地址)的nonce值
  nonce, err := op.client.Eth.PendingNonceAt(context.Background(), fromAddress)
  if err != nil {
    return "", err
  }
  // 5. 设置转账金额
  val := big.NewInt(int64(value * 1000000000000000000)) // in wei (1 eth)
  // 6. 设置gasLimit
  //gasLimit := uint64(21000) // in units
  gasLimit, err := op.client.Eth.EstimateGas(context.Background(), ethereum.CallMsg{
    From:  fromAddress,
    To:    &toAddress,
    Value: val,
    Data:  nil,
  })
  if err != nil {
    return "", err
  }
  // 7. 获取当前推荐的gasPrice
  gasPrice, err := op.client.Eth.SuggestGasPrice(context.Background())
  if err != nil {
    return "", err
  }
  // 8. 获取当前网络的chainID
  chainId, err := op.client.Eth.NetworkID(context.Background())
  if err != nil {
    return "", err
  }
  // 9. 创建交易
  tx := types.NewTransaction(nonce, toAddress, val, gasLimit, gasPrice, nil)
  // 10. 签名交易
  signTx, err := types.SignTx(tx, types.NewEIP155Signer(big.NewInt(chainId.Int64())), privateKey)
  if err != nil {
    return "", err
  }
  // 11. 发送交易
  err = op.client.Eth.SendTransaction(context.Background(), signTx)
  if err != nil {
    return "", err
  }
  return signTx.Hash().Hex(), err
}

func (op *OpTrans) GetHeaderTransactionCount() (uint, error) {
  headerBlockNum, err := op.client.Eth.HeaderByNumber(context.Background(), nil)
  if err != nil {
    log.Fatal(err)
  }
  blockInfo, err := op.client.Eth.BlockByNumber(context.Background(), big.NewInt(int64(headerBlockNum.Number.Int64())))
  if err != nil {
    log.Fatal(err)
  }
  return op.client.Eth.TransactionCount(context.Background(), blockInfo.Hash())
}

func (op *OpTrans) ListTX(blkHash string) ([]TXInfo, error) {
  block, err := op.client.Eth.BlockByHash(context.Background(), common.HexToHash(blkHash))
  if err != nil {
    log.Fatal(err)
    return nil, err
  }
  var txInfos []TXInfo

  for _, tx := range block.Transactions() {
    txHash := tx.Hash()
    receipt, err := op.client.Eth.TransactionReceipt(context.Background(), txHash)
    if err != nil {
      continue
    }
    chainId := tx.ChainId()
    from, err := types.Sender(types.NewEIP155Signer(chainId), tx)
    if err != nil {
      continue
    }
    txInfo := TXInfo{
      TxHash:     txHash.Hex(),
      TxValue:    tx.Value().Uint64(),
      TxGas:      tx.Gas(),
      TxGasPrice: tx.GasPrice().Uint64(),
      TxNonce:    tx.Nonce(),
      TxData:     tx.Data(),
      TxTo:       tx.To().Hex(),
      TxReceipt:  uint8(receipt.Status),
      TxFrom:     from.Hex(),
    }
    txInfos = append(txInfos, txInfo)
  }
  return txInfos, nil
}
```

## 合约的部署

首先编写一个Solidity合约。

*mytoken.sol*
```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract MyToken {
    string public constant name = "Bob's Token";
    string public constant symbol = "BBK";
    uint8 public constant decimals = 18;
    uint16 private constant increase = 1000;
    uint256 public constant totalLimit = 27000000 * (10 ** decimals);
    uint256 public totalSupply = 0;
    address private owner;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);

    modifier checkAddress(address _addr) {
        require(address(_addr) != address(0), "Invalid address.");
        _;
    }

    modifier checkBalanceOf(address _addr, uint256 _value) {
        require(balanceOf[_addr] >= _value, "Insufficient balance");
        _;
    }

    modifier checkOwner() {
        require(msg.sender == owner, "Not owner.");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function mint(address _to) public checkOwner returns (uint256) {
        uint256 mintValue = increase * (10  ** decimals);
        require(totalLimit >= (totalSupply + mintValue), "Out of limit.");
        totalSupply += mintValue;
        balanceOf[_to] += mintValue;
        return balanceOf[_to];
    }

    function transfer(address _to, uint256 _value) public checkAddress(_to) checkBalanceOf(msg.sender, _value) returns (bool success){
        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) public checkAddress(_to) checkBalanceOf(_from, _value) returns (bool success) {
        require(allowance[_from][msg.sender] >= _value, "No enough approve value.");
        allowance[_from][msg.sender] -= _value;
        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(_from, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value) public returns (bool success){
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }
}
```

然后使用工具`abigen`将这个合约编译并导出成`mytoken.go`的源代码文件。

```shell
$ abigen --bin ./MyToken.bin --abi MyToken.abi --out ./mytoken.go --pkg mytoken
```

最后在部署的时候调用引入这个包并调用其中的部署方法完成部署。

*contract.go*
```go
type contract struct {
  client *blockchain.Client
}

func NewContract(client *blockchain.Client) *contract {
  return &contract{client: client}
}

func (c *contract) DeployContract() (string, string, error) {
  privateKey, err := crypto.HexToECDSA("ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80")
  if err != nil {
    return "", "", err
  }
  chainId, err := c.client.Eth.ChainID(context.Background())
  if err != nil {
    return "", "", err
  }
  txOpts, err := bind.NewKeyedTransactorWithChainID(privateKey, chainId)
  if err != nil {
    return "", "", err
  }
  bk := backend.NewMyTokenCB(c.client.Eth)
  // TODO: 部署合约
  contractAddress, txHash, _, err := mytoken.DeployMytoken(txOpts, *bk)
  return contractAddress.Hex(), txHash.Hash().Hex(), err
}
```

### 关于abigen工具的编译安装

克隆`go-ethereum`代码

```shell
$ git clone git@github.com:ethereum/go-ethereum.git ethereum/go-ethereum
```

编译`go-ethereum`

```shell
$ make
go run build/ci.go install ./cmd/geth
>>> /usr/local/go/bin/go build -ldflags "--buildid=none -X github.com/ethereum/go-ethereum/internal/version.gitCommit=f861535f1ecc59ad279c35f77f3962efc14dcf98 -X github.com/ethereum/go-ethereum/internal/version.gitDate=20241219 -s" -tags urfave_cli_no_docs,ckzg -trimpath -v -o /Users/zhoub/Labs/ethereum/go-ethereum/build/bin/geth ./cmd/geth
internal/goarch
internal/profilerecord
internal/unsafeheader
internal/race
internal/goexperiment
internal/coverage/rtcov
internal/byteorder
...
github.com/cockroachdb/pebble
github.com/ethereum/go-ethereum/ethdb/pebble
github.com/ethereum/go-ethereum/node
github.com/ethereum/go-ethereum/ethstats
github.com/ethereum/go-ethereum/graphql
github.com/ethereum/go-ethereum/eth
github.com/ethereum/go-ethereum/eth/catalyst
github.com/ethereum/go-ethereum/cmd/utils
github.com/ethereum/go-ethereum/cmd/geth
Done building.
Run "./build/bin/geth" to launch geth.
```

```shell
$ make devtool
env GOBIN= go install golang.org/x/tools/cmd/stringer@latest
go: downloading golang.org/x/tools v0.28.0
env GOBIN= go install github.com/fjl/gencodec@latest
go: downloading github.com/fjl/gencodec v0.0.0-20230517082657-f9840df7b83e
go: downloading github.com/garslo/gogen v0.0.0-20170306192744-1d203ffc1f61
go: downloading golang.org/x/tools v0.0.0-20191126055441-b0650ceb63d9
env GOBIN= go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go: downloading google.golang.org/protobuf v1.36.1
env GOBIN= go install ./cmd/abigen
solc is /opt/homebrew/bin/solc
protoc is /opt/homebrew/bin/protoc
```

## 合约的调用

*contract.go*
```go
func (c *contract) Call(addr string, category string, params interface{}) (interface{}, error) {
  var resp interface{}
  var err error

  instance, err := mytoken.NewMytoken(common.HexToAddress(addr), c.client.Eth)
  if err != nil {
    return nil, err
  }
  switch category {
  case "BalanceOf":
    callOpts := &bind.CallOpts{
      Pending: false,
      Context: context.Background(),
    }
    if account, ok := params.(string); ok {
      bBlance, err := instance.BalanceOf(callOpts, common.HexToAddress(account))
      if err != nil {
        return nil, err
      }
      resp = bBlance.String()
    } else {
      err = errors.New("invalid params")
    }
  case "Transfer":
    // 使用私钥生成 ECDSA 密钥对
    privateKey, err := crypto.HexToECDSA("ac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80")
    if err != nil {
      return nil, errors.New("failed to get private key")
    }
    // 获取当前账户的地址
    pubAddr := crypto.PubkeyToAddress(privateKey.PublicKey)
    // 获取当前账户的nonce值
    nonce, err := c.client.Eth.PendingNonceAt(context.Background(), pubAddr)
    if err != nil {
      return nil, errors.New("failed to get nonce")
    }
    // 获取当前推荐的gasPrice
    gasPrice, err := c.client.Eth.SuggestGasPrice(context.Background())
    if err != nil {
      return nil, errors.New("failed to get gas price")
    }
    gasLimit := uint64(30000000)
    txOpts := &bind.TransactOpts{
      From:  pubAddr,
      Nonce: big.NewInt(int64(nonce)),
      Signer: func(addr common.Address, tx *types.Transaction) (*types.Transaction, error) {
        chainId, err := c.client.Eth.ChainID(context.Background())
        if err != nil {
          return nil, errors.New("failed to get chain id")
        }
        return types.SignTx(tx, types.NewEIP155Signer(chainId), privateKey)

      },
      Value:    nil,
      GasPrice: gasPrice,
      GasLimit: gasLimit,
      Context:  context.Background(),
    }
    if txParams, ok := params.(TransactParams); ok {
      resp, err = instance.TransferFrom(txOpts, common.HexToAddress(txParams.TxFrom),
        common.HexToAddress(txParams.TxTo), big.NewInt(int64(txParams.TxValue)))
    } else {
      return nil, errors.New("invalid params")
    }

  default:
    return nil, errors.New("unsupported category")
  }
  return resp, err
}
```

## 合约的事件

智能合约具有在执行期间“发出”事件的能力。 事件在以太坊中也称为“日志”。 事件的输出存储在日志部分下的事务处理中。 事件已经在以太坊智能合约中被广泛使用，以便在发生相对重要的动作时记录，特别是在代币合约（即ERC-20）中，以指示代币转账已经发生。

*event.go*
```go
func (e *ContractEvent) Run(contractAddr string) {
  query := ethereum.FilterQuery{
    Addresses: []common.Address{common.HexToAddress(contractAddr)},
  }

  ch := make(chan types.Log)
  sp, err := e.Client.SubscribeFilterLogs(context.Background(), query, ch)
  if err != nil {
    fmt.Printf("SubscribeFilterLogs err: %v\n", err)
    return
  }
  for {
    select {
    case err := <-sp.Err():
      fmt.Printf("sp err: %v\n", err)
      return
    case vLog := <-ch:
      logJSON, err := json.Marshal(vLog)
      if err != nil {
        fmt.Printf("json.Marshal err: %v\n", err)
        return
      }
      fmt.Printf("vLog: %s\n", logJSON)
    }
  }
}
```

event.go
```go
func (e *ContractEvent) ListWithBlkId(contractAddr string, fromBlock, toBlock uint64) ([]LogInfo, error) {
  if fromBlock > toBlock {
    return nil, fmt.Errorf("fromBlock > toBlock")
  }
  var fBlock *big.Int
  var tBlock *big.Int
  if fromBlock > 0 {
    fBlock = big.NewInt(int64(fromBlock))
  }
  if toBlock > 0 {
    tBlock = big.NewInt(int64(toBlock))
  }

  query := ethereum.FilterQuery{
    Addresses: []common.Address{common.HexToAddress(contractAddr)},
    FromBlock: fBlock,
    ToBlock:   tBlock,
  }

  logs, err := e.Client.FilterLogs(context.Background(), query)
  if err != nil {
    return nil, err
  }

  contractABI, err := abi.JSON(strings.NewReader(mytoken.MytokenABI))
  if err != nil {
    return nil, err
  }
  var logsInfo []LogInfo
  var logType string
  for _, vLog := range logs {
    if topic, ok := e.TopicMap[vLog.Topics[0].Hex()]; !ok {
      fmt.Printf("topic not found: %s\n", vLog.Topics[0].Hex())
      continue
    } else {
      logType = topic
    }
    parseData, err := contractABI.Unpack(logType, vLog.Data)
    if err != nil {
      if logJSON, err := json.Marshal(vLog); err == nil {
        fmt.Printf("Unpack err: %v\n %s", err, logJSON)
      } else {
        fmt.Printf("Unpack err: %v\n %v", err, vLog)
      }
      continue
    }
    var strData string
    switch logType {
    case TransferTopic:
      strData = parseData[0].(*big.Int).String()
    case ApproveTopic:
      strData = parseData[0].(*big.Int).String()
    default:
      fmt.Printf("Unknow parse data: %v\n", parseData)
      strData = "unknow"
    }
    logsInfo = append(logsInfo, LogInfo{
      //Log: vLog,
      LogType:     logType,
      FromAddress: common.HexToAddress(vLog.Topics[1].Hex()).Hex(),
      ToAddress:   common.HexToAddress(vLog.Topics[2].Hex()).Hex(),
      ParseData:   strData,
    })
  }
  return logsInfo, nil
}
```

