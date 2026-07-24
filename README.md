# freeagent

A small CLI for the FreeAgent API, built in Go.

## Features

- OAuth login (local callback or manual paste)
- Keychain-backed token storage with file fallback
- Create and send invoices
- Explain (categorize) bank transactions and attach receipts
- Break-glass `raw` command for any FreeAgent endpoint
- JSON output mode for scripting / agents

## Install

```bash
go install github.com/anjor/freeagent-cli/cmd/freeagent@latest
```

Or build from source:

```bash
go build ./cmd/freeagent
```

## Configure

Create a FreeAgent API application and note the client ID + secret.

Save app credentials:

```bash
./freeagent auth configure \
  --client-id YOUR_ID \0001
  --client-secret YOUR_SECRET \0001
  --redirect http://127.0.0.1:8797/callback
```

You can also use env vars:

```bash
export FREEAGENT_CLIENT_ID=0001
export FREEAGENT_CLIENT_SECRET=0001
export FREEAGENT_USER_AGENT=MirrorMe
export FREEAGENT_REDIRECT_URI=http://127.0.0.1:8797/callback
```

> **Security:** Never commit OAuth client secrets. If a secret is exposed, revoke and rotate it immediately.

## Login

Local callback (default):

```bash
./freeagent auth login
```

Manual flow:

```bash
./freeagent auth login --manual
```

## Usage

Create a draft invoice:

```bash
./freeagent invoices create \
  --contact CONTACT_ID \
  --reference INV-001 \
  --lines ./invoice-lines.json
```

You can also pass a contact name or email and the CLI will resolve it:

```bash
./freeagent invoices create \
  --contact "Acme Ltd" \
  --reference INV-002 \
  --lines ./invoice-lines.json
```

Send an invoice email:

```bash
./freeagent invoices send --id INVOICE_ID --email-to you@company.com
```

Mark as sent (no email):

```bash
./freeagent invoices send --id INVOICE_ID
```

Break-glass request:

```bash
./freeagent raw --method GET --path /v2/invoices
```

Contacts:

```bash
./freeagent contacts list
./freeagent contacts search --query "Acme"
./freeagent contacts get --id CONTACT_ID
./freeagent contacts create --organisation "Acme Ltd" --email accounts@acme.test
```

Bank transactions (bulk approve):

```bash
./freeagent bank approve \
  --bank-account BANK_ACCOUNT_ID \
  --from 2025-01-01 \
  --to 2025-01-31

./freeagent bank approve --ids ./transaction-ids.txt
./freeagent bank approve --ids ./explanation-ids.txt --ids-type explanation
```

Explain (categorize) a transaction and optionally attach a receipt:

```bash
./freeagent bank explain \
  --transaction BANK_TRANSACTION_ID \
  --dated-on 2025-01-15 \
  --gross-value -6.17 \
  --category CATEGORY_ID \
  --description "AWS EMEA" \
  --attach ./receipt.pdf
```

`--gross-value` is passed through verbatim (negative for money out). Attachments
must be png, jpg, jpeg, gif or pdf and at most 5MB; the content type is detected
from the file extension, or override it with `--content-type`.

## Files

- Config: `~/.config/freeagent/config.json`
- Tokens (fallback): `~/.config/freeagent/tokens/PROFILE.json`

## Notes

- Default API base URL is production; use `--sandbox` for the sandbox API.
- Use `--json` to print raw JSON for automation or piping into other tools.

## License

MIT. See `LICENSE`.
