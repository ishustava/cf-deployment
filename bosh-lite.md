# Deploying Cloud Foundry against bosh-lite using bosh-bootloader (aka bbl)

## Prerequisites

- You are using `GCP` or `AWS` - `bbl` only supports these
- `bbl` is installed
- You have set the required environment variables for your IaaS environment as documented in `bbl up --help` and also the [README](https://github.com/cloudfoundry/bosh-bootloader/blob/master/README.md) of bosh-bootloader
- You have both `cf-deployment` and `bosh-deployment` repos handy

## 1. Concatenate needed ops files

`bbl` only accepts a single ops file, so you need to concatenate all the necessary ops files into a single file, and then pass that to the `bbl up` command.  The ops files you will need reside in the [bosh-deployment](https://github.com/cloudfoundry/bosh-deployment) repository.

For `AWS`, you will need:
- bosh-lite.yml
- bosh-lite-runc.yml
- jumpbox-user.yml

For `GCP`, you will need the above and also:
-  gcp/bosh-lite-vm-type.yml

## 2. `bbl up` the environment

The command you'll need to run will look something like:

```
bbl \
up \
--name <ENV_NAME> \
--ops-file <CONCATENATED_OPS_FILE>
```

## 3. Ensure required firewall ports open

Until `bbl` offers a `--lite` option, you'll also need to ensure the ports `80,443,2222` are opened on the firewall to the vm created by `bbl`.

## 4. Upload the cloud config

```
bosh \
-e $(bbl director-address) \
update-cloud-config \
cf-deployment/bosh-lite/cloud-config.yml
```

## 5. Deploy CF

```
bosh \
-e $(bbl director-address) \
-d cf \
deploy \
cf-deployment/cf-deployment.yml \
-o cf-deployment/operations/bosh-lite.yml \
--vars-store vars-store.yml \
-v system_domain=<SYSTEM_DOMAIN>
```
