# Installation

<VideoPreview src="https://www.youtube.com/watch?v=RHUwCvo7xng" />

October CMS is a web application platform with a simple and intuitive interface. A web platform provides a consistent structure with an emphasis on reusability so you can focus on building something unique while we handle the boring bits.

October CMS makes one bold but obvious assumption: clients don't build websites; developers do. When platforms have a client-centric development process, it only results in one thing: an unhappy developer.

Whether you're new to web development or have years of experience, October CMS is a platform that makes it easy to create bespoke experiences for you and your clients. We hope you enjoy your journey with us and discover happiness in simplicity.

To run October CMS locally, we recommend the following software:

- [Laravel Valet for macOS](https://laravel.com/docs/valet)
- [Laragon for Windows 10](https://laragon.org/)

## Installing Composer

October CMS uses [Composer](http://getcomposer.org/) to manage its dependencies. So before getting started, you will need to make sure you have Composer installed.

You should also check that your computer or server meets the [minimum system requirements](#oc-minimum-system-requirements) for running the PHP application.

## Installing October CMS

You can then create a new October CMS project by using `create-project` command in your terminal. The following creates a new project in a directory called **myoctober**.

```bash
composer create-project october/october myoctober "^2.0"
```

You may also install to the current directory using this command instead.

```bash
composer create-project october/october . "^2.0"
```

When the task finishes, run the installation command to guide you through the next steps.

```bash
php artisan october:install
```

Next, migrate the database with the following command.

```bash
php artisan october:migrate
```

You can then serve the application and open it in your browser.

```bash
php artisan serve
```

> **Note**: If you are using a project, continue reading the [Projects article](https://octobercms.com/help/site/projects) for information on how to set up your project.

## Post-Installation Steps

There are some things you may need to set up after the installation is complete.

### Review Configuration

Configuration files are stored in the **config** directory of the application. While each file contains descriptions for each setting, it is important to review the [common configuration options](../setup/configuration.md) available for your circumstances.

For example, in production environments you may want to do the following:

- Enable [CSRF protection](../setup/configuration.md#oc-csrf-protection)
- Disable [debug mode](../setup/configuration.md#oc-debug-mode)
- Use a [public folder](../setup/deployment.md#oc-public-folder) for additional security

<a id="oc-setting-up-the-scheduler"></a>
### Setting Up The Scheduler

For *scheduled tasks* to operate correctly, you should add the following Cron entry to your server. Editing the crontab is commonly performed with the command `crontab -e`.

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

Be sure to replace **/path/to/artisan** with the absolute path to the *artisan* file in the root directory of October CMS. This Cron will call the command scheduler every minute. Then October CMS evaluates any scheduled tasks and runs the tasks that are due.

> **Note**: If you are adding this to `/etc/cron.d` you'll need to specify a user immediately after `* * * * *`.

### Setting Up Queue Workers

You may optionally set up an external queue for processing *queued jobs*, by default these will be handled asynchronously by the platform. This behavior can be changed by setting the `default` parameter in the `config/queue.php`.

If you decide to use the `database` queue driver, it is a good idea to add a Crontab entry for the command `php artisan queue:work --once` to process the first available job in the queue.

You can also run the queue as a daemon process with

```bash
php artisan queue:work
```

<a id="oc-minimum-system-requirements"></a>
## Minimum System Requirements

October CMS has some server requirements for web hosting:

1. PHP version 7.4 or higher
1. Composer 2.0 or higher
1. PDO PHP Extension (and relevant driver for the database you want to connect to)
1. cURL PHP Extension
1. OpenSSL PHP Extension
1. Mbstring PHP Extension
1. ZipArchive PHP Extension
1. GD PHP Extension
1. SimpleXML PHP Extension

Support is provided for these databases with minimum requirements:

1. MySQL 5.7 or MariaDB 10.2
1. PostgreSQL 9.6
1. SQLite 3.8.8

Please note that October CMS v2 is not compatible with PHP 8.1, the [version 3 beta](https://octobercms.com/support/article/rn-28) is available to meet this requirement. It is recommended to downgrade to PHP 8.0 and use version 2 for the best stability and support.

If you are using an older version of MySQL or MariaDB, you may need to [configure the index lengths](../database/structure.md#oc-index-lengths-using-mysql-mariadb) to support the `utf8mb4` character set.

Some OS distributions may require you manually install some of the necessary PHP extensions. When using Ubuntu, the following command can be run to install all required extensions:

```bash
sudo apt-get update &&
sudo apt-get install php php-ctype php-curl php-xml php-fileinfo php-gd php-json php-mbstring php-mysql php-sqlite3 php-zip
```

When using the SQL Server database engine, you will need to install the [group concatenation](https://github.com/orlando-colamatteo/ms-sql-server-group-concat-sqlclr) user-defined aggregate.

## Troubleshooting Installation

1. **The installation hangs or freezes when I enter the license key**: This can happen in some environments when pasting the key contents. Press the ENTER key multiple times to allow the installation process to continue.

1. **An error "Specified key was too long" is displayed during migration**: This happens when you are using an older version of MySQL or MariaDB. To resolve this issue, you may need to [configure the index lengths](../database/structure.md#oc-index-lengths-using-mysql-mariadb) to support the `utf8mb4` character set.

1. **An error "Deprecated: Return type of offsetExists($key) should either be compatible with offsetExists(mixed $offset)..."**: This happens when using PHP 8.1 or above, use a lower version of PHP to resolve this issue. The recommended PHP version for October CMS 2.0 is PHP 8.0.

1. **A blank screen is displayed when opening the application**: Check the permissions are set correctly on the `/storage` files and folders, they should be writable for the web server.

1. **The back-end area displays "Page not found" (404)**: If the application cannot find the database then a 404 page will be shown for the back-end. Try enabling [debug mode](../setup/configuration.md#oc-debug-mode) to see the underlying error message.

1. **An error 500 is displayed when updating the application**: You may need to increase or disable the timeout limit on your webserver. For example, Apache's FastCGI sometimes has the `-idle-timeout` option set to 30 seconds.

> **Note**: A detailed system log can be found in the `storage/logs` directory.
