# 签名分析

bitcoind/src/script/sign.cpp  - &gt; ProduceSignature



**签名调用代码**

```
ProduceSignature(MutableTransactionSignatureCreator(
                                 &keystore, &mergedTx, i, amount, nHashType),
                             prevPubKey, sigdata);
```

参数 1：创建一个签名对象 Creator 

