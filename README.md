# Magento 2 Docker

A collection of Docker images for running Magento 2 through nginx and on the command line.

## 1. Quick Start

    1. cp composer.env.sample composer.env  - put the correct tokens into composer.env. See "2. Configuration" 

    2. export MAGENTO_BASE_URL=http://magento2.local/   - You should execute this command under the same user that launches the docker. Otherwise, the variable will not be visible. 
   
    3. docker-compose up

    4. Run "docker inspect magento2_web_1"  and copy "IPAddress":
    5. Paste IP address and host name into your hosts file.
    6. Go to http://magento2.local/index.php
   
Admin url is http://magento2.local/index.php/admin
login: admin
password: password1

## 2. Configuration

Configuration is driven through environment variables.  A comprehensive list of the environment variables used can be found in each `Dockerfile` and the commands in each `bin/` directory.

* `PHP_MEMORY_LIMIT` - The memory limit to be set in the `php.ini`
* `UPLOAD_MAX_FILESIZE` - Upload filesize limit for PHP and Nginx
* `MAGENTO_RUN_MODE` - Valid values, as defined in `Magento\Framework\App\State`: `developer`, `production`, `default`.
* `MAGENTO_ROOT` - The directory to which Magento should be installed (defaults to `/var/www/magento`)
* `COMPOSER_GITHUB_TOKEN` - Your [GitHub OAuth token](https://getcomposer.org/doc/articles/troubleshooting.md#api-rate-limit-and-oauth-tokens), should it be needed
* `COMPOSER_MAGENTO_USERNAME` - Your Magento Connect public authentication key ([how to get](http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html))
* `COMPOSER_MAGENTO_PASSWORD` - Your Magento Connect private authentication key
* `COMPOSER_BITBUCKET_KEY` - Optional - Your Bitbucket OAuth key ([how to get](https://confluence.atlassian.com/bitbucket/oauth-on-bitbucket-cloud-238027431.html))
* `COMPOSER_BITBUCKET_SECRET` - Optional - Your Bitbucket OAuth secret
* `DEBUG` - Toggles tracing in the bash commands when exectued; nothing to do with Magento`
* `PHP_ENABLE_XDEBUG` - When set to `true` it will include the Xdebug ini file as part of the PHP configuration, turning it on. It's recommended to only switch this on when you need it as it will slow down the application.
* `UPDATE_UID_GID` - If this is set to "true" then the uid and gid of `www-data` will be modified in the container to match the values on the mounted folders.  This seems to be necessary to work around virtualbox issues on OSX.

A sample `docker-compose.yml` is provided in this repository.

## 3. CLI Usage

A number of commands are baked into the image and are available on the `$PATH`. These are:

* `magento-command` - Provides a user-safe wrapper around the `bin/magento` command.
* `magento-installer` - Installs and configures Magento into the directory defined in the `$MAGENTO_ROOT` environment variable.
* `magento-extension-installer` - Installs a Magento 2 extension from the `/extensions/<name>` directory, using symlinks.
* `magerun2` - A user-safe wrapper for `n98-magerun2.phar`, which provides a wider range of useful commands. [Learn more here](https://github.com/netz98/n98-magerun2)

It's recommended that you mount an external folder to `/root/.composer/cache`, otherwise you'll be waiting all day for Magento to download every time the container is booted.

CLI commands can be triggered by running:

    docker-compose run cli magento-installer

Shell access to a CLI container can be triggered by running:

    docker-compose run cli bash

# How to test API customization

1. Send first request and get token
```
curl -X POST "http://magento2.local/index.php/rest/V1/integration/admin/token" \
     -H "Content-Type:application/xml"  \
     -d "<login><username>admin</username><password>password1</password></login>"
```
2. ``` curl -X GET "http://magento2.local/index.php/rest/V1/product/getByVPN/{vpn}" -H "Authorization: Bearer xxxxxxxxxxxxxxxxxx"  ```

Where  xxxxxxxxxxxxxxxxxx   is your token.

3. Test  update/  url

`` curl -H "Content-Type: application/json" -X PUT \
-d '{"data":{"entity_id":"1", "copywrite_info":"hello", "store_id":"0", "vpn":"75756"}}' \
http://magento2.local/index.php/rest/V1/product/update/  ``
