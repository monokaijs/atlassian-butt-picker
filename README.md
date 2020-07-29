## Jira in docker
### Something to note
- With local deployment, we need to add `127.0.0.1 jira.internal` into hosts file:
```
echo '127.0.0.1 jira.internal' | sudo tee -a /etc/hosts
```
