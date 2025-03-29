# 🍃🐳 Criação de um cluster MongoDB com Docker

Este documento descreve os passos para configurar um cluster MongoDB utilizando Docker, implementando um Replica Set com cinco nós. O teste de replicação de dados será feito via MongoDB Compass.

## 📌 Pré-requisitos
- Docker instalado;
- MongoDB Compass instalado.

## ⌨️ Principais comandos

### 1️⃣ Instalação de um cluster no MongoDB utilizando o Docker e criação de nós

#### Criação de uma rede Docker para comunicação entre os nós
```bash
docker network create ntwkClusterMongo
```

#### Verificação das redes existentes
```bash
docker network ls
```

#### Criação das instâncias do MongoDB
##### `mongo1`
```bash
- docker run -d --rm -p 27018:27017 --name mongo1 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo1
```
##### `mongo2`
```bash
- docker run -d --rm -p 27019:27017 --name mongo2 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo2
```
##### `mongo3`
```bash
- docker run -d --rm -p 27020:27017 --name mongo3 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo3
```
##### `mongo4`
```bash
- docker run -d --rm -p 27021:27017 --name mongo4 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo4
```
##### `mongo5`
```bash
- docker run -d --rm -p 27022:27017 --name mongo5 --network ntwkClusterMongo mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo5
```

#### Verificação se os nós estão rodando
```bash
docker ps
```

#### Acesso ao nó `mongo1`
```bash
docker exec -it mongo1 mongosh
```

#### Verificação dos status do Replica Set do MongoDB
```javascript
db.runCommand({hello:1})
```

#### Inicialização e configuração do Replica Set
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

#### Verificação dos status do Replica Set
```javascript
rs.status()
```

### 2️⃣ Funcionamento do cluster
Copiar os seguintes endereços e colar no MongoDB Compass para testar a conexão:
- **mongo1**: `mongodb://127.0.0.1:27018/?directConnection=true`
- **mongo2**: `mongodb://127.0.0.1:27019/?directConnection=true`
- **mongo3**: `mongodb://127.0.0.1:27020/?directConnection=true`
- **mongo4**: `mongodb://127.0.0.1:27021/?directConnection=true`
- **mongo5**: `mongodb://127.0.0.1:27022/?directConnection=true`

#### Verificação de qual nó é primário:
```javascript
rs.isMaster().primary
```

#### Acesso à collection pessoas e inserção de dados
```javascript
use pessoas
db.pessoas.insertMany([{
 "id": 1,
  "first_name": "Hewett",
  "last_name": "Claw",
  "email": "hclaw0@google.cn",
  "gender": "Male",
  "ip_address": "10.28.26.218"
},
{
  "id": 2,
  "first_name": "Vachel",
  "last_name": "Beszant",
  "email": "vbeszant1@slate.com",
  "gender": "Male",
  "ip_address": "194.13.160.166"
}, {
  "id": 3,
  "first_name": "Chan",
  "last_name": "Skerrett",
  "email": "cskerrett2@accuweather.com",
  "gender": "Male",
  "ip_address": "99.131.157.181"
}])
```

#### Consulta dos dados inseridos na collection pessoas
```javascript
db.pessoas.find()
```

### 3️⃣ Simulação de queda de nó secundário

#### Comando de parada do nó secundário `mongo2`
```bash
docker stop mongo2
```

#### Comando para acessar o nó secundário `mongo2`
```bash
docker exec -it mongo2 mongosh
```

#### Verificação dos status do cluster após a queda de nó secundário
```javascript
rs.status()
```

#### Seleção de dados na collection pessoas
```bash
db.pessoas.find()
```

#### Acesso à collection pessoas e inserção de dados pelo nó primário
```bash
use pessoas
db.pessoas.insertMany([{
  "id": 4,
  "first_name": "Tildy",
  "last_name": "Lilford",
  "email": "tlilford3@reverbnation.com",
  "gender": "Female",
  "ip_address": "8.99.197.114"
}, {
  "id": 5,
  "first_name": "Lon",
  "last_name": "Chinnock",
  "email": "lchinnock4@t.co",
  "gender": "Male",
  "ip_address": "162.90.141.177"
}, {
  "id": 6,
  "first_name": "Dwayne",
  "last_name": "Petersen",
  "email": "dpetersen5@home.pl",
  "gender": "Male",
  "ip_address": "45.86.111.93"}])
```

