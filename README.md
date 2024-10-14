# Projeto Docker

## Índice
1. [Pré-requisitos](#pré-requisitos)
2. [Configuração do Projeto](#configuração-do-projeto)
3. [Estrutura do Repositório](#estrutura-do-repositório)
4. [Detalhes de Configuração](#detalhes-de-configuração)
5. [Configuração de Balanceamento com NGINX](#configuração-de-balanceamento-com-nginx)
6. [Guia de Atualização](#guia-de-atualização)

---

## **Pré-requisitos
**
Certifique-se de ter os seguintes itens instalados:
- **[Docker](https://docs.docker.com/engine/install/)**
- **[Docker Compose](https://docs.docker.com/compose/install/)**
- **[Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)**

---

## **Configuração do Projeto**

1. Clone o repositório:
   ``git clone https://github.com/leonamvasquez/Projeto-Docker``

2. Navegue para o diretório do projeto:
  ``cd Projeto-Docker``

3. Inicie os serviços com Docker Compose:
``docker-compose up``

4. Acesse a aplicação no navegador em http://localhost.

---

## **Estrutura do Repositório**

* **``docker-compose.yml``: Configura os serviços Docker.**

* **Dockerfile do Backend: Localizado na raiz do projeto (``Dockerfile``).**

* **Dockerfile do Frontend: Localizado em ``frontend/Dockerfile``.**

* **Configuração do NGINX: Arquivo ``nginx.conf`` na raiz.**

---

## **Detalhes de Configuração**

### Arquivo ``docker-compose.yml``

O ``docker-compose.yml`` inclui a configuração para os serviços: frontend, backend, banco de dados e NGINX.


**Serviço Frontend** <br>
Configurado para permitir atualizações independentes do backend.
````
frontend:
  build:
    context: ./frontend
    dockerfile: Dockerfile
  restart: always
  environment:
    - REACT_APP_BACKEND_URL=http://localhost
  depends_on:
    - guess
````

**Serviço Backend** <br>
Inclui um healthcheck para verificar a disponibilidade antes de iniciar o NGINX.
````
  guess:
    build: .
    restart: always
    environment:
      - FLASK_APP=run.py
      - FLASK_DB_TYPE=postgres
      - FLASK_DB_USER=myuser
      - FLASK_DB_PASSWORD=mypassword
      - FLASK_DB_NAME=mydatabase
      - FLASK_DB_HOST=postgres
      - FLASK_DB_PORT=5432
    depends_on:
      - postgres
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 15s      
````

**Serviço de Banco de Dados (PostgreSQL)** <br>
Utiliza um volume para persistência dos dados.
````
  postgres:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_PASSWORD: example
      POSTGRES_USER: myuser
      POSTGRES_DB: mydatabase
    volumes:
      - postgres-data:/var/lib/postgresql/data

````
**Serviço NGINX** <br>
Atua como balanceador de carga, direcionando o tráfego entre frontend e backend.
````
nginx:
  image: nginx:latest
  restart: always
  ports:
    - "80:80"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf
  depends_on:
    - frontend
    - guess
````

---

## **Configuração de Balanceamento com NGINX**

O arquivo ``nginx.conf`` define o balanceamento de carga:

* ``/`` direciona para o frontend.

* ``/create``, ``/breaker``, ``/guess`` direcionam para o backend.

**Exemplo de configuração no nginx.conf:**

````
events { }

http {
    server {
        listen 80;

        location / {
            proxy_pass http://frontend:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/(create|breaker|guess) {
            proxy_pass http://guess:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
````
## **Guia de Atualização**

### **Atualizar NGINX**

1. Modifique o arquivo ``nginx.conf`` ou o ``docker-compose.yml``.

2. Execute os comandos:
````
docker-compose down
docker-compose up
````

### **Atualizar o Frontend**

1. Edite o Dockerfile localizado em ``frontend`` ou o ``docker-compose.yml``.

2. Execute:
````
docker-compose down
docker-compose up
````

### **Atualizar o Backend**

1. Modifique o Dockerfile na raiz do projeto ou variáveis de ambiente no ``docker-compose.yml``.

2. Execute:
````
docker-compose down
docker-compose up
````

### **Atualizar o ``docker-compose.yml``**

1. Faça as alterações necessárias no ``docker-compose.yml``.

2. Execute:
````
docker-compose down
docker-compose up
````
