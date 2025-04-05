# Criação de um cluster MongoDB com Docker
Passo a passo para a configuraração de um cluster MongoDB utilizando Docker. Em seguida realizamos testes de replicação de dados através do MongoDB.

## Pré-requisitos
- Docker.
- MongoDB.

## Comandos
### Instalação de um cluster no MongoDB utilizando o Docker e criação de nós
#### Criação de uma rede Docker
```bash
docker network create mongoCluster
```
#### Criação dos containers
##### `mongo1`
```bash
docker run -d --rm -p 27018:27017 --name mongo1 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo1
```
##### `mongo2`
```bash
docker run -d --rm -p 27019:27017 --name mongo2 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo2
```
##### `mongo3`
```bash
docker run -d --rm -p 27020:27017 --name mongo3 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo3
```
##### `mongo4`
```bash
docker run -d --rm -p 27021:27017 --name mongo4 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo4
```
##### `mongo5`
```bash
docker run -d --rm -p 27022:27017 --name mongo5 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo5
```

#### Acesso ao container `mongo1`
```bash
docker exec -it mongo1 mongosh
```

#### Verificação dos status do ReplicaSet
```javascript
db.runCommand({hello:1})
```
#### Inicialização o ReplicaSet
```javascript
rs.initiate({
_id: "myReplicaSet",
members: [
{_id:0, host: "mongo1:27017"},
{_id:1, host: "mongo2:27017"},
{_id:2, host: "mongo3:27017"},
{_id:3, host: "mongo4:27017"},
{_id:4, host: "mongo5:27017"},
]
})
```
#### Verificação dos status do ReplicaSet
```javascript
rs.status()
```

### Funcionamento do cluster</h3>
Copiar os seguintes endereços e colar no campo URI MongoDB Compass para criar cada conexão:
- mongodb://127.0.0.1:27018/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
- mongodb://127.0.0.1:27019/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
- mongodb://127.0.0.1:27020/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
- mongodb://127.0.0.1:27021/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
- mongodb://127.0.0.1:27022/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2

#### Verificação de qual nó é primário:
```javascript
rs.isMaster().primary
```

#### Acesso à collection alunos e inserção de dados
```javascript
use alunos
```
```javascript 
db.alunos.insertOne({ra: 1, nome: "Joao"})
```
  ```javascript 
  db.alunos.insertOne({ra: 2, nome: "Richard"})
```
  ```javascript 
  db.alunos.insertOne({ra: 3, nome: "Diego"})
```
  ```javascript 
  db.alunos.insertOne({ra: 4, nome: "Vinicius"})
```
#### Consulta dos dados 
```javascript
db.alunos.find()
```
### Tese 1: Queda de nó secundário

#### Parar nó secundário `mongo2`
Clicar no ícone "Stop" do container 'mongo2'

#### Verificar se o nó secundário `mongo2` caiu
```bash
docker exec -it mongo2 mongosh
```

#### Verificar se o nó primário `mongo1` caiu
```bash
docker exec -it mongo1 mongosh
```

#### Verificar status do cluster após a queda
```javascript
rs.status()
```

#### Consulta de dados
```bash
db.alunos.find()
```

#### Adicionar mais dados e consulta-los no nó primário
```bash
db.alunos.insertOne({ra: 5, nome: "Teste1"})
```
```bash
db.alunos.find()
```

#### Verificar em um nó secundário se o novo dado foi incluído
```bash
db.alunos.find()
```

### Teste 2: Queda de um nó primário

#### Parar nó primário `mongo1`
Clicar no ícone "Stop" do container 'mongo2'

#### #### Verificar se o nó primário `mongo1` caiu
```bash
docker exec -it mongo1 mongosh
```

#### Entrar na instância do 'mongo3'
```bash
docker exec -it mongo3 mongosh
```
#### Verificação de qual é o novo nó primário 
```bash
rs.status()
```
#### Consultar dados no antigo nó primário para verificar se a conexão está realmente caiu
```bash
db.alunos.find()
```
#### Entrar no novo nó primário, inserir novos dados e consulta-los
```javascript
rs.isMaster().primary
```
```javascript 
  db.alunos.insertOne({ra: 6, nome: "Teste2"})
```
```javascript 
  db.alunos.find()
```
#### Acessar outro nó secundário e consultar dados
```javascript 
  db.alunos.find()
```
