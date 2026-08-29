# Inventory

Boilerplate inventory for [ansible-autobott](https://github.com/ansible-autobott/autobott).

It configures a single example host, `ansible-autobott-example.lan`, and is meant
as a starting point you copy and make your own.

## Usage

To set up your own inventory we recommend to clone this repository and push it to your own git.

1. Clone this repository
```
git clone git@github.com:ansible-autobott/inventory.git
cd inventory
```
2. remove the original remote
```
git remote remove origin
```
3. add the new remote, replace the URL with your new repository's address.

```
git remote add origin https://github.com/your-username/your-new-repo.git
```

4. check your remote
```
git remote -v
```
5. push to your new repository
```
git branch -M main
git push -u origin main
```

note: if you don't intend to use vagrant VMs, you can delete the `vagrant` folder

## Secrets (SOPS + age)

Secrets are encrypted with [SOPS](https://github.com/getsops/sops) using
[age](https://github.com/FiloSottile/age) keys. All the secret-management
commands live in the **autobott (playbook) repo's Makefile** and take this
inventory via `INV=`; this repo holds only data. Install the tools once with
`make prepare` (or `make check-tools`) from the autobott repo.

Secrets sealed with SOPS live in `*.sops.yaml` files (e.g.
`host_vars/<host>/secrets.sops.yaml`). When encrypted, only the values are
ciphertext — keys and structure stay in plaintext, so diffs stay readable and
the files are safe to commit.

**This boilerplate ships secrets as clear text** in
`host_vars/ansible-autobott-example.lan/secrets.yaml` so the example runs
out-of-box. (Ansible loads any `*.yaml` in the dir; only the `.sops.yaml` suffix
triggers SOPS. A plaintext `*.sops.yaml` would make the sops vars plugin refuse
to load.) When you make this inventory your own, encrypt it:

```bash
# from the autobott repo, pointing INV at this inventory
make age-key INV=/path/to/inventory/inventory.yaml   # writes sops_key here, prints your pubkey
# put that public key in this repo's .sops.yaml (replace the sample recipient), then:
git mv host_vars/ansible-autobott-example.lan/secrets.yaml \
       host_vars/ansible-autobott-example.lan/secrets.sops.yaml
make seal-secrets INV=/path/to/inventory/inventory.yaml HOST=ansible-autobott-example.lan
```

Day-to-day:

```bash
make edit-secrets INV=/path/to/inventory/inventory.yaml HOST=ansible-autobott-example.lan  # edit in $EDITOR
make view-secrets INV=/path/to/inventory/inventory.yaml HOST=ansible-autobott-example.lan  # decrypt & print
make rekey        INV=/path/to/inventory/inventory.yaml                                     # re-encrypt after editing .sops.yaml recipients
```

The age **private key** (`sops_key`) is gitignored and never committed — every
operator generates their own. See [`sops_key.README.md`](sops_key.README.md).

## Vagrant

This project includes an example Vagrant VM that can be used to test the inventory.

**Important notes about this vagrant VM**:
* This vagrant image builds on top of the base image from Autobott, check Autobott documentation for details
* This Vagrant VM will use your host network and **expects** that your network has an DHCP server.
* the DHCP server should also assign the domain ansible-autobott-example.lan AND all subdomains (otherwise you need to add the relevant entry to /etc/hosts)

start the vm with
```
cd vagrant
vagrant up
```

You can then access the VM with `ssh vagrant@ansible-autobott-example` use password `vagrant`

to enroll into autobott (run from the autobott playbook repo)

```
# cd autobott
make enroll INV=../inventory/inventory.yaml HOST=ansible-autobott-example.lan ANSIBLE_USER=vagrant ANSIBLE_PASS=vagrant
```

after enrollment is done you can verify the ans service account by accessing with:
```
ssh ans@ansible-autobott-example.lan -i vagrant/autobott-key
```

To run ansible now, add the provided **INSECURE** key to your ssh agent
```
ssh-add vagrant/autobott-key
```
and run ansible
```
make run INV=../inventory/inventory.yaml TAG=homepage
```
