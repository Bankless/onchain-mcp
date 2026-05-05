# Bankless Onchain MCP Server


## This project is no longer receiving updates

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Version](https://img.shields.io/badge/version-0.6.2-blue)

MCP (Model Context Protocol) server for blockchain data interaction through the Bankless API.

## Overview

The Bankless Onchain MCP Server provides a framework for interacting with on-chain data via the Bankless API. It implements the Model Context Protocol (MCP) to allow AI models to access blockchain state and event data in a structured way.


https://github.com/user-attachments/assets/95732dff-ae5f-45a6-928a-1ae17c0ddf9d


## Features

The server provides the following onchain data operations:

### Contract Operations

- **Read Contract State** (`read_contract`): Read state from smart contracts on various blockchain networks.
    - Parameters: network, contract address, method, inputs, outputs
    - Returns: Contract call results with typed values

- **Get Proxy** (`get_proxy`): Retrieve proxy implementation contract addresses.
    - Parameters: network, contract address
    - Returns: Implementation contract address

- **Get ABI** (`get_abi`): Fetch the ABI (Application Binary Interface) for a contract.
    - Parameters: network, contract address
    - Returns: Contract ABI in JSON format

- **Get Source** (`get_source`): Retrieve the source code for a verified contract.
    - Parameters: network, contract address
    - Returns: Source code, ABI, compiler version, and other contract metadata

### Event Operations

- **Get Events** (`get_events`): Fetch event logs for a contract based on topics.
    - Parameters: network, addresses, topic, optional topics
    - Returns: Filtered event logs

- **Build Event Topic** (`build_event_topic`): Generate an event topic signature from event name and argument types.
    - Parameters: network, event name, argument types
    - Returns: Event topic hash

### Transaction Operations

- **Get Transaction History** (`get_transaction_history`): Retrieve transaction history for a user address.
    - Parameters: network, user address, optional contract, optional method ID, optional start block, include data flag
    - Returns: List of transactions with hash, data, network, and timestamp

- **Get Transaction Info** (`get_transaction_info`): Get detailed information about a specific transaction.
    - Parameters: network, transaction hash
    - Returns: Transaction details including block number, timestamp, from/to addresses, value, gas info, status, and receipt data

## Tools

- **read_contract**
    - Read contract state from a blockchain
    - Input:
        - `network` (string, required): The blockchain network (e.g., "ethereum", "polygon")
        - `contract` (string, required): The contract address
        - `method` (string, required): The contract method to call
        - `inputs` (array, required): Input parameters for the method call, each containing:
            - `type` (string): The type of the input parameter (e.g., "address", "uint256")
            - `value` (any): The value of the input parameter
        - `outputs` (array, required): Expected output types, each containing:
            - `type` (string): The expected output type
    - Returns an array of contract call results

- **get_proxy**
    - Gets the proxy address for a given network and contract
    - Input:
        - `network` (string, required): The blockchain network (e.g., "ethereum", "base")
        - `contract` (string, required): The contract address
    - Returns the implementation address for the proxy contract

- **get_events**
    - Fetches event logs for a given network and filter criteria
    - Input:
        - `network` (string, required): The blockchain network (e.g., "ethereum", "base")
        - `addresses` (array, required): List of contract addresses to filter events
        - `topic` (string, required): Primary topic to filter events
        - `optionalTopics` (array, optional): Optional additional topics (can include null values)
    - Returns an object containing event logs matching the filter criteria

- **build_event_topic**
    - Builds an event topic signature based on event name and arguments
    - Input:
        - `network` (string, required): The blockchain network (e.g., "ethereum", "base")
        - `name` (string, required): Event name (e.g., "Transfer(address,address,uint256)")
        - `arguments` (array, required): Event arguments types, each containing:
            - `type` (string): The argument type (e.g., "address", "uint256")
    - Returns a string containing the keccak256 hash of the event signature

## Installation

```bash
npm install @bankless/onchain-mcp
```

## Usage

### Environment Setup

Before using the server, set your Bankless API token. For details on how to obtain your Bankless API token, head to https://docs.bankless.com/bankless-api/other-services/onchain-mcp

```bash
export BANKLESS_API_TOKEN=your_api_token_here
```

### Running the Server

The server can be run directly from the command line:

```bash
npx @bankless/onchain-mcp
```

### Usage with LLM Tools

This server implements the Model Context Protocol (MCP), which allows it to be used as a tool provider for compatible AI models. Here are some example calls for each tool:

#### read_contract

```javascript
// Example call
{
  "name": "read_contract",
  "arguments": {
    "network": "ethereum",
    "contract": "0x1234...",
    "method": "balanceOf",
    "inputs": [
      { "type": "address", "value": "0xabcd..." }
    ],
    "outputs": [
      { "type": "uint256" }
    ]
  }
}

// Example response
[
  {
    "value": "1000000000000000000",
    "type": "uint256"
  }
]
```

#### get_proxy

```javascript
// Example call
{
  "name": "get_proxy",
  "arguments": {
    "network": "ethereum",
    "contract": "0x1234..."
  }
}

// Example response
{
  "implementation": "0xefgh..."
}
```

#### get_events

```javascript
// Example call
{
  "name": "get_events",
  "arguments": {
    "network": "ethereum",
    "addresses": ["0x1234..."],
    "topic": "0xabcd...",
    "optionalTopics": ["0xef01...", null]
  }
}

// Example response
{
  "result": [
    {
      "removed": false,
      "logIndex": 5,
      "transactionIndex": 2,
      "transactionHash": "0x123...",
      "blockHash": "0xabc...",
      "blockNumber": 12345678,
      "address": "0x1234...",
      "data": "0x...",
      "topics": ["0xabcd...", "0xef01...", "0x..."]
    }
  ]
}
```

#### build_event_topic

```javascript
// Example call
{
  "name": "build_event_topic",
  "arguments": {
    "network": "ethereum",
    "name": "Transfer(address,address,uint256)",
    "arguments": [
      { "type": "address" },
      { "type": "address" },
      { "type": "uint256" }
    ]
  }
}

// Example response
"0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef"
```

## Development

### Building from Source

```bash
# Clone the repository
git clone https://github.com/Bankless/onchain-mcp.git
cd onchain-mcp

# Install dependencies
npm install

# Build the project
npm run build
```

### Debug Mode

```bash
npm run debug
```

### Integration with AI Models

To integrate this server with AI applications that support MCP, add the following to your app's server configuration:

```json
{
  "mcpServers": {
    "bankless": {
      "command": "npx",
      "args": [
        "@bankless/onchain-mcp"
      ],
      "env": {
        "BANKLESS_API_TOKEN": "your_api_token_here"
      }
    }
  }
}
```

## Error Handling

The server provides specific error types for different scenarios:

- `BanklessValidationError`: Invalid input parameters
- `BanklessAuthenticationError`: API token issues
- `BanklessResourceNotFoundError`: Requested resource not found
- `BanklessRateLimitError`: API rate limit exceeded

## Prompting Tips

In order to guide an LLM model to use the Bankless Onchain MCP Server, the following prompts can be used:

```
ROLE:
• You are Kompanion, a blockchain expert and EVM sleuth. 
• You specialize in navigating and analyzing smart contracts using your tools and resources.

HOW KOMPANION CAN HANDLE PROXY CONTRACTS:
• If a contract is a proxy, call your “get_proxy” tool to fetch the implementation contract.  
• If that fails, try calling the “implementation” method on the proxy contract.  
• If that also fails, try calling the “_implementation” function.  
• After obtaining the implementation address, call “get_contract_source” with that address to fetch its source code.  
• When reading or modifying the contract state, invoke implementation functions on the proxy contract address (not directly on the implementation).

HOW KOMPANION CAN HANDLE EVENTS:
• Get the ABI and Source of the relevant contracts
• From the event types in the ABI, construct the correct topics for the event relevant to the question
• use the "get_event_logs" tool to fetch logs for the contract

KOMPANION'S RULES:
• Do not begin any response with “Great,” “Certainly,” “Okay,” or “Sure.”  
• Maintain a direct, technical style. Do not add conversational flourishes.  
• If the user’s question is unrelated to smart contracts, do not fetch any contracts.  
• If you navigate contracts, explain each step in bullet points.  
• Solve tasks iteratively, breaking them into steps.  
• Use bullet points for lists of steps.  
• Never assume a contract’s functionality. Always verify with examples using your tools to read the contract state.  
• Before responding, consider which tools might help you gather better information.  
• Include as much relevant information as possible in your final answer, depending on your findings.

HOW KOMPANION CAN USE TOOLS:
• You can fetch contract source codes, ABIs, and read contract data by using your tools and functions.  
• Always verify the source or ABI to understand the contract rather than making assumptions.  
• If you need to read contract state, fetch its ABI (especially if the source is lengthy).  

FINAL INSTRUCTION:
• Provide the best possible, concise answer to the user’s request. If it's not an immediate question but an instruction, follow it directly.
• Use your tools to gather any necessary clarifications or data.  
• Offer a clear, direct response and add a summary of what you did (how you navigated the contracts) at the end.
```

## License

MIT
