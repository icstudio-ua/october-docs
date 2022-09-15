# |media

The `|media` filter returns an address relative to the public path of the [media manager library](../cms/mediamanager.md). The result is a URL to the media file specified in the filter parameter.

```twig
<img src="{{ 'banner.jpg'|media }}" />
```

If the media manager address is __https://cdn.octobercms.com__ the above example would output the following:

```twig
<img src="https://cdn.octobercms.com/banner.jpg" />
```