### 4️⃣ Simulação de queda do nó primário

#### Parada do nó primário `mongo1` 
```bash
docker stop mongo1
```

#### Acesso à instância do `mongo3` e verificação dos status do cluster
```bash
docker exec -it mongo3 mongosh
rs.status()
```

#### Verificação de qual é o novo nó primário 
```javascript
rs.isMaster().primary
```

#### Acesso à collection pessoas e inserção de dados pelo nó primário
```bash
use pessoas
db.pessoas.insertMany([{
  "id": 9,
  "first_name": "Mendie",
  "last_name": "Oattes",
  "email": "moattes8@nsw.gov.au",
  "gender": "Male",
  "ip_address": "154.10.228.57"
}, {
  "id": 10,
  "first_name": "Joelie",
  "last_name": "Casina",
  "email": "jcasina9@51.la",
  "gender": "Female",
  "ip_address": "71.122.205.167"
}, {
  "id": 11,
  "first_name": "Pierette",
  "last_name": "Goard",
  "email": "pgoarda@icio.us",
  "gender": "Female",
  "ip_address": "150.59.107.98"
}])
```

#### Seleção de dados na collection pessoas
```bash
db.pessoas.find()
```

###  5️⃣ Outras funcionalidades do Cluster

##### Priorização de eleição do nó primário

###### Criando as instâncias do MongoDB
`mongo10`
```javascript
docker run -d --rm -p 27022:27017 --name mongo10 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo10
```
`mongo20`
```javascript
docker run -d --rm -p 27023:27017 --name mongo20 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo20
```
`mongo30`
```javascript
docker run -d --rm -p 27024:27017 --name mongo30 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo30
```

##### Acesso à instância do `mongo10`
```javascript
docker exec -it mongo10 mongosh
```

##### Configuração do Replica Set
```javascript
rs.initiate({
  _id: "myReplicaSet2",
  members: [
    { _id: 1, host: "mongo10", priority: 1 },
    { _id: 2, host: "mongo20", priority: 2 },
    { _id: 3, host: "mongo30", priority: 10 }
  ]
})
```

##### Verificação dos status do cluster
```javascript
rs.status()
```

#### Delay na Replicação

##### Criando instâncias do MongoDB
`mongo10`
```bash
docker run -d --rm -p 27022:27017 --name mongo10 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo10
```
`mongo20`
```bash
docker run -d --rm -p 27023:27017 --name mongo20 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo20
```
`mongo30`
```bash
docker run -d --rm -p 27024:27017 --name mongo30 --network testeCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet2 --bind_ip localhost,mongo30
```

##### Acesso à instância do `mongo30` e configuração do cluster
```bash
docker exec -it mongo30 mongosh
rs.initiate({
 _id: "myReplicaSet2",
members: [
 { _id: 1, host: "mongo10", priority: 1 },
 { _id: 2, host: "mongo20", priority: 0, secondaryDelaySecs: 300 }, 
 { _id: 3, host: "mongo30", priority: 2 } ]})

```

##### Verificação da replicação com delay na instância do `mongo30`
```javascript
use admin
db.runCommand({replSetGetStatus: 1})
```

##### Configurando as instâncias do MongoDB no MongoDB Compass
Copiar os seguintes endereços e colar no MongoDB Compass para testar a conexão:
- **mongo10**: `mongodb://127.0.0.1:27022/?directConnection=true`
- **mongo20**: `mongodb://127.0.0.1:27023/?directConnection=true`
- **mongo30**: `mongodb://127.0.0.1:27024/?directConnection=true`


##### Verificação de qual nó é primário
```javascript
rs.isMaster().primary
```

##### Acesso à collection pessoas e inserção de dados na collection pessoas pelo nó primário
```javascript
use pessoas
db.pessoas.insertMany([
   "id": 7,
   "first_name": "Darda",
   "last_name": "McGahern",
   "email": "dmcgahern6@usnews.com",
   "gender": "Female",
   "ip_address": "211.150.191.224"
},
{  "id": 8,
   "first_name": "Templeton",
   "last_name": "Gerriet",
   "email": "tgerriet7@hubpages.com",
   "gender": "Male",
   "ip_address": "86.238.94.23"
}])
```
##### Seleção de dados na collection pessoas
```javascript
use pessoas
db.pessoas.find()
```
