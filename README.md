# Resolução dos exercícios de Docker

## Fáceis
- Exercício 1:

Primeiro criei uma pasta meusite em C:\Users\Arthur.

Dentro dessa pasta criei um index.html contendo o código html para o site.

Depois utilizei o comando abaixo para rodar um container nginx na porta 80, mandando o index.html para o /usr/share/nginx/html do container:

docker run -dti -p 80:80 -v C:\Users\Arthur\meusite:/usr/share/nginx/html nginx

Por último, só acessar e testar o site pelo http://localhost

- Exercício 2:

docker run -dti --name ubuntu1 ubuntu, para rodar um container Ubuntu.

Coloquei -dti para detach + criar um pseudo terminal (tty) + interactive.

Depois só acessar o container pelo comando:  docker exec -it ubuntu1 /bin/bash (-it para poder interagir com o terminal) (pode deixar só bash, sem o /bin).

Após ter o acesso, pode realizar comandos, criar pastas, arquivos e utilizar o container ubuntu para suas atividades.

- Exercício 3:

docker ps (lista os containers em execução)

docker ps -a (containers parados e em execução)

docker stop "tag ou nome" (para um container)

docker rm "tag ou nome" (remove um container)

docker container prune (remove todos os containers parados)

- Exercício 4:

Primeiramente, criei uma pasta onde coloquei o app.py, o Dockerfile e o requirements.txt (para instalar o flask no python).

APP.PY:
```
from flask import Flask

app = Flask(__name__)

@app.route('/')

def home():

    return "Olá, este é um app Flask rodando em um container Docker!"

if __name__ == '__main__':

    app.run(host='0.0.0.0', port=5000)
```

requirements.txt:
flask


Dockerfile:

```
FROM python:3.11

#Definindo um diretório de trabalho

WORKDIR /app

#Copiando os arquivos do projeto para o container

COPY requirements.txt .

COPY app.py .

#Instalando as dependências

RUN pip install --no-cache-dir -r requirements.txt

#Expondo a porta que o Flask vai usar

EXPOSE 5000

#Definindo o comando para rodar o app

CMD ["python", "app.py"] 
```

Construir a imagem flask-app com esse Dockerfile:

docker build -t flask-app .

Rodar o container com essa imagem na porta 5000

docker run -d -p 5000:5000 flask-app

Por fim, acessar o http://localhost:5000 para testar.

## Médias:

- Exercício 5:

docker volume create mysql_data (para criar um volume para armazenar dados)

docker run -d \
  --name mysql_container \
  -e MYSQL_ROOT_PASSWORD=senha123 \
  -e MYSQL_DATABASE=meubanco \
  -e MYSQL_USER=meuusuario \
  -e MYSQL_PASSWORD=senhadousuario \
  -p 3306:3306 \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0

(Executar o container mysql com o volume)
-d roda em segundo plano (detach)
-e para setar variáveis de ambiente do mysql
-p para mapear a porta
-v usar o volume mysql_data para a persistência dos dados

docker exec -it mysql_container mysql -u root -p

-u root: Especifica o nome de usuário para se conectar ao banco de dados MySQL.

-p: Indica que será solicitada a senha do usuário especificado.

Acessar o mysql dentro do container e a senha será senha123 quando solicitar.

- Exercício 6:

Criei um arquivo main.go com um servidor HTTP simples:

```
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {

	fmt.Fprintln(w, "Olá, este é um servidor Go otimizado com Multi-Stage Build no Docker!")
}

func main() {

	http.HandleFunc("/", handler)
	fmt.Println("Servidor rodando na porta 8080...")
	http.ListenAndServe(":8080", nil)
}
```

Criando o Dockerfile com Multi-Stage Build:

```
#Etapa 1: Construção da aplicação
FROM golang:1.21 AS builder

#Definir diretório de trabalho

WORKDIR /app

#Copiar arquivos do projeto

COPY . .

#Baixar dependências e compilar a aplicação

RUN go mod init app && go mod tidy

RUN go build -o servidor

#Etapa 2: Criando imagem final otimizada

FROM alpine:latest

#Definir diretório de trabalho

WORKDIR /app

#Copiar apenas o binário compilado da etapa anterior

COPY --from=builder /app/servidor .

#Expor a porta da aplicação

EXPOSE 8080

#Definir comando de execução

CMD ["./servidor"]
```

Construir a imagem Docker

docker build -t go-app .

Rodar o container

docker run -d -p 8080:8080 go-app

http://localhost:8080 para acessar e testar

- Exercício 7:

Criar uma rede Docker personalizada

docker network create minha_rede

Criar e rodar um container MongoDB

docker run -d \
  --name meu_mongo \
  --network minha_rede \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=senha123 \
  mongo:latest

Criar um aplicativo Node.js que se conecta ao MongoDB:

mkdir node-mongo && cd node-mongo

Crie um arquivo server.js:

```

const express = require('express');

const mongoose = require('mongoose');

const app = express();

const PORT = 3000;

//Conectar ao MongoDB (usando o nome do container MongoDB como hostname)
mongoose.connect('mongodb://root:senha123@meu_mongo:27017/admin', {
    useNewUrlParser: true,
    useUnifiedTopology: true
})
.then(() => console.log('Conectado ao MongoDB'))
.catch(err => console.error('Erro ao conectar:', err));

app.get('/', (req, res) => {
    res.send('Aplicação Node.js conectada ao MongoDB!');
});

app.listen(PORT, () => {
    console.log(`Servidor rodando na porta ${PORT}`);
});

```

