1. Goto https://zkrepl.dev

### Groth16
2. After compliation finish click Groth16
3. Download main.wtns, main.groth16.zkey and main.groth16.vkey.json
4. Generate prove and verify
```
snarkjs groth16 prove main.groth16.zkey main.wtns proof.json public.json
snarkjs groth16 verify main.groth16.vkey.json public.json proof.json

```

5. Download main.wasm
6. Generate fullprove and verify
```
cat << EOF > input.json
{"a":"5","b":"77"}
EOF

snarkjs groth16 fullprove input.json main.wasm main.groth16.zkey proof.json public.json
snarkjs groth16 verify main.groth16.vkey.json public.json proof.json

```

### Plonk
2. After compliation finish click Plonk
3. Download main.wtns, main.plonk.zkey and main.plonk.vkey.json
4. Generate prove and verify
```
snarkjs plonk prove main.plonk.zkey main.wtns proof.json public.json
snarkjs plonk verify main.plonk.vkey.json public.json proof.json

```

5. Download main.wasm
6. Generate fullprove and verify
```
cat << EOF > input.json
{"a":"5","b":"77"}
EOF

snarkjs plonk fullprove input.json main.wasm main.plonk.zkey proof.json public.json
snarkjs plonk verify main.plonk.vkey.json public.json proof.json

```

