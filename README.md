# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.2-3), with the goal of providing a reproducible workflow for future use.

**Contents:**

- [Compiling from source](#compile)
- [Generating an ID for use on Verus Testnet](#idgen)
- [Launching a PBaaS chain](#chaingen)

---

<h2 id="compile">Compiling from Source</h2>

**Step 1: Clone, build dependencies:**

```
cd ~
git clone https://github.com/VerusCoin/VerusCoin.git verus
cd ~/verus/depends
make -j4
```

Wait for `depends` to finish building.

**Step 2: Configure build enviroment, and build from source:**

```
cd ~/verus
./autogen.sh
CONFIG_SITE=$PWD/depends/x86_64-unknown-linux-gnu/share/config.site ./configure
make -j4
```

If all steps completed successfully, you should have resulting binaries located in `~/verus/src`.

**Step 3: Launch Verus daemon**

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

**Step 1: Register name commitment** 

```
./verus -chain=vrsctest registernamecommitment chipstest1 RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "4861c5db433c8fe34399d7a1f2ea332cce9c3ec1946144f2b0bd4b6b2d98d02a",
  "namereservation": {
    "version": 1,
    "name": "chipstest1",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "560aeae6464cbab4fbd54ead804390ada238567cfe0d38aba01b4faad82dade8",
    "referral": "",
    "nameid": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F"
  }
}
```

**Step 2: Register identity**

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{
  "txid": "4861c5db433c8fe34399d7a1f2ea332cce9c3ec1946144f2b0bd4b6b2d98d02a",
  "namereservation": {
    "version": 1,
    "name": "chipstest1",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "560aeae6464cbab4fbd54ead804390ada238567cfe0d38aba01b4faad82dade8",
    "referral": "",
    "nameid": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F"
  },
  "identity": {
        "name":"chipstest1", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"],
        "minimumsignatures":1, 
        "privateaddress": ""
  }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
04bfbf8517e776d8177e7e735948aad47079c0f70a6263944966adcc5805a201
```
Wait for this transaction to be confirmed.

**Step 3: Verify identity has been created**

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chipstest1@"

# By i-address
./verus -chain=vrsctest getidentity iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F
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
    "name": "chipstest1",
    "identityaddress": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "revocationauthority": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
    "recoveryauthority": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 29363,
  "txid": "04bfbf8517e776d8177e7e735948aad47079c0f70a6263944966adcc5805a201",
  "vout": 0
}
```

---

<h2 id="chaingen">Launching a PBaaS Chain</h2>

**Step 1: Send VRSCTEST to identity address**

To launch a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.


```
./verus -chain=vrsctest sendtoaddress chipstest0@ 10200
```


**Step 2: Define a currency with desired parameters**

```
./verus -chain=vrsctest definecurrency '{"name":"CHIPSTEST1","options":264,"currencies":["vrsctest","nexus","BTC"],"minpreconversion":[100.00,5000.00,4.00],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["Notary1@","Notary2@","Notary3@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"},{"networkaddress":"149.56.13.160:12121"}], "gatewayconvertername":"Cashier", "gatewayconverterissuance":1000000}' '{"currencies":["vrsctest","nexus","BTC","chipstest1"],"initialcontributions":[0,0,0,0],"initialsupply":1000000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


**Step 3: Send resulting raw transaction to network**

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f890301a20558ccad66499463620af7c07970d4aa4859737e7e17d876e71785bfbf0400000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940f2632b107403f770c5bc1d938a267595b0c8348d1f3a1d463f0ae025dcd6c66f1e8fac380a8135bb2b2ba8f010cfa2ea9a408a183e160790416e82d8c12198b5ffffffffb9649426bef74068c596273701d735c83a05bd99ed9a4dce3b61425824ac209101000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc99403b26c64f50acaf5a9477a922438d2640d8591505acf8d19ed8320b00e722cbf542164c7f511ebd52e62fb3006386121ad56c13a5126a42eb4c3d56b56e3ec2c9ffffffffd65c068a3ef55e302ad0c3b13def857e6f1b5a7b401f08da444cc577e008d8d201000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc994061f7c8904b2afef390e7ed19637fc605a509fef4f973c5fef78ce2eb8fea226e3f9bd439cbcf059d7609f5a0312ea2483058a799541400a88b004a49c5af9f72ffffffff0c0000000000000000fd250147040300010315045d32b4c906315b6e29345c2da442acec39fa61cb15045d32b4c906315b6e29345c2da442acec39fa61cb15045d32b4c906315b6e29345c2da442acec39fa61cbcc4cd904030e010115045d32b4c906315b6e29345c2da442acec39fa61cb4c8403000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a6368697073746573743100005d32b4c906315b6e29345c2da442acec39fa61cb5d32b4c906315b6e29345c2da442acec39fa61cb00a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f010115045d32b4c906315b6e29345c2da442acec39fa61cb1b040310010115045d32b4c906315b6e29345c2da442acec39fa61cb750000000000000000fdf3012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4dc60104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d9c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a43484950535445535431a6ef9ea235635e328124ff3429db9f9e91b64e2d5d32b4c906315b6e29345c2da442acec39fa61cb0100000001000000000080e4660000000000000000000000407a10f35a000003a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa407600030000000000000000000000000000000000000000000000000300e40b54020000000088526a740000000084d71700000000000300000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000003f9fc05c0d2aa4c1be97ef0e11fc1f9d17e11aefa5ff0a8b4d60c591a1cfd592a2304936f86af3fdacbb76c71819024b9e5ac9fd74257737cf4bd8c6a02a49faec70003aed6c100c9bfde8f009c8ca4939f00a49faec700bc8340bc83400743617368696572e1e1011e01000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001005d32b4c906315b6e29345c2da442acec39fa61cb010000005d32b4c906315b6e29345c2da442acec39fa61cb0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fd70022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d430204030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d190201800300005d32b4c906315b6e29345c2da442acec39fa61cb010002005d32b4c906315b6e29345c2da442acec39fa61cb03a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40760300000000000000000000000003000000000000000000000000000000000000000000000000000095ddb082e7ff0000000000000000000000000000000000000000000000000000000000000000000300000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000030000000000000000000000000000000000000000000000000300000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000030000000000000000000000000000000000000000000000000300000000000000000000000003000000000000000000000000000000000000000000000000d27200000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000133134392e35362e31332e3136303a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000005d32b4c906315b6e29345c2da442acec39fa61cb5d32b4c906315b6e29345c2da442acec39fa61cb0080e4520000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a74000000000000ffffffff00750000000000000000fd9a012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d6d0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d430101000000210200005d32b4c906315b6e29345c2da442acec39fa61cb0743617368696572a6ef9ea235635e328124ff3429db9f9e91b64e2d5d32b4c906315b6e29345c2da442acec39fa61cb0100000001000000000080e4660000407a10f35a00000000407a10f35a000004a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40765d32b4c906315b6e29345c2da442acec39fa61cb0440787d0140787d0140787d0140787d01040000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000000000000000000a49faec70003aed6c100750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001005d32b4c906315b6e29345c2da442acec39fa61cb0100000053a02488625bd4bd8161d0a5af49d590fa9fbaba00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd81022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d540204030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d2a02018003000053a02488625bd4bd8161d0a5af49d590fa9fbaba0100030053a02488625bd4bd8161d0a5af49d590fa9fbaba04a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40765d32b4c906315b6e29345c2da442acec39fa61cb0440787d0140787d0140787d0140787d010400000000000000000000000000000000000000000000000000407a10f35a000095ddb082e7ff000095ddb082e7ff0000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000407a10f35a00000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000490010000000000009001000000000000900100000000000090010000000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000d27200000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000005d32b4c906315b6e29345c2da442acec39fa61cb53a02488625bd4bd8161d0a5af49d590fa9fbaba0080e4520000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000000ffffffff00750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f005d32b4c906315b6e29345c2da442acec39fa61cb7500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec7005d32b4c906315b6e29345c2da442acec39fa61cb75f0ca052a01000000541b040300010115045d32b4c906315b6e29345c2da442acec39fa61cbcc35040309010115045d32b4c906315b6e29345c2da442acec39fa61cb190154852c4e9fb1d4c4291fc093e41ce2c7befa4076deae83007500000000e77200000000000000000000000000
```

If succesful, a txid will display as output.  Wait for this transaction to be included in a block.

Resulting txid: `09984eab1264282ce18d43d222b569b1b71d17b4e639ca1a08dc32065b3202c4``


**Step 4: Verify currency generation was succesful**

```
./verus -chain=vrsctest getcurrency chipstest1
```

If successful, output should show similar to the following:

```
./verus -chain=vrsctest getcurrency CHIPSTEST1
{
  "version": 1,
  "options": 264,
  "name": "CHIPSTEST1",
  "currencyid": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
  "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "systemid": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
  "notarizationprotocol": 1,
  "proofprotocol": 1,
  "launchsystemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "startblock": 29414,
  "endblock": 0,
  "currencies": [
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "iCBaGN6LsoEjSXzWRYuS2UEvKzDTDT3JXD",
    "iBBRjDbPf3wdFpghLotJQ3ESjtPBxn6NS3"
  ],
  "conversions": [
    0.00000000,
    0.00000000,
    0.00000000
  ],
  "minpreconversion": [
    100.00000000,
    5000.00000000,
    4.00000000
  ],
  "initialcontributions": [
    0.00000000,
    0.00000000,
    0.00000000
  ],
  "gatewayconverterissuance": 1000000.00000000,
  "idregistrationfees": 100.00000000,
  "idreferrallevels": 3,
  "idimportfees": 1.00000000,
  "notaries": [
    "iSGKbfsLYUXuf6J8NaETLyNT7xFyCNKcQY",
    "iCDovahDV4pT3WoLe18DbA325edTfxmwRq",
    "iN3gLWmAxczjxSLXfpRLRKA7rzE5X4LnzK"
  ],
  "minnotariesconfirm": 2,
  "currencyregistrationfee": 200.00000000,
  "pbaassystemregistrationfee": 10000.00000000,
  "currencyimportfee": 100.00000000,
  "transactionimportfee": 0.01000000,
  "transactionexportfee": 0.01000000,
  "gatewayconverterid": "iB6hMufwVkPVVqdndPCmtS7bfznRdvmnkq",
  "gatewayconvertername": "Cashier",
  "initialtarget": "000001e1e1000000000000000000000000000000000000000000000000000000",
  "eras": [
    {
      "reward": 0,
      "decay": 0,
      "halving": 0,
      "eraend": 0
    }
  ],
  "currencyidhex": "cb61fa39ecac42a42d5c34296e5b3106c9b4325d",
  "fullyqualifiedname": "CHIPSTEST1",
  "currencynames": {
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": "vrsctest",
    "iCBaGN6LsoEjSXzWRYuS2UEvKzDTDT3JXD": "nexus",
    "iBBRjDbPf3wdFpghLotJQ3ESjtPBxn6NS3": "BTC"
  },
  "definitiontxid": "09984eab1264282ce18d43d222b569b1b71d17b4e639ca1a08dc32065b3202c4",
  "definitiontxout": 1,
  "bestheight": 29397,
  "lastconfirmedheight": 29397,
  "bestcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
    "launchcurrencies": [
      {
        "currencyid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
        "weight": 0.00000000,
        "reserves": 0.00000000,
        "priceinreserve": 0.00000000
      },
      {
        "currencyid": "iCBaGN6LsoEjSXzWRYuS2UEvKzDTDT3JXD",
        "weight": 0.00000000,
        "reserves": 0.00000000,
        "priceinreserve": 0.00000000
      },
      {
        "currencyid": "iBBRjDbPf3wdFpghLotJQ3ESjtPBxn6NS3",
        "weight": 0.00000000,
        "reserves": 0.00000000,
        "priceinreserve": 0.00000000
      }
    ],
    "initialsupply": 0.00000000,
    "emitted": 0.00000000,
    "supply": 1000000.00000000,
    "currencies": {
      "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 0.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      },
      "iCBaGN6LsoEjSXzWRYuS2UEvKzDTDT3JXD": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 0.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      },
      "iBBRjDbPf3wdFpghLotJQ3ESjtPBxn6NS3": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 0.00000000,
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
  "lastconfirmedcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iByK23ZYmvow63D9t4N5JCqBeUcBJQS21F",
    "launchcurrencies": [
      {
        "currencyid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
        "weight": 0.00000000,
        "reserves": 0.00000000,
        "priceinreserve": 0.00000000
      },
      {
        "currencyid": "iCBaGN6LsoEjSXzWRYuS2UEvKzDTDT3JXD",
        "weight": 0.00000000,
        "reserves": 0.00000000,
        "priceinreserve": 0.00000000
      },
      {
        "currencyid": "iBBRjDbPf3wdFpghLotJQ3ESjtPBxn6NS3",
        "weight": 0.00000000,
        "reserves": 0.00000000,
        "priceinreserve": 0.00000000
      }
    ],
    "initialsupply": 0.00000000,
    "emitted": 0.00000000,
    "supply": 1000000.00000000,
    "currencies": {
      "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 0.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      },
      "iCBaGN6LsoEjSXzWRYuS2UEvKzDTDT3JXD": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 0.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      },
      "iBBRjDbPf3wdFpghLotJQ3ESjtPBxn6NS3": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 0.00000000,
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


