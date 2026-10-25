# Audit gateway Python client

Single Python file, Python 3.9+, no dependencies. Calls the existing gateway on
port 8402 using `GET /api/audit-contract?address=...`.

## Quick test

```sh
python3 demo_audit_client.py --base-url http://127.0.0.1:8402
```

Run this on the machine hosting the gateway. A working unpaid request returns
HTTP 402 and exits with code **2**, showing the **0.50 USDC on Base** quote.
No funds are sent. The client checks the quoted recipient, token, target and
price ceiling before displaying payment details. Exit 1 indicates an error.

To access a remote gateway without publishing its port, forward it over SSH:

```sh
ssh -N -L 18402:127.0.0.1:8402 USER@YOUR_SERVER
python3 demo_audit_client.py --base-url http://127.0.0.1:18402
```

The public repository hosts this client and documentation, not the API server.
Clients need a reachable gateway origin. No private keys or server credentials
are included.

## Requesting a report after an independently arranged payment

```sh
python3 demo_audit_client.py --base-url http://127.0.0.1:8402 \
  --address 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 \
  --tx-hash YOUR_EXISTING_0x_TRANSACTION_HASH
```

The hash is sent as `X-PAYMENT-TX`. The client does not create wallets, sign,
transfer, approve tokens, or retry payments. It reports the server's assertion
of verification; it does not independently validate receipts or book revenue.
A report response exits 0 only if its address and transaction match the request.

## Compatibility and limits

The inspected gateway labels its JSON `x402-v1`, but uses a custom transaction-hash
header and body quote. This client is specific to that interface and does not
claim compatibility with standard x402 v2 authorization or facilitator flows.

The gateway performs basic bytecode heuristics, including substring checks that
can mistake PUSH data for opcodes. Its results are not a security audit or a
guarantee of contract safety. The inspected payment handler does not bind a
payment to the requesting customer and allows reusing a transaction hash.
This demo is for interface testing; production payment isolation and report
accuracy need additional work before commercial reliance.

The live unpaid 402 path is tested separately from mocked report responses.
No paid end-to-end transaction has been performed for this demonstration.

## Local verification

```sh
python3 -m unittest -v
```

Tests cover a real local HTTP 402 exchange, destination/token/price mismatch,
invalid inputs, mismatched reports and payment rejection. No tests transfer funds.
