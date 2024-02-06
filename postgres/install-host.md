# postgresql to host

```
sudo apt install postgresql

create user qxoo with encrypted password 'xxx';
alter user qxoo with encrypted password 'xxx';
grant all privileges on database mydb to qxoo;

CREATE ROLE qxoo SUPERUSER LOGIN PASSWORD 'password';
```

```
# pg16 install

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg

sudo apt update
sudo apt install postgresql-16 postgresql-contrib-16
```
