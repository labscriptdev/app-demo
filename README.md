# Laravel + Nuxt + Docker
## Start dev server
```bash
bash bin/dev
```

or use composer command

```bash
docker-compose up
```

## Start prod server
```bash
docker-compose -f docker-compose-prod.yml up
```

or use short command
```bash
yarn build
```

Bash commands
```bash
# Get atualizations
bash bin/app-pull

# Send atualizations
bash bin/app-push

# Start server
bash bin/dev
```

## TODO
- ⬛ Ports in settings .env
- ⬛ Nuxt 3
- ⬛ Laravel
  - ⬛ File garbage collector

<hr />

Nuxt Releases <br />
https://github.com/vuetifyjs/vuetify/releases

Vuetify Releases <br />
https://github.com/nuxt/framework/releases

https://github.com/swooletw/laravel-swoole/wiki <br>

Crontab <br />
https://forums.docker.com/t/cron-does-not-run-in-a-php-docker-container/103897 <br />
https://stackoverflow.com/questions/40180326/cron-isnt-running-when-i-start-my-docker-container <br />
https://laracasts.com/discuss/channels/servers/cron-does-not-run-in-a-php-docker-container <br />
https://serverfault.com/questions/1034381/cron-is-not-running-from-docker-container-failed <br />