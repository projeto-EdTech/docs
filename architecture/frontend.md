# Arquitetura do Frontend

> Conteúdo movido do README do repositório `frontend` (antes um único arquivo de 42KB) para manter o README enxuto como porta de entrada e concentrar aqui a documentação extensa de arquitetura, estrutura de pastas e convenções.

## Stack Técnica

| Camada | Tecnologia |
| --- | --- |
| Framework | Next.js 16 (App Router) |
| Runtime | React 19 + TypeScript 5 |
| Estilização | Tailwind CSS 4 + tailwind-merge + CVA |
| Componentes base | shadcn/ui (Radix UI primitives) |
| Animações | Framer Motion 12 + Anime.js 4 |
| Formulários | React Hook Form 7 + Zod |
| Autenticação | NextAuth v4 (Google, Azure AD, Facebook, Discord) |
| Pagamentos | MercadoPago SDK + Stripe (com failover automático — [ADR-0002](../adr/0002-payment-gateway-failover.md)) |
| IA | Google Generative AI (`@google/genai`, `@google/generative-ai`) |
| Analytics | Google Analytics 4, PostHog, Microsoft Clarity |
| Gráficos | Recharts 2 |
| Calendário | FullCalendar 6 (daygrid, timegrid, interaction) |
| Virtualização | react-window 2 + react-virtualized-auto-sizer |
| Drag & Drop | react-dnd 16 |
| HTTP client | SWR 2 (dados client-side), `fetch` nativo (server-side) |
| Markdown | react-markdown + remark-gfm + remark-math + rehype-katex |
| Testes | Vitest 4 |
| Linting | ESLint 9 + eslint-config-next |

## Funcionalidades

### Autenticação e Perfil

- Login social via Google, Microsoft (Azure AD), Facebook, Discord
- Sync de usuário com backend Java via `/api/sync-user` (salva cookie `user_data` HttpOnly)
- Perfil completo: foto, estatísticas, badges, conquistas, configurações de conta
- Integração Discord: geração de token OTP `VEST-XXXXX` para vincular conta ao bot
- Badge "Guerreiro do Discord" desbloqueada ao vincular conta Discord

### Sistema de Tiers

Tiers: `FREE` | `Simula PRO` | `TEACHER` | `ADMIN`

Fluxo: Java BFF → `/api/sync-user` → cookie `user_data` (HttpOnly JWT) + localStorage → hook `useUserTier` decodifica via `jwtDecoder.ts` (ver [ADR-0003](../adr/0003-jwt-single-decode-point.md))

### Simulados

- Criação de simulados por universidade (`/simulation/[university]`)
- Simulado misto cross-universidade (`/api/simulations/create-mix`)
- Tela de questão com suporte a LaTeX (KaTeX)
- Resumo/resultado ao finalizar (`/simulation/[university]/summary`)
- Store em memória com TTL 10 min para passar questões entre rotas (`simulationStore.ts`)

### Ranking e Elo

- Ranking global de usuários (`/ranking`)
- Sistema de elo com modal de subida animado (Glassmorphism + Claymorphism)
- Tema claro/escuro sincronizado com `data-theme` no modal de elo

### Mini-games (Arena)

Rota `/Arena` com 4 jogos educativos:

| Jogo | Descrição |
| --- | --- |
| **Enigma** | Quiz de múltipla escolha por matéria com visual colorido por disciplina |
| **Lexoo** | Jogo de palavras educativo |
| **Nexo** | Jogo de conexões temáticas |
| **Flash Cards** | Revisão por cartões (deck via `/api/games/flash-cards`) |

### Simula PRO (conteúdo exclusivo para assinantes)

- Questões resolvidas com IA (Google Gemini): seletor hierárquico Matéria → Conteúdo com modal de seleção
- Chatbot IA (`/api/ai/chat`) com histórico (`/api/ai/historico`)
- Relatório de desempenho gerado por IA (`/api/relatorio-IA`)
- Planner de estudos com integração Google Calendar (`/api/planner/Google`)
- Estatísticas detalhadas por matéria/conteúdo (`/estatisticas/[subject]`)
- Geração de explicações por questão (`/api/generate-explanation`)

