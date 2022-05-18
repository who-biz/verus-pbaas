# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.2-2), with the goal of providing a reproducible workflow for future use.

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
./verus -chain=vrsctest registernamecommitment "chipstest0" "RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "f4655a6290e39ed2ff36da6a1ec7f25eacf1d142cb00fa7862cb85125325bcfa",
  "namereservation": {
    "version": 1,
    "name": "chipstest0",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "7aa8c4ef5ed9d2d75db2013386e64d5ce66d22d52b9a8d2762e6bbe7dcc2a7fe",
    "referral": "",
    "nameid": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA"
  }
}
```

**Step 2: Register identity**

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{  
  "txid": "f4655a6290e39ed2ff36da6a1ec7f25eacf1d142cb00fa7862cb85125325bcfa",
  "namereservation": {
    "version": 1,
    "name": "chipstest0",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "7aa8c4ef5ed9d2d75db2013386e64d5ce66d22d52b9a8d2762e6bbe7dcc2a7fe",
    "referral": "",
    "nameid": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA"
  },
  "identity": {
        "name":"chipstest0", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
f2e5b040dd18ce691e47de60cf9cc5d867769b5d09d25b6a7ae2ea9e0fa71059
```
Wait for this transaction to be confirmed.

**Step 3: Verify identity has been created**

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chipstest0@"

# By i-address
./verus -chain=vrsctest getidentity iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA
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
    "name": "chipstest0",
    "identityaddress": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "revocationauthority": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
    "recoveryauthority": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 10037,
  "txid": "f2e5b040dd18ce691e47de60cf9cc5d867769b5d09d25b6a7ae2ea9e0fa71059",
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
./verus -chain=vrsctest definecurrency '{"name":"CHIPSTEST0","options":264,"currencies":["VRSCTEST"],"conversions":[1.00],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["Notary1@","Notary2@","Notary3@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"}],"gatewayconvertername":"Cashier","gatewayconverterissuance":1000000}' '{"currencies":["VRSCTEST","CHIPSTEST0","BTC"],"initialcontributions":[50,0,0],"initialsupply":3000000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


