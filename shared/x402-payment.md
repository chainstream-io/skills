# x402 Payment Protocol

When the API returns `402 Payment Required`, the user needs to purchase a quota plan with USDC.

## Flow

1. API returns 402 with `X-Payment-Required` header containing plan options
2. User selects a plan
3. CLI/SDK calls `POST /x402/purchase` with wallet signature
4. User's wallet signs a USDC transfer to ChainStream's receiving address
5. quota-service credits the purchased CU to the wallet address
6. Subsequent requests succeed with wallet auth headers

## Plans

| Plan | USDC | Compute Units | Duration |
|------|------|--------------|----------|
| nano | $1 | 50,000 | 30 days |
| micro | $5 | 350,000 | 30 days |
| starter | $20 | 1,500,000 | 30 days |
| growth | $50 | 4,000,000 | 30 days |
| pro | $150 | 15,000,000 | 30 days |
| business | $500 | 55,000,000 | 30 days |

## Supported Payment Networks

- **EVM**: Base mainnet (USDC)
- **Solana**: Solana mainnet (USDC)

## Error Recovery

- If payment fails: check wallet USDC balance, retry
- If 402 persists after payment: wait 5s for quota-service to process, retry
- Plan pricing available at `GET /x402/pricing`
