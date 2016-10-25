# Домашнє завдання: Додайте безпеки на свій веб-сайт

Ви могли помітити, що не використовуете свій пароль, за винятком тих випадків, коли ми використовували інтерфейс адміністратора. Зауважимо, що це означає, що будь-хто може додавати або редагувати повідомлення до вашого блогу. Я не знаю як ви, але я не хочу, щьоб хтось розміщував пости в моєму блозі. Так що давайте робити щось з цим.

## Авторизація для додавання або редагування повідомлень

По-перше довайте зробимо речі більш безпечнимшими. Ми будемо захищати наші перегляди, такі як `post_new`, `post_edit`, `post_draft_list`, `post_remove` і `post_publish`, таким чином, що отримати доступ до них можуть тільки ті користувачі що увійшли. Django поставляється з деякими хорошими помічниками для цього, такі як додаткові теми, _decorators_. Не будем зараз вдоватись у технічні деталі, про них ви зможете прочитати де що пізніше. Декоратор користувача размишенний в Django у модулі `django.contrib.auth.decorators` і називается `login_required`.

Так відредагуйте ваш  `blog/views.py` і додайте ці рядки до верхньої частині, разом з іншою частиною імпорту:  

```python
from django.contrib.auth.decorators import login_required
```

Потім додайте перед кожним відображенням `post_new`, `post_edit`, `post_draft_list`, `post_remove` та `post_publish` наступні рядки

```python
@login_required
def post_new(request):
    [...]
```
Ось воно! Тепер спробуйте отримати доступ до `http://localhost:8000/post/new/`, помічаєте різницю?

> Якщо ви тільки що отримали порожню форму, ви напевно увійшли у систему з головного інтерфейсу адміністратора. Перейти до `http://localhost:8000/admin/logout/`, щоб вийти з облікового запису, а потім ввійдіть ще раз `http://localhost:8000/post/new`.

Ви повинні отримати одну з улюблених помилок. Це - досить цікавий факт: додавши декоратор, ми спочатку будемо потрапляти на сторінку входу в систему. Але на разі це ще не доступно, такщо ми отрімаемо лише на старинку з написом "Сторінку не знайдено (404)"/"Page not found (404)". 

Не забудьте додати декоратор зверху також до `post_edit`, `post_remove`, `post_draft_list` і `post_publish`.

Чудово, ми досягли частково мети! Інші люди не зможуть більше просто створювати повідомлення на нашому блозі. На жаль, ми теж більше не можемо створити повідомлення. Так що давайте виправимо це.


## Login users Вхід користувачів

Now we could try to do lots of magic stuff to implement users and passwords and authentication but doing this kind of stuff correctly is rather complicated. As Django is "batteries included", someone has done the hard work for us, so we will make further use of the authentication stuff provided.
Тепер ми могли б спробувати зробити багато магічних штучок для реалізації користувачів і паролів і аутентифікації, але робити такого роду речі правильно досить складний. Як Django є "включені батарейки", хтось зробив важку роботу для нас, тому ми зробимо подальше використання матеріалу аутентифікації за умови.

In your `mysite/urls.py` add a url `url(r'^accounts/login/$', django.contrib.auth.views.login)`. So the file should now look similar to this:
У вашому файлі `MySite / urls.py` додати URL-адрес` URL (ґ ^ рахунки / Увійти / $ ', django.contrib.auth.views.login)`. Таким чином, файл повинен виглядати приблизно так:

```python
from django.conf.urls import include, url
import django.contrib.auth.views

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', django.contrib.auth.views.login, name='login'),
    url(r'', include('blog.urls')),
)
```

Then we need a template for the login page, so create a directory `blog/templates/registration` and a file inside named `login.html`:
Тоді нам потрібен шаблон для сторінки входу в систему, так що створити каталог `блог / шаблони / registration` і файл з ім'ям внутрі` login.html`:

```django
{% extends "blog/base.html" %}

{% block content %}

{% if form.errors %}
<p>Your username and password didn't match. Please try again.</p>
{% endif %}

<form method="post" action="{% url 'login' %}">
{% csrf_token %}
<table>
<tr>
    <td>{{ form.username.label_tag }}</td>
    <td>{{ form.username }}</td>
</tr>
<tr>
    <td>{{ form.password.label_tag }}</td>
    <td>{{ form.password }}</td>
</tr>
</table>

<input type="submit" value="login" />
<input type="hidden" name="next" value="{{ next }}" />
</form>

{% endblock %}
```

You will see that this also makes use of our base-template for the overall look and feel of your blog.
Ви побачите, що це також робить використання нашого базового шаблону для загальний зовнішній вигляд вашого блогу.

The nice thing here is that this _just works[TM]_. We don't have to deal with handling of the forms submission nor with passwords and securing them. Only one thing is left here, we should add a setting to `mysite/settings.py`:
Хороша річ тут є те, що це працює _just [TM] _. Ми не повинні мати справу з обробкою уявлення форм, ні з паролями і забезпечення їх. Тільки одна річ залишається тут, ми повинні додати параметр в `MySite / settings.py`:

