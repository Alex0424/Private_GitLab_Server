# PostgreSQL Configuration (Database Server)
```
sudo ufw allow http
sudo ufw allow https
sudo ufw allow ssh
sudo ufw allow 5432/tcp
sudo ufw enable
```


```
sudo apt update
sudo apt install postgresql postgresql-contrib
```

vim /etc/postgresql/16/main/postgresql.conf
```
listen_addresses = '0.0.0.0' 

shared_buffers = 512MB
```

vim /etc/postgresql/16/main/pg_hba.conf
```
host    all             gitlab          0.0.0.0/0         md5
```


```
sudo -u postgres psql

CREATE DATABASE gitlabhq_production;
CREATE USER gitlab WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE gitlabhq_production TO gitlab;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO gitlab;
GRANT ALL ON SCHEMA public TO gitlab;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO gitlab;
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO gitlab;



sudo systemctl restart postgresql
```

# Gitlab Configuration (Gitlab Server)

```
sudo ufw allow http
sudo ufw allow https
sudo ufw allow ssh
sudo ufw allow 5432/tcp
sudo ufw enable
```

```
sudo apt update
sudo apt install -y curl openssh-server ca-certificates tzdata perl

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

sudo apt install gitlab-ce
```






vim /etc/gitlab/gitlab.rb
```
postgresql['enable'] = false

gitlab_rails['db_adapter'] = 'postgresql'
gitlab_rails['db_encoding'] = 'unicode'
gitlab_rails['db_host'] = 'DB-IP-HERE' #<------------------ REPLACE DB-IP-HERE with database ip address
gitlab_rails['db_port'] = 5432
gitlab_rails['db_username'] = 'gitlab'
gitlab_rails['db_password'] = 'secure_password'
gitlab_rails['db_database'] = 'gitlabhq_production'
gitlab_rails['db_pool'] = 10

## Disable everything else
#sidekiq['enable'] = false
#puma['enable'] = false
#registry['enable'] = false
#gitaly['enable'] = false
#gitlab_workhorse['enable'] = false
#nginx['enable'] = false
#prometheus_monitoring['enable'] = false
#redis['enable'] = false
#gitlab_kas['enable'] = false
```
```
sudo gitlab-ctl reconfigure
```

# Trouble-Shooting (if you get errors do this):

## on GITLAB VM


install db (on GITLAB VM): `sudo apt install postgresql-client`

REPLACE "DB-IP-HERE" with your database IP:

`PGPASSWORD="secure_password" psql -h DB-IP-HERE -U gitlab -d gitlabhq_production -p 5432`

if root password doesent work in: `vim /etc/gitlab/initial_root_password`

then create a new root password: `sudo gitlab-rake "gitlab:password:reset"`