### Biblioteca e Blog

- Biblioteca de provas por universidade (`/library/[university]`)
- Blog com posts e playlists (`/blog`, `/blog/[slug]`, `/blog/playlist/[id]`)
- Playlists de questões: criar, adicionar questão, jogar

### Acessibilidade e Tema

- Fonte OpenDyslexic alternável via `AccessibilityContext`
- Tema claro/escuro persistido em localStorage + atributo `data-theme` no `<html>`
- Modo escuro apenas para usuários autenticados
- Todas as cores via CSS Custom Properties em `globals.css` — sem hex hardcoded

## Arquitetura

### Padrão BFF Proxy

Todos os `src/app/api/` Route Handlers são proxies finos para o Java BFF. Sem lógica de negócio nos handlers — lógica fica em `src/app/service/`. Detalhes e motivação em [ADR-0001](../adr/0001-bff-proxy-pattern.md).

```text
Browser
  └── Next.js Route Handler (src/app/api/)
        └── Service (src/app/service/)
              └── Java BFF (BACKEND_API_URL)
```

### Divisão Server / Client

Toda feature que combina dados BFF com interatividade usa o padrão obrigatório:

```text
page.tsx (thin — só Suspense)
  └── Suspense fallback={Skeleton}
        └── *DataServer.tsx  (Server Component — fetch BFF)
              └── *Client.tsx  ('use client' — estado + animações)
```

### Cache Strategy

Cada `page.tsx` ou hook de dados tem comentário obrigatório:

```ts
// CACHE STRATEGY: ISR — revalidate 60s — conteúdo estático
// CACHE STRATEGY: SWR — revalidateOnFocus — dados do usuário
// CACHE STRATEGY: no-store — dados sensíveis/financeiros
```

## Estrutura de Pastas

```text
front/
├── src/
│   ├── app/
│   │   ├── api/                    Route Handlers (proxies BFF)
│   │   │   ├── ai/                 chat, historico
│   │   │   ├── auth/               NextAuth handler
│   │   │   ├── badges/
│   │   │   ├── blog/               listagem e [slug]
│   │   │   ├── estatisticas/       [subject]
│   │   │   ├── games/              flash-cards
│   │   │   ├── gateway-health/     health check das gateways de pagamento
│   │   │   ├── generate-explanation/
│   │   │   ├── get-logo/
│   │   │   ├── Nota-corte/
│   │   │   ├── planner/            Google Calendar
│   │   │   ├── playlist/           CRUD + play
│   │   │   ├── process-subscription/  boleto, credit-card, pix
│   │   │   ├── questions/          [university]
│   │   │   ├── ranking/
│   │   │   ├── relatorio-IA/
│   │   │   ├── simulations/        create, create-mix, save-result, [id]
│   │   │   ├── subscribe/
│   │   │   ├── sync-user/
│   │   │   ├── universities/       listagem e [university]
│   │   │   ├── user/               profile, stats
│   │   │   ├── users/              generate-token (Discord OTP)
│   │   │   └── webhooks/           mercadopago, stripe
│   │   ├── service/                Lógica de negócio server-side
│   │   │   ├── payment/            Router de gateway + adapters
│   │   │   ├── badge.service.ts
│   │   │   ├── discordToken.service.ts
│   │   │   ├── game.service.ts
│   │   │   ├── jwtDecoder.ts       ÚNICO ponto de decode JWT
│   │   │   ├── playlist.service.ts
│   │   │   ├── pricing.service.ts
│   │   │   ├── ranking.service.ts
│   │   │   ├── simulation.service.ts
│   │   │   ├── statistics.service.ts
│   │   │   └── university.service.ts
│   │   ├── Arena/                  Mini-games hub
│   │   ├── blog/
│   │   ├── contato/
│   │   ├── create/
│   │   ├── estatisticas/
│   │   ├── library/
│   │   ├── paidPlan/
│   │   ├── privacy/
│   │   ├── profile/
│   │   ├── ranking/
│   │   ├── simulation/
│   │   ├── terms/
│   │   ├── VestIA/
│   │   ├── globals.css             CSS Custom Properties (tokens de cor, espaçamento)
│   │   └── layout.tsx              Root layout + provider stack
│   ├── components/
│   │   ├── Arena/                  Componentes dos mini-games
│   │   ├── Estatisticas/
│   │   ├── Filtros/
│   │   ├── games/                  Enigma, Lexoo, Nexo, Flash Cards
│   │   ├── Library/
│   │   ├── payment/                StripeCardForm, CreditCardForm
│   │   ├── pricing/                PricingClient (seleção de plano + pagamento)
│   │   ├── profile/                Perfil, badges, conquistas, Discord
│   │   ├── ranking/                RankingUpNotification (modal elo)
│   │   ├── Simula_PRO/             Questões IA, Planner, Stats, Chatbot
│   │   ├── Simulation/             Tela de simulado
│   │   ├── Skeletons/              Skeleton screens por rota
│   │   ├── blog/
│   │   ├── community/
│   │   ├── contato/
│   │   ├── mockups/
│   │   └── ui/                     shadcn/ui + componentes base customizados
│   ├── contexts/
│   │   ├── AccessibilityContext.tsx   Fonte OpenDyslexic
│   │   ├── LoadingContext.tsx
│   │   ├── ProfileIconContext.tsx
│   │   ├── ThemeContext.tsx           Tema claro/escuro + data-theme
│   │   └── UniversityStorage.tsx      Cache de universidades (fallback estático)
│   ├── hooks/
│   │   ├── use-mobile.ts
│   │   ├── use-toast.ts
│   │   └── useUserTier.ts            Lê tier do usuário via JWT/localStorage
│   ├── lib/
│   │   ├── badges/                   badgeUtils.ts + badges.json
│   │   ├── core/                     auth.ts · analytics.ts · utils.ts · discordLinked.ts
│   │   ├── data/                     dados estáticos (universidades, posts, playlists…)
│   │   ├── games/                    config.ts · games.ts
│   │   ├── planner/                  planner.ts
│   │   ├── ranking/                  rankUtils · rankUpUtils · ranking.ts
│   │   └── store/                    simulationStore.ts · userStatsCache.ts
│   ├── providers/
│   │   └── PostHogProvider.tsx
│   └── types/
│       └── next-auth.d.ts
├── tests/                            Specs Vitest
├── vitest.config.ts
├── next.config.ts
├── tailwind.config (inline via Tailwind v4)
└── package.json
```

