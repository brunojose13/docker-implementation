# Ambiente Docker para Laravel

Repositório com arquivos prontos para rodar um projeto [Laravel](https://laravel.com) em containers Docker: API PHP (Artisan), MySQL 8.4 e Composer.

## O que está incluído

| Arquivo | Função |
|---------|--------|
| `Dockerfile` | Imagem PHP 8.5 CLI com extensões comuns do Laravel e Composer |
| `docker-compose.yml` | Serviços da aplicação e do banco, rede, volume e healthcheck |
| `.env.example` | Variáveis de banco usadas pelo Compose e pelo Laravel |

## Pré-requisitos

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/) (v2+)

## Uso em um projeto Laravel existente

Na **raiz do seu projeto Laravel** (onde ficam `artisan`, `composer.json` e a pasta `app/`):

### 1. Obter os arquivos deste repositório

**Opção A — clone temporário (recomendado)**

```bash
git clone git@github.com:brunojose13/docker-implementation.git docker
cp docker/Dockerfile docker/docker-compose.yml docker/.env.example ./
rm -rf docker
```

**Opção B — copiar manualmente**

Copie `Dockerfile`, `docker-compose.yml` e `.env.example` para a raiz do Laravel.

### 2. Personalizar nomes do projeto

No `docker-compose.yml`, substitua `project-name` pelo nome do seu projeto (serviços, containers, rede e volume devem ficar consistentes).

Exemplo: `meu-app-api`, `meu-app-database`, rede `meu-app`, etc.

### 3. Configurar variáveis de ambiente

```bash
cp .env.example .env
```

Edite o `.env` do Laravel com os valores de banco. O host do MySQL **dentro da rede Docker** deve ser o **nome do serviço** do banco no `docker-compose.yml` (por padrão, após renomear: `seu-projeto-database`).

Exemplo (ajuste nomes e senhas):

```env
DB_CONNECTION=mysql
DB_HOST=meu-app-database
DB_PORT=3306
DB_DATABASE=meu_app_db
DB_USERNAME=user_name
DB_PASSWORD=user_password
```

As variáveis `MYSQL_*` no serviço do banco são lidas do mesmo `.env` via `docker-compose.yml`.

### 4. Subir os containers

```bash
docker compose up -d --build
```

A API sobe na porta **8000** (`http://localhost:8000`). O MySQL fica exposto na porta **3307** do host (mapeamento `3307:3306`).

### 5. Dependências e setup do Laravel

Substitua `project-name-api` pelo nome do container da API definido no compose:

```bash
docker exec project-name-api composer install
docker exec project-name-api php artisan key:generate
docker exec project-name-api php artisan migrate
```

O `Dockerfile` já define o comando padrão `php artisan serve --host=0.0.0.0 --port=8000`. Em condições normais não é necessário iniciar o Artisan manualmente.

## Comandos úteis

```bash
# Ver logs
docker compose logs -f

# Parar
docker compose stop

# Parar e remover containers (volume do banco é mantido)
docker compose down

# Reiniciar tudo
docker compose down && docker compose up -d --build
```

Artisan e Composer sempre via container da API:

```bash
docker exec project-name-api php artisan <comando>
docker exec project-name-api composer <comando>
```

## Estrutura dos serviços

- **API** (`project-name-api`): monta o diretório atual em `/var/www/html`, depende do banco estar saudável (healthcheck).
- **Banco** (`project-name-database`): MySQL 8.4 com volume persistente `project-name-database`.

## Solução de problemas

| Problema | O que verificar |
|----------|-----------------|
| Laravel não conecta ao MySQL | `DB_HOST` = nome do **serviço** no compose, não `localhost` |
| Porta 8000 em uso | Altere o mapeamento em `ports` no serviço da API |
| Erro ao subir o banco | `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` e `DB_ROOT_PASSWORD` no `.env` |
| Mudanças no Dockerfile sem efeito | `docker compose up -d --build` |

## Licença

Uso livre conforme necessidade do seu projeto.
