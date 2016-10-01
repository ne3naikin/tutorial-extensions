# Домашнє завдання: додамо більше на ваш сайт!

Наш блог виконав довгий шлях, але є ще можливості для вдосконалення. Далі, ми будемо додавати нові функції для постів - чернетки та їх публікацію. Ми також додамо видалення повідомлень, які нам більше не потрібні. Чудово!

## Зберегати нові повідомлення як чернетки

В даний час, коли ми створюємо нові повідомлення за допомогою нашої *New post (Новий пост)* форми, пост публікується відразу. Для того, щоб зберегти повідомлення як чернетку, **видаліть** цей рядок в  `blog/views.py` в `post_new` і `post_edit` методів:

```python
post.published_date = timezone.now()
```

Таким чином, нові повідомлення будуть збережені як чернетки, які ми можемо розглянути пізніше, а не миттєво опубліковати. Все, що нам зараз потрібно, це шлях до списка де публікуются чернетоки, давайте зробемо це!

## Сторінка зі списком неопублікованих постів

Пам'ятайте розділ про множинні запити? Ми створили подання `post_list`, який відображає лише опубліковані записи в блогах (тих, хто з непустих `published_date`).

Час, щоб зробити щось подібне, але для чорнових постів.

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

Цей рядок `Post.objects.filter(published_date__isnull=True).order_by('created_date')` гарантує, що ми приймаємо тільки неопубліковані повідомлення (`published_date__isnull=True`) згідно з їх замовленням доводити до ладу and order them by `created_date` (`order_by('created_date')`).

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

## Add publish button

It would be nice to have a button on the blog post detail page that will immediately publish the post, right?

Let's open `blog/template/blog/post_detail.html` and change these lines:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% endif %}
```

into these:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% else %}
    <a class="btn btn-default" href="{% url 'blog.views.post_publish' pk=post.pk %}">Publish</a>
{% endif %}
```

As you noticed, we added `{% else %}` line here. That means, that if the condition from `{% if post.published_date %}` is not fulfilled (so if there is no `published_date`), then we want to do the line `<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>`. Note that we are passing a `pk` variable in the `{% url %}`.

Time to create a URL (in `blog/urls.py`):

```python
url(r'^post/(?P<pk>\d+)/publish/$', views.post_publish, name='post_publish'),
```

and finally, a *view* (as always, in `blog/views.py`):

```python
def post_publish(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.publish()
    return redirect('blog.views.post_detail', pk=pk)
```

Remember, when we created a `Post` model we wrote a method `publish`. It looked like this:

```python
def publish(self):
    self.published_date = timezone.now()
    self.save()
```

Now we can finally use this!

And once again after publishing the post we are immediately redirected to the `post_detail` page!

![Publish button](images/publish2.png)

Congratulations! You are almost there. The last step is adding a delete button!

## Delete post

Let's open `blog/templates/blog/post_detail.html` once again and add this line:

```django
<a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

just under a line with the edit button.

Now we need a URL (`blog/urls.py`):

```python
url(r'^post/(?P<pk>\d+)/remove/$', views.post_remove, name='post_remove'),
```

Now, time for a view! Open `blog/views.py` and add this code:

```python
def post_remove(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('blog.views.post_list')
```

The only new thing is to actually delete a blog post. Every Django model can be deleted by `.delete()`. It is as simple as that!

And this time, after deleting a post we want to go to the webpage with a list of posts, so we are using `redirect`.

Let's test it! Go to the page with a post and try to delete it!

![Delete button](images/delete3.png)

Yes, this is the last thing! You completed this tutorial! You are awesome!

