# KYT Compliance Reference

## CLI Command → REST API Mapping

| CLI Command | REST Endpoint | Method |
|-------------|--------------|--------|
| `kyt risk --chain sol --address ADDR` | `POST /v2/kyt/address` + `GET /v2/kyt/addresses/ADDR/risk` | POST+GET |

## All KYT Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v2/kyt/address` | Register address for risk assessment |
| GET | `/v2/kyt/addresses/{address}/risk` | Get address risk score and categories |
| POST | `/v2/kyt/transfer` | Register transfer for compliance screening |
| GET | `/v2/kyt/transfers/{transferId}/summary` | Transfer risk summary |
| GET | `/v2/kyt/transfers/{transferId}/alerts` | Transfer compliance alerts |
| GET | `/v2/kyt/transfers/{transferId}/exposures/direct` | Direct exposure to risky entities |
| POST | `/v2/kyt/withdrawal` | Register withdrawal for screening |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/summary` | Withdrawal risk summary |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/alerts` | Withdrawal alerts |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/exposures/direct` | Withdrawal direct exposure |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/fraud-assessment` | Fraud assessment score |

## Risk Levels

| Level | Score Range | Meaning | Action |
|-------|------------|---------|--------|
| Low | 0-25 | Minimal risk | Proceed |
| Medium | 26-50 | Some flags | Inform user |
| High | 51-75 | Significant risk | Warn user strongly |
| Critical | 76-100 | Known bad actor / sanctioned | Block transaction, require explicit override |

## When to Use KYT

- Before recommending a token for purchase (check token creator address)
- Before executing a swap to an unknown token (check output token contract)
- When analyzing a wallet's risk profile
- For compliance screening in regulated workflows

## Billing

KYT calls have separate billing:
- Address risk assessment: $0.25 per call
- Transfer/withdrawal screening: $1.25 per call
