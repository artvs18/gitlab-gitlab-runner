# gitlab-gitlab-runner
Custom docker-compose configuration of gitlab and gitlab-runner for less memory consumption

```yaml
version: '3.7'
services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'localhost'
    container_name: gitlab-ce
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost'
        sidekiq['max_concurrency'] = 10
        postgresql['shared_buffers'] = "256MB"
        puma['worker_processes'] = 0
        gitlab_rails['env'] = {
          'MALLOC_CONF' => 'dirty_decay_ms:1000,muzzy_decay_ms:1000'
        }
        gitaly['env'] = {
          'MALLOC_CONF' => 'dirty_decay_ms:1000,muzzy_decay_ms:1000'
        }
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    networks:
      - gitlab
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always 
    depends_on:
      - web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - '$GITLAB_HOME/gitlab-runner:/etc/gitlab-runner'
    networks:
      - gitlab

networks:
  gitlab:
    name: gitlab-network
```

Be sure that you have installed docker engine and docker compose on your machine
```bash
docker -v
# Output should look like this - Docker version 23.0.1, build a5ee5b1
docker compose version
# Output should look like this - Docker Compose version v2.16.0
