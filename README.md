# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.6-1), with the goal of providing a reproducible workflow for future use.

**Contents:**

- [Compiling from source](#compile)
- [Generating an ID for use on Verus Testnet](#idgen)
- [Generating a PBaaS chain](#chaingen)
  - [Conceptual description of chosen `definecurrency` options](#chaingen-concept)
    - [Background on CHIPS](#background)
    - [Itemized options for chips10sec chain](#options-chips10sec)
    - [Itemized options for gateway converter](#options-gateway)
  - [Funding identity](#chaingen-fund)
  - [Defining a currency](#chaingen-define)
  - [Verifying currency generation](#chaingen-verify)
- [Launching a PBaaS Chain](#launch)
- [Creating Tokens on PBaaS Chain](#token)

---

<h2 id="compile">Compiling from Source</h2>

<h3>Step 1: Clone, Chain-build with dependencies:</h3>

```
cd ~
git clone https://github.com/VerusCoin/VerusCoin.git verus
cd ~/verus
./zcutil/build.sh -j4
```

Replace `-j4` with your desired threads, if 4 exceeds your system's resources.

If all steps completed successfully, you should have resulting binaries located in `~/verus/src`.

<h3>Step 2: Launch Verus daemon</h3>

```
# Create a new tmux session
tmux new -s vrsctest

# Launch vrsctest chain
cd ~/verus/src
./verusd -chain=vrsctest &

# Detaching from tmux session can be done with Ctrl+B, then D

```

If you wish to mine/stake, you can launch with following options:

```
./verusd -chain=vrsctest -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

Fill in equivalent values for your wallet, desired mining process limits, etc.  When you need to re-attach to your `tmux` session, you can do so with: `tmux a -t vrsctest`.

---

<h2 id="idgen">Generate an ID for use on Verus Testnet</h2>

*You must have at least 100 VRSCTEST in your wallet to generate a primary ID on the Verus testnet*

<h3>Step 1: Register name commitment</h3> 

```
cd ~/verus
./src/verus -chain=vrsctest registernamecommitment chips10sec RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "182fd857eb5205d6484e289aeb7c9295c9bd923d48aad4c4a7cc5f7a5cec0bd9",
  "namereservation": {
    "version": 1,
    "name": "chips10sec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "909de66a7d747ac0929d4133ffe71300efa91103b5ba5fe3e3eeee1091951114",
    "referral": "",
    "nameid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg"
  }
}
```

<h3>Step 2: Register identity</h3>

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./src/verus -chain=vrsctest registeridentity '{
  "txid": "182fd857eb5205d6484e289aeb7c9295c9bd923d48aad4c4a7cc5f7a5cec0bd9",
  "namereservation": {
    "version": 1,
    "name": "chips10sec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "909de66a7d747ac0929d4133ffe71300efa91103b5ba5fe3e3eeee1091951114",
    "referral": "",
    "nameid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg"
  },
    "identity":{
        "name":"chips10sec", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
50203a38cffbb313a290ef5635e8baa9a1e70833d71a7cbaefda8bc708f30c77
```

Wait for this transaction to be confirmed.

<h3>Step 3: Verify identity has been created</h3>

Identities can be located by name, or by i-address:

```
# By name
./src/verus -chain=vrsctest getidentity "chips10sec@"

# By i-address
./src/verus -chain=vrsctest getidentity iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg
```

If successful, output should show similar to the following:

```
{
  "fullyqualifiedname": "chips10sec.VRSCTEST@",
  "identity": {
    "version": 3,
    "flags": 0,
    "primaryaddresses": [
      "RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"
    ],
    "minimumsignatures": 1,
    "name": "chips10sec",
    "identityaddress": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "contentmultimap": {
    },
    "revocationauthority": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "recoveryauthority": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 25932,
  "txid": "50203a38cffbb313a290ef5635e8baa9a1e70833d71a7cbaefda8bc708f30c77",
  "vout": 0
}
```

**Before generating PBaaS chain, repeat Steps 2 & 3 to generate verus ids for all desired notaries in `definecurrency` command.** 

---

<h2 id="chaingen">Generating a PBaaS Chain</h2>

<h3 id="chaingen-concept">Conceptual description of chosen `definecurrency` options</h3>

In [step 2, we will issue a `definecurrency` command to vrsctest daemon](#chaingen-define), which includes selected options tailored to CHIPS unique case and blockchain history.  It is important to understand both why these options were chosen and what they do.

<h4 id="background">Background on CHIPS blockchain</h4>

CHIPS is a coin that has already reached maximum total supply.  In its present form, mining is largely altruistic since block rewards have reduced to zero, and transactions are quite infrequent (miners not being paid by fees).  Price discovery has not occurred, and we want to ensure it takes place using Verus's PBaaS tooling.  We want existing holders of CHIPS to retain their coins while migrating to a new PBaaS chain, and to provide some backing in basket currencies for price discovery via a gateway converter.

Verus's fee market presents a unique opportunity for a coin that has already reached its max supply and will emit no new coins.  Actions taken by users via id generation, cross-chain conversions, and more, will provide greater block rewards to miners.  Combined with merge mining capabilities on Verus, this is a big improvement in theoretical security model for CHIPS.

The options below will generate a `chips10sec` blockchain, as well as a gateway converter `bridge.chips10sec`.

<h4 id="options-chips10sec">`definecurrency` options for `chips10sec` and justifications</h4>

- `"name":"chips10sec"`

Name for our generated chain

- `"options": 264`

Defines our chain as a PBaaS chain, with `OPTION_PBAAS: 0x100 = 256`, and `OPTION_ID_REFERRALS: 8`

- `"currencies":["vrsctest"]`

Defines our chain as being based on existing `vrsctest` currency

- `"maxpreconversion":[0]`

Prevents users of `vrsctest` blockchain from preconverting `vrsctest` to `chips10sec` in during pre-launch period.  If we wanted to accept pre-launch conversions, we could set `minpreconversion` and `maxpreconversion` to specify required preconversion amounts for launch to proceed.

- `"conversions":[1]`

Specifies the desired conversion ratio for `vrsctest` to `chips10sec`

- `"eras"`

Describes reward schedule on new `chips10sec` chain.  We want block subsidies to be zero, so that all emission comes from gateway converter.  Any additional issuance beyond that will be supply-neutral.

- `"notaries":["biz@","BizNotary@","BizNotary1@"]`

These notaries will be responsible for notarizing from `chips10sec` to `vrsctest` and vice versa.  These are required for cross-chain interaction.  Specified notaries here are all owned by Biz's control address.

- `"nodes":[{...},{...}]`

Seed nodes for new PBaaS chain

- `"preallocations":[{"biz@":20950000}]`

Pre-allocated amount from supply of new chain.  In this example, we are pre-allocating 20,950,000 `chips10sec` coins to identity `biz@`.  These can then be re-allocated based on snapshot from legacy CHIPS codebase.

<h4 id="options-gateway">`definecurrency` options for gateway converter</h4>

- `"gatewayconvertername":"bridge"`

Defines a separate blockchain for gateway conversions named `bridge.chips10sec`

- `"gatewayconverterissuance":50000`

Specifies that our final 100,000 `chips10sec` supply will be issued through the gateway converter, for a max supply of 21 million coins.

- `"blocktime":10`

Specified that our target block time for difficulty adjustment should be 10 seconds

- `"currencies":["vrsctest","kmd","btc","chips10sec"]`

Specifies a basket of currencies backing our gateway issuance.

- `"initialcontributions":[1000,2200,0.025,0]`

Specifies amounts of backing currencies to contribute on launch of gateway from our `chips10sec@` identity address.

- `"initialsupply":4000`


<h3 id="chaingen-fund">Step 1: Send VRSCTEST, and Basket currencies to identity address</h3>

To generate a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.

For this chain, we will be generating both a PBaaS chain (`chips10sec@`), and a gatewayconverter chain (`bridge.chips10sec`).  We will be initially contributing a basket of currencies to the gateway converter.  In example below, we are sending `1200 VRSCTEST` to `chips10sec@` for funding of basket.  Afterwards, we'll convert 200 VRSCTEST to KMD & BTC via gateway converters, which will be sent to our `chips10sec@` VerusID as well.

```
# send vrsctest (10200 for registration, 1200 for basket)
./verus -chain=vrsctest sendtoaddress chips10sec@ 11400
0208d818859574ba2d54048629776958382e065a02e7feae7be3b3b04f0dd42b
```

Now, we should convert some `vrsctest` to `kmd` and `btc` for funding of basket:

```
# convert VRSCTEST to KMD & BTC in one go:
./src/verus -chain=vrsctest sendcurrency "*" '[{"amount":100,"via":"VRSC-KMD","convertto":"KMD","address":"chips10sec@"},{"amount":100,"via":"VRSC-BTC","convertto":"BTC","address":"chips10sec@"}]'
opid-1db08a57-9c04-47bc-96be-aaabe860dbef
```

Check operation status:

```
./src/verus -chain=vrsctest z_getoperationresult '["opid-1db08a57-9c04-47bc-96be-aaabe860dbef"]'
[
  {
    "id": "opid-1db08a57-9c04-47bc-96be-aaabe860dbef",
    "status": "success",
    "creation_time": 1683745148,
    "result": {
      "txid": "d2bc9b56383e1fc48af84ef1cb42f29373391eefca7be767d7585301d9d711bb"
    },
    "execution_secs": 0.072621353,
    "method": "sendcurrency",
    "params": [
      {
        "amount": 100,
        "via": "VRSC-KMD",
        "convertto": "KMD",
        "address": "chips10sec@"
      },
      {
        "amount": 100,
        "via": "VRSC-BTC",
        "convertto": "BTC",
        "address": "chips10sec@"
      }
    ]
  }
]
```

After verifying that `"status": "success"` is present in operationresult above, we need to wait for a notarization to occur before the converted KMD & BTC will show in our Identity's wallet currency balance.

Check this with the following command. Once it shows entries for `VRSCTEST`, `KMD`, and `BTC`, notarization has occurred and funds are free for use.

```
./src/verus -chain=vrsctest getcurrencybalance "chips10sec@"
{
  "VRSCTEST": 11400.00000000,
  "KMD": 1437.57296112,
  "BTC": 0.01781147
}
```

<h3 id="chaingen-define">Step 2: Define a currency with desired parameters</h3>

```
./verus -chain=vrsctest definecurrency '{"name":"chips10sec","options":264,"currencies":["vrsctest"],"maxpreconversion":[0], "conversions":[1],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["biz@","biznotary@","biznotary1@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"},{"networkaddress":"149.56.13.160:12121"}],"preallocations":[{"biz@":20950000}], "gatewayconvertername":"bridge", "gatewayconverterissuance":50000, "blocktime":10}' '{"currencies":["vrsctest","kmd","btc","chips10sec"],"initialcontributions":[1200,1430,0.017,0],"initialsupply":4000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


<h3>Step 3: Send resulting raw transaction to network</h3>

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f890496e539c8763ddba98a424708d8dd46bd8f678dcf0658046b62bf1a223fbcad9400000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940912e4cd0fe38de6f25775e698e791009480e9e689fae90ab24e93a768c7ab7c80475a14c8003e88f632635bb00a0c3c8fdb2aa1b7800c93d288dccd67981e310ffffffff2a55058097299e83e083f6a2842da721e484a95e5e3124dea88775ef7123625c00000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc994049c925af85001d58d753b0542ce09b4eb723667face5bd32936b912b1b6342c43cac1668c12a67165adb97351208cdefc5ffa47acd190141db117379322d9530ffffffff2a55058097299e83e083f6a2842da721e484a95e5e3124dea88775ef7123625c01000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc994025a291401a0948f87ed35329ddec565afa106ea5561518372332f358f868e0851a0ca54bc7d75afe9e366a1cf2c0e1985f211e0a6a98c8f04f0a95ffb100a378ffffffffb30baadb9d7f48f1e57fb43c6c420362c16875d7631ebb18d1d108be021093fb01000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940a6ca67a61122ff5d6cca46bfc508c11c41f4f6b9fe201d7c6e30d11bd5da2a210a1da01abaf8b54819972e862d946fc36527045190d2c07c4cebfd762dbfe94affffffff0f0000000000000000fd25014704030001031504ba5270d765535b4afaa44f23ab334fcb31c967da1504ba5270d765535b4afaa44f23ab334fcb31c967da1504ba5270d765535b4afaa44f23ab334fcb31c967dacc4cd904030e01011504ba5270d765535b4afaa44f23ab334fcb31c967da4c8403000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a636869707331307365630000ba5270d765535b4afaa44f23ab334fcb31c967daba5270d765535b4afaa44f23ab334fcb31c967da00a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f01011504ba5270d765535b4afaa44f23ab334fcb31c967da1b04031001011504ba5270d765535b4afaa44f23ab334fcb31c967da750000000000000000fdc3012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d960104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d6c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a63686970733130736563a6ef9ea235635e328124ff3429db9f9e91b64e2dba5270d765535b4afaa44f23ab334fcb31c967da01000000010000000000000000000000000000000000000000000000000081b24a000000000000000000018e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e00f0cd3264710700005039278c04000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f50500000000000100000000000000000100000000000000000100000000000000000000000000038e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e6df4164224d7eb36df23a3fccf2cd6ae7b5827dc4d3213c88cd5b55c4e16c151a34a63f37e52f1ae02a49faec70003f98800c9bfde8f009c8ca4939f00a49faec700bc8340bc8340066272696467650fff001e0a0000002d0000003c0001000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a01000100ba5270d765535b4afaa44f23ab334fcb31c967da01000000ba5270d765535b4afaa44f23ab334fcb31c967da0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fdb9012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d8c0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d62010280030000ba5270d765535b4afaa44f23ab334fcb31c967da01000200ba5270d765535b4afaa44f23ab334fcb31c967da01a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000082dcbd84cf9bff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f505000000000100000000000000000100000000000000000100000000010000000000000000b69900000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000133134392e35362e31332e3136303a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d0000000000000000000000000000000000000000000000000000000000000000ba5270d765535b4afaa44f23ab334fcb31c967daba5270d765535b4afaa44f23ab334fcb31c967da0000ffffffff000000000081b23601a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a740000000000750000000000000000fdac012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d7f0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d55010100000021020000ba5270d765535b4afaa44f23ab334fcb31c967da06627269646765a6ef9ea235635e328124ff3429db9f9e91b64e2dba5270d765535b4afaa44f23ab334fcb31c967da01000000010000000000ba5270d765535b4afaa44f23ab334fcb31c967da81b24a0000a0db215d00000000005039278c04000004a6ef9ea235635e328124ff3429db9f9e91b64e2d3c28dd25a7127ce1b1e9a8dd9fa550a1b805dc1054852c4e9fb1d4c4291fc093e41ce2c7befa4076ba5270d765535b4afaa44f23ab334fcb31c967da0440787d0140787d0140787d0140787d0104000000000000000000000000000000000000000000000000000000000000000000000400e87648170000000098053933000000a02526000000000000000000000000000400e87648170000000098053933000000a025260000000000000000000000000000000000000000a49faec70003f98800750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a01000100ba5270d765535b4afaa44f23ab334fcb31c967da0100000091ccbce2bc71649d2d91bf6384e38d266a1d7fa500000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd7f022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d520204030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d2802028003000091ccbce2bc71649d2d91bf6384e38d266a1d7fa50100030091ccbce2bc71649d2d91bf6384e38d266a1d7fa504a6ef9ea235635e328124ff3429db9f9e91b64e2d3c28dd25a7127ce1b1e9a8dd9fa550a1b805dc1054852c4e9fb1d4c4291fc093e41ce2c7befa4076ba5270d765535b4afaa44f23ab334fcb31c967da0440787d0140787d0140787d0140787d0104000000000000000000000000000000000000000000000000005039278c0400008ad18dedbf00008ad18dedbf00000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000005039278c04000004000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000004a086010000000000a086010000000000a086010000000000a0860100000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000b69900000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d0000000000000000000000000000000000000000000000000000000000000000ba5270d765535b4afaa44f23ab334fcb31c967da91ccbce2bc71649d2d91bf6384e38d266a1d7fa50000ffffffff000000000081b23601a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000075cbc6f44917000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d01a6ef9ea235635e328124ff3429db9f9e91b64e2d81f3ced0f02b05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b20041491ccbce2bc71649d2d91bf6384e38d266a1d7fa591ccbce2bc71649d2d91bf6384e38d266a1d7fa575204e000000000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d013c28dd25a7127ce1b1e9a8dd9fa550a1b805dc1085b2e1b3917905a6ef9ea235635e328124ff3429db9f9e91b64e2d809b20041491ccbce2bc71649d2d91bf6384e38d266a1d7fa591ccbce2bc71649d2d91bf6384e38d266a1d7fa575204e000000000000961a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c77040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5b0154852c4e9fb1d4c4291fc093e41ce2c7befa40768098e64005a6ef9ea235635e328124ff3429db9f9e91b64e2d809b20041491ccbce2bc71649d2d91bf6384e38d266a1d7fa591ccbce2bc71649d2d91bf6384e38d266a1d7fa5750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f00ba5270d765535b4afaa44f23ab334fcb31c967da7500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec700ba5270d765535b4afaa44f23ab334fcb31c967da75f58aad9a040000007a1b04030001011504ba5270d765535b4afaa44f23ab334fcb31c967dacc4c5a04030901011504ba5270d765535b4afaa44f23ab334fcb31c967da3e86fefeff01023c28dd25a7127ce1b1e9a8dd9fa550a1b805dc108706fac70200000054852c4e9fb1d4c4291fc093e41ce2c7befa4076c0450400000000007500000000cb9900000000000000000000000000

# resulting txid
8b0f68665e198294ff848de1772f413a70da65685f57dd0e8fffa75d18c5b147
```

<h3 id="chaingen-verify">Step 4: Verify currency generation was succesful</h3>

```
./verus -chain=vrsctest getcurrency chips10sec
```

If successful, output should show similar to the following:

```
{
  "version": 1,
  "options": 264,
  "name": "chips10sec",
  "currencyid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
  "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "systemid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
  "notarizationprotocol": 1,
  "proofprotocol": 1,
  "launchsystemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "startblock": 39370,
  "endblock": 0,
  "currencies": [
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq"
  ],
  "conversions": [
    1.00000000
  ],
  "maxpreconversion": [
    0.00000000
  ],
  "preallocations": [
    {
      "iGTdav42siHgX96ybWn3pRTFxQgiUpZ9K9": 20950000.00000000
    }
  ],
  "initialcontributions": [
    0.00000000
  ],
  "gatewayconverterissuance": 50000.00000000,
  "idregistrationfees": 100.00000000,
  "idreferrallevels": 3,
  "idimportfees": 0.02000000,
  "notaries": [
    "iGTdav42siHgX96ybWn3pRTFxQgiUpZ9K9",
    "iDVuVSjzYatSYq19YkPJdCH6SuA59WBWvC",
    "iAWhTH5VMq4Ech2zz2ZrASsyRtWyUFb9yw"
  ],
  "minnotariesconfirm": 2,
  "currencyregistrationfee": 200.00000000,
  "pbaassystemregistrationfee": 10000.00000000,
  "currencyimportfee": 100.00000000,
  "transactionimportfee": 0.01000000,
  "transactionexportfee": 0.01000000,
  "gatewayconverterid": "iGmSgKstjRWTnGcjhytDH4pRpTKuWJzC8x",
  "gatewayconvertername": "bridge",
  "initialtarget": "000000ff0f000000000000000000000000000000000000000000000000000000",
  "blocktime": 10,
  "powaveragingwindow": 45,
  "notarizationperiod": 60,
  "eras": [
    {
      "reward": 0,
      "decay": 0,
      "halving": 0,
      "eraend": 0
    }
  ],
  "currencyidhex": "da67c931cb4f33ab234fa4fa4a5b5365d77052ba",
  "fullyqualifiedname": "chips10sec",
  "currencynames": {
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": "VRSCTEST"
  },
  "definitiontxid": "8b0f68665e198294ff848de1772f413a70da65685f57dd0e8fffa75d18c5b147",
  "definitiontxout": 1,
  "nodes": [
    {
      "networkaddress": "51.222.159.244:12121",
      "nodeidentity": "i3UXS5QPRQGNRDDqVnyWTnmFCTHDbzmsYk"
    },
    {
      "networkaddress": "149.56.13.160:12121",
      "nodeidentity": "i3UXS5QPRQGNRDDqVnyWTnmFCTHDbzmsYk"
    }
  ],
  "lastconfirmedheight": 39350,
  "lastconfirmedtxid": "8b0f68665e198294ff848de1772f413a70da65685f57dd0e8fffa75d18c5b147",
  "lastconfirmedcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "launchcurrencies": [
      {
        "currencyid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
        "weight": 0.00000000,
        "reserves": 1.00000000,
        "priceinreserve": 1.00000000
      }
    ],
    "initialsupply": 0.00000000,
    "emitted": 0.00000000,
    "supply": 21000000.00000000,
    "currencies": {
      "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 1.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      }
    },
    "primarycurrencyfees": 0.00000000,
    "primarycurrencyconversionfees": 0.00000000,
    "primarycurrencyout": 0.00000000,
    "preconvertedout": 0.00000000
  },
  "bestheight": 39350,
  "besttxid": "8b0f68665e198294ff848de1772f413a70da65685f57dd0e8fffa75d18c5b147",
  "bestcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "launchcurrencies": [
      {
        "currencyid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
        "weight": 0.00000000,
        "reserves": 1.00000000,
        "priceinreserve": 1.00000000
      }
    ],
    "initialsupply": 0.00000000,
    "emitted": 0.00000000,
    "supply": 21000000.00000000,
    "currencies": {
      "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 1.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      }
    },
    "primarycurrencyfees": 0.00000000,
    "primarycurrencyconversionfees": 0.00000000,
    "primarycurrencyout": 0.00000000,
    "preconvertedout": 0.00000000
  }
}
```

**Cross-reference `startblock` value with current blockchain height.  Do not begin mining/staking until after this height!**


---

<h2 id="launch">Launching a PBaaS chain</h4>

<h3 id="launch-firstblock">Merge mine the first block on generated chain</h3>

Now that we have defined our chain's currency, generated and committed the launch parameters...  we need to merge mine at least the first block in conjunction with the launch system chain.

Only the first block must be merge mined.  Afterward, we can choose to merge mine or mine the newly launched chain independently.

To do this, we must be already mining on `VRSCTEST` in our example.  This can be performed with the following startup options (for solo mining):

```
./verusd -chain=vrsctest -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

Next, we must spin up our `chips10sec` daemon, and start mining.  We can either: a.) fetch the address generated on wallet initialization, or b.) import our WIF private key from `vrsctest` chain.

```
# Start chips10sec daemon
./verusd -chain=chips10sec &

# option a.) Fetch address generated on init
./verus -chain=chips10sec getaddressesbyaccount ""

# option b.) import WIF
./verus -chain=chips10sec importprivkey <WIF here>
```

Once we've fetched our new address, or imported a known address, we can restart our `chips10sec` daemon to begin merge mining with parent `vrsctest`:

```
# Stop chips10sec daemon
./verus -chain=chips10sec stop

# Restart with mining options
./verusd -chain=chips10sec -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

If successful, you should see output in your terminal:

```
Merge mining chips10sec with vrsctest as the hashing chain
Found block 2885
staking reward 0.00010000 chips10sec!
POS hash: 00000000007fca5a2624d9276d2ddd7074f68fd32b9f6c81904d96b4a12f6301
target:   0000000002e3ae00000000000000000000000000000000000000000000000000

Block 2885 added to chain
```

Similar messages should be seen in `vrsctest` daemon output.  Once the first block is mined, running `vrsctest` daemon at the same time is optional.

---

<h2 id="token">Creating a Token on Chips PBaaS Chain</h2>

After creating our own chain using Verus's PBaaS tooling, we can issue tokens with various properties on our newly launched chain.

This allows us to create IDs on `chips10sec`'s mineable blockchain, and also allows us to create subIDs within the token's namespace. SubIDs have the added benefit of greatly reduced registration costs compared to IDs on the parent chain.

Below, we'll create a token called `cashiers` on the `chips10sec` blockchain. This token is 100% arbitrary, coins should have no real value, and we will only be using it for recording certain game data for Chips.

1. Register name commitment and identity in the same fashion as on `vrsctest` blockchain.  This time we perform the operation on `chips10sec`:

```
./verus -chain=chips10sec registernamecommitment cashiers RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

The response should look the same as before, with a different value for "parent":

```
{
  "txid": "5308a944b4e5fa7b3f52755312dba248f2b952d243c2a49e78bfd01b3cb55b3a",
  "namereservation": {
    "version": 1,
    "name": "cashiers",
    "parent": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "salt": "01afedbd3d9ff510192cf4b63381225fe7d5aa1e20daccee190732527c9dde46",
    "referral": "",
    "nameid": "i6CS9ewyp4oWozG2eceXPk3uSHg3dihdPg"
  }
}
```

2. Next, register the identity:

```
./verus -chain=chips10sec registeridentity '{
  "txid": "5308a944b4e5fa7b3f52755312dba248f2b952d243c2a49e78bfd01b3cb55b3a",
  "namereservation": {
    "version": 1,
    "name": "cashiers",
    "parent": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "salt": "01afedbd3d9ff510192cf4b63381225fe7d5aa1e20daccee190732527c9dde46",
    "referral": "",
    "nameid": "i6CS9ewyp4oWozG2eceXPk3uSHg3dihdPg"
  }, 
    "identity":{
        "name":"cashiers", 
        "primaryaddresses":["RAaHAuEqo7Ek2WMvEtsRRKg7QjABaJsx9v"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```

3. Fund the registered identity with 200 `chips10sec`, and issue a `definecurrency` command:

```
./verus -chain=chips10sec definecurrency '{"options":32, "name":"cashiers.chips10sec","preallocations":[{"cashiers.chips10sec@":1000000}],"proofprotocol":2}'
```

The above options should be fairly self-explanatory.  We are creating a token with `options = 32`, and preallocating 1 million tokens to ourselves. `proofprotocol = 2` allows us to generate subIDs within the token's namespace.

Wait for the `startblock` height to be reached on parent chain, and token will be live.

<h3 id="subID">Creating a subID in Token Namespace</h3>

We can register a subID in the same fashion as an upper-level ID, but there are a couple differences in arguments when doing so:

1. Register name commitment *(take note of the extra arguments being passed to command)*:

```
./verus -chain=chips10sec registernamecommitment test RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph "" "cashiers.chips10sec"
{
  "txid": "72b48428e5f19e5e95165f7f3cdd40234264d3cd845f531fb5592c0e7a6fed2a",
  "namereservation": {
    "version": 1,
    "name": "test",
    "parent": "i6CS9ewyp4oWozG2eceXPk3uSHg3dihdPg",
    "salt": "6944c4ec9804a08b4bcdcd490fb6f17aedca3ecc6e67d9f92a0277c0f222ed8f",
    "referral": "",
    "nameid": "iBiobcQ49xpTuL897iAjkYfosbQLMNpUjH"
  }
}
```

2. Register the subID *(take note that we need to reference token name in `"identity"` object of JSON)*:

```
./verus -chain=chips10sec registeridentity '{
  "txid": "72b48428e5f19e5e95165f7f3cdd40234264d3cd845f531fb5592c0e7a6fed2a",
  "namereservation": {
    "version": 1,
    "name": "test",
    "parent": "i6CS9ewyp4oWozG2eceXPk3uSHg3dihdPg",
    "salt": "6944c4ec9804a08b4bcdcd490fb6f17aedca3ecc6e67d9f92a0277c0f222ed8f",
    "referral": "",
    "nameid": "iBiobcQ49xpTuL897iAjkYfosbQLMNpUjH"
  }, 
    "identity":{
        "name":"test.cashiers", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```

3. Verify Creation of our subID:

```
./verus -chain=chips10sec getidentity test.cashiers.chips10sec@
{
  "identity": {
    "version": 3,
    "flags": 0,
    "primaryaddresses": [
      "RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"
    ],
    "minimumsignatures": 1,
    "name": "test",
    "identityaddress": "iBiobcQ49xpTuL897iAjkYfosbQLMNpUjH",
    "parent": "i6CS9ewyp4oWozG2eceXPk3uSHg3dihdPg",
    "systemid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg",
    "contentmap": {
    },
    "contentmultimap": {
    },
    "revocationauthority": "iBiobcQ49xpTuL897iAjkYfosbQLMNpUjH",
    "recoveryauthority": "iBiobcQ49xpTuL897iAjkYfosbQLMNpUjH",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 1803,
  "txid": "002d2c3be9d34ab75a152af6f8bded6af5c985dbb774342826143e9f97c71321",
  "vout": 0
}
```

In addition to lowering identity registration costs, subIDs reduce costs for updating the identity later.  In Chips, we will be using the `contentmultimap` to store various game-related data.  Lower costs at the subID level make this much less expensive.
