# Coloque seu website no Heroku (como em PythonAnyhere)

è sempre bom para uma desenvolvedora ter algumas opções de desenvolvimento na manga. Por que não tentar colocar seu site no Heroku, como fizemos com o PythonAnywhere?

[Heroku](http://heroku.com/) também é grátis para pequenas aplicações que não têm muitos visitantes, mas é um pouquinho mais complicado para colocar o site.

Seguiremos este tutorial: https://devcenter.heroku.com/articles/getting-started-with-django, mas colamos aqui para faciliatr as coisas pra você.


## O arquivo `requirements.txt` 

Se você não criou um antes, precisaremos criar um arquivo `requirements.txt` para dizer ao Heroku que pacotes de Python precisam ser instalados em nosso servidor.

Mas antes o Heroku precisa que instalemos alguns novos pacotes. Vá para seu console com o `virtualenv` ativado e digite isso:

    (myvenv) $ pip install dj-database-url gunicorn whitenoise

Depois que a instalação terminar, vá para o diretório `djangogirls` e execute esse comando:

    (myvenv) $ pip freeze > requirements.txt

Isso vai criar um arquivo chamado `requirements.txt` com a lista dos seus pacotes instalados (isso é, das bibliotecas de Python que você está usando, por exemplo o Django :)).

> __Nota__: `pip freeze` retorna um lista de todas as biblitecas de Python instaladas em seu ambiente virtual (virtualenv) e o sinal `>` pega o resultado de `pip freeze` e coloca dentro de um arquivo. Tente executar `pip freeze` sem o `> requirements.txt` para ver o que acontece!

Abra esse arquivo e acrescente a seguinte linha no final:

    psycopg2==2.5.4

Essa linha é necessária para que sua aplicação funcione no Heroku.


## Procfile

Outra coisa que o Heroku vai precisar é um Procfile. Isso vai dizer ao Heroku quais comandos executar para criar nosso website. Abra seu editor de código e crie um arquivo chamado `Procfile` no diretório `djangogirls` e acrescente a seguinte linha:

    web: gunicorn mysite.wsgi

Essa linha vai dizer que iremos colocar uma aplicação `web` e vamos fazer isso executando o comando `gunicorn mysite.wsgi` (`gunicorn` é um programa que é tipo uma versão mais poderosa do comando `runserver` no Django).

Salve e está pronto!

## O arquivo `runtime.txt` 

Precisamos também dizer ao Heroku que versão do Python queremos usar. Isso é feito criando o arquivo `runtime.txt` no diretório `djangogirls` usando o comnado "new file" ou "novo arquivo" em seu editor de código e colocando o seguinte texto (e nada mais!) dentro do arquivo:

    python-3.4.2

## `mysite/local_settings.py`

Como ele é um pouco mais restritivo que o PythonAnywhere, o Heroku quer usar configurações diferentes daquelas que usamos localmente (em nosso comutador). O Heroku vai querer usar Postgres enquanto usamos SQLite, por exemplo. É por isso que precisamos criar um arquivo separado para as configurações que ficarão diponíveis apenas em nosso ambiente local.

Vá em frente e crie o arquivo `mysite/local_settings.py`. Ele deverá conter a configuração de seu `DATABASE` a partir de seu arquivo `mysite/settings.py`. Bem assim:

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

E é só salvar! :)

## mysite/settings.py

Outra coisa que precisaremos fazer é modificar o arquivo `settings.py` de nosso website. Abra `mysite/settings.py` em seu editor e acrescente as seguintes linhas ao final do código:

```python
import dj_database_url
DATABASES['default'] = dj_database_url.config()

SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

ALLOWED_HOSTS = ['*']

DEBUG = False

try:
    from .local_settings import *
except ImportError:
    pass
```

Isso irá fazer a configuração necessária para o Heroku e também importará todas as suas configurações locais se o `mysite/local_settings.py` existir.

E então salve o aqruivo.

## mysite/wsgi.py

Abra o arquivo `mysite/wsgi.py` e acrescente essas linhas ao final:

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

Tá pronto!

## Conta no Heroku

Você precisa intalar sua barra de ferramentas do Heroku que você pode acesar aqui (você pode pular essa instalação se já tiver instalado a barra durante o setup): https://toolbelt.heroku.com/

> Quando estiver executrando o programa de instalção da barra de ferramentas do Heroku no Windows marque a opção "Custom Installation" quando for perguntada quais componentes quer instalar. Na lista de componentes logo a seguir, por favor marque também a caixa que corresponde a "Git and SSH".

> No Windows você precisa executar o seguinte comando (no prompt de comando) para adicionar o Git e o SSH ao seu `PATH`: `setx PATH "%PATH%;C:\Program Files\Git\bin"`. Reinicie o prompt de comando depois para habilitar essa mudança.

