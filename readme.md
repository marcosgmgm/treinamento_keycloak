# Treinamento Keycloak

## Subindo keycloak
`docker-compose up`

### Desabilitando sslRequired
Desativar apenas para execução local. Para produção, não editar essa propriedade.

**Realizar login interno**  
O comando para realizar o login é:  
`/opt/jboss/keycloak/bin/kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin --password changeit`

Executar automaticamente no docker
`docker ps | grep treinamento | grep keycloak_1 | awk '{print $1}' | xargs -i docker exec '{}' /opt/jboss/keycloak/bin/kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin --password changeit`

**Atualizar o server para desativar sslRequired**
O comando para desativar o sslRequired é
`docker ps | grep treinamento | grep keycloak_1 | awk '{print $1}' | xargs -i docker exec '{}' /opt/jboss/keycloak/bin/kcadm.sh update realms/master -s sslRequired=NONE`

## Setup inicial
### Criando client realwave_iam no realm master
Este client será o _usuário_ admin que o **realmwave-iam** irá utilizar para fazer a manutenção do keycloak.

1. Criar client `realwave_iam`
2. Trocar **Access Type** para **confidential**
3. Desativar **Standard Flow Enabled**
4. Desativar **Direct Access Grants Enabled**
5. Ativar **Service Accounts Enabled** (isso cria um usuário pro IAM por debaixo dos panos e permite ter roles)
6. Adicionar a role de admin

### Criando o realm Zup
Mesmo que o Keycloak seja on-premise para um usuário, o realm Zup deverá ser criado.

Para tal, seguir o tutorial do confluence [https://zupjira.atlassian.net/wiki/spaces/RW/pages/99319992?search_id=0985c1ae-efe1-4edd-aa16-2f988193027f](https://zupjira.atlassian.net/wiki/spaces/RW/pages/99319992?search_id=0985c1ae-efe1-4edd-aa16-2f988193027f)  

**No final da instalação, alterar o Require SSL do realm Zup para none apenas para este treinamento. Em produção, deixar external requests.**  

**Alterar a tela de login para o do realwave**  

## Rodando o IAM
```bash
git clone git@github.com:ZupIT/realwave-iam.git
cd realwave-iam
mvn clean install -DskipTests
docker-compose up -d postgres
cd rw-iam-application
mvn spring-boot:run -Drun-arguments="keycloak.api.credentials.secret=120a2c7b-d8ab-4a08-a87c-bde7fc42b9f0,keycloak.credentials.secret=b730743f-9833-4281-8c83-7e9ff320b84c"
```

## Administrando IAM
1. Criar usuário para o iam. Normalmente eu coloco como **rw.iam.ambiente**, por exemplo, **rw.iam.dev**.
2. Em **Credentials**, colocar uma senha. Desmarcar **Temporary**.
3. Adicionar todas as roles do client **realwave_iam**.

**Fazer o Download da collection do realwave-iam no realwave-collections**
```bash
git clone git@github.com:ZupIT/realwave-collections.git
```
Importar a collection `Realwave IAM API.postman_collection.json` da pasta **iam** e também importar o environment `realwave-iam-local.postman_environment.json`.

### Criando Roles
Roles não estão associadas e nenhum módulo, elas são entidades livres. Entretanto, é comum prefixa-la com a sigla do módulo para sabermos quem é o dono da role.

### Criando Módulo
Será criado um client em todos os realms (no caso por enquanto é apenas Zup).

É possível criar um módulo apenas para um realm. Isso é normalmente utilizado para criar `extensions`, impossibilitando assim que o extensions tenha acesso a dados de outro realm.

### Adicionar Role a um Módulo
As roles adicionadas serão replicadas em todos os realms. Por enquanto não é possível dar roles customizadas para um realm. Ou tem em todos, ou em nenhum.

### Criando Application
Uma aplicação é criada apenas para um realm específico. Não existem aplicações globais.

### Criando um Realm
Quando um realm é criado, todas as roles e módulos globais são replicados. Não são consideradas globais roles e módulos que foram criadas para um realm específico.

Quando um realm é criado, não são criados os clientes `realwave_iam`, `realwave_iam_ui` e nem os clientes de login, como o `backoffice`, `crm`, `sfm`, `reports`, etc.