## Jira in docker

### 0. Prerequisites
- `docker` 19.03+, `docker-compose` 1.27+

### I. Initial Config
You have 3 choices with 3 different kinds of deployment.

1. With local deployment (no need SSL), we need to add `127.0.0.1 jira.internal` into hosts file:
   ```bash
   echo '127.0.0.1 jira.internal' | sudo tee -a /etc/hosts
   ```

2. With other deployment that requires SSL, simply change docker-compose files by referring `.env`:
   ```properties
   COMPOSE_FILE=docker-compose.yml:docker-compose.ssl.yml
   ```
   Add your cert stuff into `certs/`, with full chain as `fullchain1.pem`, trusted chain as `chain1.pem` and private key as `privkey1.pem`.

3. For Let's Encrypt SSL certificate auto-renewal, update your `.env` like below:
   ```properties
   COMPOSE_FILE=docker-compose.yml:docker-compose.acme.yml
   DOMAIN_NAME=jira.your-domain.com
   ACCOUNT_EMAIL=you@your-domain.com
   ```
   Your SSL cert will be renewed automatically every **60 days**, and configured with `OCSP Must Staple` and `Certificate Transparency`.

   Moreover, your proxy will be enabled `TLS 1.3`, `OCSP Stapling`, `Forward Secrecy` and `HSTS`.

### II. Step by step
1. Build them up:
   In case of LE cert auto-renewal, you need to start `nginx-proxy` and `nginx-proxy-acme` first:
   ```bash
   docker-compose up -d nginx-proxy-acme && docker-compose logs -f nginx-proxy nginx-proxy-acme
   ```
   Then start the other containers with:
   ```bash
   docker-compose up -d && docker-compose logs -f
   ```
   If you don't need auto-renewal, only need to run the latter command.

2. Everything will be automatically configured, until you reach the license prompt screen:

   ![licsense-prompt](images/license-prompt.png)

3. Copy the license server id, for example `BPOD-YXDV-LMVB-DIC0`, then run command below, with `-p` for product, `-m` for email, `-n` for license name, `-s` for license server id:
   ```bash
   java -jar atlassian-agent.jar \
	-p jira \
	-m your@email.com \
	-n hino \
	-o akatsuki \
	-s BPOD-YXDV-LMVB-DIC0
   ```

4. Copy the generated license code, get back to your browser and submit it, then BOOM!!! you're done :whale:

   ![finished](images/finished.png)

5. Check what they are doing behind your back:
   ```bash
   docker-compose logs -f --tail 69
   ```

6. Tired of Jira, burn them down, but keep the volumes in case you change your mind:
   ```bash
   docker-compose down
   ```

### IV. Frequently Q&A
1. At **step 3**, if **java** is not installed at your machine, use below docker trick:
   ```bash
   docker run --rm -v "${PWD}/atlassian-agent.jar:/atlassian-agent.jar" \
	openjdk:8-jre-alpine \
	java -jar atlassian-agent.jar \
	-p jira -m your@email.com -n hino -o akatsuki \
	-s BPOD-YXDV-LMVB-DIC0
   ```

2. Be aware that changing postgres default database name (`postgres` -> `jira`), it'll create a new database `jira` and doesn't accept incoming connection during that time. Therefore you may encouter a problem when Jira perform DB pre-database startup check and can not start Jira server. You may need to:
   + Simply restart jira service with: `docker-compose restart jira`
   + Change postgres config `POSTGRES_DB` and `POSTGRES_USER` back to default as `postgres`
   + Implement docker delayed startup config, pls refer: https://docs.docker.com/compose/startup-order/

   For the sake of simplicity, and meaningful DB config, i wont re-config it now, but will track and update if needed.

3. `TODO` - SSL cert autorenew config + fully-automated cert generating :penguin:

4. Last but not least, the Jira crack mechanism is thank to Zhile, you can refer the source and donate to him via: https://gitee.com/pengzhile/atlassian-agent
