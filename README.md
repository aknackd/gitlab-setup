The following is a simple guide for setting up [GitLab CE](https://about.gitlab.com/) on an Ubuntu server using an existing NGINX setup.

## Setup

1. Install [NGINX](https://nginx.org/en/)
2. [Install GitLab CE Ominbus package](https://about.gitlab.com/downloads/)
3. Configure GitLab
    - Copy `gitlab.rb` to `/etc/gitlab/gitlab.rb` and modify according to your environment
    - Copy `nginx.conf` to `/etc/nginx/sites-available/gitlab` and modify according to your environment
    - Create GitLab directories:

        ```
        $ export GITLAB_DATA_DIR=/path/to/gitlab
        $ sudo mkdir -p $GITLAB_DATA_DIR/{backups,uploads,git-data}
        $ sudo chown -R git:root $GITLAB_DATA_DIR
        $ sudo chmod -R 700 $GITLAB_DATA_DIR/{git-data,uploads}
        ```

    - Restart GitLab via `sudo gitlab-ctl restart`
    - Enable `gitlab` nginx site and restart nginx:

        ```
        $ sudo ln -s /etc/nginx/conf.d/sites-enabled/gitlab /etc/nginx/conf.d/sites-available/gitlab
        $ sudo service nginx restart
        ```

### Environment-specific settings

#### [GitLab](gitlab.rb)

| *Setting*                               | *Description*                                       | *Notes*                                                                            |
|-----------------------------------------|-----------------------------------------------------|------------------------------------------------------------------------------------|
| `external_url`                          | External URL of GitLab instance                     |                                                                                    |
| `git_data_dir`                          | Git repo storage directory                          |                                                                                    |
| `gitlab_rails['gitlab_email_from']`     | From email address of GitLab-sent emails            | Replace `mycompany.com` with the appropriate domain                                |
| `gitlab_rails['gitlab_email_reply_to']` | Reply-To email address of GitLab-sent emails        | GitLab will not process emails unless the `Reply by email` section is configured   |
| `gitlab_rails['backup_path']`           | Path where GitLab backups are stored                | Path must exist and be writable by the `git` user                                  |
| `gitlab_rails['backup_keep_time']`      | How far back backups are kept                       | In seconds (ex: `604800` keeps backups from the last 7 days)                       |
| `gitlab_rails['uploads_directory']`     | Path where attachments and other uploads are stored | Path must exist and be writable by the `git` user                                  |
| `gitlab_rails['smtp_*']`                | GitLab email configuration                          | `gitlab_rails['smtp_domain']`: Replace `mycompany.com` with the appropriate domain |

#### [NGINX](nginx.conf)

| *Setting*             | *Description*               | *Notes*                                                                                          |
|-----------------------|-----------------------------|--------------------------------------------------------------------------------------------------|
| `server_name`         | Server name                 | Should match the `external_url` setting in `gitlab.rb` (domain only, no `http://` or `https://`) |
| `ssl_certificate`     | Path to SSL certificate     |                                                                                                  |
| `ssl_certificate_key` | Path to SSL certificate key |                                                                                                  |
| `ssl_dhparam`         | Path to DHE file            | Generate via `openssl dhparam -out /path/to/dhparam.pem 4096`                                    |

## Upgrading

Since we're using the GitLab CE Omnibus package, simply update the package:

        $ sudo apt-get update
        $ sudo apt-get install gitlab-ce

GitLab and all its components (database, unicorn, etc.) will automatically be updated and restarted.

## Backup

Add the following entry to your crontab to backup GitLab every morning at 02:00:

        0  2  *  *  *    /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1 1>&2