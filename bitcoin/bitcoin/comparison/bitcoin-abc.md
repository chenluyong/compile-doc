# Bitcoin-abc 与 Bitcoin 差别

\(注：C++源码分析，逻辑相同\)

# 签名部分差别

比特币默认选择 SIGVERSION\_BASE = 0 作为参数

```
enum SigVersion  // 该参数决定了签名的方式
{
    SIGVERSION_BASE = 0,      // 默认签名方式
    SIGVERSION_WITNESS_V0 = 1,// 隔离验证签名
};
```

比特币 0.16 源码1

bitcoin\src\script\interpreter.cpp

```
uint256 SignatureHash(const CScript& scriptCode, const CTransaction& txTo,
                      unsigned int nIn, int nHashType,
                      const CAmount& amount,
                      SigVersion sigversion, const PrecomputedTransactionData* cache)
{
    assert(nIn < txTo.vin.size());
    // [1] 验证是否属于隔离验证
    if (sigversion == SIGVERSION_WITNESS_V0) {
        uint256 hashPrevouts;
        uint256 hashSequence;
        uint256 hashOutputs;
        const bool cacheready = cache && cache->ready;

        if (!(nHashType & SIGHASH_ANYONECANPAY)) {
            hashPrevouts = cacheready ? cache->hashPrevouts : GetPrevoutHash(txTo);
        }

        if (!(nHashType & SIGHASH_ANYONECANPAY) && (nHashType & 0x1f) != SIGHASH_SINGLE &&
             (nHashType & 0x1f) != SIGHASH_NONE) {
            hashSequence = cacheready ? cache->hashSequence : GetSequenceHash(txTo);
        }


        if ((nHashType & 0x1f) != SIGHASH_SINGLE && (nHashType & 0x1f) != SIGHASH_NONE) {
            hashOutputs = cacheready ? cache->hashOutputs : GetOutputsHash(txTo);
        } else if ((nHashType & 0x1f) == SIGHASH_SINGLE && nIn < txTo.vout.size()) {
            CHashWriter ss(SER_GETHASH, 0);
            ss << txTo.vout[nIn];
            hashOutputs = ss.GetHash();
        }

        CHashWriter ss(SER_GETHASH, 0);
        // Version
        ss << txTo.nVersion;
        // Input prevouts/nSequence (none/all, depending on flags)
        ss << hashPrevouts;
        ss << hashSequence;
        // The input being signed (replacing the scriptSig with scriptCode + amount)
        // The prevout may already be contained in hashPrevout, and the nSequence
        // may already be contain in hashSequence.
        ss << txTo.vin[nIn].prevout;
        ss << scriptCode;
        ss << amount;
        ss << txTo.vin[nIn].nSequence;
        // Outputs (none/one/all, depending on flags)
        ss << hashOutputs;
        // Locktime
        ss << txTo.nLockTime;
        // Sighash type
        ss << nHashType;

        return ss.GetHash();
    }
    // [2] 开始生成交易哈希
    static const uint256 one(uint256S("0000000000000000000000000000000000000000000000000000000000000001"));

    // Check for invalid use of SIGHASH_SINGLE
    if ((nHashType & 0x1f) == SIGHASH_SINGLE) {
        if (nIn >= txTo.vout.size()) {
            //  nOut out of range
            return one;
        }
    }

    // Wrapper to serialize only the necessary parts of the transaction being signed
    CTransactionSignatureSerializer txTmp(txTo, scriptCode, nIn, nHashType);

    // Serialize and hash
    CHashWriter ss(SER_GETHASH, 0);
    ss << txTmp << nHashType;
    return ss.GetHash();
}
```

nHashType 是比特币现金的主要修改点

```
// 如果是分叉前的币 && 这次属于分叉币交易类型
if (sigHashType.hasForkId() && (flags & SCRIPT_ENABLE_SIGHASH_FORKID)) {
    uint256 hashPrevouts;
    uint256 hashSequence;
    uint256 hashOutputs;

    if (!sigHashType.hasAnyoneCanPay()) {
        hashPrevouts = cache ? cache->hashPrevouts : GetPrevoutHash(txTo);
    }

    if (!sigHashType.hasAnyoneCanPay() &&
        (sigHashType.getBaseSigHashType() != BaseSigHashType::SINGLE) &&
        (sigHashType.getBaseSigHashType() != BaseSigHashType::NONE)) {
        hashSequence = cache ? cache->hashSequence : GetSequenceHash(txTo);
    }

    if ((sigHashType.getBaseSigHashType() != BaseSigHashType::SINGLE) &&
        (sigHashType.getBaseSigHashType() != BaseSigHashType::NONE)) {
        hashOutputs = cache ? cache->hashOutputs : GetOutputsHash(txTo);
    } else if ((sigHashType.getBaseSigHashType() ==
                BaseSigHashType::SINGLE) &&
               (nIn < txTo.vout.size())) {
        CHashWriter ss(SER_GETHASH, 0);
        ss << txTo.vout[nIn];
        hashOutputs = ss.GetHash();
    }

    CHashWriter ss(SER_GETHASH, 0);
    // Version
    ss << txTo.nVersion;
    // Input prevouts/nSequence (none/all, depending on flags)
    ss << hashPrevouts;
    ss << hashSequence;
    // The input being signed (replacing the scriptSig with scriptCode +
    // amount). The prevout may already be contained in hashPrevout, and the
    // nSequence may already be contain in hashSequence.
    ss << txTo.vin[nIn].prevout;
    ss << static_cast<const CScriptBase &>(scriptCode);
    ss << amount.GetSatoshis();
    ss << txTo.vin[nIn].nSequence;
    // Outputs (none/one/all, depending on flags)
    ss << hashOutputs;
    // Locktime
    ss << txTo.nLockTime;
    // Sighash type
    ss << sigHashType;

    return ss.GetHash();
}
```

代码解析

