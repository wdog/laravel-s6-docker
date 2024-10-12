#!/bin/bash

set_permissions () {
    sudo chown $USER: * -R
    chmod 777 bootstrap/cache/
    chmod 777 storage -R
}


CUID=$(id -u)
sudo rm -rf BASE
mkdir BASE
cd BASE
echo $PWD
git clone git@github.com:wdog/laravel-s6-docker .

src_path=$PWD/src
composer="docker run -u $CUID --rm --interactive --tty --volume .:/app composer"
$composer create-project --prefer-dist laravel/laravel src

composer="docker run -u $CUID --rm --interactive --tty --volume $src_path:/app composer"
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
$composer require laravel/jetstream


cp $ENV_FILE ..

docker-compose up db -d
sleep 10
docker-compose up app -d
docker-compose up mailpit -d

docker-compose exec app npm install
set_permissions

docker-compose up -d

docker-compose exec app php artisan jetstream:install livewire --dark -n
set_permissions

sudo sed -i "14i server: { host: '0.0.0.0', hmr: { host: 'localhost' } }," vite.config.js

docker-compose exec app php artisan optimize:clear
$composer dump -o
