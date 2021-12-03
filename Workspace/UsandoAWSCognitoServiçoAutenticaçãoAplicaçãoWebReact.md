# Usando o AWS Cognito como serviço de autenticação em uma aplicação Web em React

[![Lucas Maia](https://miro.medium.com/fit/c/56/56/0*X8wRZDuxPhaDLTeO.jpg)](https://medium.com/@lucaslucm?source=post_page-----33b5ac2e7448-----------------------------------)

[Lucas Maia](https://medium.com/@lucaslucm?source=post_page-----33b5ac2e7448-----------------------------------) - [Jun 2, 2019·16 min read](https://medium.com/@lucaslucm/usando-o-aws-cognito-como-serviço-de-autenticação-com-react-e-spring-33b5ac2e7448?source=post_page-----33b5ac2e7448-----------------------------------)



![img](https://miro.medium.com/max/2000/0*3WJ2Y3R4K9w5GJTP)

Photo by [Serena Repice Lentini](https://unsplash.com/@serenarepice?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

Autenticação e manutenção de usuários é uma parte importante da maioria das aplicações web construídas hoje, muito embora seja um processo não intrinsicamente ligado às regras negociais principais da maior parte dessas aplicações. Além disso, é algo que precisa de uma manutenção cuidadosa devido a sensibilidade do tipo de informação armazenada e a construção e manutenção de um conjunto de funcionalidades que não costumam variar de aplicação para aplicação. Validação de e-mail, telefone, autenticação multifator (MFA), processo de troca de senha, criptografia de senhas: todas essas são funcionalidades que são importantes e que precisam ser implementadas dentro de um serviço de autenticação.

Dependendo da escala, faz mais sentido delegar essa parte para um serviço terceiro e focar no núcleo negocial de um sistema, ao invés de implementar e manter um serviço próprio desse tipo. A AWS possui um serviço desses, chamado Cognito, a quem você pode optar por delegar todo esse processo de autenticação. Este artigo, tem a intenção de propor uma integração de uma aplicação usando React com o AWS Cognito.

# **Para quem este artigo é indicado**

- Desenvolvedores que não possuem largo conhecimento em Cognito e nos seus processos de autenticação, que já ouviram falar de JWT e Oauth, mas que desejam entender melhor o assunto de maneira prática
- Desenvolvedores que tem intenção de usar um serviço de autenticação terceiro e desejam entender um pouco mais em como o Cognito funciona
- Desenvolvedores que desejam criar uma aplicação com Frontend em React e integrar com um Backend próprio, e desejam utilizar o Cognito como serviço de autenticação, mas que não desejam depender do processo de *scaffolding* do Amplify (se você não entende o que esse processo de *scaffolding* significa, eu tento explicá-lo um pouco melhor numa seção mais a frente)

Antes de iniciar, este artigo utiliza termos da AWS em inglês e presume que sua interface esteja em inglês. Recomendo alterar a linguagem, na opção que aparece no rodapé das páginas do console web da AWS, caso não esteja configurado para inglês. Esse artigo subentende também que você já tenha criado uma conta na AWS e que seja levemente familiar com seu console web.

# **Introdução ao AWS Cognito**

O AWS Cognito é um serviço da AWS de autenticação de usuários. Ele consegue ser usado para centralização de armazenamento de informações de usuários e como centralizador do processo de autenticação, através de tokens JWT (https://jwt.io/), inclusive fazendo o processo de autenticação com serviços externos que fornecem autenticação OpenID (https://pt.wikipedia.org/wiki/OpenID), ou serviços parecidos, como o Facebook e o Google.

O serviço já vem embutido com funcionalidades bem completas, como possibilidade de implementação de um *Multifactor Authentication* (MFA) — que está fora do escopo deste artigo; processo de troca de senha; processo de confirmação de e-mail cadastrado; processo de confirmação de telefone; entre outros.

Tudo isso o AWS Cognito fornece através de seus *User Pools*, que podem ser criados dentro da sua conta AWS. O preço desse serviço depende da quantidade de usuários ativos mensalmente, ou seja, a quantidade de usuários que se autenticam na sua plataforma por mês, o que significa que, mesmo que você tenha cem mil usuários cadastrados na base, se uma parte desses usuários não se autenticarem na sua aplicação pelo menos uma vez por mês, você não será cobrado pela totalidade dos usuários; mas, sim, pela quantidade que de fato usou o sistema. Na data de criação deste artigo, o AWS Cognito é gratuito caso você tenha menos de 7000 usuários ativos mensalmente.

Além das *User Pools*, o Cognito fornece um segundo serviço chamado *Identity Pools,* com o qual você pode associar usuários da sua aplicação com IAMs da AWS (https://aws.amazon.com/pt/iam/), que é um serviço de autorização da nuvem da Amazon, no qual você gerencia o controle de acesso de usuários a estruturas da sua *cloud*, como *buckets* dentro do S3, ou acesso através do API Gateway. Isso pode ser bem útil caso se queira fazer comunicação direta entre uma aplicação Frontend com um serviço da nuvem sem necessidade de uma aplicação intermediária. Os *Identity Pools* podem ser bem úteis em aplicações que utilizem o *AppSync*, ou outros serviços da AWS que foquem no desenvolvimento *serverless* e mobile. As *Identity Pools* fogem um pouco do escopo deste artigo, mas recomendo a introdução dada na página do Amplify (https://aws-amplify.github.io/docs/js/authentication), embora este artigo só introduza o Amplify mais a frente. Recomendo a leitura da introdução até a parte onde o autor começa a introduzir o processo de *Setup*, principalmente por conta dos diagramas da página, mas aconselho a ignorar os trechos de código, ao menos por enquanto.

# **Alternativas ao AWS Cognito**

Existem algumas alternativas ao AWS Cognito que fornecem serviços similares. Citaremos dois neste artigo, que parecem ser os principais concorrentes: o Auth0 (https://auth0.com/) e o Firebase Authentication (https://firebase.google.com/docs/auth/?hl=pt-br). O Cognito possui vantagem do preço sobre o Auth0, cobrando um pouco menos por usuário ativo. O modo de cobrança do Firebase Authentication parece ser por processo de autenticação bem sucedido, o que deixa um pouco mais complicado fazer uma comparação direta, já que o mesmo usuário pode se autenticar em múltiplos dispositivos simultaneamente, ou um usuário em um mesmo dispositivo pode perder seus tokens de acesso, forçando um novo processo.

O AWS Cognito pode ser uma alternativa interessante pelo seu preço e por sua possibilidade de integração fácil com outros serviços da AWS, através dos *Identity Pools.* Se a idéia é utilizar algum outro serviço de nuvem, as outras alternativas podem ser mais interessantes.

# Criando e configurando uma nova User Pool

Para iniciar a parte prática deste artigo, precisamos inicialmente criar uma *User Pool* para nossa aplicação. Ela é uma parte fundamental para todo o restante deste artigo.

Na página inicial do Cognito, existem duas opções: a de *Manage User Pools* e *Manage Identity Pools*. As *User Pools* do Cognito são de fato onde as informações dos usuários como login, senha, e-mail e telefone serão armazenadas e por onde podemos realizar o processo de *sign in* e *sign up*. As *Identity Pools* servem para gerenciar acessos e são bem úteis para arquiteturas *serverless.* Nós focaremos nas *User Pools*, já que temos a intenção de interagir com um backend próprio no futuro.

Na tela inicial, clique em *Manage User Pools.*

![img](https://miro.medium.com/max/60/1*3CbgxvYIbkyRw1Kl7h_tWA.png?q=20)

![img](https://miro.medium.com/max/630/1*3CbgxvYIbkyRw1Kl7h_tWA.png)

Na tela inicial de gerência de *User Pools,* clique no botão *Create a user pool*, no canto superior direito da interface.

Na nova tela, dê um nome para identificação dessa *pool* no campo *Pool name* e clique em *Review Defaults.* Essa opção permite que uma nova *User Pool* seja criada com configurações padrões. Caso deseje uma visão mais detalhada dos passos de criação, recomendo o seguinte vídeo no youtube: https://www.youtube.com/watch?v=TowcW1aTDqE. Manterei uma listagem com este link, junto com alguns outros no final desta seção.

![img](https://miro.medium.com/max/60/1*XQaATJu0EE0e38-oyT9P2A.png?q=20)

![img](https://miro.medium.com/max/630/1*XQaATJu0EE0e38-oyT9P2A.png)

Revisar todas as opções de customização do Cognito foge um pouco do propósito deste artigo, mas é importante citar que aqui se pode configurar opções como MFA, campos customizados para o usuário, que campos são obrigatórios para o usuário ser cadastrado, se o email do usuário deve ser validado etc. Para a continuidade desse artigo, precisamos além do básico apenas adicionar *App Clients* para nossa *User Pool.*

Na tela de *review,* existe uma sessão *App clients,* com uma opção clicável *Add app client*. Clique nessa opção. Na nova tela, clique em *Add an app client.* Dê um nome para seu *App Client.* No nosso caso daremos um sufixo "-react" para indicar que a chave desse *client* será usada em uma aplicação react, para fins de organização. **Desmarque** a opção *Generate client secret*, pois as libs JavaScript do Cognito não funcionam bem no caso de *App clients* com uma *client secret*. Pode deixar o restante das opções como está e clique em *Create app client*.

![img](https://miro.medium.com/max/60/1*xXNdr7UdbqmQjC9dQkNSZw.png?q=20)

![img](https://miro.medium.com/max/630/1*xXNdr7UdbqmQjC9dQkNSZw.png)

Adicionado esse *App Client*, clique novamente em *review* no menu esquerdo e clique em *Create pool,* que é o último botão da página. Isso irá criar nossa nova *User Pool*.

Dessa nova *User Pool*, é importante anotarmos algumas informações para serem usadas para autenticação: a *Pool Id* que aparece logo na primeira página, a *General Settings;* a região onde está hospedada, que pode ser presumida pelo novo *Pool Id;* e o *App Client Id,* que pode ser obtido na página *App client settings*, acessível pelo menu a esquerda.

Com a *User Pool* criada e todas essas informações anotadas, já podemos registrar novos usuários através de uma aplicação React usando o Amplify. Esse é o assunto a ser tratado no tópico seguinte; mas antes de pular para esse tópico, vou finalizar este com um conjunto de links que serviram bastante a mim durante o processo de aprendizado, que podem servir para aprofundamento de conhecimento do leitor. Uma parte desses inclusive já foram citados no decorrer da seção. Os links abaixo tratam de introduções ao Cognito e a configuração de uma *User Pool*.

- Developing Authentication with Cognito User Pool and JavaScript Apps — https://ehsangazar.com/developing-authentication-with-cognito-and-javascript-apps-bbf3686612e7
- Live Coding with AWS: API Authentication with Amazon Cognito — https://www.youtube.com/watch?v=TowcW1aTDqE
- What is Amazon Cognito? — https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html
- Step 1. Create a User Directory with a User Pool — https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pool-as-user-directory.html
- Authentication — https://aws-amplify.github.io/docs/js/authentication

# **Criando a aplicação React**

Antes de falar de código, preciso salientar que criei alguns repositórios no Github para servir como suporte para este artigo e os demais artigos relacionados. Cuidarei para explicar as partes principais de cada aplicação, iniciando pelo lado React, mas esses repositórios podem servir como base para criação de novas aplicações. Sinta-se livre para fazer *forks* ou copiar esse código para customização.

O repositório no Github pode ser acessado nesse endereço: https://github.com/lucasluc4/my-react-cognito-app . Deixarei este link também na seção de endereços desta seção para deixá-lo mais visível. Sinta-se livre para propor melhorias e realizar *Pull Requests.*

**Amplify**

A principal dependência deste projeto React é o Amplify, que é uma grande lib *client* que possui variantes Web, iOS*,* Android e React Native para serviços da AWS. Ela não possui apenas funções para interagir com o Cognito, mas também com várias outras partes componentes da AWS, como o S3, API e afins. Além desse tipo de método, a biblioteca possui um sistema de *scaffolding* que cria, através do *CloudFormation* da AWS, estruturas na nuvem, de maneira que um desenvolvedor Frontend, ou Mobile, pode abstrair boa parte do processo de criação de estruturas Backend e focar, quase exclusivamente, no desenvolvimento da experiência do usuário e interfaces.

A grande desvantagem desse processo de *scaffolding* é a perda do controle sobre o que se está sendo criado. Explico: pode ser que você deseje utilizar a UI web da AWS para administrar seus componentes da nuvem, ou delegar essa criação para um serviço de infraestrutura como código, como o *Terraform* (https://www.terraform.io/). Nesse caso, você pode utilizar as libs do Amplify para interação com sua estrutura AWS, deixando de lado essa possibilidade de criação de componentes através dela. Pela minha experiência enquanto estudante, os guias disponíveis pela internet dão pouco suporte a usuários desse tipo, se baseando bastante no processo de *scaffolding* do Amplify. Este artigo tentará fugir o máximo possível desse tipo de abordagem.

**Criando a aplicação e configurando dependências**

A primeira coisa que precisamos fazer é criar a aplicação react. Para isso, utilizei o create-react-app do facebook (https://github.com/facebook/create-react-app).

```
$ npx create-react-app my-react-cognito-app
```

Depois, podemos dar a sequência de comandos:

```
$ cd my-react-cognito-app
```

Para entrar na pasta do projeto, e o comando:

```
$ yarn add aws-amplify
```

Para adicionar a lib do Amplify como dependência do nosso projeto.

Aqui vale um comentário sobre a lib `aws-amplify-react` (https://aws-amplify.github.io/docs/js/react) que pode ser usada para construir aplicações React com o Amplify. Ela possui uma série de páginas já prontas, com formulários, que podem ser customizadas, para serem incluídas em aplicações React. Não usaremos essa lib porque queremos implementar todos os processos.

Por último, dentro de `src` vamos criar um arquivo `amplify-config.js` para armazenar configurações do Amplify sobre nossa estrutura AWS, substituindo os valores respectivos obtidos enquanto configuramos nossa *User Pool.*

Para finalizar, adicionamos a importação do bootstrap dentro do nosso `index.html` da pasta `public` . Poderíamos ter usado, naturalmente, uma implementação do bootstrap para React e utilizar o yarn para importação dessa dependência, mas preferi utilizar o bootstrap padrão para manter, nos nossos exemplos, o uso de componentes nativos do HTML, de forma que o leitor possa customizar esse componentes mais facilmente sem precisar depender de uma biblioteca de componentes.

Por último, precisamos chamar o método `configure` do Amplify dentro do nosso `App.js` para configurar globalmente o Amplify utilizado na nossa aplicação. Para isso, adicionamos as linhas seguintes dentro do nosso arquivo.

```
import Amplify from 'aws-amplify';import amplify_config from './amplify_config';Amplify.configure(amplify_config);
```

De maneira que nosso `App.js` seja algo parecido com o código abaixo.

Agora, na raiz do projeto podemos dar o comando`$ yarn start dev` para inicializar nossa aplicação bem rudimentar. Navegando para `localhost:3000` no navegador, nossa aplicação deve parecer algo como a seguinte imagem.

![img](https://miro.medium.com/max/60/1*if0_3BxXqU8Iz7OKKFASCw.png?q=20)

![img](https://miro.medium.com/max/630/1*if0_3BxXqU8Iz7OKKFASCw.png)

**O componente de SignUp**

É preciso dizer que algumas simplificações foram feitas para controlar o estado da nossa aplicação, para fins de ilustração. Recomendo, para uso em aplicações em produção, o uso de algum controle de estado melhor elaborado, e utilizando, no mínimo, um react-router.

Nosso componente de SignUp precisa ser mais ou menos parecido com o código apresentado a seguir.

Boa parte desse código executa o controle de estado dos formulários mostrados na tela e das informações inseridas. Um leitor com alguma experiência em React pode estar bem familiarizado com a estrutura desse componente. Precisamos focar especialmente em duas funções do Amplify que estão sendo executadas: a primeira na linha 38, que é a `Auth.signUp` que inicia o processo de cadastro de novos usuários; e a segunda, na linha 62, a `Auth.confirmSignUp` que finaliza o processo de cadastro, enviando a informação de código de confirmação, que deve ser recebido no email inserido no primeiro formulário.

No nosso arquivo `App.js`, se importarmos o componente, devemos obter algo como as telas mostradas a seguir. Tenha atenção para preencher a informação de e-mail com um endereço válido, pois é onde será recebido o código para confirmação. O telefone deve ser preenchido adicionando um +55 na frente e apenas com números, exceto pelo caractere + no início. A senha também deve estar de acordo com as políticas definidas na nossa *User Pool*, que por padrão precisa ter no mínimo 8 caracteres, com maiúsculas e minúsculas e um caractere especial.

![img](https://miro.medium.com/max/60/1*d0H5Bo0_Dr2QYqbGgWNJlw.png?q=20)

![img](https://miro.medium.com/max/630/1*d0H5Bo0_Dr2QYqbGgWNJlw.png)

Preenchendo o formulário corretamente, o formulário de confirmação deve ser apresentado. Preencha com o código que deve ser enviado para o e-mail inserido e clique em Confirmar.

![img](https://miro.medium.com/max/60/1*rB2wb0YxggQoCcMYRNxWBw.png?q=20)

![img](https://miro.medium.com/max/630/1*rB2wb0YxggQoCcMYRNxWBw.png)

No fim do processo, na página inicial da nossa *User Pool*, a de *General Settings,* no console web do Cognito na AWS, na opção *Estimated number of users*, o valor 1 deve estar sendo apresentado. Acessando a página *Users and groups* no menu da esquerda deve listar o usuário que foi registrado.

**O Componente de SignIn**

No repositório dessa aplicação, construímos dentro do nosso arquivo `App.js` um controle de alteração do formulário visível (*SignIn* ou *SignUp*) para que a navegação dentro da aplicação exemplo fosse completa. Contudo, para efeito de simplificação, o leitor pode simplesmente substituir o formulário importado no arquivo para testar o processo de SignIn.

No geral, a estrutura do componente de SignIn é bastante similar ao do SignUp, com todos os controles de estado envolvidos. Discutiremos nesta seção os principais métodos utilizados dentro do processo de SignIn. Caso o leitor tenha interesse em visualizar o código completo do componente, ele pode ser encontrado no repositório associado a este artigo.

Colocarei aqui o código de exemplo do *handler* de *submit* do formulário de autenticação, para ser discutido logo em seguida.

Alguns trechos desse código valem a pena ser destacados. O primeiro é o uso do método `Auth.signIn` que recebe um objeto contendo *username* e *password* e retorna uma Promise a ser resolvida quando o processo de autenticação for concluído. A partir desse momento, podemos passar a chamar o método `Auth.currentSession` para obter os *tokens* de acesso do usuário. Esse método retorna uma Promise que, em caso de sucesso, vai devolver um objeto com uma série de informações, incluindo os *tokens* de identificação e *refresh* nos caminhos que são explicitados nesse trecho de código: `userSession.idToken.jwtToken` e `userSession.refreshToken.token` , respectivamente.

Vale salientar que é importante manter esse controle de cache dos tokens de acesso ao Amplify e invocar o método `Auth.currentSession` toda vez que for necessário obter o token de acesso do usuário para realizar chamadas autenticadas. Esse método não faz novas chamadas pela internet caso não seja preciso, guardando localmente as informações necessárias, em *localStorage* por padrão, no caso de aplicações Javascript Web como a nossa. Convido o leitor a fazer testes nesse sentido para perceber, via Console do navegador, que novas chamadas HTTP não são feitas ao invocar esse método seguidamente. O Amplify, inclusive, já executa de maneira transparente o processo de *refresh* do *token* de acesso em caso de expiração. Vale observar que esses *tokens* estão sendo inseridos no estado do componente apenas para fim de visualização no caso do nosso código acima, mas é desinteressante guardar esses valores em variáveis para reuso.

Com isso, podemos inclusive modificar o método `componentDidMount` do nosso *Form* para verificar se existe um usuário logado no sistema, como descrito no trecho a seguir.

Caso não exista um usuário logado a *Promise* sofre um *reject* e o trecho de código dentro do bloco *catch* é executado.

Para realizar o processo de *SignOut,* podemos executar o método `Auth.signOut` como descrito a seguir. Depois de executado, novas chamadas ao `Auth.currentSession` deve retornar a mensagem que não existe um usuário logado no sistema.

Recomendo o leitor juntar as peças e implementar uma página de login com os métodos descritos nesta seção. Caso prefira, o repositório associado a este artigo possui a implementação de uma página com esse objetivo. A tela da aplicação do repositório é mostrado na imagem a seguir.

![img](https://miro.medium.com/max/60/1*L4RyM1RGnE2PG-Kf4YtOpw.png?q=20)

![img]()

Nossa aplicação está com o fluxo quase totalmente já descrito. A imagem anterior mostra, contudo, uma funcionalidade que não falamos até então, que é a funcionalidade de dar a opção do usuário escolher que suas informações sejam esquecidas no caso de abertura de uma nova sessão na nossa aplicação. Uma das maneiras de implementar essa funcionalidade é alterar o local de armazenamento dessas informações, através de uma configuração do Amplify. A próxima seção descreve essa funcionalidade em detalhes.

**Alterando a configuração de \*Storage\* do Amplify e implementando a funcionalidade de "Lembrar-se de mim"**

O Auth do Amplify pode ser configurado de maneira global através de uma chamada para `Auth.configure` . Dentre as possíveis opções, temos a opção de `storage` que determina onde as informações do usuário serão armazenadas na nossa aplicação. Por padrão, as informações são cacheadas no *localStorage* do navegador que permite que tais informações permaneçam salvas mesmo que o navegador seja fechado.

Uma alternativa ao *localStorage* é o *sessionStorage*, que possui um armazenamento mais volátil. Caso nossa aplicação seja fechada e reaberta, as informações do usuário serão perdidas. Essa é uma funcionalidade importante para garantir uma maior segurança aos usuários, que podem estar acessando nossas aplicações através de um computador público, ou em algum equipamento que não seja seu.

Para implementar essa funcionalidade, adicionamos na nossa página um *input* do tipo *checkbox* que altera a configuração de *storage* do Auth dependendo do seu valor, associando um método ao *onChange* do componente, dessa forma: `<input defaultChecked type="checkbox" className="form-check-input" id="rememberMeSignInInput" onChange={this.changeAuthStoragaConfiguration} />` .

Nosso método que altera o *storage* de maneira bem rudimentar pode ser implementado da seguinte forma:

<iframe src="https://medium.com/media/562cd503480e6e371a86cc5a5ade6b0d" allowfullscreen="" frameborder="0" height="523" width="680" title="changeAuthStorageConfiguration.js" class="ef es eo ex w" scrolling="auto" style="box-sizing: inherit; width: 680px; position: absolute; left: 0px; top: 0px; height: 522.986px;"></iframe>

Basicamente, estamos instanciando um tipo de *Storage* dependendo da informação que vem do nosso *checkbox,* chamando o método `Cache.createInstance` que deve ser importado do pacote do Amplify, modificando nossa linha de importação do Auth para algo como: `import {Auth, Cache} from "amplify";` . Apontamos nosso storage para o `window.sessionStorage` ou o `window.localStorage` dependendo da opção do nosso usuário.

Recomendo o usuário testar essa funcionalidade fazendo o processo de login na nossa aplicação, fechando a janela do navegador e acessando a aplicação novamente. Dois comportamentos diferentes devem ser observados dependendo do tipo de opção selecionado na nossa *checkbox* durante o processo de *login*.

Isso conclui a discussão de todas as funcionalidades da nossa aplicação React, que já é capaz de realizar o registro de novos usuários, finalizar o processo de confirmação por e-mail, fazer o login, obter *tokens* JWT de autenticação de identificação e *refresh*, e armazenamos essas informações no *localStorage* ou no *sessionStorage* dependendo da opção do usuário. Já podemos focar agora em construir nossa aplicação sem se preocupar com nosso processo de autenticação.

# **Conclusão**

Este artigo propõe o uso da biblioteca Amplify para permitir uma aplicação React se conectar a uma *User Pool* existente na nossa nuvem da AWS. Embora os exemplos e o código ser de uma aplicação React, nada impede que com leves adaptações a parte principal desse código seja usado em outras bibliotecas JavaScript, uma vez que essa parte principal está escrita em um JavaScript agnóstico a *frameworks.*

Um dos maiores objetivos deste artigo é mostrar uma aplicação que não depende do processo de *scaffolding* do Amplify que pode gerar uma dependência e que pode não ser compatível com o modelo de desenvolvimento de *software* de um time já existente, que pode preferir utilizar outros serviços, como o *Terraform* para adminstração da sua infraestrutura.

Tentei manter essa aplicação o mais simples possível, tentando focar didaticamente nos fluxos de autenticação de maneira a deixá-la facilmente adaptável. Espero ter cumprido meu objetivo, mas aceito sugestões de melhoria, na forma de comentário, ou na forma de *Pull Request* no repositório Github associado.

Pretendo fazer no mínimo dois novos posts: um sobre a integração desta aplicação desenvolvida a um *backend* em Spring Security; e um sobre autenticação através de provedores terceiros, como o *Facebook*. Devo editar este artigo para incluir os links quando eles estiverem concluídos, mas recomendo me seguir no twitter ( @maialc ) caso tenha interesse em ser avisado na data da publicação desses artigos.

Finalizo com alguns links úteis:

- my-react-cognito-app repository — https://github.com/lucasluc4/my-react-cognito-app
- Amplify React Documentation — https://aws-amplify.github.io/docs/js/react
- Amplify React Getting Started — https://aws-amplify.github.io/docs/js/start
- How to Easily Customize The Aws Amplify Authentication UI — https://blog.kylegalbraith.com/2018/11/29/how-to-easily-customize-the-aws-amplify-authentication-ui/
- Amplify auto snippet documentation — https://github.com/aws-amplify/amplify-js/blob/master/vscode/auto_snippet_documentation.md
- AWS Amplify | Custom Authentication — https://medium.com/@jordan.eckowitz/aws-amplify-custom-authentication-3876fb3f6712
- AWS Part 7: User Sign-Up, Sign-In, and Access Control Using Amplify, Cognito and React — https://www.youtube.com/watch?v=62kKbCmb7mIrite](https://medium.com/new-story?source=post_page-----33b5ac2e7448-----------------------------------)

