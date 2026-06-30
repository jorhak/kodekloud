```
The Nautilus DevOps team is focusing on improving their data security by using Azure Key Vault. Your task is to create a Key Vault with an RSA key and manage the encryption and decryption of a pre-existing sensitive file using this key.

Specific Requirements:

Create a Key Vault:

Name the Key Vault xfusion-4812.
Create the vault in the East US region.
Use the Standard pricing tier.
Set Soft Delete retention to 7 days.
Use the Vault access policy permission model.
Configure an access policy that allows Get, List, Encrypt, and Decrypt permissions for the lab identity.
Create an RSA Key:

Create a key named xfusion-key within the Key Vault.
Key type: RSA.
RSA key size: 4096.
Leave all other settings as default.
Encrypt the Sensitive Data:

Use the key to encrypt the provided SensitiveData.txt file (located in /root/) on the azure-client host.
Use the RSA-OAEP algorithm.
Base64 encode the plaintext before encryption.
Save the encrypted version as EncryptedData.bin in the /root/ directory.
Note: If you encounter a permissions error, retrieve the Service Principal ID using:
az account show --query user.name -o tsv
and grant the required Key Vault permissions.

Verify Decryption:
Decrypt EncryptedData.bin.
Base64 decode the decrypted output.
Save the result as DecryptedData.txt in /root/.
Ensure the decrypted file matches the original SensitiveData.txt.
Ensure that the Key Vault and key are correctly configured. The validation script will test your configuration by decrypting the EncryptedData.bin file using the key you created.


Use the Azure Portal URL and login credentials below:

Portal URL	https://portal.azure.com
Username	kk_lab_user_main@azureprod.onmicrosoft.com
Password	contra

Notes:

Create the resources only in the East US region.
Network restrictions or private endpoints are NOT required for this task.
```
# Variables de entorno
```
KV_NAME=xfusion-7456
LOCATION=eastus
SKU=standard
RETENTION=7
K_NAME=xfusion-key
ORIGEN_DATA=/root/SensitiveData.txt
ENCRYPT_DATA=/root/EncryptedData.bin
DENCRYPT_DATA=/root/DecryptedData.txt
```
# Obtener Resource Group
```
RG_NAME=$(az group list --query [0].name --output tsv)
```
# Obtener User ID
```
USER_ID=$(az account show --query "user.name" -o tsv)
```
# 1 Crear y configuar Key Vault
#### Crear vault
```
az keyvault create \
   --resource-group $RG_NAME \
   --name $KV_NAME \
   --location $LOCATION \
   --retention-days $RETENTION \
   --sku $SKU \
   --enable-rbac-authorization false
```
#### Configurar vault
```
az keyvault set-policy \
   -n $KV_NAME \
   --object-id $USER_ID \
   --key-permissions get list encrypt decrypt
```
# 2 Crear llave para el vault
```
az keyvault key create \
   --name $K_NAME \
   --kty RSA \
   --size 4096 \
   --vault-name $KV_NAME
```
# 3 Encriptar informacion sensible
```
TEXTO_BASE64=$(cat "$ORIGEN_DATA" | base64 | tr -d '\n\r')
```

```
CHIPERTEXT=$(az keyvault key encrypt \
    --algorithm RSA-OAEP\
    --value "$TEXTO_BASE64"\
    --data-type base64 \
    --name $K_NAME\
    --vault-name $KV_NAME\
    --query "result" \
    -o tsv)
```

```
echo $CHIPERTEXT> $ENCRYPT_DATA
```

```
wc -c $ENCRYPT_DATA
```
# Verificar
```
DESENCRIPTADO_BASE64=$(az keyvault key decrypt \
    --vault-name $KV_NAME\
    --name $K_NAME\
    --algorithm RSA-OAEP\
    --value $(cat "$ENCRYPT_DATA")\
    --data-type base64 \
    --query "result" \
    -o tsv)
```

```
echo "$DESENCRIPTADO_BASE64" | base64 --decode > $DENCRYPT_DATA
```

```
diff $ORIGEN_DATA $DENCRYPT_DATA && echo "¡Validación Exitosa! Los
archivos coinciden perfectamente."
```
