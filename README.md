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
./verus -chain=vrsctest registernamecommitment chipstest2 RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "1ba6d12f6645e93fda7a96cdaa1686b09712df5a3693f14c08c2329b76156ae7",
  "namereservation": {
    "version": 1,
    "name": "chipstest2",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "4e070d521f213777fad182d3525ffdabf2a55cc4d612276163af5cb47d3084df",
    "referral": "",
    "nameid": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i"
  }
}
```

**Step 2: Register identity**

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{
  "txid": "1ba6d12f6645e93fda7a96cdaa1686b09712df5a3693f14c08c2329b76156ae7",
  "namereservation": {
    "version": 1,
    "name": "chipstest2",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "4e070d521f213777fad182d3525ffdabf2a55cc4d612276163af5cb47d3084df",
    "referral": "",
    "nameid": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i"
  },
    "identity":{
        "name":"chipstest2", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
beaff25a7ae6c616828c6e62d752fe51d570a6463274d3d3fc89ba6f17112a4d
```

Wait for this transaction to be confirmed.

**Step 3: Verify identity has been created**

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chipstest2@"

# By i-address
./verus -chain=vrsctest getidentity iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i
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
    "name": "chipstest2",
    "identityaddress": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "revocationauthority": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
    "recoveryauthority": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 32120,
  "txid": "beaff25a7ae6c616828c6e62d752fe51d570a6463274d3d3fc89ba6f17112a4d",
  "vout": 0
}
```

---

<h2 id="chaingen">Launching a PBaaS Chain</h2>

**Step 1: Send VRSCTEST, and Basket currencies to identity address**

To launch a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.

For this chain, we will be launching both a PBaaS chain (`chipstest2@`), and a gatewayconverter chain (`Cashier.chipstest2`).  We will be initially contributing a basket of currencies to the gateway converter.  In example below, we are sending `15000 VRSCTEST`, `2 BTC`, and `2400 nexus` to fund the basket.

```
# send vrsctest
./verus -chain=vrsctest sendtoaddress chipstest2@ 15000
5f423ae417e84d98ac34a6aa3ea97da056717e80ab2a75a238f4a691e3b1f1b7

# send btc
./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chipstest2@","currency":"btc","amount":2}]'
opid-a862d7a2-8103-42f2-a5a4-472cbd1068a2

# send nexus
./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chipstest2@","currency":"nexus","amount":2500}]'
opid-5aa85330-7b12-4259-be27-039b7dbea16b
```

We also want to verify that the resulting operation ids for `btc` and `nexus` went through:

```
./verus -chain=vrsctest z_getoperationresult '["opid-a862d7a2-8103-42f2-a5a4-472cbd1068a2","opid-5aa85330-7b12-4259-be27-039b7dbea16b"]'
[
  {
    "id": "opid-a862d7a2-8103-42f2-a5a4-472cbd1068a2",
    "status": "success",
    "creation_time": 1653495949,
    "result": {
      "txid": "f893e949e3b6510ac5cb6b70c81ed03ed026e127034dacc1e55dc2a4dd138daa"
    },
    "execution_secs": 0.035671532,
    "method": "sendcurrency",
    "params": [
      {
        "address": "chipstest2@",
        "currency": "btc",
        "amount": 2
      }
    ]
  },
  {
    "id": "opid-5aa85330-7b12-4259-be27-039b7dbea16b",
    "status": "success",
    "creation_time": 1653496075,
    "result": {
      "txid": "e2217ae1d5df8fef916cf31a24ff4336548a92cdaa4fdb85d5aed095065261a0"
    },
    "execution_secs": 0.04157405,
    "method": "sendcurrency",
    "params": [
      {
        "address": "chipstest2@",
        "currency": "nexus",
        "amount": 2500
      }
    ]
  }
]
```

**Step 2: Define a currency with desired parameters**

```
./verus -chain=vrsctest definecurrency '{"name":"CHIPSTEST2","options":264,"currencies":["vrsctest"],"maxpreconversion":[0], "conversions":[1],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["Notary1@","Notary2@","Notary3@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"},{"networkaddress":"149.56.13.160:12121"}],"preallocations":[{"biz@":10000000},{"allbits@":10000000}], "gatewayconvertername":"Cashier", "gatewayconverterissuance":1000000}' '{"currencies":["vrsctest","nexus","BTC","chipstest2"],"initialcontributions":[1000,2300,1.9,0],"initialsupply":4000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


