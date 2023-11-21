## Padrão de Arquitetura Backend - Pub Connect API
O projeto 'Pub Connect' incorpora um serviço ASP.NET Core 7.0 denominado 'Pub Connect API', responsável pela manipulação nos bancos de dados utilizando as regras de negócio definidas no sistema. Este componente do projeto recebe e processa as solicitações HTTP originadas de nossos clientes.

## Visão Geral
O projeto, identificado pelo codinome 'Pub Connect', tem como objetivo principal o desenvolvimento de um serviço API REST. Este serviço expõe endpoints para que os clientes possam automatizar processos em um ERP desenvolvido pela mesma empresa. Anteriormente, este ERP não oferecia uma forma de automação dos processos, limitando os clientes à interação por meio de interfaces gráficas (Desktop, Mobile, Web). Surgiu então a iniciativa do projeto Pub Connect, possibilitando aos clientes criar seus próprios fluxos de automação por meio de requisições HTTP.

O projeto é dividido em dois serviços: o primeiro, 'Pub Connect API', é responsável pelo processamento de todas as requisições HTTP, incluindo validação das regras de negócio e operações no banco de dados. O segundo serviço, chamado 'Auth Connect', é um provedor de identidades OAuth2 encarregado do fluxo de autenticação e autorização. Esse serviço, ao finalizar, produz um token de autenticação para que o cliente possa utilizar dentro do Pub Connect API.

## Padrões de Projeto
### Arquitetura do Projeto

