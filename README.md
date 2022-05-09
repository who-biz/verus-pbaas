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

---

<h2 id="idgen">Generate an ID for use on Verus Testnet</h2>