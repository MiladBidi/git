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

### 2. Save the kubeconfig file to Vault

```bash
cat ~/.kube/config | base64 > kubeconfig_base64.txt
vault kv put secret/kubeconfig data="$(cat kubeconfig_base64.txt)"
```



