# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.5-2), with the goal of providing a reproducible workflow for future use.

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

---

<h2 id="compile">Compiling from Source</h2>

<h3>Step 1: Clone, build dependencies:</h3>

```
cd ~
git clone https://github.com/VerusCoin/VerusCoin.git verus
cd ~/verus/depends
make -j4
```

Wait for `depends` to finish building.

<h3>Step 2: Configure build enviroment, and build from source:</h3>

```
cd ~/verus
./autogen.sh
CONFIG_SITE=$PWD/depends/x86_64-unknown-linux-gnu/share/config.site ./configure
make -j4
```

If all steps completed successfully, you should have resulting binaries located in `~/verus/src`.

<h3>Step 3: Launch Verus daemon</h3>

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
./verus -chain=vrsctest registernamecommitment chips10sec RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "61839b7108b6ba5fa8c5c8483df5fb9eb823264cb349c33151928d3fbf98fe4c",
  "namereservation": {
    "version": 1,
    "name": "chips10sec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "99784ab53ec52f3158ca2c636b2b3dd7304708fece587038b231d6821c7df45a",
    "referral": "",
    "nameid": "iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg"
  }
}
```

<h3>Step 2: Register identity</h3>

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{
  "txid": "61839b7108b6ba5fa8c5c8483df5fb9eb823264cb349c33151928d3fbf98fe4c",
  "namereservation": {
    "version": 1,
    "name": "chips10sec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "99784ab53ec52f3158ca2c636b2b3dd7304708fece587038b231d6821c7df45a",
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
b0a2687a721d5da11d4605d0d711f973676a214da365bdc1d4871d1a30dbbec9
```

Wait for this transaction to be confirmed.

<h3>Step 3: Verify identity has been created</h3>

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chips10sec@"

