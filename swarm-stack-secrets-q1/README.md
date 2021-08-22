### Create a Swarm Stack with Secrets


- In the ym file we have told to get the secrets from a file and passed the location in environment variable.

```yaml
  postgres:
    image: postgres:12.1
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/psql-pw

```

- And then mentioned that the password will be generated externally.

```yaml
  secrets:
	psql-pw:
	  external: true
```

- These commands will run the compose file written for the swarm.

```shell
	echo "mypass" | docker secret create psql-pw -

	docker stack deploy -c docker-compose.yml drupal
	
	docker stack ps drupal
```