# AAVB: Anonymous yet Accountable VANET System atop Blockchain

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

## Implementation and Experiments

### source code

- [AAVB](https://github.com/imtypist/jsnark)
- [group/ring signature](https://github.com/imtypist/group-signature-interface)

### experiment setting

- Here we used `jsnark` to implement zkSNARKs, according to the performance comparison of three classic zero knowledge proof (ZKP) algorithms provided in `libsnark`, we chose the better one [Groth16](https://github.com/akosba/libsnark/tree/master/libsnark/zk_proof_systems/ppzksnark) as our underlying ZKP technology.
- The experiment setting is that using a virtual machine (VM) `ubuntu14.04` running on VirtualBox, the number of VM's CPU core is 2, the size of memory is 4GB; The localhost CPU is `Intel i7-7700`, the size of memory is 16GB.
- In this experiment, the implemented group signature scheme is [BBS04](http://crypto.stanford.edu/~dabo/abstracts/groupsigs.html) and the ring signature scheme is [LSAG](https://www.semanticscholar.org/paper/Linkable-Spontaneous-Anonymous-Group-Signature-for-Liu-Wei/3c63f7c90d79593fadfce16d54078ec1850bedc9).


### performance

#### zk-SNARK

The RSA key length is set to 2048 bits, which is safe enough currently.

- [rsa2048_encryption.log](https://github.com/imtypist/jsnark/blob/master/rsa2048_encryption.log)
- [rsa2048_sha256_sig_verify.log](https://github.com/imtypist/jsnark/blob/master/rsa2048_sha256_sig_verify.log)
- [vanet_rsa2048.log](https://github.com/imtypist/jsnark/blob/master/vanet_rsa2048.log)

[VANETCircuitGenerator](https://github.com/imtypist/jsnark/blob/master/JsnarkCircuitBuilder/src/examples/generators/rsa/VANETCircuitGenerator.java) is the circuit generator code of VANET.

The output below is the testcase for VANET, the example input is a pseudo random address concatenating a piece of GPS raw data.

```bash
Running Circuit Generator for < vanet_rsa2048 >
Circuit Generation Done for < vanet_rsa2048 >  
 	 Total Number of Constraints :  414319

PlainText:0xd91c747b4a76B8013Aa336Cbc52FD95a7a9BD3D9$GPRMC,092927.000,A,2235.9058,N,11400.0518,E,0.000,74.11,151216,,D*49
Running Circuit Evaluator for < vanet_rsa2048 >
	[output] Value of Wire # 1277072 (Is Signature valid?) :: 1
	[output] Value of Wire # 2226921 (Output cipher text[0]) :: 1251022490261258172529308360859369551837157967434844421554503068336263232
	[output] Value of Wire # 2226922 (Output cipher text[1]) :: 782513615157148428896018740256285560893017553685924761331573477021861754
	[output] Value of Wire # 2226923 (Output cipher text[2]) :: 297893711879754138931627910652983846133902364790173190431445656834653104
	[output] Value of Wire # 2226924 (Output cipher text[3]) :: 1062246178463496446723238490343351688931627517611251013590768981856952089
	[output] Value of Wire # 2226925 (Output cipher text[4]) :: 283619758270485012554111906691482726594266063316848294719061553920244069
	[output] Value of Wire # 2226926 (Output cipher text[5]) :: 1605604628605955420311218435655442487833758868828180874969980792959738819
	[output] Value of Wire # 2226927 (Output cipher text[6]) :: 1765324337636077192622074629173695556494918446370154704523581054010828502
	[output] Value of Wire # 2226928 (Output cipher text[7]) :: 229242121535333726124694280674701732758091959395234671423966224579886508
	[output] Value of Wire # 2226929 (Output cipher text[8]) :: 41995595892867847343690929056587572862
Circuit Evaluation Done for < vanet_rsa2048 >

```


#### group/ring signature

- [test_group_signature.log](https://github.com/imtypist/group-signature-interface/blob/master/test_group_signature.log)
- [test_ring_signature.log](https://github.com/imtypist/group-signature-interface/blob/master/test_ring_signature.log)

> Note that maximum ring size is hard coded as 32 in [FISCO-BCOS/group-signature-server](https://github.com/FISCO-BCOS/group-signature-server/). Here I set it to 1024 for testing a larger ring size. You can change it by yourselves.

- [test_revoke_member_1024.log (partial)](https://github.com/imtypist/group-signature-interface/blob/master/test_revoke_member_1024.log)
- [test_revoke_member_512.log (partial)](https://github.com/imtypist/group-signature-interface/blob/master/test_revoke_member_512.log)
- [test_revoke_member_256.log](https://github.com/imtypist/group-signature-interface/blob/master/test_revoke_member_256.log)
- [test_revoke_member_64.log](https://github.com/imtypist/group-signature-interface/blob/master/test_revoke_member_64.log)
- [test_revoke_member_32.log](https://github.com/imtypist/group-signature-interface/blob/master/test_revoke_member_32.log)

In this experiment result, I added the performance of `revoke` and `update member private key` operations. To achieve that, I implemented `revoke_member` and `revoke_update_private_key` RPC interfaces, which are not yet implemented in the original repository. If you want to use them directly, you can git clone from [imtypist/group-signature-server](https://github.com/imtypist/group-signature-server).
