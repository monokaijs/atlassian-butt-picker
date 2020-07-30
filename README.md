## Jira in docker
### Something to note
1. With local deployment, we need to add `127.0.0.1 jira.internal` into hosts file:
   ```
   echo '127.0.0.1 jira.internal' | sudo tee -a /etc/hosts
   ```
2. Everything will be automatically configured, until you reach the license prompt screen:

   ![licsense-prompt](images/license-prompt.png)

3. Copy the license server id, for example `BPOD-YXDV-LMVB-DIC0`, then run command below, with `-p` for product, `-m` for email, `-n` for license name, `-s` for license server id:
   ```
   java -jar atlassian-agent.jar \
	-p jira \
	-m your@email.com \
	-n hino \
	-o akatsuki \
	-s BPOD-YXDV-LMVB-DIC0
   ```
3. Copy the generated license code, get back to your browser and submit it, then BOOM!!! you're done :whale:

   ![finished](images/finished.png)

### Frequently Q&A
- At **step 2**, if **java** is not installed at your machine, use below docker trick:
   ```
   docker run --rm -v "${PWD}/atlassian-agent.jar:/atlassian-agent.jar" \
	openjdk:8-jre-alpine \
	java -jar atlassian-agent.jar \
	-p jira -m your@email.com -n hino -o akatsuki \
	-s BPOD-YXDV-LMVB-DIC0
   ```
- Be aware that changing postgres default database name (`postgres` -> `jira`), it'll create a new database `jira` and doesn't accept incoming connection during that time. Therefore you may encouter a problem when Jira perform DB pre-database startup check and can not start Jira server. You may need to:
   + Simply restart jira service with: `docker-compose restart jira`
   + Change postgres config `POSTGRES_DB` and `POSTGRES_USER` back to default as `postgres`
   + Implement docker delayed startup config, pls refer: https://docs.docker.com/compose/startup-order/

   For the sake of simplicity, and meaningful DB config, i wont re-config it now, but will track and update if needed.
- Last but not least, the Jira crack mechanism is thank to Zhile, you can refer the source and donate to him via: https://gitee.com/pengzhile/atlassian-agent
