#step-by-step

#Login to Vault pod

kubectl exec -it vault-0 -- /bin/sh

#Create key-value store in Vault to keep my secrets

vault secrets enable -path=mysqlwordpress kv-v2

#Write secret

vault kv put mysqlwordpress/database/config MYSQL_USER="wordpress" MYSQL_PASSWORD="wordpress" MYSQL_ROOT_PASSWORD="somewordpress" MYSQL_DATABASE="wordpress"

#Check, that secret

vault kv get mysqlwordpress/database/config

#Create Vault policy, that will allow to read secrets in store

vault policy write wordpressdb - <<EOF
path "mysqlwordpress/data/database/config" {
  capabilities = ["read"]
}
EOF

#Bound k8s service account and Vault policy

vault write auth/kubernetes/role/wordpressdb \
    bound_service_account_names=wordpressdb \
    bound_service_account_namespaces=default \
    policies=wordpressdb \
    ttl=24h

#Create k8s service account

kubectl create sa wordpressdb

#Get container logs

kubectl logs pod/db-7b6d744689-msgfc --container db

#Login to DB container, login as root, list all users

kubectl exec -it pod/db-7b6d744689-msgfc --container db -- /bin/bash
mysql -u root -p
SELECT User FROM mysql.user;