**Step 3: Send resulting raw transaction to network**

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f89044d2a11176fba89fcd3d3743246a670d551fe52d7626e8c8216c6e67a5af2afbe00000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940e6757aaa419d1a18888744e3e77711e5b619a0373a5497be8a0556179dfdbb912cc597972ec31e8ad2ab3765e4bdbd72fa44885d2b2d355dda014445c7671c4effffffffa061520695d0aed585db4faacd928a543643ff241af36c91ef8fdfd5e17a21e200000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940c7c724fb935030ca95fc0fc2f6587d58d59194befa07fffc70daaa0c37dee0191fc355f2b1fca323ddeb6ac5cad8aa2acb5ec98180fae1a6d82f310d33e2bbeeffffffffaa8d13dda4c25de5c1ac4d0327e126d03ed01ec8706bcbc50a51b6e349e993f800000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc99405c7403865d24a21f721c38e553b8064e2d9964ba6c88edd2403264cf6a6d3bf43912024a0cb77c93ad3e67c4a312a1072c15721616dabffd08c501e12fa05c8affffffffb7f1b1e391a6f438a2752aab807e7156a07da93eaaa634ac984de817e43a425f01000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc994038155022caa1eb3ea5ec6f21f7ee7ce852f741b49e28012313072525d5a37b884e07107d25405252138b2392229d6194d4f44cbb29147ff69b7fd625bfca7502ffffffff0f0000000000000000fd250147040300010315049c64e2e85108b76c41a26c97f34ef757f93f325615049c64e2e85108b76c41a26c97f34ef757f93f325615049c64e2e85108b76c41a26c97f34ef757f93f3256cc4cd904030e010115049c64e2e85108b76c41a26c97f34ef757f93f32564c8403000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a6368697073746573743200009c64e2e85108b76c41a26c97f34ef757f93f32569c64e2e85108b76c41a26c97f34ef757f93f325600a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f010115049c64e2e85108b76c41a26c97f34ef757f93f32561b040310010115049c64e2e85108b76c41a26c97f34ef757f93f3256750000000000000000fdc3012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d960104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d6c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a43484950535445535432a6ef9ea235635e328124ff3429db9f9e91b64e2d9c64e2e85108b76c41a26c97f34ef757f93f32560100000001000000000080fa78000000000000000000028e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e0080c6a47e8d03006760a36ddeed08e1862f8cf5cf424f13636a0e4a0080c6a47e8d030000407a10f35a000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f5050000000000010000000000000000010000000000000000010000000000000000000000000003f9fc05c0d2aa4c1be97ef0e11fc1f9d17e11aefa5ff0a8b4d60c591a1cfd592a2304936f86af3fdacbb76c71819024b9e5ac9fd74257737cf4bd8c6a02a49faec70003aed6c100c9bfde8f009c8ca4939f00a49faec700bc8340bc83400743617368696572e1e1011e01000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001009c64e2e85108b76c41a26c97f34ef757f93f3256010000009c64e2e85108b76c41a26c97f34ef757f93f32560000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fdb9012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d8c0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d620101800300009c64e2e85108b76c41a26c97f34ef757f93f3256010002009c64e2e85108b76c41a26c97f34ef757f93f325601a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000082dcbd84cf9bff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f505000000000100000000000000000100000000000000000100000000010000000000000000e47d00000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000133134392e35362e31332e3136303a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000009c64e2e85108b76c41a26c97f34ef757f93f32569c64e2e85108b76c41a26c97f34ef757f93f32560080fa640000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a74000000000000ffffffff00750000000000000000fd9a012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d6d0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d430101000000210200009c64e2e85108b76c41a26c97f34ef757f93f32560743617368696572a6ef9ea235635e328124ff3429db9f9e91b64e2d9c64e2e85108b76c41a26c97f34ef757f93f32560100000001000000000080fa780000a0db215d0000000000407a10f35a000004a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40769c64e2e85108b76c41a26c97f34ef757f93f32560440787d0140787d0140787d0140787d0104000000000000000000000000000000000000000000000000000000000000000000000400e8764817000000007c118d35000000802b530b0000000000000000000000000400e8764817000000007c118d35000000802b530b00000000000000000000000000000000000000a49faec70003aed6c100750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001009c64e2e85108b76c41a26c97f34ef757f93f3256010000001697f4671639ed992a4d347064afb3e89c4da59a00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd7f022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d520204030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d280201800300001697f4671639ed992a4d347064afb3e89c4da59a010003001697f4671639ed992a4d347064afb3e89c4da59a04a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40769c64e2e85108b76c41a26c97f34ef757f93f32560440787d0140787d0140787d0140787d010400000000000000000000000000000000000000000000000000407a10f35a00008ad18dedbf00008ad18dedbf0000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000407a10f35a000004000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000004a086010000000000a086010000000000a086010000000000a0860100000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000e47d00000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000009c64e2e85108b76c41a26c97f34ef757f93f32561697f4671639ed992a4d347064afb3e89c4da59a0080fa640000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000000ffffffff0075cbc6f44917000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d01a6ef9ea235635e328124ff3429db9f9e91b64e2d81f3ced0f02b05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b2004141697f4671639ed992a4d347064afb3e89c4da59a1697f4671639ed992a4d347064afb3e89c4da59a75204e000000000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d015f846cd993507f8a47d65809198d6649bea6004d85d882fbaa0a05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b2004141697f4671639ed992a4d347064afb3e89c4da59a1697f4671639ed992a4d347064afb3e89c4da59a75204e000000000000961a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c77040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5b0154852c4e9fb1d4c4291fc093e41ce2c7befa4076d9cec91705a6ef9ea235635e328124ff3429db9f9e91b64e2d809b2004141697f4671639ed992a4d347064afb3e89c4da59a1697f4671639ed992a4d347064afb3e89c4da59a750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f009c64e2e85108b76c41a26c97f34ef757f93f32567500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec7009c64e2e85108b76c41a26c97f34ef757f93f325675f55c4578580000007a1b040300010115049c64e2e85108b76c41a26c97f34ef757f93f3256cc4c5a040309010115049c64e2e85108b76c41a26c97f34ef757f93f32563e86fefeff010254852c4e9fb1d4c4291fc093e41ce2c7befa4076e9dc9700000000005f846cd993507f8a47d65809198d6649bea6004d762eaaa4040000007500000000f97d00000000000000000000000000

