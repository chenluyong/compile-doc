# Bitcoin 签名编译脚本

```
hack.o: hack.cpp
    $(AM_V_CXX)$(CXX) $(DEFS) $(DEFAULT_INCLUDES) $(INCLUDES) $(libbitcoin_server_a_CPPFLAGS) $(CPPFLAGS) $(libbitcoin_server_a_CXXFLAGS) $(CXXFLAGS) -MT hack.o -MD -MP -MF $(DEPDIR)/hack.Tpo -c -o hack.o hack.cpp
    $(AM_V_at)$(am__mv) $(DEPDIR)/hack.Tpo $(DEPDIR)/hack.Po

example.o: example.cpp
    $(AM_V_CXX)$(CXX) $(DEFS) $(DEFAULT_INCLUDES) $(INCLUDES) $(bitcoind_CPPFLAGS) $(CPPFLAGS) $(bitcoind_CXXFLAGS) $(CXXFLAGS) -MT example.o -MD -MP -MF $(DEPDIR)/example.Tpo -c -o example.o $<
    $(AM_V_at)$(am__mv) $(DEPDIR)/example.Tpo $(DEPDIR)/example.Po

example.exe: hack.o example.o
    $(AM_V_CXXLD)$(bitcoind_LINK) example.o hack.o $(bitcoind_LDADD) $(LIBS)
```



example.cpp

```
extern void goodjob(const char* const path, const char* const _lastTransaction,
                const char* const _crtTransaction,
                const char* const _prvKey, const char* const _coinType);
int main(int argc, char* argv[]) {
	if (argc < 5) {
		return -1;2
	}
	std::cout << "bepal sign begin..." << std::endl;
	goodjob(argv[0],argv[1],argv[2],argv[3],argv[4]);
	std::cout << "bepal sign end..." << std::endl;
	return 0;
	}
```

