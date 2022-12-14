# abort()

`abort()` 函数通过更改响应代码和内容来中止成功的请求路径。 这对于设置自定义 HTTP 代码或在未找到记录时显示 404 页面很有用。

```twig
{% if record.notFound %}
    {% do abort(404) %}
{% endif %}
```

要设置响应代码并显示主题 404 页面，请使用`404`代码。

```twig
{% do abort(404) %}
```

任何其他代码都将显示错误页面，其中第二个参数是可选消息。

```twig
{% do abort(403, '拒绝访问') %}
```

要在标头中设置 HTTP 代码而不更改响应内容，请将 `false` 作为第二个参数传递。

```twig
{% do abort(404, false) %}
```
