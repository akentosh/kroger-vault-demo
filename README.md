# Kroger Vault Demo

## Demo

### Setup
    vault server -dev-root-token-id=myroottoken -dev
    export VAULT_ADDR='http://127.0.0.1:8200'
    vault login myroottoken

### Static Secrets
    vault secrets list -detailed
    vault secrets enable -path kroger kv-v2
    vault kv put kroger/demo/creds username="sysdba" password="staticpassword"
    vault kv get kroger/demo/creds
    vault kv put -output-curl-string kroger/demo/app-creds username="app1" password="12345678"
    vault kv get -output-curl-string kroger/demo/app-creds
    vault kv get -field=username kroger/demo/creds
    vault kv put kroger/demo/creds password="another-static-password"
    vault kv get kroger/demo/creds
    vault kv patch kroger/demo/creds username="akentosh"
    vault kv get kroger/demo/creds
    vault kv put kroger/company @data.json
    vault kv get kroger/company
    vault kv delete kroger/company
    vault kv get kroger/company

### Dynamic Secrets
    vault secrets enable -path my-app database
    vault write my-app/config/myapp-database plugin_name=postgresql-database-plugin allowed_roles="readonly" connection_url="postgresql://postgres@localhost:5432/?sslmode=disable" username="postgres"
    vault write my-app/roles/readonly db_name=myapp-database creation_statements=@readonly.sql default_ttl=1h max_ttl=24h
    vault read my-app/creds/readonly
    vault read -output-curl-string my-app/creds/readonly
    vault lease revoke -prefix my-app/creds/readonly

### AWS
    vault secrets enable aws
    vault write aws/config/root access_key=$AWS_ACCESS_KEY_ID secret_key=$AWS_SECRET_ACCESS_KEY region=us-east-1
    
    vault write aws/roles/my-role \
        credential_type=iam_user \
        default_ttl=1h \
        max_ttl=2h \
        policy_document=-<<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "ec2:*",
          "Resource": "*"
        }
      ]
    }
    EOF
    
    vault read aws/creds/my-role

### Transit

    vault secrets enable transit
    vault write -f transit/keys/cards
    vault write transit/encrypt/cards plaintext=$(base64 <<< "1234-5678-1234-5678")
    vault write -output-curl-string transit/decrypt/cards ciphertext=""
    | base64 --decode <<< $(jq -r '.data.plaintext' )
