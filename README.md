# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.2-1), with the goal of providing a reproducible workflow for future use.

**Contents:**

- [Compiling from Source](#compile)
- [Generating an ID for use on Verus Testnet](#idgen)

---

<h2 id="compile">Compiling from Source</h2>

**Step 1:** Clone, build dependencies:

```
cd ~
git clone https://github.com/VerusCoin/VerusCoin.git verus
cd ~/verus/depends
make -j4
```

Wait for `depends` to finish building.

**Step 2:** Configure build enviroment, and build from source:

```
cd ~/verus
./autogen.sh
CONFIG_SITE=$PWD/depends/x86_64-unknown-linux-gnu/share/config.site ./configure
make -j4
```

If all steps completed successfully, you should have resulting binaries located in `~/verus/src`.

**Step 3:** Launch Verus daemon

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

**Step 1:** Register name commitment 

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

**Step 2:** Register identity

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

**Step 3:** Verify identity has been created

Identities can be located by name, or by i-address:

By name:

```
./verus -chain=vrsctest getidentity "chipstest0@"
```

By i-address:

```
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