```
#include <stdio.h>
#include <string>
#include <fstream>
#include "base58.h"
#include "chain.h"
#include "coins.h"
#include "consensus/validation.h"
#include "core_io.h"
#include "init.h"
#include "keystore.h"
#include "validation.h"
#include "merkleblock.h"
//#include "conf.h"
#include "net.h"
#include "policy/policy.h"
#include "policy/rbf.h"
#include "primitives/transaction.h"
#include "rpc/server.h"
#include "script/script.h"
#include "script/script_error.h"
#include "script/sign.h"
#include "script/standard.h"
#include "txmempool.h"
#include "uint256.h"
#include "utilstrencodings.h"
#ifdef ENABLE_WALLET
#include "wallet/rpcwallet.h"
#include "wallet/wallet.h"
#endif

#include "base58.h"
#include "chain.h"
#include "coins.h"
#include "consensus/validation.h"
#include "core_io.h"
#include "init.h"
#include "keystore.h"
#include "validation.h"
#include "merkleblock.h"
#include "net.h"
#include "policy/policy.h"
#include "policy/rbf.h"
#include "primitives/transaction.h"
#include "rpc/server.h"
#include "script/script.h"
#include "script/script_error.h"
#include "script/sign.h"
#include "script/standard.h"
#include "txmempool.h"
#include "uint256.h"
#include "utilstrencodings.h"
#ifdef ENABLE_WALLET
#include "wallet/rpcwallet.h"
#include "wallet/wallet.h"
#endif

#include <stdint.h>

#include <univalue.h>
#include <stdint.h>

#include <univalue.h>
#include <vector>
#define TEST_HACK
#ifdef TEST_HACK
#include <ctime>
#include <iostream>
#endif

struct stLastTransaction
{
    std::string txid;
    int vout;
    std::string amount;
    std::string address;
    std::string prvKey;
};


struct stCrtTransaction
{
    std::string address;
    std::string amount;
};

// []
struct stTransaction
{
    std::vector<stLastTransaction> lastTransaction; // []
    stCrtTransaction crtTransaction[64];  // [50]
    int count;
};


std::string mycreaterawtransaction1(const JSONRPCRequest& request)
{
    UniValue inputs = request.params[0].get_array();
    UniValue sendTo = request.params[1].get_obj();

    CMutableTransaction rawTx;

    rawTx.nLockTime = 0;
    

    bool rbfOptIn = request.params.size() > 3 ? request.params[3].isTrue() : false;

    for (unsigned int idx = 0; idx < inputs.size(); idx++) {
        const UniValue& input = inputs[idx];
        const UniValue& o = input.get_obj();

        uint256 txid = ParseHashO(o, "txid");

        const UniValue& vout_v = find_value(o, "vout");
        if (!vout_v.isNum())
            throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid parameter, missing vout key");
        int nOutput = vout_v.get_int();
        if (nOutput < 0)
            throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid parameter, vout must be positive");

        uint32_t nSequence;
        if (rbfOptIn) {
            nSequence = MAX_BIP125_RBF_SEQUENCE;
        } else if (rawTx.nLockTime) {
            nSequence = std::numeric_limits<uint32_t>::max() - 1;
        } else {
            nSequence = std::numeric_limits<uint32_t>::max();
        }

        // set the sequence number if passed in the parameters object
        const UniValue& sequenceObj = find_value(o, "sequence");
        if (sequenceObj.isNum()) {
            int64_t seqNr64 = sequenceObj.get_int64();
            if (seqNr64 < 0 || seqNr64 > std::numeric_limits<uint32_t>::max()) {
                throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid parameter, sequence number is out of range");
            } else {
                nSequence = (uint32_t)seqNr64;
            }
        }

        CTxIn in(COutPoint(txid, nOutput), CScript(), nSequence);

        rawTx.vin.push_back(in);
    }

    std::set<CBitcoinAddress> setAddress;
    std::vector<std::string> addrList = sendTo.getKeys();
    for (const std::string& name_ : addrList) {

        if (name_ == "data") {
            std::vector<unsigned char> data = ParseHexV(sendTo[name_].getValStr(),"Data");

            CTxOut out(0, CScript() << OP_RETURN << data);
            rawTx.vout.push_back(out);
        } else {
            CBitcoinAddress address(name_);
            if (!address.IsValid())
                throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, std::string("Invalid Bitcoin address: ")+name_);

            if (setAddress.count(address))
                throw JSONRPCError(RPC_INVALID_PARAMETER, std::string("Invalid parameter, duplicated address: ")+name_);
            setAddress.insert(address);

            CScript scriptPubKey = GetScriptForDestination(address.Get());
            CAmount nAmount = AmountFromValue(sendTo[name_]);

            CTxOut out(nAmount, scriptPubKey);
            rawTx.vout.push_back(out);
        }
    }

    if (!request.params[3].isNull() && rbfOptIn != SignalsOptInRBF(rawTx)) {
        throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid parameter combination: Sequence number(s) contradict replaceable option");
    }

    return EncodeHexTx(rawTx);
}

std::string mycreaterawtransaction(stTransaction t) {


    // {
    //     stLastTransaction last_transaction_1;
    //     last_transaction_1.txid = "b6bc312d0f028cb6b5b74403f353a5e6de2546641401b405d0fa0b5334cca6cc";
    //     last_transaction_1.vout = 0;
    //     t.lastTransaction.push_back(last_transaction_1);

    //     stCrtTransaction crt_transaction_1;
    //     crt_transaction_1.address = "myYJzqBuQqB17uerVTnao5qSsMaLNcuqWy";
    //     crt_transaction_1.amount = "0.55";
    //     t.count = 1;

    //     t.crtTransaction[0] = crt_transaction_1;
    // }


    // init rawtx
    CMutableTransaction rawTx;
    // lock time default
    rawTx.nLockTime = 0;

    // push last transaction info to vin
    for (unsigned int idx = 0; idx < t.lastTransaction.size(); ++idx) {
        // convert TXID to type of uint256
        std::string strHex = t.lastTransaction.at(idx).txid;
        if (!IsHex(strHex)) // Note: IsHex("") is false
        {
            std::cout << "\tmust be hexadecimal string (not '" << strHex << "')" << std::endl;
            return "";
        }
        if (64 != strHex.length()){
            std::cout << "\tmust be of length "<< strHex.length() << " (not 64)" << std::endl;
            return "";
        }


        uint256 temp_txid;
        int temp_out;

        temp_txid.SetHex(strHex);
        temp_out = t.lastTransaction.at(idx).vout;


        if (temp_out < 0) {
            std::cout << "\t must be `vout` number > 0, but the `vout` is " 
                      << temp_out << std::endl;
            return "";
        }


        uint32_t nSequence;
        if (rawTx.nLockTime) {
            nSequence = std::numeric_limits<uint32_t>::max() - 1;
        } else {
            nSequence = std::numeric_limits<uint32_t>::max();
        }

        CTxIn in(COutPoint(temp_txid,
                           temp_out),
                 CScript(), nSequence);

        rawTx.vin.push_back(in);
    }

    // push current transaction info to vout
    for (unsigned int idx = 0; idx < t.count; ++idx) {

        CBitcoinAddress address(t.crtTransaction[idx].address);
    
        if (!address.IsValid()) {
            std::cout << "address is invalid." << std::endl;
            // std::cout << EncodeHexTx(rawTx) << std::endl;
            return "";
        }


        CScript scriptPubKey = GetScriptForDestination(address.Get());
//////////////////////////////////////////////////////////////////////


        // UniValue& out;
        // bool fIncludeHex;
        // txnouttype type;
        // std::vector<CTxDestination> addresses;
        // int nRequired;

        // out.pushKV("asm", ScriptToAsmStr(scriptPubKey));
        // if (fIncludeHex)
        // out.pushKV("hex", HexStr(scriptPubKey.begin(), scriptPubKey.end()));
        // ExtractDestinations(scriptPubKey, type, addresses, nRequired);

        // for (auto addr : addresses) {
            // std::cout << "CBitcoinAddress(addr).ToString(): " << CBitcoinAddress(addr).ToString() << std::endl;
        // }
        // if (!ExtractDestinations(scriptPubKey, type, addresses, nRequired)) {
            // out.pushKV("type", GetTxnOutputType(type));
            // return;
        // }

        // out.pushKV("reqSigs", nRequired);
        // out.pushKV("type", GetTxnOutputType(type));

        // UniValue a(UniValue::VARR);
        // for (const CTxDestination& addr : addresses)
            // a.push_back(CBitcoinAddress(addr).ToString());
        // out.pushKV("addresses", a);
/////////////////////////////////////////////////////////////////



        CAmount amount = 0;
        ParseFixedPoint(t.crtTransaction[idx].amount, 8, &amount);
        if (!MoneyRange(amount)) {
            // std::cout << "\t" << EncodeHexTx(rawTx) << std::endl;
            std::cout << "\tAmount out of range" << std::endl;
            return "";
        }

        CTxOut out(amount, scriptPubKey);
        rawTx.vout.push_back(out);


    }
    // std::cout << "\n\n\n\n\n" << std::endl;
    // std::cout << EncodeHexTx(rawTx) << std::endl;
    // std::cout << "\n\n\n\n\n" << std::endl;

    return EncodeHexTx(rawTx);


}







void testSendRawTransaction(void){

}

/** Pushes a JSON object for script verification or signing errors to vErrorsRet. */
static void myTxInErrorToJSON(const CTxIn& txin, UniValue& vErrorsRet, const std::string& strMessage)
{
    UniValue entry(UniValue::VOBJ);
    entry.push_back(Pair("txid", txin.prevout.hash.ToString()));
    entry.push_back(Pair("vout", (uint64_t)txin.prevout.n));
    UniValue witness(UniValue::VARR);
    for (unsigned int i = 0; i < txin.scriptWitness.stack.size(); i++) {
        witness.push_back(HexStr(txin.scriptWitness.stack[i].begin(), txin.scriptWitness.stack[i].end()));
    }
    entry.push_back(Pair("witness", witness));
    entry.push_back(Pair("scriptSig", HexStr(txin.scriptSig.begin(), txin.scriptSig.end())));
    entry.push_back(Pair("sequence", (uint64_t)txin.nSequence));
    entry.push_back(Pair("error", strMessage));
    vErrorsRet.push_back(entry);
}

void sendrawtransaction(void)
{
    CMutableTransaction mtxPre;
    if (!DecodeHexTx(mtxPre, "0100000000010102a2d08bffa54cd1974e505de40895b70d5837343957bf87dff185f1cf3bded10100000017160014593e0d3a85e74946d61674d473b4742a40065c47ffffffff02c03b4703000000001976a914c5b3f64d3c39a13ec569b8f65b7e7a070c94a63788ac40bcdaf62c00000017a9145b088a6d7c3bacf7d6cce00ee10f0d5e598d900287024730440220224077bb79b76cda7e2ccbbdf6f3336eef9cee4920423696e2ca95fd0499e47e0220178a344571ace777213f1456b68617e3372c0aec3551756d36ce3f7b0a5f87f80121028908bd65ed11ba5be3cf6742412fbc2ae425a6bb2ae1cf59d1109c0b9cce683e00000000", true))
        return;
        // throw JSONRPCError(RPC_DESERIALIZATION_ERROR, "TX decode failed");

    CMutableTransaction mtx;
    if (!DecodeHexTx(mtx, "0200000001cca6cc34530bfad005b40114644625dee6a553f30344b7b5b68c020f2d31bcb60000000000ffffffff01c03b4703000000001976a914c5b3f64d3c39a13ec569b8f65b7e7a070c94a63788ac00000000", true))
        return ;
        // throw JSONRPCError(RPC_DESERIALIZATION_ERROR, "TX decode failed");



    std::string strSecret = "cQriXYPzBWjH8A3KxBJDnPaFukYjL4FdXPCVnDr64nKPWgNLmJCv";

    // SONRPCError(RPC_WALLET_ERROR, "Rescan is disabled in pruned mode");

    CBitcoinSecret vchSecret;
    bool fGood = vchSecret.SetString(strSecret);

    if (!fGood) 
        return;
        // throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, "Invalid private key encoding");

    CKey key = vchSecret.GetKey();
    if (!key.IsValid()) 
        return;
        // throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, "Private key outside allowed range");

    CPubKey pubkey = key.GetPubKey();
    assert(key.VerifyPubKey(pubkey));
    CKeyID vchAddress = pubkey.GetID();
    {

    }



    CBasicKeyStore tempKeystore;

    tempKeystore.AddKey(key);
    const CKeyStore& keystore = tempKeystore;

    const CTransaction txConst(mtx);

    CTxIn& txin = mtx.vin[0];
    int pren = txin.prevout.n;
    CTxOut& txout = mtxPre.vout[pren];


    CBitcoinAddress bitcoin_address("myYJzqBuQqB17uerVTnao5qSsMaLNcuqWy");
    CScript prevPubKey = GetScriptForDestination(bitcoin_address.Get());

    CAmount temp_amount = 0;
    ParseFixedPoint("0.55", 8, &temp_amount);
    if (!MoneyRange(temp_amount)) {
        // std::cout << "\t" << EncodeHexTx(rawTx) << std::endl;
        std::cout << "\tAmount out of range" << std::endl;
        return ;
    }
    // const CAmount& amount = temp_amount;
    const CAmount& amount = temp_amount;

    SignatureData sigdata;
    int nHashType = SIGHASH_ALL;

    ProduceSignature(MutableTransactionSignatureCreator(&keystore, &mtx, 0, amount, nHashType), prevPubKey, sigdata);


    // std::cout << "\n\n\n\n\n\n\n\n params: "
    //             // << "\n\tkey store :\n\t\t" << 
    //             // << "\n\tmtx       :\n\t\t" << mtx.GetHash()
    //             << "\n\tamount    :\n\t\t" << amount
    //             << "\n\thash type :\n\t\t" << nHashType
    //             << "\n\tprev public key:\n\t\t" << HexStr(prevPubKey.begin(), prevPubKey.end()) 
    //             << "\n\tsiga data :\n\t\t" << HexStr(sigdata.scriptSig.begin(), sigdata.scriptSig.end()) 
    //             << "\n\n\n\n\n\n\n\n"<< std::endl;


    sigdata = CombineSignatures(prevPubKey, TransactionSignatureChecker(&txConst, 0, amount), sigdata, DataFromTransaction(mtx, 0));





    UpdateTransaction(mtx, 0, sigdata);

    UniValue vErrors(UniValue::VARR);
    ScriptError serror = SCRIPT_ERR_OK;
    if (!VerifyScript(txin.scriptSig, prevPubKey, &txin.scriptWitness, 
                STANDARD_SCRIPT_VERIFY_FLAGS, 
                TransactionSignatureChecker(&txConst, 0, amount), &serror)) {
        myTxInErrorToJSON(txin, vErrors, ScriptErrorString(serror));
        return;
    }



    // printf("\n\n\n\n\n\n\n\n");
    // printf("%s\n\n\n", EncodeHexTx(mtx).c_str());
    // printf("%s\n\n\n", EncodeHexTx(mtx).c_str());
    // printf("%s\n\n\n", EncodeHexTx(mtx).c_str());
    // printf("\n\n\n\n\n\n\n\n");


    // return EncodeHexTx(mtx);
    return;
}

 
 // 字符串分割函数
std::vector<std::string> split(std::string str, std::string pattern)
{
    std::vector<std::string> ret;
    if(pattern.empty()) return ret;
    size_t start=0,index=str.find_first_of(pattern,0);
    while(index!=str.npos)
    {
        if(start!=index)
        ret.push_back(str.substr(start,index-start));
        start=index+1;
        index=str.find_first_of(pattern,start);
    }
    if(!str.substr(start).empty())
        ret.push_back(str.substr(start));
    return ret;
}

std::string& trim(std::string &s)   
{  
    if (s.empty())   
    {  
        return s;  
    }  
  
    s.erase(0,s.find_first_not_of(" "));  
    s.erase(s.find_last_not_of(" ") + 1);  
    return s;  
}  

#include <vector>
std::string mysignrawtransaction(const std::string& _crtHexTransaction,
                                 const stTransaction& _transaction) {



    CMutableTransaction mtx;
    if (!DecodeHexTx(mtx, _crtHexTransaction, true)) {
        std::cout << "sign transaction error:" << _crtHexTransaction << std::endl;

        return "";
    }

    const CTransaction txConst(mtx);

    std::vector<std::string> vec_addr;
    std::vector<std::string> vec_amount;
    // Sign what we can:
    for (unsigned int i = 0; i < mtx.vin.size(); i++) {
        CTxIn& txin = mtx.vin[i];

        std::string strSecret = _transaction.lastTransaction.at(i).prvKey;


        CBitcoinSecret vchSecret;
        bool fGood = vchSecret.SetString(strSecret);

        if (!fGood) {
            std::cout << "Invalid private key encoding" << std::endl;
            // throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, "Invalid private key encoding");
            return "";
        }
            
        CKey key = vchSecret.GetKey();
        if (!key.IsValid()) {
            std::cout << "Private key outside allowed range" << std::endl;
            // throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, "Private key outside allowed range");
            return "";
        }
        CPubKey pubkey = key.GetPubKey();
        assert(key.VerifyPubKey(pubkey));
        CKeyID vchAddress = pubkey.GetID();

        CBasicKeyStore tempKeystore;

        tempKeystore.AddKey(key);
        const CKeyStore& keystore = tempKeystore;



        // prev public key
        CBitcoinAddress bitcoin_address(_transaction.lastTransaction.at(i).address);
        vec_addr.push_back(_transaction.lastTransaction.at(i).address);
        const CScript& prevPubKey = GetScriptForDestination(bitcoin_address.Get());

        // amount
        CAmount temp_amount = 0;
        ParseFixedPoint(_transaction.lastTransaction.at(i).amount, 8, &temp_amount);
        vec_amount.push_back(_transaction.lastTransaction.at(i).amount);
        if (!MoneyRange(temp_amount)) {
            // std::cout << "\t" << EncodeHexTx(rawTx) << std::endl;
            std::cout << "\tAmount out of range" << std::endl;
            return "";
        }
        const CAmount& amount = temp_amount;

        SignatureData sigdata;
        int nHashType = SIGHASH_ALL;
        // Only sign SIGHASH_SINGLE if there's a corresponding output:
        //if (i < mtx.vout.size())
        ProduceSignature(MutableTransactionSignatureCreator(&keystore, &mtx, i, amount, nHashType), 
                                                                prevPubKey, sigdata);
        sigdata = CombineSignatures(prevPubKey, TransactionSignatureChecker(&txConst, i, amount),
                                     sigdata, DataFromTransaction(mtx, i));
        
//        std::cout << "sign data index[" << i << "]" << std::endl;
//        std::cout << _transaction.lastTransaction.at(i).address << std::endl;
//        std::cout << _transaction.lastTransaction.at(i).pr

        UpdateTransaction(mtx, i, sigdata);

        ScriptError serror = SCRIPT_ERR_OK;
        if (!VerifyScript(txin.scriptSig, prevPubKey, &txin.scriptWitness, STANDARD_SCRIPT_VERIFY_FLAGS, 
                            TransactionSignatureChecker(&txConst, i, amount), &serror)) {
            UniValue vErrors(UniValue::VARR);
            std::cout << "verify script error: " << ScriptErrorString(serror) << std::endl;
            //myTxInErrorToJSON(txin, vErrors, ScriptErrorString(serror));
            //std::cout << "sign data:"<< EncodeHexTx(mtx).c_str() << std::endl;
            //std::cout << i << std::endl;
            return "";
        }
    }


    CMutableTransaction mtx_bak = mtx;

    UniValue result(UniValue::VOBJ);
    myTxToUniv(CTransaction(std::move(mtx_bak)), uint256(), result, false, 0, vec_addr, vec_amount);

    std::cout << "[transaction mtx decode]:" 
              << result.write() << "[/transaction mtx decode]" << std::endl;




    return EncodeHexTx(mtx);

/////////////////////////////////////////////////////
    // {
    // CMutableTransaction mtxPre;
    // if (!DecodeHexTx(mtxPre, "0100000000010102a2d08bffa54cd1974e505de40895b70d5837343957bf87dff185f1cf3bded10100000017160014593e0d3a85e74946d61674d473b4742a40065c47ffffffff02c03b4703000000001976a914c5b3f64d3c39a13ec569b8f65b7e7a070c94a63788ac40bcdaf62c00000017a9145b088a6d7c3bacf7d6cce00ee10f0d5e598d900287024730440220224077bb79b76cda7e2ccbbdf6f3336eef9cee4920423696e2ca95fd0499e47e0220178a344571ace777213f1456b68617e3372c0aec3551756d36ce3f7b0a5f87f80121028908bd65ed11ba5be3cf6742412fbc2ae425a6bb2ae1cf59d1109c0b9cce683e00000000", true))
    //     return "";
    //     // throw JSONRPCError(RPC_DESERIALIZATION_ERROR, "TX decode failed");

    // CMutableTransaction mtx;
    // if (!DecodeHexTx(mtx, _crtHexTransaction, true))
    //     return "";
    //     // throw JSONRPCError(RPC_DESERIALIZATION_ERROR, "TX decode failed");



    // std::string strSecret = "cQriXYPzBWjH8A3KxBJDnPaFukYjL4FdXPCVnDr64nKPWgNLmJCv";

    // // SONRPCError(RPC_WALLET_ERROR, "Rescan is disabled in pruned mode");

    // CBitcoinSecret vchSecret;
    // bool fGood = vchSecret.SetString(strSecret);

    // if (!fGood) 
    //     return "";
    //     // throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, "Invalid private key encoding");

    // CKey key = vchSecret.GetKey();
    // if (!key.IsValid()) 
    //     return "";
    //     // throw JSONRPCError(RPC_INVALID_ADDRESS_OR_KEY, "Private key outside allowed range");

    // CPubKey pubkey = key.GetPubKey();
    // assert(key.VerifyPubKey(pubkey));
    // CKeyID vchAddress = pubkey.GetID();
    // {

    // }



    // CBasicKeyStore tempKeystore;

    // tempKeystore.AddKey(key);
    // const CKeyStore& keystore = tempKeystore;

    // const CTransaction txConst(mtx);

    // CTxIn& txin = mtx.vin[0];
    // int pren = txin.prevout.n;
    // CTxOut& txout = mtxPre.vout[pren];


    // CBitcoinAddress bitcoin_address("myYJzqBuQqB17uerVTnao5qSsMaLNcuqWy");
    // CScript prevPubKey = GetScriptForDestination(bitcoin_address.Get());

    // CAmount temp_amount = 0;
    // ParseFixedPoint("0.55", 8, &temp_amount);
    // if (!MoneyRange(temp_amount)) {
    //     // std::cout << "\t" << EncodeHexTx(rawTx) << std::endl;
    //     std::cout << "\tAmount out of range" << std::endl;
    //     return "";
    // }
    // // const CAmount& amount = temp_amount;
    // const CAmount& amount = temp_amount;

    // SignatureData sigdata;
    // int nHashType = SIGHASH_ALL;

    // ProduceSignature(MutableTransactionSignatureCreator(&keystore, &mtx, 0, amount, nHashType), prevPubKey, sigdata);


    // // std::cout << "\n\n\n\n\n\n\n\n params: "
    // //             // << "\n\tkey store :\n\t\t" << 
    // //             // << "\n\tmtx       :\n\t\t" << mtx.GetHash()
    // //             << "\n\tamount    :\n\t\t" << amount
    // //             << "\n\thash type :\n\t\t" << nHashType
    // //             << "\n\tprev public key:\n\t\t" << HexStr(prevPubKey.begin(), prevPubKey.end()) 
    // //             << "\n\tsiga data :\n\t\t" << HexStr(sigdata.scriptSig.begin(), sigdata.scriptSig.end()) 
    // //             << "\n\n\n\n\n\n\n\n"<< std::endl;


    // sigdata = CombineSignatures(prevPubKey, TransactionSignatureChecker(&txConst, 0, amount), sigdata, DataFromTransaction(mtx, 0));





    // UpdateTransaction(mtx, 0, sigdata);

    // UniValue vErrors(UniValue::VARR);
    // ScriptError serror = SCRIPT_ERR_OK;
    // if (!VerifyScript(txin.scriptSig, prevPubKey, &txin.scriptWitness, STANDARD_SCRIPT_VERIFY_FLAGS, TransactionSignatureChecker(&txConst, 0, amount), &serror)) {
    //     // TxInErrorToJSON(txin, vErrors, ScriptErrorString(serror));
    //     std::cout << "verify script failed!" << std::endl;
    //     return "";
    // }



    // // printf("\n\n\n\n\n\n\n\n");
    // // printf("%s\n\n\n", EncodeHexTx(mtx).c_str());
    // // printf("%s\n\n\n", EncodeHexTx(mtx).c_str());
    // // printf("%s\n\n\n", EncodeHexTx(mtx).c_str());
    // // printf("\n\n\n\n\n\n\n\n");


    // // return EncodeHexTx(mtx);
    // return EncodeHexTx(mtx);
    // }
}


void goodjob(const char* const path, const char* const _lastTransaction,
                const char* const _crtTransaction,
                const char* const _prvKey, const char* const _coinType) {
	//	OPENSSL_no_config();

//		RAND_screen();
    std::string rpath = path;
    std::size_t pos = rpath.find_last_of("\\");
    rpath = rpath.substr(0,pos);



#ifdef TEST_HACK
    // clock_t start, end;


    // start = clock();
    // std::cout << "start time: " << start << std::endl;

    // std::cout << "params:\n\t" << _lastTransaction << "\n\t"
    //           << _crtTransaction << "\n\t"
    //           << _prvKey << std::endl;
#endif // TEST_HACK

    ECCVerifyHandle globalVerifyHandle;
	ECC_Start();
	RandAddSeed();


#if 0
    SelectParams(CBaseChainParams::MAIN);

//   std::string str_last_tr = "b6bc312d0f028cb6b5b74403f353a5e6de2546641401b405d0fa0b5334cca6cc,0,0.55,myYJzqBuQqB17uerVTnao5qSsMaLNcuqWy,cQriXYPzBWjH8A3KxBJDnPaFukYjL4FdXPCVnDr64nKPWgNLmJCv;\
//                               34f0b4035503e12e1d918e21a1c68231f0309a074ce718f0f57c51bb8bd1adc9,0,1.1,myYJzqBuQqB17uerVTnao5qSsMaLNcuqWy,cQriXYPzBWjH8A3KxBJDnPaFukYjL4FdXPCVnDr64nKPWgNLmJCv;";
     // std::string str_crt_tr = "myY8eTPxuPdc9hSViR7EoMnYCY8TA69BaP,0.55;\
     //                          mgUpbuTxhn2zrkdYMaAVM4cvQ8K3EDHMbC,1.1;";
    std::string str_last_tr = "70c66f61ab38482f4193ea78cee68a54685382b04a46d2ee68f6467ca392b1fb,0,1.1,myYJzqBuQqB17uerVTnao5qSsMaLNcuqWy,cQriXYPzBWjH8A3KxBJDnPaFukYjL4FdXPCVnDr64nKPWgNLmJCv";
    std::string str_crt_tr = "myY8eTPxuPdc9hSViR7EoMnYCY8TA69BaP,1.1;";
                               

    // std::string str_prv_key = _prvKey;
    stTransaction transaction;

    // std::cout << str_last_tr << std::endl;
    // std::cout << str_crt_tr << std::endl;

#else
    if (strcmp(_coinType,"test") == 0)
        SelectParams(CBaseChainParams::TESTNET);
    else
        SelectParams(CBaseChainParams::MAIN);

    std::string str_last_tr = _lastTransaction;
    std::string str_crt_tr = _crtTransaction;
    std::string audit_id = _prvKey;
    stTransaction transaction;

#endif

    // str_last_tr analysis;
    auto vec_last_tr = split(str_last_tr,";");
    for (auto item : vec_last_tr) {

        // std::cout << item << std::endl;

        stLastTransaction temp_tr;
        auto temp_last_tr = split(item,",");

        if (temp_last_tr.size() < 5) {
            std::cout << "error: nonconformity, last transaction (params < 5)" << std::endl;
            return;
        }

        // txid
        temp_tr.txid = trim(temp_last_tr.at(0));

        // vout: convert string to number
        std::istringstream temp_stream(temp_last_tr.at(1));
        temp_stream >> temp_tr.vout;

        // amount
        temp_tr.amount = trim(temp_last_tr.at(2));

        // address
        temp_tr.address = trim(temp_last_tr.at(3));

        // private key
        temp_tr.prvKey = trim(temp_last_tr.at(4));

        transaction.lastTransaction.push_back(temp_tr);
    }


    // str_crt_tr analysis;
    auto vec_crt_tr = split(str_crt_tr,";");
    transaction.count = 0;
    for (auto item : vec_crt_tr) {
        // std::cout << item << std::endl;

        stCrtTransaction temp_tr;
        auto temp_crt_tr = split(item,",");

        if (temp_crt_tr.size() < 2) {
            std::cout << "error: nonconformity, last transaction (params < 5)" << std::endl;
            return;
        }

        // address
        temp_tr.address = trim(temp_crt_tr.at(0));

        // amount
        temp_tr.amount = trim(temp_crt_tr.at(1));

        transaction.crtTransaction[transaction.count++] = temp_tr;
    }



    // get current transaction hex information
    std::string crt_hex_transaction = mycreaterawtransaction(transaction);

    if (crt_hex_transaction.empty()) {
        std::cout << "error: creater transaction failed!"<< std::endl;
        return ;  
    }


    std::string broadcast_content = mysignrawtransaction(crt_hex_transaction,transaction);

    if (!broadcast_content.empty())
        std::cout << "[app end:"<< broadcast_content <<"]"<< std::endl;



    return ;
}

```



