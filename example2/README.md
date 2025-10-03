# Ejemplo 2: Restringir el multiplicador

Necesitamos agregar restricciones al circuito para que sean válidos los casos triviales ej (a=1, b=33)

## Paso 1: Write

Completar la definición del circuito en [example2.circom](./example2.circom)

## Paso 2: Build

Ejecutar

```bash
# Agregamos este step para instalar deps
npm i

mkdir target
circom "example2.circom" --r1cs --wasm -l node_modules --output target/
```

## Paso 3: Setup

### Fase 1

En este caso no hace falta rehacer la fase 1

### Fase 2

```bash
# Copy the pot12 (phase1 output) from example1
mkdir ceremony && cp ../example1/ceremony/pot12_0001.ptau ceremony && cd ceremony

# Start phase 2 - circuit-specific trusted setup
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v

# Generate a zkey file named according to the R1CS filename
snarkjs groth16 setup ../target/example2.r1cs pot12_final.ptau example2_0000.zkey

# Contribute to phase 2
snarkjs zkey contribute example2_0000.zkey example2_0001.zkey --name="Foo" -v

# Export verification key
snarkjs zkey export verificationkey example2_0001.zkey example2_verification_key.json
```

## Paso 4: Prove

```bash
# Generar witness
node ../utils/generate_witness.js target/example2_js/example2.wasm input.json target/witness.wtns

# Generar prueba
snarkjs groth16 prove ceremony/example2_0001.zkey target/witness.wtns target/proof.json target/public.json
```

## Paso 5: Verify

```bash
snarkjs groth16 verify ceremony/example2_verification_key.json target/public.json target/proof.json
```
