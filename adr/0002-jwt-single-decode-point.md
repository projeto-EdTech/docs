# 0003. Ponto único de decodificação de JWT no frontend

- **Status:** Aceito
- **Data:** 2026-07

## Contexto

O tier do usuário (`FREE`, `TEACHER`, `ADMIN`) e outros dados de sessão vêm de um JWT emitido pelo backend Java. Decodificar o token em múltiplos lugares do frontend aumenta o risco de divergência de lógica (ex.: um componente tratando um campo ausente de forma diferente de outro) e dificulta auditar onde dados sensíveis do token são lidos.

## Decisão

Toda decodificação de JWT no frontend passa exclusivamente por `src/app/service/jwtDecoder.ts`. Nenhum outro arquivo deve reimplementar `jwt-decode` ou parsear o token manualmente. Consumidores (como o hook `useUserTier`) leem o resultado já processado, nunca o token bruto.

Complementarmente, o JWT em si nunca é armazenado em `localStorage`/`sessionStorage` — apenas em cookies `HttpOnly` (cookie `user_data`), e os Route Handlers validam o JWT antes de executar qualquer ação (ver [ADR-0001](0001-bff-proxy-pattern.md)).

## Alternativas consideradas

- **Decodificar o JWT onde for necessário** — rejeitada: gera duplicação de lógica e dificulta trocar a estrutura do token no futuro sem caçar todos os pontos de uso.

## Consequências

- Qualquer mudança na estrutura do JWT (novos campos, mudança de claim) exige alterar um único arquivo.
- Facilita auditoria de segurança: revisar o tratamento de dados de sessão do usuário significa revisar um arquivo, não o código-base inteiro.
- Novos desenvolvedores precisam conhecer essa convenção — reforçada tanto no README do frontend quanto neste ADR.
