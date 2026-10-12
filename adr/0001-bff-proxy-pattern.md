# 0001. Frontend como BFF Proxy do backend Java

- **Status:** Aceito
- **Data:** 2026-07

## Contexto

O frontend (Next.js) precisa consumir dados do backend Java (Spring Boot) para renderizar páginas e executar ações do usuário (simulados, ranking, pagamentos, IA). Expor o backend diretamente ao navegador exigiria distribuir credenciais e URLs internas ao cliente, e duplicar validação de JWT em dois lugares.

## Decisão

Todas as chamadas ao backend passam por Route Handlers Next.js em `src/app/api/`, que funcionam como proxies finos: sem lógica de negócio no handler, apenas repasse para uma função em `src/app/service/`, que por sua vez chama `process.env.BACKEND_API_URL`. O navegador nunca conhece a URL do backend Java nem envia requisições diretas a ele.

## Alternativas consideradas

- **Chamada direta do browser ao backend Java** — rejeitada: exigiria CORS aberto, exposição de URL interna e duplicação de lógica de autenticação no cliente.
- **API Gateway externo dedicado** — rejeitada por ora: complexidade operacional adicional não justificada no estágio atual do projeto; o Next.js já cumpre esse papel.

## Consequências

- JWT é decodificado em um único ponto (`jwtDecoder.ts`, ver [ADR-0003](0003-jwt-single-decode-point.md)), nunca no cliente.
- Segredos (chaves de API, `BACKEND_API_URL`) ficam apenas em variáveis de ambiente server-side.
- Toda página que combina dados do backend com interatividade segue o padrão `page.tsx` (Suspense) → `*DataServer.tsx` → `*Client.tsx`.
- Custo: uma camada extra de rede (browser → Next.js → Java) e Route Handlers que precisam ser mantidos em paralelo com os endpoints do backend.
