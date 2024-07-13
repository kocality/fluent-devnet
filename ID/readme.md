# Panduan untuk Membangun HelloWorld (Aplikasi Campuran) di Fluent Devnet

<img src="https://images.mirror-media.xyz/publication-images/_89lCC1I0m5JlMwv14wo3.png?height=360&width=720" width="700"/>

## Tentang Fluent
Jaringan eksekusi campuran pertama. Fluent mencampur aplikasi Wasm, EVM, dan SVM ke dalam lingkungan eksekusi yang terintegrasi.
* [Twitter](https://x.com/fluentxyz)
* [Website](https://fluent.xyz/)
* [Discord](https://discord.gg/fluentlabs)
* [Docs](https://docs.fluentlabs.xyz/learn/introduction/what-is-fluent)
* [Github](https://github.com/fluentlabs-xyz)
* [Devnet Explorer](https://blockscout.dev.thefluent.xyz/)

Untuk informasi lebih lanjut tentang Fluent, Anda dapat menemukan artikel saya "Fluent: Simplifying Blockchain Technology with Blended Execution" dari [tautan ini](https://mirror.xyz/kocality.eth/orzqskeUXS_lefo0oo99nB9r21wdKzJGp7gfquYdLlc).


## Tentang Panduan
Panduan ini memberikan instruksi langkah demi langkah untuk membuat aplikasi Campuran (HelloWorld) di Fluent Devnet. Aplikasi ini terdiri dari smart contract Rust yang mencetak "Hello" dan smart contract Solidity yang mencetak "World." Pengaturan ini menunjukkan:

- **Keterhubungan:** Mengintegrasikan berbagai bahasa pemrograman (Solidity dan Rust) ke dalam satu aplikasi.
- **Interoperabilitas** Memastikan operasi yang mulus antara berbagai target mesin virtual (EVM dan Wasm).


Dengan mengikuti panduan ini, Anda akan belajar cara:

- Menggabungkan berbagai bahasa pemrograman ke dalam satu proyek.
- Memastikan operasi tanpa hambatan di berbagai target mesin virtual.
- Mengelola semuanya dalam lingkungan eksekusi yang terintegrasi.

## Langkah 1: Pembaruan Sistem dan Instalasi Alat yang Diperlukan

### Perbarui Paket Sistem
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential -y
```

```bash
# Instalasi Node.js dan npm
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Instalasi pnpm
npm install -g pnpm
```

### Instalasi Rust dan Cargo
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

rustup target add wasm32-unknown-unknown
```

## Langkah 2: Inisialisasi Proyek Rust

### Siapkan Proyek Rust
```bash
cargo new --lib greeting
cd greeting
```

### Konfigurasi Proyek Rust
Masukkan file dengan perintah `nano Cargo.toml` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
[package]
edition = "2021"
name = "greeting"
version = "0.1.0"

[dependencies]
alloy-sol-types = {version = "0.7.4", default-features = false}
fluentbase-sdk = {git = "https://github.com/fluentlabs-xyz/fluentbase", default-features = false}

[lib]
crate-type = ["cdylib", "staticlib"] # For accessing the C lib
path = "src/lib.rs"

[profile.release]
lto = true
opt-level = 'z'
panic = "abort"
strip = true

[features]
default = []
std = [
  "fluentbase-sdk/std",
]
```

### Write Rust Smart Contract
Pertama, hapus kode Smart Contract Rust dengan perintah `rm src/lib.rs`. Kemudian masukkan file dengan perintah `nano src/lib.rs` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
#![cfg_attr(target_arch = "wasm32", no_std)]
extern crate alloc;
extern crate fluentbase_sdk;

use alloc::string::{String, ToString};
use fluentbase_sdk::{
    basic_entrypoint,
    derive::{router, signature},
    SharedAPI,
};

#[derive(Default)]
struct ROUTER;

pub trait RouterAPI {
    fn greeting<SDK: SharedAPI>(&self) -> String;
}

#[router(mode = "solidity")]
impl RouterAPI for ROUTER {
    #[signature("function greeting() external returns (string)")]
    fn greeting<SDK: SharedAPI>(&self) -> String {
        "Hello".to_string()
    }
}

impl ROUTER {
    fn deploy<SDK: SharedAPI>(&self) {
        // any custom deployment logic here
    }
}
basic_entrypoint!(ROUTER);
```

### Buat Makefile
Masukkan file dengan perintah `nano Makefile` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
.DEFAULT_GOAL := all

# Compilation flags
RUSTFLAGS := '-C link-arg=-zstack-size=131072 -C target-feature=+bulk-memory -C opt-level=z -C strip=symbols'

# Paths to the target WASM file and output directory
WASM_TARGET := ./target/wasm32-unknown-unknown/release/greeting.wasm
WASM_OUTPUT_DIR := bin
WASM_OUTPUT_FILE := $(WASM_OUTPUT_DIR)/greeting.wasm

# Commands
CARGO_BUILD := cargo build --release --target=wasm32-unknown-unknown --no-default-features
RM := rm -rf
MKDIR := mkdir -p
CP := cp

# Targets
all: build

build: prepare_output_dir
	@echo "Building the project..."
	RUSTFLAGS=$(RUSTFLAGS) $(CARGO_BUILD)

	@echo "Copying the wasm file to the output directory..."
	$(CP) $(WASM_TARGET) $(WASM_OUTPUT_FILE)

prepare_output_dir:
	@echo "Preparing the output directory..."
	$(RM) $(WASM_OUTPUT_DIR)
	$(MKDIR) $(WASM_OUTPUT_DIR)

.PHONY: all build prepare_output_dir
```

### Bangun Proyek Wasm
```bash
make
```
## Langkah 3: Inisialisasi Proyek Solidity

### Buat Direktori Proyek
```bash
cd
mkdir typescript-wasm-project

mkdir -p ~/typescript-wasm-project/greeting/bin
cp ~/greeting/target/wasm32-unknown-unknown/release/greeting.wasm ~/typescript-wasm-project/greeting/bin/

cd typescript-wasm-project
npm init -y
```

### Instalasi Dependensi
```bash
npm install --save-dev typescript ts-node hardhat hardhat-deploy ethers dotenv @nomicfoundation/hardhat-toolbox @typechain/ethers-v6 @typechain/hardhat @types/node
pnpm add ethers@^5.7.2 @nomiclabs/hardhat-ethers@2.0.6
pnpm install
npx hardhat
```

Setelah perintah `npx hardhat`, ia akan meminta beberapa informasi, masukkan seperti pada gambar di bawah ini

![hardhat2](https://github.com/kocality/fluent-devnet/assets/69348404/ba8d407c-6d61-4fac-b628-f170ba13e2cd)

## Konfigurasi TypeScript dan Hardhat

### Perbarui Konfigurasi Hardhat
hapus kode Hardhat dengan perintah `rm hardhat.config.ts`. Kemudian masukkan file dengan perintah `nano hardhat.config.ts` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "hardhat-deploy";
import * as dotenv from "dotenv";
import "./tasks/get-greeting"; 
import "@nomiclabs/hardhat-ethers"; 

dotenv.config();

const config: HardhatUserConfig = {
  defaultNetwork: "dev",
  networks: {
    dev: {
      url: process.env.RPC_URL || "https://rpc.dev.thefluent.xyz/",
      accounts: [process.env.DEPLOYER_PRIVATE_KEY || "your-private-key"],
      chainId: 20993,
    },
  },
  solidity: {
    compilers: [
      {
        version: "0.8.20",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
      {
        version: "0.8.24",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
  },
  namedAccounts: {
    deployer: {
      default: 0,
    },
  },
};

export default config;
```

### Perbarui Paket
Hapus kode package.json dengan perintah `rm package.json`. Kemudian masukkan file dengan perintah `nano package.json` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
{
  "name": "blendedapp",
  "version": "1.0.0",
  "description": "Blended Hello, World",
  "main": "index.js",
  "scripts": {
    "compile": "npx hardhat compile",
    "deploy": "npx hardhat deploy"
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-ethers": "^3.0.0",
    "@nomicfoundation/hardhat-toolbox": "^5.0.0",
    "@nomicfoundation/hardhat-verify": "^2.0.0",
    "@openzeppelin/contracts": "^5.0.2",
    "@typechain/ethers-v6": "^0.5.0",
    "@typechain/hardhat": "^9.0.0",
    "@types/node": "^20.12.12",
    "dotenv": "^16.4.5",
    "hardhat": "^2.22.4",
    "hardhat-deploy": "^0.12.4",
    "ts-node": "^10.9.2",
    "typescript": "^5.4.5"
  },
  "dependencies": {
    "ethers": "^6.12.2",
    "fs": "^0.0.1-security"
  }
}
```

### Atur Variabel Lingkungan
Di sini kita akan memasukkan kunci pribadi. Saya menggunakan dompet Metamask saya. Dapatkan ETH dari [Fluent Devnet Faucet](https://faucet.dev.thefluent.xyz/) ke dompet yang akan digunakan.


Masukkan kunci pribadi Anda di tempat yang bertuliskan `your-private-key-here`.

```bash
nano .env
```

```bash
DEPLOYER_PRIVATE_KEY=your-private-key-here
```

![private](https://github.com/kocality/fluent-devnet/assets/69348404/767b2ce6-caf0-4885-8f28-77f2a50d6af0)


## Tulis Kontrak Solidity

### Tentukan Antarmuka
Masukkan file dengan perintah `nano contracts/IFluentGreeting.sol` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IFluentGreeting {
    function greeting() external view returns (string memory);
}
```

### Implementasikan Kontrak Greeting
Masukkan file dengan perintah `nano contracts/GreetingWithWorld.sol` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./IFluentGreeting.sol";

contract GreetingWithWorld {
    IFluentGreeting public fluentGreetingContract;

    constructor(address _fluentGreetingContractAddress) {
        fluentGreetingContract = IFluentGreeting(_fluentGreetingContractAddress);
    }

    function getGreeting() external view returns (string memory) {
        string memory greeting = fluentGreetingContract.greeting();
        return string(abi.encodePacked(greeting, ", World"));
    }
}
```

## Langkah 4: Sebarkan Kedua Kontrak Menggunakan Hardhat

### Buat Skrip Penyebaran

Skrip penyebaran ini bertanggung jawab untuk menyebarkan baik smart contract Rust (dikompilasi ke Wasm) maupun smart contract Solidity.

Pertama, buat folder `deploy` dengan perintah `mkdir deploy`, kemudian masukkan file dengan perintah `nano deploy/01_deploy_contracts.ts` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { DeployFunction } from "hardhat-deploy/types";
import { ethers } from "hardhat";
import fs from "fs";
import crypto from "crypto";
import path from "path";
require("dotenv").config();

const DEPLOYER_PRIVATE_KEY = process.env.DEPLOYER_PRIVATE_KEY || "your-private-key";

const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts, network } = hre;
  const { deploy, save, getOrNull } = deployments;
  const { deployer: deployerAddress } = await getNamedAccounts();

  console.log("deployerAddress", deployerAddress);

  // Deploying WASM Contract
  console.log("Deploying WASM contract...");
  const wasmBinaryPath = "./greeting/bin/greeting.wasm";
  const provider = new ethers.providers.JsonRpcProvider(network.config.url);
  const deployer = new ethers.Wallet(DEPLOYER_PRIVATE_KEY, provider);

  const fluentGreetingContractAddress = await deployWasmContract(wasmBinaryPath, deployer, provider, getOrNull, save);

  // Deploying Solidity Contract
  console.log("Deploying GreetingWithWorld contract...");
  const greetingWithWorld = await deploy("GreetingWithWorld", {
    from: deployerAddress,
    args: [fluentGreetingContractAddress],
    log: true,
  });

  console.log(`GreetingWithWorld contract deployed at: ${greetingWithWorld.address}`);
};

async function deployWasmContract(
  wasmBinaryPath: string,
  deployer: ethers.Wallet,
  provider: ethers.providers.JsonRpcProvider,
  getOrNull: any,
  save: any
) {
  const wasmBinary = fs.readFileSync(wasmBinaryPath);
  const wasmBinaryHash = crypto.createHash("sha256").update(wasmBinary).digest("hex");
  const artifactName = path.basename(wasmBinaryPath, ".wasm");
  const existingDeployment = await getOrNull(artifactName);

  if (existingDeployment && existingDeployment.metadata === wasmBinaryHash) {
    console.log("WASM contract bytecode has not changed. Skipping deployment.");
    console.log(`Existing contract address: ${existingDeployment.address}`);
    return existingDeployment.address;
  }

  const gasPrice = (await provider.getFeeData()).gasPrice;

  const transaction = {
    data: "0x" + wasmBinary.toString("hex"),
    gasLimit: 3000000,
    gasPrice: gasPrice,
  };

  const tx = await deployer.sendTransaction(transaction);
  const receipt = await tx.wait();

  if (receipt && receipt.contractAddress) {
    console.log(`WASM contract deployed at: ${receipt.contractAddress}`);

    const artifact = {
      abi: [],
      bytecode: "0x" + wasmBinary.toString("hex"),
      deployedBytecode: "0x" + wasmBinary.toString("hex"),
      metadata: wasmBinaryHash,
    };

    const deploymentData = {
      address: receipt.contractAddress,
      ...artifact,
    };

    await save(artifactName, deploymentData);
  } else {
    throw new Error("Failed to deploy WASM contract");
  }

  return receipt.contractAddress;
}

export default func;
func.tags = ["all"];
```

### Buat Tugas Hardhat
Pertama, buat folder `tasks` dengan perintah `mkdir tasks`, kemudian masukkan file dengan perintah `nano tasks/get-greeting.ts` dan edit seperti kode di bawah ini.
(Ctrl + X, Y dan Enter akan menyimpan perubahan)

```bash
import { task } from "hardhat/config";
import { ethers } from "hardhat";

task("get-greeting", "Fetches the greeting from the deployed GreetingWithWorld contract")
  .addParam("contract", "The address of the deployed GreetingWithWorld contract")
  .setAction(async ({ contract }, hre) => {
    const GreetingWithWorld = await hre.ethers.getContractAt("GreetingWithWorld", contract);
    const greeting = await GreetingWithWorld.getGreeting();
    console.log("Greeting:", greeting);
  });
```

## Langkah 5: Kompilasi dan Sebarkan Kontrak
Setelah menyebarkan kontrak dengan perintah `pnpm hardhat deploy`, kita akan memasukkan output tx yang kita dapatkan seperti pada foto di bagian 0x.... dalam kode `pnpm hardhat get-greeting --contract 0x......`

![deploy](https://github.com/kocality/fluent-devnet/assets/69348404/65c39233-29cf-4110-afa1-13c1edbca9e7)


```bash
pnpm hardhat compile

pnpm hardhat deploy

pnpm hardhat get-greeting --contract 0x.....
```

Jika Anda mendapatkan output seperti di bawah ini, berarti sudah selesai. Anda dapat mencari tx melalui Explorer.

![son](https://github.com/kocality/fluent-devnet/assets/69348404/a83133f6-3ef3-44a8-8a88-d49beef76f1f)
