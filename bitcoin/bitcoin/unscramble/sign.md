# 签名分析

bitcoind/src/script/sign.cpp  - &gt; ProduceSignature

**签名调用代码**

```
ProduceSignature(MutableTransactionSignatureCreator(
                                 &keystore, &mergedTx, i, amount, nHashType),
                             prevPubKey, sigdata);
```

参数 1：创建一个签名对象 Creator \(MutableTransactionSignatureCreator  - &gt;  TransactionSignatureCreator - &gt; BaseSignatureCreator\)

参数 2：交易公钥脚本

参数 3：签名数据 （ out ）



