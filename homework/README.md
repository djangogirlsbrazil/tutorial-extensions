# Trabalho de casa: adicione mais em seu website!

Nosso blog já melhorou muito, mas ainda tem espaço para melhorar mais. A seguir, vamos adicionar funcionalidades para postar rascunhos e sua publicação. Vamos também adicionar a exclusão de posts que não queremos mais. Ótimo!

## Salve novos posts como rascunhos

Atualmente quando estamos criando novos posts usando nosso formulário *New post* o post é publicado diretamente. Para, ao invés disso, salvar seu post como rascunho, **retire** essa linha no `blog/views.py` no método `post_new`:

```python
post.published_date = timezone.now()
```

Dessa forma, novos posts serão salvos como rascunhos que poderemos reler mais tarde ao invés de serem instantâneamente publicados. Tudo o que precisamo agora é uma forma de listar e publicar os rascunhos, vamos lá!

## Página com lista de posts não publicados

Lembra do capítulo sobre querysets? Criamos uma view `post_list` que exibe só os post publicados no blog (aqueles com a `published_date` não vazias).

Agora é hora de fazer algo parecido, mas para post em rascunho.

Vamos adicionar um link em `blog/templates/blog/base.html` perto do botão para adicionar novos posts (bem em cima da linha `<h1><a href="/">Django Girls Blog</a></h1>` !):

```django
<a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
```

A seguir: urls! Em `blog/urls.py` adicionamos:

```python
url(r'^drafts/$', views.post_draft_list, name='post_draft_list'),
```

Hora de criar a view em `blog/views.py`:

```python
def post_draft_list(request):
    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
    return render(request, 'blog/post_draft_list.html', {'posts': posts})
```

Essa linha `Post.objects.filter(published_date__isnull=True).order_by('created_date')` assegura que pegaremos somente os posts ainda não publicados (`published_date__isnull=True`) e os ordena pela data de criação (`created_date`) (`order_by('created_date')`).

Ok, a última parte é um template, é claro! Crie um arquivo `blog/templates/blog/post_draft_list.html` e adicione o seguinte:

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

Parece bastante com nosso `post_list.html`, certo?

Agora quando você for para `http://127.0.0.1:8000/drafts/` você verá a lista dos posts ainda não publicados.

Yay! Você concluiu sua primeira tarefa!

## Adicione um botão para publicar

Seria legal ter um botão nos detalhes do post que imediatamente publique o post no blog, certo?

Vamos abrir `blog/template/blog/post_detail.html` e modificar as seguintes linhas:

```django
{% if post.published_date %}
    {{ post.published_date }}
{% endif %}
```

para essas:

```django
{% if post.published_date %}
    {{ post.published_date }}
{% else %}
    <a class="btn btn-default" href="{% url 'blog.views.post_publish' pk=post.pk %}">Publish</a>
{% endif %}
```

Como você notou, adicionamos a linha `{% else %}` aqui. Isso significa que se (if) a condição de `{% if post.published_date %}` não for verificada (ou seja, se não tiver data de publicação (`published_date`)), então queremos que seja executada a linha  `<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>`. Note que estamos passando a variável `pk` no `{% url %}`.

Hora de criar uma URL (in `blog/urls.py`):

```python
url(r'^post/(?P<pk>\d+)/publish/$', views.post_publish, name='post_publish'),
```

e filmanete, uma *view* (como sempre, em `blog/views.py`):

```python
def post_publish(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.publish()
    return redirect('blog.views.post_detail', pk=pk)
```

Lembre-se, quando criamos um modelo de `Post` escrevemos um método `publish`. Parecia com isso:

```python
def publish(self):
    self.published_date = timezone.now()
    self.save()
```

Agora podemos finalmente usar isso!

E mais uma vez, depois de publicar o post somos imediatamente redirecionados para a página `post_detail`!

![Publish button](images/publish2.png)

Parabéns! Você tá quase lá. O último passo é adicionar um botão para deletar!

## Deletando um post

Vamos abrir o `blog/templates/blog/post_detail.html` mais uma vez e adicione a seguinte linha:

```django
<a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

abaixo da linha com o botão de edição.

Agora precisamos adicionar uma URL (`blog/urls.py`):

```python
url(r'^post/(?P<pk>\d+)/remove/$', views.post_remove, name='post_remove'),
```

Agora, é a vez da view! Abra `blog/views.py` e adicione esse código:

```python
def post_remove(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('blog.views.post_list')
```

A única coisa nova é deletar um post do blog. Todo modelo de Django pode ser deletado por `.delete()`. Tão simples quanto isso!

E dessa vez, depois de deletar um post queremos ir para a webpage com a lista dos posts, por isso estamos usando `redirect`.

Vamos testar isso! Vá para a página com o post e tente deletar ele!

![Delete button](images/delete3.png)

Sim, isso era a última coisa! Você completou esse tutorial! Você é demais!
