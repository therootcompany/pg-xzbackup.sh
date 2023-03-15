# [pg-xzbackup](https://github.com/therootcompany/pg-xzbackup.sh)

Scripts for automated backup and manual restore of PostgreSQL databases. \
(e.g. to encrypted, redundant Block Storage on Digital Ocean Volumes)

Especially useful for _not_ paying $200/month for small datasets. \
(e.g. Heroku where only "Premium" databases have 30+ day backups)

1. Authorize

    ```sh
    # save your credentials to a single-entry pgpass file
    echo "hostname:5432:dbname:username:secret" \
        > ./heroku-foo-100.pgpass

    # or convert an existing PG_URL into pgpass format
    pg-url-to-passfile \
        ./pg-url.txt \
        ./heroku-foobar-app.pgpass
    ```

2. Backup
    ```sh
    # pg-xzbackup <pg-passfile> <backup-dir> [file-prefix]
    pg-xzbackup \
        ./heroku-foo-100.pgpass \
        /mnt/pg-backups/ \
        foobar-app
    ```
3. Restore
    ```sh
    # pg-xzbackup <pg-passfile> <xz-backup-file>
    pg-xzrestore \
        ./heroku-foo-100.pgpass \
        /mnt/pg-backups/postgres-latest.sql.xz
    ```
4. Schedule
    - Install Serviceman
        ```sh
        curl -fsS https://webi.sh/serviceman | sh
        source ~/.config/envman/PATH.env
        ```
    - Schedule to run every _H_ hours and delete after _M_ months
        ```sh
        # pg-schedule-backups <h> <m> <pgpass> <bak-dir> [prefix]
        pg-schedule-backups \
            8 \
            60 \
            /mnt/pg-backups/heroku-foo-100.pgpass \
            /mnt/pg-backups/ \
            foobar-app
        ```
        (`pg-loop-backup` must be in `PATH`, [`pathman`](https://webinstall.dev/pathman) can help)
    - Check the logs to make sure the service is running successfully
        ```sh
        sudo journalctl -xef --unit pg-loop-backup.service
        ```

Your backups directory will look something like this:

```text
/mnt/pg_backups/
├── heroku-postgresql-foo-100.pgpass
├── foobar-app-2023-03-14_16.56.06.sql.xz
├── foobar-app-2023-03-15_01.02.33.sql.xz
└── foobar-app-2023-03-15_09.09.50.sql.xz
```

# Notes

-   You'll need to install `postgresql-client` and `xz`
    ```sh
    sudo apt install -y postgresql-client xz
    ```
-   You could use `cron` rather than `pg-loop-backup` + `serviceman`

## Heroku

-   To get your PostgreSQL Database URL:
    ```sh
    heroku config:get -a foobar-app DATABASE_URL > ./pg-url.txt
    pg-url-to-passfile ./pg-url.txt ./heroku-foobar-app.pgpass
    ```

## Secure, Inexpensive, Long-Term Backups

You can get years worth of storage for less than the price of a hamburger per month.

-   The cheapest Digital Ocean instance is about ~$5/month
-   10gb of encrypted, CEPH-redundant Block Storage is about ~$1/month

Vultr, Scaleway, Linode, etc are all similar.

## Utils

-   You may also find [`bat`](https://webinstall.dev/bat) useful for viewing the sql files.

# LICENSE

Copyright (c) 2023 AJ ONeal \
Copyright (c) 2023 Root, LLC

MPL-2.0
