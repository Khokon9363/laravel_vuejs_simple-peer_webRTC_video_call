## Video Chat Application Laravel Vuejs Pusher Simple-peer

## Get it up and running.

After you clone this project, do the following:

```bash
# go into the project
cd laravel

# create a .env file
cp .env.example .env

# setup pusher in .env

# install composer dependencies
composer update

# install npm dependencies
npm install

# generate a key for your application
php artisan key:generate

# mrun the migration files to generate the schema
php artisan migrate

# run webpack and watch for changes
npm run watch
```

Good Luck :)

https://medium.com/@otacorporation0520/create-a-video-chat-application-with-laravel-7-vue-js-based-on-webrtc-58c88a503c17