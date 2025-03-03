```bash
#!/bin/bash

set_permissions () {
    sudo chown $USER: * -R
    chmod 777 bootstrap/cache/
    chmod 777 storage -R
}


CUID=$(id -u)
BASE=$1
sudo rm -rf $BASE
mkdir $BASE
cd $BASE
echo $PWD
git clone git@github.com:wdog/laravel-s6-docker .

src_path=$PWD/src
composer="docker run -u $CUID --rm --interactive --tty --volume .:/app composer"
$composer create-project --prefer-dist laravel/laravel src

composer="docker run -u $CUID --rm --interactive --tty --volume $src_path:/app composer"
$composer clear-cache --gc

cd src
echo $PWD
rm -f src/database/database.sqlite

ENV_FILE=".env"
sudo chmod 666 $ENV_FILE
# Sostituisce il valore di DB_CONNECTION con 'mysql'
sudo sed -i "s/DB_CONNECTION=.*/DB_CONNECTION=mysql/" .env

# Decommenta le righe per DB_HOST, DB_PORT, DB_DATABASE, DB_USERNAME, e DB_PASSWORD
sudo sed -i "s@APP_URL=.*@APP_URL=https://localhost@" $ENV_FILE
sudo sed -i "s/APP_LOCALE=en/APP_LOCALE=it/" $ENV_FILE

sudo sed -i "s/# DB_HOST=.*/DB_HOST=db/" $ENV_FILE
sudo sed -i "s/# DB_PORT=.*/DB_PORT=3306/" $ENV_FILE
sudo sed -i "s/# DB_DATABASE=.*/DB_DATABASE=laravel/" $ENV_FILE
sudo sed -i "s/# DB_USERNAME=.*/DB_USERNAME=laravel/" $ENV_FILE
sudo sed -i "s/# DB_PASSWORD=.*/DB_PASSWORD=laravel/" $ENV_FILE


sudo echo "VITE_ENABLED=true" >> $ENV_FILE
sudo echo "XDEBUG_ENABLED=false" >> $ENV_FILE

$composer require filament/filament --ignore-platform-req=ext-intl


cp $ENV_FILE ..

docker-compose up db -d
sleep 10
docker-compose up app -d
docker-compose up mailpit -d

docker-compose exec app npm install
set_permissions

docker-compose up -d


set_permissions

sudo sed -i "11i server: { host: '0.0.0.0', hmr: { host: 'localhost' } }," vite.config.js

docker-compose exec -u $CUID app php artisan migrate
docker-compose exec -u $CUID app php artisan filament:install --panels -n
docker-compose exec -u $CUID app php artisan optimize:clear
docker-compose exec -u $CUID app php artisan vendor:publish --tag="livewire:config"
set_permissions

# from:
# 'layout' => 'components.layouts.app',
# to:
# 'layout' => 'layouts.app',

$composer dump -o

docker-compose exec app php artisan tin --execute "User::factory()->create(['email'=>'admin@example.com','password'=> 'password'])"
```

## VITE RELOAD PAGE

```javascript
import { defineConfig } from "vite";
import laravel, { refreshPaths } from "laravel-vite-plugin";

export default defineConfig({
  plugins: [
    laravel({
      input: ["resources/css/app.css", "resources/js/app.js"],
      refresh: [
        ...refreshPaths,
        "app/Http/Livewire/**/*", // Custom Livewire components
        "app/Filament/**/*", // Filament Resources
        "app/Providers/Filament/**",
      ],
    }),
  ],
  server: {
    hmr: {
      host: "localhost",
    },
    host: "0.0.0.0",
  },
});
```

```php
$panel
    ->sidebarWidth('16rem')
    ->maxContentWidth(MaxWidth::Full)
    # HOT RELOAD
    ->renderHook('panels::body.end', fn (): string =>
        Blade::render("@vite('resources/js/app.js')")
        )
```
