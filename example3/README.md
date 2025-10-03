# Ejemplo 3: Firmas Digitales

Queremos implementar un sistema de firmas digitales usando zk

## Paso 1: Write

Completar la definición del circuito en [example3.circom](./example3.circom)

## Paso 2: Build

Ejecutar

```bash
# Agregamos este step para instalar deps
npm i

mkdir target
circom "example3.circom" --r1cs --wasm -l node_modules --output target/
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
snarkjs groth16 setup ../target/example3.r1cs pot12_final.ptau example3_0000.zkey

# Contribute to phase 2
snarkjs zkey contribute example3_0000.zkey example3_0001.zkey --name="Foo" -v

# Export verification key
snarkjs zkey export verificationkey example3_0001.zkey example3_verification_key.json
```

## Paso 4: Prove

### Generar input

Para usar el esquema de firmas necesitamos un secreto y un commitment de ese secreto. Generamos un numero random y lo commiteamos calculando su hash

```bash
node generate_identity.js
```

Escribir el contenido en `input.json`

```bash
# Volver al root dir del ejemplo :P
cd ..

# Generar witness
node ../utils/generate_witness.js target/example3_js/example3.wasm input.json target/witness.wtns

# Generar prueba
snarkjs groth16 prove ceremony/example3_0001.zkey target/witness.wtns target/proof.json target/public.json
```

## Paso 5: Verify

```bash
snarkjs groth16 verify ceremony/example3_verification_key.json target/public.json target/proof.json
```