Criar um Dockerfile para o Node.js:

```

#Usando a imagem oficial do Node.js

FROM node:18

#Definir diretório de trabalho

WORKDIR /app

#Copiar arquivos do projeto

COPY package*.json ./

RUN npm install

#Copiar código-fonte

COPY . .

#Expor a porta

EXPOSE 3000

#Comando para rodar o app

CMD ["node", "server.js"]
```

Criar o arquivo package.json:

npm init -y

Adicionar a dependência do MongoDB:

npm install express mongoose

Criar e rodar o container Node.js

docker build -t meu_node .

docker run -d \
  --name app_node \
  --network minha_rede \
  -p 3000:3000 \
  meu_node

Testar a conexão

http://localhost:3000

Vai exibir a mensagem "Aplicação Node.js conectada ao MongoDB!" se tudo estiver certo.

- Exercício 8:

Crie um diretório para o projeto e entre nele:

mkdir django_postgres && cd django_postgres

Criar os seguintes arquivos:

django_postgres/

docker-compose.yml

Dockerfile

requirements.txt

app/

      manage.py

      settings.py


Criar o Dockerfile para o Django:

```

#Usar a imagem oficial do Python

FROM python:3.11

#Definir diretório de trabalho dentro do container

WORKDIR /app

#Copiar arquivos do projeto para dentro do container

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

#Copiar código-fonte

COPY . .

#Expor a porta usada pelo Django

EXPOSE 8000

#Comando para rodar a aplicação

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

Criar o requirements.txt (lista as dependências do Django):

Django

psycopg2-binary

Criar o docker-compose.yml:

```
version: "3.9"

services:

  web:

    build: .

    container_name: django_app

    depends_on:

      - db

    ports:

      - "8000:8000"

    environment:

      - DEBUG=1

      - DJANGO_DB_HOST=db
      
      - DJANGO_DB_NAME=postgres

      - DJANGO_DB_USER=postgres

      - DJANGO_DB_PASSWORD=postgres

    volumes:

      - .:/app

  db:

    image: postgres:15

    container_name: postgres_db

    restart: always

    environment:

      - POSTGRES_USER=postgres

      - POSTGRES_PASSWORD=postgres

      - POSTGRES_DB=postgres

    volumes:

      - postgres_data:/var/lib/postgresql/data

volumes:

  postgres_data:
```

Configurar o Django para usar PostgreSQL

No arquivo settings.py do Django, altere a 
configuração do banco:

```
import os

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DJANGO_DB_NAME', 'postgres'),
        'USER': os.getenv('DJANGO_DB_USER', 'postgres'),
        'PASSWORD': os.getenv('DJANGO_DB_PASSWORD', 'postgres'),
        'HOST': os.getenv('DJANGO_DB_HOST', 'db'),
        'PORT': '5432',
    }
}
```

Construir e rodar os containers:

docker-compose up -d --build

Criar as migrações do banco de dados

docker-compose exec web python manage.py migrate

Acesse a aplicação Django no navegador:

http://localhost:8000

## Difícil:

- Exercício 9:

Crie um diretório e entre nele

mkdir nginx-site && cd nginx-site

Estrutura:

nginx-site/

    docker-compose.yml

    Dockerfile

    site/

        index.html

        styles.css

Criar o index.html:

```<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meu Site com Nginx</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Bem-vindo ao meu site estático!</h1>
    <p>Este site está rodando dentro de um container Nginx.</p>
</body>
</html>
```

Criar o styles.css:

```body {
    font-family: Arial, sans-serif;
    text-align: center;
    background-color: #f4f4f4;
    padding: 50px;
}

h1 {
    color: #333;
}
```
Criar o Dockerfile:

```
#Usar a imagem oficial do Nginx

FROM nginx:latest

#Remover a configuração padrão do Nginx

RUN rm -rf /usr/share/nginx/html/*

#Copiar os arquivos do site para o diretório de publicação do Nginx

COPY site /usr/share/nginx/html

#Expor a porta padrão do Nginx

EXPOSE 80
```

Se quiser rodar a aplicação com Docker Compose, crie um docker-compose.yml:

```
version: "3.9"

services:
  web:
    build: .
    container_name: nginx_site
    ports:
      - "8080:80"
```

Construir e rodar o container:

docker build -t meu-site-nginx .

docker run -d -p 8080:80 --name site_container meu-site-nginx

Usando Docker Compose:

docker-compose up -d --build

Acessar o site:
http://localhost:8080


## Como adicionar um user non-root ao container

Colocar isso no Dockerfile que será utilizado para criar a imagem do nginx:
```

#Usando a imagem base do Nginx

FROM nginx:1.24

#Criando um usuário e grupo não-root

RUN addgroup --system nginxgroup && adduser --system --ingroup nginxgroup nginxuser

#Mudando o dono dos arquivos necessários para o novo usuário

RUN chown -R nginxuser:nginxgroup /var/cache/nginx /var/run /var/log/nginx

#Alterando o usuário do container para o novo usuário

USER nginxuser

#Expondo a porta do Nginx

EXPOSE 80

#Iniciando o Nginx

CMD ["nginx", "-g", "daemon off;"]
```
