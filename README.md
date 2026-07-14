# Traefik Stack

Este repositório monta uma stack simples com Traefik como entrypoint reverso, com roteamento baseado em Docker labels e configuração dinâmica para TLS local.

## Visão geral

A stack inclui:

- Traefik v3.7 como proxy reverso e dashboard
- Provider Docker para descoberta automática de serviços
- Provider de arquivos para configuração dinâmica de TLS e middlewares
- Exemplo de aplicação `whoami` para testes de roteamento

## Estrutura do projeto

```text
.
├── docker-compose.yml
├── certs/
├── config/
│   └── config.yaml
├── dynamic/
│   └── tls.yaml
└── letsencrypt/
```

## Pré-requisitos

- Docker
- Docker Compose
- Certificado TLS local em `certs/` com os nomes:
  - `certs/local.crt`
  - `certs/local.key`

Se ainda não existir, gere um certificado self-signed para testes locais:

```bash
docker run --rm -v "$PWD/certs:/certs" alpine/openssl \
  req -x509 -nodes -newkey rsa:2048 \
  -keyout /certs/local.key \
  -out /certs/local.crt \
  -days 365 -subj "/CN=*.docker.localhost"
```

## Subir a stack

```bash
docker compose up -d
```

## Verificar status

```bash
docker compose ps
docker compose logs -f traefik
```

## Acessar os serviços

Os serviços abaixo estão configurados para responder em hosts locais:

- Dashboard do Traefik: https://dashboard.docker.localhost
- Exemplo `whoami`: https://whoami.docker.localhost

> Se o navegador não resolver esses nomes, adicione ao arquivo `/etc/hosts`:
>
> ```text
> 127.0.0.1 dashboard.docker.localhost whoami.docker.localhost
> ```

## Como funciona

- A porta `80` recebe tráfego e redireciona para `443`.
- A porta `443` serve os serviços HTTPS.
- O Traefik descobre containers automaticamente via labels do Docker.
- A configuração TLS está em `dynamic/tls.yaml`.
- Middlewares e configurações extras podem ser adicionados em `config/config.yaml`.

## Configurações importantes

### Dashboard

O dashboard está habilitado em `docker-compose.yml` com:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.dashboard.rule=Host(`dashboard.docker.localhost`)"
  - "traefik.http.routers.dashboard.entrypoints=websecure"
  - "traefik.http.routers.dashboard.service=api@internal"
  - "traefik.http.routers.dashboard.tls=true"
```

### Aplicação exemplo

A aplicação `whoami` está configurada com o roteamento:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.whoami.rule=Host(`whoami.docker.localhost`)"
  - "traefik.http.routers.whoami.entrypoints=websecure"
  - "traefik.http.routers.whoami.tls=true"
```

## Personalização

Você pode:

- adicionar novos serviços com labels do Traefik;
- habilitar autenticação básica no dashboard;
- ajustar middlewares e headers de segurança em `config/config.yaml`;
- trocar os nomes de host e certificados conforme necessário.

## Parar a stack

```bash
docker compose down
```
