# Vault-Postgre

__O que é o Vault e pra que serve?__


Vault é uma ferramenta desenvolvida pela HashiCorp, essa ferramenta tem como objetivo fazer um armazenamento inteligente de “segredos”, podem ser eles, chaves de ssh, dados de acesso a um banco e dados, api tokens e assim por diante. 

[Saiba mais...](https://www.netbr.com.br/hashicorp-brasil/)

## Pré requisitos

[minikube com vault e consul instalados em um mesmo namespace.](https://learn.hashicorp.com/tutorials/vault/kubernetes-minikube)

## Configurando integração do vault com o postgreSQL

- Criando um namespace chamado postgres.

    ```bash
    $ kubectl create namespace postgres
    ```
- Verificando seus pods.
    ```bash
    $ kubectl get pods -n postgres
    ```
- Criando um pod com base no postgre.yml.
    ```bash
    $ kubectl apply -f postgre.yml -n postgres
    ```
- Fazendo login com o usuario root do vault.
    ```bash
    $ kubectl exec -ti vault-0 -n vault -- vault login
    ```

- Dizendo ao vault que a secret será um database.
    ```bash
    $ kubectl exec -ti vault-0 -n vault -- vault secrets enable database
    ```
- Checando pods do vault e postgress.
    ```bash 
    $ kubectl get pods -n vault
    $ kubectl get pods -n postgres
    ```
- Acessando o postgres.
    ```bash
    $ kubectl exec -it -n postgres $(kubectl get pods -n postgres --selector "app=postgres" -o jsonpath="{.items[0].metadata.name}") -c postgres -- bash -c 'PGPASSWORD=password psql -U postgres'
    ```

- Pegar o ip do namespace postgres.
    ```bash
    $ kubectl get services -n postgres
    ```
- Declarando url de conexão com o banco. Na url de conexão foi colocado __postgres.postgres__ isso é importante pois não precisamos colocar o ip do namespace postgres, só precisamos dizer o namespace e o service. Nesse caso tanto o namespace quanto o service contem o mesmo nome.
    ```bash
    $ kubectl exec -ti vault-0 -n vault -- vault write database/config/dev     plugin_name=postgresql-database-plugin     allowed_roles="*"     connection_url="postgresql://{{username}}:{{password}}@postgres.postgres:5432/dev?sslmode=disable" username="postgres"     password="password"
    ```

- Declarando uma regra.
    ```bash
    $ kubectl exec -ti vault-0 -n vault -- vault write database/roles/dev     db_name=dev     creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
        GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";"     revocation_statements="ALTER ROLE \"{{name}}\" NOLOGIN;"    default_ttl="1h"     max_ttl="24h"
    ```
- Delegando ao vault o gerenciamento da senha e do nome de usuário.
    ```bash
    $ kubectl exec -ti vault-0 -n vault -- vault write --force /database/rotate-root/dev
    ```
- Tentando acessar o banco do postgre. Aqui o vault irá bloquear o acesso, pois é necessário solicitar um usuário e senha para ele.
    ```bash
    $ kubectl exec -it -n postgres $(kubectl get pods -n postgres --selector "app=postgres" -o jsonpath="{.items[0].metadata.name}") -c postgres -- bash -c 'PGPASSWORD=password psql -U postgres'
    ```
- Com o comando abaixo o vault irá criar um __username__ e __password__ criptografados. Esse acesso irá atender as regras especificadas. 
    ```bash
    $ kubectl exec -ti vault-0 -n vault -- vault read database/creds/dev
    ```
- Entrando com o password e username dados pelo vault.
    ```bash
    $ kubectl exec -it -n postgres $(kubectl get pods -n postgres --selector "app=postgres" -o jsonpath="{.items[0].metadata.name}") -c postgres -- bash -c 'PGPASSWORD=A1a-3af2yrGLTL3bRoeC psql -U v-root-dev-bhedY39fUUzZHPtmy2Sy-1609169341 dev'
    ```
