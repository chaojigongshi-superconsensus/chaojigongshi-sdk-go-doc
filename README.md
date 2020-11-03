# chaojigongshi sdk go 中文版

# 1. Chaojigongshi-SDK简介

为方便GO开发者调试和快速接入超级共识超级链，提供了一套使用超级共识超级链的强有力工具Chaojigongshi-SDK。Chaojigongshi-SDK简单易用，这里将向您介绍Chaojigongshi-SDK，并通过简单的示例帮助您快速熟悉Chaojigongshi-SDK的使用方法，并构建自己的应用。

# 2. 模块介绍

使用Chaojigongshi-SDK可以快速的开发与超级共识超级链交互的应用程序，Chaojigongshi-SDK提供以下几个方向的API:

- 账户, 包括创建区块链账号，通过助记词恢复区块链账户，保存账号加密文件到本地，从本地加密文件恢复账户等
- 合约账户, 包括创建合约账户等
- 智能合约, 包括智能合约的部署、调用、查询等
- 交易, 包括转账交易，查询交易，查询余额等

# 3. Quick Start

## 3.1 环境配置

- 操作系统：支持Linux以及Mac OS
- 开发语言：Go 1.12.x及以上
- 编译器：GCC 4.8.x及以上
- 版本控制工具：Git

## 3.2 代码获取及安装

