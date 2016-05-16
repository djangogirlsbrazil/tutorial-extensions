# Domínio

O PythonAnywhere te deu um domínio grátis, mas talvez você não queira ter ".pythonanywhere.com" no final da URL do seu blog. Talvez você queira que seu blog esteja no "www.fotos-infinitas-de-gatinhos.org" ou "www.partes-de-motor-impressas-em-3d.com" ou "www.vintage-buttons.com" ou "www.unicornios-mutantes.net", ou qualquer outra coisa.

Aqui nós vamos falar um pouco sobre onde conseguir um domínio, e como ligá-lo ao seu aplicativo web no PythonAnywhere. Contudo, você precisa saber que a maioria dos domínios é paga e o PythonAnywere também tem a opção de, mediante um pagamento mensal, usar o nome de domínio que você quiser -- não é muito dinheiro, mas isso é algo que você provavelmente só vai querer fazer se fizer muita questão disso.


## Onde registrar um domínio?

Um típico domínio custa em torno de US$ 15 por ano. Existem opções mais baratas e mais caras, dependendo da empresa, e existem um monte de empresas em que você pode comprar um domínio. Uma simples busca no Google [google search](https://www.google.com/search?q=register%20domain) vai te retornar centenas de opções.

Nossa favorita é [I want my name](https://iwantmyname.com/). Eles anunciam como "gestão de domínio sem dor de cabeça" e realmente o são.

Vcoê pode também conseguir domínios de graça. [dot.tk](http://www.dot.tk) é O lugar para conseguir isso, mas você deve estar ciente de que domínios grátis também podem parecer amadores -- se o seu site for destinado a um negócio ou empresa, você pode cogitar preferir pagar para ter um domínio "adequado", que termine em `.com`.


## Como apontar para seu domínio em PythonAnywhere

Se você optou por ter seu próprio domínio em *iwantmyname.com*, clique `Domains` no menu e escolha seu domínio recém adquirido. Ache então o link `manage DNS records` e clique nele:

![](images/4.png)

Agora você precisa achar esse formulário:

![](images/5.png)

E preencher com os seguintes detalhes:
- Hostname: www
- Type: CNAME
- Value: your domain from PythonAnywhere (for example djangogirls.pythonanywhere.com)
- TTL: 60

![](images/6.png)

Clique no botão "Add" e salve as alteração na parte de baixo.


> **Nota** Se você quiser usar um outro provedor de domínio, o UI exato para achar suas configurações de DNS / CNAME serão diferentes, mas seu objetivo é o mesmo: configurar um CNAME que aponte para seu novo domínio em `yourusername.pythonanywhere.com`.

Pode demorar alguns minutos para seu dompinio começar a funcionar. Então, seja paciente. :) 


## Configure o domínio por um web app em PythonAnywhere.

Você precisa também contar pro PythonAnywhere que você quer usar seu domínio customizado.

Vá para [PythonAnywhere Accounts page](https://www.pythonanywhere.com/account/) e faça o upgrade de sua conta. A opção mais barata (um plano "Hacker") está ótima para começar. Você pode fazer o upgrade depois, quando estiver super famosa e conseguir milhões de acessos.

A seguir, vá para a [Web tab](https://www.pythonanywhere.com/web_app_setup/) e faça duas coisas:

* Copie o caminho **path to your virtualenv** e coloque em algum lugar seguro
* Clique em **wsgi config file**, copie o conteúdo e cole em algum lugar seguro.

A seguir, **Delete** seu antigo web app. Não se preocupe, isso não vai deletar nada do seu código, apenas apaga o domínio *yourusername.pythonanywhere.com*. A seguir crie um novo web app, e siga esses passos:

* Digite seu novo nome de domínio
* Escolha "manual configuration"
* Escolha Python 3.4
* E acabamos!

Aí você será levado de volta para a aba web (web tab).

* Cole o caminho para o ambiente virtual (virtualenv path) que você salvou antes
* Clique no arquivo wsgi configuration e cole o conteúdo do seu antigo arquivo config 

Clique no reload web app e você vai ver seu site rodando no seu novo domínio!

Se você encontrar algum problema, clique no link "Send feedback" no site do PythonAnywhere e um de seus amigáveis administradores vai te ajudar rapidinho.

