# Banco de Imagens

Repositório de imagens (figuras, gráficos, mapas, tabelas etc.) extraídas das questões de provas de vestibulares e do ENEM. A estrutura de pastas foi organizada para espelhar exatamente a origem dos dados em `IA/service/API_GEMINI_PROVAS/Resultados`, de forma que cada imagem tenha um caminho previsível e possa ser localizada, extraída ou associada à sua questão de origem por um processo automatizado.

## Por que essa estrutura

O objetivo é permitir que o sistema extraia as imagens de cada questão de cada prova de vestibular de forma automática e determinística. Para isso, cada universidade, ano e questão tem uma pasta própria, criada a partir dos metadados dos arquivos JSON de resultados (campo `qtdeQuestoes`). Assim, um script de extração só precisa saber a universidade, o ano e o número da questão para saber exatamente onde salvar (ou onde buscar) a imagem correspondente — sem depender de nomes de arquivo variáveis ou de lógica de busca.

## Estrutura de pastas

```
Banco de Imagens/
└── {universidade}/              # ex: Fuvest, Unicamp, Unesp, Enem
    └── {ano}/                   # ex: 2023, 2024
        └── questao-{id}/        # id com 2 dígitos: questao-01, questao-02, ...
            └── {imageName}      # arquivo real da imagem, ex: figura-1.png
```

- **{universidade}**: nome da pasta da universidade/prova, igual ao usado em `Resultados/` (ex: `Fuvest`, `Unicamp`, `Unesp`, `Ita`, `Enem`, `Mackenzie`, `Puc-mg`, `Puc-sp`, `Puccamp`, `Fei`, `Famerp`, `Fatec`, `Albert_Einstein`). A pasta `Antigo` foi ignorada, pois contém provas fora do escopo atual.
- **{ano}**: ano da prova, extraído do nome do arquivo JSON correspondente (ex: `FUVEST_2024.json` → `2024`).
- **questao-{id}**: uma pasta por questão, numerada sequencialmente com 2 dígitos (`questao-01` a `questao-NN`). A quantidade de pastas por ano corresponde à soma do campo `qtdeQuestoes` de todos os arquivos JSON daquele ano/universidade (por exemplo, quando uma prova tem edições de verão e inverno, ou dia 1 e dia 2, como Enem e Puccamp, as questões são somadas e numeradas em sequência dentro do mesmo ano).
- **{imageName}**: dentro de cada `questao-{id}/` deve ficar o(s) arquivo(s) de imagem reais pertencentes àquela questão (figura, gráfico, mapa, tabela etc.), com o nome original do arquivo de imagem.

## Origem dos dados

- **Universidades e anos**: obtidos a partir das pastas e nomes de arquivo em `IA/service/API_GEMINI_PROVAS/Resultados` (pasta `Antigo` desconsiderada).
- **Quantidade de questões por ano**: obtida do campo `qtdeQuestoes` dentro de cada arquivo JSON de resultados. Quando existe mais de um arquivo para o mesmo ano (dia1/dia2, regular/inverno), os valores são somados para formar o total de pastas `questao-{id}` daquele ano.

## Resumo do conteúdo atual

| Universidade | Total de questões | Anos |
|---|---|---|
| Albert_Einstein | 250 | 2022, 2023, 2024, 2025, 2026 |
| Enem | 720 | 2022, 2023, 2024, 2025 |
| Famerp | 400 | 2022, 2023, 2024, 2025, 2026 |
| Fatec | 54 | 2023 |
| Fei | 231 | 2024, 2025 |
| Fuvest | 450 | 2022, 2023, 2024, 2025, 2026 |
| Ita | 286 | 2022, 2023, 2024, 2025, 2026 |
| Mackenzie | 180 | 2024, 2025, 2026 |
| Puc-mg | 160 | 2023, 2024, 2025, 2026 |
| Puc-sp | 150 | 2024, 2025 |
| Puccamp | 330 | 2022, 2023, 2024, 2025 |
| Unesp | 450 | 2022, 2023, 2024, 2025, 2026 |
| Unicamp | 360 | 2022, 2023, 2024, 2025, 2026 |

**Total geral: 13 universidades/provas, 4.021 pastas de questão.**

## Como usar

Para extrair ou inserir a imagem de uma questão específica, basta montar o caminho:

```
Banco de Imagens/{universidade}/{ano}/questao-{id com 2 dígitos}/
```

e salvar ali o arquivo de imagem correspondente, mantendo o nome original da imagem extraída da prova.
