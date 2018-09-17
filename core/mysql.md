# Mysql instead of Postgresql

If you want to use mysql instead of Postgres here's are a list of files you have to edit.

Note: this is based on [api-platform 2.3.2](https://github.com/api-platform/api-platform/releases/tag/v2.3.2).

- /api/Dockerfile
```
        docker-php-ext-configure zip --with-libzip; \
        docker-php-ext-install -j$(nproc) \
                intl \
-               pdo_pgsql \
+               pdo_mysql \
                zip \
        ; \
        pecl install \
```
- /api/config/packages/doctrine.yaml
```
 doctrine:
     dbal:
         # configure these for your database server
-        driver: 'pdo_pgsql'
-        server_version: '9.6'
+        driver: 'pdo_mysql'
+        server_version: '5.7'
```
- /api/config/packages/doctrine.yaml
```
if [ "$1" = 'php-fpm' ] || [ "$1" = 'bin/console' ]; then

        if [ "$APP_ENV" != 'prod' ]; then
                composer install --prefer-dist --no-progress --no-suggest --no-interaction
-               >&2 echo "Waiting for Postgres to be ready..."
-               until pg_isready --timeout=0 --dbname="${DATABASE_URL}"; do
-                       sleep 1
-               done
                bin/console doctrine:schema:update --force --no-interaction
        fi
 fi
```

- /docker-compose.yml
```
   db:
-    image: postgres:9.6-alpine
+    image: mysql:5.7
     environment:
-      - POSTGRES_DB=api
-      - POSTGRES_USER=api-platform
-      - POSTGRES_PASSWORD=!ChangeMe!
+      MYSQL_ROOT_PASSWORD: "root"
+      MYSQL_DATABASE: "api"
+      MYSQL_USER: "api-platform"
+      MYSQL_PASSWORD: "!ChangeMe!"
     volumes:
-      - db-data:/var/lib/postgresql/data:rw
+    - db-data:/var/lib/mysql
     ports:
-      - "5432:5432"
+    - 3306:3306

```
## Some errors I encountered while writing
```
An exception occurred in driver: SQLSTATE[HY000] [1044] Access denied for user 'api-platform'@'%' to database 'api'
```
It might help to remove the db volume `docker volume rm api-platform-232_db-data`

If you run `docker-compose up` for the first time you might have to redo it once because mysql isn't ready while api-platform tries to update the database.
