# Config Vault To Apply Kubeconfig

### Launching HashiCorp Vault
```bash
wget https://releases.hashicorp.com/vault/1.17.2/vault_1.17.2_linux_amd64.zip
unzip vault_1.17.2_linux_amd64.zip
sudo mv vault /usr/local/bin/
vault --version
```

**HINT:** For production, set up Vault in High Availability (HA) mode with a storage provider like `Consul` or another database.

### Running Vault in development mode (for testing)
```
vault server -dev -dev-listen-address=0.0.0.0:8200
```
* This command will start a temporary Vault server and display a Root Token in the output.
* Copy this token, as you will need it for authentication.
* Warning: Development mode is not suitable for production, as data is not encrypted and is deleted after the server shuts down.

### Setting environment variables to access Vault
```bash
export VAULT_TOKEN="your-root-token"
```
```bash
export VAULT_ADDR="http://0.0.0.0:8200"
```

### Activating the Secrets engine:
First, let's check if this engine is activated or not:
```bash
vault secrets list
```
![image](https://github.com/user-attachments/assets/c6329abb-05e1-40a9-bd16-a96f37620c3f)

```bash
vault secrets enable -path=secret kv
```
This command creates a path called secret to store data.

### Save the kubeconfig file to Vault

```bash
cat ~/.kube/config | base64 > kubeconfig_base64.txt
vault kv put secret/kubeconfig data="$(cat kubeconfig_base64.txt)"
```
`secret/kubeconfig` is the storage path in Vault and `data` is the key that holds the kubeconfig value. \

Make sure the data is saved correctly:
```bash
vault kv get secret/kubeconfig
```
You should see output similar to this:
![image](https://github.com/user-attachments/assets/dcab1559-3413-4960-9f24-727e621f89cf)

### Creating a secure token for CI/CD
First we need to create a policy to restrict access. Create a policy file called ci-policy.hcl:
```bash
path "secret/data/kubeconfig" {
  capabilities = ["read"]
}
```
This policy only allows reading of the `secret/kubeconfig` path. \
Apply the policy to the Vault:
```bash
vault policy write ci-policy ci-policy.hcl
```

```bash
vault policy list
```
![image](https://github.com/user-attachments/assets/254fdbe8-bab5-4171-a5c3-1a421717b728)

Create a token with the above Policy:
```bash
vault token create -policy=ci-policy -ttl=8760h
```
The output will be a new token, Copy this token (hvs.XXXXXXXXXXXXXXXXXX).

![image](https://github.com/user-attachments/assets/e070c07e-0cb4-42ad-b274-7f9bd1ae0e5e)

Create variables `VAULT_ADDR` and `VAULT_TOKEN` in GitLab. \

And now we use it in CI/CD job:
```
stages:
  - check

check_cluster:
  stage: check
  image: bitnami/kubectl:latest
  before_script:
    - export VAULT_ADDR=$VAULT_ADDR
    - export VAULT_TOKEN=$VAULT_TOKEN
  script:
    - export KUBECONFIG_BASE64=$(vault kv get -field=data secret/kubeconfig)
    - echo "$KUBECONFIG_BASE64" | base64 -d > kubeconfig
    - export KUBECONFIG=$PWD/kubeconfig
    - kubectl get nodes
```

**Hint:** The image from which the container for this job is built, must have the necessary tools, such as  `kubectl` and `Vault` binaries.

