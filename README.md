Playground:

- https://www.katacoda.com/scenario-examples/courses/environment-usages/nodejs
- https://codedamn.com/online-compiler/node#start

1. install cargo-circom
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
alias circom2='circom'

```
or install circom2
```
# install circom and snarkjs
npm install -g circom2
npm install -g snarkjs

```

2. test with command line
```
# generate wasm
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

circom2 multiplier2.circom --r1cs --wasm --sym --c

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

3. test with index.js
```
mkdir js && cd js
npm init -y
npm install snarkjs

# copy the necessary resources
cp ../multiplier2.wasm .
cp ../circuit_final.zkey .
snarkjs zkey export verificationkey circuit_final.zkey verification_key.json

cat << "EOF" > index.js
const snarkjs = require("snarkjs");
const fs = require("fs");
const WASM = "multiplier2.wasm";
const ZKEY = "circuit_final.zkey";

async function run() {
    const p1 = await snarkjs.plonk.fullProve({a: 2, b: 17}, WASM, ZKEY);

    const p2 = await snarkjs.plonk.fullProve({a: 3, b: 19}, WASM, ZKEY);

    const p3 = await snarkjs.plonk.fullProve({a: 1, b: 34}, WASM, ZKEY);

    //console.log("Proof: ");
    //console.log(JSON.stringify(proof, null, 1));

    const vKey = JSON.parse(fs.readFileSync("verification_key.json"));

    publicSignals = ['34'];
	
	// valid proof 2x17
    res = await snarkjs.plonk.verify(vKey, publicSignals, p1.proof);
    console.log("Factorization: 2x17==34: ", res);

	// invalid proof 3x19
    res = await snarkjs.plonk.verify(vKey, publicSignals, p2.proof);
    console.log("Factorization: 3x19==34: ", res);

	// invalid proof 1x34
    res = await snarkjs.plonk.verify(vKey, publicSignals, p3.proof);
    console.log("Factorization: 1x34==34: ", res);
}

run().then(() => {
    process.exit(0);
});
EOF

node index.js

```

test with html
```
cp node_modules/snarkjs/build/snarkjs.min.js .

cat << EOF > index.html
<!doctype html>
<html>
<head>
  <title>Snarkjs client example</title>
</head>
<body>

  <h1>Snarkjs client example</h1>
  <button id="bGenProof1"> Create proof(3x11==33) </button>
  <button id="bGenProof2"> Create proof(3x11==37) </button>
  <button id="bGenProof3"> Create proof(1x33==33) </button>

  <!-- JS-generated output will be added here. -->
  <pre class="proof"> Proof: <code id="proof"></code></pre>

  <pre class="proof"> Result: <code id="result"></code></pre>


  <script src="snarkjs.min.js">   </script>
  
   <!-- This is the bundle generated by rollup.js -->
  <script>

const proofCompnent = document.getElementById('proof');
const resultComponent = document.getElementById('result');
const bGenProof = document.getElementById("bGenProof");

bGenProof1.addEventListener("click", prove33);
bGenProof2.addEventListener("click", prove37);
bGenProof3.addEventListener("click", proveSecond33);

function prove33() {calculateProof(3, 11, 33);}
function prove37() {calculateProof(3, 11, 37);}
function proveSecond33() {calculateProof(1, 33, 33);}

async function calculateProof(w1, w2, numToFactorize) {
    proofCompnent.innerHTML = "Processing...";
    resultComponent.innerHTML = "";
    const p1 = await snarkjs.plonk.fullProve( { a: w1, b: w2}, "multiplier2.wasm", "multiplier2_0001.zkey");

    //proofCompnent.innerHTML = JSON.stringify(p1.proof, null, 1);
    //proofCompnent.innerHTML = JSON.stringify(p1.publicSignals, null, 1);
    console.log("w1:%d, w2:%d, numToFactorize:%d", w1, w2, numToFactorize);
    
    publicSignals = [numToFactorize];
    proofCompnent.innerHTML = "witness: a:"+w1+", b:"+w2+ " public input:" + numToFactorize;
    
    //proofCompnent.innerHTML = JSON.stringify(publicSignals, null, 1);    
    
    const vkey = await fetch("verification_key.json").then( function(res) {
        return res.json();
    });

    const res = await snarkjs.plonk.verify(vkey, publicSignals, p1.proof);

    resultComponent.innerHTML = res;
}

  </script>
</body>
</html>  
EOF

npx http-server

```

2. ZkREPL
https://zkrepl.dev/