**Step 3: Send resulting raw transaction to network**

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f89035910a70f9eeae27a6a5bd2095d9b7667d8c59ccf60de471e69ce18dd40b0e5f200000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940273e444e3df359a664283c83d7badbf98423c196e3384265985a99c2dde2c38f2ebdf2fd06fd0a7e993a96135de9532962d33c869eea90d4b08c6192bd5f61f1ffffffffc6a401bfc226a36b15d2847e0639ea0f5f37cfb698a1598ac684883042e0cf6301000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc99403cb655726bca50c481a8dc819d2174aa29041ce8359c8ed3c0d79704054417a917fb69aa24bb32192a1a07fa5ed40cb270368aa7fcd9044f45f87d8390114dadffffffffc1bcdafaa3ebf1b7ea53173e84c54163304864e6f26fd9796f26b3f6e3b93d9601000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940b20c522f2bc24e2fea12529cfc18af4387e50067315970e0fa7e275e457a948e6e6372431ea7618fb37848b3b2279855c6f7b2269cf2ad1aac6e64e2a1d65460ffffffff0d0000000000000000fd2501470403000103150474200aff604c8459f68a2ea267f629815d6d2071150474200aff604c8459f68a2ea267f629815d6d2071150474200aff604c8459f68a2ea267f629815d6d2071cc4cd904030e0101150474200aff604c8459f68a2ea267f629815d6d20714c8403000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a63686970737465737430000074200aff604c8459f68a2ea267f629815d6d207174200aff604c8459f68a2ea267f629815d6d207100a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f0101150474200aff604c8459f68a2ea267f629815d6d20711b0403100101150474200aff604c8459f68a2ea267f629815d6d2071750000000000000000fd83012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d560104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d2c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0a43484950535445535430a6ef9ea235635e328124ff3429db9f9e91b64e2d74200aff604c8459f68a2ea267f629815d6d20710100000001000000000080ad180000000000000000000000407a10f35a000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f505000000000000010000000000000000010000000000000000000000000003f9fc05c0d2aa4c1be97ef0e11fc1f9d17e11aefa5ff0a8b4d60c591a1cfd592a2304936f86af3fdacbb76c71819024b9e5ac9fd74257737cf4bd8c6a02a49faec70003aed6c100c9bfde8f009c8ca4939f00a49faec700bc8340bc83400743617368696572e1e1011e01000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a0100010074200aff604c8459f68a2ea267f629815d6d20710100000074200aff604c8459f68a2ea267f629815d6d20710000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fd90012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d630104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d3901018003000074200aff604c8459f68a2ea267f629815d6d20710100020074200aff604c8459f68a2ea267f629815d6d207101a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000095ddb082e7ff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f505000000000100000000000000000100000000000000000100000000010000000000000000045700000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000011435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d000000000000000000000000000000000000000000000000000000000000000074200aff604c8459f68a2ea267f629815d6d207174200aff604c8459f68a2ea267f629815d6d20710080ad040000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a74000000000000ffffffff00750000000000000000fd6a012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d3d0104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d1301010000002102000074200aff604c8459f68a2ea267f629815d6d20710743617368696572a6ef9ea235635e328124ff3429db9f9e91b64e2d74200aff604c8459f68a2ea267f629815d6d20710100000001000000000080ad180000c06e31d91001000000407a10f35a000003a6ef9ea235635e328124ff3429db9f9e91b64e2d74200aff604c8459f68a2ea267f629815d6d207154852c4e9fb1d4c4291fc093e41ce2c7befa40760356a0fc0155a0fc0155a0fc010300000000000000000000000000000000000000000000000000000300f2052a01000000000000000000000000000000000000000300f2052a010000000000000000000000000000000000000000000000000000a49faec70003aed6c100750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a0100010074200aff604c8459f68a2ea267f629815d6d20710100000086bdb46e88756df3ccf391d73b87d4d87edce58500000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd25022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4df80104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34dce01018003000086bdb46e88756df3ccf391d73b87d4d87edce5850100030086bdb46e88756df3ccf391d73b87d4d87edce58503a6ef9ea235635e328124ff3429db9f9e91b64e2d74200aff604c8459f68a2ea267f629815d6d207154852c4e9fb1d4c4291fc093e41ce2c7befa40760356a0fc0155a0fc0155a0fc0103000000000000000000407a10f35a00000000000000000000c39a928ab9ff0000c39a928ab9ff00000000000000000000000000000000000000000000000000000000000000000003000000000000000000407a10f35a0000000000000000000003000000000000000000000000000000000000000000000000030000000000000000000000000000000000000000000000000363000000000000006400000000000000640000000000000003000000000000000000000000000000000000000000000000030000000000000000000000000000000000000000000000000300000000000000000000000003000000000000000000000000000000000000000000000000045700000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d000000000000000000000000000000000000000000000000000000000000000074200aff604c8459f68a2ea267f629815d6d207186bdb46e88756df3ccf391d73b87d4d87edce5850080ad040000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000000ffffffff00752854192a01000000971a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c78040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5c01a6ef9ea235635e328124ff3429db9f9e91b64e2d91cfe38b0805a6ef9ea235635e328124ff3429db9f9e91b64e2d809b20041486bdb46e88756df3ccf391d73b87d4d87edce58586bdb46e88756df3ccf391d73b87d4d87edce585750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f0074200aff604c8459f68a2ea267f629815d6d20717500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec70074200aff604c8459f68a2ea267f629815d6d207175d8570ad20500000024050403000000cc1b0403000101150474200aff604c8459f68a2ea267f629815d6d20717500000000195700000000000000000000000000
```

If succesful, a txid will display as output.  Wait for this transaction to be included in a block.


**Step 4: Verify currency generation was succesful**

```
./verus -chain=vrsctest getcurrency chipstest0
```

If successful, output should show similar to the following:

```
./verus -chain=vrsctest getcurrency CHIPSTEST0
{
  "version": 1,
  "options": 264,
  "name": "CHIPSTEST0",
  "currencyid": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
  "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "systemid": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
  "notarizationprotocol": 1,
  "proofprotocol": 1,
  "launchsystemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "startblock": 22296,
  "endblock": 0,
  "currencies": [
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq"
  ],
  "conversions": [
    1.00000000
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
  "gatewayconverterid": "iFkyEhkUx7uUXpJexLU2gphsaCVWUGrdzS",
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
  "currencyidhex": "71206d5d8129f667a22e8af659844c60ff0a2074",
  "fullyqualifiedname": "CHIPSTEST0",
  "currencynames": {
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": "VRSCTEST"
  },
  "definitiontxid": "ae824717143144e6198c3d8def80c12bc271bc9750790a9437c84740028b76a2",
  "definitiontxout": 1,
  "bestheight": 22280,
  "lastconfirmedheight": 22280,
  "bestcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
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
    "supply": 1000000.00000000,
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
    "currencyid": "iE4YCiJTGnYZdHBkDbzPka3GsNPoAXmoTA",
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
    "supply": 1000000.00000000,
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


