## Jira in docker
### Something to note
- We need to add `127.0.0.1 jira.internal` into hosts file:
```
echo '127.0.0.1 jira.internal' | sudo tee -a /etc/hosts
```
