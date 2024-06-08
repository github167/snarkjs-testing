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