## Rotas e API

### Rotas de Página

| Rota | Descrição |
| --- | --- |
| `/` | Landing page |
| `/Arena` | Hub de mini-games |
| `/Arena/[game]` | Jogo específico (Enigma, Lexoo, Nexo, Flash Cards) |
| `/blog` | Listagem de posts |
| `/blog/[slug]` | Post individual |
| `/blog/playlist/[id]` | Playlist de questões |
| `/contato` | Formulário de contato |
| `/create` | Criação de simulado |
| `/estatisticas/[subject]` | Estatísticas por matéria |
| `/library` | Biblioteca de provas (ISR — revalidate 1h) |
| `/library/[university]` | Provas por universidade |
| `/paidPlan` | Página de planos e pagamento |
| `/privacy` | Política de privacidade |
| `/profile` | Perfil do usuário |
| `/ranking` | Ranking global (ISR — revalidate 1min) |
| `/simulation/[university]` | Simulado |
| `/simulation/[university]/summary` | Resumo do simulado |
| `/terms` | Termos de uso |
| `/VestIA` | Assistente IA |

### Route Handlers (API)

| Endpoint | Método | Descrição |
| --- | --- | --- |
| `/api/ai/chat` | POST | Chat com IA (Google Gemini) |
| `/api/ai/historico` | GET | Histórico de conversas IA |
| `/api/auth/[...nextauth]` | GET/POST | Handler NextAuth |
| `/api/badges` | GET | Badges do usuário |
| `/api/blog` | GET | Posts do blog |
| `/api/blog/[slug]` | GET | Post por slug |
| `/api/estatisticas/[subject]` | GET | Estatísticas por matéria |
| `/api/games/flash-cards` | GET | Deck de flash cards |
| `/api/gateway-health` | GET | Health check MercadoPago + Stripe (ISR 30s) |
| `/api/generate-explanation` | POST | Gera explicação de questão via IA |
| `/api/get-logo` | GET | Logo de universidade |
| `/api/Nota-corte` | GET | Notas de corte |
| `/api/planner` | GET/POST | Planner de estudos |
| `/api/planner/Google` | POST | Sincroniza com Google Calendar |
| `/api/playlist` | GET/POST | Playlists do usuário |
| `/api/playlist/[id]` | GET/PUT/DELETE | Playlist específica |
| `/api/playlist/[id]/add-question` | POST | Adiciona questão à playlist |
| `/api/playlist/[id]/play` | GET | Modo de jogo da playlist |
| `/api/process-subscription/boleto` | POST | Processa boleto (MP → Stripe fallback) |
| `/api/process-subscription/credit-card` | POST | Processa cartão (hint-based: Stripe ou MP) |
| `/api/process-subscription/pix` | POST | Processa PIX (MP → Stripe fallback) |
| `/api/questions/[university]` | GET | Questões por universidade |
| `/api/ranking` | GET | Ranking global |
| `/api/relatorio-IA` | POST | Relatório de desempenho por IA |
| `/api/simulations/create` | POST | Cria simulado |
| `/api/simulations/create-mix` | POST | Cria simulado misto |
| `/api/simulations/save-result` | POST | Salva resultado do simulado |
| `/api/simulations/[id]` | GET | Simulado por ID |
| `/api/subscribe` | POST | Inscrição em plano |
| `/api/sync-user` | POST | Sincroniza usuário com BFF (seta cookie `user_data`) |
| `/api/universities` | GET | Lista universidades |
| `/api/universities/[university]` | GET | Universidade específica |
| `/api/user/profile` | GET/PUT | Perfil do usuário |
| `/api/user/stats` | GET | Estatísticas do usuário |
| `/api/users/generate-token` | POST | Gera token OTP Discord (`VEST-XXXXX`, TTL 5min) |
| `/api/webhooks/mercadopago` | POST | Webhook MercadoPago |
| `/api/webhooks/stripe` | POST | Webhook Stripe (verifica assinatura) |

