# Version History

It is good practice for plugins to maintain a change log that documents any changes or improvements in the code. In addition to writing notes about changes, this process has the useful ability to execute [migration and seed files](../database/structure.md) in their correct order.

The change log is stored in a YAML file called `version.yaml` inside the **updates** directory of a plugin, which co-exists with migration and seed files. This example displays a typical plugin updates directory structure:

::: dir
├── plugins
|   └── author
|       └── myplugin
|           ├── `updates`
|           |   ├── `version.yaml`     _<== Version File_
|           |   ├── create_tables.php _<== Database Script_
|           |   ├── seed_the_database.php
|           |   └── create_another_table.php
|           └── Plugin.php
:::

### Plugin Dependencies

Updates are applied in a specific order, based on the [defined dependencies in the plugin registration file](../plugin/registration.md#oc-dependency-definitions). Plugins that are dependant will not be updated until all their dependencies have been updated first.

```php
<?php namespace Acme\Blog;

class Plugin extends \System\Classes\PluginBase
{
    public $require = ['Acme.User'];
}
```

In the example above the **Acme.Blog** plugin will not be updated until the **Acme.User** plugin has been fully updated.

## Plugin Version File

The **version.yaml** file, called the *Plugin Version File*, contains the version comments and refers to database scripts in the correct order. Please read the [Database structure](../database/structure.md) article for information about the migration files. This file is required if you're going to submit the plugin to the [Marketplace](https://octobercms.com/help/site/marketplace). Here is an example of a plugin version file:

```yaml
v1.0.1: First version
v1.0.2: Second version
v1.0.3:
    - Update with a migration and seed
    - create_tables.php
    - seed_the_database.php
v2.0.0: Important update
v2.0.1: Latest version
```

> **Note**: The `version.yaml` file should always use the first line for a text update that describes the changes and the remaining lines for update scripts. For more verbose updates, consider using a dedicated changelog file.

As you can see above, there should be a key that represents the version number followed by the update message, which is either a string or an array containing update messages. For updates that refer to migration or seeding files, lines that are script file names can be placed in any position. An example of a comment with no associated update files.

```yaml
v1.0.1: A single comment that uses no update scripts.
```

<a id="oc-important-updates"></a>
### Important Updates

Sometimes a plugin needs to introduce features that will break websites already using the plugin. To prevent the changes from being deployed automatically, you should increase the **major** segment of the version string (`major.minor.patch`). An example of an important update comment is below.

```yaml
v2.1.0: This is an important update from v1 that contains breaking changes.
```

When tagging the new version `v2` from a version `v1` then the changes are not deployed as part of a regular update. The user must install the plugin again to receive the latest version via Composer.

<a id="oc-migration-files"></a>
### Migration and Seed Files

As previously described, updates also define when [migration and seed files](../database/structure.md) should be applied. An update line with a comment and updates:

```yaml
v1.1.1:
    - This update will execute the two scripts below.
    - some_upgrade_file.php
    - some_seeding_file.php
```

The update file name should use *snake_case* while the containing PHP class should use *CamelCase*. For a file named **some_upgrade_file.php** the corresponding class would be `SomeUpgradeFile`.

```php
<?php namespace Acme\Blog\Updates;

use Schema;
use October\Rain\Database\Updates\Migration;

/**
 * some_upgrade_file.php
 */
class SomeUpgradeFile extends Migration
{
    ///
}
```
