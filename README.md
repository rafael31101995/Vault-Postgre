# Vault-Postgre

- Criando um namespace chamdo postgres

        kubectl create namespace postgres

- Verificando seus pods

        kubectl get pods -n postgres

- Criando um pod com base no postgre.yml

        kubectl apply -f postgre.yml -n postgres

- Fazendo login com o usuario root do vault.

        kubectl exec -ti vault-0 -n vault -- vault login

- Dando um enable no database.

        kubectl exec -ti vault-0 -n vault -- vault secrets enable database

- Pegando checando pods do vault e postgress
        
        kubectl get pods -n vault
        kubectl get pods -n postgres

- Acessando o postgres

        kubectl exec -it -n postgres $(kubectl get pods -n postgres --selector "app=postgres" -o jsonpath="{.items[0].metadata.name}") -c postgres -- bash -c 'PGPASSWORD=password psql -U postgres'

- Secrets enable

        kubectl exec -ti vault-0 -n vault -- vault secrets enable database

- Pegar o up do namespace postgres.

        kubectl get services -n postgres

- Declarando url de conexão com o banco.

        kubectl exec -ti vault-0 -n vault -- vault write database/config/dev     plugin_name=postgresql-database-plugin     allowed_roles="*"     connection_url="postgresql://{{username}}:{{password}}@10.109.152.53:5432/dev?sslmode=disable" username="postgres"     password="password"


- Declarando uma regra.

        kubectl exec -ti vault-0 -n vault -- vault write database/roles/dev     db_name=dev     creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
        GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";"     revocation_statements="ALTER ROLE \"{{name}}\" NOLOGIN;"    default_ttl="1h"     max_ttl="24h"

- Tentando acessar o banco do postgre.

        kubectl exec -it -n postgres $(kubectl get pods -n postgres --selector "app=postgres" -o jsonpath="{.items[0].metadata.name}") -c postgres -- bash -c 'PGPASSWORD=password psql -U postgres'

- Bloqueia o banco de dev

        kubectl exec -ti vault-0 -n vault -- vault write --force /database/rotate-root/dev

- Gerando o password e username

        kubectl exec -ti vault-0 -n vault -- vault read database/creds/dev

- Entrando com o password e username

        kubectl exec -it -n postgres $(kubectl get pods -n postgres --selector "app=postgres" -o jsonpath="{.items[0].metadata.name}") -c postgres -- bash -c 'PGPASSWORD=A1a-3af2yrGLTL3bRoeC psql -U v-root-dev-bhedY39fUUzZHPtmy2Sy-1609169341 dev'