```python
LOGIN_REDIRECT_URL = '/'
```

Now when the login is accessed directly, it will redirect successful login to the top level index.
Тепер, коли Логін здійснюється безпосередньо, він буде перенаправляти успішної реєстрації на індекс верхнього рівня.

## Improving the layout
## Поліпшення макета

So now we made sure that only authorized users (ie. us) can add, edit or publish posts. But still everyone gets to view the buttons to add or edit posts, lets hide these for users that aren't logged in. For this we need to edit the templates, so lets start with the base template from `blog/templates/blog/base.html`:
Так що тепер ми переконалися, що тільки авторизовані користувачі (тобто. Нас) можна додавати, редагувати або публікувати повідомлення. Але все-таки кожен отримує, щоб переглянути кнопки для додавання або редагування повідомлення, дозволяє приховати їх для користувачів, які не ввійшли в систему. Для цього потрібно відредагувати шаблони, так що давайте почнемо з базового шаблону з `блогу / шаблони / блог / base.html`:

```django
<body>
    <div class="page-header">
        {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
        <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        {% else %}
        <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
        {% endif %}
        <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
    </div>
    <div class="content">
        <div class="row">
            <div class="col-md-8">
            {% block content %}
            {% endblock %}
            </div>
        </div>
    </div>
</body>
```

You might recognize the pattern here. There is an if-condition inside the template that checks for authenticated users to show the edit buttons. Otherwise it shows a login button.
Ви могли б визнати картину тут. Існує Умовний стан усередині шаблону, який перевіряє наявність пройшли перевірку автентичності користувачів, щоб показати кнопки редагування. В іншому випадку він показує кнопку входу.

*Homework*: Edit the template `blog/templates/blog/post_detail.html` to only show the edit buttons for authenticated users.
* Домашнє завдання *: Редагувати шаблон `блог / шаблони / блог / post_detail.html`, щоб показувати тільки кнопки редагування для авторизованих користувачів.

## More on authenticated users
## Додаткова інформація по користувачам, які пройшли перевірку

Lets add some nice sugar to our templates while we are at it. First we will add some stuff to show that we are logged in. Edit `blog/templates/blog/base.html` like this:
Давайте додамо деякий хороший цукор наших шаблонів, поки ми на нього. По-перше, ми додамо деякі речі, щоб показати, що ми увійшли в Edit `Блог / шаблони / блог / base.html` як це .:

```django
<div class="page-header">
    {% if user.is_authenticated %}
    <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
    <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
    <p class="top-menu">Hello {{ user.username }}<small>(<a href="{% url 'logout' %}">Log out</a>)</small></p>
    {% else %}
    <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
    <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
</div>
```

This adds a nice "Hello &lt;username&gt;" to remind us who we are and that we are authenticated. Also this adds a link to log out of the blog. But as you might notice this isn't working yet. Oh nooz, we broke the internetz! Lets fix it!
Це додає хороший "Hello & лт; ім'я користувача & GТ;" щоб нагадати нам, хто ми і що ми ідентифікуються. Крім того, це додає посилання, щоб вийти з блогу. Але, як ви могли помітити, це ще не працює. Про nooz, ми зламали internetz! Дозволяє виправити!

We decided to rely on django to handle login, lets see if Django can also handle logout for us. Check https://docs.djangoproject.com/en/1.8/topics/auth/default/ and see if you find something.
Ми вирішили покладатися на Джанго обробляти логін, дозволяє побачити, якщо Django може також обробляти вихід з системи для нас. Перевірте https://docs.djangoproject.com/en/1.8/topics/auth/default/ і подивитися, якщо ви знайдете щось.

Done reading? You should by now think about adding a url (in `mysite/urls.py`) pointing to the `django.contrib.auth.views.logout` view. Like this:
Вчинено читання? Ви повинні зараз думати про додавання URL (в `MySite / urls.py`), який вказує на 'django.contrib.auth.views.logout` зору. Подобається це:

```python
from django.conf.urls import include, url, patterns
from django.contrib.auth import views

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', views.login, name='login'),
    url(r'^accounts/logout/$', views.logout, name='logout', kwargs={'next_page': '/'}),
    url(r'', include('blog.urls')),
)
```

Thats it! If you followed all of the above until this point (and did the homework), you now have a blog where you

 - need a username and password to log in,
 - need to be logged in to add/edit/publish(/delete) posts
 - and can log out again

Це воно! Якщо ви виконали всі вище до цього моменту (і зробив домашнє завдання), тепер у вас є блог, де ви

 - Потрібно ввести ім'я користувача і пароль, щоб увійти,
 - Потрібно увійти в систему, щоб додавати / редагувати / публікувати (/ видалення) повідомлення
 - І може вийти знову
