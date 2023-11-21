# Projeto Pub Connect
A seguir, apresento um projeto que abrange a infraestrutura na nuvem da AWS e o serviço backend desenvolvido em ASP.NET Core. 
Este projeto foi concebido e implementado por mim para atender uma demanda de um sistema ERP. Dado o seu caráter privado, não é possível compartilhar detalhes específicos, como código-fonte. Em vez disso, fornecerei uma visão abrangente da infraestrutura implementada, destacando as decisões tomadas durante o processo de design do sistema.

## Visão Geral
O projeto, identificado pelo codinome 'Pub Connect', visa desenvolver um serviço API REST que expõe endpoints, permitindo que os clientes automatizem processos em um ERP da mesma empresa. Até então, esse ERP não oferecia uma maneira de automatizar seus processos, limitando os clientes à interação por meio de interfaces gráficas (Desktop, Mobile, Web).

Diante dessa demanda, surgiu a iniciativa do projeto Pub Connect. Com ele, os clientes ganham a capacidade de criar seus próprios fluxos de automação por meio de requisições HTTP.

O projeto é dividido em dois serviços essenciais: O primeiro, denominado 'Pub Connect API', é responsável por processar todas as requisições HTTP, realizando validação das regras de negócio e operações no banco de dados. Por sua vez, o segundo serviço, denominado 'Auth Connect', atua como um provedor de identidades OAuth2, gerenciando o fluxo de autenticação e autorização. Ao final do processo, ele emite um token de autenticação, possibilitando que o cliente o utilize dentro do Pub Connect API.

* [Apresentação da Arquitetura Backend .NET](/Infra%20Pub%20Connect.md)
* [Apresentação da Infraestrutura Cloud AWS](/Backend%20Pub%20Connect.md)
