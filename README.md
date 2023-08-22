# Redmine

Redmine is a flexible project management web application written using Ruby on Rails framework.

More details can be found in the doc directory or on the [official website](http://www.redmine.org)

[Redmine installation](https://www.redmine.org/projects/redmine/wiki/redmineinstall)

---

## Installation on Windows

### Install dependencies for redmine

1. Install postgresql 10.23 via following command:

```cmd
./postgress.exe --install_runtimes 0
```

2. Install rubyinstaller+devkit 3.0

3. Compile xapian-core and xapian-bindings for windows from [here](https://xapian.org/download) (from bash with unix tools):

```bash
cd {folder_with_packages}
tar -xJf {core}.tar.xz
tar -xJf {bindings}.tar.xz
cd {core}
./configure
make
make install
cd ../{bindings}
./configure --with-ruby LDFLAGS='-L{path_to_ruby_folder}/lib' RUBY_LIB={path_to_site_lib} RUBY_LIB_ARCH={path_to_site_lib}
make
make install
```

4. Install nginx

### Setup configuration for redmine

1. Create database and user for redmine

```sql
CREATE ROLE redmine LOGIN ENCRYPTED PASSWORD 'my_password' NOINHERIT VALID UNTIL 'infinity';
CREATE DATABASE redmine WITH ENCODING='UTF8' OWNER=redmine;
ALTER DATABASE "redmine_db" SET datestyle="ISO,MDY"
```

2. Setup *database.yml* in config folder

```yml
production:
  adapter: postgresql
  database: <your_database_name>
  host: <postgres_host>
  username: <postgres_user>
  password: "<postgres_user_password>"
  encoding: utf8
  schema_search_path: <database_schema> (default - public)
```

3. Setup nginx congiguration

```config
server {
 listen 3001;
    root C:/redmine/public;

    try_files $uri/index.html $uri @redmine;

    upstream redmine {
        server 127.0.0.1:3000;
 }

 location @redmine {
  proxy_pass http://redmine;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_redirect off;
 }
}
```

4. Load submodules with plugins, more information [here](https://github.com/mrsadman99/redmine_dmsf)

```cmd
git submodule update --init --recursive
```

5. Setup config and install dependencies for ruby

```cmd
bundle config set --local without 'development test'
bundle install
```

6. Generate secret token

```cmd
bundle exec rake generate_secret_token
```

7. Migrate and load default data for db

```cmd
set RAILS_ENV=production
bundle exec rake db:migrate
bundle exec rake redmine:load_default_data
bundle exec rake redmine:plugins:migrate NAME=redmine_dmsf
```

8. Run server and nginx

```cmd
start nginx
bundle exec rails server -u puma -e production
```
