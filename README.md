# API de Autenticação e Autorização JWT (Emissão e Validação Interna)

Este projeto fornece uma API Spring Boot para autenticação e autorização baseada em JWT, gerando e validando tokens internamente. É ideal para arquiteturas de microsserviços que requerem autenticação centralizada.

---

## 📋 Sumário

* [Descrição](#descrição)
* [Tecnologias e Dependências](#tecnologias-e-dependências)
* [Configuração do Ambiente](#configuração-do-ambiente)
* [Implementação](#implementação)

  * [Entidade `User`](#entidade-user)
  * [Repositório `UserRepository`](#repositório-userrepository)
  * [Serviço `JwtService`](#serviço-jwtservice)
  * [Serviço `AuthService`](#serviço-authservice)
  * [Controlador `AuthController`](#controlador-authcontroller)
  * [Configuração de Segurança](#configuração-de-segurança)
  * [Controlador de Teste Protegido](#controlador-de-teste-protegido)
* [Testes com JUnit](#testes-com-junit)
* [Testes de Carga com JMeter](#testes-de-carga-com-jmeter)
* [Documentação com Swagger / OpenAPI](#documentação-com-swagger--openapi)
* [Entrega e Deploy](#entrega-e-deploy)
* [Recomendações Adicionais](#recomendações-adicionais)

---

## Descrição

Esta API é responsável por gerar e validar tokens JWT. Ao fazer login, o usuário recebe um token assinado internamente, que deve ser enviado em todas as requisições subsequentes. A segurança é configurada via Spring Security, sem estado de sessão.

---

## Tecnologias e Dependências

No `pom.xml`, inclua as dependências abaixo (Spring Boot 3.x ou superior):

* `spring-boot-starter-web`: APIs RESTful.
* `spring-boot-starter-security`: Autenticação e autorização.
* `spring-boot-starter-oauth2-resource-server`: Validação de JWT.
* `spring-boot-starter-data-jpa`: Persistência com JPA.
* `com.h2database:h2`: Banco em memória para dev/testes.
* `org.springdoc:springdoc-openapi-starter-webmvc-ui`: Swagger UI.
* `org.springframework.boot:spring-boot-devtools`: Hot reload.
* `org.projectlombok:lombok`: Getter/setter boilerplate.
* `spring-boot-starter-test`: JUnit 5, Mockito.

---

## Configuração do Ambiente

Edite `src/main/resources/application.yml`:

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
      path: /h2-console
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: update  # NÃO USE EM PRODUÇÃO!
    show-sql: true
    properties:
      hibernate:
        format_sql: true

development:
  devtools:
    restart.enabled: true
    livereload.enabled: true

jwt:
  secret: umaChaveSecretaMuitoLongaEComplexaParaAssinarTokensJWT
  expiration: 3600000  # 1 hora

springdoc:
  swagger-ui:
    path: /swagger-ui.html
    disable-swagger-default-url: true
  api-docs:
    path: /v3/api-docs
```

> **Atenção:** Em produção, gerencie `jwt.secret` como variável de ambiente ou via serviço de secrets.

---

## Implementação

### Entidade `User` (`User.java`)

Define o modelo de usuário com campos `id`, `username`, `password` e `role`.

### Repositório `UserRepository` (`UserRepository.java`)

Interface JPA para CRUD de `User` e busca por username.

### Serviço `JwtService` (`JwtService.java`)

Gera e valida tokens JWT internamente:

* `generateToken(username, role)`
* `validateToken(token)`
* `getUsernameFromToken(token)`
* `getAllClaimsFromToken(token)`

### Serviço `AuthService` (`AuthService.java`)

Valida credenciais e delega geração de token.

### Controlador `AuthController` (`AuthController.java`)

Endpoints:

* `POST /auth/login` → recebe `username` e `password`, retorna token.
* `POST /auth/validate` → recebe `token`, retorna status de validação.

### Configuração de Segurança (`SecurityConfig.java`)

* Desabilita CSRF.
* Stateless (sem sessão).
* Permite `/auth/login`, `/auth/validate`, console H2 e Swagger.
* Qualquer outro endpoint requer JWT válido.
* Bean `JwtDecoder` para validação automática.
* `PasswordEncoder` com BCrypt.
* `CommandLineRunner` para criar usuários iniciais `admin` e `user`.

### Controlador de Teste Protegido (`TestProtectedController.java`)

Exemplos de recursos protegidos:

* `GET /api/hello` → qualquer token válido.
* `GET /api/admin` → requer `ROLE_ADMIN`.

---

## Testes com JUnit

Os testes de integração (`AuthIntegrationTests.java`) cobrem:

1. Login com credenciais válidas/invalidas.
2. Acesso negado sem token.
3. Acesso permitido com token.
4. Restrições de `ROLE_ADMIN` em endpoint `/api/admin`.

Para executar:

```bash
mvn test
```

---

## Testes de Carga com JMeter

1. Baixe e descompacte o [Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi).
2. Crie um plano de teste (`.jmx`) simulando logins:

   * Thread Group: 200 usuários, ramp-up 20s, loops 10.
   * HTTP Request: POST `localhost:8080/auth/login` com parâmetros `username=admin`, `password=123456`.
3. Adicione Listeners: View Results Tree e Summary Report.
4. Salve o arquivo em `jmeter-tests/login_stress_test.jmx` e versione no Git.

---

## Documentação com Swagger / OpenAPI

A configuração em `SwaggerConfig.java` gera documentação interativa:

* Acesse: `http://localhost:8080/swagger-ui.html`
* Schemas, endpoints, modelos e segurança (bearerAuth).

---

## Entrega e Deploy

1. Clone o repositório:

   ```bash
   ```

git clone [https://github.com/SEU\_USUARIO/SEU\_REPOSITORIO.git](https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git)
cd SEU\_REPOSITORIO

````
2. Execute:
```bash
mvn spring-boot:run
````

3. Acesse:

   * API: `http://localhost:8080`
   * Console H2: `http://localhost:8080/h2-console`
   * Swagger UI: `http://localhost:8080/swagger-ui.html`
4. Execute testes e JMeter conforme descrito.
5. Para produção, crie um `Dockerfile` e utilize serviços como Render ou Railway.

---

## Recomendações Adicionais

* **Spring Boot Actuator**: adicione `spring-boot-starter-actuator` para métricas.
* **Prometheus & Grafana**: colete métricas do `/actuator/prometheus` e visualize no Grafana.
* **Deploy**: escreva instruções no README e configure CI/CD no GitHub Actions.

---

⭐ **Bom trabalho e bons estudos!**