## Camada de Serviço

`src/app/service/` contém funções server-side puras. Regras:

- Sem imports React ou hooks
- Retorno explicitamente tipado
- Erros descritivos em `!response.ok` — sem expor detalhes internos ao cliente
- Sempre usa `process.env.BACKEND_API_URL` — sem URLs hardcoded

| Arquivo | Responsabilidade |
| --- | --- |
| `jwtDecoder.ts` | **Único ponto** de decode JWT na aplicação |
| `badge.service.ts` | Fetch e parse de badges do BFF |
| `discordToken.service.ts` | Geração de token OTP Discord |
| `game.service.ts` | Dados de mini-games |
| `payment/` | Router de gateway + adapters MercadoPago e Stripe |
| `playlist.service.ts` | CRUD de playlists |
| `pricing.service.ts` | Planos e preços |
| `ranking.service.ts` | Ranking e elo |
| `simulation.service.ts` | Criação e gestão de simulados |
| `statistics.service.ts` | Estatísticas de desempenho |
| `university.service.ts` | Dados de universidades |

## Contextos e Hooks Globais

### Provider Stack (ordem em `layout.tsx`)

```text
NextAuthProvider
  ThemeProviderWrapper
    ProfileIconProvider
      AccessibilityProvider
        UniversityStorage
          PHProvider (PostHog)
```

### Contextos

| Contexto | Arquivo | Descrição |
| --- | --- | --- |
| `ThemeContext` | `contexts/ThemeContext.tsx` | Aplica `data-theme` (light/dark) no elemento html. Dark mode só para autenticados. |
| `ProfileIconContext` | `contexts/ProfileIconContext.tsx` | Estado do ícone/avatar do perfil |
| `AccessibilityContext` | `contexts/AccessibilityContext.tsx` | Alterna fonte OpenDyslexic (`--font-opendyslexic`) |
| `LoadingContext` | `contexts/LoadingContext.tsx` | Loading global entre navegações |
| `UniversityStorage` | `contexts/UniversityStorage.tsx` | Cache de universidades; fallback para `lib/data/universities.ts` se BFF indisponível |