O Pub Connect API utiliza o padrão de arquitetura [_Vertical Slice Architecture_](https://www.jimmybogard.com/vertical-slice-architecture/). Esse padrão propõe organizar o código-fonte da aplicação em fatias funcionais, em oposição à organização por camadas horizontais tradicionais (como apresentação, lógica de negócios e persistência). 
Em vez de dividir a aplicação por responsabilidades tecnológicas, a abordagem de fatia vertical divide por funcionalidades específicas do negócio.

![n_layer (3)](https://github.com/Breno709/PubApi_P/assets/8755602/40c48f66-c554-45ec-8a42-f51f908f5ed4)

Como é possível observar as fatias verticais cortam diretamente por meio dessas camadas, criando um sistema que é particionado tanto horizontal quanto verticalmente.

A organização interna dos arquivos e pastas do projeto utiliza o padrão CX (Code Excellence) desenvolvido pela [DoFactory](https://www.dofactory.com/).
O padrão simplifica o acesso do desenvolvedor aos arquivos do projeto, através de convenções de nomenclatura e organização dos diretórios.

### CQRS Com MediatR
O uso do padrão CQRS (Command Query Responsibility Segregation) com a biblioteca MediatR foi motivado pelos seguintes fatores:
* **Manutenibilidade**: A separação de comandos e consultas torna o código mais modular e fácil de entender, facilitando a manutenção e a introdução de novos recursos.
* **Centralização do Processo de Comando**: Os desenvolvedores podem se concentrar na modelagem do domínio sem se preocupar com as complexidades de gerenciamento de estados e interações entre comandos e consultas.
* **Testabilidade**: A utilização dos handlers do CQRS facilita a implementação de testes unitários.

**Referencias**:

[CQRS Pattern - Microsoft](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)

[Implementar Camada de Aplicativo - Microsoft](https://learn.microsoft.com/pt-br/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-application-layer-implementation-web-api)

### Fluente Validation
O Fluent Validation foi integrado ao MediatR para realizar as validações dos comandos CQRS. A utilização do Fluent Validation separa a validação das propriedades do comando das validações de regra de negócio realizadas pelos Handlers, tornando o código mais objetivo. Além disso, garante uma validação "fail-fast", impedindo o acesso ao banco de dados caso o objeto esperado pelo handler não esteja com as propriedades preenchidas corretamente.

### Manipulação e Persistência de Dados
Para realizar a manipulação e persistência dos dados, foi utilizado o Entity Framework Core com o AutoMapper. 
Dessa forma, facilitamos os mapeamentos entre DTOs e entidades do banco de dados, além de simplificar a manutenção e evolução do código.

### Segurança
Alidado ao serviço 'Auth Connect' do projeto, o Pub Connect API utiliza o padrão Oauth2 para autenticação e autorização de acesso dos clientes.
Os fluxos OAuth2 suportados são [Authorization Code](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow) e [Client Credentials](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow)

### Integração com Serviços AWS
Pub Connect API conta com integração ao [Elasticache Redis](https://aws.amazon.com/pt/elasticache/redis/) e [Secrets manager](https://aws.amazon.com/pt/secrets-manager/), ambos serviços AWS.

O Redis fornece armazenamento em cache na memória, acelerando o acesso a dados frequentemente utilizados e reduzindo a latência em comparação com operações de leitura no banco de dados principal.

Já o Secrets Manager facilita a gestão centralizada de credenciais, como senhas e chaves de API, simplificando a administração e reduzindo a probabilidade de vazamentos de informações sensíveis.
Vale ressaltar uma preocupação adicional onde desde o dia 0 de desenvolvimento do projeto, nenhum dado sensível foi exposto ao código fonte graças a utilização do Secrets Manager desde o começo da implementação do serviço.

### Versionamento
Como requisito do projeto, era necessário que o mesmo tivesse um mecanismo interno de versionamento das apis, onde fosse possível gerenciar diferentes versões de api dentro do mesmo serviço.
Para o cumprimento deste requisito foi utilizado [Asp.Versioning](https://www.nuget.org/packages/Asp.Versioning.Mvc/).

### Testes Unitários
O serviço backend Pub Connect API utiliza o [xUnit](https://github.com/xunit/xunit) para a realização dos testes unitários.

### Rate Limiting Pattern
Utilizando do padrão [Rate Limiting](https://learn.microsoft.com/en-us/azure/architecture/patterns/rate-limiting-pattern) o serviço fica protegido contra abusos de usuários invasivos.

### Health Endpoint Monitoring pattern
Através do [padrão de verificação de saúde da aplicação](https://learn.microsoft.com/en-us/azure/architecture/patterns/health-endpoint-monitoring), é possível monitorar pontos funcionais da aplicação, bem como suas integrações com outros serviços e banco de dados.

### Especificação de API
Para definir a API, o projeto utiliza o Swagger, agora conhecido como [OpenAPI](https://github.com/microsoft/OpenAPI.NET). O Open API permite descrever de forma padronizada os endpoints, operações, parâmetros e modelos de dados da API. 
Essa abordagem baseada em padrões promove uma documentação consistente, facilitando a compreensão da API por desenvolvedores e facilitadores de integração.

### Estrutura de Container
O projeto suporta execução dentro de containers, adicionando flexibilidade e portabilidade à sua infraestrutura. 
Ao adotar o Docker como tecnologia de container, o projeto se beneficia de um empacotamento que inclui não apenas o código-fonte, mas também todas as dependências e configurações necessárias para a execução da aplicação.

## Considerações
A utilização de padrões de projeto como CQRS, Fluent Validation, e Vertical Slice Architecture contribui significativamente para a manutenibilidade, modularidade e testabilidade do código. 
A escolha de tecnologias como Entity Framework Core com AutoMapper para manipulação de dados e a integração com serviços AWS, incluindo Elasticache Redis e Secrets Manager, demonstram a busca incessante por desempenho, segurança e boas práticas de desenvolvimento.

Além disso, a adoção de práticas de versionamento, testes unitários, padrões de rate limiting e health endpoint monitoring evidenciam a atenção aos aspectos operacionais e de segurança do serviço. 
A flexibilidade proporcionada pela execução em containers, com o suporte do Docker, adiciona uma camada adicional de portabilidade e facilita a escalabilidade.

A documentação padronizada da API por meio do OpenAPI (Swagger) não apenas atende às necessidades de desenvolvedores, mas também facilita a integração de terceiros e promove uma abordagem centrada no usuário.

Por fim, o projeto Pub Connect API reforça a importância da coesão do código, uma vez que a alta coesão é um dos pilares essenciais para a compreensão e manutenção sustentável do sistema. 
A aplicação consistente de padrões e práticas reflete o comprometimento com a qualidade e a evolução contínua do serviço, proporcionando uma base sólida para futuras melhorias e expansões.
