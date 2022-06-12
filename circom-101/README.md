# Step by Step Tutorial from [circom](https://docs.circom.io/getting-started/installation/)

Refer to repo `circom`

Flow according to [iden3 Circom Tutorial](https://iden3-docs.readthedocs.io/en/latest/iden3_repos/circom/TUTORIAL.html)

![Circom & SnarkJs Flow](./Image/circuit%20flow.drawio.png)

Note: I'm unable to compile circuit to json file by running command `circom circuit.circom -o circuit.json` but can compile to other format like cpp, wasm. So I guess it's the problem of installing circom. There are two ways to install circom: 
1. According to Iden3 Tutorial: `npm install -g circom`
2. According to Circom Tutorial:  [Install circom](#install-circom)

I've tried first one but couldn't compile to json file, so I used second one instead.

This flow chart is simpler and you can get a general idea of what they are doing, so I demonstrate the flow from [iden3 Circom Tutorial](https://iden3-docs.readthedocs.io/en/latest/iden3_repos/circom/TUTORIAL.html), you get the idea.


## Installation
### Install circom
**For Linux or MacOs**

Install circom compiler(written in Rust)
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

    // Compiler look for main component to compile circuit
    component main = Multiplier2()


## Compile Circuit
`circom multiplier2.circom --r1cs --wasm --sym --c`

**Output will be four folders/files**

1. `multiplier2_cpp` folder
2. `multiplier_js` folder
3. `multiplier2.r1cs` file
4. `multiplier2.sym` file

## Compute witness
Witness includes input, intermediate and output signals which shows that you are able to generate a proof based on these inputs. You will need input signals (in JSON form) to generate a witness. Only public input and output(public by default) is visible to public.
### Create `input.json` file
insert the following json data into the file

`{"a": 3, "b": 11}`

By doing this, you are giving an example to the circuit and the circuit will generate witness that satisfy the circuit constrains(a bunch of equations).

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
zkSNARK need to `trusted setup` to generate a proof. Basically what **Powers of Tau** do is `trusted setup`.
### Powers of Tau

Generate `pot12_0000.ptau`

`snarkjs powersoftau new bn128 12 pot12_0000.ptau -v`

Generate `pot12_0001.ptau`

`snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v`

### Phase 2

Generate `pot12_final.ptau`

`snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v`

Generate `multiplier1_0000.zkey`

`snarkjs groth16 setup multiplier2.r1cs pot12_final.ptau multiplier2_0000.zkey`

Generate `multiplier2_0001.zkey`

`snarkjs zkey export verificationkey multiplier2_0001.zkey verification_key.json`

### Generate a proof
`snarkjs groth16 prove multiplier2_0001.zkey witness.wtns proof.json public.json`

### Verifying a Proof (Off chain)
`snarkjs groth16 verify verification_key.json public.json proof.json`

### Verifying from a Smart contract (On chain)
This method is used to verify your proof on Ethereum smart contract.
`snarkjs zkey export solidityverifier multiplier2_0001.zkey verifier.sol`

It will create `verifier.sol` file, you can copy and paste it on REMIX and deploy.

To generate a proof, run `snarkjs generatecall`.
Copy the output as the input for verifier.verifyProof() function. The method will return `TRUE` if works fine.