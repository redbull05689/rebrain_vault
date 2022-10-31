# Create my_database and Configure a User for Access
# In the Database Server, log in to the database:
sudo mysql -u root -p
# Create a new database named, my_database:
create database my_database;
CREATE USER 'vault'@'%' IDENTIFIED BY '!!!Something21!!';
GRANT ALL PRIVILEGES ON my_database.* TO 'vault'@'%' WITH GRANT OPTION;
GRANT CREATE USER ON *.* to 'vault'@'%';
FLUSH PRIVILEGES;
# Create a table and populate it with generic content:
use my_database
create table test_table (msg VARCHAR(100));
insert into test_table (msg) values ('Hello!');
# Verify that the table was created successfully:
select * from my_database.test_table;
# Create a second table:
create table another_table(msg VARCHAR(100));
insert into another_table(msg) value ('Something');
# Grant Vault Access to the Database and Create a User with Access to the test_table
# In the Vault Server, enable a database secrets engine:
vault secrets enable database
# In the Vault Server, grant access to the database:
vault write database/config/my_database plugin_name=mysql-legacy-database-plugin connection_url=`{{username}}:{{password}}@tcp(<DATABASE_SERVER_PRIVATE_IP_ADDRESS>)/` allowed_roles='my_database' username='vault' password='!!!Something21!!'
# Create a user with access to the test_table:
vault write database/roles/my_database \
db_name=my_database \
creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON my_database.test_table TO '{{name}}'@'%';" \
default_ttl="5m" \
max_ttl="10m"
# Create a policy file with read-only access to the database:
vim dbPolicy.hcl
# In the file, paste the following:
path "database/creds/my_database" {
  capabilities = ["read"]
}
# Save the file:
ESC
:wq     
ENTER
# Write the policy:
vault policy write dbPolicy dbPolicy.hcl
# Create a token with the policy:
vault token create -policy=dbPolicy
# Save the token for later use.
# Get the Credentials and Test it Out
# In the Vault Server, retrieve the Domain name:
cat Domain
# In the Client server, install jq:
sudo apt install jq
# Install mariadb-client:
sudo apt install mariadb-client
# Request the credentials and copy the password and username:
curl -H "X-Vault-Token: token" http://<VAULT_SERVER_DOMAIN_NAME>/v1/database/creds/my_database | jq
# From the Client server attempt to authenticate against the database with the newly created credentials:
mysql -u <USERNAME> -h <DATABASE_SERVER_PRIVATE_IP_ADDRESS> -p
# Access the database:
use my_database
# Fetch the data from test_table:
select * from my_database.test_table;
# Attempt to select data from another_table:
select * from my_database.another_table;