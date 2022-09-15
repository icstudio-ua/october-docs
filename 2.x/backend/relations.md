# Relations

**Relation Behavior** is a controller modifier used for easily managing complex [model](../database/model.md) relationships on a page. Not to be confused with [List relation columns](lists.md#oc-available-column-types) or [Form relation fields](forms.md#widget-relation) that only provide simple management.

Relation behavior depends on [relation definitions](#oc-relationship-types). In order to use the relation behavior you should add the `Backend\Behaviors\RelationController` definition to the `$implement` field of the controller class. Also, the `$relationConfig` class property should be defined and its value should refer to the YAML file used for [configuring the behavior options](#oc-configuring-the-relation-behavior).

```php
namespace Acme\Projects\Controllers;

class Projects extends Controller
{
    public $implement = [
        \Backend\Behaviors\FormController::class,
        \Backend\Behaviors\RelationController::class
    ];

    public $formConfig = 'config_form.yaml';
    public $relationConfig = 'config_relation.yaml';
}
```

> **Note**: Very often the relation behavior is used together with the [form behavior](forms.md).

<a id="oc-configuring-the-relation-behavior"></a>
## Configuring the Relation Behavior

The configuration file referred in the `$relationConfig` property is defined in YAML format. The file should be placed into the controller's [views directory](controllers-ajax.md). The required configuration depends on the [relationship type](#oc-relationship-types) between the target model and the related model.

The first level field in the relation configuration file defines the relationship name in the target model. For example:

```php
class Invoice
{
    public $hasMany = [
        'items' => \Acme\Pay\Models\InvoiceItem::class,
    ];
}
```

An *Invoice* model with a relationship called `items` should define the first level field using the same relationship name:

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

items:
    label: Invoice Line Item
    view:
        list: $/acme/pay/models/invoiceitem/columns.yaml
        toolbarButtons: create|delete
    manage:
        form: $/acme/pay/models/invoiceitem/fields.yaml
        recordsPerPage: 10
```

The following options are then used for each relationship name definition:

Option | Description
------------- | -------------
**label** | a label for the relation, in the singular tense, required.
**view** | configuration specific to the view container, see below.
**manage** | configuration specific to the management popup, see below.
**pivot** | a reference to form field definition file, used for [relations with pivot table data](#oc-belongs-to-many-with-pivot-data).
**emptyMessage** | a message to display when the relationship is empty, optional.
**readOnly** | disables the ability to add, update, delete or create relations. default: false
**deferredBinding** | [defers all binding actions using a session key](../database/relations.md#oc-deferred-binding) when it is available. default: false

These configuration values can be specified for the **view** or **manage** options, where applicable to the render type of list, form or both.

Option | Type | Description
------------- | ------------- | -------------
**form** | Form | a reference to form field definition file, see [backend form fields](forms.md#oc-defining-form-fields).
**list** | List | a reference to list column definition file, see [backend list columns](lists.md#oc-defining-list-columns).
**showFlash** | Both | enables the display of flash messages after a successful action. Default: true
**showSearch** | List | display an input for searching the records. Default: false
**showSorting** | List | displays the sorting link on each column. Default: true
**defaultSort** | List | sets a default sorting column and direction when user preference is not defined. Supports a string or an array with keys `column` and `direction`.
**recordsPerPage** | List | maximum rows to display for each page.
**noRecordsMessage** | List | a message to display when no records are found, can refer to a [localization string](../plugin/localization.md).
**conditions** | List | specifies a raw where query statement to apply to the list model query.
**scope** | List | specifies a [query scope method](../database/model.md#oc-query-scopes) defined in the _related form model_ to apply to the list query always. The model that this relationship will be attached to (i.e. the parent model) is passed to this scope method as the second parameter (`$query` is the first).
**searchMode** | List | defines the search strategy to either contain all words, any word or exact phrase. Supported options: all, any, exact. Default: all.
**searchScope** | List | specifies a [query scope method](../database/model.md#oc-query-scopes) defined in the **related form model** to apply to the search query, the first argument will contain the search term.
**filter** | List | a reference to a filter scopes definition file, see [backend list filters](lists.md#oc-using-list-filters).

These configuration values can be specified only for the **view** options.

Option | Type | Description
------------- | ------------- | -------------
**showCheckboxes** | List | displays checkboxes next to each record.
**recordUrl** | List | link each list record to another page. Eg: **users/update/:id**. The `:id` part is replaced with the record identifier.
**customViewPath** | List | specify a custom view path to override partials used by the list.
**recordOnClick** | List | custom JavaScript code to execute when clicking on a record.
**toolbarPartial** | Both | a reference to a controller partial file with the toolbar buttons. Eg: **_relation_toolbar.htm**. This option overrides the *toolbarButtons* option.
**toolbarButtons** | Both | the set of buttons to display. This can be formatted as an array or a pipe separated string, or set to `false` to show no buttons. Available options are: `create`, `update`, `delete`, `add`, `remove`, `link`, & `unlink`. Example: `add|remove`.
**structure** | List | options to enable [sorting records](../backend/reorder.md) for the list.

These configuration values can be specified only for the **manage** options.

Option | Type | Description
------------- | ------------- | -------------
**title** | Both | a popup title, can refer to a [localization string](../plugin/localization.md).
**context** | Form | context of the form being displayed. Can be a string or an array with keys: create, update.

### Custom Messages

Specify the `customMessages` option to override the default messages used by the Relation Controller. The values can be plain text or can refer to a [localization string](../plugin/localization.md).

```yaml
customMessages:
    buttonCreate: Make Thing
    buttonDelete: Destroy Thing
```

You may also modify messages in the context of the related field being displayed. The following will override the `createButton` message for the `items` relation only.

```yaml
items:
    customMessages:
        buttonCreate: New Item!
```

The following messages are available to override as custom messages.

Message | Default Message
------------- | -------------
buttonCreate | Create :name
buttonUpdate | Update :name
buttonAdd | Add :name
buttonLink | Link :name
buttonDelete | Delete
buttonRemove | Remove
buttonUnlink | Unlink
confirmDelete | Are you sure?
confirmUnlink | Are you sure?
titlePreviewForm | Preview :name
titleCreateForm | Create :name
titleUpdateForm | Update :name
titleLinkForm | Link a New :name
titleAddForm | Add a New :name
titlePivotForm | Related :name Data
flashCreate | :name Created
flashUpdate | :name Updated
flashDelete | :name Deleted
flashAdd | :name Added
flashLink | :name Linked
flashRemove | :name Removed
flashUnlink | :name Unlinked

<a id="oc-relationship-types"></a>
## Relationship Types

How the relation manager is displayed depends on the relationship definition in the target model. The relationship type will also determine the configuration requirements, these are shown in **bold**. The following relationship types are available:

### Has Many

1. Related records are displayed as a list (**view.list**).
1. Clicking a record will display an update form (**manage.form**).
1. Clicking *Add* will display a selection list (**manage.list**).
1. Clicking *Create* will display a create form (**manage.form**).
1. Clicking *Delete* will destroy the record(s).
1. Clicking *Remove* will orphan the relationship.

For example, if a *Blog Post* has many *Comments*, the target model is set as the blog post and a list of comments is displayed, using columns from the **list** definition. Clicking on a comment opens a popup form with the fields defined in **form** to update the comment. Comments can be created in the same way. Below is an example of the relation behavior configuration file:

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

comments:
    label: Comment
    manage:
        form: $/acme/blog/models/comment/fields.yaml
        list: $/acme/blog/models/comment/columns.yaml
    view:
        list: $/acme/blog/models/comment/columns.yaml
        toolbarButtons: create|delete
```

### Belongs to Many

1. Related records are displayed as a list (**view.list**).
1. Clicking *Add* will display a selection list (**manage.list**).
1. Clicking *Create* will display a create form (**manage.form**).
1. Clicking *Delete* will destroy the pivot table record(s).
1. Clicking *Remove* will orphan the relationship.

For example, if a *User* belongs to many *Roles*, the target model is set as the user and a list of roles is displayed, using columns from the **list** definition. Existing roles can be added and removed from the user. Below is an example of the relation behavior configuration file:

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

roles:
    label: Role
    view:
        list: $/acme/user/models/role/columns.yaml
        toolbarButtons: add|remove
    manage:
        list: $/acme/user/models/role/columns.yaml
        form: $/acme/user/models/role/fields.yaml
```

<a id="oc-belongs-to-many-with-pivot-data"></a>
### Belongs to Many (with Pivot Data)

> **Note**: Pivot data is not supported by [deferred bindings](../database/relations.md#oc-deferred-binding) at this time, so the parent model should exist. If your relation behavior config has `deferredBinding: true`, the pivot data will **not** be available to use in the list configuration (ex.`pivot[attribute]`).

1. Related records are displayed as a list (**view.list**).
1. Clicking a record will display an update form (**pivot.form**).
1. Clicking *Add* will display a selection list (**manage.list**), then a data entry form (**pivot.form**).
1. Clicking *Remove* will destroy the pivot table record(s).

Continuing the example in *Belongs To Many* relations, if a role also carried an expiry date, clicking on a role will open a popup form with the fields defined in **pivot** to update the expiry date. Below is an example of the relation behavior configuration file:

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

roles:
    label: Role
    view:
        list: $/acme/user/models/role/columns.yaml
    manage:
        list: $/acme/user/models/role/columns.yaml
    pivot:
        form: $/acme/user/models/role/fields.yaml
```

Pivot data is available when defining form fields and list columns via the `pivot` relation, see the example below:

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

teams:
    label: Team
    view:
        list:
            columns:
                name:
                    label: Name
                pivot[team_color]:
                    label: Team color
    manage:
        list:
            columns:
                name:
                    label: Name
    pivot:
        form:
            fields:
                pivot[team_color]:
                    label: Team color
```

### Belongs To

1. Related record is displayed as a preview form (**view.form**).
1. Clicking *Create* will display a create form (**manage.form**).
1. Clicking *Update* will display an update form (**manage.form**).
1. Clicking *Link* will display a selection list (**manage.list**).
1. Clicking *Unlink* will orphan the relationship.
1. Clicking *Delete* will destroy the record.

For example, if a *Phone* belongs to a *Person* the relation manager will display a form with the fields defined in **form**. Clicking the Link button will display a list of People to associate with the Phone. Clicking the Unlink button will dissociate the Phone with the Person.

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

person:
    label: Person
    view:
        form: $/acme/user/models/person/fields.yaml
        toolbarButtons: link|unlink
    manage:
        form: $/acme/user/models/person/fields.yaml
        list: $/acme/user/models/person/columns.yaml
```

### Has One

1. Related record is displayed as a preview form (**view.form**).
1. Clicking *Create* will display a create form (**manage.form**).
1. Clicking *Update* will display an update form (**manage.form**).
1. Clicking *Link* will display a selection list (**manage.list**).
1. Clicking *Unlink* will orphan the relationship.
1. Clicking *Delete* will destroy the record.

For example, if a *Person* has one *Phone* the relation manager will display form with the fields defined in **form** for the Phone. When clicking the Update button, a popup is displayed with the fields now editable. If the Person already has a Phone the fields are update, otherwise a new Phone is created for them.

```yaml
# ===================================
#  Relation Behavior Config
# ===================================

phone:
    label: Phone
    view:
        form: $/acme/user/models/phone/fields.yaml
        toolbarButtons: update|delete
    manage:
        form: $/acme/user/models/phone/fields.yaml
        list: $/acme/user/models/phone/columns.yaml
```

## Displaying a Relation Manager

Before relations can be managed on any page, the target model must first be initialized in the controller by calling the `initRelation` method.

```php
$post = Post::where('id', 7)->first();
$this->initRelation($post);
```

> **Note**: The [form behavior](forms.md) will automatically initialize the model on its create, update and preview actions.

The relation manager can then be displayed for a specified relation definition by calling the `relationRender` method. For example, if you want to display the relation manager on the [Preview](forms.md#oc-preview-view) page, the **preview.htm** view contents could look like this:

```php
<?= $this->formRenderPreview() ?>

<?= $this->relationRender('comments') ?>
```

You may instruct the relation manager to render in read only mode by passing the option as the second argument:

```php
<?= $this->relationRender('comments', ['readOnly' => true]) ?>
```

## Extending Relation Behavior

Sometimes you may wish to modify the default relation behavior and there are several ways you can do this.

### Extending Relation Configuration

Provides an opportunity to manipulate the relation configuration. The following example can be used to inject a different columns.yaml file based on a property of your model.

```php
public function relationExtendConfig($config, $field, $model)
{
    // Make sure the model and field matches those you want to manipulate
    if (!$model instanceof MyModel || $field != 'myField') {
        return;
    }

    // Show a different list for business customers
    if ($model->mode == 'b2b') {
        $config->view['list'] = '$/author/plugin_name/models/mymodel/b2b_columns.yaml';
    }
}
```

### Extending the View Widget

Provides an opportunity to manipulate the view widget. For example you might want to toggle showCheckboxes based on a property of your model.

```php
public function relationExtendViewWidget($widget, $field, $model)
{
    // Make sure the model and field matches those you want to manipulate
    if (!$model instanceof MyModel || $field != 'myField') {
        return;
    }

    if ($model->constant) {
        $widget->showCheckboxes = false;
    }
}
```

#### How to Remove a Column

Since the widget has not completed initializing at this point of the runtime cycle you can't call `$widget->removeColumn()`. The `addColumns()` method as described in the [ListController documentation](../backend/lists.md#oc-extending-column-definitions) will work as expected, but to remove a column we need to listen to the 'list.extendColumns' event within the `relationExtendViewWidget()` method. The following example shows how to remove a column.

```php
public function relationExtendViewWidget($widget, $field, $model)
{
    // Make sure the model and field matches those you want to manipulate
    if (!$model instanceof MyModel || $field != 'myField') {
        return;
    }

    // This will work
    $widget->bindEvent('list.extendColumns', function () use ($widget) {
        $widget->removeColumn('my_column');
    });
}
```

### Extending the Manage Widget

Provides an opportunity to manipulate the manage widget of your relation.

```php
public function relationExtendManageWidget($widget, $field, $model)
{
    // Make sure the field is the expected one
    if ($field != 'myField') {
        return;
    }

    // Manipulate widget as needed
}
```

### Extending the Pivot Widget

Provides an opportunity to manipulate the pivot widget of your relation.

```php
public function relationExtendPivotWidget($widget, $field, $model)
{
    // Make sure the field is the expected one
    if ($field != 'myField') {
        return;
    }

    // Manipulate widget as needed
}
```

### Extending the Filter Widgets

There are two filter widgets that may be extended using the following methods, one for the view mode and one for the manage mode of the `RelationController`.

```php
public function relationExtendViewFilterWidget($widget, $field, $model)
{
    // Extends the view filter widget
}

public function relationExtendManageFilterWidget($widget, $field, $model)
{
    // Extends the manage filter widget
}
```

Examples on how to add or remove scopes programmatically in the filter widgets can be found in the **Extending filter scopes** section of the [Backend list documentation](../backend/lists.md#oc-extending-filter-scopes).

### Extending the Refresh Results

The view widget is often refreshed when the manage widget makes a change, you can use this method to inject additional containers when this process occurs. Return an array with the extra values to send to the browser, eg:

```php
public function relationExtendRefreshResults($field)
{
    // Make sure the field is the expected one
    if ($field != 'myField') {
        return;
    }

    return ['#myCounter' => 'Total records: 6'];
}
```
