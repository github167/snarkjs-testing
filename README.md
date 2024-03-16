Playground:

template Multiplier2() {
    signal input a;
    signal input b;
    signal output c;
    c <== a*b;
 }

component main = Multiplier2();
EOF

circom multiplier2.circom --r1cs --wasm --sym --c

# generate wtns
cd multiplier2_js
cat << EOF > input.json
{"a": "3", "b": "11"}
EOF

node generate_witness.js multiplier2.wasm input.json witness.wtns

# generate powersoftau
snarkjs powersoftau new bn128 6 pot12_0000.ptau -v
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v -e="some text"
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v

# setup keys
snarkjs plonk setup ../multiplier2.r1cs pot12_final.ptau circuit_final.zkey
snarkjs zkey export verificationkey circuit_final.zkey verification_key.json

# generate prove and verify
snarkjs plonk prove circuit_final.zkey witness.wtns proof.json public.json
snarkjs plonk verify verification_key.json public.json proof.json

# fullprove
rm -f public.json proof.json
snarkjs plonk fullprove input.json multiplier2.wasm circuit_final.zkey proof.json public.json
snarkjs plonk verify verification_key.json public.json proof.json

snarkjs zkey export solidityverifier circuit_final.zkey verifier.sol
snarkjs generatecall

```
 
2. ZkREPL
https://zkrepl.dev/

