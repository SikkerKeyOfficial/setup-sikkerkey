# Setup SikkerKey

Install the SikkerKey CLI and inject secrets into your GitHub Actions workflows. Secrets are fetched on demand from your SikkerKey vault using Ed25519 machine authentication.

## Usage

```yaml
- uses: SikkerKeyOfficial/setup-sikkerkey@v1
  with:
    vault-id: ${{ secrets.SIKKERKEY_VAULT_ID }}
    project-id: ${{ secrets.SIKKERKEY_PROJECT_ID }}
    machine-id: ${{ secrets.SIKKERKEY_MACHINE_ID }}
    private-key: ${{ secrets.SIKKERKEY_PRIVATE_KEY }}
    export: true

- run: echo "Connected to database at $DB_CREDS_HOST"
```

All exported values are automatically masked in GitHub Actions logs.

## Setup

1. From the SikkerKey dashboard, go to **Machines > + Validate** and select **CI/CD**
2. Run the bootstrap command -- it prints your vault ID, machine ID, and private key
3. Approve the machine in the dashboard and grant it access to your secrets
4. Add the credentials as GitHub repository secrets:

| Secret | Value |
|--------|-------|
| `SIKKERKEY_VAULT_ID` | Vault ID from bootstrap output |
| `SIKKERKEY_PROJECT_ID` | Your project ID (from dashboard sidebar) |
| `SIKKERKEY_MACHINE_ID` | Machine ID from bootstrap output |
| `SIKKERKEY_PRIVATE_KEY` | Full PEM private key from bootstrap output |

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `vault-id` | Yes | | Vault ID |
| `project-id` | Yes | | Project ID to unlock |
| `machine-id` | Yes | | Machine UUID |
| `private-key` | Yes | | Ed25519 private key (PEM) |
| `api-url` | No | `https://api.sikkerkey.com` | API URL |
| `version` | No | `latest` | CLI version to install |
| `export` | No | `false` | Export secrets as environment variables |
| `export-prefix` | No | | Prefix for exported variable names |

## Outputs

| Output | Description |
|--------|-------------|
| `secrets-count` | Number of secrets exported (when `export: true`) |

## Examples

### Export secrets as environment variables

```yaml
- uses: SikkerKeyOfficial/setup-sikkerkey@v1
  with:
    vault-id: ${{ secrets.SIKKERKEY_VAULT_ID }}
    project-id: ${{ secrets.SIKKERKEY_PROJECT_ID }}
    machine-id: ${{ secrets.SIKKERKEY_MACHINE_ID }}
    private-key: ${{ secrets.SIKKERKEY_PRIVATE_KEY }}
    export: true

- run: ./deploy.sh
```

### Export with prefix

```yaml
- uses: SikkerKeyOfficial/setup-sikkerkey@v1
  with:
    vault-id: ${{ secrets.SIKKERKEY_VAULT_ID }}
    project-id: ${{ secrets.SIKKERKEY_PROJECT_ID }}
    machine-id: ${{ secrets.SIKKERKEY_MACHINE_ID }}
    private-key: ${{ secrets.SIKKERKEY_PRIVATE_KEY }}
    export: true
    export-prefix: APP_

- run: echo "$APP_DATABASE_PASSWORD"
```

### Use the CLI directly

```yaml
- uses: SikkerKeyOfficial/setup-sikkerkey@v1
  with:
    vault-id: ${{ secrets.SIKKERKEY_VAULT_ID }}
    project-id: ${{ secrets.SIKKERKEY_PROJECT_ID }}
    machine-id: ${{ secrets.SIKKERKEY_MACHINE_ID }}
    private-key: ${{ secrets.SIKKERKEY_PRIVATE_KEY }}

- run: |
    DB_PASS=$(sikkerkey get sk_db_prod password)
    echo "::add-mask::$DB_PASS"

- run: sikkerkey run --secret sk_db_prod -- ./deploy.sh
```

### Multiple projects

```yaml
- uses: SikkerKeyOfficial/setup-sikkerkey@v1
  with:
    vault-id: ${{ secrets.SIKKERKEY_VAULT_ID }}
    project-id: ${{ secrets.SIKKERKEY_PROJECT_ID }}
    machine-id: ${{ secrets.SIKKERKEY_MACHINE_ID }}
    private-key: ${{ secrets.SIKKERKEY_PRIVATE_KEY }}

- run: |
    sikkerkey unlock proj_staging --alias staging
    sikkerkey get sk_api_key --project staging
```

## Security

- The private key is stored in GitHub Secrets and never printed to logs
- All exported secret values are masked via `::add-mask::`
- The identity is created in the runner's home directory and discarded when the job ends
- Every secret read is authenticated with Ed25519 signatures and recorded in the SikkerKey audit log

## License

MIT
