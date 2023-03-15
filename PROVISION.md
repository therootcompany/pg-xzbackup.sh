# Server Setup

## 0. Get Heroku DB URL & Your SSH Key

On the dev machine that logs into the Heroku web portal \
(commandline tool must be run from the same IP address)

```sh
my_app="foobar"
heroku config:get -a "${my_app}" DATABASE_URL
```

```sh
# Mac, Linux
curl -fsS https://webi.sh/ssh-pubkey | sh

# Windows
curl.exe https://webi.sh/ssh-pubkey | powershell
```

Save these for later...

## 1. VPS + Block Storage Volume

Pick any VPS service of your choosing (or self-host).

-   Pick the smallest VPS instance, ~$5/month \
    (preferably with VPS-level backups enabled)
-   Get 10gb+ of encrypted, redundant Block Storage, ~$1/month
-   Save your **IP Address**

Digital Ocean, Vultr, Scaleway, Linode, and many others all do a fine job.

## 2. Create `app` user

This will create an `app` user with the same `ssh` keys as `root`:

```sh
my_host=127.0.0.1
ssh root@"${my_host}" 'curl https://webi.sh/ssh-adduser | sh'
```

Login as `app`:

```sh
my_host=127.0.0.1
ssh app@"${my_host}"
```

Apply any security hardening that may have been performed by `ssh-adduser`:

```sh
sudo systemctl restart ssh
```

Change the ownership of the Block Storage volume:

## 3. Get good dev tools

```sh
sudo apt update
sudo apt install -y fish postgresql-client

webi beyond-shell
webi bat rg serviceman

# one-time, to update current shell session
source ~/.config/envman/PATH.env
source ~/.config/envman/alias.env
```

## 4. Setup Postgres Backups

Change ownership of encrypted, redundant backup volume:

```sh
my_backup_vol="/mnt/pg_backups_10gb/"

sudo chown -R app:app "${my_backup_vol}"
```

Your Heroku app's `DATABASE_URL` (from way up above) will be in this format:

```text
# Format
postgres://<alpha-user>:<hex-pass>@<aws-ec2>:5432/<alphanum-dbname>

# Example
postgres://aaaaaaaaaaaaaa:ffffffffffffffff@ec2-000-0-00-00.compute-0.amazonaws.com:5432/xxxxxxxxxxxxxx
```

You'll need to adjust that into colon-separated (`:`) `pgpass` format:

```text
# Format
<db-host>:5432:<db-name>:<db-user>:<db-pass>

# Example
ec2-000-0-00-00.compute-0.amazonaws.com:5432:xxxxxxxxxxxxxx:aaaaaaaaaaaaaa:ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```

```sh
my_backup_vol="/mnt/pg_backups_10gb/"

my_app="foobar"
heroku config:get -a "${my_app}" DATABASE_URL > "${my_backup_vol}/pg-url.txt"

pg-url-to-passfile "${my_backup_vol}/pg-url.txt" "${my_backup_vol}/heroku.pgpass"

rm "${my_backup_vol}/pg-url.txt"
```

```sh
my_backup_vol="/mnt/pg_backups_10gb/"

sudo chmod -R 600 "${my_backup_vol}"
```


## 5. Schedule backups

```sh
git clone
```

