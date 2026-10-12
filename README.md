# Vestibuline — Documentação

Índice central de documentação técnica do projeto Vestibuline (plataforma de preparação para vestibulares: simulados, IA pedagógica, gamificação e mini-games, planner de estudos e consultor de notas de corte).

## Repositórios

| Repositório | Descrição |
| --- | --- |
| [frontend](https://github.com/projeto-EdTech/frontend) | Next.js — interface web (BFF) |
| [backend](https://github.com/projeto-EdTech/backend) | Spring Boot (Java 21) — API principal |
| [IA](https://github.com/projeto-EdTech/IA) | Serviço de IA e aprendizado personalizado |
| [APPS](https://github.com/projeto-EdTech/APPS) | Aplicações mobile/desktop multiplataforma |

## Índice

- [Arquitetura](architecture/README.md) — visão geral do sistema e fluxogramas
  - [Arquitetura do Frontend](architecture/frontend.md)
- [Decisões técnicas (ADRs)](adr/README.md) — histórico de decisões de arquitetura

## Como contribuir com a documentação

1. Toda mudança de arquitetura relevante (nova stack, novo padrão, mudança de fluxo de dados) deve ser registrada como um ADR em [`adr/`](adr/README.md), usando o [template](adr/template.md).
2. Diagramas usam [Mermaid](https://mermaid.js.org/) embutido em Markdown, para renderizar direto no GitHub sem imagens externas.
3. Documentação específica de um repositório (ex.: setup local, scripts, variáveis de ambiente) continua no `README.md` daquele repositório — aqui ficam apenas arquitetura, decisões e visão de sistema.

