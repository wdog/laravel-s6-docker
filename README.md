# env


add to .env

```
VITE_ENABLED=true
XDEBUG_ENABLED=false
```

## first run

```
docker-compose up
docker-compose exec app npm install
docker-compose exec app php artisan migrate:fresh
```



## production

ERR. vite to 5137

- remove debug env
- add production env
- remove ./public/hot
- run npm run build
