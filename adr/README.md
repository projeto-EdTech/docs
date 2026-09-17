# Architecture Decision Records (ADRs)

Registro histórico das decisões técnicas relevantes do projeto Vestibuline. Cada ADR descreve um contexto, a decisão tomada, e as consequências — para que decisões antigas não precisem ser reconstruídas de memória ou reabertas sem necessidade.

## Índice

| ADR | Título | Status |
| --- | --- | --- |
| [0001](0001-bff-proxy-pattern.md) | Frontend como BFF Proxy do backend Java | Aceito |
| [0002](0002-payment-gateway-failover.md) | Failover automático entre gateways de pagamento | Aceito |
| [0003](0003-jwt-single-decode-point.md) | Ponto único de decodificação de JWT no frontend | Aceito |

## Como criar um novo ADR

1. Copie o [template](template.md) para um novo arquivo `NNNN-titulo-curto.md` (numeração sequencial, 4 dígitos).
2. Preencha contexto, decisão e consequências — sem reescrever a decisão depois que ela foi tomada; se ela mudar, crie um novo ADR que a substitui e referencie o antigo.
3. Adicione a linha correspondente na tabela acima.
4. Abra um PR neste repositório para revisão.
