# Instalação do PostgreSQL

> Parte desse capítulo é baseada nos tutotiais da Geek Girls Carrots (http://django.carrots.pl/).

> Partes desse capítulo são baseadas no [django-marcador
tutorial](http://django-marcador.keimlink.de/) licenciado sob a licença Creative Commons
Attribution-ShareAlike 4.0 International License. O django-marcador tutorial
é registrado para Markus Zapke-Gründemann et al.


## Windows

A forma mais fácil de instalar o Postgres no Windows é usando um programa que você pode encontrar aqui: http://www.enterprisedb.com/products-services-training/pgdownload#windows

Escolha a versão disponível mais atualizada para seu sistema operacional. Faça o download do instalador, execute e então siga as instruções disponíveis aqui: http://www.postgresqltutorial.com/install-postgresql/. Anote o diretório de instalação porque você irá precisar dele no próximo passo (normalmente é `C:\Program Files\PostgreSQL\9.3`).

## Mac OS X

A forma mais fácil é fazer o download gratuito do [Postgres.app](http://postgresapp.com/) e instalar como qualquer outra aplicação em seu sistema operacional.

Faça o download, arraste para o diretório de Aplicações e execute clicando duas vezes. E é isso!

Você também precisará adicionar as ferramentas de comando de linha do Postgres para sua variável `PATH`, conforme descrito [aqui](http://postgresapp.com/documentation/cli-tools.html).

## Linux

Os passos de instalação variam de distribuição para distribuição. Abaixo estão os comandos para Ubuntu e Fedora, mas se você estiver usando uma distro diferente, [dê uma olhada na documentação do PostgreSQL](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux).

### Ubuntu

Execute o seguinte comando:

    sudo apt-get install postgresql postgresql-contrib

### Fedora

Execute o seguinte comando:

    sudo yum install postgresql93-server

# Crie um banco de dados

A seguir, precisamos criar nosso primeiro banco de dados e um usuário que possa acessar esse banco. O PostgreSQL permite criar quantos bancos de dados e usuários você quiser, então se vcoê estiver criando mais de um site, você deverá criar um banco de dados para cada um dos sites.

## Windows

Se você estiver usando Windows existem dois passos adicionais a seguir para terminar. Por enquanto não é importante para você entender a configuração que estamos fazendo aqui, mas sinta-se à vontade para perguntar à sua treinadora se você estiver curiosa sobre o que está acontecendo. 

1. Abra o prompt de comando (Menu iniciar → Todos os Programas → Accessórios → Prompt de Comando)
2. Execute o comando: `setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"` e aperte a tecla enter. Você pode colar coisas no Prompt de Comando clicando no botão direito do mouse e selecionando `Colar`. Assegure-se de que o caminho é o mesmo que você anotou durante a instalação com `\bin` adicionado ao final. Você deverá ver a mensagem `SUCESSO: O valor especificado foi salvo.`.
3. Feche e abra novamente o Prompt de Comando.

## Crie o banco de dados

Primeiro, vamos abrir o console do Postgres executando `psql`. Lembra como abrir o console?
>No Mac OS X você pode fazer isso abrindo o `Terminal` (está em Aplicações → Utilidades). No Linux provavelmente está em 
Aplicações → Acessórios → Terminal. No Windows você precisará ir para o Menu Iniciar → Todos os Programas → Acessórios → Prompt de Comando. Alem disso, no Windows o `psql`pode exigir o login com senha e usuário durante a instalação. Se o `psql` pedir por uma senha, mas se não estiver funcionando, tente `psql -U <username> -W` primeiro e digite a senha depois.

    $ psql
    psql (9.3.4)
    Type "help" for help.
    #

Nosso `$` agora mudou para um `#`, o que significa que agora estamos mandando comandos para o PostgreSQL. Vamos criar um usuário:

    # CREATE USER name;
    CREATE ROLE

Substitua `name` por seu próprio nome. Você não deve usar acentos ou espaços em branco (por exemplo, `bożena maria` é inválido e precisará ser convertido para `bozena_maria`).

Agora é hora de criar um banco de dados para seu projeto Django:

    # CREATE DATABASE djangogirls OWNER name;
    CREATE DATABASE

Lembre-se de substituir `name` pelo nome que você escolheu (por exemplo, `bozena_maria`).

Ótimo! Tudo certo com os bancos de dados!

# Atualizando as configurações

Encontre essa parte em seu arquivo `mysite/settings.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

E substitua por isso:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

Lembre-se de substituir `name` pelo nome de usuário que você criou conforme o início desse capítulo.

# Instalando o pacote PostgreSQL para Python

Primeiro, instale a barra de ferramentas do Heroku a partir de https://toolbelt.heroku.com/. Apesar do fato que iremos precisar dela mais tarde, principalmente para implementar seu site, ela inclui o Git, que nós precisaremos agora.

A seguir, precisaremos instalar um pacote que permite ao Python "conversar" com o PostgreSQL - chamado `psycopg2`. As instruções de intalação são um pouco diferentes entre o Windows e o Linux/OS X.

## Windows

No Windows, faça o download do arquivo a partir de http://www.stickpeople.com/projects/python/win-psycopg/

Assegure-se de escolher o arquivo correspondente a sua versão do Python (3.4 deve ser a última linha) e à correta arquitetura (32 bit na colona esquerda ou 64 bit na coluna direita) 

Renomeie o arquivo baixado e mova para C:, de forma que fique disponível em `C:\psycopg2.exe`.

Uma vez feito isso, digite o seguinte comando no terminal (assegure-se de que seu `virtualenv` esteja ativado):

    easy_install C:\psycopg2.exe

## Linux e OS X

Execute o seguinte em seu console:

    (myvenv) ~/djangogirls$ pip install psycopg2

Se tudo der certo, você verá algo parecido com isso

    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...

---

Assim que estiver concluído, execute `python -c "import psycopg2"`. Se não aparecer nenhuma mensagem de erro, está tudo instalado corretamente.
