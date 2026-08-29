# sops_key — create your own (never committed)

`sops_key` is the age private key sops uses to decrypt this inventory's
`*.sops.yaml` secrets. The autobott Makefile derives its path from the
inventory: `<inventory-dir>/sops_key`.

`sops_key` is **gitignored** — every operator generates their own, and the
private key must never be committed. Only this file is tracked.

This is a boilerplate inventory: the recipient in `.sops.yaml` is a **sample**
public key whose private half was never shared, and secrets ship as **clear
text** in `host_vars/<host>/secrets.yaml` so the example runs out-of-box. When
you make this inventory your own, create your key and seal the secrets — the
steps below.

## Create your key

From the autobott (playbook) repo, pointing `INV` at this inventory:

```bash
make age-key INV=/path/to/your-inventory/inventory.yaml
```

That writes `sops_key` here and prints your **public** key (`age1...`). Or, if
you prefer, directly:

```bash
age-keygen -o sops_key
```

## Seal the secrets for the first time

1. Put your public key in this repo's `.sops.yaml` `age:` recipients (replacing
   the sample one).
2. Rename the plaintext secrets to the `.sops.yaml` suffix, then encrypt in place:

   ```bash
   git mv host_vars/ansible-autobott-example.lan/secrets.yaml \
          host_vars/ansible-autobott-example.lan/secrets.sops.yaml
   make seal-secrets INV=/path/to/your-inventory/inventory.yaml HOST=ansible-autobott-example.lan
   ```

Afterwards edit or view secrets with `make edit-secrets` / `make view-secrets`
(same `INV` / `HOST`).

## Give another operator access to existing secrets

1. Add their public key to the `age:` recipients in this repo's `.sops.yaml`.
2. Have someone who can already decrypt run:

   ```bash
   make rekey INV=/path/to/your-inventory/inventory.yaml
   ```
