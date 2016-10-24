The following is a simple guide for setting up [GitLab CE](https://about.gitlab.com/) on an Ubuntu server.

## Setup

1. Install [nginx](https://nginx.org/en/)
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

## Upgrading

Since we're using the GitLab CE Omnibus package, simply update the package:

        $ sudo apt-get update
        $ sudo apt-get install gitlab-ce

## Backup

Add the following entry to your crontab to backup GitLab every morning at 02:00:

        0  2  *  *  *    /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1 1>&2