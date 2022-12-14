# {% for %}

Теги `{% for %}` и `{% endfor %}` используются для перебирания значений в коллекции. При этом коллекция может быть как массивами, так и объектом.

```twig
<ul>
    {% for user in users %}
        <li>{{ user.username }}</li>
    {% endfor %}
</ul>
```

Вы также можете получить ключи и значения:

```twig
<ul>
    {% for key, user in users %}
        <li>{{ key }}: {{ user.username }}</li>
    {% endfor %}
</ul>
```

Вы можете использовать `else`, чтобы отобразить произвольный код, когда в коллекции нет значений:

```twig
<ul>
    {% for user in users %}
        <li>{{ user.username }}</li>
    {% else %}
        <li><em>There are no users found</em></li>
    {% endfor %}
</ul>
```

## Итерация

Если Вам нужно пройтись по всей коллекции с определенным количеством значений, то Вы можете использовать `..`:

```twig
{% for i in 0..10 %}
    - {{ i }}
{% endfor %}
```

Тогда на страница отобразятся числа от 0 до 10.

Вы также можете использовать буквы вместо цифр:

```twig
{% for letter in 'a'..'z' %}
    - {{ letter }}
{% endfor %}

{% for letter in 'a'|upper..'z'|upper %}
    - {{ letter }}
{% endfor %}
```

## Условия

К сожалению, Вы не можете использовать  `break` или `continue` в циклах. Однако Вы можете сначала отфильтровать коллекцию. Пример:

```twig
<ul>
    {% for user in users if user.active %}
        <li>{{ user.username }}</li>
    {% endfor %}
</ul>
```

## Переменная `loop`

Внутри цикла Вы можете использовать следующие переменные:

Переменная | Описание
------------- | -------------
`loop.index` | Номер текущий итерации. (начинается с 1)
`loop.index0` | Номер текущий итерации. (начинается с 0)
`loop.revindex` | Номер текущий итерации с конца (начинается с 1)
`loop.revindex0` | Номер текущий итерации с конца (начинается с 0)
`loop.first` | Возвращает `true`, если первая итерация
`loop.last` |  Возвращает `true`, если последняя итерация
`loop.length` | Количество элементов в коллекции
`loop.parent` | Родительский контекст

```twig
{% for user in users %}
    {{ loop.index }} - {{ user.username }}
{% endfor %}
```
