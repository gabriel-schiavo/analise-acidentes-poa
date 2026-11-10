# Acidentes de trânsito em Porto Alegre: dá para prever se houve vítima?

Projeto aplicado de Ciência de Dados com os dados abertos de acidentes de trânsito da Prefeitura de Porto Alegre (cerca de 87 mil acidentes, de 2021 a 2026).

**Grupo**: DataCore - Artur Diel, Gabriel Schiavo, Lucas Zimmermann, Vitor Mankowski

## Problema

Quais fatores estão ligados a um acidente ter vítimas (feridos ou mortos)? E, sabendo **quais veículos se envolveram, se era dia ou noite e se era fim de semana**, dá para prever se o acidente teve vítima?

Para responder, os dados são limpos e analisados com pandas e matplotlib e depois usados para treinar um modelo **KNN (scikit-learn)**, cuja acurácia é comparada com o "chute" de dizer sempre "sem vítima".

## Dados

- **Fonte:** portal de dados abertos da Prefeitura de Porto Alegre ([dadosabertos.poa.br](https://dadosabertos.poa.br)), conjunto "Acidentes de trânsito - Acidentes".
- **Arquivo:** `cat_acidentes.csv`, lido direto da URL no notebook (não é preciso baixar nada).
- **Extração usada neste trabalho:** 01/10/2026. Como o portal pode atualizar o arquivo, os números podem mudar se o notebook for executado depois.
- **Tamanho:** 87.323 linhas e 34 colunas; 83.948 linhas após a limpeza.

## Como executar

**No Google Colab:** abra o `analise_acidentes_poa.ipynb` e clique em *Ambiente de execução > Executar tudo*.

**Localmente:**

```bash
pip install -r requirements.txt
jupyter notebook analise_acidentes_poa.ipynb
```

É necessário acesso à internet, porque o CSV é baixado do portal.

## O que o notebook faz

| Etapa | Descrição |
|---|---|
| 1. Obtenção | Leitura do CSV pela URL com `pandas` |
| 2. Inspeção | Tamanho, tipos de dados e valores vazios |
| 3. Limpeza | Remove colunas não usadas, linhas sem hora, datas impossíveis e tipo "NAO CADASTRADO" (87.323 → 83.948 linhas) |
| 4. Novas colunas | `noite`, `fim_de_semana` e `hora_cheia` |
| 5. Alvo e X | `vitima` = 1 se houve ferido ou morto. X = veículos, `noite` e `fim_de_semana` |
| 6. Análise exploratória | 7 perguntas, cada uma com tabela e gráfico |
| 7. KNN (k=5) | Validação cruzada estratificada (5 partes, embaralhada) e comparação com o chute |
| 8. Melhor k | Testa os ímpares de 1 a 29 (em amostra de 20 mil acidentes) |
| 9. Predict | Previsão para 2 acidentes inventados |
| 10. Conclusões | Interpretação dos resultados |

**Sobre o vazamento de informação:** `feridos`, `mortes` e semelhantes, que definem o alvo, ficam fora do X. A coluna `cont_vit` também: ela coincide com o alvo em 99,999% dos acidentes, então usá-la seria entregar a resposta ao modelo.

## Principais resultados

**Análise exploratória** (percentual de acidentes com vítima; a média geral é 35,3%):

- **Veículo:** bicicleta 90,4%, patinete 80,3%, moto 76,6%. Já táxi (12,7%) e caminhão (13,4%) ficam abaixo da média.
- **Dia/noite:** 45,0% à noite contra 31,5% de dia.
- **Fim de semana:** 42,6% contra 33,6% nos dias úteis.
- **Horário:** mínimo às 10h (25,9%) e máximo às 21h (52,3%).
- **Região:** Sul 40,1% e Centro 26,8%.
- **Ano:** estável de 2021 a 2024, com queda em 2025 e 2026 (2026 é parcial, até outubro).

**Modelo KNN:**

| | Acurácia | Chute (sempre "sem vítima") |
|---|---|---|
| k = 5 (todos os dados) | 81,2% | 64,7% |
| melhor k = 23 (amostra de 20 mil) | 82,8% | 65,0% |

O KNN supera o chute em mais de 16 pontos percentuais, então veículos, dia/noite e fim de semana ajudam a prever se houve vítima. Nos acidentes inventados, o modelo previu **sem vítima** para 2 autos à noite em fim de semana (22% de probabilidade de vítima) e **com vítima** para moto + auto em dia útil (74%).

## Limitações

- O modelo usa poucas variáveis e muitos acidentes têm características idênticas.
- Só a acurácia foi medida; não há recall nem matriz de confusão da classe "com vítima".
- O melhor k foi escolhido na própria validação cruzada, então a nota é levemente otimista.
- As relações encontradas são associações, não provam causa.

## Arquivos

- `analise_acidentes_poa.ipynb`: notebook completo, com código, saídas e conclusões
- `requirements.txt`: bibliotecas necessárias
- `README.md`: este arquivo
