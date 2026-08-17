```
The Nautilus DevOps team is focusing on improving their data security by using AWS KMS. Your task is to create a KMS key and manage the encryption and decryption of a pre-existing sensitive file using the KMS key.

Specific Requirements:

Create a symmetric KMS key named xfusion-KMS-Key to manage encryption and decryption.
Encrypt the provided SensitiveData.txt file (located in /root/), base64 decode the ciphertext, and save the encrypted version as EncryptedData.bin in the /root/ directory.
Try to decrypt the same and verify that the decrypted data matches the original file.
Make sure that the KMS key is correctly configured. The validation script will test your configuration by decrypting the EncryptedData.bin file using the KMS key you created.


Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://700493584428.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Mon Aug 17 17:48:32 UTC 2026
End Time	Mon Aug 17 18:48:32 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```
# Variables de entorno
```
PREFIX=devops
KMS_NAME=$PREFIX-KMS-Key
SRC=/root/SensitiveData.txt
DEST=/root/EncryptedData.bin
REGION=us-east-1
```
# 1 Crear llave
```
KMS_ID=$(aws kms create-key \
   --key-spec SYMMETRIC_DEFAULT \
   --key-usage ENCRYPT_DECRYPT \
   --tags TagKey=Name,TagValue=$KMS_NAME \
   --region $REGION \
   --query "KeyMetadata.KeyId" \
   --output text)
```
## 1.1 Asignar alias
```
aws kms create-alias \
    --alias-name "alias/${KMS_NAME}" \
    --target-key-id $KMS_ID
```
# 2 Encriptar
```
aws kms encrypt \
    --key-id "alias/${KMS_NAME}" \
    --plaintext fileb://$SRC \
    --output text \
    --query CiphertextBlob | base64 \
    --decode > $DEST
```
# 3 Desencriptar
```
aws kms decrypt \
    --ciphertext-blob fileb://$DEST \
    --key-id "alias/${KMS_NAME}" \
    --output text \
    --query Plaintext | base64 \
    --decode > Desencriptado.txt
```
# 4 Verificar
```
diff -s $SRC /root/Desencriptado.txt
```
