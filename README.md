# zokrates_vanet

## Requirement

This project is built on Zokrates, so you should first install the ZoKrates CLI:

```bash
curl -LSfs get.zokrat.es | sh
```

Have a look at the [documentation](https://zokrates.github.io/) for more information about using ZoKrates.

## Primitives

- RA has a public-private key pair `(mpk, msk)`

- A vehicle has a public-private key pair `(pk, sk)`

- RA generates certificates for vehicles

```
CertGen(msk, pk) -> cert
```

- RA generates `proof.key` and `verification.key` of zkSNARKs

```
KeyGen() ->  (proof.key, verification.key)
```

- Vehicles prove message `m` and pseudo random address `ppk`

```
Auth(ppk||m, mpk, sk, pk, cert, proof.key) -> (ppk||m, mpk, pi=(t, η))
```

where `t=Enc_mpk(pk||ppk||m)`, `η` is the zkSNARKs proof.

- RSUs verify the proof

```
Verify(ppk||m, mpk, pi, verification.key) -> {0, 1}
```

- RA can reveal the real identity of vehicles by decrypting `t`