> Depois que reiniciar seu prompt de comando, não se esqueça de ir para a sua pasta `djangogirls` novamente e ativar seu ambiente virtual (virtualenv)! (Dica: [Consulte o capítulo sobre a instalação do Django](http://tutorial.djangogirls.org/en/django_installation/index.html#working-with-virtualenv))

Crie também uma conta gratuita no Heroku aqui: https://id.heroku.com/signup/www-home-top

E então autentique sua conta no Heroku em seu computador executando esse comando:

    $ heroku login

Caso você não tenha uma chave SSH esse comando vai automaticamente criar uma. As chaves SSH são necessárias para fazer o push do código no Heroku.

## Git commit

O Heroku usa o git para fazer sua implementação. Ao contrário do PythonAnywhere, você pode fazer o push diretamente para o Heroku, sem precisar fazer isso via Github, mas precisaremos ajustar algumas coisas antes.

Abra o arquivo chamado `.gitignore` no seu diretório `djangogirls` e adicione `local_settings.py` nele.  Queremos que o git ignore `local_settings`, de forma que ele fique em nosso computador e não acabe subindo para o Heroku.

    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py


E então fazemos o commit das nossas mudanças

    $ git status
    $ git add -A .
    $ git commit -m "additional files and changes for Heroku"


## Escolha um nome para a aplicação

Tornaremos nosso blog disponível na Web no `[nome do seu blog].herokuapp.com`, então precisamos escolher um nome que ninguém pegou ainda. Esse nome não precisa estar relacionado ao app `blog` Django ou ao `mysite` ou a qualquer coisa que tenhamos criado até agora. O nome pode ser qualquer coisa que você queira, mas o Heroku é bem restrito quanto a quais caracteres você pode usar: você somente pode usar letras minúsculas simples (e não letras maiúsculas ou acentos), números e travessões (`-`).

Uma vez que você tenha pensado em um nome (talvez alguma coisa com seu nome ou apelido nele), execute esse comando, substituindo `djangogirlsblog` pelo nome da aplicação escolhido por você:

    $ heroku create djangogirlsblog

> __Nota__: Lembre-se de substituir `djangogirlsblog` pelo nome da sua aplicação no Heroku.

Se você não conseguir pensar em um nome, você pode executar

    $ heroku create

e o Heroku vai escolher um nome não utilizado para você (provavelmente algo do tipo `enigmatic-cove-2527`).

Se você quiser mudar o nome da sua aplicação no Heroku, você pode fazê-lo, a qualquer tempo, com esse comando (substitua `the-new-name` pelo nome que você quer usar):

    $ heroku apps:rename the-new-name

> __Nota__: Lembre-se que depois que alterar o nome da sua aplicação, você precisará visitar `[o-seu-novo-nome].herokuapp.com` para ver seu site.

## Implementando no Heroku!

Foi um bocado de configuração e instalação, certo? Mas você precisa fazer isso apenas uma vez! Agora você pode implementar!

Quando você executou `heroku create`, esse comando automaticamente adicionou o Heroku remoto para nosso app ao nosso repositório. Agora podemos fazer um simples  git push para implementar nossa aplicação:

    $ git push heroku master

> __Nota__: Isso provavelmente vai produzir um *monte* de saídas na primeira vez que você executar o comando, porque o Heroku compila e instala o psycopg. Você saberá que a execução foi bem-sucedida se vir alguma coisa como `https://yourapplicationname.herokuapp.com/ deployed to Heroku` próximo ao final das saídas.

## Visite sua aplicação

Você implementou seu código no Heroku e especificou os tipos de processo no `Procfile` (escolhemos o processo `web` anteriormente).
Agora podemos dizer ao Heroku para iniciar esse processo web (`web process`).

Para fazer isso execute o seguinte comando:

    $ heroku ps:scale web=1

Isso dirá ao Heorku para exeuctar apenas uma instância do nosso processo "web". Como nosso blog é bem simples, não precisamos de muita coisa então está bem em executar somente um processo. É possível pedir ao Heroku para executar mais processos (a propósito, o Heroku chama esses processos de "Dynos", então não se surpreenda se você vir esse termo), mas isso não é mais gratuito.

Nós podemos agora visitar o app em nosso navegador com `heroku open`.

    $ heroku open

> __Nota__: você verá uma página de erro! Vamos falar disso em um minuto.

Isso vai abrir uma url tipo [https://djangogirlsblog.herokuapp.com/]() em seu navegador, e no momento, você provavelmente vai ver uma página de erro.

O erro que você viu foi causado pelo fato de quando implementamos no Heroku, criamos um novo banco de dados e ele está vazio. Precisamos executar os comandos `migrate` e `createsuperuser` como fizemos no PythonAnywhere.  Dessa vez, eles vêm através de uma linha de comando especial no seu computador `heroku run`:

    $ heroku run python manage.py migrate

    $ heroku run python manage.py createsuperuser

O prompt de comando pedirá para você escolher um nome de usuário e senha novamente. Esses serão seus dados de login para a página de admin do seu website.

Clique para atualizar em seu navegador e tá pronto!  Agora você sabe como implementar seu website em duas diferentes plataformas de hopedagem. Escolha a sua favorita :)

