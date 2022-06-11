# Step by Step Tutorial from [circom](https://docs.circom.io/getting-started/installation/)

Refer to repo `circom`

## Installation
### Install circom
**For Linux or MacOs**
`curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh`

`git clone https://github.com/iden3/circom.git`

`cargo build --release`

`cargo install --path circom`


(optional)
`circom --help`

### Install snarkjs
`npm install -g snarkjs`

## Write Circuits
### Create `multiplier2.circom` file

    pragma circom 2.0.0;

    /*This circuit template checks that c is the multiplication of a and b.*/  

    template Multiplier2 () {  

    // Declaration of signals.  
    signal input a;  
    signal input b;  
    signal output c;  

    // Constraints.  
    c <== a * b;  
    }


## Compile Circuit
`circom multiplier2.circom --r1cs --wasm --sym --c`

## Compute witness
### Create `input.json` file
insert the following json data into the file

`{"a": 3, "b": 11}`

### Compute witness with WebAssembly or with C++
#### With WebAssembly
Navigate to repo `multiplier2_js` and add the `input.json` file there, and run
`node generate_witness.js multiplier2.wasm input.json witness.wtns`

### With C++
Navigate to repo `multiplier2_cpp` and add `input.json` file there, and run
`make`, it will create an executable called `multiplier2`.
After that, run `./multiplier2 input.json witness.wtns`.

**Both options will create the same `witness.wtns` file, which we need later.**


## Proving circuits
### Powers of Tau
`snarkjs powersoftau new bn128 12 pot12_0000.ptau -v`

`snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v`

### Phase 2
`snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v`

`snarkjs groth16 setup multiplier2.r1cs pot12_final.ptau multiplier2_0000.zkey`

`snarkjs zkey export verificationkey multiplier2_0001.zkey verification_key.json`

### Generate a proof
`snarkjs groth16 prove multiplier2_0001.zkey witness.wtns proof.json public.json`

### Verifying a Proof
`snarkjs groth16 verify verification_key.json public.json proof.json`

### Verifying from a Smart contract
This method is used to verify your proof on Ethereum smart contract.
`snarkjs zkey export solidityverifier multiplier2_0001.zkey verifier.sol`

It will create `verifier.sol` file, you can copy and paste it on REMIX and deploy.

To generate a proof, run `snarkjs generatecall`.
Copy the output as the input for verifier.verifyProof() function. The method will return `TRUE` if works fine.