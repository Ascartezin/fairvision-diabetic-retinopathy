> Nota: arquivo commitado em 05/11/2026, após a geração das predições de teste. O histórico do Git não comprova que o protocolo antecede o teste; as análises de equidade devem ser lidas com essa ressalva.

# Protocolo de equidade — Artigo 3 (IA II, UnB)

Viés demográfico na detecção de retinopatia diabética com risco de perda visual (vtDR).

## 0. Registro da ordem dos fatos (leia primeiro)

Este arquivo foi escrito e commitado **depois** de gerar as predições de teste (Etapa 8) e de rodar a auditoria por grupo (Etapa 9). Por isso ele **não é um pré-registro**. As definições abaixo (hipóteses, métricas, limiar, bootstrap e permutação) já constavam do guia de planejamento do trabalho, escrito em 04/11/2026, antes de qualquer execução. O guia está em `docs/Guia_Unificado_Artigo_3.pdf`.

Ordem real da execução (05/11/2026):

1. Treino do baseline e do mitigado; escolha do checkpoint pelo maior AUROC na validação.
2. Limiar único por modelo escolhido na **validação** (índice de Youden) e salvo em `results/limiares.json`.
3. Predições de teste geradas **uma única vez** por modelo e salvas.
4. Auditoria por grupo (Etapa 9) rodada **uma única vez**, sem alterar o código depois de ver os resultados.
5. Comparação pareada baseline × mitigado (Etapa 10), com os mesmos índices de bootstrap.
6. Este arquivo commitado. Data e hash do commit: `[preencher]`.

## 1. Pergunta e hipóteses

**Pergunta.** Um modelo que detecta vtDR em imagens SLO tem desempenho diferente entre grupos demográficos, e essas diferenças excedem o esperado ao acaso? Uma reponderação simples por raça × classe reduz essas diferenças, e a que custo?

- **H1.** A sensibilidade é menor em Asian e Black do que em White.
- **H2.** A reponderação reduz alguns gaps, mas pode custar em outras métricas.
- Sexo: análise descritiva.

Não se afirma previamente que existe viés nem se promete eliminá-lo. Resultados contrários às hipóteses serão reportados.

## 2. Dados e rótulo

- Harvard-FairVision, subconjunto DR, apenas imagem SLO (Luo et al., arXiv:2310.02492).
- 10.000 sujeitos, um registro por sujeito segundo a documentação do dataset. Divisão oficial: treino 6000, validação 1000, teste 3000 (60/10/30), sem interseção (verificada no notebook).
- Rótulo binário conforme fornecido pelo dataset: "vision threatening dr" = 1 (909 sujeitos) e "non-vision threatening dr" = 0 (9091).
- Teste: 261 positivos (Asian 16, Black 59, White 186; feminino 118, masculino 143).

## 3. Grupos

- Primários: raça (Asian, Black, White) e sexo (feminino, masculino). Categorias autodeclaradas e grosseiras.
- Etnia e idioma: não analisados.
- Raça e sexo **não** entram como entrada do modelo. Idade não foi usada como covariável nem como entrada.

## 4. Modelos

- ResNet18 pré-treinada no ImageNet, fine-tuning completo, saída de 1 logit, perda BCE ponderada.
- Adam, lr 1e-4, batch 32, até 20 épocas, paciência 5, seed 42, AMP, Colab com GPU T4.
- Baseline: peso inversamente proporcional à frequência da classe (média dos pesos = 1).
- Mitigado: peso inversamente proporcional à frequência de cada célula raça × classe, `N / (6 · n_célula)`, com teto de 10. O teto atuou apenas no grupo Asian positivo (n = 41; peso bruto 24,4). Os pesos não foram renormalizados após o teto.
- Pesos por célula em `results/tabelas/pesos_mitigado.csv`.

## 5. Limiar

Único por modelo, escolhido na validação pelo índice de Youden, congelado antes do teste: baseline 0,114 e mitigado 0,305. Os limiares são diferentes, então diferenças entre modelos misturam o efeito da reponderação com o ponto de operação.

## 6. Métricas e convenções

Por grupo, no teste: n, positivos, prevalência, AUROC, TPR, FPR e PPV.

- **ΔTPR** (igualdade de oportunidade) = máx − mín do TPR entre grupos.
- **ΔFPR** = máx − mín do FPR entre grupos.
- **Equalized odds** = máximo entre ΔTPR e ΔFPR. Por ser dominado pela componente mais ruidosa, ΔTPR e ΔFPR são reportados também em separado.
- **ΔAUC** = máx − mín do AUROC entre grupos.
- **Diferença com sinal** contra White (raça: Black − White e Asian − White) e masculino − feminino (sexo). É a medida principal de interpretação.
- Paridade demográfica: não reportada neste trabalho (as prevalências diferem entre grupos).

## 7. Incerteza e controle contra o acaso

- **IC 95% por bootstrap** (percentis 2,5 e 97,5): 1000 reamostragens dos sujeitos do teste, seed 42. Os mesmos índices foram usados para os dois modelos, o que torna a comparação pareada.
- **Permutação** do rótulo de grupo entre sujeitos, 1000 vezes, p = (contagem + 1) / (1000 + 1). Para estatísticas de amplitude (ΔTPR, ΔFPR, equalized odds) é unilateral (perm ≥ obs). Para diferenças com sinal é bilateral (|perm| ≥ |obs|).
- O AUROC é avaliado apenas por IC de bootstrap, sem permutação.
- Nenhuma correção para comparações múltiplas foi aplicada. Os p-valores são indícios e não evidência confirmatória.

## 8. Regras de interpretação

- Um gap máx − mín nunca é negativo e tende a ser positivo mesmo sem viés; o controle contra o acaso é a permutação.
- A permutação não diz **por que** um gap existe. Causas (prevalência, idade, qualidade de imagem, equipamento, rótulos) são hipóteses não testadas.
- Com poucos positivos (Asian: 16 no teste, 2 na validação) o resultado pode ser inconclusivo, e isso será dito.
- Sem afirmação de benefício ou dano clínico. O modelo não é ferramenta clínica; uso apenas de pesquisa.

## 9. O que é exploratório

Foi definido ou calculado depois de ver resultados do teste e deve ser descrito como exploratório no artigo:

- A comparação pareada baseline × mitigado com IC de bootstrap da diferença.
- A razão de FPR Black/White e a observação sobre o peso total da classe positiva no mitigado.
- Qualquer análise por idade ou por outro atributo, caso seja feita.
