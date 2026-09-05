Workshop Spring Boot — API de pedidos

API REST para gerenciamento de usuários e consulta de pedidos, produtos e categorias, desenvolvida com Java 25, Spring Boot 4.1.0 e Spring Data JPA.

Projeto de estudo voltado à construção de um backend com persistência relacional, organização em camadas e tratamento de exceções. O domínio representa um cenário de vendas: cada pedido pertence a um cliente, reúne itens com quantidade e preço de venda e pode possuir um pagamento associado.

Funcionalidades

Cadastro, listagem, consulta, atualização e exclusão de usuários.

Atualização dos campos name, email e phone por meio de um método que controla os dados copiados para a entidade.

Consulta de pedidos com cliente, itens, pagamento e valor total.

Consulta de produtos e suas categorias.

Consulta de categorias.

Cálculo do subtotal de cada item e do total do pedido.

Respostas padronizadas para usuário não encontrado na consulta e na atualização, além de violações de integridade tratadas na exclusão.

Banco H2 em memória com dados iniciais para demonstração local.

Tecnologias

Tecnologia

Aplicação no projeto

Java 25

Linguagem e versão configurada para compilação

Spring Boot 4.1.0

Configuração e inicialização da aplicação

Spring Web MVC

Endpoints REST e respostas HTTP

Spring Data JPA / Hibernate

Repositórios, mapeamento das entidades e persistência

H2 Database

Banco em memória utilizado pelo perfil test

Jackson

Conversão entre objetos Java e JSON

Maven Wrapper

Build e execução; distribuição Maven 3.9.16 configurada no repositório

Spring Boot Test / JUnit Jupiter

Teste de carregamento do contexto da aplicação

O driver PostgreSQL também está declarado no pom.xml. A configuração disponível para execução local utiliza H2; um perfil próprio para PostgreSQL pode ser acrescentado em uma próxima etapa.

Arquitetura

As requisições entram pelos resources, que recebem os dados e constroem as respostas HTTP. Os services coordenam as operações e traduzem exceções específicas. Os repositories acessam a persistência por meio do Spring Data JPA, enquanto as entities representam o domínio e seus relacionamentos.

Pacote

Responsabilidade

resources

Controllers REST de usuários, pedidos, produtos e categorias

services

Operações da aplicação e controle dos campos atualizados

repositories

Interfaces de acesso a dados

entities

Entidades, associações, enum e chave composta

resources/exceptions

Tratamento HTTP das exceções e estrutura StandardError

services/exceptions

Exceções próprias da aplicação

config

Carga inicial de dados do perfil test

Modelo de domínio

