# permission_saas_config

Config Server do **Permission SaaS** (`config-server`, porta 8888), com Spring Cloud Config. Serve a
configuração de ambiente do profile `prod` ao `permission-service` e ao `audit-service`: endereços de
banco, do RabbitMQ e do `audit-service`, e log de SQL. Senhas e segredos não passam por aqui (ADR-012
no [log de ADRs](https://github.com/Permission-SaaS/permission_saas/blob/main/docs/ARCHITECTURE.md)).

Faz parte da organização [Permission-SaaS](https://github.com/Permission-SaaS). O repositório
[`permission_saas`](https://github.com/Permission-SaaS/permission_saas) é o guarda-chuva: reúne todos
os repositórios como submódulos e sobe o sistema inteiro pelo Docker Compose.

## Onde estão os arquivos de configuração

**Não neste repositório.** Eles ficam no `config-repo/` do guarda-chuva, ao lado do
`docker-compose.yml`, porque os dois descrevem a mesma topologia: os nomes `postgres`, `rabbitmq` e
`audit-service` dos arquivos são os serviços do Compose. Aqui fica só o servidor.

O backend é `native` (lido do disco), na pasta de `CONFIG_REPO_LOCATION`. O padrão é
`file:../config-repo`, que aponta para o `config-repo/` do guarda-chuva quando este repositório está
clonado como submódulo dele. No Compose, a pasta é montada em `/config-repo`, somente leitura.

## Rodar

```bash
./mvnw spring-boot:run                                              # dentro do guarda-chuva
CONFIG_REPO_LOCATION=file:/caminho/do/config-repo ./mvnw spring-boot:run   # fora dele
```

Os serviços só consultam o Config Server no profile `prod`. Rodando-os na máquina (profile `dev`), ele
não é necessário.

## `GET /{aplicação}/{profile}`

O endpoint padrão do Spring Cloud Config. Devolve a configuração de uma aplicação num profile. A
resposta lista as fontes (`propertySources`) que se aplicam, da mais específica para a mais geral;
quando duas definem a mesma propriedade, vale a primeira.

```json
{
  "name": "permission-service",
  "profiles": ["prod"],
  "propertySources": [
    { "name": "file:/config-repo/permission-service-prod.yml",
      "source": { "spring.datasource.url": "jdbc:postgresql://postgres:5432/permissions_saas",
                  "audit.service.url": "http://audit-service:8081" } },
    { "name": "file:/config-repo/application-prod.yml",
      "source": { "spring.jpa.show-sql": false } }
  ]
}
```

Um nome de aplicação sem arquivo próprio não dá erro: devolve só as fontes gerais
(`application-<profile>.yml`).

```bash
curl http://localhost:8888/permission-service/prod
curl http://localhost:8888/audit-service/prod
```

## Build

```bash
./mvnw clean package
```
