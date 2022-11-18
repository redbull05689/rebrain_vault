sudo systemctl restart vault

vault -autocomplete-install

export VAULT_SKIP_VERIFY="true"

Проведите инициализацию кластера с 3 ключами, любые два из которых распечатывают Vault. Сохраните root token в файл /home/user/root_token.

Ответ: vault operator init -key-shares=3 -key-threshold=2

echo "hvs.umnHpFG85W6BtXRoG8gIETVB" > /home/user/root_token

export VAULT_TOKEN="hvs.umnHpFG85W6BtXRoG8gIETVB"

vault operator unseal
vault operator unseal

Включите SecretEngine database c mount point=database


Ответ: vault secrets enable -path=database database

Включить AuthMethod userpass с mount point db-users

Ответ: vault auth enable -path=db-users userpass

Создайте пользователя mongo-dev с политикой mongo-dev и паролем password-mongo-dev.

Ответ: vault write auth/db-users/users/mongo-dev password=password-mongo-dev policies=mongo-dev

Создайте пользователя postgres-dev с политикой postgres-dev и паролем password-postgres-dev.

Ответ: vault write auth/db-users/users/postgres-dev password=password-postgres-dev policies=postgres-dev

Сконфигурируйте подключение к MongoDB в mount-point database с названием mongo-dev и следующими параметрами:


connection string -  mongodb://{{username}}:{{password}}@[ип_баз_данных]:27017/admin?tls=false

username - mongouser
password - mongopass
allowed roles - mongo-dev-role

Ответ:  vault write database/config/mongo-dev plugin_name=mongodb-database-plugin allowed_roles="mongo-dev-role" connection_url="mongodb://{{username}}:{{password}}@158.160.10.253:27017/admin?tls=false" username="mongouser" password="mongopass"

Сконфигурируйте подключение к Postgres в mount-point database с названием postgres-dev и следующими параметрами:


connection string -  postgresql://{{username}}:{{password}}@[ип_баз_данных]:5432/?sslmode=disable

username - vault
password - vaultpass
allowed roles - postgres-dev-role

Ответ: vault write database/config/postgres-dev plugin_name=postgresql-database-plugin allowed_roles="postgres-dev-role" connection_url="postgresql://{{username}}:{{password}}@158.160.10.253:5432/postgres?sslmode=disable" username="vault" password="vaultpass"

Сконфигурируйте роль postgres-dev-role для инстанса postgres-dev. Используйте следующие параметры:


creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; 
                       GRANT USAGE ON SCHEMA stage_schema TO \"{{name}}\";
                       GRANT SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA stage_schema TO \"{{name}}\";" \
default_ttl=30m
max_ttl=1h


Ответ:
vault write database/roles/postgres-dev-role 
db_name=postgres-dev 
creation_statements="CREATE ROLE "{{name}}" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';
GRANT USAGE ON SCHEMA stage_schema TO "{{name}}";
GRANT SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA stage_schema TO "{{name}}";" 
default_ttl="30m" 
max_ttl="1h"

Сконфигурируйте роль mongo-dev-role для инстанса mongo-dev



creation-statement = '{ "db": "admin", "roles": [{"role": "readWrite", "db": "dev-app"}] }'
default_ttl=30m
max_ttl=1h

Ответ:
vault write database/roles/mongo-dev-role 
db_name=mongo-dev 
creation_statements='{ "db": "admin", "roles": [{"role": "readWrite", "db": "dev-app"}] }' 
default_ttl="1h" 
max_ttl="24h"

Напишите политику с именем mongo-dev, которая будет позволять читать креды для mongo с помощью роли mongo-dev-role.

Ответ:

vault policy write mongo-dev - <<EOF
path "database/creds/mongo-dev-role" {
  capabilities = ["read"]
}
EOF



Напишите политику с именем postgres-dev, которая будет позволять читать креды для postgres с помощью роли postgres-dev-role.

Ответ:

vault policy write postgres-dev - <<EOF
path "database/creds/postgres-dev-role" {
  capabilities = ["read"]
}