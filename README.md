Playground:

- https://www.katacoda.com/scenario-examples/courses/environment-usages/nodejs
- https://codedamn.com/online-compiler/node#start

1. run the following script
```
# install circom
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf > install_cargo.sh
chmod u+x install_cargo.sh
./install_cargo.sh -y
source "$HOME/.cargo/env"
git clone https://github.com/iden3/circom.git
cd circom

cargo build --release
cargo install --path circom

# generate wasm
npm install -g snarkjs

cat << EOF > multiplier2.circom
pragma circom 2.0.0;

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
snarkjs powersoftau verify pot12_final.ptau

# phase 2
snarkjs groth16 setup ../multiplier2.r1cs pot12_final.ptau multiplier2_0000.zkey
snarkjs zkey contribute multiplier2_0000.zkey multiplier2_0001.zkey --name="1st Contributor Name" -v -e="some text"
snarkjs zkey verify ../multiplier2.r1cs pot12_final.ptau multiplier2_0001.zkey

# setup keys
snarkjs zkey export verificationkey multiplier2_0001.zkey verification_key.json

# generate prove and verify
snarkjs groth16 prove multiplier2_0001.zkey witness.wtns proof.json public.json
snarkjs groth16 verify verification_key.json public.json proof.json

# generate fullprove and verify
snarkjs groth16 fullprove input.json multiplier2.wasm multiplier2_0001.zkey proof.json public.json
snarkjs groth16 verify verification_key.json public.json proof.json

# generate sol
snarkjs zkey export solidityverifier multiplier2_0001.zkey verifier.sol
snarkjs generatecall

```
 
2. ZkREPL
https://zkrepl.dev/

