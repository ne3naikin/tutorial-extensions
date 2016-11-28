# Домашнє завдання: створить модель коментарів

Нині ми маємо тільки модель повідомлень (модель постів). А що ви можете сказати, якщо будуть надходити відгуки від ваших читачів через їх коментарі?

## Створення моделі коментарів у блозі

Нумо відкриймо `blog/models.py` і додамо цей шматок коду в кінець файлу:

```python
class Comment(models.Model):
    post = models.ForeignKey('blog.Post', related_name='comments')
    author = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    approved_comment = models.BooleanField(default=False)

    def approve(self):
        self.approved_comment = True
        self.save()

    def __str__(self):
        return self.text
```

Ви можете повернутися до глави **Django models** в підручнику, якщо вам потрібно освіжити в пам'яті, що означає кожен з цих типів полів.

У цьому розширенні з підручника ми маємо новий тип поля:
- `models.BooleanField` - це поле істина/брехня.

Елемент `related_name` в `models.ForeignKey` дозволяє нам мати доступ до коментарів всередині моделі постів.

## Створення таблиць для моделей в базі даних

Тепер прийшов час, щоб додати нашу модель коментарів в базу даних. Для цього ми повинні сказати Django, що ми внесли зміни в нашу модель. Для цього введемо в командному рядку такий тип рядка `python manage.py makemigrations blog`. Ви повинні побачити, таке:

    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migrations for 'blog':
      0002_comment.py:
        - Create model Comment

Ви можете бачити, що ця команда створила інший файл міграції для нас в директорії `blog/migrations/`. Тепер нам потрібно застосувати ці зміни, запровадивши `python manage.py migrate blog` в командному рядку. На виході повинні побачити це: 

```
    (myvenv) ~/djangogirls$ python manage.py migrate blog
    Operations to perform:
      Apply all migrations: blog
    Running migrations:
      Rendering model states... DONE
      Applying blog.0002_comment... OK
```

Наша модель *коментарів* вже існує в базі даних! А не було б це добре, якби ми мали доступ до неї в нашій панелі адміністратора?

## Реєстрація моделі коментарів в панелі адміністратора

Для того, щоб зареєструвати модель коментарів в панелі адміністратора, перейдіть в `blog/admin.py` і додайте наступний рядок:

```python
admin.site.register(Comment)
```

Безпосередньо під цією лінією:

```python
admin.site.register(Post)
```

Також не забудьте імпортувати модель коментарів у верхній частині файлу, як це:

```python
from django.contrib import admin
from .models import Post, Comment

admin.site.register(Post)
admin.site.register(Comment)
```

Якщо ви наберете `python manage.py runserver` в командному рядку і перейдете до [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) в вашому браузері, ви повинні отримати доступ до списку коментарів, а також можливість додавати й видаляти коментарі. Пограйте з новою функцією коментарів!

## Зробимо наші коментарі видимими

Перейти до `blog/templates/blog/post_detail.html` і додайте наступні рядки перед тегом `{% endblock %}`:

```django
<hr>
{% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

Тепер ми можемо побачити розділ коментарів на сторінках як частину посту.

Але це все може виглядати трохи краще, так що додамо трохи CSS до нижньої частини у файлі `static/css/blog.css`:

```css
.comment {
    margin: 20px 0px 20px 20px;
}
```

Ми також можемо дозволити відвідувачам ознайомитесь з коментарями на сторінці постів. Перейти до `blog/templates/blog/post_list.html` і додайте рядок:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

Після цього наш шаблон повинен виглядати наступним чином:

```django
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <div class="post">
            <div class="date">
                {{ post.published_date }}
            </div>
            <h1><a href="{% url 'blog.views.post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
            <p>{{ post.text|linebreaks }}</p>
            <a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
        </div>
    {% endfor %}
{% endblock content %}
```

## Нехай ваші читачі пишуть коментарі

Зараз ми можемо бачити коментарі на нашому блозі, але ми не можемо додати їх. Давайте змінимо це!

Перейти до `blog/forms.py` і додайте наступні рядки в кінець файлу:

```python
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = ('author', 'text',)
```

Не забудьте імпортувати модель *коментарив*, змінивши рядок:

```python
from .models import Post
```

на:

```python
from .models import Post, Comment
```

Тепер перейдіть в `blog/templates/blog/post_detail.html` і перед строкой `{% for comment in post.comments.all %}`, додайте:

```django
<a class="btn btn-default" href="{% url 'add_comment_to_post' pk=post.pk %}">Add comment</a>
```

Якщо ви зайдете на сторінку посту ви повинні побачити цю помилку:

![NoReverseMatch](images/url_error.png)

Ми знаємо, як це виправити! Перейти до `blog/urls.py` і додати цей шаблон у `urlpatterns`:

```python
url(r'^post/(?P<pk>\d+)/comment/$', views.add_comment_to_post, name='add_comment_to_post'),
```

Оновимо сторінку, і ми отримуємо іншу помилку!

![AttributeError](images/views_error.png)

To fix this error, add this view to `blog/views.py`:
Щоб виправити цю помилку, додайте цей вид `blog/views.py`:

```python
def add_comment_to_post(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.post = post
            comment.save()
            return redirect('blog.views.post_detail', pk=post.pk)
    else:
        form = CommentForm()
    return render(request, 'blog/add_comment_to_post.html', {'form': form})
```

Remember to import `CommentForm` at the beginning of the file:
Не забудьте імпортувати `CommentForm` на початку файлу:

```python
from .forms import PostForm, CommentForm
```


Now, on the post detail page, you should see the "Add Comment" button.
Тепер на сторінку записи, ви повинні побачити кнопку "Додати коментар".

![AddComment](images/add_comment_button.png)

However, when you click that button, you'll see:
Проте, при натисканні на цю кнопку, ви побачите:

![TemplateDoesNotExist](images/template_error.png)


Like the error tells us, the template doesn't exist yet. So, let's create a new one at `blog/templates/blog/add_comment_to_post.html` and add the following code:
Як помилка говорить нам, шаблон ще не існує. Отже, давайте створимо нову в `blog/templates/blog/add_comment_to_post.html` і додайте наступний код:

```django
{% extends 'blog/base.html' %}

{% block content %}
    <h1>New comment</h1>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Send</button>
    </form>
{% endblock %}
```

Yay! Now your readers can let you know what they think of your blog posts!
Yay! Тепер ваші читачі можуть повідомити вам, що вони думають про ваших постів в блозі!

## Moderating your commentsМодерація ваші коментарі

Not all of the comments should be displayed. As the blog owner, you probably want the option to approve or delete comments. Let's do something about it.
Не всі коментарі повинні бути відображені. Як власник блогу, ви, ймовірно, хочете можливість схвалити або видаляти коментарі. Давайте робити щось про це.

Go to `blog/templates/blog/post_detail.html` and change lines:
Перейти до `blog/templates/blog/post_detail.html` і зміни ліній:

```django
{% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

to:
щоб:

```django
{% for comment in post.comments.all %}
    {% if user.is_authenticated or comment.approved_comment %}
    <div class="comment">
        <div class="date">
            {{ comment.created_date }}
            {% if not comment.approved_comment %}
                <a class="btn btn-default" href="{% url 'comment_remove' pk=comment.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
                <a class="btn btn-default" href="{% url 'comment_approve' pk=comment.pk %}"><span class="glyphicon glyphicon-ok"></span></a>
            {% endif %}
        </div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
    {% endif %}
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

You should see `NoReverseMatch`, because no URL matches the `comment_remove` and `comment_approve` patterns... yet!
Ви повинні побачити `NoReverseMatch`, тому що ні один URL-адреса не соответствует `comment_remove` і `comment_approve` візерунки ... ще!

To fix the error, add these URL patterns to `blog/urls.py`:
Щоб виправити цю помилку, додайте ці шаблони URL в `блог / urls.py`:

```python
url(r'^comment/(?P<pk>\d+)/approve/$', views.comment_approve, name='comment_approve'),
url(r'^comment/(?P<pk>\d+)/remove/$', views.comment_remove, name='comment_remove'),
```

Now, you should see `AttributeError`. To fix this error, add these views in `blog/views.py`:
Тепер ви повинні побачити `AttributeError`. Щоб виправити цю помилку, додайте ці погляди в `blog/views.py`:

```python
@login_required
def comment_approve(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    comment.approve()
    return redirect('blog.views.post_detail', pk=comment.post.pk)

@login_required
def comment_remove(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    post_pk = comment.post.pk
    comment.delete()
    return redirect('blog.views.post_detail', pk=post_pk)
```

You'll also need to import `login_required` at the beginning of the file:
Вам також необхідно імпортувати `login_required` на початку файлу:

```python
from django.contrib.auth.decorators import login_required
```

And of course, remember to import `Comment` at the top of the file:
І, звичайно ж, не забудьте імпортувати `Comment` у верхній частині файлу:

```python
from .models import Post, Comment
```

Everything works! There is one small tweak we can make. In our post list page -- under posts -- we currently see the number of all the comments the blog post has received. Let's change that to show the number of *approved* comments there.
Все працює! Існує один невеликої настройки, ми можемо зробити. У нашому списку повідомлень сторінці - під пости - в даний час ми бачимо кількість всіх коментарів повідомлення в блозі отримав. Давайте змінимо, що, щоб показати кількість * затверджених * коментарі там.

To fix this, go to `blog/templates/blog/post_list.html` and change the line:
Щоб це виправити, перейдіть в `blog/templates/blog/post_list.html` і змініть рядок:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

to:
щоб:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.approved_comments.count }}</a>
```

Finally, add this method to the Post model in `blog/models.py`:
Нарешті, додайте цей метод до моделі постів в `blog/models.py`:

```python
def approved_comments(self):
    return self.comments.filter(approved_comment=True)
```

Now your comment feature is finished! Congrats! :-)
Тепер Ваш коментар особливість закінчена! Вітаю! :-)