### Hooks

| Hook | Arquivo | Descrição |
| --- | --- | --- |
| `useUserTier` | `hooks/useUserTier.ts` | Lê tier do usuário do localStorage (polling 2s); retorna `FREE \| Simula PRO \| TEACHER \| ADMIN` |
| `use-mobile` | `hooks/use-mobile.ts` | Detecta viewport mobile |
| `use-toast` | `hooks/use-toast.ts` | Toast notifications (Sonner) |

### Stores em Memória

| Store | Arquivo | Descrição |
| --- | --- | --- |
| `simulationStore` | `lib/store/simulationStore.ts` | `Map` com TTL 10min para passar questões entre `/api/simulations/create` e o client do simulado |
| `userStatsCache` | `lib/store/userStatsCache.ts` | Cache de estatísticas do usuário |

## Sistema de Pagamentos

Ver [ADR-0002](../adr/0002-payment-gateway-failover.md) para o histórico da decisão. Resumo dos arquivos envolvidos:

```text
src/app/service/payment/
  payment-gateway.types.ts    Interface IPaymentGateway + tipos normalizados + GatewayError
  mercadopago.gateway.ts      Adapter MercadoPago (DI via construtor)
  stripe.gateway.ts           Adapter Stripe (DI via construtor, API v2026-06-24.dahlia)
  payment-router.service.ts   Router com timeout + failover + singletons lazy

src/components/payment/
  StripeCardForm.tsx           Stripe Elements card form
  CreditCardForm.tsx           MercadoPago card form (fallback)

src/app/api/
  gateway-health/route.ts      Ping ambas gateways, cache ISR 30s
  webhooks/stripe/route.ts     Verifica assinatura + notifica BFF
  webhooks/mercadopago/route.ts
```

## Padrões e Convenções

### Nomenclatura de Arquivos

| Tipo | Convenção | Exemplo |
| --- | --- | --- |
| Server Component | `<Nome>DataServer.tsx` | `ProfileDataServer.tsx` |
| Client Component | `<Nome>Client.tsx` | `PricingClient.tsx` |
| Skeleton | `<Nome>Skeleton.tsx` em `components/Skeletons/` | `RankingSkeleton.tsx` |
| Hook customizado | `use<Nome>.ts` | `useUserTier.ts` |
| Tipos | `<nome>.types.ts` | `questao.types.ts` |
| Service (server-side) | `<nome>.service.ts` | `ranking.service.ts` |

**Regra de localização:** componente usado em 1 rota → `components/<NomeDaRota>/`. Usado em 2+ rotas → `components/ui/`.

### Estilização

- `cn()` de `src/lib/core/utils.ts` para merge de classes (clsx + tailwind-merge)
- Cores: sempre CSS variables de `globals.css` — nunca hex literal em componentes
- Tema: ler `data-theme` no `<html>`, nunca depender apenas de `prefers-color-scheme`
- Espaçamento/raios: tokens `--space-*`, `--radius-*` de `globals.css`
- Dynamic imports obrigatórios para: modais, drawers, Recharts, editores ricos, qualquer lib >50kb gzipped

### Performance

- Todo `<Suspense>` deve ter Skeleton Screen como fallback — nunca `null` ou spinner puro
- Múltiplos fetches em Server Component usam `Promise.all` — sem waterfalls
- Listas com >50 itens usam `react-window`
- Imagem LCP de cada rota deve ter `priority` e usar `next/image`

## Segurança

- **JWT decodificado apenas em `src/app/service/jwtDecoder.ts`** — nunca replicar lógica JWT ([ADR-0003](../adr/0003-jwt-single-decode-point.md))
- JWTs nunca armazenados em `localStorage` ou `sessionStorage` — apenas em cookies HttpOnly
- Route Handlers validam JWT antes de executar qualquer ação
- Respostas de erro ao cliente nunca expõem stack traces, IPs ou detalhes internos
- Dados pessoais de alunos nunca persistidos client-side — sempre via endpoints autenticados
- Variáveis com `NEXT_PUBLIC_` apenas para identificadores não-secretos
