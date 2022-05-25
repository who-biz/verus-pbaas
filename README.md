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
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


**Step 3: Send resulting raw transaction to network**

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
```

If succesful, a txid will display as output.  Wait for this transaction to be included in a block.

Resulting txid: ``


**Step 4: Verify currency generation was succesful**

```
./verus -chain=vrsctest getcurrency chipstest2
```

If successful, output should show similar to the following:

```
```


