Goto https://zkrepl.dev
After it finish compling click Groth16
Download main.wtns, main.groth16.zkey and main.groth16.vkey.json
generate prove and verify
```
snarkjs groth16 prove main.groth16.zkey main.wtns proof.json public.json
snarkjs groth16 verify main.groth16.vkey.json public.json proof.json
```

Download main.wasm
generate fullprove and verify
```
cat << EOF > input.json
{"a":"5","b":"77"}
EOF

snarkjs groth16 fullprove input.json main.wasm main.groth16.zkey proof.json public.json
snarkjs groth16 verify main.groth16.vkey.json public.json proof.json
```
