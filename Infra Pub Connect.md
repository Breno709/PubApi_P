# Arquitetura do Pub Connect
O projeto a seguir é uma apresentação de infraestrura cloud AWS e serviço backend ASPNet Core, ambos projetados e implementados por mim. 
Por ser um projeto de origem privada, não posso demonstrar detalhes como código fonte e afins. Ao invés disso, mostrarei uma visão geral da infraestrutura implementada, assim como detalhes sobre decisões tomadas no momento de projetar o sistema.

## Visão Geral
O projeto, identificado pelo codinome 'Pub Connect', visa desenvolver um serviço API REST que expõe endpoints, permitindo que os clientes automatizem processos em um ERP da mesma empresa. Até então, esse ERP não oferecia uma maneira de automatizar seus processos, limitando os clientes à interação por meio de interfaces gráficas (Desktop, Mobile, Web).

Diante dessa demanda, surgiu a iniciativa do projeto Pub Connect. Com ele, os clientes ganham a capacidade de criar seus próprios fluxos de automação por meio de requisições HTTP.

O projeto é dividido em dois serviços essenciais: O primeiro, denominado 'Pub Connect API', é responsável por processar todas as requisições HTTP, realizando validação das regras de negócio e operações no banco de dados. Por sua vez, o segundo serviço, denominado 'Auth Connect', atua como um provedor de identidades OAuth2, gerenciando o fluxo de autenticação e autorização. Ao final do processo, ele emite um token de autenticação, possibilitando que o cliente o utilize dentro do Pub Connect API.

## Modelo da Arquitetura
O diagrama de contexto a seguir mostra a visão geral do sistema e seus principais componentes, destacando as interações e fluxos que serão explicados logo abaixo.

![dia](https://github.com/Breno709/PubApi_Presentation/assets/8755602/91ae0de6-5440-4d4c-9dcc-515e8e2396de)

### Componentes e Tecnologias
Iniciando a leitura do fluxo do diagrama a partir do objeto 'Clientes', temos então, em resumo, os principais componentes

1. _Route 53 & CloudFront_: As 'portas de entrada' das requisições HTTP, responsável pelo direcionamento da rota até o serviço correspondente.
A decisão da utilização do cloudfront no intermédio do direcionamento das requisições HTTP, veio a partir da melhoria de desempenho e facilidade para a implementação de certificados SSL/TLS, uma vez que, no contexto da empresa, o certificado é gerenciado pelo AWS ACM e, portanto não é possível de implementá-lo diretamente nos serviços HTTP.

1. _Load Balancer & ASG_: Uma abordagem clássica recomendada pela própria AWS, e consequentemente, um padrão adotado pelo mercado é o uso do ELB e ASG em conjunto para garantir a escalabilidade dos serviços em momento de alta demanda. Como notado no diagrama, a implantação do autoscaling multi-az garante também a alta disponibilidade dos serviços, uma vez que, com essa configuração, eles podem ser implantados em máquinas EC2 em diferentes zonas de disponbilidade (az).

1.  Instâncias _EC2_: As instâncias EC2, a partir do gerenciamento do ECS, executam um container docker rodando a imagem dos serviços Pub Connect API e Auth Connect. Ambos seguem o mesmo padrão.
Apesar do aumento da complexidade da arquitetura a decisão da utilização de containers Docker, seu uso é vantajoso uma vez que, compilando as dependencias externas do projeto dentro de uma imagem Docker, garantimos consistência nos ambientes dev/prod e abrimos a possibildade de uma portabilidade mais simples para outros ambientes, caso se faça necessário.

1. Banco de dados _RDS_: Como é possível notar, existe uma diferença na infraestrutura de comunicação do banco ligado ao serviço Auth Connect para o serviço Pub Connect.
O serviço Auth Connect, por ser somente um provedor de identidade, é conectado dentro da mesma subnet ao banco responsável pelos dados.
Já o serviço Pub Connect conecta-se com seus bancos dados atravessando a VPC, pois seus bancos estão localizados em outra VPC.
Vale ressaltar que os bancos do Pub Connect são compartilhados com outros serviços, como o ERP citado na Visão Geral. A decisão da separação das VPCs foi uma decisão estratégica interna da empresa.
Como sugestão de melhoria, é possível utilizar o VPC Peering para conectar as duas VPCs e garantir um roteamento privado de tráfego dos dados.

1. _Git e Azure Pipelines_: O controle e versionamento de código fonte é realizado através do GIT, e o processo de entrega contínua é realizado pelo Azure Pipelines.

## Descrição dos Fluxos
### Fluxo Cliente - API
O primeiro fluxo destacado em vermelho, é o fluxo do cliente comunicando-se com a API. Vale ressaltar que esta conexão somente é realizada após a obtenção do token, através do serviço Auth Connect. 
Neste fluxo, o cliente realiza uma requisição HTTP informando os dados necessário para o método chamado, como headers e payload. A requisição entra pelo Route 53 e Cloudfront, passa por mecanismos de proteção contra ataques DDOS, SQL Injection e XSS fornecidos pelo WAF e Shield e, uma vez legitimada pelos mesmos, é então direcionada ao Load Balancer que distribui a requisição entre os serviços sendo executados nas instâncias EC2.
Após o serviço Pub Connect API processar a requisição e realizar suas devidas operações, ele então retorna uma resposta ao solicitante.

### Fluxo Cliente - Autenticação
O segundo fluxo destacado é, na verdade, a primeira ponta de conexão entre o cliente e o projeto como um todo. Através de solicitações HTTP, o cliente, informado seus devidos dados, realiza o fluxo de autenticação OAuth, obtendo então um token. 
A primeira parte do fluxo é similar ao descrito no Fluxo Cliente API (item 1). A ligeira diferença, como já mencionado anteriormente, é que o serviço Auth Connect conecta-se apenas com seu banco que contém as informações necessárias para autenticação do usuário.

### Fluxo CD
Por fim, o terceiro fluxo destacado é o fluxo de entrega contínua. Este mesmo não está diretamente relacionado às requisições do cliente, mas ajuda a melhorar a visão geral do ciclo de vida da aplicação.
Inicialmente, o desenvolvedor responsável pela implementação, melhoria ou correção de algum bug realiza o commit da revisão de código. Este commit, em seguida é avaliado por um outro desenvolvedor responsável pelo code review.
Uma vez revisado e aprovado, a alteração do código pode então ser processada pelo pipeline.
A pipeline compila, cria uma imagem Docker e envia esta imagem a um repositório ECR não mencionado no diagrama. Após o sucesso do envio, a pipeline comunica-se com o ECS indicando a necessidade de gerar uma nova versão dos serviços.
Então, finalmente o ECS implanta e disponibiliza esta nova versão dos serviços nas máquinas EC2.
Além disso, o código compilado disponibiliza uma documentação OpenAPI que é enviada pela pipeline para um serviço externo que expõe a referencia de API para os clientes.

## Considerações
A visão geral do projeto Pub Connect apresenta características da arquitetura do projeto, bem como expõe detalhes de implementação e motivação para tomadas de decisões.
Este é um projeto de média complexidade, onde atuei em todo o ecossistema de desenvolvimento do mesmo, desde o desenvolvimento do serviço backend Pub Connect API até a concepção do modelo de infraestrutura a ser seguido.
Como forma de simplificar a explicação, alguns detalhes e serviços não foram exibidos, como o relacionamento do ERP com os bancos de dados, e serviços auxiliares.
Atualmente este projeto encontra-se em fase de desenvolvimento e detalhes da implementação podem ser alterados ao longo do tempo.
