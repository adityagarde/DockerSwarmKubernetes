### Using Secrets with Swarm Stack

```shell
docker stack deploy -c docker-compose.yml mydb

docker stack ps mydb
docker stack services mydb
```
This is alternative to mapping secrets using CLI -

```shell
docker service create --name psql --secret psql_user --secret psql_pass 
    -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass 
    -e POSTGRES_USER_FILE=/run/secrets/psql_user postgres
```