# By i-address
./verus -chain=vrsctest getidentity iLThsqsgwFRKzRG11j7QaYgNQJ9q16VGpg
```

If successful, output should show similar to the following:

```
{
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
  "blockheight": 30839,
  "txid": "b0a2687a721d5da11d4605d0d711f973676a214da365bdc1d4871d1a30dbbec9",
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

- `"currencies":["vrsctest","kmd","chips10sec"]`

Specifies a basket of currencies backing our gateway issuance.

- `"initialcontributions":[1000,200,0]`

Specifies amounts of backing currencies to contribute on launch of gateway from our `chips10sec@` identity address.

- `"initialsupply":2400`


<h3 id="chaingen-fund">Step 1: Send VRSCTEST, and Basket currencies to identity address</h3>

To generate a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.

For this chain, we will be generating both a PBaaS chain (`chips10sec@`), and a gatewayconverter chain (`bridge.chips10sec`).  We will be initially contributing a basket of currencies to the gateway converter.  In example below, we are sending `12000 VRSCTEST` and `200 KMD` to fund the basket.

```
# send vrsctest
./verus -chain=vrsctest sendtoaddress chips10sec@ 12000
58ae24dfc439dd8de9e53a3fddc8237482d8d298a4d5c70af2c34595b1f9b862

 send kmd
./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chips10sec@","currency":"kmd","amount":110}]'
opid-6865bf89-ff94-4cf9-aa75-aa9176e58763

```

We also want to verify that the resulting operation id for 'kmd` transfer succeeded:

```
./verus -chain=vrsctest z_getoperationresult '["opid-6865bf89-ff94-4cf9-aa75-aa9176e58763"]'
[
  {
    "id": "opid-6865bf89-ff94-4cf9-aa75-aa9176e58763",
    "status": "success",
    "creation_time": 1668717541,
    "result": {
      "txid": "781bb2b906bc523717f2aa13962a8b272472302dc8d09815f2c22ae0bc22f9b7"
    },
    "execution_secs": 0.045698179,
    "method": "sendcurrency",
    "params": [
      {
        "currency": "KMD",
        "address": "chips10sec@",
        "amount": 110.00
      }
    ]
  }
]
```

<h3 id="chaingen-define">Step 2: Define a currency with desired parameters</h3>

```
./verus -chain=vrsctest definecurrency '{"name":"chips10sec","options":264,"currencies":["vrsctest"],"maxpreconversion":[0], "conversions":[1],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["biz@","biznotary@","biznotary1@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:21211"},{"networkaddress":"149.56.13.160:21211"}],"preallocations":[{"biz@":20950000}], "gatewayconvertername":"bridge", "gatewayconverterissuance":50000, "blocktime":10}' '{"currencies":["vrsctest","kmd","chips10sec"],"initialcontributions":[1000,200,0],"initialsupply":2400}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


<h3>Step 3: Send resulting raw transaction to network</h3>

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f8904c9bedb301a1d87d4c1bd65a34d216a6773f911d7d005461da15d1d727a68a2b000000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc99408cff75e2cab0c8790c2f8d6e6d7ee871629280007a8ff2e1733ae5a1c02860947ef9d5f78eb18eb23def5ad2e0c740fe646cbbdd714e9cb78dbc6640a57be115ffffffff195da351ca10c68cdb5495690c463e1c78988801123a18ddd7f54a78da78c24000000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940bae02dad15abdd543a570cc5ebe36d592a8a1865697b5c9b97aa9165dd00be537012c37c1190399325f765ba09e626531085fd187a1f26ac322834497c4ce53bffffffffb7f922bce02ac2f21598d0c82d307224278b2a9613aaf2173752bc06b9b21b7800000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940034d14db3e197491eee08dc9e66713270c0df27a1d19d22d8489cb373c99819a37213baf4e6f7d1908acfd7b3c8f5edda366b0d9aa65c0dee5c4f1abb47d0ceeffffffff62b8f9b19545c3f20ac7d5a498d2d8827423c8dd3f3ae5e98ddd39c4df24ae5801000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc994074b04fdc3e140840548ec2bd8f19e6b428036fe6fd276d5a35edd1bd502cb5ab1516a1347285f56879bf461338027b8285a3ca20756b0eac6c594e73222ba0f5ffffffff0e0000000000000000fd25014704030001031504ba5270d765535b4afaa44f23ab334fcb31c967da1504ba5270d765535b4afaa44f23ab334fcb31c967da1504ba5270d765535b4afaa44f23ab334fcb31c967dacc4cd904030e01011504ba5270d765535b4afaa44f23ab334fcb31c967da4c8403000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a636869707331307365630000ba5270d765535b4afaa44f23ab334fcb31c967daba5270d765535b4afaa44f23ab334fcb31c967da00a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f01011504ba5270d765535b4afaa44f23ab334fcb31c967da1b04031001011504ba5270d765535b4afaa44f23ab334fcb31c967da750000000000000000fdc3012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d960104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d6c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a63686970733130736563a6ef9ea235635e328124ff3429db9f9e91b64e2dba5270d765535b4afaa44f23ab334fcb31c967da01000000010000000000000000000000000000000000000000000000000080f031000000000000000000018e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e00f0cd3264710700005039278c04000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f50500000000000100000000000000000100000000000000000100000000000000000000000000038e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e6df4164224d7eb36df23a3fccf2cd6ae7b5827dc4d3213c88cd5b55c4e16c151a34a63f37e52f1ae02a49faec70003f98800c9bfde8f009c8ca4939f00a49faec700bc8340bc8340066272696467650fff001e0a0000002d0000003c0001000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a01000100ba5270d765535b4afaa44f23ab334fcb31c967da01000000ba5270d765535b4afaa44f23ab334fcb31c967da0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fdb9012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d8c0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d62010280030000ba5270d765535b4afaa44f23ab334fcb31c967da01000200ba5270d765535b4afaa44f23ab334fcb31c967da01a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000082dcbd84cf9bff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f5050000000001000000000000000001000000000000000001000000000100000000000000009d7800000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a32313231310000000000000000000000000000000000000000133134392e35362e31332e3136303a32313231310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d0000000000000000000000000000000000000000000000000000000000000000ba5270d765535b4afaa44f23ab334fcb31c967daba5270d765535b4afaa44f23ab334fcb31c967da0000ffffffff000000000080f01d01a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a740000000000750000000000000000fd7c012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d4f0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d25010100000021020000ba5270d765535b4afaa44f23ab334fcb31c967da06627269646765a6ef9ea235635e328124ff3429db9f9e91b64e2dba5270d765535b4afaa44f23ab334fcb31c967da01000000010000000000ba5270d765535b4afaa44f23ab334fcb31c967da80f0310000601de13700000000005039278c04000003a6ef9ea235635e328124ff3429db9f9e91b64e2d3c28dd25a7127ce1b1e9a8dd9fa550a1b805dc10ba5270d765535b4afaa44f23ab334fcb31c967da0356a0fc0155a0fc0155a0fc010300000000000000000000000000000000000000000000000000000300e876481700000000c817a80400000000000000000000000300e876481700000000c817a804000000000000000000000000000000000000a49faec70003f98800750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a01000100ba5270d765535b4afaa44f23ab334fcb31c967da0100000091ccbce2bc71649d2d91bf6384e38d266a1d7fa500000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd23022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4df60104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34dcc01028003000091ccbce2bc71649d2d91bf6384e38d266a1d7fa50100030091ccbce2bc71649d2d91bf6384e38d266a1d7fa503a6ef9ea235635e328124ff3429db9f9e91b64e2d3c28dd25a7127ce1b1e9a8dd9fa550a1b805dc10ba5270d765535b4afaa44f23ab334fcb31c967da0356a0fc0155a0fc0155a0fc010300000000000000000000000000000000005039278c04000085fd87f4bf000085fd87f4bf0000000000000000000000000000000000000000000000000000000000000000000300000000000000000000000000000000005039278c04000003000000000000000000000000000000000000000000000000030000000000000000000000000000000000000000000000000347e801000000000048e801000000000048e8010000000000030000000000000000000000000000000000000000000000000300000000000000000000000000000000000000000000000003000000000000000000000000030000000000000000000000000000000000000000000000009d7800000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d0000000000000000000000000000000000000000000000000000000000000000ba5270d765535b4afaa44f23ab334fcb31c967da91ccbce2bc71649d2d91bf6384e38d266a1d7fa50000ffffffff000000000080f01d01a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000075cbc6f44917000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d01a6ef9ea235635e328124ff3429db9f9e91b64e2d81f3ced0f02b05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b20041491ccbce2bc71649d2d91bf6384e38d266a1d7fa591ccbce2bc71649d2d91bf6384e38d266a1d7fa575204e000000000000971a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c78040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5c013c28dd25a7127ce1b1e9a8dd9fa550a1b805dc10c9c28faf2205a6ef9ea235635e328124ff3429db9f9e91b64e2d809b20041491ccbce2bc71649d2d91bf6384e38d266a1d7fa591ccbce2bc71649d2d91bf6384e38d266a1d7fa5750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f00ba5270d765535b4afaa44f23ab334fcb31c967da7500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec700ba5270d765535b4afaa44f23ab334fcb31c967da7515f3e09e12000000551b04030001011504ba5270d765535b4afaa44f23ab334fcb31c967dacc3604030901011504ba5270d765535b4afaa44f23ab334fcb31c967da1a013c28dd25a7127ce1b1e9a8dd9fa550a1b805dc1082d9b8f25e7500000000b27800000000000000000000000000

# resulting txid
6b56b932421bc3779e1c3dce386ab5015b7302ce8e88c9f145ce9e4d53a471a7
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
  "startblock": 30897,
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
  "definitiontxid": "6b56b932421bc3779e1c3dce386ab5015b7302ce8e88c9f145ce9e4d53a471a7",
  "definitiontxout": 1,
  "nodes": [
    {
      "networkaddress": "51.222.159.244:21211",
      "nodeidentity": "i3UXS5QPRQGNRDDqVnyWTnmFCTHDbzmsYk"
    },
    {
      "networkaddress": "149.56.13.160:21211",
      "nodeidentity": "i3UXS5QPRQGNRDDqVnyWTnmFCTHDbzmsYk"
    }
  ],
  "lastconfirmedheight": 30877,
  "lastconfirmedtxid": "6b56b932421bc3779e1c3dce386ab5015b7302ce8e88c9f145ce9e4d53a471a7",
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
  "bestheight": 30877,
  "besttxid": "6b56b932421bc3779e1c3dce386ab5015b7302ce8e88c9f145ce9e4d53a471a7",
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