请前往[github代码地址](https://github.com/chaojigongshi-superconsensus/chaojigongshi-sdk-go)下载或克隆最新的代码

```
git clone https://github.com/chaojigongshi-superconsensus/chaojigongshi-sdk-go

cd chaojigongshi-sdk-go
```

编译

```
make
make test
```

## 3.3 使用样例

### 3.3.1 账号

查看github.com/chaojigongshi-superconsensus/chaojigongshi-sdk-go/example/main.go文件中的testAccount()函数

```
func testAccount() {
   // 创建中文助记词、强度弱(12个词)的账户
   acc, err := account.CreateAccount(1, 1)
   if err != nil {
      fmt.Printf("create account error: %v\n", err)
   }
   // 打印账户、助记词
   fmt.Println(acc)
   fmt.Println(acc.Mnemonic)

   // 通过助记词恢复账户
   acc, err = account.RetrieveAccount(助记词, 1)   // 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }
   fmt.Printf("RetrieveAccount: %v\n", acc)

   // 创建账户, 使用账户安全码加密后保存到本地文件路径中
   acc, err = account.CreateAndSaveAccountToFile("./keys", "123", 1, 1)
   if err != nil {
      fmt.Printf("createAndSaveAccountToFile err: %v\n", err)
   }
   fmt.Printf("CreateAndSaveAccountToFile: %v\n", acc)

   // 使用账户安全码从本地文件中的加密账户中恢复出原始账户
   acc, err = account.GetAccountFromFile("./keys/", "123")
   if err != nil {
      fmt.Printf("getAccountFromFile err: %v\n", err)
   }
   fmt.Printf("GetAccountFromFile: %v\n", acc)
}
```

### 3.3.2 合约账号

查看github.com/chaojigongshi-superconsensus/chaojigongshi-sdk-go/example/main.go文件中的testContractAccount()函数

```
// 先声明超级链网络的地址和链名称
var (
   node         = "10.144.94.18:37201"
   bcname       = "xuper"
)
func testContractAccount() {
   // 通过助记词恢复出用户的账户
   account, err := account.RetrieveAccount(助记词, 1)  // 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }

   // 指定要创建的合约账户名称
   // 合约账户名为  XC+16位数字+@+链名
   contractAccount := "XC1234567890123456@xuper"

   // 实例化一个可以创建合约账户的客户端对象
   ca := contractaccount.InitContractAccount(account, node, bcname)

   // 创建合约账户, 也可以通过预执行+执行完成合约账户的创建, 详见API说明
   txid, err := ca.CreateContractAccount(contractAccount)
   if err != nil {
      log.Printf("CreateContractAccount err: %v", err)
      os.Exit(-1)
   }
   fmt.Println(txid)
}
```

### 3.3.3 转账

查看github.com/chaojigongshi-superconsensus/chaojigongshi-sdk-go/example/main.go文件中的testTransfer()、testGetBalance()、testQueryTx()函数

```
// 先声明超级链网络的地址和链名称
var (
   node         = "x.x.x.x:8080" // 节点ip：port
   bcname       = "xuper"
)
// 转账
func testTransfer() {
   // 通过助记词恢复出用户的账户
   acc, err := account.RetrieveAccount(助记词, 1)   // 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }
   fmt.Printf("account: %v\n", acc)

   // 实例化一个交易相关的客户端对象
   trans := transfer.InitTrans(acc, node, bcname)

   // 转账目的地址to、金额amount、手续费fee、转账备注desc
   to := "UgdxaYwTzUjkvQnmeB3VgnGFdXfrsiwFv"
   amount := "200"
   fee := "0"
   desc := ""
   // 发起转账操作
   tx, err := trans.Transfer(to, amount, fee, desc)
   if err != nil {
      fmt.Printf("Transfer err: %v\n", err)
   }
   fmt.Printf("transfer tx: %v\n", tx)
}
// 查询账户余额
func testGetBalance() {
   // 通过助记词恢复出用户的账户
   acc, err := account.RetrieveAccount(助记词, 1)   // 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }
   fmt.Printf("account: %v\n", acc)

   // 实例化一个交易相关的客户端对象
   trans := transfer.InitTrans(acc, node, bcname)
   // 查询账户的余额
   balance, err := trans.GetBalance()
   log.Printf("balance %v, err %v", balance, err)
}
// 查询交易
func testQueryTx() {
   // 实例化一个交易相关的客户端对象
   trans := transfer.InitTrans(nil, node, bcname)
   txid := "3a78d06dd39b814af113dbdc15239e675846ec927106d50153665c273f51001e"
   // 根据交易id 查询tx详情
   tx, err := trans.QueryTx(txid)
   log.Printf("query tx %v, err %v", tx, err)
}
```

### 3.3.4 智能合约

查看github.com/chaojigongshi-superconsensus/chaojigongshi-sdk-go/example/main.go文件中的testDeployWasmContract()、testInvokeWasmContract()、testQueryWasmContract()函数

```
// 先声明超级链网络的地址、链名称以及合约名
var (
   contractName = "counter"
   node         = "x.x.x.x:8080" // 节点ip:port
   bcname       = "xuper"
)
// 部署合约
func testDeployWasmContract() {
   // 通过助记词恢复出用户的账户
   acc, err := account.RetrieveAccount(助记词, 1) / 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }
   fmt.Printf("account: %v\n", acc)

   // 设置合约账户
   contractAccount := "XC1234xxx@xuper" // 传入账号对应的合约账户

   // 初始化一个合约相关的客户端对象
   wasmContract := contract.InitWasmContract(acc, node, bcname, contractName, contractAccount)

   // 合约初始化参数、wasm合约文件地址
   args := map[string]string{
      "creator": "xchain",
   }
   codePath := "example/contract_code/counter.wasm"

   // 部署安装合约, 也可以通过预执行+执行完成合约的部署安装, 详解API接口说明
   txid, err := wasmContract.DeployWasmContract(args, codePath, "c")
   if err != nil {
      log.Printf("DeployWasmContract err: %v", err)
      os.Exit(-1)
   }
   fmt.Printf("DeployWasmContract txid: %v\n", txid)
}
// 合约调用 Invoke上链操作
func testInvokeWasmContract() {
   // 通过助记词恢复出用户的账户
   acc, err := account.RetrieveAccount(助记词, 1) // 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }
   fmt.Printf("account: %v\n", acc)

   // 初始化一个合约相关的客户端对象; 这里合约账户可以为空或者为用户创建的合约账户
   contractAccount := ""
   wasmContract := contract.InitWasmContract(acc, node, bcname, contractName, contractAccount)

   // 合约方法参数
   args := map[string]string{
      "key": "counter",
   }
   // 调用合约, 也可以通过预执行+执行完成合约的调用, 详解API接口说明
   txid, err := wasmContract.InvokeWasmContract("increase", args)
   if err != nil {
      log.Printf("InvokeWasmContract PostWasmContract failed, err: %v", err)
      os.Exit(-1)
   }
   log.Printf("txid: %v", txid)
}
// 合约调用 Query不上链查询
func testQueryWasmContract() {
   // 通过助记词恢复出用户的账户
   acc, err := account.RetrieveAccount(助记词, 1) // 传入自己的助记词
   if err != nil {
      fmt.Printf("retrieveAccount err: %v\n", err)
   }
   fmt.Printf("account: %v\n", acc)

   // 初始化一个合约相关的客户端对象; 这里合约账户可以为空或者为用户创建的合约账户
   contractAccount := ""
   wasmContract := contract.InitWasmContract(acc, node, bcname, contractName, contractAccount)

   args := map[string]string{
      "key": "counter",
   }

   // 合约调用
   preExeRPCRes, err := wasmContract.QueryWasmContract(methodName, args)
   if err != nil {
      log.Printf("QueryWasmContract failed, err: %v", err)
      os.Exit(-1)
   }
   gas := preExeRPCRes.GetResponse().GetGasUsed()
   fmt.Printf("gas used: %v\n", gas)
   for _, res := range preExeRPCRes.GetResponse().GetResponse() {
      fmt.Printf("contract response: %s\n", string(res))
   }
}
```

# 4. API详解

除用户账户相关的操作外，其余类型的操作均需要先初始化一个相关的客户端，然后通过客户端调用相关方法。这个客户端实际上维持了对应操作所需要的基础数据，一个客户端可以多次调用同类型的操作。

## 4.1 账户

账户是指用户在区块链上的账户，账户一般包括地址、公私钥、助记词。其中，通过助记词可以恢复账户的公私钥和地址，助记词丢失则不可恢复，创建账户后请妥善保存自己的助记词。在创建账户时，也可以选择将私钥通过安全码加密后保存到本地的方式，使用时必须使用安全码解密，这样可以防止私钥文件的泄露。

### a. 创建账户

创建一个区块链账户，账户包括助记词、地址、公私钥，请妥善保存助记词，之后可以通过助记词来恢复账户。创建时，也可以通过参数指定助记词的强度以及助记词的语言。

#### 1.方法名称

- func CreateAccount(strength uint8, language int) (*Account, error)

#### 2.参数

| 参数名   | 含义                                                         | 类型  |
| -------- | ------------------------------------------------------------ | ----- |
| strength | 助记词强度，取值可以为1:助记词个数 122:助记词个数 183:助记词个数 24 | unit8 |
| language | 助记词语言，取值可以为1:中文2:英文                           | int   |

#### 3.返回

| 类型     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *Account | account的结构体，成员为Address, PrivateKey, PublicKey, Mnemonic |
| error    | nil: 创建成功non-nil: 失败                                   |

### b. 通过助记词恢复账户

使用助记词来恢复一个区块链账户

#### 1.方法名称

- func RetrieveAccount(mnemonic string, language int) (*Account, error)

#### 2.参数

| 参数名   | 含义                               | 类型   |
| -------- | ---------------------------------- | ------ |
| mnemonic | 助记词                             | string |
| language | 助记词语言，取值可以为1:中文2:英文 | int    |

#### 3.返回

| 类型     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *Account | account的结构体，成员为Address, PrivateKey, PublicKey, Mnemonic |
| error    | nil: 创建成功non-nil: 失败                                   |

### c. 创建账户并本地保存

创建一个区块链账户，创建时传入文件地址及一个安全码，可以将账户的私钥使用安全码加密后保存到对应的文件地址，后续通过文件和安全码也可恢复账户。通过此方法创建的账户也会返回助记词、公私钥、地址，请妥善保存助记词，助记词丢失则无法找回。

#### 1.方法名称

- func CreateAndSaveAccountToFile(path, passwd string, strength uint8, language int) (*Account, error)

#### 2.参数

| 参数名   | 含义                                                         | 类型   |
| -------- | ------------------------------------------------------------ | ------ |
| path     | 私钥存储路径                                                 | string |
| passwd   | 安全码，用来加密私钥                                         | string |
| strength | 助记词强度，取值可以为1:助记词个数 122:助记词个数 183:助记词个数 24 | unit8  |
| language | 助记词语言，取值可以为1:中文2:英文                           | int    |

#### 3.返回

| 类型     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *Account | account的结构体，成员为Address, PrivateKey, PublicKey, Mnemonic |
| error    | nil: 创建成功non-nil: 失败                                   |

### d. 从本地恢复账户

通过本地加密的私钥文件以及安全码恢复账户

#### 1.方法名称

- func GetAccountFromFile(path, passwd string) (*Account, error)

#### 2.参数

| 参数名 | 含义                 | 类型   |
| ------ | -------------------- | ------ |
| path   | 私钥存储路径         | string |
| passwd | 安全码，用来解密私钥 | string |

#### 3.返回

| 类型     | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *Account | account的结构体，成员为Address, PrivateKey, PublicKey, Mnemonic |
| error    | nil: 创建成功non-nil: 失败                                   |

## 4.2 合约账户

合约账户相关的操作，目前仅提供合约账户创建的功能。合约账户是管理合约的单位，即合约必须安装在合约账户上。

### a. 初始化合约账户客户端

构造一个可以操作合约账户的客户端，调用其他合约账户相关的函数必须通过该客户端。

#### 1.方法名称

- func InitContractAccount(account *account.Account, node, bcname string) *ContractAccount

#### 2.参数

| 参数名  | 含义             | 类型             |
| ------- | ---------------- | ---------------- |
| account | 用户区块链账户   | *account.Account |
| node    | 超级链网络的地址 | string           |
| bcname  | 链名称           | string           |

#### 3.返回

| 类型             | 含义               |
| ---------------- | ------------------ |
| *ContractAccount | 合约账户操作客户端 |

### b. 创建合约账户

直接创建一个合约账户,并返回创建合约账户的交易id。在创建合约账户的过程中，需要支付创建合约账户的gas，gas会根据执行的难度收取，如果想获取gas具体数值需要通过方法c预执行和d执行来创建合约账户。

#### 1.方法名称

- func (ca *ContractAccount) CreateContractAccount(contractAccount string) (string, error)

#### 2.参数

| 参数名          | 含义                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| contractAccount | 合约账户名字, 规则为 XC+16个数字+@+链名，例如: "XC1234567890123456@xuper" | string |

#### 3.返回

| 类型   | 含义                       |
| ------ | -------------------------- |
| string | 创建合约账号的交易ID       |
| error  | nil: 创建成功non-nil: 失败 |

### c. 预执行创建合约账户

预执行创建合约账户，在预执行的结果中可以获取创建合约账户需要花费的gas费用。

#### 1.方法名称

- func (ca *ContractAccount) PreCreateContractAccount(contractAccount string) (*pb.PreExecWithSelectUTXOResponse, error)

#### 2.参数

| 参数名          | 含义                                                         | 类型   |
| --------------- | ------------------------------------------------------------ | ------ |
| contractAccount | 合约账户名字, 规则为 XC+16个数字+@+链名，例如: "XC1234567890123456@xuper" | string |

#### 3.返回

| 类型                              | 含义                       |
| --------------------------------- | -------------------------- |
| *pb.PreExecWithSelectUTXOResponse | 预执行返回的结果           |
| error                             | nil: 创建成功non-nil: 失败 |

### d. 通过预执行结果创建合约账户

通过预执行的结果来完成创建合约账户。

#### 1.方法名称

- func (ca *ContractAccount) PostCreateContractAccount(preExeResp *pb.PreExecWithSelectUTXOResponse) (string, error)

#### 2.参数

| 参数名     | 含义       | 类型                              |
| ---------- | ---------- | --------------------------------- |
| preExeResp | 预执行结果 | *pb.PreExecWithSelectUTXOResponse |

#### 3.返回

| 类型   | 含义                       |
| ------ | -------------------------- |
| string | 创建合约账号的交易ID       |
| error  | nil: 创建成功non-nil: 失败 |

## 4.3 转账

普通转账交易，可以完成区块链上价值的转移。

### a. 初始化交易客户端

构造一个可以进行转账交易的客户端，调用交易相关的其他方法必须通过该客户端。

#### 1.方法名称

- func InitTrans(account *account.Account, node, bcname string) *Trans

#### 2.参数

| 参数名  | 含义             | 类型             |
| ------- | ---------------- | ---------------- |
| account | 用户区块链账户   | *account.Account |
| node    | 超级链网络的地址 | string           |
| bcname  | 链名称           | string           |

#### 3.返回

| 类型   | 含义       |
| ------ | ---------- |
| *Trans | 交易客户端 |

### b. 转账交易

直接进行转账交易

#### 1.方法名称

- func (t *Trans) Transfer(to, amount, fee, desc string) (string, error)

#### 2.参数

| 参数名 | 含义                 | 类型   |
| ------ | -------------------- | ------ |
| to     | 转账目的地址         | string |
| amount | 转账金额             | string |
| fee    | 转账可选支付的手续费 | string |
| desc   | 交易描述信息         | string |

#### 3.返回

| 类型   | 含义                       |
| ------ | -------------------------- |
| string | 生成的转账交易ID           |
| error  | nil: 创建成功non-nil: 失败 |

### c. 查询交易详情

根据交易id来查询交易的详情，该交易可以为普通转账交易、合约创建交易、合约调用交易等

#### 1.方法名称

- func (t *Trans) QueryTx(txid string) (*pb.TxStatus, error)

#### 2.参数

| 参数名 | 含义           | 类型   |
| ------ | -------------- | ------ |
| txid   | 待查询的交易ID | string |

#### 3.返回

| 类型         | 含义                       |
| ------------ | -------------------------- |
| *pb.TxStatus | 得到交易的详细信息和状态   |
| error        | nil: 创建成功non-nil: 失败 |

### d. 查询用户账户余额

查询本区块链账户的链上余额

#### 1.方法名称

- func (t *Trans) GetBalance() (string, error)

#### 2.参数

无

#### 3.返回

| 类型   | 含义                             |
| ------ | -------------------------------- |
| string | 得到所在账户的余额，和冻结的金额 |
| error  | nil: 创建成功non-nil: 失败       |

## 4.4 智能合约

智能合约相关的操作，包括智能合约的部署、调用、查询等。合约名为区分一条链上合约的唯一标识。

### a. 初始化智能合约客户端

构造一个可以操作智能合约的客户端,调用智能合约的相关操作时必须通过该客户端。

#### 1.方法名称

- func InitWasmContract(account *account.Account, node, bcName, contractName, contractAccount string) *WasmContract

#### 2.参数

| 参数名          | 含义             | 类型             |
| --------------- | ---------------- | ---------------- |
| account         | 区块链账户       | *account.Account |
| node            | 超级链网络的地址 | string           |
| bcName          | 链名称           | string           |
| contractName    | 要操作的合约名称 | string           |
| contractAccount | 合约账号         | string           |

#### 3.返回

| 类型          | 含义           |
| ------------- | -------------- |
| *WasmContract | 合约操作客户端 |

### b. 部署安装合约

直接部署合约，并返回部署合约的交易id。也可以通过操作c预执行+操作d执行来完成相同的操作。

#### 1.方法名称

- func (c *WasmContract) DeployWasmContract(args map[string]string, codepath string, runtime string) (string, error)

#### 2.参数

| 参数名   | 含义                 | 类型              |
| -------- | -------------------- | ----------------- |
| args     | 合约部署参数         | map[string]string |
| codepath | 合约文件位置         | string            |
| runtime  | 合约语言类型 c or go | string            |

#### 3.返回

| 类型   | 含义                       |
| ------ | -------------------------- |
| string | 创建合约账号的交易ID       |
| error  | nil: 创建成功non-nil: 失败 |

### c. 预执行部署安装合约

预执行部署安装合约，通过预执行结果可以获取安装合约需要支付的gas费用。

#### 1.方法名称

- func (c *WasmContract) PreDeployWasmContract(arg map[string]string, codepath string, runtime string) (*pb.PreExecWithSelectUTXOResponse, error)

#### 2.参数

| 参数名   | 含义                 | 类型              |
| -------- | -------------------- | ----------------- |
| args     | 合约部署参数         | map[string]string |
| codepath | 合约文件位置         | string            |
| runtime  | 合约语言类型 c or go | string            |

#### 3.返回

| 类型                              | 含义                   |
| --------------------------------- | ---------------------- |
| *pb.PreExecWithSelectUTXOResponse | 预执行结果             |
| error                             | nil: 成功non-nil: 失败 |

### d. 根据预执行结果部署安装合约

根据预执行的结果继续完成合约的安装部署

#### 1.方法名称

- func (c *WasmContract) PostWasmContract(preExeWithSelRes *pb.PreExecWithSelectUTXOResponse) (string, error)

#### 2.参数

| 参数名           | 含义             | 类型                              |
| ---------------- | ---------------- | --------------------------------- |
| preExeWithSelRes | 预执行返回的结果 | *pb.PreExecWithSelectUTXOResponse |

#### 3.返回

| 类型   | 含义                     |
| ------ | ------------------------ |
| string | 安装合约的交易ID         |
| error  | nil：成功；non-nil：失败 |

### e. 调用合约

调用智能合约，并返回调用智能合约交易的交易id。该操作也可以通过操作f预执行+操作g执行来完成。

#### 1.方法名称

- func (c *WasmContract) InvokeWasmContract(methodName string, args map[string]string) (string, error)

#### 2.参数

| 参数名     | 含义         | 类型              |
| ---------- | ------------ | ----------------- |
| methodName | 合约方法     | string            |
| args       | 合约方法参数 | map[string]string |

#### 3.返回

| 类型   | 含义                       |
| ------ | -------------------------- |
| string | 调用合约的交易ID           |
| error  | nil: 创建成功non-nil: 失败 |

### f. 预执行调用合约

预执行调用合约，通过该函数返回值可以获取执行合约需要的gas费用

#### 1.方法名称

- func (c *WasmContract) PreInvokeWasmContract(methodName string, args map[string]string) (*pb.PreExecWithSelectUTXOResponse, error)

#### 2.参数

| 参数名     | 含义         | 类型              |
| ---------- | ------------ | ----------------- |
| methodName | 合约方法     | string            |
| args       | 合约部署参数 | map[string]string |

#### 3.返回

| 类型                              | 含义                       |
| --------------------------------- | -------------------------- |
| *pb.PreExecWithSelectUTXOResponse | 预执行结果                 |
| error                             | nil: 创建成功non-nil: 失败 |

### g. 根据预执行结果调用合约

通过预执行的结果继续完成合约的调用。

#### 1.方法名称

- func (c *WasmContract) PostWasmContract(preExeWithSelRes *pb.PreExecWithSelectUTXOResponse) (string, error)

#### 2.参数

| 参数名           | 含义       | 类型                              |
| ---------------- | ---------- | --------------------------------- |
| preExeWithSelRes | 预执行结果 | *pb.PreExecWithSelectUTXOResponse |

#### 3.返回

| 类型   | 含义                       |
| ------ | -------------------------- |
| string | 调用合约的交易ID           |
| error  | nil: 创建成功non-nil: 失败 |

### h. 查询合约

调用合约查询方法，查询方法仅访问合约维护的数据，并不能将操作上链。

#### 1.方法名称

- func (c *WasmContract) QueryWasmContract(methodName string, args map[string]string) (*pb.InvokeRPCResponse, error)

#### 2.参数

| 参数名     | 含义         | 类型              |
| ---------- | ------------ | ----------------- |
| methodName | 合约方法     | string            |
| args       | 合约方法参数 | map[string]string |

#### 3.返回

| 类型                              | 含义                             |
| --------------------------------- | -------------------------------- |
| *pb.PreExecWithSelectUTXOResponse | 预执行结果，解析response得到结果 |
| error                             | nil: 创建成功non-nil: 失败       |

# 5 词汇表

| 类型       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| 区块链账户 | 普通用户，有自己的addres，公私钥                             |
| 合约账户   | 合约账户名字, 规则为 XC+16个数字+@+链名，例如: “XC1234567890123456@xuper” |
| 交易       | 普通转账                                                     |
| 智能合约   | 用户实现的分布式应用                                         |