# resulting txid
fe8ac7c7b3c543ff82becba1a27efd244690dfa7bb737946a31cd953dfcd3bd6
```

**Step 4: Verify currency generation was succesful**

```
./verus -chain=vrsctest getcurrency chipstest2
```

If successful, output should show similar to the following:

```
{
  "version": 1,
  "options": 264,
  "name": "CHIPSTEST2",
  "currencyid": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
  "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "systemid": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
  "notarizationprotocol": 1,
  "proofprotocol": 1,
  "launchsystemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "startblock": 32248,
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
      "iGTdav42siHgX96ybWn3pRTFxQgiUpZ9K9": 10000000.00000000
    },
    {
      "iCu8oPVnPvLmbiNSDx3tRSRGumEtFV69FJ": 10000000.00000000
    }
  ],
  "initialcontributions": [
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
  "gatewayconverterid": "i5XzLJ2zimjo2AHTEVEs4LNbQdX1wxkiqd",
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
  "currencyidhex": "56323ff957f74ef3976ca2416cb70851e8e2649c",
  "fullyqualifiedname": "CHIPSTEST2",
  "currencynames": {
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": "vrsctest"
  },
  "definitiontxid": "fe8ac7c7b3c543ff82becba1a27efd244690dfa7bb737946a31cd953dfcd3bd6",
  "definitiontxout": 1,
  "bestheight": 32236,
  "lastconfirmedheight": 32236,
  "bestcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
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
  "lastconfirmedcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i",
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


