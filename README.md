# Workshop Spring Boot

API REST desenvolvida durante meus estudos de backend Java, com gerenciamento de usuários e consulta de pedidos, produtos e categorias.

## Tecnologias

Java 25 · Spring Boot 4.1.0 · Spring Data JPA / Hibernate · H2 · Maven

## Funcionalidades

- Cadastro, consulta, atualização e exclusão de usuários.
- Consulta de pedidos, produtos e categorias.
- Cálculo de subtotais dos itens e total dos pedidos.
- Tratamento de exceções com respostas HTTP padronizadas no fluxo de usuários.

## Como executar

Requisitos: **JDK 25** e **Git**.

```bash
git clone https://github.com/pedrohsantosdev/whorkshop-springboot.git
cd whorkshop-springboot
```

**Windows:**

```powershell
.\mvnw.cmd spring-boot:run
```

**Linux / macOS:**

```bash
chmod +x mvnw
./mvnw spring-boot:run
```

Acesse [a API de usuários](http://localhost:8080/users). O perfil `test` utiliza H2 em memória com dados iniciais. Os dados são reiniciados ao encerrar e executar novamente a aplicação.

Projeto de estudo para demonstração local. Utilize dados fictícios.

## Autor

**Pedro Henrique dos Santos Silva**

[GitHub](https://github.com/pedrohsantosdev) · [Email](mailto:pedrohenrique.santos.dev@gmail.com)
