---
subtitle: Make a global blueprint available to the page or layout.
---
# Global

The `global` component makes a global record available to the page, layout or partial. The global component can be used on any page, layout or partial.

## Available Properties

The following properties are supported by the component.

Property | Description
-------- | -------------
**handle** | The handle of the [global blueprint](../blueprints/global.md).

## Basic Usage

The following adds a global using the handle **Blog\Config** to the page. The component is assigned a name of **blogConfig** using the component alias and this is the variable name that becomes available to the page. The page access the `about_this_blog` text field and displays it on the page.

```ini
[global blogConfig]
handle = "Blog\Config"
==
<p>{{ blogConfig.about_this_blog }}</p>
```
