# Trabalho de casa: Adicione segurança para o seu website

Você pode ter reparado que não precisou usar sua senha depois de ter acessado a interface de admin. Você pode também ter notado que isso significa que qualquer pessoa pode adicionar ou editar posts no seu blog. Eu não sei quanto a você, mas eu não gostaria que qualquer pessoa poste no meu blog. Então vamos fazer alguma coisa a respeito.

## Autorizando a adição/edição de posts

Primeiro, vamos tornar as coisas seguras. Vamos proteger suas views `post_new`, `post_edit`, `post_draft_list`, `post_remove` e `post_publish` de forma que só usuários logados podem acessá-las. O Django possui algumas boas ferramentas para isso. É um tipo de tópico avançado, _decorators_ , mas não se preocupe agora com questões técnicas, você pode ler sobre isso mais tarde. O decorator que vamos usar está incluso no Django, no módulo `django.contrib.auth.decorators` e é chamado `login_required`.

Edite seu `blog/views.py` e adicione essas linhas no topo, junto com o resto das importações:

```python
from django.contrib.auth.decorators import login_required
```

Adicione então uma linha antes de cada view `post_new`, `post_edit`, `post_draft_list`, `post_remove` e `post_publish` (usando o decorator) assim:

```python
@login_required
def post_new(request):
    [...]
```

É isso! Agora tente acessar `http://localhost:8000/post/new/`, notou a difereça?

> Se você apenas aparecer a página vazia, você provavelmente ainda está logada por causa do capítulo sobre a admin-interface. Vá para 'http://localhost:8000/admin/logout/' para deslogar, e entção vá para 'http://localhost:8000/post/new' novamente.

Você deve conseguir um desses amados erros. Em, na verdade, isso é bastante interessante: O decorator que nós adicionamos antes vai redirecionar você para a página de login, mas isso ainda não está disponível. Por isso aparece a mensagem "Page not found (404)".

Não se esqueça de adicionar o decorator acima em `post_edit`, `post_remove`, `post_draft_list` e `post_publish` também.

Yay! Alçançamos parte da nossa meta! Outras pessoas não conseguem mais criar posts em nosso blog. Infelizmente, nós também não. Então o próximo passo é consertar isso.

## Login de usuários 

Agora nós poderíamos fazer várias coisas mágicas para implementar usuários e senhas e autenticação, mas fazer esse tipo de coisa corretamente é meio complicado. Como o Django tem "baterias inclusas", alguém já fez esse trabalho duro pra gente. Então vamos usar a parte de autenticação inclusa.

No seu `mysite/urls.py` adicione a url `url(r'^accounts/login/$', 'django.contrib.auth.views.login')`. O arquivo deve parecer agora com algo parecido com isso:

```python
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'', include('blog.urls')),
)
```

Aí então precisaremos de um template para a página de login. Crie então o diretório `blog/templates/registration` e um arquivo chamado `login.html`:

```django
{% extends "blog/base.html" %}

{% block content %}

{% if form.errors %}
<p>Seu nome de usuário e senha não conferem. Por favor, tente novamente.</p>
{% endif %}

<form method="post" action="{% url 'django.contrib.auth.views.login' %}">
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

Como você perceberá, isso fará uso do seu template base (base-template) para manter o mesmo visual do seu blog.

A coisa boa disso tudo é que isso  _simplesmentefunciona[TM]_. Não precisamos lidar com submissão de formuários ou senhas ou como protegê-los. A única coisa que precisamos fazer é adicionar uma configuração em `mysite/settings.py`:

```python
LOGIN_REDIRECT_URL = '/'
```

Agora quando o login for acessado diretamente, ele vai recirecionar o login bem-sucedido para o topo do índice.

## Melhorando o layout

Agora nós nos certificamos que somente usuários autorizados (isso é, você) podem adicionar, editar ou publicar posts, mas todo mundo ainda pode ver os botões para adicionar ou editar posts. Vamos então esconder isso dos usuários que não estão logados. Para isso, precisamos editar os templates. Vamos então começar com o template base em  `blog/templates/blog/base.html`:

```django
<body>
    <div class="page-header">
        {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
        <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        {% else %}
        <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
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

Você pode já ter reconhecido o padrão aqui: existe uma condição "se" (if) dentro do template que checa se o usuário é autenticado para exibir os botões de edição. Caso contrário, ele mostra o botão de login.

*Dever de casa*: Edite o template `blog/templates/blog/post_detail.html` para mostrar os botões de edição apenas para usuários autenticados.

## Mais sobre usuários autenticados

Vamos colocar mais um tempero em nossos templates enquanto ainda estamos aqui. Primeiro, vamos acrescentar algumas coisas para mostrar que estamos logados. Edite `blog/templates/blog/base.html` assim:

```django
<div class="page-header">
    {% if user.is_authenticated %}
    <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
    <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
    <p class="top-menu">Olá {{ user.username }}<small>(<a href="{% url 'django.contrib.auth.views.logout' %}">Log out</a>)</small></p>
    {% else %}
    <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
    <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
</div>
```

Isso vai acrescentar um  "Olá &lt;username&gt;" para lembrar nosso usuário e que estamos autenticados. Isso também acrescenta um link para fazer o log out do blog, mas como você pode perceber ainda não está funcionando. Ai meu deus, quebramos a internet! Vamos arrumar isso!

Decidimos confiar no django para lidar com login, vamos ver agora se o Django também pode lidar com o logout pra gente. Leia https://docs.djangoproject.com/en/1.8/topics/auth/default/ e veja se descobre algo.

Leu? Agora você deve estar pensando em acrescentar uma url (em `mysite/urls.py`) apontando para a view `django.contrib.auth.views.logout`, assim:

```python
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'^accounts/logout/$', 'django.contrib.auth.views.logout', {'next_page': '/'}),
    url(r'', include('blog.urls')),
)
```

É isso! Se vocÇe seguiu todos os passoas até esse momento (e fez o trabalho de casa), você agora tem um blog onde você:

 - precisa de um nome de usuário (username) e senha para fazer o log in,
 - precisa estar logado para adicionar/editar/publicar(/deletar) posts
 - e que pode deslogar novamente
