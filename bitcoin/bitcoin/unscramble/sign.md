# 签名分析

bitcoind/src/script/sign.cpp  - &gt; ProduceSignature

## **签名调用代码**

```
ProduceSignature(MutableTransactionSignatureCreator(
                                 &keystore, &mergedTx, i, amount, nHashType),
                             prevPubKey, sigdata);
```

参数 1：创建一个签名对象 Creator \(MutableTransactionSignatureCreator  - &gt;  TransactionSignatureCreator - &gt; BaseSignatureCreator\)

参数 2：交易公钥脚本

参数 3：签名数据 （ out ）



### ProduceSignature 函数内部



制作签名前缀数据

```
SignStep(creator, script, result, whichType, SIGVERSION_BASE);
```

参数 1：签名对象

参数 2：签名公钥脚本

参数 3：签名数据前缀 （ out ）

参数 4：描述签名数据前缀的返回类型 （ out ）

参数 5：签名方式 SIGVERSION\_BASE （固定参数：SIGVERSION\_BASE）







### SignSetp 函数内部

解析参数

```
Solver(scriptPubKey, whichTypeRet, vSolutions)
```

参数 1：签名公钥脚本

参数 2：数据前缀的返回类型

参数 3：函数的返回值 （ out ）





### Solver 函数内部

1. 设置参数模版

 模版排序顺序：公钥、公钥哈希、多重签名

```
static std::multimap<txnouttype, CScript> mTemplates;
if (mTemplates.empty())
{
    // Standard tx, sender provides pubkey, receiver adds signature
    mTemplates.insert(std::make_pair(TX_PUBKEY, CScript() << OP_PUBKEY << OP_CHECKSIG));

    // Bitcoin address tx, sender provides hash of pubkey, receiver provides signature and pubkey
    mTemplates.insert(std::make_pair(TX_PUBKEYHASH, CScript() << OP_DUP << OP_HASH160 
                                                << OP_PUBKEYHASH << OP_EQUALVERIFY << OP_CHECKSIG));

    // Sender provides N pubkeys, receivers provides M signatures
    mTemplates.insert(std::make_pair(TX_MULTISIG, CScript() << OP_SMALLINTEGER << OP_PUBKEYS 
                                                        << OP_SMALLINTEGER << OP_CHECKMULTISIG));
}
```

2. 截取公钥 20 字节

```
if (scriptPubKey.IsPayToScriptHash())
{
    typeRet = TX_SCRIPTHASH;
    std::vector<unsigned char> hashBytes(scriptPubKey.begin()+2, scriptPubKey.begin()+22);
    vSolutionsRet.push_back(hashBytes);
    return true;
}
```

3. 隔离验证

```
int witnessversion;
std::vector<unsigned char> witnessprogram;
if (scriptPubKey.IsWitnessProgram(witnessversion, witnessprogram)) {
    if (witnessversion == 0 && witnessprogram.size() == 20) {
        typeRet = TX_WITNESS_V0_KEYHASH;
        vSolutionsRet.push_back(witnessprogram);
        return true;
    }
    if (witnessversion == 0 && witnessprogram.size() == 32) {
        typeRet = TX_WITNESS_V0_SCRIPTHASH;
        vSolutionsRet.push_back(witnessprogram);
        return true;
    }
    if (witnessversion != 0) {
        typeRet = TX_WITNESS_UNKNOWN;
        vSolutionsRet.push_back(std::vector<unsigned char>{(unsigned char)witnessversion});
        vSolutionsRet.push_back(std::move(witnessprogram));
        return true;
    }
    return false;
}
```









