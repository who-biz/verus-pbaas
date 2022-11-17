# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.5-2), with the goal of providing a reproducible workflow for future use.

**Contents:**

- [Compiling from source](#compile)
- [Generating an ID for use on Verus Testnet](#idgen)
- [Generating a PBaaS chain](#chaingen)
  - [Conceptual description of chosen `definecurrency` options](#chaingen-concept)
    - [Background on CHIPS](#background)
    - [Itemized options for chips chain](#options-chips)
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
./verus -chain=vrsctest registernamecommitment chips RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
  "txid": "2f405700b51c4ef8003a2ea554ea7cd0358c113920018b83bc4dd8d0737eab4a",
  "namereservation": {
    "version": 1,
    "name": "chips",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "67291251c09309dbf9e71f4b526179a15be27d8d3c7a0144bbf452446565c6cb",
    "referral": "",
    "nameid": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS"
  }
```

<h3>Step 2: Register identity</h3>

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{
  "txid": "2f405700b51c4ef8003a2ea554ea7cd0358c113920018b83bc4dd8d0737eab4a",
  "namereservation": {
    "version": 1,
    "name": "chips",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "67291251c09309dbf9e71f4b526179a15be27d8d3c7a0144bbf452446565c6cb",
    "referral": "",
    "nameid": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS"
  },
    "identity":{
        "name":"chips", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
c0db5980f6307ddf6b8bc04858c4e4b35dd524363ef04d88a18a74aa6a6415ee
```

Wait for this transaction to be confirmed.

<h3>Step 3: Verify identity has been created</h3>

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chips@"

# By i-address
./verus -chain=vrsctest getidentity i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS
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
    "name": "chips",
    "identityaddress": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "contentmultimap": {
    },
    "revocationauthority": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
    "recoveryauthority": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 29929,
  "txid": "c0db5980f6307ddf6b8bc04858c4e4b35dd524363ef04d88a18a74aa6a6415ee",
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

The options below will generate a `chips` blockchain, as well as a gateway converter `bridge.chips`.

<h4 id="options-chips">`definecurrency` options for `chips` and justifications</h4>

- `"name":"chips"`

Name for our generated chain

- `"options": 264`

Defines our chain as a PBaaS chain, with `OPTION_PBAAS: 0x100 = 256`, and `OPTION_ID_REFERRALS: 8`

- `"currencies":["vrsctest"]`

Defines our chain as being based on existing `vrsctest` currency

- `"maxpreconversion":[0]`

Prevents users of `vrsctest` blockchain from preconverting `vrsctest` to `chips` in during pre-launch period.  If we wanted to accept pre-launch conversions, we could set `minpreconversion` and `maxpreconversion` to specify required preconversion amounts for launch to proceed.

- `"conversions":[1]`

Specifies the desired conversion ratio for `vrsctest` to `chips`

- `"eras"`

Describes reward schedule on new `chips` chain.  We want block subsidies to be zero, so that all emission comes from gateway converter.  Any additional issuance beyond that will be supply-neutral.

- `"notaries":["biz@","BizNotary@","BizNotary1@"]`

These notaries will be responsible for notarizing from `chips` to `vrsctest` and vice versa.  These are required for cross-chain interaction.  Specified notaries here are all owned by Biz's control address.

- `"nodes":[{...},{...}]`

Seed nodes for new PBaaS chain

- `"preallocations":[{"biz@":2950000}]`

Pre-allocated amount from supply of new chain.  In this example, we are pre-allocating 20,900,000 `chips` coins to identities `biz@`, `BizNotary@`, and `BizNotary2@`.  These can then be re-allocated based on snapshot from legacy CHIPS codebase.

<h4 id="options-gateway">`definecurrency` options for gateway converter</h4>

- `"gatewayconvertername":"bridge"`

Defines a separate blockchain for gateway conversions named `bridge.chips`

- `"gatewayconverterissuance":50000`

Specifies that our final 100,000 `chips` supply will be issued through the gateway converter, for a max supply of 21 million coins.

- `"currencies":["vrsctest","chips"]`

Specifies a basket of currencies backing our gateway issuance.

- `"initialcontributions":[1000,0]`

Specifies amounts of backing currencies to contribute on launch of gateway from our `chips@` identity address.

- `"initialsupply":2000`

1:1 backing against `vrsctest` amount of 1000 in a basket of 2 currencies


<h3 id="chaingen-fund">Step 1: Send VRSCTEST, and Basket currencies to identity address</h3>

To generate a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.

For this chain, we will be generating both a PBaaS chain (`chips@`), and a gatewayconverter chain (`bridge.chips`).  We will be initially contributing a basket of currencies to the gateway converter.  In example below, we are sending `14600 VRSCTEST` to fund the basket.

```
# send vrsctest
./verus -chain=vrsctest sendtoaddress chips@ 14600
e26b129e25f8372a9190ff4d88a76601fd85631fa8cae41061bf067a4c07095d

#DISREGARD BELOW, LEFT AS EXAMPLE. NOT NECESSARY FOR UPDATED PROCEDURE ABOVE

# send kmd
#./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chips@","currency":"kmd","amount":120}]'
#opid-90c1e936-830f-416b-8532-dd75fc14ad85
```

~~We also want to verify that the resulting operation id for 'kmd` transfer succeeded:~~

```
#./verus -chain=vrsctest z_getoperationresult '["opid-90c1e936-830f-416b-8532-dd75fc14ad85"]'
#[
#  {
#    "id": "opid-e96fa394-7a8d-4782-a05b-af9ae3852fce",
#    "status": "success",
#    "creation_time": 1659457274,
#    "result": {
#      "txid": "34f81f6ad827f42788f1683a57895d8e071a6c04758e40870f005e6efa418f11"
#    },
#    "execution_secs": 0.305115012,
#    "method": "sendcurrency",
#    "params": [
#      {
#        "address": "chips@",
#        "currency": "kmd",
#        "amount": 100
#      }
#    ]
#  }
#]
```

<h3 id="chaingen-define">Step 2: Define a currency with desired parameters</h3>

```
./verus -chain=vrsctest definecurrency '{"name":"chips","options":264,"currencies":["vrsctest"],"maxpreconversion":[0], "conversions":[1],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["biz@","BizNotary@","BizNotary1@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"},{"networkaddress":"149.56.13.160:12121"}],"preallocations":[{"biz@":29950000}], "gatewayconvertername":"bridge", "gatewayconverterissuance":50000}' '{"currencies":["vrsctest","chips"],"initialcontributions":[1000,0],"initialsupply":2000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


<h3>Step 3: Send resulting raw transaction to network</h3>

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f8902ee15646aaa748aa1884df03e3624d55db3e4c45848c08b6bdf7d30f68059dbc000000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940f7cab3fbd473748e5a0a112f3e48e3ecebafb650eac777aaea25e0714ab927980c974058ae8f2b57552c8a1ddedc98fdedfc341a195d27895c3768bb56c24439ffffffff5d09074c7a06bf6110e4caa81f6385fd0166a7884dff90912a37f8259e126be201000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940ca6ab9b812daa04ad3066748fb991a77cf177d09e5c7a25257a21b10a7e5adaf726ea1641c426b85f39afd075a6886c3ee80326de366b3cdb232a6166fb71e93ffffffff0d0000000000000000fd200147040300010315040d6e14a9882b5b157348af5001beb8fdefedeec115040d6e14a9882b5b157348af5001beb8fdefedeec115040d6e14a9882b5b157348af5001beb8fdefedeec1cc4cd404030e010115040d6e14a9882b5b157348af5001beb8fdefedeec14c7f03000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d05636869707300000d6e14a9882b5b157348af5001beb8fdefedeec10d6e14a9882b5b157348af5001beb8fdefedeec100a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f010115040d6e14a9882b5b157348af5001beb8fdefedeec11b040310010115040d6e14a9882b5b157348af5001beb8fdefedeec1750000000000000000fdbe012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d910104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d67010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d056368697073a6ef9ea235635e328124ff3429db9f9e91b64e2d0d6e14a9882b5b157348af5001beb8fdefedeec101000000010000000000000000000000000000000000000000000000000080e907000000000000000000018e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e00301ac7efa30a00005039278c04000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f50500000000000100000000000000000100000000000000000100000000000000000000000000038e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e6df4164224d7eb36df23a3fccf2cd6ae7b5827dc4d3213c88cd5b55c4e16c151a34a63f37e52f1ae02a49faec70003f98800c9bfde8f009c8ca4939f00a49faec700bc8340bc8340066272696467650fff001e3c0000002d0000000a0001000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001000d6e14a9882b5b157348af5001beb8fdefedeec1010000000d6e14a9882b5b157348af5001beb8fdefedeec10000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fdb9012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d8c0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d620102800300000d6e14a9882b5b157348af5001beb8fdefedeec1010002000d6e14a9882b5b157348af5001beb8fdefedeec101a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000084a98ebdf1ccff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f505000000000100000000000000000100000000000000000100000000010000000000000000f37400000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000133134392e35362e31332e3136303a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000000d6e14a9882b5b157348af5001beb8fdefedeec10d6e14a9882b5b157348af5001beb8fdefedeec10000ffffffff000000000080e87301a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a740000000000750000000000000000fd4b012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d1e0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54cf501000000210200000d6e14a9882b5b157348af5001beb8fdefedeec106627269646765a6ef9ea235635e328124ff3429db9f9e91b64e2d0d6e14a9882b5b157348af5001beb8fdefedeec1010000000100000000000d6e14a9882b5b157348af5001beb8fdefedeec180e9070000d0ed902e00000000005039278c04000002a6ef9ea235635e328124ff3429db9f9e91b64e2d0d6e14a9882b5b157348af5001beb8fdefedeec10280f0fa0280f0fa02020000000000000000000000000000000000000200e876481700000000000000000000000200e8764817000000000000000000000000000000000000a49faec70003f98800750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a010001000d6e14a9882b5b157348af5001beb8fdefedeec101000000f8d5a8b1a192be2e5e2f661b17b17583f64dbb7200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fdc7012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d9a0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d70010280030000f8d5a8b1a192be2e5e2f661b17b17583f64dbb7201000300f8d5a8b1a192be2e5e2f661b17b17583f64dbb7202a6ef9ea235635e328124ff3429db9f9e91b64e2d0d6e14a9882b5b157348af5001beb8fdefedeec10280f0fa0280f0fa02020000000000000000005039278c04000084e886b69f000084e886b69f000000000000000000000000000000000000000000000000000000000000000000020000000000000000005039278c0400000200000000000000000000000000000000020000000000000000000000000000000002a086010000000000a086010000000000020000000000000000000000000000000002000000000000000000000000000000000200000000000000000200000000000000000000000000000000f37400000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d00000000000000000000000000000000000000000000000000000000000000000d6e14a9882b5b157348af5001beb8fdefedeec1f8d5a8b1a192be2e5e2f661b17b17583f64dbb720000ffffffff000000000080e87301a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000075cbc6f44917000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d01a6ef9ea235635e328124ff3429db9f9e91b64e2d81f3ced0f02b05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b200414f8d5a8b1a192be2e5e2f661b17b17583f64dbb72f8d5a8b1a192be2e5e2f661b17b17583f64dbb72750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f000d6e14a9882b5b157348af5001beb8fdefedeec17500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec7000d6e14a9882b5b157348af5001beb8fdefedeec175356916284f00000024050403000000cc1b040300010115040d6e14a9882b5b157348af5001beb8fdefedeec17500000000087500000000000000000000000000

# resulting txid
ec8ca436fc152b6666765879aa12939e2fdf136d6e5d0dd353aabef752c2022b
```

<h3 id="chaingen-verify">Step 4: Verify currency generation was succesful</h3>

```
./verus -chain=vrsctest getcurrency chips
```

If successful, output should show similar to the following:

```
{
  "version": 1,
  "options": 264,
  "name": "chips",
  "currencyid": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
  "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "systemid": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
  "notarizationprotocol": 1,
  "proofprotocol": 1,
  "launchsystemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "startblock": 29959,
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
      "iGTdav42siHgX96ybWn3pRTFxQgiUpZ9K9": 29950000.00000000
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
  "gatewayconverterid": "iSAExwDHrLYKi5smLAdrygifgsovtf3fVv",
  "gatewayconvertername": "bridge",
  "initialtarget": "000000ff0f000000000000000000000000000000000000000000000000000000",
  "blocktime": 60,
  "powaveragingwindow": 45,
  "notarizationperiod": 10,
  "eras": [
    {
      "reward": 0,
      "decay": 0,
      "halving": 0,
      "eraend": 0
    }
  ],
  "currencyidhex": "c1eeedeffdb8be0150af4873155b2b88a9146e0d",
  "fullyqualifiedname": "chips",
  "currencynames": {
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": "VRSCTEST"
  },
  "definitiontxid": "ec8ca436fc152b6666765879aa12939e2fdf136d6e5d0dd353aabef752c2022b",
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
  "lastconfirmedheight": 29939,
  "lastconfirmedtxid": "ec8ca436fc152b6666765879aa12939e2fdf136d6e5d0dd353aabef752c2022b",
  "lastconfirmedcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
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
    "supply": 30000000.00000000,
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
  "bestheight": 29939,
  "besttxid": "ec8ca436fc152b6666765879aa12939e2fdf136d6e5d0dd353aabef752c2022b",
  "bestcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "i4hY67UdHJcbj4DP2xiMtUZRNhLq4BsbsS",
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
    "supply": 30000000.00000000,
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

Next, we must spin up our `chips` daemon, and start mining.  We can either: a.) fetch the address generated on wallet initialization, or b.) import our WIF private key from `vrsctest` chain.

```
# Start chips daemon
./verusd -chain=chips &

# option a.) Fetch address generated on init
./verus -chain=chips getaddressesbyaccount ""

# option b.) import WIF
./verus -chain=chips importprivkey <WIF here>
```

Once we've fetched our new address, or imported a known address, we can restart our `chips` daemon to begin merge mining with parent `vrsctest`:

```
# Stop chips daemon
./verus -chain=chips stop

# Restart with mining options
./verusd -chain=chips -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

If successful, you should see output in your terminal:

```
Merge mining chips with vrsctest as the hashing chain
Found block 2885
staking reward 0.00010000 chips!
POS hash: 00000000007fca5a2624d9276d2ddd7074f68fd32b9f6c81904d96b4a12f6301
target:   0000000002e3ae00000000000000000000000000000000000000000000000000

Block 2885 added to chain
```

Similar messages should be seen in `vrsctest` daemon output.  Once the first block is mined, running `vrsctest` daemon at the same time is optional.


