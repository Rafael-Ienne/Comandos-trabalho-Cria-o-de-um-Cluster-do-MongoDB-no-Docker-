# Configuração de um Cluster MongoDB com Docker

Este documento descreve os passos para configurar um cluster MongoDB utilizando Docker, implementando um Replica Set com no mínimo quatro nós. O teste será feito via MongoDB Compass.

## 📌 Pré-requisitos
- Docker instalado
- MongoDB Compass instalado

## Principais comandos

### Instalação de um cluster do MongoDB

#### Criação de uma rede Docker para comunicação entre os containers
```bash
docker network create ntwkClusterMongo
```

#### Verificação das redes existentes:
```bash
docker network ls
```

#### Criação das instâncias do MongoDB
```bash
docker run -d --rm -p 27018:27017 --name mongo1 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo1
docker run -d --rm -p 27019:27017 --name mongo2 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo2
docker run -d --rm -p 27020:27017 --name mongo3 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo3
docker run -d --rm -p 27021:27017 --name mongo4 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo4
docker run -d --rm -p 27022:27017 --name mongo5 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo5
```

#### Verificação se os containers estão rodando:
```bash
docker ps
```

#### Acesso à instância do MongoDB e configuração do Replica Set

##### Entrada no container `mongo1`
```bash
docker exec -it mongo1 mongosh
```

##### Verificação dos status do Replica Set MongoDB
```javascript
db.runCommand({hello:1})
```

##### Inicialização e configuração do Replica Set
```javascript
rs.initiate({
  _id: "myReplicaSet",
  members: [
    {_id: 0, host: "mongo1"},
    {_id: 1, host: "mongo2"},
    {_id: 2, host: "mongo3"},
    {_id: 3, host: "mongo4"},
    {_id: 4, host: "mongo5"}
  ]
})
```

##### Verificação do status do Replica Set:
```javascript
rs.status()
```

### 4️⃣ Teste de conexão no MongoDB Compass
Copie os seguintes endereços e cole no MongoDB Compass para testar a conexão:
- **Mongo1**: `mongodb://127.0.0.1:27018/?directConnection=true`
- **Mongo2**: `mongodb://127.0.0.1:27019/?directConnection=true`
- **Mongo3**: `mongodb://127.0.0.1:27020/?directConnection=true`
- **Mongo4**: `mongodb://127.0.0.1:27021/?directConnection=true`
- **Mongo5**: `mongodb://127.0.0.1:27022/?directConnection=true`

Verificação se o nó é primário:
```javascript
rs.isMaster().primary
```

### 5️⃣ Inserção e consulta de dados no MongoDB

#### Criação de uma collection e inserção de dados
```javascript
use pessoas
db.pessoas.insertMany([
  {"id": 1, "first_name": "João", "last_name": "Silva", "email": "joao@email.com"},
  {"id": 2, "first_name": "Maria", "last_name": "Santos", "email": "maria@email.com"}
])
```

#### Consulta dos dados inseridos
```javascript
db.pessoas.find()
```

## Simulação de queda de nó secundário

### Comando de parada de um nó secundário
```bash
docker stop mongo2
```

### Verificação de status do cluster após a queda
```javascript
rs.status()
```

### Parada do nó primário e verificação de nova eleição
```bash
docker stop mongo1
docker exec -it mongo3 mongosh
rs.status()
```

Verificação de qual é o novo nó primário:
```javascript
rs.isMaster().primary
```

### 🔄 Priorização de Eleição do Nó Primário
Criando um novo cluster com prioridades definidas:
```javascript
rs.initiate({
  _id: "myReplicaSet2",
  members: [
    { _id: 1, host: "mongo10", priority: 1 },
    { _id: 2, host: "mongo20", priority: 0, secondaryDelaySecs: 300 },
    { _id: 3, host: "mongo30", priority: 2 }
  ]
})
```

### ⏳ Configurando Delay na Replicação
Criando instâncias:
```bash
docker run -d --rm -p 27018:27017 --name mongo10 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo10
docker run -d --rm -p 27019:27017 --name mongo20 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo20
docker run -d --rm -p 27020:27017 --name mongo30 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo30
```

Verifique a replicação com delay:
```javascript
use admin
db.runCommand({replSetGetStatus: 1})
```

## 📢 Conclusão
Com este setup, você criou um cluster MongoDB usando Docker, testou failover, inseriu e consultou dados, e configurou replicação com delay e prioridade de eleição de nós.
