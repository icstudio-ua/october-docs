# Deployment

## Concepts

### Authenticating with Composer

When deploying your application with Composer, you may be prompted to provide your login credentials for the October CMS website. These credentials will authenticate your Composer session as having permission to download the October CMS source code.

To avoid manually typing in these credentials, you should make sure that the [Composer auth.json file](https://getcomposer.org/doc/articles/http-basic-authentication.md) is deployed with your application.

Alternatively, you can recreate this file using the `project:set` [artisan command](../console/commands.md#oc-set-project).

```sh
php artisan project:set <license key>
```

If the above command is not available, you may create the file using composer instead.

```sh
​composer config --auth http-basic.gateway.octobercms.com <email address> <license key>
```

### Deployment without Composer

<ProductIcon src="https://github.com/octobercms/docs/blob/develop/images/deploy-plugin.png?raw=true" />

If your server does not have the ability to run Composer or command line tools, such as micro instances and shared environments, you may use the [official deployment plugin](https://octobercms.com/plugin/rainlab-deploy) as a solution to deploying your website.

<div class="clearfix"></div>

## Security & Performance

<a id="oc-public-folder"></a>
### Public Folder

For ultimate security in production environments you may configure your web server to use a **public/** folder to ensure only public files can be accessed. First you will need to spawn a public folder using the `october:mirror` command.

```sh
php artisan october:mirror
```

This will create a new directory called **public/** in the project's base directory, from here you should modify the webserver configuration to use this new path as the home directory, also known as *wwwroot*.

The command should be performed after each system update or when a new plugin is installed. You may instruct October CMS to automatically do this using the `system.auto_mirror_public` configuration item.

```
AUTO_MIRROR_PUBLIC=true
```

> **Note**: For Windows operating systems, the `october:mirror` command must be performed in a console running as administrator. Therefore it is more suitable to run as part of a deployment process.

### Improving Performance

October CMS has been fine-tuned for performance and there are some extra steps you can take to increase this performance further, especially for larger sized applications.

Disable [debug mode](../setup/configuration.md#oc-debug-mode) and enable cache settings inside your [environment configuration](../setup/configuration.md#oc-environment-configuration).

```
APP_DEBUG=false
CMS_ROUTE_CACHE=true
CMS_ASSET_CACHE=true
CMS_TWIG_CACHE=true
```

Cache the configuration and route tables with these artisan commands.

```
php artisan config:cache
php artisan route:cache
```

### Shared Hosting

If you share a server with other users, you should act as if your neighbor's site was compromised. Make sure all files with passwords (e.g. CMS configuration files like `config/database.php`) cannot be read from other user accounts, even if they figure out absolute paths of your files. Setting permissions of such important files to 600 (read and write only to the owner and nothing to anyone else) is a good idea.

You can setup this protection in the file location `config/system.php` in the section titled **Default Permission Mask** or via the environment variables below.

```
DEFAULT_FILE_MASK=644
DEFAULT_FOLDER_MASK=755
```

> **Note**: Don't forget to manually check to see if the files are already set to 644, as you may need to go into your control panel and set them.

## Web Server Configuration

October CMS has basic configuration that should be applied to your webserver. Common webservers and their configuration can be found below.

### Apache Configuration

If your webserver is running Apache there are some extra system requirements:

1. mod_rewrite should be installed
1. AllowOverride option should be switched on

In some cases you may need to uncomment this line in the `.htaccess` file:

```
##
## You may need to uncomment the following line for some hosting environments,
## if you have installed to a subdirectory, enter the name here also.
##
# RewriteBase /
```

If you have installed to a subdirectory, you should add the name of the subdirectory also:

```
RewriteBase /mysubdirectory/
```

### Nginx Configuration

There are small changes required to configure your site in Nginx.

```sh
nano /etc/nginx/sites-available/default
```

Use the following code in **server** section. If you have installed October into a subdirectory, replace the first `/` in location directives with the directory October was installed under:

```
location / {
    # Let October CMS handle everything by default.
    # The path not resolved by October CMS router will return October CMS's 404 page.
    # Everything that does not match with the allowlist below will fall into this.
    rewrite ^/.*$ /index.php last;
}

# Pass the PHP scripts to FastCGI server
location ~ ^/index.php {
    # Write your FPM configuration here

}

# Allowlist
location ~ ^/favicon\.ico { try_files $uri /index.php; }
location ~ ^/sitemap\.xml { try_files $uri /index.php; }
location ~ ^/robots\.txt { try_files $uri /index.php; }
location ~ ^/humans\.txt { try_files $uri /index.php; }

# Block all .dotfiles except well-known
location ~ /\.(?!well-known).* { deny all; }

## Let nginx return 404 if static file not exists
location ~ ^/storage/app/uploads/public { try_files $uri 404; }
location ~ ^/storage/app/media { try_files $uri 404; }
location ~ ^/storage/app/resources { try_files $uri 404; }
location ~ ^/storage/temp/public { try_files $uri 404; }

location ~ ^/modules/.*/assets { try_files $uri 404; }
location ~ ^/modules/.*/resources { try_files $uri 404; }
location ~ ^/modules/.*/behaviors/.*/assets { try_files $uri 404; }
location ~ ^/modules/.*/behaviors/.*/resources { try_files $uri 404; }
location ~ ^/modules/.*/widgets/.*/assets { try_files $uri 404; }
location ~ ^/modules/.*/widgets/.*/resources { try_files $uri 404; }
location ~ ^/modules/.*/formwidgets/.*/assets { try_files $uri 404; }
location ~ ^/modules/.*/formwidgets/.*/resources { try_files $uri 404; }
location ~ ^/modules/.*/reportwidgets/.*/assets { try_files $uri 404; }
location ~ ^/modules/.*/reportwidgets/.*/resources { try_files $uri 404; }

location ~ ^/plugins/.*/.*/assets { try_files $uri 404; }
location ~ ^/plugins/.*/.*/resources { try_files $uri 404; }
location ~ ^/plugins/.*/.*/behaviors/.*/assets { try_files $uri 404; }
location ~ ^/plugins/.*/.*/behaviors/.*/resources { try_files $uri 404; }
location ~ ^/plugins/.*/.*/reportwidgets/.*/assets { try_files $uri 404; }
location ~ ^/plugins/.*/.*/reportwidgets/.*/resources { try_files $uri 404; }
location ~ ^/plugins/.*/.*/formwidgets/.*/assets { try_files $uri 404; }
location ~ ^/plugins/.*/.*/formwidgets/.*/resources { try_files $uri 404; }
location ~ ^/plugins/.*/.*/widgets/.*/assets { try_files $uri 404; }
location ~ ^/plugins/.*/.*/widgets/.*/resources { try_files $uri 404; }

location ~ ^/themes/.*/assets { try_files $uri 404; }
location ~ ^/themes/.*/resources { try_files $uri 404; }
```

### Lighttpd Configuration

If your webserver is running Lighttpd you can use the following configuration to run October CMS. Open your site configuration file with your favorite editor.

```sh
nano /etc/lighttpd/conf-enabled/sites.conf
```

Paste the following code in the editor and change the **host address** and  **server.document-root** to match your project.

```
$HTTP["host"] =~ "domain.example.com" {
    server.document-root = "/var/www/example/"

    url.rewrite-once = (
        "^/(plugins|modules/(system|backend|cms))/(([\w-]+/)+|/|)assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
        "^/(system|themes/[\w-]+)/assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
        "^/storage/app/uploads/public/[\w-]+/.*$" => "$0",
        "^/storage/app/media/.*$" => "$0",
        "^/storage/app/resources/.*$" => "$0",
        "^/storage/temp/public/[\w-]+/.*$" => "$0",
        "^/(favicon\.ico)$" => "$0",
        "(.*)" => "/index.php$1"
    )
}
```

### IIS Configuration

If your webserver is running Internet Information Services (IIS) you can use the following in your **web.config** configuration file to run October CMS.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <clear />
                <rule name="October CMS to handle all non-allowlisted URLs" stopProcessing="true">
                    <match url="^(.*)$" ignoreCase="false" />
                    <conditions logicalGrouping="MatchAll">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/.well-known/*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/uploads/public/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/media/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/resources/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/temp/public/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/themes/.*/(assets|resources)/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/plugins/.*/(assets|resources)/.*" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/modules/.*/(assets|resources)/.*" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" appendQueryString="true" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```
