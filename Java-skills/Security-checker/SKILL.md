---
name: "Java Code Security Checker (Spring Boot & Quarkus)"
description: "Realiza validações de segurança e procura por vulnerabilidades em código Java, com foco especializado em aplicações Spring Boot e Quarkus."
---

# Java Code Security Checker

Este skill capacita o assistente a analisar códigos Java (focando em frameworks modernos como Spring Boot e Quarkus) em busca de vulnerabilidades de segurança, más práticas e problemas de configuração.

## Diretrizes de Análise de Segurança

Ao analisar repositórios ou trechos de código Java (Spring Boot / Quarkus) em busca de problemas de segurança, siga as seguintes diretrizes ativamente:

### 1. Injeção (SQL, NoSQL, OS Command, LDAP)
- **Spring Boot (JPA/Hibernate):** Procure por uso de concatenação de strings em consultas SQL (`@Query`, `EntityManager.createQuery()`). Recomende o uso de *Prepared Statements* ou *Named Parameters* (`:paramName`).
- **Quarkus (Hibernate ORM / Panache):** Verifique métodos que aceitam strings puras (concatenação) em `.find()`, `.update()`, ou `.delete()`.
- **Validação Criteriosa:** Garanta que todos os dados fornecidos pelo usuário sejam validados (usando `@Valid`, `@Validated` e restrições do `jakarta.validation.constraints`).

### 2. Autenticação e Autorização (Broken Access Control)
- **Spring Security:** 
  - Verifique as configurações no `SecurityFilterChain`. Endpoints sensíveis estão abertos (`permitAll`) ou usando curingas equivocadamente?
  - O CSRF está desabilitado desnecessariamente (`csrf().disable()`) em aplicações que contam com sessão de usuário baseada em cookies?
  - Analise o uso correto de `@PreAuthorize`, `@Secured`, ou `@RolesAllowed`. Os papéis (roles) estão sendo informados adequadamente?
- **Quarkus (Security):**
  - Verifique o uso de anotações como `@RolesAllowed`, `@PermitAll`, `@Authenticated` nos endpoints JAX-RS/RESTEasy. E tenha atenção a endpoints públicos sem intenção.
  - Revise as configurações no `application.properties` (ex: `quarkus.http.auth.permission.*`). O controle de acesso baseado em URLs base cobre de fato o que importa?
- **Geral (IDOR):** Procure por IDs previsíveis em URLs (Insecure Direct Object Reference). O usuário consegue acessar ou alterar dados de outros usuários apenas manipulando um ID (uuid x long) na API sem a devida checagem de propriedade pelo backend?

### 3. Gerenciamento de Segredos e Dados Sensíveis
- **Credenciais Fixas no Código (Hardcoded Secrets):** Busque por senhas, tokens de API, chaves de criptografia (JWT, etc) e credenciais de banco de dados diretamente no código, classes de configuração ou em arquivos de propriedade (`application.properties`, `application.yml`).
- **Exposição de Dados Sensíveis (Data Exposure):**
  - A aplicação loga informações sensíveis (senhas, emails, dados de usuários)? Busque coisas como `log.info("Dados do usuário: {}", user)`.
  - Entidades JPA estão sendo retornadas diretamente nas requisições da API (faltando DTOs protetores)? Isso vaza frequentemente metadados internos e hashes de credenciais.
- Cobre o uso de variáveis de ambiente para a injeção via CI/CD, cofres corporativos (Vault), etc.

### 4. Configuração de Criptografia (Cryptographic Failures)
- **Hashing de Senhas:** Valide se algoritmos legados (MD5, SHA-1, SHA-256 base) estão em uso. Recomende hashes modernos adaptativos para senhas como BCrypt (`BCryptPasswordEncoder`, `BcryptUtil`), Argon2 ou PBKDF2.
- **Criptografia Simétrica (AES, etc):** O código usa modos defasados (como `ECB`) ou constrói as chaves de maneira insegura? `CBC` sempre precisa de IV.
- **Validação de TLS/SSL:** Clientes HTTP vêm configurados para "Trust All" (`TrustManager` falso) aceitando qualquer certificado SSL e abertos a ataque Man in the Middle (MitM)?

### 5. Configurações Inseguras (Security Misconfiguration & XXE)
- **CORS (Cross-Origin Resource Sharing):** Procure por falhas de "Wildcards" nas origens. Spring (`@CrossOrigin("*")` ou CORS Configurations permitindo tudo) ou Quarkus (`quarkus.http.cors.origins=*`). Isso expõe a APIs ao roubo de sessão num navegador.
- **Processamento XML (XXE):** Ao analisar endpoints ou rotinas que processam XML (Jackson XML, DocumentBuilderFactory, etc), avalie se ataques XXE são possíveis garantindo que a resolução de "External Entities" (DTDs) esteja explicitamente desativada.
- **Actuators & Metrics:**
  - **Spring Boot Actuator:** Certifique-se de que endpoints sensíveis da JVM/Infra (`/actuator/env`, `/actuator/heapdump`, `/actuator/prometheus`) não estejam expostos publicamente ou sem restrições fortes.
  - **Quarkus Health/Metrics:** Verifique de que forma `/q/health` ou `/q/metrics` expõe sua stack, controlando o roteamento se estiver público na web.

### 6. Desserialização Insegura de Objetos
- Busque implementações onde arquivos, payloads JSON/XML complexas contendo declaração de tipos/classes explícitos estão sendo submetidas num request causando Remote Code Execution. (Ex: velhas classes aceitando Object ou `@JsonTypeInfo` inseguro).

### 7. Verificando o "Supply Chain"
- Examine por alto nos arquivos construtores (`pom.xml`, `build.gradle`) o uso de bibliotecas datadas que possuem CVEs de alto impacto para o ecossistema e sugira atualização (ex: jackson/log4j desatualizado). 

---

## Fluxo Orientado de Ação do Skill

Quando você, como assistente, aplicar ativamente o "Security Checker", execute passo-a-passo:
1. Analise o segmento alvo (arquivo local, trecho de código colado, projeto).
2. Processe mentalmente seu código com cada um dos 7 pontos base desta skill (Injeção, Autenticação, Segredos, Criptografia, Config, etc).
3. Caso detecte uma vulnerabilidade/má prática, faça uma pontuação relatando sua existência e impacto estimado (Severidade/CVE se aplicável).
4. **Resolução:** Forneça o snippet correspondente de como consertar especificamente na linguagem e ambiente (Spring/Quarkus). Não entregue relatórios genéricos, priorize o conserto do que você encontrou em código.
