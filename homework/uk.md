# Домашнє завдання: додамо більше на ваш сайт!

Наш блог пройшов довгий шлях, але є ще можливості для вдосконалення. Далі, ми будемо додавати нові функції для постів - чернетки та їх публікацію. Ми також додамо можливість видалення не потрібних нам більше повідомлень. Чудово!

## Зберегати нові повідомлення як чернетки

В даний час, коли ми створюємо нові повідомлення за допомогою нашої *New post (Новий пост)* форми, пост публікується відразу. Для того, щоб зберегти повідомлення як чернетку, **видаліть** цей рядок в  `blog/views.py` в `post_new` і `post_edit` методів:

```python
post.published_date = timezone.now()
```

Таким чином, нові повідомлення будуть збережені як чернетки, які ми можемо розглянути пізніше, а не миттєво опубліковати. Все, що нам зараз потрібно, це шлях до списка де публікуются чернетки, давайте зробемо це!

## Сторінка зі списком неопублікованих постів

Пам'ятаете розділ про множинні запити? Ми створили подання `post_list`, який відображає лише опубліковані записи в блогах (тих, що не є пустими `published_date`).

Час зробити щось подібне, але для чорнових постів.

Давайте додамо посилання у `blog/templates/blog/base.html` біля кнопки для додавання нових постів (трохи вище `<h1><a href="/">Django Girls Blog</a></h1>` рядка!):

```django
<a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
```

Далі: посилання! У `blog/urls.py` ми добавим:

```python
url(r'^drafts/$', views.post_draft_list, name='post_draft_list'),
```

Тепер настав час, створити їого відображення у `blog/views.py`:

```python
def post_draft_list(request):
    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
    return render(request, 'blog/post_draft_list.html', {'posts': posts})
```

Цей рядок `Post.objects.filter(published_date__isnull=True).order_by('created_date')` гарантує, що ми прийматемемо тільки неопубліковані повідомлення (`published_date__isnull=True`) та у порядку їх надходження `created_date` (`order_by('created_date')`).

Добре, останній крок, звичайно, шаблон! створіть файл `blog/templates/blog/post_draft_list.html` та додайте наступні рядки:

```django
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <div class="post">
            <p class="date">created: {{ post.created_date|date:'d-m-Y' }}</p>
            <h1><a href="{% url 'blog.views.post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
            <p>{{ post.text|truncatechars:200 }}</p>
        </div>
    {% endfor %}
{% endblock %}
```

Він дуже схожий на наш `post_list.html`, чи не так? 

Тепер коли ви перейдете по посиланню `http://127.0.0.1:8000/drafts/` ви побачете список неопублікованих постив.

Прекрасно! Ваша перша задача виконана!

## Додамо кнопку публікації

Було б непогано мати кнопку на сторінці, в блозі поста, елемент, який дозволить негайно опублікувати пост, чи не так?

Давайте відкриємо `blog/template/blog/post_detail.html` та змінимо ці рядки:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% endif %}
```

на:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% else %}
    <a class="btn btn-default" href="{% url 'blog.views.post_publish' pk=post.pk %}">Publish</a>
{% endif %}
```

Як ви помітили, ми додали тут рядок `{% else %}`. Це означає, що якщо умова з `{% if post.published_date %}` не виконується (тобто, якщо немає `published_date`), то ми хочемо щьоб виконувася рядок `<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>`. Зверніть увагу, що ми передаємо `pk` змінну в `{% url %}`.

Час для створення URL - адреси ресурсу (у `blog/urls.py`):

```python
url(r'^post/(?P<pk>\d+)/publish/$', views.post_publish, name='post_publish'),
```

і, нарешті, *view (вид)* (як завжди, у `blog/views.py`):

```python
def post_publish(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.publish()
    return redirect('blog.views.post_detail', pk=pk)
```

Пам'ятаєте, колись ми створили модель `Post` у яку ми написали метод `publish`. Виглядало це так:

```python
def publish(self):
    self.published_date = timezone.now()
    self.save()
```

Тепер ми можемо нарешті використати це!

І ще одне, після публікації повідоилення ми негайно перенаправемось до сторінки `post_detail`!

![Publish button](images/publish2.png)

Вітаємо! Ви майже підійшли до кінця. Останній крок, додамо кнопку видалення!

## Видалити запис

Давайте відкриємо ще раз `blog/templates/blog/post_detail.html` і додамо наступний рядок:

```django
<a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

як раз під рядком біля кнопки редагування.

Тепер нам потрібен URL (`blog/urls.py`):

```python
url(r'^post/(?P<pk>\d+)/remove/$', views.post_remove, name='post_remove'),
```

Тепер, час для файлу view! Відкриемо `blog/views.py` та додамо цей код:

```python
def post_remove(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('blog.views.post_list')
```

Де єдина нова річ, це рядок видалення у блозі повідомлення. Кожна модель Django може бути видалена за допомоги `.delete()`. Це простіше нікуди!

Та на цей раз, після видалення повідомлення ми хочемо, щоб був перехід на веб-сторінку зі списком повідомлень, тому ми використовуємо `redirect`.

Давайте перевіримо! Перейдіть на сторінку з повідомленням і спробуйте видалити його!

![Delete button](images/delete3.png)

Так, це все! Ви закінчили цей урок! Ви чудові!

