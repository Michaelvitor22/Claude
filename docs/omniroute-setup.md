# Configurando o OmniRoute localmente

Este repositório traz um ponto de partida (`docker-compose.yml` e `.env.example`
na raiz) para rodar o [OmniRoute](https://github.com/diegosouzapw/OmniRoute) —
um gateway de IA self-hosted que expõe um único endpoint OpenAI-compatível na
frente de vários provedores. O código-fonte do OmniRoute **não está vendorizado
aqui** — você clona o repositório deles à parte e usa os arquivos deste repo
para configurá-lo.

> **Antes de rodar:** o OmniRoute é um projeto de terceiros, não mantido pela
> Anthropic. Ele expõe um dashboard de administração e passa a ter acesso às
> chaves de API dos provedores que você configurar nele. Revise o código-fonte
> e as permissões que ele pede antes de colocar chaves reais.

## Passo a passo

1. **Clone o OmniRoute** ao lado deste repositório:

   ```bash
   git clone https://github.com/diegosouzapw/OmniRoute.git
   ```

   A estrutura de pastas deve ficar assim:

   ```
   seu-projeto/
   ├── docker-compose.yml   (deste repo)
   ├── .env.example         (deste repo)
   └── OmniRoute/           (clone acima)
   ```

2. **Configure as variáveis de ambiente:**

   ```bash
   cp .env.example .env
   ```

   Preencha pelo menos:
   - `JWT_SECRET` — gere com `openssl rand -base64 48`
   - `API_KEY_SECRET` — gere com `openssl rand -hex 32`
   - `INITIAL_PASSWORD` — senha inicial do dashboard (troque no primeiro login)

   O `.env.example` deste repo é um **subconjunto curado** das variáveis mais
   usadas. A lista completa (20+ seções, autenticação, cache, compressão,
   provedores, etc.) está no `.env.example` oficial do OmniRoute:
   https://github.com/diegosouzapw/OmniRoute/blob/main/.env.example

3. **Suba os containers:**

   ```bash
   docker compose up -d
   ```

   Isso builda a imagem a partir do clone em `./OmniRoute` (target
   `runner-base`, sem CLIs extras nem navegador embutido) e sobe um Redis
   auxiliar para rate limiting.

4. **Acesse o dashboard** em `http://localhost:20128` e faça login com o
   `INITIAL_PASSWORD` definido no `.env`. A partir daí você cadastra as chaves
   de API dos provedores que quiser usar (OpenAI, Anthropic, etc.) — nunca
   direto no `.env` versionado.

5. **Aponte suas ferramentas** (Claude Code, Cursor, etc.) para
   `http://localhost:20128/v1` como endpoint OpenAI-compatível.

## Indo além do básico

O `docker-compose.yml` deste repo cobre apenas o perfil `base` (gateway +
Redis). O compose oficial do OmniRoute tem perfis adicionais — `web` (Chromium
para provedores que dependem de cookie de navegador), `cli`, `host`, `memory`
(Qdrant) e `bifrost` (roteador Go). Para usá-los, rode o `docker-compose.yml`
de dentro do clone `OmniRoute/` diretamente, seguindo as instruções no README
deles.

## Segurança

- Nunca commite um `.env` preenchido — o `.env.example` deste repo é só um
  template.
- O `INITIAL_PASSWORD` padrão do upstream é `CHANGEME`; troque antes de expor
  a porta do dashboard além de `localhost`.
- Revise `docs/architecture` no repositório do OmniRoute se for habilitar
  perfis que expõem serviços adicionais (Redis, Qdrant, Bifrost) — por padrão
  o Redis deste compose só escuta em loopback.
