# Serviços disponíveis no Laravel Sail

## Bancos de Dados

### MySQL (padrão)
- Parâmetro: `mysql`
- Porta padrão: `3306`
- `.env`: `DB_CONNECTION=mysql`, `DB_HOST=mysql`
- Melhor para: maioria dos projetos, ecossistema amplo

### PostgreSQL
- Parâmetro: `pgsql`
- Porta padrão: `5432`
- `.env`: `DB_CONNECTION=pgsql`, `DB_HOST=pgsql`
- Melhor para: recursos avançados (JSON nativo, full-text, arrays)

### MariaDB
- Parâmetro: `mariadb`
- Porta padrão: `3306`
- `.env`: igual ao MySQL

---

## Cache e Filas

### Redis
- Parâmetro: `redis`
- Porta padrão: `6379`
- `.env`:
  ```dotenv
  CACHE_DRIVER=redis
  SESSION_DRIVER=redis
  QUEUE_CONNECTION=redis
  REDIS_HOST=redis
  ```
- Melhor para: cache, sessões, filas (Horizon), broadcasting (Reverb)

---

## Busca

### Meilisearch
- Parâmetro: `meilisearch`
- Porta padrão: `7700`
- `.env`: `SCOUT_DRIVER=meilisearch`, `MEILISEARCH_HOST=http://meilisearch:7700`
- Instalar: `./vendor/bin/sail composer require laravel/scout meilisearch/meilisearch-php`

### Typesense
- Parâmetro: `typesense`
- Porta padrão: `8108`
- `.env`: `SCOUT_DRIVER=typesense`, `TYPESENSE_HOST=typesense`

---

## E-mail

### Mailpit
- Parâmetro: `mailpit`
- Porta SMTP: `1025`, Interface web: `8025`
- `.env`:
  ```dotenv
  MAIL_MAILER=smtp
  MAIL_HOST=mailpit
  MAIL_PORT=1025
  ```
- Acesso: `http://localhost:8025`

---

## Armazenamento

### MinIO (S3-compatible)
- Parâmetro: `minio`
- Console: `http://localhost:8900`
- `.env`:
  ```dotenv
  FILESYSTEM_DISK=s3
  AWS_ACCESS_KEY_ID=sail
  AWS_SECRET_ACCESS_KEY=password
  AWS_DEFAULT_REGION=us-east-1
  AWS_BUCKET=local
  AWS_ENDPOINT=http://minio:9000
  AWS_USE_PATH_STYLE_ENDPOINT=true
  ```

---

## Testes de Browser

### Selenium
- Parâmetro: `selenium`
- Usar com Laravel Dusk
- Instalar: `./vendor/bin/sail composer require --dev laravel/dusk`

---

## Combinações comuns

| Tipo de projeto | Serviços sugeridos |
|---|---|
| API simples | `mysql,redis` |
| App web completo | `mysql,redis,mailpit` |
| App com busca | `mysql,redis,meilisearch,mailpit` |
| App com arquivos | `mysql,redis,minio,mailpit` |
| App completo | `mysql,redis,meilisearch,minio,mailpit` |