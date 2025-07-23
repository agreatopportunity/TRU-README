# TRU Blockchain

Welcome to the TRU Blockchain repository! This project provides a comprehensive blockchain ecosystem, including a node with an integrated RPC server, standalone CPU and GPU miners, a command-line wallet, and a web-based block explorer. This README explains how to install, build, and run each component, along with details on their interactions via JSON-RPC.

## Table of Contents

1. [Overview](#overview)
2. [Installation & Dependencies](#installation--dependencies)
3. [Building the Project](#building-the-project)
4. [Running the Components](#running-the-components)
   - [TRU Node with RPC Server](#tru-node-with-rpc-server)
   - [CPU Miner](#cpu-miner)
   - [GPU Miner](#gpu-miner)
   - [Wallet CLI](#wallet-cli)
   - [Block Explorer](#block-explorer)
5. [Smart Contracts](#Smart-Contracts-Section)
6. [P2P Messaging](#P2P-Messaging)
7. [Tokens on TRU Blockchain](#Tokens-on-TRU-Blockchain)
8. [RPC Reference](#rpc-reference)
9. [Troubleshooting](#troubleshooting)
10. [Notes](#notes)

---

## Overview

The TRU Blockchain consists of the following executables, each built from the provided source files:

- **TRU Node with RPC Server** (`tru_advanced` from `rpc_server.cpp`): The core blockchain node that maintains the blockchain state, processes transactions, and serves JSON-RPC requests on port `8332` by default. It’s essential for all other components to function.
- **CPU Miner** (`tru_miner_cpu` from `my_miner.cpp`): A standalone CPU-based miner that fetches block templates via RPC, mines blocks, and submits them to the node.
- **GPU Miner** (`tru_miner` from `my_gpu_miner.cpp`): A standalone GPU-based miner leveraging OpenCL for parallel mining, interacting with the node via RPC.
- **Wallet CLI** (`tru_wallet` from `wallet_cli.cpp`): A standalone command-line wallet for managing keys, sending TRU, issuing tokens, and more, all via RPC communication with the node.
- **Block Explorer** (`blockexplorer` from `blockexplorer.cpp`): A standalone web-based interface for exploring blockchain data (blocks, transactions, addresses, tokens), using RPC to fetch data from the node.

All components rely on the TRU Node’s RPC server for blockchain interaction, ensuring a modular and distributed architecture.

---

## Installation & Dependencies

To build and run the TRU Blockchain components, install the following dependencies:

### System Requirements
- **Operating System**: Linux (e.g., Ubuntu 20.04+ recommended)
- **Compiler**: C++17-compatible (e.g., GCC 9+, Clang 10+)
- **CMake**: Version 3.10 or higher

### Core Dependencies
- **OpenSSL** (for cryptography):
  ```bash
  apt-get install libssl-dev
  ```
- **LevelDB** (for blockchain storage):
  ```bash
  apt-get install libleveldb-dev
  ```
- **libfmt** (for formatting):
  ```bash
  apt-get install libfmt-dev
  ```
- **nlohmann/json** (JSON parsing, header-only):
  ```bash
  apt-get install nlohmann-json3-dev
  ```
- **cpp-httplib** (HTTP client/server, header-only):
  ```bash
  apt-get install cpp-httplib-dev
  ```
- **libcurl** (for RPC communication in GPU miner):
  ```bash
  apt-get install libcurl4-openssl-dev
  ```
- **jsoncpp** (for json communication):
  ```bash
  apt-get install libjsoncpp-dev
  ```
- **Proto Buff** (TODO):
  ```bash
  sudo apt install protobuf-compiler
  ```
- **Ethash** (TODO):
  ```bash
  sudo apt-get install ethash
  ```  
- **crc32c** (LeveDB enhancement):
  ```bash
  git clone https://github.com/google/crc32c.git
  git submodule update --init --recursive
  mkdir build && cd build
  cmake -DCRC32C_BUILD_TESTS=OFF -DCRC32C_BUILD_BENCHMARKS=OFF ..
  make
  make install
  ```
- **Libsodium** (provides a robust implementation of BLAKE2b):
  ```bash
  apt install libsodium-dev
  ```
  
### GPU Miner Dependencies
- **OpenCL** (for GPU mining):
  ```bash
  sudo apt-get install ocl-icd-opencl-dev
  ```
  - Ensure GPU drivers (e.g., NVIDIA CUDA, AMD ROCm) are installed.

### Wallet Dependencies
- **libwally-core** (for wallet cryptography):
  ```bash
  sudo apt-get install cmake libtool autoconf automake pkg-config
  git clone https://github.com/ElementsProject/libwally-core.git
  cd libwally-core
  ./tools/autogen.sh
  ./configure --enable-swig-python=no --enable-swig-java=no
  make && sudo make install && sudo ldconfig
  ```

### One-Liner for Core Dependencies
```bash
sudo apt-get update && sudo apt-get install -y cmake g++ libssl-dev libleveldb-dev libfmt-dev nlohmann-json3-dev cpp-httplib-dev libcurl4-openssl-dev ocl-icd-opencl-dev
```

---

## Building the Project

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd tru-blockchain
   ```

2. **Create Build Directory**:
   ```bash
   mkdir build
   cd build
   ```

3. **Generate Build System**:
   ```bash
   cmake .. -DCMAKE_BUILD_TYPE=Release
   ```

4. **Compile**:
   ```bash
   make
   ```

5. **Executables Location**:
   After a successful build, find the executables in `build/bin/`:
   - `tru_advanced` (node with RPC server)
   - `tru_miner_cpu` (CPU miner)
   - `tru_miner` (GPU miner)
   - `tru_wallet` (wallet CLI)
   - `blockexplorer` (block explorer)

---

## Running the Components

### TRU Node with RPC Server

The TRU Node (`tru_advanced`) manages the blockchain and provides RPC services.

- **Command**:
  ```bash
  ./tru_advanced --cli 
  ```
- **Details**:
  - Listens on `127.0.0.1:8332` for RPC requests by default.
  - Must be running for other components to function.
  - Use `--cli` for interactive mode (if implemented) and `--enable-explorer` to support explorer features.

### CPU Miner

The CPU Miner (`tru_miner_cpu`) mines blocks using multiple threads.

- **Command**:
  ```bash
  ./tru_miner_cpu --node-ip 127.0.0.1 --node-port 8332 --mineraddr <YOUR_ADDRESS> --threads 4 --max-nonce 2000000
  ```
- **Options**:
  - `--node-ip`: Node’s IP (default: `127.0.0.1`).
  - `--node-port`: Node’s RPC port (default: `8332`).
  - `--mineraddr`: Your TRU address for rewards.
  - `--threads`: Number of CPU threads (default: 4).
  - `--max-nonce`: Max nonce to try (default: 2,000,000).
- **Behavior**:
  - Fetches block templates via RPC, mines with CPU, submits blocks, and reports activity.

### GPU Miner

The GPU Miner (`tru_miner`) uses OpenCL for GPU-based mining.

- **Command**:
  ```bash
  ./tru_miner 127.0.0.1 8332 <YOUR_ADDRESS>
  ```
- **Arguments**:
  - Positional: `<nodeIP> <nodePort> [minerAddr]`.
  - Example address: `1Er2pcB7ZQYfcpb1ySByAFFmQD68DunU1b`.
- **Behavior**:
  - Detects and uses all available GPUs, mines blocks, and submits via RPC.
  - Requires OpenCL support and compatible GPU drivers.

### Wallet CLI

The Wallet CLI (`tru_wallet`) manages wallet operations via RPC.

- **Command**:
  ```bash
  ./tru_wallet --node-ip 127.0.0.1 --node-port 8332
  ```
- **Options**:
  - `--node-ip`: Node’s IP (default: `127.0.0.1`).
  - `--node-port`: Node’s RPC port (default: `8332`).
- **Interactive Menu**:
  | Option | Action                          |
  |--------|---------------------------------|
  | 1      | Create new wallet              |
  | 2      | Load wallet from file          |
  | 3      | Save wallet to file            |
  | 4      | Generate new address           |
  | 5      | Send TRU                       |
  | 6      | Check balance                  |
  | 7      | CPU mine block (with 5 TRU fee)|
  | 7a     | GPU mine block (with 5 TRU fee)|
  | 8      | View chain info                |
  | 9      | List peers                     |
  | 10     | Lookup block by hash           |
  | 11     | Exit                           |
  | 12     | List tokens                    |
  | 13     | Issue new token                |
  | 14     | Show wallet addresses          |
  | 15     | Send token (not implemented)   |
  | 16     | Connect to peer                |
  | 17     | Send message to peer           |
  | 18     | Create smart contract          |
  | 19     | List mempool transactions      |
  | 20     | Query contract state           |
  | C      | Clear output                   |
- **Behavior**:
  - Manages a local wallet file (`tru.dat`), generates addresses, signs transactions locally, and communicates with the node via RPC.

### Block Explorer

The Block Explorer (`blockexplorer`) provides a web interface for blockchain data.

- **Command**:
  ```bash
  ./build/bin/blockexplorer <utxoDBPath> [port]
  ```
- **Example**:
  ```bash
  ./build/bin/blockexplorer /path/to/utxo.db 8080
  ```
- **Arguments**:
  - `<utxoDBPath>`: Path to the LevelDB UTXO database.
  - `[port]`: Web server port (default: `8080`).
- **Access**:
  - Open `http://127.0.0.1:8080` in a browser.
- **Web Files**:
  - Serves static files from `/home/gw878/TRU/web` (hardcoded in `blockexplorer.cpp`).
  - **For Standalone Use**: Copy the web files to the same path on the target machine, or modify the code to embed them (e.g., as resources).
- **Database Communication**:
  - Uses RPC to fetch data from the node, not direct database access. Ensure the node is running and accessible at `127.0.0.1:8332`.

---

### Smart Contracts Section

This section incorporates details from your `opcodes.h` (custom opcodes like `OP_STORE`, `OP_LOAD`, etc.), `script_interpreter.cpp` (execution logic, gas, and state management), and assumes you have working smart contract examples to showcase.

```markdown
## Smart Contracts

Smart contracts in the TRU Blockchain are self-executing scripts embedded in transaction outputs, enabling decentralized applications (dApps) with persistent state and blockchain interaction. They leverage a stack-based scripting language enhanced with custom opcodes for advanced functionality.

### How Smart Contracts Work

- **Script-Based**: Contracts are written using a stack-based language that includes both standard Bitcoin-like opcodes and TRU-specific custom opcodes.
- **Execution**: When a transaction spends a contract’s unspent transaction output (UTXO), the script executes, potentially modifying its state or performing computations.
- **State Storage**: Persistent key-value pairs are managed using `OP_STORE` (to write) and `OP_LOAD` (to read), stored in a contract-specific state map.
- **Gas System**: A gas mechanism limits execution to prevent abuse, with each opcode consuming gas based on its complexity (e.g., 1 gas for basic ops, 100 for `OP_CHECKSIG`).
- **Context Awareness**: Opcodes like `OP_CALLER` (sender address) and `OP_CONTRACT_ADDR` (contract address) provide transaction context.

### Key Opcodes

- **State Management**:
  - `OP_STORE`: Stores a key-value pair in the contract’s state.
  - `OP_LOAD`: Retrieves a value by key from the state.
- **Context**:
  - `OP_CALLER`: Pushes the sender’s address to the stack.
  - `OP_CONTRACT_ADDR`: Pushes the contract’s address.
  - `OP_OUTPUTAMOUNT`: Pushes the output amount of the current transaction.
- **Control**:
  - `OP_HALT`: Stops execution successfully.
  - `OP_REVERT`: Reverts the transaction and state changes.
- **Gas**:
  - `OP_GAS`: Pushes the remaining gas to the stack.
- **Data Access**:
  - `OP_BLOCKTIME`: Pushes the current block time.
  - `OP_EXTERNALDATA`: Pushes external data (e.g., via oracles).
  - `OP_DATAFEED`: Retrieves data feed values by key.

### Creating and Interacting with Smart Contracts

1. **Create a Contract**:
   - Embed the contract script in a transaction output’s `scriptPubKey`.
   - Use the `createsmartcontract` RPC method or wallet CLI option 18 to deploy.

2. **Interact with a Contract**:
   - Spend the contract’s UTXO with a transaction providing inputs to the script.
   - The script executes, updating state or performing actions as defined.

### Example: Access Control Contract

Here’s a simple smart contract that restricts updates to its state to the original creator:

```
OP_CALLER OP_PUSHDATA1 20 0123456789abcdef0123456789abcdef01234567 OP_EQUALVERIFY
OP_PUSHDATA1 03 6b6579 OP_PUSHDATA1 05 76616c7565 OP_STORE OP_RETURN
```

- **Explanation**:
  - `OP_CALLER`: Pushes the sender’s address.
  - `OP_PUSHDATA1 20 0123...4567`: Pushes the creator’s 20-byte address.
  - `OP_EQUALVERIFY`: Ensures the sender matches the creator, failing if not.
  - `OP_PUSHDATA1 03 6b6579`: Pushes the key "key" (hex: 6b6579).
  - `OP_PUSHDATA1 05 76616c7565`: Pushes the value "value" (hex: 76616c7565).
  - `OP_STORE`: Stores "key" → "value" in the contract’s state.
  - `OP_RETURN`: Ends execution successfully.

*Note*: Replace this with your own working example if preferred. This is a demonstration based on your opcode set.

---
```

---

### P2P Messaging Section

This section is based on `message_handler.cpp`, `peer_connection.cpp`, `p2p.cpp`, and the `.proto` file, detailing your P2P messaging system with a focus on blockchain sync and chat functionality.

```markdown
## P2P Messaging

The TRU Blockchain’s peer-to-peer (P2P) messaging system enables nodes to synchronize blockchain data and communicate directly via chat messages. It uses Protocol Buffers for efficient message serialization and supports both standard blockchain messages and custom extensions.

### Message Types

- **Standard Messages**:
  - `VERSION`: Exchanges protocol version and node details (e.g., version 2025, subVersion "/TruChain:2.5.0-NewAge2025/").
  - `VERACK`: Acknowledges connection with an optional `quantum_ack` field.
  - `BLOCK`: Shares new blocks (via `BlockProto`).
  - `TX`: Shares new transactions (via `TxProto`).
  - `PING` / `PONG`: Maintains connection liveness.
- **Custom Messages**:
  - `CHAT`: Sends text messages between peers.

### How P2P Messaging Works

- **Connection Setup**: Nodes exchange `VERSION` and `VERACK` messages to establish a connection, including futuristic fields like `futureFlag` (e.g., "QuantumEncrypted=Yes").
- **Blockchain Sync**: Nodes broadcast `BLOCK` and `TX` messages to propagate new blocks and transactions across the network.
- **Chat Functionality**: The `CHAT` message type allows peers to send text messages, logged to the console or broadcasted to all connected peers.

### Usage

- **Automatic Blockchain Sync**:
  - Nodes handle `BLOCK` and `TX` messages automatically to maintain a consistent ledger.
- **Chat Messaging**:
  - Send messages to a specific peer using wallet CLI option 17 (e.g., `sendmessage <ip> <port> "Hello"`).
  - Broadcast chat to all peers with the `broadcastChat` method.

### Example

- **Sending a Chat Message**:
  - Command: `sendmessage 203.0.113.45 8333 "Hello, TRU Network!"`
  - Output: `[Chat from 203.0.113.46:8333] Hello, TRU Network!` (seen by the recipient).

---

## Tokens on TRU Blockchain

The TRU Blockchain supports a versatile token system that allows users to create, manage, and transfer various types of tokens directly on the blockchain. These tokens are seamlessly integrated into the wallet and blockchain infrastructure, enabling functionalities such as issuance, transfer, and metadata management.

### Token Types

The TRU Blockchain supports four distinct token types, each designed for specific use cases:

- **Fungible Tokens (FT)**: Divisible tokens suitable for currencies, utility tokens, or other applications requiring interchangeable units. Defined as `TokenType::TOKEN` in the code.
- **Non-Fungible Tokens (NFT)**: Unique, indivisible tokens representing ownership of specific items like digital art, collectibles, or real-world assets. Defined as `TokenType::NFT`.
- **Semi-Fungible Tokens (SFT)**: Tokens that combine properties of fungibility and uniqueness, ideal for use cases like gaming assets or tiered loyalty points. Defined as `TokenType::SFT`.
- **Non-Custodial Fungible Tokens (NCFT)**: Fungible tokens with properties tailored for non-custodial management, enhancing user control. Defined as `TokenType::NCFT`.

These token types are managed through the `TokenType` enum in `tokens.h`, with utility functions `tokenTypeToString` and `stringToTokenType` for converting between string representations and enum values.

### Token Structure

Tokens on the TRU Blockchain are represented by the `ExtendedTokenData` struct, which encapsulates the following fields:

- **`tokenID`**: A unique identifier for the token (e.g., "TRUFT" for a fungible token).
- **`type`**: The token type (FT, NFT, SFT, or NCFT), as defined by `TokenType`.
- **`amount`**: The quantity of the token. For FT and SFT, this represents the total supply or transferable amount; for NFTs, it’s typically set to 1.
- **`version`**: The version of the token standard (default is 1).
- **`meta`**: A `TokenMeta` struct containing a map of metadata attributes (e.g., `name`, `symbol`, `description`, `image`).
- **`offChainMetadata`**: An optional field for referencing off-chain metadata (currently unused in the provided code).
- **`metadataSignature`**: An optional signature to verify metadata integrity (currently unused but available for future enhancements).

### Issuing Tokens

Tokens are issued through the `Wallet` class using dedicated methods for each token type:

- **`issueExtendedFT`**: Issues a fungible token with a specified total supply and decimal precision.
- **`issueExtendedNFT`**: Issues a unique non-fungible token with metadata like creator and image URL.
- **`issueExtendedSFT`**: Issues a semi-fungible token with a total supply and metadata.
- **`issueExtendedNCFT`**: Issues a non-custodial fungible token with a specified quantity.

The issuance process follows these steps:

1. **Prepare Metadata**: Collect details such as `name`, `description`, and `image` into a `TokenMeta` struct.
2. **Create Token Data**: Construct an `ExtendedTokenData` object with the token’s properties.
3. **Generate Script**: Use `createExtendedTokenScriptPubKeyHex` to create a hexadecimal `scriptPubKey` embedding the token data in an OP_RETURN output.
4. **Build Transaction**: Create a transaction that spends a coin-based UTXO (for fees) and includes the token issuance output.
5. **Sign and Broadcast**: Sign the transaction with the wallet’s private key and broadcast it to the network, either via the local mempool or RPC.

#### Example: Issuing a Fungible Token

```cpp
std::string result = wallet.issueExtendedFT(
    "TRUFT",            // tokenID
    1000000,            // totalSupply (1,000,000 units)
    "TRU Fungible Token", // name
    "TRUFT",            // symbol
    "A fungible token on TRU Blockchain", // description
    "https://example.com/image.png", // image URL
    8                   // decimals
);
```

This creates a fungible token with 1,000,000 units and 8 decimal places, embedding its metadata on-chain.

### Managing Tokens

The wallet provides several methods to manage tokens:

- **Listing Tokens**: Use `listMyTokensFancy` to generate a formatted list of tokens owned by the wallet, displaying details like `tokenID`, `type`, `amount`, and metadata (e.g., name, symbol, image).
- **Transferring Tokens**: Use `transferExtendedToken` to send a specified quantity of a token to another address. For fungible tokens (FT and SFT), this can split the amount, creating a change output if necessary. For NFTs, the entire token (amount = 1) is transferred.
- **Finding UTXOs**: Use `findOneSpendableUtxo` to locate a coin-based UTXO for transaction fees, and `findTokenUTXO` to find a specific token’s UTXO by `tokenID`.

#### Example: Transferring a Token

```cpp
std::string txid = wallet.transferExtendedToken(
    "oldTxid",          // Transaction ID of the token UTXO
    0,                  // vout of the token UTXO
    500000,             // Quantity to send (e.g., 500,000 units of an FT)
    "recipientAddress"  // Destination address
);
```

### Integration with Blockchain

- **ScriptPubKey**: Tokens are embedded in transaction outputs using an OP_RETURN script, generated by `createExtendedTokenScriptPubKeyHex`. The script includes a compact binary format with token type, ID, amount, owner hash160, and a metadata hash.
- **Metadata Storage**: Metadata is stored on-chain via `storeTokenMetadata`, linked to the transaction ID and a hash generated by `generateMetaHash`. This allows retrieval via the blockchain’s `fetchTokenMetadata` method.
- **RPC Interaction**: The wallet supports RPC calls like `rpcBroadcastTx` to broadcast transactions and `rpcGetUtxoValue` to query UTXO values, enabling operation without a local blockchain instance.

### Utilities

- **Script Generation**: `createExtendedTokenScriptPubKeyHex` builds the binary OP_RETURN script for token issuance.
- **Address Extraction**: `extractAddressFromScriptPubKey` retrieves the controlling address from a `scriptPubKey`, supporting both standard P2PKH and token scripts.
- **Metadata Hashing**: `generateMetaHash` creates a SHA256 hash of the metadata for on-chain reference and verification.

### Best Practices

- **Secure Wallet Management**: Regularly back up your wallet file (`tru.dat`) and safeguard your master seed to prevent loss of access to tokens.
- **Transaction Fees**: Ensure sufficient coin-based UTXOs are available to cover fees (e.g., 10,000 satoshis per transaction).
- **Metadata Integrity**: While `metadataSignature` is available, consider implementing signing logic to enhance security for critical token metadata.

---

## RPC Reference

The RPC server (in `tru_advanced`) accepts POST requests at `http://127.0.0.1:8332/rpc`. Key methods include:

| **Method**                  | **Description**                              | **Parameters**                       |
|-----------------------------|----------------------------------------------|--------------------------------------|
| `getblocktemplate`          | Get block template for mining                | None                                 |
| `submitblock`               | Submit a mined block                         | `blockHex`                           |
| `getbalancebyaddress`       | Get balance for an address                   | `address`, `includeUnconfirmed`      |
| `getblock`                  | Get block by hash                            | `hash`, `verbose`                    |
| `getpeerinfo`               | List connected peers                         | None                                 |
| `sendtransaction`           | Broadcast a transaction                      | `txHex`                              |
| `listmytokens2025`          | List tokens owned by an address              | `address`                            |
| `sendchatmessage`           | Send a chat message                          | `chatText`                           |
| `getchaininfo`              | Get blockchain info                          | None                                 |
| `startmining`               | Start mining (CPU/GPU)                       | `type`, `minerAddress`, `quiet`      |
| `listtransactions`          | List recent transactions                     | `mempool`, `count`                   |
| `listunspent`               | List UTXOs for an address                    | `address`                            |
| `createrawtransaction`      | Create a raw transaction                     | `inputs`, `outputs`                  |
| `signrawtransactionwithkey` | Sign a raw transaction                       | `txHex`, `privKeys`                  |
| `getmempooltransactions`    | Get mempool transactions                     | None                                 |
| `createsmartcontract`       | Create a smart contract transaction          | `scriptHex`, `senderAddress`         |
| `reportmineractivity`       | Report miner activity                        | `minerAddress`, `hashesTried`, `timeTaken` |

- **Example**:
  ```bash
  curl -X POST http://127.0.0.1:8332/rpc -H "Content-Type: application/json" -d '{"method": "getchaininfo", "params": {}}'
  ```

---

## Troubleshooting

- **RPC Connection Failure**:
  - Ensure `tru_advanced` is running and accessible at `127.0.0.1:8332`.
- **Block Explorer Web Pages Missing**:
  - Copy the web files to `/home/gw878/TRU/web` on the target machine or embed them in the executable.
- **GPU Miner Fails**:
  - Verify OpenCL drivers and GPU compatibility.
- **Wallet CLI Errors**:
  - Check `tru.dat` permissions and node connectivity.

---

## Notes

- **IP Addresses**: Examples use `127.0.0.1` to avoid exposing real IPs.
- **Block Explorer Portability**: Web files must be present or embedded; RPC ensures database access.
- **Database**: The explorer uses RPC, not direct LevelDB access, so the node handles database communication.
- **Wallet Security**: Private keys are managed locally in `tru_wallet`; transactions are signed offline and broadcast via RPC.

**Update Note: Future Token Enhancements**

The TRU Blockchain is excited to announce upcoming enhancements to our token system, specifically for Non-Custodial Fungible Tokens (NCFT) and Semi-Fungible Tokens (SFT). These future updates will introduce advanced AI-driven and dynamic features, expanding the possibilities for token functionality and creativity. Here's what’s on the horizon:

### Planned Enhancements for NCFT

We’re planning to add AI and art-related fields to NCFT tokens, enabling them to evolve dynamically and incorporate artistic elements. These fields will allow creators to design tokens with unique, adaptive characteristics. The planned fields include:

- **`ai_engine`**: Specifies the AI model powering the token’s dynamic features (e.g., "StableDiffusion-v2").
- **`style_descriptor`**: Defines the artistic style or theme of the token (e.g., "Van Gogh meets fractal geometry").
- **`dynamic_morph`**: Indicates the mechanism driving the token’s evolution (e.g., "transfer-based evolution").
- **`update_interval`**: Sets how often or under what conditions the token updates (e.g., "1 transfer").
- **`creator_signature`**: Provides a signature for authenticity and provenance (e.g., "0xabcd...").
- **`last_evolution`**: Tracks the most recent change or evolution of the token (e.g., "None").

These additions will enable NCFT tokens to evolve over time, respond to interactions, or serve as canvases for generative art, making them ideal for creative and innovative use cases.

### Planned Enhancements for SFT

For SFT tokens, we’re introducing AI and "living" fields to create adaptive, responsive tokens that can adjust their behavior based on usage or other factors. These enhancements will make SFTs more versatile for applications like gaming, loyalty programs, or automated adjustments. The upcoming fields include:

- **`ai_version`**: Specifies the version of the AI model or logic used (e.g., "1.2").
- **`learning_mode`**: Defines how the token learns or adapts (e.g., "on-chain usage patterns").
- **`growth_algorithm`**: Indicates the algorithm controlling the token’s adaptation (e.g., "neural-adaptive").
- **`adaptation_rate`**: Sets the speed or rate of the token’s changes (e.g., "0.05").
- **`evolution_epoch`**: Tracks the current stage of the token’s evolution (e.g., "5000").
- **`description_ai`**: Offers additional details about the AI features (e.g., "This SFT can auto-adjust minting fees if usage is low.").

These fields will allow SFT tokens to become "living" entities on the blockchain, capable of real-time adaptation and intelligence based on their environment or user interactions.

### Looking Ahead

These enhancements are part of our development roadmap and will be rolled out in future versions of the TRU Blockchain. We’re working to ensure these features are secure, efficient, and valuable for our community. Stay tuned for more updates as we bring these exciting capabilities to life!

Enjoy exploring the TRU Blockchain! For issues, ensure dependencies are installed and the node is running. Contributions are welcome!

--- 




