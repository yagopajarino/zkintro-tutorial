# Ejemplo 1: Circuito de multiplicación

Vamos a crear un "Hola Mundo" en ZKPs: un programa que demuestra conocimiento de dos números secretos cuya multiplicación es pública, sin revelarlos.

Por ejemplo, si el número público es 33, los secretos serían 11 y 3. Esta idea sirve como base para lo que vendrá luego.

## Paso 1: Write

Completar la definición del circuito en [example1.circom](./example1.circom)

## Paso 2: Build

Ejecutar

```bash
mkdir target
circom "example1.circom" --r1cs --wasm --output target/
```

## Paso 3: Setup

### Fase 1

```bash
mkdir ceremony && cd ceremony

# Init ceremony
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v

# Contribute with entropy
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="Foo" -v
```

### Fase 2

```bash
# Start phase 2 - circuit-specific trusted setup
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v

# Generate a zkey file named according to the R1CS filename
snarkjs groth16 setup ../target/example1.r1cs pot12_final.ptau example1_0000.zkey

# Contribute to phase 2
snarkjs zkey contribute example1_0000.zkey example1_0001.zkey --name="Foo" -v

# Export verification key
snarkjs zkey export verificationkey example1_0001.zkey example1_verification_key.json
```

## Paso 4: Prove

```bash
# Generar witness
node ../utils/generate_witness.js target/example1_js/example1.wasm input.json target/witness.wtns

# Generar prueba
snarkjs groth16 prove ceremony/example1_0001.zkey target/witness.wtns target/proof.json target/public.json
```

## Paso 5: Verify

```bash
snarkjs groth16 verify ceremony/example1_verification_key.json target/public.json target/proof.json
```
