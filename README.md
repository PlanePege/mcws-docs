# Multichain Wallet Backend API Documentation

Complete API reference for all blockchain endpoints. All requests are queued for processing with rate limiting and load balancing.

---

## Table of Contents

- [Ethereum (ETH)](#ethereum-eth)
- [Binance Smart Chain (BSC)](#binance-smart-chain-bsc)
- [Base](#base)
- [Solana](#solana)
- [Common Response Formats](#common-response-formats)
- [Error Handling](#error-handling)

---

## Ethereum (ETH)

Base URL: `/api/eth`

### 1. Health Check

**Endpoint:** `GET /api/eth/health`

**Description:** Check ETH RPC connection health and endpoint status.

**Request Parameters:** None

**Response:**
```json
{
  "status": "healthy",
  "endpoints": [
    {
      "name": "string",
      "url": "string",
      "isHealthy": true,
      "responseTime": 123,
      "priority": 1,
      "consecutiveFailures": 0,
      "lastHealthCheck": "2025-01-09T12:00:00.000Z",
      "totalRequests": 100,
      "successfulRequests": 98,
      "failedRequests": 2,
      "successRate": 0.98,
      "lastUsed": "2025-01-09T12:00:00.000Z"
    }
  ],
  "summary": {
    "totalEndpoints": 3,
    "healthyEndpoints": 3,
    "unhealthyEndpoints": 0,
    "healthPercentage": 100
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 2. Create Wallet

**Endpoint:** `GET /api/eth/wallet/create`

**Description:** Create a new Ethereum wallet with encrypted private key.

**Request Parameters:** None

**Response:**
```json
{
  "success": true,
  "wallet": {
    "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
    "privateKey": "encrypted_private_key_string",
    "encrypted": true
  },
  "message": "Wallet created successfully",
  "encrypted": true,
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 3. Get Balance

**Endpoint:** `GET /api/eth/balance/:address`

**Description:** Get ETH balance for a wallet address.

**Request Parameters:**
- `address` (path parameter, required): Ethereum wallet address (0x...)

**Response:**
```json
{
  "success": true,
  "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
  "balance": {
    "eth": "1.234567890123456789",
    "wei": "1234567890123456789"
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 4. Get All Token Balances

**Endpoint:** `GET /api/eth/tokens/:address`

**Description:** Get all ERC20 token balances for a wallet using Etherscan Pro API.

**Request Parameters:**
- `address` (path parameter, required): Ethereum wallet address (0x...)

**Response:**
```json
{
  "success": true,
  "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
  "tokens": [
    {
      "contractAddress": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
      "name": "Tether USD",
      "symbol": "USDT",
      "decimals": 6,
      "balance": "1000000000",
      "balanceFormatted": "1000"
    }
  ],
  "count": 1,
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 5. Get Specific Token Balance

**Endpoint:** `GET /api/eth/token/:address/:tokenAddress`

**Description:** Get balance for a specific ERC20 token.

**Request Parameters:**
- `address` (path parameter, required): Ethereum wallet address (0x...)
- `tokenAddress` (path parameter, required): ERC20 token contract address (0x...)

**Response:**
```json
{
  "success": true,
  "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
  "tokenAddress": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
  "tokenBalance": {
    "name": "Tether USD",
    "symbol": "USDT",
    "decimals": 6,
    "balance": "1000000000"
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 6. Transfer ETH or Tokens

**Endpoint:** `POST /api/eth/transfer`

**Description:** Send ETH or ERC20 tokens to another address with smart amount conversion.

**Request Body:**
```json
{
  "privateKey": "encrypted_or_plain_private_key",
  "to": "0xRecipientAddress",
  "amount": "eth<1.5> or usdt<100> or token<10>",
  "token": "0xTokenContractAddress (optional, required for token<> transfers)"
}
```

**Required Fields:**
- `privateKey` (string): Wallet private key (encrypted or plain text)
- `to` (string): Recipient address
- `amount` (string): Amount with format prefix:
  - `eth<amount>` - ETH amount (e.g., "eth<1.5>")
  - `bsc<amount>` - BNB amount converted to ETH equivalent (e.g., "bsc<2>")
  - `base<amount>` - BASE amount (e.g., "base<1>")
  - `usdt<amount>` - USDT amount converted to ETH (e.g., "usdt<100>")
  - `sol<amount>` - SOL amount converted to ETH (e.g., "sol<5>")
  - `token<amount>` - Exact token amount, no conversion (e.g., "token<10>" sends exactly 10 tokens)

**Optional Fields:**
- `token` (string): Token contract address (required for `token<>` format, omit for native ETH transfer)

**Response:**
```json
{
  "success": true,
  "transactionHash": "0x...",
  "from": "0xSenderAddress",
  "to": "0xRecipientAddress",
  "amount": "1.5",
  "token": "ETH",
  "amountInfo": {
    "original": "usdt<100>",
    "originalType": "usdt",
    "originalValue": 100,
    "converted": 0.0285,
    "convertedFrom": "100 USDT (~$100.00)",
    "isTokenTransfer": false
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 7. Get Swap Quote

**Endpoint:** `POST /api/eth/swap/quote`

**Description:** Get swap quote from Uniswap with smart amount conversion.

**Request Body:**
```json
{
  "tokenIn": "0xTokenInAddress",
  "tokenOut": "0xTokenOutAddress",
  "amountIn": "eth<1> or usdt<100> or token<10>",
  "slippage": 0.5,
  "deadline": 1800
}
```

**Required Fields:**
- `tokenIn` (string): Input token address
- `tokenOut` (string): Output token address
- `amountIn` (string): Amount with format prefix:
  - `eth<amount>` - ETH amount (e.g., "eth<1>")
  - `bsc<amount>` - BNB amount converted to ETH equivalent (e.g., "bsc<2>")
  - `base<amount>` - BASE amount (e.g., "base<1>")
  - `usdt<amount>` - USDT amount converted to ETH (e.g., "usdt<100>")
  - `sol<amount>` - SOL amount converted to ETH (e.g., "sol<5>")
  - `token<amount>` - Exact token amount (e.g., "token<10>")
- `slippage` (number): Slippage tolerance (e.g., 0.5 for 0.5%)

**Optional Fields:**
- `deadline` (number): Transaction deadline in seconds (default: 1800)

**Response:**
```json
{
  "success": true,
  "quote": {
    "amountOut": "95.5",
    "minimumAmountOut": "94.5",
    "priceImpact": "0.2",
    "route": ["0xToken1", "0xToken2"]
  },
  "amountInfo": {
    "original": "usdt<100>",
    "originalType": "usdt",
    "originalValue": 100,
    "converted": 0.0285,
    "convertedFrom": "100 USDT (~$100.00)",
    "isTokenTransfer": false
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 8. Execute Swap

**Endpoint:** `POST /api/eth/swap/execute`

**Description:** Execute token swap on Uniswap with smart amount conversion.

**Request Body:**
```json
{
  "privateKey": "encrypted_or_plain_private_key",
  "tokenIn": "0xTokenInAddress",
  "tokenOut": "0xTokenOutAddress",
  "amountIn": "eth<1> or usdt<100> or token<10>",
  "slippage": 0.5,
  "deadline": 1800
}
```

**Required Fields:**
- `privateKey` (string): Wallet private key (encrypted or plain text)
- `tokenIn` (string): Input token address
- `tokenOut` (string): Output token address
- `amountIn` (string): Amount with format prefix (see Get Swap Quote for formats)
- `slippage` (number): Slippage tolerance

**Optional Fields:**
- `deadline` (number): Transaction deadline in seconds

**Response:**
```json
{
  "success": true,
  "transactionHash": "0x...",
  "amountIn": "1",
  "amountOut": "95.5",
  "amountInfo": {
    "original": "eth<1>",
    "originalType": "eth",
    "originalValue": 1,
    "converted": 1,
    "isTokenTransfer": false
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 9. Complete Swap

**Endpoint:** `POST /api/eth/swap`

**Description:** Get quote and execute swap in one call with smart amount conversion.

**Request Body:**
```json
{
  "privateKey": "encrypted_or_plain_private_key",
  "tokenIn": "0xTokenInAddress",
  "tokenOut": "0xTokenOutAddress",
  "amountIn": "usdt<100>",
  "slippage": 0.5,
  "deadline": 1800
}
```

**Required Fields:** Same as Execute Swap

**Response:**
```json
{
  "success": true,
  "transactionHash": "0x...",
  "amountIn": "0.0285",
  "amountOut": "95.5",
  "quote": {
    "amountOut": "95.5",
    "minimumAmountOut": "94.5",
    "priceImpact": "0.2",
    "route": ["0xToken1", "0xToken2"]
  },
  "amountInfo": {
    "original": "usdt<100>",
    "originalType": "usdt",
    "originalValue": 100,
    "converted": 0.0285,
    "convertedFrom": "100 USDT (~$100.00)",
    "isTokenTransfer": false
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

---

## Binance Smart Chain (BSC)

Base URL: `/api/bsc`

All BSC endpoints follow the same structure as Ethereum endpoints, with the following differences:

### Balance Response Format

```json
{
  "success": true,
  "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
  "balance": {
    "bnb": "2.5",
    "wei": "2500000000000000000"
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### Transfer Request Format

BSC transfers support the same smart amount conversion as ETH:

```json
{
  "privateKey": "encrypted_or_plain_private_key",
  "to": "0xRecipientAddress",
  "amount": "bsc<2> or eth<1> or usdt<100> or token<10>",
  "token": "0xTokenContractAddress (optional)"
}
```

**Amount Formats:**
- `bsc<amount>` - BNB amount (e.g., "bsc<2>")
- `eth<amount>` - ETH amount converted to BNB equivalent (e.g., "eth<1>")
- `base<amount>` - BASE amount converted to BNB (e.g., "base<1>")
- `usdt<amount>` - USDT amount converted to BNB (e.g., "usdt<100>")
- `sol<amount>` - SOL amount converted to BNB (e.g., "sol<5>")
- `token<amount>` - Exact token amount (e.g., "token<10>")

### Transfer Response

Native token is `BNB` instead of `ETH`. Response includes `amountInfo` with conversion details.

### Swap Support

BSC supports all swap endpoints (`/swap/quote`, `/swap/execute`, `/swap`) using PancakeSwap instead of Uniswap. All swap endpoints support the same smart amount formatting as transfers (bsc<>, eth<>, usdt<>, token<>, etc.).

**All other endpoints are identical to ETH endpoints.**

---

## Base

Base URL: `/api/base`

All Base endpoints follow the same structure as Ethereum endpoints, with the following differences:

### Balance Response Format

```json
{
  "success": true,
  "publicKey": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
  "balance": {
    "base": "3.14",
    "wei": "3140000000000000000"
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### Transfer Request Format

Base transfers support the same smart amount conversion:

```json
{
  "privateKey": "encrypted_or_plain_private_key",
  "to": "0xRecipientAddress",
  "amount": "base<1> or eth<1> or usdt<100> or token<10>",
  "token": "0xTokenContractAddress (optional)"
}
```

**Amount Formats:**
- `base<amount>` - BASE (ETH) amount (e.g., "base<1>")
- `eth<amount>` - ETH amount (e.g., "eth<1>")
- `bsc<amount>` - BNB amount converted to BASE equivalent (e.g., "bsc<2>")
- `usdt<amount>` - USDT amount converted to BASE (e.g., "usdt<100>")
- `sol<amount>` - SOL amount converted to BASE (e.g., "sol<5>")
- `token<amount>` - Exact token amount (e.g., "token<10>")

### Transfer Response

Native token is `BASE` instead of `ETH`. Response includes `amountInfo` with conversion details.

### Swap Support

Base supports all swap endpoints (`/swap/quote`, `/swap/execute`, `/swap`) using Uniswap on Base network. All swap endpoints support the same smart amount formatting as transfers (base<>, eth<>, usdt<>, token<>, etc.).

**All other endpoints are identical to ETH endpoints.**

---

## Solana

Base URL: `/api/solana`

### 1. Health Check

**Endpoint:** `GET /api/solana/health`

**Description:** Check Solana RPC connection health and endpoint status.

**Request Parameters:** None

**Response:** Same format as ETH health check.

### 2. Create Wallet

**Endpoint:** `GET /api/solana/wallet/create`

**Description:** Create a new Solana wallet with encrypted private key.

**Request Parameters:** None

**Response:**
```json
{
  "success": true,
  "wallet": {
    "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
    "privateKey": "encrypted_base58_private_key",
    "encrypted": true
  },
  "message": "Wallet created successfully",
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 3. Get Balance

**Endpoint:** `GET /api/solana/balance/:publicKey`

**Description:** Get SOL balance for a wallet.

**Request Parameters:**
- `publicKey` (path parameter, required): Solana wallet public key

**Response:**
```json
{
  "success": true,
  "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "balance": {
    "lamports": "1000000000",
    "sol": 1.0
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 4. Get All Token Balances

**Endpoint:** `GET /api/solana/tokens/:publicKey`

**Description:** Get all SPL token balances for a wallet.

**Request Parameters:**
- `publicKey` (path parameter, required): Solana wallet public key

**Response:**
```json
{
  "success": true,
  "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "tokens": [
    {
      "mint": "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB",
      "symbol": "USDT",
      "name": "USDT",
      "decimals": 6,
      "balance": "1000000000",
      "balanceFormatted": "1000"
    }
  ],
  "count": 1,
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 5. Transfer SOL or Tokens

**Endpoint:** `POST /api/solana/transfer`

**Description:** Send SOL or SPL tokens to another address.

**Request Body:**
```json
{
  "privateKey": "encrypted_or_plain_base58_private_key",
  "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "toAddress": "RecipientPublicKey",
  "amount": "sol<1.5> or usdt<100> or token<50>",
  "tokenMint": "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB (optional)"
}
```

**Required Fields:**
- `privateKey` (string): Wallet private key in base58 format (encrypted or plain text)
- `publicKey` (string): Sender's public key
- `toAddress` (string): Recipient's public key
- `amount` (string): Amount with format prefix:
  - `sol<amount>` - for SOL (e.g., "sol<1.5>")
  - `usdt<amount>` - for USDT (e.g., "usdt<100>")
  - `token<amount>` - for other tokens (e.g., "token<50>")

**Optional Fields:**
- `tokenMint` (string): Token mint address (required for token transfers, omit for SOL)

**Response:**
```json
{
  "success": true,
  "signature": "transaction_signature",
  "from": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "to": "RecipientPublicKey",
  "amount": "1.5",
  "tokenMint": "SOL",
  "amountInfo": {
    "original": "sol<1.5>",
    "originalType": "sol",
    "originalValue": 1.5,
    "converted": 1.5,
    "isTokenTransfer": false
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 6. Check Transfer

**Endpoint:** `POST /api/solana/transfer/check`

**Description:** Check if a transfer can be executed (validates balance and fees).

**Request Body:**
```json
{
  "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "mintAddress": "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB",
  "amount": "usdt<100>"
}
```

**Required Fields:**
- `publicKey` (string): Sender's public key
- `mintAddress` (string): Token mint address
- `amount` (string): Amount with format prefix (same as transfer)

**Response:**
```json
{
  "success": true,
  "canTransfer": true,
  "balance": "1000",
  "required": "100",
  "sufficient": true,
  "amountInfo": {
    "original": "usdt<100>",
    "originalType": "usdt",
    "originalValue": 100,
    "converted": 100,
    "isTokenTransfer": true
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 7. Get Swap Quote

**Endpoint:** `POST /api/solana/swap/quote`

**Description:** Get swap quote from Jupiter aggregator.

**Request Body:**
```json
{
  "inputMint": "So11111111111111111111111111111111111111112",
  "outputMint": "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB",
  "amount": "sol<1.0>",
  "slippageBps": 50
}
```

**Required Fields:**
- `inputMint` (string): Input token mint address
- `outputMint` (string): Output token mint address
- `amount` (string): Amount with format prefix (e.g., "sol<1.0>", "usdt<100>", "token<50>")

**Optional Fields:**
- `slippageBps` (number): Slippage in basis points (default: 50 = 0.5%)

**Response:**
```json
{
  "success": true,
  "quote": {
    "inAmount": "1000000000",
    "outAmount": "145000000",
    "priceImpact": "0.15",
    "route": ["SOL", "USDT"]
  },
  "amountInfo": {
    "original": "sol<1.0>",
    "originalType": "sol",
    "originalValue": 1.0,
    "converted": 1.0
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 8. Execute Swap

**Endpoint:** `POST /api/solana/swap/execute`

**Description:** Execute token swap via Jupiter.

**Request Body:**
```json
{
  "privateKey": "encrypted_or_plain_base58_private_key",
  "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "instructionResponse": {
    "swapTransaction": "base64_encoded_transaction",
    "requestId": "request_id",
    "inAmount": "1000000000",
    "outAmount": "145000000",
    "feeBps": 25,
    "router": "Jupiter"
  }
}
```

**Required Fields:**
- `privateKey` (string): Wallet private key (encrypted or plain text)
- `publicKey` (string): Wallet public key
- `instructionResponse` (object): Response from Jupiter swap instruction endpoint

**Response:**
```json
{
  "success": true,
  "result": {
    "signature": "transaction_signature",
    "status": "confirmed"
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

### 9. Complete Swap

**Endpoint:** `POST /api/solana/swap`

**Description:** Get quote and execute swap in one call using Jupiter Ultra API.

**Request Body:**
```json
{
  "privateKey": "encrypted_or_plain_base58_private_key",
  "publicKey": "7EqQdEUAxGce4kCXGFqDT9qfZqJQzxTqmPJqWqzjrKqQ",
  "inputMint": "So11111111111111111111111111111111111111112",
  "outputMint": "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB",
  "amount": "sol<1.0>",
  "slippageBps": 50
}
```

**Required Fields:**
- `privateKey` (string): Wallet private key (encrypted or plain text)
- `publicKey` (string): Wallet public key
- `inputMint` (string): Input token mint address
- `outputMint` (string): Output token mint address
- `amount` (string): Amount with format prefix (e.g., "sol<1.0>", "usdt<100>", minimum: 0.00001 SOL = 10000 lamports)

**Optional Fields:**
- `slippageBps` (number): Slippage in basis points (default: 50)

**Response:**
```json
{
  "success": true,
  "order": {
    "requestId": "request_id",
    "inAmount": "1000000000",
    "outAmount": "145000000",
    "feeBps": 25,
    "router": "Jupiter"
  },
  "executionResult": {
    "signature": "transaction_signature",
    "status": "confirmed"
  },
  "amountInfo": {
    "original": "sol<1.0>",
    "originalType": "sol",
    "originalValue": 1.0,
    "converted": 1.0,
    "lamports": 1000000000
  },
  "timestamp": "2025-01-09T12:00:00.000Z"
}
```

---

## Common Response Formats

### Success Response

All successful responses include:
- `success: true`
- Relevant data fields
- `timestamp`: ISO 8601 formatted timestamp

### Error Response

All error responses include:
```json
{
  "success": false,
  "error": "Error type or message",
  "message": "Detailed error description (optional)"
}
```

**HTTP Status Codes:**
- `200` - Success
- `400` - Bad Request (missing or invalid parameters)
- `500` - Internal Server Error

---

## Error Handling

### Common Error Types

1. **Missing Parameters**
```json
{
  "error": "Missing required fields: privateKey, to, amount"
}
```

2. **Invalid Address**
```json
{
  "success": false,
  "error": "Invalid wallet address: 0xinvalid. Must start with '0x' and be 42 characters long"
}
```

3. **Provider Not Configured**
```json
{
  "success": false,
  "error": "ETH provider not configured",
  "message": "ETH provider not configured"
}
```

4. **Transaction Failed**
```json
{
  "success": false,
  "error": "Transfer failed"
}
```

5. **API Error**
```json
{
  "success": false,
  "error": "ETH Explorer API error: Invalid API key",
  "message": "Failed to get tokens"
}
```

---

## Notes

### Private Key Encryption

- All endpoints accept both encrypted and plain text private keys
- Encrypted private keys are automatically detected and decrypted
- Wallet creation endpoints always return encrypted private keys
- Use the `EncryptionUtil` to encrypt/decrypt private keys

### Amount Formatting

All chains now support **unified smart amount formatting** with automatic conversion:

**Format:** `<type><amount>` where type can be:
- `eth<amount>` - Ethereum amount (e.g., "eth<1.5>")
- `bsc<amount>` - BNB amount (e.g., "bsc<2>")
- `base<amount>` - Base (ETH) amount (e.g., "base<1>")
- `sol<amount>` - Solana amount (e.g., "sol<5>")
- `usdt<amount>` - USDT amount in USD (e.g., "usdt<100>")
- `token<amount>` - Exact token amount, no conversion (e.g., "token<10>")

**How It Works:**
1. **Native Token Transfers:** Use matching type (e.g., "eth<1>" for ETH chain)
2. **Cross-Chain Conversion:** Use different type to auto-convert via USD value (e.g., "usdt<100>" converts to equivalent ETH/BNB/BASE based on current prices)
3. **Token Transfers:** Use "token<amount>" to send exact token quantity regardless of value (e.g., "token<10>" sends exactly 10 tokens from your balance)

**Examples:**
```json
// Send $100 worth of ETH
{"amount": "usdt<100>"}

// Send exactly 10 tokens (from 10,000 balance)
{"amount": "token<10>", "token": "0xTokenAddress"}

// Send 1 ETH worth of BNB on BSC chain
{"amount": "eth<1>"}

// Send 2 BNB on ETH chain (converts to ETH equivalent)
{"amount": "bsc<2>"}
```

**Price Fetching:**
- Real-time prices from CoinGecko API
- 1-minute price caching for efficiency
- Automatic USD conversion between all supported tokens

**Minimum Amounts:**
- Solana swaps: 0.00001 SOL (10000 lamports)
- EVM chains: No minimum enforced

### Rate Limiting

All requests are queued with priority-based processing:
- Priority 1 (Low): Health checks
- Priority 3 (Low-Medium): Balance and token queries
- Priority 5 (Normal): Wallet creation, swap quotes
- Priority 7 (Medium-High): Transfer validation
- Priority 10 (High): Transfers and swap execution

### Request Context

All requests automatically receive a `requestId` for tracking and logging purposes.
