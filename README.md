# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.2-3), with the goal of providing a reproducible workflow for future use.

**Contents:**

- [Compiling from source](#compile)
- [Generating an ID for use on Verus Testnet](#idgen)
- [Generating a PBaaS chain](#chaingen)
  - [Conceptual description of chosen `definecurrency` options](#chaingen-concept)
    - [Background on CHIPS](#background)
    - [Itemized options for chipstensec chain](#options-chipstensec)
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
./verus -chain=vrsctest registernamecommitment chipstensec RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "a5e1cc0dd3ec5774c35087adb2bb7aff7bd74836c6a4c0b75c280aa6419d04a1",
  "namereservation": {
    "version": 1,
    "name": "chipstensec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "6b2ff5b181e2788b5108592742d9214b0e9a33569e5e1ebad48e4000e9aad986",
    "referral": "",
    "nameid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty"
  }
}
```

<h3>Step 2: Register identity</h3>

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{
  "txid": "a5e1cc0dd3ec5774c35087adb2bb7aff7bd74836c6a4c0b75c280aa6419d04a1",
  "namereservation": {
    "version": 1,
    "name": "chipstensec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "6b2ff5b181e2788b5108592742d9214b0e9a33569e5e1ebad48e4000e9aad986",
    "referral": "",
    "nameid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty"
  },
    "identity":{
        "name":"chipstensec", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
b4e78320eb5ecc9d2ff2f3efd2f6fcea9182393658dc589d023825d0ae83b680
```

Wait for this transaction to be confirmed.

<h3>Step 3: Verify identity has been created</h3>

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chipstensec@"

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
    "name": "chipstensec",
    "identityaddress": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "revocationauthority": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "recoveryauthority": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 73188,
  "txid": "b4e78320eb5ecc9d2ff2f3efd2f6fcea9182393658dc589d023825d0ae83b680",
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

The options below will generate a `chipstensec` blockchain, as well as a gateway converter `Cashier.chipstensec`.

<h4 id="options-chipstensec">`definecurrency` options for `chipstensec` and justifications</h4>

- `"name":"CHIPSTENSEC"`

Name for our generated chain

- `"options": 264`

Defines our chain as a PBaaS chain, with `OPTION_PBAAS: 0x100 = 256`, and `OPTION_ID_REFERRALS: 8`

- `"currencies":["vrsctest"]`

Defines our chain as being based on existing `vrsctest` currency

- `"maxpreconversion":[0]`

Prevents users of `vrsctest` blockchain from preconverting `vrsctest` to `chipstensec` in during pre-launch period.  If we wanted to accept pre-launch conversions, we could set `minpreconversion` and `maxpreconversion` to specify required preconversion amounts for launch to proceed.

- `"conversions":[1]`

Specifies the desired conversion ratio for `vrsctest` to `chipstest`

- `"eras"`

Describes reward schedule on new `chipstensec` chain.  We want block subsidies to be zero, so that all emission comes from gateway converter.  Any additional issuance beyond that will be supply-neutral.

- `"notaries":["Notary1@","Notary2@","Notary3@"]

These notaries will be responsible for notarizing from `chipstensec` to `vrsctest` and vice versa.  These are required for cross-chain interaction.  Specified notaries here belong to identities owned by Verus foundation.

- `"nodes":[{...},{...}]`

Seed nodes for new PBaaS chain

- `"preallocations":[{"biz@":10000000},{"allbits@":10000000}]`

Pre-allocated amount from supply of new chain.  In this example, we are pre-allocating 20 million `chipstensec` coins to identities `biz@` and `allbits@` in equal amounts.  These could then be re-allocated based on snapshot from legacy CHIPS codebase.

<h4 id="options-gateway">`definecurrency` options for gateway converter</h4>

- `"gatewayconvertername":"Cashier"`

Defines a separate blockchain for gateway conversions named `Cashier.chipstensec`

- `"gatewayconverterissuance":1000000`

Specifies that our final 1 million `chipstensec` supply will be issued through the gateway converter, for a max supply of 21 million coins.

- `"currencies":["vrsctest","nexus","BTC","chipstensec"]`

Specifies a basket of currencies backing our gateway issuance.

- `"initialcontributions":[1000,2300,1.9,0]`

Specifies amounts of backing currencies to contribute on launch of gateway from our `chipstensec@` identity address.

- `"initialsupply":4000`

1:1 backing against `vrsctest` amount of 1000 in a basket of 4 currencies


<h3 id="chaingen-fund">Step 1: Send VRSCTEST, and Basket currencies to identity address</h3>

To generate a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.

For this chain, we will be generating both a PBaaS chain (`chipstensec@`), and a gatewayconverter chain (`Cashier.chipstensec`).  We will be initially contributing a basket of currencies to the gateway converter.  In example below, we are sending `15000 VRSCTEST`, `2 BTC`, and `2400 nexus` to fund the basket.

```
# send vrsctest
./verus -chain=vrsctest sendtoaddress chipstensec@ 15000
5f423ae417e84d98ac34a6aa3ea97da056717e80ab2a75a238f4a691e3b1f1b7

# send btc
./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chipstensec@","currency":"btc","amount":2}]'
opid-a862d7a2-8103-42f2-a5a4-472cbd1068a2

# send nexus
./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chipstensec@","currency":"nexus","amount":2500}]'
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
        "address": "chipstensec@",
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
        "address": "chipstensec@",
        "currency": "nexus",
        "amount": 2500
      }
    ]
  }
]
```

<h3 id="chaingen-define">Step 2: Define a currency with desired parameters</h3>

```
./verus -chain=vrsctest definecurrency '{"name":"chipstensec","options":264,"currencies":["vrsctest"],"maxpreconversion":[0], "conversions":[1],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["Notary1@","Notary2@","Notary3@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"},{"networkaddress":"149.56.13.160:12121"}],"preallocations":[{"biz@":10000000},{"allbits@":10000000}], "gatewayconvertername":"Cashier", "gatewayconverterissuance":1000000}' '{"currencies":["vrsctest","nexus","BTC","chipstensec"],"initialcontributions":[1000,2300,1.9,0],"initialsupply":4000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


<h3>Step 3: Send resulting raw transaction to network</h3>

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f89044d2a11176fba89fcd3d3743246a670d551fe52d7626e8c8216c6e67a5af2afbe00000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940e6757aaa419d1a18888744e3e77711e5b619a0373a5497be8a0556179dfdbb912cc597972ec31e8ad2ab3765e4bdbd72fa44885d2b2d355dda014445c7671c4effffffffa061520695d0aed585db4faacd928a543643ff241af36c91ef8fdfd5e17a21e200000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940c7c724fb935030ca95fc0fc2f6587d58d59194befa07fffc70daaa0c37dee0191fc355f2b1fca323ddeb6ac5cad8aa2acb5ec98180fae1a6d82f310d33e2bbeeffffffffaa8d13dda4c25de5c1ac4d0327e126d03ed01ec8706bcbc50a51b6e349e993f800000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc99405c7403865d24a21f721c38e553b8064e2d9964ba6c88edd2403264cf6a6d3bf43912024a0cb77c93ad3e67c4a312a1072c15721616dabffd08c501e12fa05c8affffffffb7f1b1e391a6f438a2752aab807e7156a07da93eaaa634ac984de817e43a425f01000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc994038155022caa1eb3ea5ec6f21f7ee7ce852f741b49e28012313072525d5a37b884e07107d25405252138b2392229d6194d4f44cbb29147ff69b7fd625bfca7502ffffffff0f0000000000000000fd250147040300010315049c64e2e85108b76c41a26c97f34ef757f93f325615049c64e2e85108b76c41a26c97f34ef757f93f325615049c64e2e85108b76c41a26c97f34ef757f93f3256cc4cd904030e010115049c64e2e85108b76c41a26c97f34ef757f93f32564c8403000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a6368697073746573743200009c64e2e85108b76c41a26c97f34ef757f93f32569c64e2e85108b76c41a26c97f34ef757f93f325600a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f010115049c64e2e85108b76c41a26c97f34ef757f93f32561b040310010115049c64e2e85108b76c41a26c97f34ef757f93f3256750000000000000000fdc3012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d960104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d6c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a43484950535445535432a6ef9ea235635e328124ff3429db9f9e91b64e2d9c64e2e85108b76c41a26c97f34ef757f93f32560100000001000000000080fa78000000000000000000028e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e0080c6a47e8d03006760a36ddeed08e1862f8cf5cf424f13636a0e4a0080c6a47e8d030000407a10f35a000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f5050000000000010000000000000000010000000000000000010000000000000000000000000003f9fc05c0d2aa4c1be97ef0e11fc1f9d17e11aefa5ff0a8b4d60c591a1cfd592a2304936f86af3fdacbb76c71819024b9e5ac9fd74257737cf4bd8c6a02a49faec70003aed6c100c9bfde8f009c8ca4939f00a49faec700bc8340bc83400743617368696572e1e1011e01000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001009c64e2e85108b76c41a26c97f34ef757f93f3256010000009c64e2e85108b76c41a26c97f34ef757f93f32560000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fdb9012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d8c0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d620101800300009c64e2e85108b76c41a26c97f34ef757f93f3256010002009c64e2e85108b76c41a26c97f34ef757f93f325601a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000082dcbd84cf9bff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f505000000000100000000000000000100000000000000000100000000010000000000000000e47d00000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000133134392e35362e31332e3136303a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000009c64e2e85108b76c41a26c97f34ef757f93f32569c64e2e85108b76c41a26c97f34ef757f93f32560080fa640000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a74000000000000ffffffff00750000000000000000fd9a012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d6d0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d430101000000210200009c64e2e85108b76c41a26c97f34ef757f93f32560743617368696572a6ef9ea235635e328124ff3429db9f9e91b64e2d9c64e2e85108b76c41a26c97f34ef757f93f32560100000001000000000080fa780000a0db215d0000000000407a10f35a000004a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40769c64e2e85108b76c41a26c97f34ef757f93f32560440787d0140787d0140787d0140787d0104000000000000000000000000000000000000000000000000000000000000000000000400e8764817000000007c118d35000000802b530b0000000000000000000000000400e8764817000000007c118d35000000802b530b00000000000000000000000000000000000000a49faec70003aed6c100750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001009c64e2e85108b76c41a26c97f34ef757f93f3256010000001697f4671639ed992a4d347064afb3e89c4da59a00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd7f022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d520204030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d280201800300001697f4671639ed992a4d347064afb3e89c4da59a010003001697f4671639ed992a4d347064afb3e89c4da59a04a6ef9ea235635e328124ff3429db9f9e91b64e2d5f846cd993507f8a47d65809198d6649bea6004d54852c4e9fb1d4c4291fc093e41ce2c7befa40769c64e2e85108b76c41a26c97f34ef757f93f32560440787d0140787d0140787d0140787d010400000000000000000000000000000000000000000000000000407a10f35a00008ad18dedbf00008ad18dedbf0000000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000407a10f35a000004000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000004a086010000000000a086010000000000a086010000000000a0860100000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000e47d00000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000009c64e2e85108b76c41a26c97f34ef757f93f32561697f4671639ed992a4d347064afb3e89c4da59a0080fa640000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000000ffffffff0075cbc6f44917000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d01a6ef9ea235635e328124ff3429db9f9e91b64e2d81f3ced0f02b05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b2004141697f4671639ed992a4d347064afb3e89c4da59a1697f4671639ed992a4d347064afb3e89c4da59a75204e000000000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d015f846cd993507f8a47d65809198d6649bea6004d85d882fbaa0a05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b2004141697f4671639ed992a4d347064afb3e89c4da59a1697f4671639ed992a4d347064afb3e89c4da59a75204e000000000000961a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c77040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5b0154852c4e9fb1d4c4291fc093e41ce2c7befa4076d9cec91705a6ef9ea235635e328124ff3429db9f9e91b64e2d809b2004141697f4671639ed992a4d347064afb3e89c4da59a1697f4671639ed992a4d347064afb3e89c4da59a750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f009c64e2e85108b76c41a26c97f34ef757f93f32567500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec7009c64e2e85108b76c41a26c97f34ef757f93f325675f55c4578580000007a1b040300010115049c64e2e85108b76c41a26c97f34ef757f93f3256cc4c5a040309010115049c64e2e85108b76c41a26c97f34ef757f93f32563e86fefeff010254852c4e9fb1d4c4291fc093e41ce2c7befa4076e9dc9700000000005f846cd993507f8a47d65809198d6649bea6004d762eaaa4040000007500000000f97d00000000000000000000000000

# resulting txid
fe8ac7c7b3c543ff82becba1a27efd244690dfa7bb737946a31cd953dfcd3bd6
```

<h3 id="chaingen-verify">Step 4: Verify currency generation was succesful</h3>

```
./verus -chain=vrsctest getcurrency chipstensec
```

If successful, output should show similar to the following:

```
{
  "version": 1,
  "options": 264,
  "name": "chipstensec",
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
  "fullyqualifiedname": "chipstensec",
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

---

<h2 id="launch">Launching a PBaaS chain</h4>

<h3 id="launch-firstblock">Merge mine the first block on generated chain</h3>

Now that we have defined our chain's currency, generated and committed the launch parameters...  we need to merge mine at least the first block in conjunction with the launch system chain.

Only the first block must be merge mined.  Afterward, we can choose to merge mine or mine the newly launched chain independently.

To do this, we must be already mining on `VRSCTEST` in our example.  This can be performed with the following startup options (for solo mining):

```
./verusd -chain=vrsctest -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

Next, we must spin up our `chipstensec` daemon, and start mining.  We can either: a.) fetch the address generated on wallet initialization, or b.) import our WIF private key from `vrsctest` chain.

```
# Start chipstensec daemon
./verusd -chain=chipstensec &

# option a.) Fetch address generated on init
./verus -chain=chipstensec getaddressesbyaccount ""

# option b.) import WIF
./verus -chain=chipstensec importprivkey <WIF here>
```

Once we've fetched our new address, or imported a known address, we can restart our `chipstensec` daemon to begin merge mining with parent `vrsctest`:

```
# Stop chipstensec daemon
./verus -chain=chipstensec stop

# Restart with mining options
./verusd -chain=chipstensec -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

If successful, you should see output in your terminal:

```
Merge mining chipstensec with vrsctest as the hashing chain
Found block 2885
staking reward 0.00010000 chipstensec!
POS hash: 00000000007fca5a2624d9276d2ddd7074f68fd32b9f6c81904d96b4a12f6301
target:   0000000002e3ae00000000000000000000000000000000000000000000000000

Block 2885 added to chain
```

Similar messages should be seen in `vrsctest` daemon output.  Once the first block is mined, running `vrsctest` daemon at the same time is optional.


