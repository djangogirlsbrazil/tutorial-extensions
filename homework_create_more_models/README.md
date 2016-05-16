# Trabalho de Casa: crie um modelo para comentários

Atualmente só temos um modelo de Post, que tal receber algum feedback de seus leitores?

## Criando um modelo de comentário para o blog

Vamos abrir `blog/models.py` e adicionar essa parte de código ao final do arquivo:

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

Você pode voltar para o capítulo **Django models** do tutorial se você quiser relembrar o que cada tipo de campo significa.

Nesse capítulo precisaremos criar um novo tipo de campo:
- `models.BooleanField` - esse é um campo do tipo verdadeiro/falso (true/false).

E a opção `related_name` em `models.ForeignKey` nos permite os comentários a partir do modelo de post.

## Criando tabelas para modelos em seu banco de dados 

Agora é hora de adicionar nosso modelo de comentário ao banco de dados. Para fazer isso, temos que informar ao Django que fizemos mudanças em nosso modelo. Digite `python manage.py makemigrations blog`. Assim:

    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migrations for 'blog':
      0002_comment.py:
        - Create model Comment

Você pode notar que esse comando criou pra gente outro arquivo de migração no diretório em `blog/migrations/`. Agora, precisamos aplicar essas mudanças com `python manage.py migrate blog`, dessa forma:

    (myvenv) ~/djangogirls$ python manage.py migrate blog
    Operations to perform:
      Apply all migrations: blog
    Running migrations:
      Rendering model states... DONE
      Applying blog.0002_comment... OK

Nosso modelo de Comentário existe no banco de dados agora. Seria legal se pudéssemos acessá-lo em nosso painel de admin...

## Registrando o modelo de comentário no painel de admin

Para registrar o modelo no painel de admin, vá para `blog/admin.py` e adicione as linhas:

```python
admin.site.register(Comment)
```

Não se esqueça de importar o modelo de Comentário, o arquivo deve parecer com isso:

```python
from django.contrib import admin
from .models import Post, Comment

admin.site.register(Post)
admin.site.register(Comment)
```

Se você digitar `python manage.py runserver` na linha de comando e for para [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) em seu navegador, você poderá acessar a lista, adicionar e remover comentários. Não deixe de brincar com isso. :)

## Torne seus comentários visíveis

Vá para o arquivo `blog/templates/blog/post_detail.html` e adicione essas linhas antes de `{% endblock %}` :

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

Agora podemos ver a seção de comentários nas páginas junto com os detalhes do post.

Mas isso pode ficar ainda melhor, adicione um pouco de css ao `static/css/blog.css`:

```css
.comment {
    margin: 20px 0px 20px 20px;
}
```

Podemos também deixar os visitantes verem os comentários na lista de posts. Vá para o arquivo `blog/templates/blog/post_list.html` e adicione a linha:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

Depois disso, nosso template deve ficar assim:

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

## DEixe seus leitores postarem comentários

Agora podemos ver comentários no nosso blog, mas não podemos adicioná-los. Vamos mudar isso!

Vá para `blog/forms.py` e adicione essas linhas no final do arquivo:

```python
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = ('author', 'text',)
```

Não se esqueça de importar o modelo de Comentário, modificando a linha:

```python
from .models import Post
```

para:

```python
from .models import Post, Comment
```

Agora, vá para `blog/templates/blog/post_detail.html` e antes da linha `{% for comment in post.comments.all %}` adicione:

```django
<a class="btn btn-default" href="{% url 'add_comment_to_post' pk=post.pk %}">Add comment</a>
```

Vá para a página de detalhe do post e você deve ver esse erro:

![NoReverseMatch](images/url_error.png)

Vamos consertar isso! Vá para `blog/urls.py` e adicione esse padrão (pattern) para `urlpatterns`:

```python
url(r'^post/(?P<pk>\d+)/comment/$', views.add_comment_to_post, name='add_comment_to_post'),
```

Agora, você deve estar vendo esse erro:

![AttributeError](images/views_error.png)

Para corrigir isso adicione esse código ao `blog/views.py`:

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

Não se esqueça dos imports no começo do arquivo:

```python
from .forms import PostForm, CommentForm
```


Agora, na página de detalhes do post, você deve estar vendo o botão "Add Comment":

![AddComment](images/add_comment_button.png)

Contudo, quando você clica nele, deve ver:

![TemplateDoesNotExist](images/template_error.png)


Como o erro menciona, não existe o template. Por isso, crie um como `blog/templates/blog/add_comment_to_post.html` e adicione as seguintes linhas:

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

Yay! Agora seus leitores podem te dizer o que acham, abaixo de seus posts!

## Moderando seus comentários

Nem todos os comentários devem ser exibidos. O proprietário do blog deve ter a opção de aprovar ou deletar os comentários. Vamos fazer algo a respeito!

Vá para `blog/templates/blog/post_detail.html` e modifique as linhas:

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

para:

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

Você deve ver `NoReverseMatch`, porque nenhuma url confere com os padrões de `comment_remove` e `comment_approve`.

Adicione os padrões de url ao `blog/urls.py`:

```python
url(r'^comment/(?P<pk>\d+)/approve/$', views.comment_approve, name='comment_approve'),
url(r'^comment/(?P<pk>\d+)/remove/$', views.comment_remove, name='comment_remove'),
```

Agora vcoê deve ver `AttributeError`. Para se livrar disso, crie mais views em `blog/views.py`:

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

E, é claro, conserte os imports.

Tudo funciona, mas tem algo errado. Em nossa página com a lista de posts, vemos o número de todos os comentários, mas queremos que apareça apenas o número de comentários aprovados lá.

Vá para `blog/templates/blog/post_list.html` e modifique a linha:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

para:

```django
<a href="{% url 'blog.views.post_detail' pk=post.pk %}">Comments: {{ post.approved_comments.count }}</a>
```

E adicione também esse método ao modelo de Post em `blog/models.py`:

```python
def approved_comments(self):
    return self.comments.filter(approved_comment=True)
```

Agora sua funcionalidade de comentário está terminada! Parabéns! :-)
