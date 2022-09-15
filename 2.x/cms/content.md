# Content Blocks

Content Blocks can be text, HTML or [Markdown](http://daringfireball.net/projects/markdown/syntax) blocks that are edited separately from the page or layout. They're designed to hold static content only and support basic templating variables. [Partials](partials.md) are more flexible and should be used for generating dynamic content.

## Introduction

Content blocks files reside in the **/content** subdirectory of a theme directory. The following extensions are supported for content files.

Extension | Description
------------- | -------------
**htm** | Used for HTML markup.
**txt** | Used for plain text.
**md** | Used for Markdown syntax.

The extension affects a content block's display mode in the back-end user interface, either with a WYSIWYG editor, plain text editor or markdown editor. It also determines rendering the blocks on the website; for example, Markdown blocks will convert to HTML before display.

## Rendering Content Blocks

Use the `{% content 'file.htm' %}` tag to render a content block in a [page](pages.md), [partial](partials.md) or [layout](layouts.md).

This example shows a complete page rendering a content block.

```twig
url = "/contacts"
==
<div class="contacts">
    {% content 'contacts.htm' %}
</div>
```

Another example rendering some markdown with the `md` extension.

```twig
{% content 'my-markdown.md' %}
```

<a id="oc-passing-variables-to-content-blocks"></a>
## Passing Variables to Content Blocks

Sometimes you may need to pass variables to a content block from the external code. While content blocks do not support Twig markup, they do support using variables with basic syntax. You can pass variables to content blocks by specifying them after the content block name in the `{% content %}` tag.

Passing the variable called `name` with a value **John** to the content block.

```twig
{% content 'welcome.htm' name='John' %}
```

Inside the content block, variables can be accessed using singular *curly brackets*.

```
<h1>This is a demo for {name}</h1>
```

> **Tip**: More information on variable use can be found [in the Markup guide](../markup/tag-content.md).

### Global Variables

You may register variables that are globally available to all content blocks with the `View::share` method.

```php
View::share('site_name', 'October CMS');
```

A common area to place this method is inside the register or boot method of a [plugin registration file](../plugin/registration.md). Using the above example, the variable `{site_name}` will be available inside all content blocks.

```
<p>Welcome to {site_name}</p>
```
