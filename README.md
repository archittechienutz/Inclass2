lcbo database with signoz demo

```bash
ansible-playbook up.yml
```

This does docker compose up on an "empty" signoz in a codespace. Use this as a starting point for instrumenting your app. It also creates a postgresql/pgadmin container with an old version of the LCBO database in it.

```bash
ansible-playbook down.yml
```

This does docker compose down on the `docker-compose.yml` (the same docker-compose file from up.yml)
