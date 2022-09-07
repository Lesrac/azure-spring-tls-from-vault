From: https://docs.microsoft.com/en-us/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-key-vault-certificates

See why not newer versions in POM are used: https://github.com/MicrosoftDocs/azure-dev-docs/issues/853

```
az group create \
    --name ssltls \
    --location westeurope
```
```
az vm create \
    --name vmmaschine \
    -g ssltls \
    --debug \
    --generate-ssh-keys \
    --assign-identity \
    --role contributor \
    --scope /subscriptions/<SubscriptionId>/resourcegroups/ssltls \
    --image UbuntuLTS \
    --admin-username azureuser
```

Output, save "publicIpAddress" and "systemAssignedIdentity":
```
{
  "fqdns": "",
  "id": "/subscriptions/<SubscriptionId>/resourceGroups/ssltls/providers/Microsoft.Compute/virtualMachines/vmmaschine",
  "identity": {
    "role": "contributor",
    "scope": "/subscriptions/<SubscriptionId>/resourcegroups/ssltls",
    "systemAssignedIdentity": "<systemAssignedIdentity>",
    "userAssignedIdentities": {}
  },
  "location": "westeurope",
  "macAddress": "<Mac>",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "20.105.178.22",
  "resourceGroup": "ssltls",
  "zones": ""
}
```
```
ssh azureuser@20.105.178.22

wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt install apt-transport-https
sudo apt update
sudo apt install msopenjdk-11
```
```
az keyvault create \
    --name epdkv2 \
    -g ssltls \
    --location westeurope
```
```
az keyvault set-policy \
    --name epdkv2 \
    --object-id 824a356f-e91e-4330-a8d0-7ff1d41a83e0 \
    --secret-permissions get list \
    --certificate-permissions get list import
```
```
az keyvault certificate create \
    --vault-name epdkv2 \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```

Verify that the network security group created within "your resource group name" allows inbound traffic on ports 22 and 8443 from your IP address.

```
mvn clean package

cd target
sftp azureuser@20.105.178.22
put *.jar

exit

set -o noglob
ssh azureuser@20.105.178.22 "java -jar *.jar"
```

```
curl --insecure https://20.105.178.22:8443/ssl-test
curl --insecure https://20.105.178.22:8443/ssl-test-outbound
```