erDiagram
    direction TB
    User ||--o{ Order : realiza
    Order ||--o{ OrderItem : contem
    Product ||--o{ OrderItem : compoe
    Product }o--o{ Category : pertence
    Order ||--o| Payment : possui

Algumas decisões de modelagem presentes no código:

Itens de pedido: OrderItem utiliza a chave composta OrderItemPK, formada por pedido e produto. A quantidade e o preço de venda pertencem ao item.

Preço registrado no pedido: OrderItem.price guarda o valor utilizado na venda, permitindo calcular o pedido independentemente de alterações posteriores no preço do catálogo.

Totais calculados: getSubTotal() multiplica preço por quantidade; getTotal() soma os subtotais. Os valores aparecem no JSON pelos respectivos getters.

Categorias: produtos e categorias possuem uma relação muitos para muitos, persistida em tb_product_category.

Pagamento: Payment utiliza @MapsId para compartilhar o identificador com o pedido. A associação admite pedidos que ainda não possuem pagamento.

Serialização: @JsonIgnore controla referências de retorno nos relacionamentos para evitar ciclos no JSON.

Os estados do pedido são definidos por OrderStatus:

Código persistido

Estado

1

WAITING_PAYMENT

2

PAID

3

SHIPPED

4

DELIVERED

5

CANCELED

Como executar

Pré-requisitos

JDK 25 instalado, com JAVA_HOME e PATH apontando para essa versão.

Git instalado.

Acesso à internet na primeira execução para baixar o Maven e as dependências.

O Maven Wrapper está incluído no repositório. A demonstração local utiliza H2 em memória e dispensa a instalação de um servidor de banco de dados.

1. Clonar o repositório

git clone https://github.com/pedrohsantosdev/whorkshop-springboot.git
cd whorkshop-springboot

2. Iniciar a aplicação

Windows — PowerShell ou terminal do IntelliJ IDEA:

.\mvnw.cmd spring-boot:run

Linux / macOS:

chmod +x mvnw
./mvnw spring-boot:run

O perfil test já está ativo em application.properties. Ele utiliza as configurações de application-test.properties e executa a carga de dados de TestConfig.

Após a inicialização, acesse a listagem de usuários. A URL base da API é http://localhost:8080.

3. Explorar os dados iniciais

A carga inicial inclui 2 usuários, 3 categorias, 5 produtos, 3 pedidos, 4 itens de pedido e 1 pagamento. Esses registros permitem explorar as associações e os cálculos logo após iniciar a aplicação.

O H2 armazena os dados em memória. As alterações feitas durante a demonstração são perdidas ao encerrar a aplicação; a carga inicial é executada novamente na próxima inicialização.

Para consultar as tabelas, abra o console H2 enquanto a aplicação estiver em execução:

Campo

Valor

Driver Class

org.h2.Driver

JDBC URL

jdbc:h2:mem:testdb

User Name

sa

Password

Deixar em branco

Endpoints

Os códigos abaixo representam o retorno das operações concluídas com sucesso.

Método

Rota

Operação

Status

GET

/users

Listar usuários

200 OK

GET

/users/{id}

Consultar um usuário

200 OK

POST

/users

Cadastrar um usuário

201 Created

PUT

/users/{id}

Atualizar nome, email e telefone

200 OK

DELETE

/users/{id}

Excluir um usuário

204 No Content

GET

/orders

Listar pedidos

200 OK

GET

/orders/{id}

Consultar um pedido

200 OK

GET

/products

Listar produtos

200 OK

GET

/products/{id}

Consultar um produto

200 OK

GET

/categories

Listar categorias

200 OK

GET

/categories/{id}

Consultar uma categoria

200 OK

Os itens e o pagamento são consultados pela representação dos pedidos.

Exemplos de uso

Utilize a URL base http://localhost:8080 no Postman, Insomnia ou outro cliente HTTP.

Cadastrar um usuário

POST /users
Content-Type: application/json

{
  "name": "Cliente Exemplo",
  "email": "cliente@example.com",
  "phone": "11999990000",
  "password": "senha-ficticia"
}

O retorno esperado é 201 Created, com o usuário cadastrado no corpo e o endereço do recurso no cabeçalho Location. Utilize o id devolvido nas próximas operações; ele é gerado pelo banco.

Atualizar o usuário criado

Substitua {id} pelo identificador recebido no cadastro:

PUT /users/{id}
Content-Type: application/json

{
  "name": "Cliente Atualizado",
  "email": "atualizado@example.com",
  "phone": "11988880000"
}

A resposta esperada é 200 OK, com o usuário atualizado. O método updateData() copia somente esses três campos. Envie os três valores, pois um campo ausente pode ser copiado como null na implementação atual.

Consultar um pedido

GET /orders/1

Com a carga inicial, esse pedido contém duas unidades de The Lord of the Rings, a 90.50 cada, e uma unidade de Macbook Pro, a 1250.00. O total calculado é 1431.00, retornado no campo total. A representação também inclui o cliente, os itens e o pagamento.

Excluir o usuário criado

DELETE /users/{id}

Para um usuário sem pedidos associados, a exclusão retorna 204 No Content, com corpo vazio.

Tratamento de erros

O ResourceExceptionHandler, anotado com @ControllerAdvice, converte as exceções próprias da aplicação em respostas com a estrutura StandardError.

Situação tratada

Exceção da aplicação

Resposta

Usuário inexistente na consulta por ID ou na atualização

ResourceNotFoundException

404 Not Found

Violação de integridade ao excluir um usuário associado a pedidos

DatabaseException

400 Bad Request

Na atualização, o service captura EntityNotFoundException ao utilizar a referência obtida por getReferenceById() e lança ResourceNotFoundException(id). O handler reaproveita o tratamento de 404 já existente.

Exemplo ilustrativo de resposta para GET /users/999, considerando que esse usuário não exista:

{
  "timestamp": "2026-09-05T15:00:00Z",
  "status": 404,
  "error": "Resource not found",
  "message": "Resource not found. Id 999",
  "path": "/users/999"
}

O horário varia conforme a requisição. O código HTTP é definido pelo ResponseEntity; o campo status também informa esse valor no corpo da resposta.

Testes e verificação local

Para executar os testes existentes:

Windows:

.\mvnw.cmd test

Linux / macOS:

./mvnw test

O teste automatizado atual, CourseApplicationTests.contextLoads(), verifica o carregamento do contexto Spring. A cobertura dos endpoints e das regras de negócio é uma possibilidade de evolução.

Os cenários abaixo podem ser verificados manualmente no cliente HTTP:

Cenário

Resultado esperado

Cadastrar um usuário

201, ID gerado e cabeçalho Location

Atualizar e consultar o usuário criado

200, com os campos alterados

Consultar ou atualizar um usuário inexistente

404, com corpo StandardError

Excluir o usuário criado, sem pedidos

204, com corpo vazio

Tentar excluir o usuário 1 da carga inicial, que possui pedidos

400, com erro de banco padronizado

Consultar o pedido 1 da carga inicial

Campo total com valor 1431.0

Estado atual e possibilidades de evolução

O projeto está em desenvolvimento como aplicação de estudo. Nesta versão, a entidade User é utilizada diretamente nas respostas, incluindo o campo password, que também é armazenado sem hashing. Os exemplos e a demonstração devem utilizar dados fictícios.

Possíveis próximos passos:

Introduzir DTOs para controlar os dados de entrada e saída e retirar senhas das respostas.

Acrescentar validação dos dados, hashing de senhas, autenticação e autorização.

Ampliar os testes de integração e padronizar o tratamento de recursos inexistentes nos demais endpoints.

Configurar um perfil PostgreSQL e migrações do banco de dados.

Utilizar BigDecimal para valores monetários e definir transações na camada de serviço.

Autor

Desenvolvido por Pedro Henrique, como parte dos estudos de desenvolvimento backend com Java.

Perfil no GitHub · Repositório do projeto
