# MATD50 • MLG (Prova 2) — Lista Longa de Exercícios Abertos Discursivos

Material inspirado diretamente nas **Listas 3, 4 e 5**, **Atividades 4 a 8** e **Desafio 3** da Profa. Rosemeire L. Fiaccone (`mlgp2.pdf`), com saídas geradas em **R 4.5.3**. Estrutura de treino em 3 níveis por questão: **Primeiro Passo → Dica & Armadilhas → Gabarito Completo + Texto de Prova**.

---

## Questão 1 — Binomial Agrupada (φ = 1): ANODEV Sequencial, Preenchimento de Tabela e Adequação Global
**Bloco:** Bloco I • ANODEV & φ̂ | **Inspiração:** Inspirada na Lista 4 (Q3) e Atividade 5 • Profa. Rosemeire

Em uma coorte perinatal hospitalar com $n = 5.420$ nascidos vivos. os dados foram agrupados segundo três fatores dicotômicos maternos: `Pre_natal` (Adequado vs. Inadequado). `Hipertensao` (Não vs. Sim) e `Tabagismo` (Não vs. Sim). formando $N = 2 \times 2 \times 2 = 8$ subpopulações distintas ($y_i$ recém-nascidos com baixo peso dentre $m_i$ gestantes no estrato $i$). Deseja-se avaliar os fatores associados à probabilidade de baixo peso ao nascer via Modelos Lineares Generalizados.

```r
Analysis of Deviance Table
Model: binomial, link: logit
Response: cbind(baixo_peso, normal)
Terms added sequentially (first to last)

            Df Deviance Resid. Df Resid. Dev  Pr(>Chi)
NULL                          [A]    282.749
Pre_natal    1  183.458         6        [B] < 2.2e-16 ***
Hipertensao  1      [C]         5      5.986 < 2.2e-16 ***
Tabagismo    1    2.895         4        [D]   0.08887 .

--- Modelos complementares (mesmo banco agrupado, N = 8 subpopulacoes) ---
Modelo com todas as interacoes duplas:  Residual Deviance = 0.115  em 1 gl
Modelo saturado (interacao tripla):    Residual Deviance = 0.000  em 0 gl
Estatistica X^2 de Pearson (Modelo aditivo sem Tabagismo): X^2 = 6.104
Estatistica X^2 de Pearson (Modelo aditivo com 3 fatores): X^2 = 3.112
```

### Subitens Discursivos
- **a)** Especifique formalmente os **três componentes do MLG** adotado (componente aleatório com parâmetro de dispersão $\phi$, função de ligação canônica e componente sistemático do modelo aditivo completo), indicando a função de variância $V(\mu_i)$ e o domínio de $\pi_i$.
- **b)** Complete os valores omitidos `[A]`, `[B]`, `[C]` e `[D]` na tabela de Análise do Desvio (ANODEV), detalhando os cálculos algébricos. Justifique por que o modelo nulo apresenta apenas $[A]$ graus de liberdade e não $5.419$.
- **c)** Ao nível de significância $\alpha = 5\%$ (lembre-se: $\chi^2_{1; 0,95} = 3,841$ e $\chi^2_{3; 0,95} = 7,815$), realize formalmente o Teste da Razão de Verossimilhanças (TRV / $\Delta D$) para: (i) testar a inclusão conjunta de todas as interações de 2ª ordem em relação ao modelo aditivo de 3 fatores; e (ii) testar a remoção de `Tabagismo` do modelo aditivo, declarando $H_0$, $H_1$ e o modelo mais parcimonioso selecionado.
- **d)** Avalie a **qualidade global do ajuste** (bondade de ajuste) do modelo final selecionado utilizando tanto a deviance residual quanto a estatística $X^2$ de Pearson (considere $\chi^2_{5; 0,95} = 11,070$). Explique por que nesse banco esse teste global $\chi^2$ é válido, ao contrário de dados binários individuais.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Comece identificando o formato da variável resposta no comando do R: `cbind(baixo_peso, normal)` e `family = binomial(link = "logit")`. Isso caracteriza dados **binomiais agrupados em proporções** (ou contagens $Y_i \sim \text{Bin}(m_i, \pi_i)$ com $m_i > 1$), nos quais o parâmetro de dispersão teórico é conhecido ($\phi = 1$).

                
**Item b):** Olhe para a lógica telescópica (sequencial) da tabela `anova()`: em cada linha $k$, o desvio residual diminui exatamente pelo valor da coluna `Deviance` da linha seguinte: $D_k = D_{k-1} - \Delta D_k$. Para `[A]`, lembre que no modelo em subpopulações a unidade observacional do MLG é cada grupo ($N = 8$), não cada indivíduo ($5.420$).

                
**Item c):** Para comparar modelos encaixados em família Binomial com $\phi = 1$, a diferença de desvios $\Delta D = D_{\text{reduzido}} - D_{\text{completo}}$ segue distribuição assintótica $\chi^2_r$ sob $H_0$, com $r = gl_{\text{red}} - gl_{\text{comp}}$ parâmetros extras testados. Verifique se $D_{\text{aditivo}} - D_{\text{int2}}$ e $\Delta D_{\text{Tabagismo}}$ ultrapassam os quantis críticos.

                
**Item d):** Compare diretamente a deviance residual $D_p$ e a estatística $X^2$ do modelo escolhido com $\chi^2_{gl_{res}; 0,95} = 11,070$. Pergunte-se: qual é a regra para dizer que o modelo ajusta bem ($p > 0,05$, ou $D_p < \chi^2_{gl, 0,95}$)? Por que aqui $m_i$ grande permite usar $\chi^2_{N-p}$?

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Fórmulas-chave:** Para $Y_i / m_i$ (proporção observada) ou $Y_i \sim \text{Bin}(m_i, \pi_i)$, temos $\mu_i = m_i \pi_i$, função de variância $V(\mu_i) = \mu_i(1 - \mu_i/m_i)$ (ou $V(\pi_i) = \pi_i(1-\pi_i)$ na escala da proporção com peso $m_i$) e ligação canônica $\eta_i = \ln\!\left(\frac{\pi_i}{1-\pi_i}\right)$.

                
**Armadilha clássica (N vs. n):** Nunca coloque $gl_N = 5.420 - 1 = 5.419$! Quando os dados estão agrupados em $N = 8$ perfis covariáveis, a log-verossimilhança saturada atribui 1 parâmetro para cada um dos $N = 8$ perfis, logo $gl_N = N - 1 = 7$.

                
**Cuidado na leitura da tabela ANODEV:** Quando comparamos dois modelos com $\phi=1$ conhecido, usamos o teste **Qui-quadrado ($\Delta D \sim \chi^2_r$)** e *não* o teste $F$ (pois $F$ só entra quando $\phi$ é desconhecido e precisa ser estimado!).

                
**Pegadinha da Bondade de Ajuste:** Enquanto nos testes de variáveis queremos $\Delta D$ *grande* ($p  0,05$, não rejeitar $H_0$)!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Especificação Formal dos 3 Componentes do MLG:**

                  

                    
**Componente Aleatório:** $Y_i \sim \text{Binomial}(m_i, \pi_i)$ independentes para $i = 1, \dots, 8$, em que $Y_i$ é o total de nascidos com baixo peso na $i$-ésima subpopulação de tamanho $m_i$. Como a família Binomial pertence à família exponencial com dispersão fixa, $\phi = 1$. A média é $\mu_i = m_i\pi_i$, a função de variância na forma de contagens é $V(\mu_i) = \mu_i(1 - \mu_i/m_i)$ e $\pi_i \in (0, 1)$ (logo $\mu_i \in (0, m_i)$).

                    
**Função de Ligação Canônica (Logito):** $g(\pi_i) = \ln\!\left(\dfrac{\pi_i}{1 - \pi_i}\right) = \eta_i$, levando à inversa $\pi_i = \dfrac{\exp(\eta_i)}{1 + \exp(\eta_i)}$.

                    
**Componente Sistemático:** $\eta_i = \beta_0 + \beta_1 I(\text{Pre\_natal}_i = \text{Inadequado}) + \beta_2 I(\text{Hipertensao}_i = \text{Sim}) + \beta_3 I(\text{Tabagismo}_i = \text{Sim})$.

                  

                

                

                  
**b) Preenchimento de `[A]`, `[B]`, `[C]`, `[D]` e Justificativa de $gl_N$:**

                  

                    
`[A]` $= N - 1 = 8 - 1 = \mathbf{7}$ graus de liberdade. Como o banco está agregado em $N = 8$ subpopulações distintas ($2^3$), o modelo saturado consome $8$ parâmetros (um $\pi_i$ por célula) e o nulo consome $1$ (apenas o intercepto), restando $8 - 1 = 7$ gl. (Os $5.420$ indivíduos já estão sintetizados nos denominadores binomiais $m_i$).

                    
`[B]` $= D_{\text{NULL}} - \Delta D_{\text{Pre\_natal}} = 282.749 - 183.458 = \mathbf{99.291}$.

                    
`[C]` $= D_{\text{Pre\_natal}} - D_{\text{Pre\_natal+Hip}} = 99.291 - 5.986 = \mathbf{93.305}$.

                    
`[D]` $= D_{\text{Pre\_natal+Hip}} - \Delta D_{\text{Tabagismo}} = 5.986 - 2.895 = \mathbf{3.091}$.

                  

                

                

                  
**c) Testes da Razão de Verossimilhanças ($\Delta D \sim \chi^2_r$ sob $H_0$) e Seleção:**

                  

                    
**(i) Teste das interações duplas ($r = 4 - 1 = 3$ gl):** $H_0$: todos os coeficientes de interação dupla são nulos vs. $H_1$: pelo menos uma interação dupla difere de zero.

                    $\Delta D = D_{\text{aditivo}} - D_{\text{int2}} = 3.091 - 0.115 = \mathbf{2.976}$. Comparando com $\chi^2_{3; 0,95} = 7,815$, como $2,976  0,05$), **não rejeitamos $H_0$**: as interações duplas não contribuem significativamente.

                    
**(ii) Teste de `Tabagismo` ($r = 5 - 4 = 1$ gl):** $H_0: \beta_3 = 0$ vs. $H_1: \beta_3 \neq 0$.

                    $\Delta D = 5.986 - 3.091 = \mathbf{2.895}$. Como $2,895  0,05$), ao nível de $5\%$ **não rejeitamos $H_0$**. Pelo princípio da parcimônia, selecionamos o modelo **`Pre_natal + Hipertensao`** ($gl_{\text{res}} = 5$, $p = 3$ parâmetros).

                  

                

                

                  
**d) Adequação Global e Texto Padrão para Prova:**

                  
No modelo final selecionado (`Pre_natal + Hipertensao`, $gl = 5$), temos $D = 5.986$ e $X^2 = 6.104$. Ambas estão muito abaixo do quantil crítico $\chi^2_{5; 0,95} = 11,070$ ($p_D \approx 0,308 > 0,05$ e $p_{X^2} \approx 0,296 > 0,05$).

                  

                    **📝 Como redigir na prova:** “Como os dados estão agrupados em $N = 8$ subpopulações com elevados efetivos $m_i$ (todos os valores esperados $\hat{\mu}_i$ e $m_i - \hat{\mu}_i$ são amplamente superiores a 5), a aproximação assintótica da deviance residual e da estatística de Pearson pela distribuição $\chi^2_{N-p} = \chi^2_5$ sob $H_0$ (modelo bem ajustado) é plenamente válida. Como $D = 5,986 < \chi^2_{5; 0,95} = 11,070$ e $X^2 = 6,104 < 11,070$, não há evidências para rejeitar a hipótese de bom ajuste, concluindo-se que o modelo logístico aditivo contendo apenas pré-natal e hipertensão materna descreve adequadamente a probabilidade de baixo peso ao nascer.”

</details>

---

## Questão 2 — Gama com Ligação Log (φ Desconhecido): Deviance Escalonada vs. Não-Escalonada, φ̂ Maximal e Teste F
**Bloco:** Bloco I • ANODEV & φ̂ | **Inspiração:** Inspirada na Lista 3 (Q3) e Atividade 4 • Profa. Rosemeire

Um hospital avaliou o custo total de internação em UTI ($Y_i > 0$, em milhares de reais) de $n = 65$ pacientes em função de `Idade` (contínua, anos), `Gravidade` (Baixa [ref.], Moderada, Alta) e `Infeccao` hospitalar (Não [ref.], Sim). Ajustou-se um MLG com distribuição **Gama** e ligação **logarítmica**. Analise as saídas do R abaixo.

```r
Analysis of Deviance Table
Model: Gamma, link: log
Response: Custo (em milhares de R$)
Terms added sequentially (first to last)

                Df Deviance Resid. Df Resid. Dev       F    Pr(>F)
NULL                               64    27.2466
Idade            1   4.0127        63    23.2339 38.3770 6.489e-08 ***
Gravidade        2  12.3632        61        [A] 59.1206 1.005e-14 ***
Infeccao         1      [B]        60     7.1254 35.8198 1.443e-07 ***
Idade:Gravidade  2   0.5210        58     6.6044     [C]   0.09163 .

--- Informacoes adicionais do Modelo Maximal (p_M = 7 parametros, gl = 58) ---
Soma de quadrados dos residuos de Pearson (X^2_P): 6.0644
Desvio residual do modelo maximal (D_M):           6.6044

--- Coeficientes estimados do Modelo Aditivo (sem a interacao Idade:Gravidade) ---
                  Estimate Std. Error t value Pr(>|t|)
(Intercept)       1.642593   0.152415  10.777  < 2e-16 ***
Idade             0.015441   0.002749   5.616 5.26e-07 ***
GravidadeModerada 0.455315   0.098126   4.640 1.94e-05 ***
GravidadeAlta     1.080359   0.111566   9.684 7.31e-14 ***
InfeccaoSim       0.538571   0.093270   5.774 2.91e-07 ***
```

### Subitens Discursivos
- **a)** Distinga matematicamente a **deviance não-escalonada** $D(\mathbf{y}; \hat{\boldsymbol{\mu}})$ da **deviance escalonada** $D^*(\mathbf{y}; \hat{\boldsymbol{\mu}})$ na família Gama, explicando por que na saída de `anova(mod, test="F")` a coluna `Deviance` não pode ser comparada diretamente com uma distribuição $\chi^2$.
- **b)** Estime o parâmetro de dispersão $\hat{\phi}$ no R pelos dois métodos clássicos — **método do desvio** ($\hat{\phi}_D$) e **método de Pearson generalizado** ($\hat{\phi}_P$) — utilizando obrigatoriamente o modelo apropriado. Explique: (i) por que não se deve estimar $\hat{\phi}$ em submodelos intermediários; (ii) qual dos dois estimadores o R utiliza por padrão em `anova(..., test="F")` e `summary()`; e (iii) sob qual condição o teste global de Cordeiro & McCullagh ($D^* \sim \chi^2_{n-p}$) seria aplicável.
- **c)** Calcule os valores omitidos `[A]`, `[B]` e `[C]` na tabela ANODEV. Considerando os quantis críticos $F_{2, 58; 0,95} = 3,156$, interprete o teste para a interação `Idade:Gravidade` e para o fator categórico `Gravidade` ao nível de $5\%$, justificando o modelo escolhido.
- **d)** A partir dos coeficientes do modelo aditivo selecionado, interprete quantitativamente em escala real (reais) o efeito da ocorrência de `InfeccaoSim` e de um aumento de $10$ anos na `Idade` sobre o custo médio esperado da internação em UTI.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Lembre da relação fundamental $D^*(\mathbf{y}; \hat{\boldsymbol{\mu}}) = \frac{D(\mathbf{y}; \hat{\boldsymbol{\mu}})}{\phi}$. Na tabela ANODEV do R, as colunas `Deviance` e `Resid. Dev` exibem sempre o desvio **não-escalonado** $D$. Como na Gama $\phi$ é um parâmetro de incômodo desconhecido ($\phi \neq 1$), dividir $\Delta D$ por $1$ não elimina a variância relativa dos dados!

                
**Item b):** Localize primeiro qual linha representa o **modelo maximal** (com todas as covariáveis + interação) para ler $D_M = 6.6044$, $X^2_M = 6.0644$ e $gl_M = n - p_M = 58$. Calcule $\hat{\phi}_D = D_M / gl_M$ e $\hat{\phi}_P = X^2_M / gl_M$. Compare $\hat{\phi}_P$ com $\frac{\Delta D / \Delta gl}{F}$ na tabela para descobrir qual o R usou!

                
**Item c):** Para `[A]` e `[B]`, use a subtração telescópica linha a linha na coluna `Resid. Dev`. Para `[C]`, use a fórmula da estatística $F$ parcial para modelos encaixados com $\phi$ desconhecido: $F = \frac{(D_0 - D_1) / r}{\hat{\phi}_M}$.

                
**Item d):** Como a função de ligação é $\ln(\mu_i) = \eta_i$ (ligação **logarítmica**), a média $\mu_i = \exp(\eta_i)$ reage de forma **multiplicativa** às covariáveis. Calcule $\exp(\hat{\beta})$ — jamais some ou subtraia $\hat{\beta}$ diretamente nos reais!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Pegadinha mortal do $\hat{\phi}$:** Mesmo quando a interação `Idade:Gravidade` não é significativa ($p = 0,0916$), o denominador da estatística $F$ em toda a tabela ANODEV é $\hat{\phi}_P$ calculado no **modelo maximal** ($gl = 58$), porque se usássemos um submodelo com variáveis omitidas importantes no denominador, o viés inflaria $\hat{\phi}$, reduzindo $F$ artificialmente e mascarando efeitos reais!

                
**R vs. Estimadores:** Em famílias com $\phi$ desconhecido (Normal, Gama, Normal Inversa, Quasi), o R usa por padrão o estimador de **Pearson generalizado** $\hat{\phi}_P = \frac{1}{n-p_M} \sum \frac{(y_i - \hat{\mu}_i)^2}{V(\hat{\mu}_i)}$ no denominador de $F$ e nos erros-padrão de `summary()`.

                
**Fórmula da Deviance Gama:** $D(\mathbf{y}; \hat{\boldsymbol{\mu}}) = 2 \sum_{i=1}^n \left[ -\ln\left(\frac{y_i}{\hat{\mu}_i}\right) + \frac{y_i - \hat{\mu}_i}{\hat{\mu}_i} \right]$.

                
**Cordeiro & McCullagh:** A aproximação $D^* \sim \chi^2_{n-p}$ para testar adequação global só tem uso direto quando $\phi$ é **conhecido** (ou possui boa aproximação independente), caso contrário em amostras pequenas com $\phi$ desconhecido os graus de liberdade corrigidos dependem de $V(\mu)$ e $\phi$.

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Deviance Não-Escalonada vs. Escalonada na Gama:**

                  
Na família Gama ($V(\mu_i) = \mu_i^2$), a **deviance não-escalonada** é $D(\mathbf{y};\hat{\boldsymbol{\mu}}) = 2\sum_{i=1}^n \left[-\ln(y_i/\hat{\mu}_i) + (y_i - \hat{\mu}_i)/\hat{\mu}_i\right]$, que mede a discrepância pura em função apenas de $\mathbf{y}$ e $\hat{\boldsymbol{\mu}}$. Já a **deviance escalonada** divide essa medida pelo parâmetro de dispersão: $D^*(\mathbf{y};\hat{\boldsymbol{\mu}}) = \frac{D(\mathbf{y};\hat{\boldsymbol{\mu}})}{\phi} \approx \chi^2_{n-p}$. Como na Gama $\phi$ é desconhecido (diferente de $1$), $\Delta D$ sozinho ainda está na escala multiplicada por $\phi$ e **não** segue $\chi^2_r$; ao dividir $\Delta D / r$ por $\hat{\phi}$ estimado no maximal, obtemos a estatística $F \approx \frac{\chi^2_r/r}{\chi^2_{n-p_M}/(n-p_M)} \sim F_{r, n-p_M}$.

                

                

                  
**b) Estimação de $\hat{\phi}$ no Modelo Maximal ($p_M = 7$, $gl_M = 65 - 7 = 58$):**

                  

                    
**Método do Desvio:** $\hat{\phi}_D = \dfrac{D_M}{n - p_M} = \dfrac{6.6044}{58} = \mathbf{0.11387}$ ($\approx 0,1139$).

                    
**Método de Pearson Generalizado (padrão do R):** $\hat{\phi}_P = \dfrac{X^2_M}{n - p_M} = \dfrac{6.0644}{58} = \mathbf{0.10456}$ ($\approx 0,1046$).

                    
**Por que usar o maximal e qual o R usa:** O R adota $\hat{\phi}_P = 0,1046$ porque é um estimador baseado em momentos robusto a assimetrias de pequenas amostras (em contraste com $\hat{\phi}_D$ que pode inflar quando existem valores próximos de zero). Usa-se exclusivamente o modelo maximal ($p_M = 7$) porque ele é o menos propenso a viés por omissão de variáveis relevantes; um submodelo mal especificado inflaria artificialmente $\hat{\phi}$, reduzindo o poder do teste $F$.

                    
**Cordeiro & McCullagh ($D^* \sim \chi^2_{n-p}$):** Só permite testar a bondade de ajuste global diretamente se $\phi$ for conhecido (ou muito bem determinado externamente), pois substituir $\phi$ por $\hat{\phi}_D$ faz a razão $D/\hat{\phi}_D = n-p$ virar uma identidade tautológica sem poder de teste!

                  

                

                

                  
**c) Preenchimento de `[A]`, `[B]`, `[C]` e Testes $F$:**

                  

                    
`[A]` $= D_{\text{Idade}} - \Delta D_{\text{Gravidade}} = 23.2339 - 12.3632 = \mathbf{10.8707}$.

                    
`[B]` $= D_{\text{Idade+Grav}} - D_{\text{Aditivo}} = 10.8707 - 7.1254 = \mathbf{3.7453}$.

                    
`[C]` $= \dfrac{\Delta D_{\text{Idade:Grav}} / 2}{\hat{\phi}_P} = \dfrac{0.5210 / 2}{0.10456} = \dfrac{0.26049}{0.10456} = \mathbf{2.4913}$.

                    
**Decisão para `Idade:Gravidade`:** $H_0$: coeficientes da interação são nulos ($2$ gl). Como $F = 2,4913  0,05$), **não rejeitamos $H_0$**: o efeito proporcional da idade no custo não depende do nível de gravidade.

                    
**Decisão para `Gravidade`:** $H_0$: não há efeito de gravidade ($\beta_{\text{Mod}} = \beta_{\text{Alta}} = 0$, $2$ gl). Como $F = 59,1206 \gg 3,156$ ($p = 1,005 \times 10^{-14} < 0,05$), **rejeitamos $H_0$** com fortíssima evidência. Selecionamos o **modelo aditivo** `Idade + Gravidade + Infeccao` ($p=5$ parâmetros).

                  

                

                

                  
**d) Interpretação Multiplicativa dos Efeitos:**

                  

                    
`InfeccaoSim`: $\exp(0.538571) = \mathbf{1.7136}$. Mantidas constantes a idade e a gravidade, pacientes que desenvolvem infecção hospitalar têm custo médio esperado de internação **71,4% maior** (multiplicado por $1,71$) do que pacientes sem infecção.

                    
`Idade` (salto de 10 anos): $\exp(10 \times 0.015441) = \exp(0.1544) = \mathbf{1.1670}$. A cada 10 anos adicionais de idade, para o mesmo perfil clínico, o custo médio esperado aumenta em **16,7%**.

                  

                  

                    **📝 Como redigir na prova:** “Na ANODEV do MLG Gama com ligação log, utilizando o estimador de dispersão de Pearson no modelo maximal ($\hat{\phi}_P = 0,1046$ em $58$ gl), a interação Idade:Gravidade não se mostrou estatisticamente significativa ao nível de 5% ($F_{2,58} = 2,491$, $p = 0,0916$). Adotando o modelo aditivo mais parcimonioso, todas as covariáveis principais são altamente significativas, estimando-se que a infecção hospitalar eleva o custo médio esperado em 71,4% ($e^{\hat{\beta}} = 1,714$) e que pacientes de gravidade alta custam, em média, 2,95 vezes o valor de pacientes de gravidade baixa.”

</details>

---

## Questão 3 — ANODEV Tipo I (Dependência da Ordem) vs. Tipo II Marginal (drop1) e Reconstrução Manual do AIC sob Colinearidade
**Bloco:** Bloco I • ANODEV & φ̂ | **Inspiração:** Inspirada na Lista 3 (Q5) e Atividade 6 • Profa. Rosemeire

Deseja-se modelar o número anual de acidentes noturnos ($Y_i$) em $n = 80$ cruzamentos urbanos por meio de regressão de **Poisson** com ligação logarítmica, avaliando as covariáveis `Fluxo` (milhares de veículos/dia), `Velocidade` média da via (km/h) e `Iluminacao` LED (Não vs. Sim). Sabe-se que `Fluxo` e `Velocidade` possuem fortíssima correlação linear ($r = 0.9507$). Compare as saídas Tipo I (duas ordens de entrada) e Tipo II.

```r
=== SAIDA 1: ANODEV Tipo I (Ordem: Fluxo + Velocidade + Iluminacao) ===
Analysis of Deviance Table

Model: poisson, link: log

Response: Acidentes

Terms added sequentially (first to last)


           Df Deviance Resid. Df Resid. Dev  Pr(>Chi)    
NULL                          79    217.010              
Fluxo       1  116.760        78    100.250 < 2.2e-16 ***
Velocidade  1    0.651        77     99.599    0.4196    
Iluminacao  1   32.986        76     66.612 9.281e-09 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

=== SAIDA 2: ANODEV Tipo I (Ordem invertida: Velocidade + Fluxo + Iluminacao) ===
Analysis of Deviance Table

Model: poisson, link: log

Response: Acidentes

Terms added sequentially (first to last)


           Df Deviance Resid. Df Resid. Dev  Pr(>Chi)    
NULL                          79    217.010              
Velocidade  1  100.426        78    116.585 < 2.2e-16 ***
Fluxo       1   16.986        77     99.599 3.766e-05 ***
Iluminacao  1   32.986        76     66.612 9.281e-09 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

=== SAIDA 3: ANODEV Tipo II / Deletamento unitario — drop1(mod, test="Chisq") ===
Single term deletions

Model:
Acidentes ~ Fluxo + Velocidade + Iluminacao
           Df Deviance    AIC    LRT  Pr(>Chi)    
<none>          66.612 338.92                     
Fluxo       1   84.700 355.01 18.087 2.110e-05 ***
Velocidade  1   66.779 337.09  0.167    0.6831    
Iluminacao  1   99.599 369.91 32.986 9.281e-09 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

=== AIC e Medidas de Ajuste do Modelo sem Velocidade (Acidentes ~ Fluxo + Iluminacao) ===
Residual deviance: 66.779  on 77 degrees of freedom
Pearson X^2:       61.105
AIC:               [CALCULAR NO ITEM c]
```

### Subitens Discursivos
- **a)** Explique estatisticamente por que, na **ANODEV Tipo I (Saída 1 vs. Saída 2)**, o termo `Velocidade` reduz o desvio em $\mathbf{100,426}$ ($p < 2,2\times 10^{-16}$) quando entra por primeiro, mas reduz apenas $\mathbf{0,651}$ ($p = 0,4196$) quando entra após `Fluxo`.
- **b)** Explique o que exatamente o comando `drop1(..., test="Chisq")` (Análise **Tipo II / Marginal**) está testando em cada linha e demonstre por que o teste de `Iluminacao` (`LRT = 32,986`) é numericamente **idêntico** na última linha da Saída 1, na última linha da Saída 2 e na Saída 3, enquanto o LRT de `Velocidade` no `drop1` ($0,167$) difere do valor $0,651$ da Saída 1.
- **c)** Demonstre passo a passo como reconstruir exatamente o **AIC do modelo reduzido** sem `Velocidade` a partir do AIC do modelo completo ($338,92$) utilizando a relação $\Delta\text{AIC} = \Delta D^* - 2(\Delta p)$, explicando a lógica de recompensa/penalidade do critério de Akaike.
- **d)** Combinando os testes de razão de verossimilhanças ($\alpha = 5\%$), o critério AIC e a adequação residual do modelo sem `Velocidade` (considere $\chi^2_{77; 0,95} = 98,48$), qual modelo final você seleciona? Há evidência de superdispersão?

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Observe que $r_{\text{Fluxo, Velocidade}} = 0,951$ (colinearidade extrema). Na tabela **Tipo I (sequencial)**, cada variável é testada adicionando-a apenas às que já entraram antes dela! Quem entra primeiro “rouba” toda a variância explicada compartilhada.

                
**Item b):** No `drop1()` (Tipo II), cada linha compara o **modelo completo** contra o modelo com **apenas aquele termo removido**, ajustando por *todas* as demais covariáveis simultaneamente! Veja o que acontece com o último termo de uma tabela Tipo I: todas as outras covariáveis já entraram antes dele!

                
**Item c):** Lembre da definição: $\text{AIC} = -2\ell(\hat{\boldsymbol{\beta}}) + 2p$. Quando removemos $1$ parâmetro ($\Delta p = -1$), a log-verossimilhança cai pela metade da deviance extra ($\Delta D^* = 66.779 - 66.612 = 0,167$), mas o termo de penalidade $2p$ cai em $2$ unidades!

                
**Item d):** Verifique simultaneamente três coisas no modelo `Acidentes ~ Fluxo + Iluminacao`: (1) LRT de remoção de `Velocidade` ($p = 0,6831$); (2) comparação de AIC ($337,09 < 338,92$); e (3) razão $D/gl$ e $X^2/gl$ no modelo reduzido ($\approx 0,87 < 1$ e $< \chi^2_{77;0,95}$).

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Regra de Ouro Tipo I vs. Tipo II:** A soma dos desvios explicados da Tipo I soma exatamente $D_{\text{Nulo}} - D_{\text{Completo}}$, mas depende da ordem de digitação na fórmula sempre que as covariáveis não forem ortogonais. Já a Tipo II (`drop1` ou `car::Anova(mod, type="II")`) é **invariante à ordem**, testando a contribuição condicional marginal de cada variável na presença das demais.

                
**Pegadinha no AIC:** Para que remover uma variável reduza o AIC (melhore o equilíbrio ajuste-parcimônia), o aumento na deviance escalonada $\Delta D^*$ precisa ser **menor que $2 \times \Delta gl$** (ou seja, $\Delta D^*  1,5$ (ou muito além dos limites da $\chi^2_{gl}$), há superdispersão; se $D/gl \approx 0,87$ e $D < \chi^2_{77;0,95}$, o modelo Poisson com $\phi = 1$ é plenamente adequado!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Dependência da Ordem na ANODEV Tipo I sob Colinearidade:**

                  
A tabela `anova()` do R realiza decomposição **sequencial (Tipo I)**, comparando modelos encaixados sucessivos na ordem exata em que foram declarados na fórmula: $\emptyset \subset M(X_1) \subset M(X_1, X_2) \subset M(X_1, X_2, X_3)$. Como `Fluxo` e `Velocidade` possuem correlação $r = 0,9507$, carregam praticamente a mesma informação sobre o risco viário. Na Saída 2, `Velocidade` entra primeiro e absorve toda a variância compartilhada ($\Delta D = 100,426$). Na Saída 1, `Fluxo` entra antes e captura essa variância ($\Delta D = 116,760$), restando para `Velocidade` apenas sua contribuição adicional líquida acima de `Fluxo` ($\Delta D = 0,651$; $p = 0,4196$).

                

                

                  
**b) Funcionamento do Tipo II (`drop1`) e Coincidência da Última Linha:**

                  
O comando `drop1(mod, test="Chisq")` avalia o efeito **marginal (Tipo II)**, comparando o modelo completo (com as 3 covariáveis) contra cada submodelo com apenas 1 termo suprimido ($\Delta D = D_{M \setminus j} - D_M$). Por isso:

                  

                    
**Coincidência de `Iluminacao` ($32,986$):** Como `Iluminacao` foi digitada por último tanto na Saída 1 quanto na Saída 2, a etapa final de ambas testa `Fluxo + Velocidade + Iluminacao` vs. `Fluxo + Velocidade` — exatamente o mesmo contraste avaliado por `drop1()`!

                    
**Diferença em `Velocidade` ($0,167$ vs. $0,651$):** Na Saída 1 (Tipo I), o valor $0,651$ compara `Fluxo + Velocidade` vs. `Fluxo` (sem `Iluminacao` no modelo). Já no `drop1()`, $0,167$ compara o modelo completo `Fluxo + Velocidade + Iluminacao` vs. `Fluxo + Iluminacao`.

                  

                

                

                  
**c) Reconstrução Manual do AIC:**

                  
Como $\text{AIC} = -2\ell + 2p$ e na família Poisson $\phi = 1$ (logo $\Delta(-2\ell) = \Delta D$), ao remover `Velocidade` ($\Delta p = -1$) a deviance residual sobe em $\text{LRT} = 66.779 - 66.612 = \mathbf{0,167}$, enquanto o termo de penalidade por complexidade cai em $2 \times 1 = 2$:

                  
$$\text{AIC}_{\text{sem Velocidade}} = \text{AIC}_{\text{completo}} + \Delta D - 2(\Delta k) = 338,92 + 0,167 - 2(1) = \mathbf{337,09}$$

                  
Como o aumento insignificante na deviance ($0,167$) não compensa o custo de $2$ unidades de penalidade por um parâmetro adicional, o AIC diminui em $1,83$ unidades, favorecendo o modelo sem `Velocidade`.

                

                

                  
**d) Seleção Final e Avaliação de Superdispersão:**

                  

                    **📝 Como redigir na prova:** “Devido à colinearidade ($r = 0,951$) entre Fluxo e Velocidade, a análise Tipo II (`drop1`) revela que, uma vez conhecido o fluxo de veículos e a iluminação, a Velocidade não agrega informação estatisticamente significativa ($\text{LRT} = 0,167$; $p = 0,6831$), e sua exclusão reduz o AIC de $338,92$ para $337,09$. Assim, seleciona-se o modelo mais parcimonioso `Acidentes ~ Fluxo + Iluminacao` ($77$ gl). Ademais, sua deviance residual ($D = 66,779 < \chi^2_{77;0,95} = 98,48$) e seu estimador de dispersão de Pearson ($\hat{\phi}_P = 61,105/77 = 0,794 \approx 1$) não indicam superdispersão nem falha de ajuste na distribuição de Poisson.”

</details>

---

## Questão 4 — Modelo Normal Linear visto como Caso Particular de MLG: Deviance = SQRes, Distribuição F Exata vs. Assintótica e Relação t² = F
**Bloco:** Bloco I • ANODEV & φ̂ | **Inspiração:** Inspirada na Lista 3 (Q1–Q2) • Profa. Rosemeire

Para estudar o volume expiratório forçado (`FEV1`, em litros) em $n = 94$ trabalhadores industriais, ajustou-se via comando `glm(..., family = gaussian(link = "identity"))` o modelo com `Tempo_exp` (anos), `Setor` (Administrativo [ref.], Operacional, Fundição) e `Fumante` (Não [ref.], Sim).

```r

Call:
glm(formula = FEV1 ~ Tempo_exp + Setor + Fumante, family = gaussian(link = "identity"), 
    data = df4)

Coefficients:
                  Estimate Std. Error t value Pr(>|t|)    
(Intercept)       4.007237   0.093654  42.788  < 2e-16 ***
Tempo_exp        -0.032617   0.005021  -6.496 4.59e-09 ***
SetorOperacional -0.227754   0.080519  -2.829 0.005776 ** 
SetorFundicao    -0.417272   0.102674  -4.064 0.000104 ***
FumanteSim       -0.349119   0.075290  -4.637 1.21e-05 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for gaussian family taken to be 0.1186319)

    Null deviance: 19.969  on 93  degrees of freedom
Residual deviance: 10.558  on 89  degrees of freedom
AIC: 73.24

Number of Fisher Scoring iterations: 2


=== anova(mod_glm, test = "F") ===
Analysis of Deviance Table

Model: gaussian, link: identity

Response: FEV1

Terms added sequentially (first to last)


          Df Deviance Resid. Df Resid. Dev      F    Pr(>F)    
NULL                         93     19.969                     
Tempo_exp  1   5.1050        92     14.864 43.032 3.449e-09 ***
Setor      2   1.7553        90     13.109  7.398  0.001066 ** 
Fumante    1   2.5508        89     10.558 21.501 1.207e-05 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

--- Informacoes auxiliares ---
Soma de Quadrados dos Residuos de Pearson (X^2): 10.5582
```

### Subitens Discursivos
- **a)** Escreva a fórmula da **deviance não-escalonada** para a família Normal com ligação identidade e prove por que, nesse caso particular, ela coincide exatamente com a Soma de Quadrados dos Resíduos ($\text{SQRes}$) da regressão linear clássica. Qual é a função de variância $V(\mu)$?
- **b)** Calcule a estimativa do parâmetro de dispersão $\hat{\phi}$ no modelo completo pelo método do desvio e pelo método de Pearson, explicando por que ambos são numericamente idênticos no MLG Normal (o que *não* ocorre em famílias como Gama ou Poisson).
- **c)** Execute manualmente o teste $F$ parcial para o fator `Setor` ($2$ gl) a partir da coluna `Resid. Dev` e do valor de $\hat{\phi}$. Explique teoricamente por que, na família Normal linear, a estatística $F$ possui distribuição $F_{r, n-p}$ **exata** sob $H_0$, enquanto nos demais MLGs (como Gama ou Binomial Negativa) o teste $F$ da ANODEV é apenas uma aproximação assintótica.
- **d)** Verifique numericamente e justifique teoricamente a relação entre a estatística de Wald $t$ apresentada em `summary()` para `FumanteSim` e a estatística $F$ da última linha da tabela ANODEV. Por que essa igualdade exata se perde ao migrar da Normal para modelos como Logística ou Gama?

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Lembre que na Normal $\mathcal{N}(\mu_i, \sigma^2)$ o desvio unitário é $(y_i - \hat{\mu}_i)^2$ e $\phi = \sigma^2$ constante ($V(\mu_i) = 1$). Some de $i=1$ até $n$ e compare com $\sum (y_i - \hat{y}_i)^2$ da regressão linear múltipla.

                
**Item b):** Pegue `Residual deviance` ($10,558$) e $X^2$ ($10,5580$) e divida pelos graus de liberdade residuais ($gl = 94 - 5 = 89$). Perceba por que o numerador do resíduo de Pearson $(y_i - \hat{\mu}_i)^2 / V(\hat{\mu}_i)$ vira $(y_i - \hat{\mu}_i)^2 / 1$!

                
**Item c):** Na linha de `Setor`, identifique a redução de desvio $\Delta D_{\text{Setor}} = 14,864 - 13,109 = 1,755$ ($2$ gl). Calcule $F = (1,7553 / 2) / \hat{\phi}$. Depois pense no Teorema de Cochran: em amostras normais homocedásticas com ligação identidade, $\text{SQRes}/\sigma^2 \sim \chi^2_{n-p}$ *exatamente* e é independente das médias ajustadas!

                
**Item d):** Olhe para o `t value` de `FumanteSim` ($t = -4.637$) e eleve-o ao quadrado! Compare $t^2$ com $F = 21.501$ da tabela `anova()`.

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Escala de Deviance na Normal:** Embora a ANODEV chame a coluna de `Deviance` e `Resid. Dev`, na regressão Normal ela corresponde exatamente às Somas de Quadrados (`Sum Sq`) de uma tabela ANOVA clássica ($\text{SQReg}$ sequencial e $\text{SQRes}$)!

                
**Por que $F$ é aproximada em Gama/Logística/Quasi-Poisson?** Fora da família Normal com ligação identidade: (i) os desvios unitários não são quadrados exatos de variáveis gaussianas, logo $D/\phi \sim \chi^2$ vale apenas assintoticamente; (ii) a estimativa de $\hat{\phi}$ e a redução $\Delta D$ não são estritamente independentes em amostra finita; e (iii) os pesos iterativos $\mathbf{W}(\hat{\boldsymbol{\mu}})$ dependem dos dados.

                
**Wald vs. TRV (ANODEV):** Na Normal linear, a log-verossimilhança é estritamente quadrática em $\boldsymbol{\beta}$! Como uma parábola tem curvatura constante (Matriz de Informação de Fisher não depende de $\boldsymbol{\beta}$), a estatística de Wald quadrática $t^2 = (\hat{\beta}_j / \text{EP})^2$ coincide **exatamente** com a estatística $F$ de diferença de deviances ($1$ gl). Nos demais MLGs, a verossimilhança não é quadrática, gerando pequenas discrepâncias entre $z^2$ (Wald) e $\Delta D$ (TRV).

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Equivalência $D(\mathbf{y};\hat{\boldsymbol{\mu}}) = \text{SQRes}$ e Função de Variância:**

                  
Na família Normal $Y_i \sim \mathcal{N}(\mu_i, \phi)$, temos parâmetro de dispersão $\phi = \sigma^2$ constante e função de variância unitária **$V(\mu_i) = 1$** (homocedasticidade absoluta). Substituindo na definição geral da deviance não-escalonada via $2\sum [y_i(\tilde{\theta}_i - \hat{\theta}_i) - (b(\tilde{\theta}_i) - b(\hat{\theta}_i))]$ com $\theta_i = \mu_i$ e $b(\theta) = \theta^2/2$, obtém-se diretamente:

                  
$$D(\mathbf{y}; \hat{\boldsymbol{\mu}}) = \sum_{i=1}^n (y_i - \hat{\mu}_i)^2 = \text{SQRes}$$

                

                

                  
**b) Igualdade $\hat{\phi}_D = \hat{\phi}_P = s^2$:**

                  
No modelo completo com $p = 5$ parâmetros ($gl = 94 - 5 = 89$), temos $D_M = X^2_M = 10.5582$. Logo:

                  
$\hat{\phi}_D = \dfrac{D_M}{n - p} = \dfrac{10.5582}{89} = \mathbf{0.11863} \quad \text{e} \quad \hat{\phi}_P = \dfrac{X^2_M}{n - p} = \dfrac{\sum (y_i - \hat{\mu}_i)^2 / V(\hat{\mu}_i)}{89} = \dfrac{10.5582}{89} = \mathbf{0.11863}$.

                  
São idênticos porque, exclusivamente na família Normal, $d_i = (y_i - \hat{\mu}_i)^2$ e $V(\hat{\mu}_i) = 1$, de modo que o termo de Pearson $(y_i - \hat{\mu}_i)^2/V(\hat{\mu}_i)$ e o desvio individual $d_i$ são exatamente a mesma quantidade algébrica.

                

                

                  
**c) Teste $F$ para `Setor` e Exatidão vs. Assintoticidade:**

                  
Para $H_0: \beta_{\text{Oper}} = \beta_{\text{Fund}} = 0$ ($r = 2$ gl), a redução de desvio é $\Delta D = 14,864 - 13,109 = 1.7553$:

                  
$F = \dfrac{\Delta D / 2}{\hat{\phi}} = \dfrac{1.7553 / 2}{0.11863} = \dfrac{0.87764}{0.11863} = \mathbf{7.398}$ ($p = 0,001066 < 0,01$, logo o setor de trabalho altera significativamente o FEV1).

                  
Na família Normal com ligação identidade, a resposta tem distribuição gaussiana exata, a estimativa de MQO é linear em $\mathbf{y}$ e, pelo Teorema de Cochran, $\Delta D / \sigma^2 \sim \chi^2_r$ e $D_M / \sigma^2 \sim \chi^2_{n-p}$ são variáveis **exatamente qui-quadrado e independentes** para qualquer $n$, tornando a razão $F_{r, n-p}$ **exata**. Já nos demais MLGs (Gama, Poisson, Quasi-Binomial), $D^*/\phi$ é apenas assintoticamente $\chi^2$, e numerador/denominador dependem de pesos iterativos $\hat{\mathbf{W}}$ aleatórios, tornando o teste $F$ aproximado (valendo apenas para $n$ grande).

                

                

                  
**d) Relação Exata $t^2 = F$ no Caso Normal Linear:**

                  
Em `summary()`, o coeficiente de `FumanteSim` tem $t = -4.6370$. Elevando ao quadrado: $t^2 = (-4.6370)^2 = \mathbf{21.501} = F_{\text{Fumante}}$ da tabela ANODEV! Como `Fumante` é a última variável a entrar na tabela Tipo I ($1$ gl), o teste $t$ parcial (condicional às demais) testa exatamente o mesmo modelo reduzido vs. completo. Na Normal linear a função de log-verossimilhança é estritamente quadrática em $\boldsymbol{\beta}$, fazendo com que a aproximação quadrática de Wald coincida perfeitamente com o Teste da Razão de Verossimilhanças ($\Delta D$). Nos MLGs não-Normais a log-verossimilhança é não-quadrática, de modo que $z^2$ (Wald) apenas aproxima $\Delta D$ (TRV) assintoticamente.

</details>

---

## Questão 5 — Diagnóstico nos 4 Gráficos (plot(mod)), Estabilização de V(μ), Dupla Convenção de φ (Paula vs. R) e Resíduo r_Di
**Bloco:** Bloco II • Diagnósticos & Envelopes | **Inspiração:** Inspirada na Lista 3 (Q4) e Atividade 4 • Profa. Rosemeire

Deseja-se modelar o tempo de recuperação pós-cirúrgica ($Y_i > 0$, em dias) de $n = 60$ pacientes em função da `Idade` e do tipo de `Procedimento` (Eletiva vs. Urgência). Inicialmente ajustou-se um MLG Normal com ligação identidade (Modelo A) e, em seguida, um MLG Gama com ligação logarítmica (Modelo B). Analise os gráficos diagnósticos e as métricas obtidas para a observação destacada `#19`.

```r
=== Comparacao de Modelos e Metricas da Observacao #19 (Modelo B - Gama Log) ===
Resumo Modelo B (glm Gamma(link="log"), n = 60, p = 3 parametros, gl = 57):
  Parametro de dispersao no R (summary(mod_gama)$dispersion):  phi_hat_R = 0.30210
  Saida equivalente de forma na literatura (Paula / glm.diag): phi_Paula = 3.3102

Dados individuais da Observacao #19 (Idade = 77, Procedimento = Urgencia):
  Tempo observado (y_19):                 3.90 dias
  Media ajustada (mu_hat_19):             13.9411 dias
  Alavancagem (h_19,19):                  0.091185
  Componente do desvio bruto (d_19):      -1.052250  (sinal negativo, pois y_19 < mu_hat_19)
  Residuo do desvio padronizado (r_D19):  [CALCULAR NO ITEM d]
```

### Subitens Discursivos
- **a)** Com base nos gráficos comparativos apresentados (e no comportamento clássico dos painéis 1 e 3 de `plot(mod)`), diagnostique o problema estrutural do Modelo A (Normal com $V(\mu)=1$) e explique matematicamente por que a migração para o Modelo B (Gama com $V(\mu)=\mu^2$) corrige a heterocedasticidade.
- **b)** Descreva sistematicamente o que deve ser inspecionado em **cada um dos 4 painéis diagnósticos gerados por `plot(mod)`** em um MLG bem especificado (1: *Residuals vs Fitted*, 2: *Normal Q-Q*, 3: *Scale-Location*, 4: *Residuals vs Leverage*).
- **c)** Um estudante ficou confuso ao ler `Dispersion parameter = 0,3021` no console do R e $\hat{\phi}_{\text{Paula}} = 3,3102$ no livro de Gilberto Paula / pacotes baseados na parametrização de forma. Reconcilie algebricamente os dois valores e calcule o **Coeficiente de Variação (CV)** intrínseco estimado para o tempo de recuperação.
- **d)** Utilizando os dados da observação `#19`, calcule passo a passo o seu **resíduo componente do desvio padronizado** $r_{D,19}$, verificando se o paciente `#19` deve ser sinalizado como um ponto discrepante ($|r_{Di}| > 2$).

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Olhe para a largura vertical dos pontos à medida que $\hat{\mu}_i$ cresce no gráfico da esquerda (Modelo Normal): em dados estritamente positivos de tempo ou custo, quanto maior a média, maior o desvio-padrão! Qual a relação entre $\text{Var}(Y_i) = \phi V(\mu_i)$ no Modelo A vs. no Modelo B?

                
**Item b):** Lembre dos 4 quadrantes padrões do R ao executar `par(mfrow=c(2,2)); plot(modelo)`: o que está no eixo $X$ e no eixo $Y$ de cada painel? Em MLG, atenção ao eixo $X$ do painel 1: o R plota o **preditor linear ajustado $\hat{\eta}_i$** (e não $\hat{\mu}_i$)!

                
**Item c):** Multiplique $\hat{\phi}_R \times \hat{\phi}_{\text{Paula}}$! Veja que $0.3021 \times 3.3102 = 1,0000$. Um é exatamente o inverso multiplicativo do outro ($
u = 1/\phi_R$)!

                
**Item d):** Aplique diretamente a fórmula do resíduo componente do desvio padronizado para MLG com dispersão $\hat{\phi}_R$: $r_{Di} = \frac{d_i}{\sqrt{\hat{\phi}_R (1 - h_{ii})}}$.

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Pegadinha da dupla notação $\phi$ (Prova clássica!):** No livro de Gilberto Paula (e nas notas de aula teóricas da disciplina), escreve-se a densidade Gama parametrizada pelo *parâmetro de forma* $\phi_{\text{Paula}} = \nu$, onde $\text{Var}(Y) = \mu^2 / \phi_{\text{Paula}}$ e $\text{CV} = 1/\sqrt{\phi_{\text{Paula}}}$. Já no R (`glm(..., family=Gamma)`), o software exibe o *parâmetro de escala/dispersão* da família exponencial $\phi_{\text{R}} = 1/\nu$, onde $\text{Var}(Y) = \phi_{\text{R}} \mu^2$ e $\text{CV} = \sqrt{\phi_{\text{R}}}$!

                
**Cuidado na fórmula de $r_{Di}$:** Jamais use $\hat{\phi}_{\text{Paula}}$ no denominador de $r_{Di} = d_i / \sqrt{\hat{\phi}(1-h_{ii})}$! Se $\hat{\phi}_{\text{R}} = 0,3021$ aparece no denominador multiplicando $(1 - h_{ii})$, usar $3,31$ por engano diminuiria $|r_{Di}|$ em $3,31$ vezes e esconderia um outlier!

                
**Eixo X do plot 1 em MLGs:** Nos MLGs não-Normais, `plot(mod)` usa *Predicted values* $\hat{\eta}_i$ (na escala do preditor linear) no eixo horizontal, pois se plotasse $\hat{\mu}_i$ os pontos se aglomerariam nas extremidades.

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Diagnóstico de Heterocedasticidade e Função de Variância:**

                  
No Modelo A (Normal, $V(\mu) = 1$), supõe-se variância constante $\text{Var}(Y_i) = \sigma^2$ independentemente da média. Contudo, em tempos de recuperação hospitalar ($Y_i > 0$), pacientes graves com médias maiores apresentam desvios-padrão proporcionalmente maiores (efeito leque/funil nos resíduos brutos ou de Pearson + forte assimetria positiva). Ao migrar para a família **Gama** ($V(\mu) = \mu^2$), assume-se que $\text{DP}(Y_i) = \sqrt{\phi_R}\cdot\mu_i$ (coeficiente de variação constante), de modo que os resíduos padronizados divididos por $\hat{\mu}_i$ estabilizam dentro da faixa $(-2, +2)$ de maneira simétrica e homocedástica ao longo de $\hat{\eta}_i$.

                

                

                  
**b) Leitura dos 4 Painéis Diagnósticos de `plot(mod)`:**

                  

                    
**Painel 1 (Residuals vs Fitted — $\hat{\eta}_i$ vs. resíduos do desvio):** Avalia a adequação do preditor linear e da função de ligação. Espera-se uma nuvem aleatória horizontal centrada em $0$, sem tendências em U ou curvas (que denunciariam ligação incorreta ou termos quadráticos faltantes).

                    
**Painel 2 (Normal Q-Q dos resíduos padronizados do desvio):** Verifica a proximidade dos resíduos $r_{Di}$ em relação à distribuição $\mathcal{N}(0,1)$. Em MLGs não-Normais pequenos desvios nas caudas são aceitáveis, devendo-se confirmar com envelopes simulados (`hnp`).

                    
**Painel 3 (Scale-Location — $\sqrt{|r_{Di}|}$ vs. $\hat{\eta}_i$):** Testa diretamente se a função de variância $V(\mu)$ foi bem escolhida. Se a linha suavizada (loess) for aproximadamente horizontal, a variância foi bem modelada; inclinações crescentes denunciam variância subestimada (funil).

                    
**Painel 4 (Residuals vs Leverage — $r_{Pi}$ vs. $h_{ii}$ com contornos de Cook):** Identifica simultaneamente pontos de alta alavanca ($h_{ii} > 2p/n$), outliers ($|r_i| > 2$) e observações globalmente influentes (que ultrapassam as curvas tracejadas da Distância de Cook $D_i = 0,5$ ou $1,0$).

                  

                

                

                  
**c) Reconciliação $\hat{\phi}_{\text{R}}$ vs. $\hat{\phi}_{\text{Paula}}$ e Coeficiente de Variação:**

                  
Na forma canônica da família exponencial adotada pelo R, $\text{Var}(Y_i) = \phi_R V(\mu_i) = \phi_R \mu_i^2$. Na parametrização de forma de Paula ($\nu$), $\text{Var}(Y_i) = \mu_i^2 / \phi_{\text{Paula}}$. Logo:

                  
$\hat{\phi}_{\text{Paula}} = \dfrac{1}{\hat{\phi}_R} = \dfrac{1}{0.30210} = \mathbf{3.3102}$. Não há contradição: são recíprocos exatos.

                  
O Coeficiente de Variação estimado do modelo Gama é $\widehat{\text{CV}} = \dfrac{\text{DP}(Y_i)}{\mu_i} = \sqrt{\hat{\phi}_R} = \dfrac{1}{\sqrt{\hat{\phi}_{\text{Paula}}}} = \sqrt{0.30210} = \mathbf{54.96\%}$.

                

                

                  
**d) Cálculo de $r_{D,19}$:**

                  
Usando obrigatoriamente a dispersão na convenção do R ($\hat{\phi}_R = 0.30210$):

                  
$$r_{D,19} = \frac{d_{19}}{\sqrt{\hat{\phi}_R (1 - h_{19,19})}} = \frac{-1.052250}{\sqrt{0.30210 \times (1 - 0.091185)}} = \frac{-1.052250}{\sqrt{0.27455}} = \frac{-1.052250}{0.52398} = \mathbf{-2.008}$$

                  

                    **📝 Como redigir na prova:** “O resíduo componente do desvio padronizado para a observação `#19` é $r_{D,19} = -2,008$, situando-se ligeiramente além da fronteira convencional $[-2, +2]$ ($|r_{D,19}| > 2$). O sinal negativo reflete que esse paciente grave e idoso teve alta muito mais rápida do que o esperado pelo modelo (3,90 dias observados contra 13,94 dias previstos). Deve-se verificar sua alavanca e realizar análise de sensibilidade antes de qualquer exclusão.”

</details>

---

## Questão 6 — Alavancagem Generalizada (H), Williams (r_Gi), Distância de Cook e Tríade de Robustez sob Remoção de Observação Extrema
**Bloco:** Bloco II • Diagnósticos & Envelopes | **Inspiração:** Inspirada na Lista 5 (crab.rtf) e Atividade 7 • Profa. Rosemeire

Em um estudo ecológico inspirado nos dados de caranguejos-ferradura (*Limulus*), modelou-se a probabilidade de ocorrência de satélites ($Y_i \in \{0, 1\}$) em $n = 140$ fêmeas por regressão logística em função da `Largura` da carapaça (cm), `Peso` (kg) e `Cor` (Clara [ref.], Escura). A observação `#42` (fêmea muito larga de $31,2$ cm, mas excepcionalmente leve com $1,15$ kg e $Y_{42} = 0$) destacou-se nos diagnósticos. Compare as medidas de diagnóstico e o impacto de sua retirada.

```r
=== Tabela de Diagnostico Selecionada (Modelo Completo, n = 140, p = 4) ===
 Obs y_i Largura Peso pi_hat   h_ii   r_Pi   r_Di   r_Gi Cook_Di
 #14   1    25.7 1.65 0.6048 0.0178  0.816  1.012  1.009  0.0030
 #16   0    28.9 2.61 0.8882 0.0224 -2.851 -2.117 -2.137  0.0466
 #42   0    31.2 1.15 0.4581 0.4005 -1.187 -1.430 -1.338  0.2354
 #88   0    24.7 1.07 0.3693 0.0259 -0.775 -0.973 -0.968  0.0040

=== COMPARATIVO: summary(mod_completo) vs. summary(mod_sem_42) ===
[A] Modelo Completo (n = 140):

Call:
glm(formula = Presenca ~ Largura + Peso + Cor, family = binomial(link = "logit"), 
    data = df6)

Coefficients:
            Estimate Std. Error z value Pr(>|z|)  
(Intercept) -3.15089    3.96852  -0.794    0.427  
Largura      0.03688    0.18661   0.198    0.843  
Peso         1.59310    0.70597   2.257    0.024 *
CorEscura   -0.20345    0.37586  -0.541    0.588  
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 191.21  on 139  degrees of freedom
Residual deviance: 165.69  on 136  degrees of freedom
AIC: 173.69

Number of Fisher Scoring iterations: 3


[B] Modelo sem a Observacao #42 (n = 139):

Call:
glm(formula = Presenca ~ Largura + Peso + Cor, family = binomial(link = "logit"), 
    data = df6[-42, ])

Coefficients:
            Estimate Std. Error z value Pr(>|z|)
(Intercept)  -6.7230     5.0203  -1.339    0.181
Largura       0.2131     0.2396   0.890    0.374
Peso          0.9918     0.8601   1.153    0.249
CorEscura    -0.2456     0.3779  -0.650    0.516

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 189.51  on 138  degrees of freedom
Residual deviance: 163.90  on 135  degrees of freedom
AIC: 171.9

Number of Fisher Scoring iterations: 3

```

### Subitens Discursivos
- **a)** Defina a matriz de projeção generalizada $\mathbf{H}$ nos MLGs, explicando por que, diferentemente da regressão linear clássica ($\mathbf{H} = \mathbf{X}(\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top$), a alavanca $h_{ii}$ depende também da variável peso iterativa $w_i$ ligada à média estimada $\hat{\mu}_i$.
- **b)** Calcule o ponto de corte clássico de alavancagem $2p/n$ e identifique na tabela qual observação é ponto de alavanca e qual é outlier puro de resposta. Em seguida, demonstre numericamente para a observação `#42` a relação algébrica exata entre a Distância de Cook $D_{42}$, $r_{P,42}$ e $h_{42,42}$.
- **c)** Compare o resíduo componente do desvio padronizado $r_{Di}$ com o resíduo studentizado / aproximado de Williams $r_{Gi}$ (especialmente para `#42` e `#16`). Escreva a definição matemática de $r_{Gi}$ e explique quando a diferença entre eles se torna mais acentuada.
- **d)** Avalie o impacto da remoção da observação `#42` utilizando a **Tríade de Robustez** (sinais das estimativas $\hat{\beta}$, significância estatística dos testes de Wald ao nível de $5\%$ e conclusão substantiva sobre `Peso` vs. `Largura`) e descreva a conduta estatística recomendada pela professora.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Escreva $\mathbf{H} = \mathbf{W}^{1/2}\mathbf{X}(\mathbf{X}^\top \mathbf{W} \mathbf{X})^{-1} \mathbf{X}^\top \mathbf{W}^{1/2}$ onde $w_i = \left(\frac{\partial \mu_i}{\partial \eta_i}\right)^2 \frac{1}{V(\mu_i)}$. Mostre que $h_{ii}$ é proporcional a $w_i \cdot \mathbf{x}_i^\top (\mathbf{X}^\top \mathbf{W} \mathbf{X})^{-1} \mathbf{x}_i$.

                
**Item b):** Calcule $2p/n = 2 \times 4 / 140$. Veja na tabela quem tem $h_{ii} > 2p/n$ vs. quem tem $|r_{Di}| > 2$! Para a Distância de Cook (com $\phi=1$ na Logística), use $D_i = \frac{r_{Pi}^2}{p} \cdot \frac{h_{ii}}{1 - h_{ii}}$.

                
**Item c):** Lembre que o resíduo de Williams $r_{Gi}$ combina o componente do desvio e o resíduo de Pearson em uma aproximação de primeira ordem da redução de deviance ao deletar o caso $i$: $r_{Gi} = \text{sinal}(y_i - \hat{\mu}_i) \sqrt{(1-h_{ii}) r_{Di}^2 + h_{ii} r_{Pi}^2}$ (ou na forma escalonada $r_{Gi}^2 \approx d_i^2/(1-h_{ii}) + \dots$).

                
**Item d):** Compare lado a lado os p-valores de `Peso` e a magnitude da estimativa de `Largura` no Bloco [A] ($n=140$) vs. Bloco [B] ($n=139$)! Veja se houve mudança drástica de inferência!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Outlier vs. Alavanca vs. Influente:** A observação `#16` ($y=0$ quando $\hat{\pi} = 0.8882$) tem resíduo elevado ($|r_D| > 2$), mas $h_{ii} = 0.0224 < 0,0571$ — logo é apenas um *outlier na resposta* com baixo impacto global ($D = 0.0466$). Já `#42` possui alavancagem $7\times$ acima do corte ($h_{42,42} = 0.4005$), combinada com discordância moderada ($r_P = -1.187$), disparando a maior Distância de Cook do banco ($D_{42} = 0.2354$)!

                
**Tríade de Robustez na Prática:** Nunca diga automaticamente “excluir o ponto #42” nem o ignore! Se a exclusão de um único ponto derruba a significância de uma covariável inteira (`Peso` muda de $p = 0,024$ para $p = 0,249$ e `Largura` multiplica por $5,8\times$!), o modelo não é robusto em relação a ele — isso revela que `#42` estava quebrando a colinearidade entre `Largura` e `Peso` e sustentando sozinha o efeito de `Peso`!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Matriz de Projeção Generalizada $\mathbf{H}$:**

                  
Na iteração final do algoritmo Escore de Fisher (IRLS), a matriz chapéu generalizada é $\mathbf{H} = \mathbf{W}^{1/2}\mathbf{X}(\mathbf{X}^\top\mathbf{W}\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{W}^{1/2}$, onde $\mathbf{W} = \text{diag}(w_1, \dots, w_n)$. Para cada ponto, $h_{ii} = w_i \,\mathbf{x}_i^\top (\mathbf{X}^\top \mathbf{W} \mathbf{X})^{-1} \mathbf{x}_i$. Portanto, ao contrário do MQO linear (em que $\mathbf{W} = \mathbf{I}$ e $h_{ii}$ depende apenas do espaço das covariáveis), em um MLG a alavancagem $h_{ii}$ resulta do produto entre a excentricidade no espaço $\mathbf{X}$ e o peso local $w_i$ (na Logística, $w_i = \hat{\pi}_i(1 - \hat{\pi}_i)$).

                

                

                  
**b) Corte $2p/n$, Classificação dos Pontos e Fórmula de Cook:**

                  

                    
**Corte de Alavancagem:** $\dfrac{2p}{n} = \dfrac{2 \times 4}{140} = \mathbf{0.0571}$.

                    
**Ponto de Alavanca e Influente:** A fêmea `#42` possui $h_{42,42} = 0.4005 \gg 0,0571$ ($7\times$ o limite!), pois combina largura enorme ($31,2$ cm) com peso baixíssimo ($1,15$ kg) fora da linha de correlação da amostra, além de apresentar a maior Distância de Cook ($D_{42} = 0.2354$).

                    
**Outlier Puro de Resposta:** A fêmea `#16` possui $|r_D| = 2.117 > 2$ (tinha $\hat{\pi} = 0.8882$, mas apresentou $y = 0$), porém baixa alavanca ($h = 0.0224  0,05$), enquanto a deviance residual passa de $165,69$ ($136$ gl) para $163,90$ ($135$ gl).

                    
**3. Conclusão Substantiva:** A conclusão científica sobre o efeito independente do `Peso` *não é robusta* à presença de `#42` (que atuava como âncora solitária quebrando a colinearidade natural entre Largura e Peso).

                  

                  

                    **📝 Como redigir na prova:** “A observação `#42` é altamente influente ($h_{42,42} = 0,4005 > 0,0571$; $D_{42} = 0,2354$). Pela tríade de robustez, embora os sinais das estimativas se preservem, sua exclusão altera a inferência principal: a covariável `Peso` deixa de ser estatisticamente significativa ao nível de 5% ($p$ passa de $0,024$ para $0,249$). Recomenda-se auditar o registro biológico de `#42` (suspeita de erro de digitação/medição em `Peso = 1,15 kg` para `Largura = 31,2 cm`); caso seja erro comprovado, corrige-se ou exclui-se; se for dado válido legítimo, reportam-se os resultados com e sem `#42`, avaliando a colinearidade entre Largura e Peso em vez de descartar seletivamente.”

</details>

---

## Questão 7 — Envelope Simulado Half-Normal (hnp): Distinção entre Superdispersão Estrutural vs. Outliers e Correção Quasi-Binomial
**Bloco:** Bloco II • Diagnósticos & Envelopes | **Inspiração:** Inspirada na Lista 4 (Q4) e Desafio 3 • Profa. Rosemeire

Em um ensaio agronômico com $N = 36$ canteiros ($m_i = 50$ sementes semeadas por canteiro), avaliou-se a proporção de germinação em função da `Dose` de bioestimulante (L/ha) e da `Umidade` do solo (Baixa [ref.] vs. Ideal). Ao ajustar o modelo Binomial padrão ($\phi = 1$), o pesquisador obteve o Painel A do gráfico de envelope simulado *Half-Normal* (`hnp`) abaixo, levando-o a testar um modelo **Quasi-Binomial** (Painel B).

```r
=== Comparacao: Binomial Padrao vs. Quasi-Binomial (N = 36 parcelas, m_i = 50) ===
[1] family = binomial(link = "logit"):
Residual deviance: 140.44  on 33  degrees of freedom
Pearson X^2:       137.45

Coefficients (Binomial):
             Estimate Std. Error z value Pr(>|z|)
Dose         0.73995   0.10133   7.302  2.82e-13 ***

[2] family = quasibinomial(link = "logit"):
Dispersion parameter for quasibinomial family taken to be 4.1653
Coefficients (Quasi-Binomial):
             Estimate Std. Error t value Pr(>|t|)
Dose         0.73995   [CALCULAR] [CALCULAR]  [avaliar]
```

### Subitens Discursivos
- **a)** Com base na razão entre a deviance residual (e $X^2$) e seus respectivos graus de liberdade ($N - p = 33$), além do Painel A do envelope simulado, diagnostique o fenômeno estatístico presente nesse ensaio e calcule a estimativa do fator de inflação de variância (dispersão) $\hat{\phi}_P$.
- **b)** Explique o **algoritmo de construção do envelope simulado Half-Normal de Atkinson (`hnp`)** e justifique por que essa ferramenta gráfica é preferível a comparar os resíduos com a curva teórica da Normal padrão em MLGs com amostras finitas.
- **c)** Como distinguir visualmente em um gráfico de envelope simulado um **problema estrutural global do modelo** (como superdispersão, ausência de termo quadrático ou ligação errada) da presença apenas de **uma ou duas observações aberrantes isoladas**?
- **d)** Explique o risco inferencial de ignorar a superdispersão (usar $\phi=1$ quando $\hat{\phi}_P \approx 4,17$) sobre os erros-padrão, os intervalos de confiança e os p-valores. Em seguida, calcule manualmente o novo erro-padrão e a estatística $t$ do termo `Dose` no modelo **Quasi-Binomial**.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Sob especificação Binomial correta ($\phi = 1$), espera-se que $D / (N-p) \approx 1$ e $X^2 / (N-p) \approx 1$. Divida $140.44/33$ e $137.45/33$. Veja que ambas são mais de 4 vezes maiores que $1$!

                
**Item b):** Lembre dos 4 passos da simulação paramétrica de Atkinson: (1) extrair os módulos ordenados $|r_{Di}|_{(i)}$ observados; (2) simular $K$ (ex.: 199) novos vetores resposta usando o próprio modelo ajustado; (3) reajustar o modelo para cada réplica e ordenar seus $|r|$ simulados; (4) tomar os percentis 2,5%, 50% e 97,5% em cada estatística de ordem $i$.

                
**Item c):** Compare o Painel A (25 de 36 pontos saindo sistematicamente por toda a extensão/inclinação fora da banda superior!) contra o cenário hipotético onde os 34 primeiros pontos ficam perfeitamente colados à mediana e apenas os 2 últimos pontos lá em cima saltam para fora.

                
**Item d):** Na estimação por **Quasi-Verossimilhança (Quasi-Binomial)**, as estimativas pontuais $\hat{\beta}_j$ **não mudam**, mas a matriz de covariâncias é multiplicada por $\hat{\phi}_P$: $\text{EP}_{\text{QB}}(\hat{\beta}_j) = \sqrt{\hat{\phi}_P} \cdot \text{EP}_{\text{Bin}}(\hat{\beta}_j)$!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Conceito de Superdispersão:** Ocorre quando os dados exibem variância superior à prevista pela relação média-variância nominal ($\text{Var}(Y_i) > m_i \pi_i (1-\pi_i)$) — muito comum em dados agrupados agronômicos/biológicos devido a correlação intraclasse entre sementes do mesmo canteiro ou fatores não medidos (heterogeneidade entre parcelas).

                
**Impacto nas inferências:** Ignorar superdispersão ($\phi = 1$ indevido) torna os erros-padrão artificialmente **subestimados** pelo fator $1/\sqrt{\hat{\phi}}$, inflando a estatística $z$, gerando ICs estreitos demais e inflacionando drasticamente o **Erro Tipo I** (falsos positivos).

                
**Pegadinha da Quasi-Binomial no R:** No modelo Quasi-Binomial os p-valores usam a distribuição $t_{N-p}$ ($t_{33}$) em vez de Normal $z$, e o AIC fica indefinido (`NA`) pois não existe log-verossimilhança completa especificada!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Diagnóstico de Superdispersão e Cálculo de $\hat{\phi}_P$:**

                  
Sob a suposição binomial padrão ($\phi = 1$), espera-se que a deviance residual e a estatística $X^2$ se aproximem de $gl = N - p = 36 - 3 = 33$. Observamos:

                  
$\dfrac{D}{N-p} = \dfrac{140.44}{33} = \mathbf{4.256} \gg 1 \quad \text{e} \quad \hat{\phi}_P = \dfrac{X^2}{N-p} = \dfrac{137.45}{33} = \mathbf{4.1653}$.

                  
Somado ao Painel A onde **25 de 36 pontos (69,4%)** ultrapassam a banda superior simulada do modelo Binomial, diagnostica-se severa **superdispersão** (variância real cerca de $4,17\times$ maior que a variância nominal binomial).

                

                

                  
**b) Algoritmo do Envelope Simulado (Atkinson) e sua Superioridade:**

                  

                    
Ajusta-se o modelo proposto aos dados observados e calculam-se os valores absolutos ordenados dos resíduos diagnósticos $|r|_{(1)} \le \dots \le |r|_{(N)}$.

                    
Utilizando os valores ajustados $\hat{\boldsymbol{\mu}}$ e o próprio $\hat{\phi}$ do modelo ajustado, simulam-se $K$ (tipicamente $99$ ou $199$) amostras artificiais independentes sob a distribuição postulada.

                    
Reajusta-se exatamente o mesmo MLG para cada amostra simulada, obtendo $K$ vetores de resíduos absolutos ordenados.

                    
Para cada posição de ordem $i = 1, \dots, N$, extraem-se o mínimo/quantil $2,5\%$, a mediana e o máximo/quantil $97,5\%$, plotando-os contra os quantis meio-normais teóricos $\Phi^{-1}\!\left(\frac{i + N - 1/8}{2N + 1/2}\right)$.

                  

                  
Esse método é superior ao QQ-plot tradicional porque, em amostras pequenas ou em MLGs não-Normais, a distribuição exata dos resíduos $r_{Di}$ pode ser assimétrica e correlacionada mesmo quando o modelo está 100% correto; o envelope captura empiricamente exatamente essa geometria sob $H_0$.

                

                

                  
**c) Padrão Estrutural Global vs. Outliers Isolados no `hnp`:**

                  

                    
**Problema Estrutural Global (caso do Painel A):** A maioria ou uma sequência contínua de pontos foge sistematicamente da banda (inclinando-se para cima desde os quantis intermediários), formando um “degrau” ou saída em bloco. Sinaliza **superdispersão**, erro na função de ligação, omissão de variável ou má escolha de $V(\mu)$. (No Painel B, ao reescalonar por $\hat{\phi}_P$, os 36 pontos passam a caber integralmente dentro do envelope!).

                    
**Outliers Isolados:** Quase todos os pontos acompanham fielmente a linha mediana dentro das bandas, escapando apenas os $1$ ou $2$ maiores valores absolutos na extremidade superior direita.

                  

                

                

                  
**d) Consequências da Superdispersão e Recálculo em Quasi-Binomial:**

                  
Se ignorarmos a superdispersão, subestimamos a variância real dos parâmetros, produzindo ICs estreitos demais e p-valores enganosamente minúsculos (**Erro Tipo I inflacionado**). Na abordagem Quasi-Binomial, $\hat{\beta}_{\text{Dose}} = 0.73995$ não muda, mas o erro-padrão é inflado pelo fator $\sqrt{\hat{\phi}_P} = \sqrt{4.1653} = \mathbf{2.0409}$:

                  
$\text{EP}_{\text{QB}}(\hat{\beta}_{\text{Dose}}) = 0.10133 \times 2.0409 = \mathbf{0.20681}$

                  $t_{\text{QB}} = \dfrac{0.73995}{0.20681} = \dfrac{7.302}{2.0409} = \mathbf{3.578}$ ($gl = 33$, $p = 0,0011$).

                  

                    **📝 Como redigir na prova:** “O ajuste binomial padrão apresenta severa superdispersão ($D/gl = 4,26$; $\hat{\phi}_P = 4,165$), confirmada pela fuga generalizada de 25/36 pontos no envelope simulado. Ao corrigir os erros-padrão pela modelagem Quasi-Binomial (fator $\sqrt{\hat{\phi}_P} = 2,041$), o erro-padrão da Dose mais que dobra ($0,0891 \to 0,2068$) e a estatística cai de $z = 7,302$ para $t_{33} = 3,578$, preservando ainda a significância do bioestimulante de forma estatisticamente honesta (sem inflacionar o Erro Tipo I).”

</details>

---

## Questão 8 — Diagnósticos Específicos em Regressão Logística Binária: Paradoxo da Alavancagem, ΔX²_i, ΔD_i de Pregibon e Medida C_i
**Bloco:** Bloco II • Diagnósticos & Envelopes | **Inspiração:** Inspirada nos Slides de Diagnóstico em Regressão Logística & Lista 5 • Profa. Rosemeire

Ajustou-se uma regressão logística para complicação pós-operatória ($Y_i \in \{0,1\}$, $n = 180$, $p = 3$ parâmetros) em função do `Escore` cirúrgico e da `Idade`. A tabela abaixo exibe as medidas diagnósticas específicas da regressão logística (Hosmer & Lemeshow / Pregibon) para quatro pacientes de perfis contrastantes: note em especial o Paciente `#12` (com `Escore = +2,85` extremo e $y_{12}=1$), o Paciente `#77` ($y_{77}=0$, covariáveis médias), o Paciente `#153` (jovem de 24 anos com `Escore = +2,15` e $y_{153}=0$) e o Paciente controle `#45`.

```r
=== Medidas Diagnosticas Logisticas (Pregibon / Hosmer-Lemeshow, n = 180) ===
 Paciente y_i pi_hat    w_i   h_ii   r_Pi   r_Di Delta_X2 Delta_D    C_i
      #12   1 0.9894 0.0105 0.0080  0.104  0.147    0.011   0.022 0.0001
      #77   0 0.9316 0.0638 0.0212 -3.729 -2.341   13.908   5.776 0.3019
     #153   0 0.6104 0.2378 0.1415 -1.351 -1.482    1.825   2.454 0.3009
      #45   0 0.0539 0.0510 0.0089 -0.240 -0.334    0.058   0.112 0.0005

--- Valores brutos adicionais fornecidos pelo console ---
Paciente #77 : resíduo de Pearson bruto = -3.6895 | desvio individual bruto d_77 = -2.3160
Paciente #153: r_P153 = -1.351 | r_D153 = -1.482 | h_153,153 = 0.1415 | Delta_D_153 = [CALCULAR] | C_153 = [CALCULAR]
```

### Subitens Discursivos
- **a)** Explique o **paradoxo da alavancagem na regressão logística** evidenciado pelo Paciente `#12`: mesmo possuindo um perfil de covariáveis extremamente distante da média (`Escore = +2,85`, 72 anos), sua alavanca $h_{12,12} = 0.0080$ é minúscula! Demonstre matematicamente por que isso ocorre a partir de $w_i = \hat{\pi}_i(1 - \hat{\pi}_i)$ e indique em qual faixa de probabilidade $\hat{\pi}_i$ a alavanca logística atinge seu comportamento normal.
- **b)** Classifique criticamente os Pacientes `#12`, `#77` e `#153` quanto aos três papéis diagnósticos: (i) ponto de alavancagem elevada; (ii) má adequação / outlier na resposta ($\Delta X^2_i$ ou $\Delta D_i$ grande); e (iii) impacto global sobre o vetor $\hat{\boldsymbol{\beta}}$ ($C_i$).
- **c)** Calcule passo a passo: (i) a variação qui-quadrado de Pearson $\Delta X^2_{77}$ do Paciente `#77`; e (ii) as medidas aproximadas de Pregibon **$\Delta D_{153}$** e **$C_{153}$** do Paciente `#153` usando as definições $\Delta D_i = r_{Di}^2 + h_{ii}r_{Pi}^2$ e $C_i = \frac{r_{Pi}^2 \, h_{ii}}{1 - h_{ii}}$.
- **d)** Descreva como interpretar o famoso **gráfico diagnóstico de $\Delta D_i$ (ou $\Delta X^2_i$) versus $\hat{\pi}_i$** (com o tamanho das bolhas proporcional a $C_i$) proposto por Hosmer & Lemeshow para separar curvas de sucesso ($y_i=1$) e fracasso ($y_i=0$) mal ajustadas.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Olhe para a coluna `pi_hat` e `w_i` do Paciente `#12`: como $\hat{\pi}_{12} = 0.9894$, o peso iterativo vale $w_{12} = 0.9894 \times (1 - 0.9894) = 0.0105$ — praticamente ZERO! Lembre que na logística $h_{ii} = w_i \,\mathbf{x}_i^\top(\mathbf{X}^\top\mathbf{W}\mathbf{X})^{-1}\mathbf{x}_i$.

                
**Item b):** Compare cada paciente nos 3 critérios: corte de alavanca $2p/n = 6/180 = 0,0333$; corte típico de má adequação em logística $\Delta X^2_i > 4$ ou $\Delta D_i > 4$ (aprox. $\chi^2_{1;0,95} = 3,84$); e corte relativo de influência $C_i$.

                
**Item c):** Basta substituir diretamente os números fornecidos na tabela/bloco auxiliar: para `#77`, $\Delta X^2_{77} = r_{P,77}^2 = (-3.729)^2$; para `#153`, use $r_{D,153} = -1.482$, $r_{P,153} = -1.351$ e $h_{153,153} = 0.1415$.

                
**Item d):** Imagine o plano $(\hat{\pi}_i, \Delta D_i)$: quando $y_i = 1$ e $\hat{\pi}_i$ vai de $1$ para perto de $0$, a curva sobe exponencialmente para o topo esquerdo; quando $y_i = 0$ e $\hat{\pi}_i$ vai de $0$ para perto de $1$, a curva sobe exponencialmente para o topo direito!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Atenção ao Paradoxo de Pregibon:** Na regressão linear, quem tem $\mathbf{x}_i$ distante do centro SEMPRE tem $h_{ii}$ alto. Na regressão logística, $w_i = \hat{\pi}_i(1-\hat{\pi}_i)$ é máximo em $\hat{\pi}_i = 0,5$ ($w = 0,25$) e cai para zero quando $\hat{\pi}_i  0,9$! Por isso pontos extremamente distantes do centro na direção em que $\hat{\pi} \to 0$ ou $1$ ficam *comprimidos* na matriz $\mathbf{H}$ e têm $h_{ii} \to 0$!

                
**Por que `#153` tem $C_i$ alto mesmo com $r_D$ pequeno?** Olhe para os dois componentes de $C_i = \Delta X^2_i \cdot \frac{h_{ii}}{1 - h_{ii}}$: `#77` chega a $C_{77} = 0.3019$ por discrepância pura altíssima ($\Delta X^2 = 13.908$, $h$ baixo $= 0.0212$), enquanto `#153` chega quase ao mesmo impacto ($C_{153} = 0.3008$) por possuir alavancagem elevada ($h = 0.1415 > 0,0333$ pois $\hat{\pi} = 0.61$ está perto de $0,5$ e $w = 0,238$ é alto!) combinada com discordância moderada!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Paradoxo da Alavancagem Logística no Paciente `#12`:**

                  
Nos modelos logísticos, a alavancagem diagonal vale $h_{ii} = w_i \cdot \mathbf{x}_i^\top (\mathbf{X}^\top \mathbf{W} \mathbf{X})^{-1} \mathbf{x}_i$, em que $w_i = \hat{\pi}_i(1 - \hat{\pi}_i)$ é a variância de Bernoulli ajustada. Como a parábola $\pi(1-\pi)$ anula-se nos extremos e atinge máximo de $0,25$ em $\hat{\pi}=0,5$, no Paciente `#12` ($\hat{\pi}_{12} = 0.9894$) o peso iterativo despenca para $w_{12} = 0.0105$. Esse peso minúsculo neutraliza o grande distanciamento do vetor $\mathbf{x}_{12}$, derrubando $h_{12,12}$ para $0,0080$. Na logística, a alavancagem máxima costuma ocorrer para combinações atípicas de covariáveis situadas na faixa central de probabilidade ($0,1  4$). Isso gera alto impacto global ($C_{77} = 0.3019$).

                    
**Paciente `#153`:** **Ponto de alavancagem extrema** ($h_{153,153} = 0.1415 > 0,0333$ em mais de $4\times$), favorecido por estar próximo da zona de incerteza ($\hat{\pi} = 0.6104$, $w = 0,2378$ alto) com perfil demográfico atípico (24 anos com escore cirúrgico alto). Embora seus resíduos individuais sejam moderados ($|r_D| = 1,482 < 2$), sua alta alavanca produz forte influência ($C_{153} = 0.3008$), praticamente empatada com `#77`!

                  

                

                

                  
**c) Cálculos Manuais de Pregibon:**

                  

                    
**Para `#77`:** $\Delta X^2_{77} = r_{P,77}^2 = (-3.729)^2 = \mathbf{13.905}$ (representa a queda aproximada na estatística de Pearson ao excluir o paciente `#77`).

                    
**Para `#153`:**

                    $\Delta D_{153} = r_{D,153}^2 + h_{153,153} \, r_{P,153}^2 = (-1.482)^2 + 0.1415 \times (-1.351)^2 = 2.196 + 0.258 = \mathbf{2.455}$ (confere com `2.454`!).

                    $C_{153} = \dfrac{r_{P,153}^2 \, h_{153,153}}{1 - h_{153,153}} = \dfrac{(-1.351)^2 \times 0.1415}{1 - 0.1415} = \dfrac{0.2583}{0.8585} = \mathbf{0.3008}$ (confere com `0.3009`!).

                  

                

                

                  
**d) Leitura dos Gráficos Diagnósticos Logísticos ($\Delta D_i$ ou $\Delta X^2_i$ vs. $\hat{\pi}_i$):**

                  

                    **📝 Como redigir na prova:** “Ao plotar $\Delta D_i$ (ou $\Delta X^2_i$) contra as probabilidades preditas $\hat{\pi}_i$, todas as observações caem sobre exatamente duas curvas parabólicas em forma de X/V espelhado: os casos ($y_i=1$) formam o ramo decrescente da esquerda para a direita (disparando para o topo esquerdo quando $\hat{\pi}_i \to 0$ e $y_i=1$), e os não-casos ($y_i=0$) formam o ramo crescente (disparando para o topo direito quando $\hat{\pi}_i \to 1$ e $y_i=0$, exatamente onde se encontra o Paciente `#77`). Observações situadas acima da linha horizontal $\approx 4$ denotam fraco ajuste pontual, e traçar círculos com diâmetro proporcional a $C_i$ permite identificar de imediato os casos em que a má adequação ou a alavancagem alta (caso `#153`) exercem distorção severa sobre o vetor estimado $\hat{\boldsymbol{\beta}}$.”

</details>

---

## Questão 9 — Regressão Logística vs. Modelo Linear (MPL) e Comparação de Ligações Logito, Probito e Cloglog
**Bloco:** Bloco III • Logística, OR, RR & Interações | **Inspiração:** Inspirada na Lista 4 (Q1) e Atividade 6 • Profa. Rosemeire

Uma rede varejista realizou um experimento oferecendo 10 faixas crescentes de desconto ($x_i \in \{5\%, 10\%, \dots, 50\%\}$), com $m_i = 200$ clientes sorteados em cada faixa ($n = 2.000$ no total), registrando o número de clientes que resgataram o cupom ($y_i$). Foram ajustados: um Modelo de Probabilidade Linear (OLS clássico) sobre a proporção $p_i = y_i/m_i$ e três MLGs Binomiais com ligações `logit`, `probit` e `cloglog`. Analise o quadro comparativo do R.

```r
=== Comparativo de Ajuste entre 4 Modelos (N = 10 subpopulacoes, m_i = 200, gl_res = 8) ===
           Modelo   Beta0  Beta1 Resid_Dev X2_Pearson    AIC Pred_0pct
 MPL (OLS Linear) -0.1473 0.0229        --         NA -23.33   -0.1473
   Binomial Logit -3.8131 0.1341    16.557     15.767  70.03    0.0216
  Binomial Probit -2.1962 0.0778    17.765     18.291  71.24    0.0140
 Binomial Cloglog -3.1228 0.0915     1.102      1.164  54.57    0.0431
 Pred_60pct
     1.2268
     0.9857
     0.9932
     1.0000

--- Valor critico de referencia (gl = 8, alfa = 5%) ---
chi^2_(8; 0,95) = 15,507
```

### Subitens Discursivos
- **a)** Apresente e explique as **três violações teóricas fundamentais** que invalidam o Modelo de Probabilidade Linear ($\mathbb{E}[Y_i] = \beta_0 + \beta_1 x_i$ ajustado por MQO/OLS) para respostas binárias ou proporções — ilustrando concretamente com as predições `Pred_0pct` e `Pred_60pct` na saída do R.
- **b)** Escreva explicitamente as três funções de ligação $g(\pi_i)$ ajustadas nos MLGs (**Logito**, **Probito** e **Complemento Log-Log**) e suas respectivas funções inversas $\pi_i(\eta_i)$, contrastando o comportamento de simetria em torno de $\pi = 0,5$ versus assimetria.
- **c)** Demonstre algebricamente por que apenas a ligação **Logito** (canônica da Binomial) fornece um parâmetro $\beta_1$ diretamente interpretável em termos de **Razão de Chances (Odds Ratio)** invariante ao longo de $x$, enquanto no Probito e na Cloglog essa propriedade desaparece.
- **d)** Compare as deviances residuais, estatísticas $X^2$ e AICs dos três modelos binomiais na tabela acima ($gl = 8$, $\chi^2_{8; 0,95} = 15,507$) e determine qual ligação deve ser escolhida para este experimento, justificando tanto estatisticamente quanto graficamente.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Olhe para os números absurdos da primeira linha (MPL) na saída do R: `Pred_0pct = -0,1473` e `Pred_60pct = 1,2268`! Quais são os outros dois defeitos clássicos da reta OLS em Bernoulli/Binomial (variância e distribuição)?

                
**Item b):** Escreva $\eta = \ln(\pi/(1-\pi))$ (logito), $\eta = \Phi^{-1}(\pi)$ (probito) e $\eta = \ln(-\ln(1-\pi))$ (cloglog), e inverta cada uma isolando $\pi$.

                
**Item c):** No logito, $\text{Chance}(x) = \frac{\pi(x)}{1-\pi(x)} = \exp(\beta_0 + \beta_1 x)$. Divida $\text{Chance}(x+1)$ por $\text{Chance}(x)$ e veja que tudo cancela sobrando $\exp(\beta_1)$ independente de $x$!

                
**Item d):** Compare os três `Resid_Dev` contra $\chi^2_{8; 0,95} = 15,507$: Logito ($16,557$) e Probito ($17,765$) passam ou excedem $15,507$? E quanto à Cloglog ($1,102$, AIC $= 54,57$)?

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**As 3 violações do MPL:** (1) *Limites probabilísticos:* a reta não respeita $[0,1]$ (aqui prevê $-14,7\%$ e $+122,7\%$); (2) *Heterocedasticidade estrutural:* $\text{Var}(Y_i) = \pi_i(1-\pi_i)$ varia com $x_i$, violando $\sigma^2$ constante do MQO; (3) *Não-normalidade:* os erros assumem apenas dois valores possíveis, não podendo ser normais.

                
**Pegadinha Logito vs. Cloglog:** Logito e Probito são **simétricas** em torno de $\pi = 0,5$ (a taxa com que $\pi$ sobe de $0,1$ para $0,5$ espelha a de $0,5$ para $0,9$). Já a Cloglog é **assimétrica**: aproxima-se lentamente de $0$, mas acelera e satura rapidamente rumo a $1$ — típico de adesões comerciais ou tempo-até-evento agrupado!

                
**Critérios concorrentes:** Para comparações de ligações diferentes com o mesmo número de parâmetros ($p = 2$), o menor AIC (diferença de $\approx 15,5$ pontos!) e o único modelo com $D < \chi^2_{8; 0,95} = 15,507$ apontam inequivocamente para `cloglog`.

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) As 3 Violações do Modelo de Probabilidade Linear (MPL):**

                  

                    
**Viola os limites $[0, 1]$ da probabilidade:** Como o preditor linear é irrestrito ($\mathbb{R}$), o MPL gera estimativas impossíveis fora de $[0,1]$: para $0\%$ de desconto estima probabilidade negativa de **$-0,1473$** e para $60\%$ de desconto estima **$1,2268$ ($122,7\%$)**!

                    
**Violação da homocedasticidade:** Em dados binomiais $\text{Var}(p_i) = \pi_i(1-\pi_i)/m_i$ depende diretamente da própria média $\pi_i = \beta_0 + \beta_1 x_i$, invalidando os erros-padrão e testes $t$ usuais do OLS.

                    
**Não-normalidade dos erros e efeito linear irrealista:** Os resíduos não são normais e supor acréscimo constante $\beta_1 = 2,29\%$ por $1\%$ de desconto não reflete o comportamento de saturação nas caudas.

                  

                

                

                  
**b) Formulação das Três Ligações e Simetria:**

                  

                    
**Logito (Simétrica em $\pi=0,5$):** $\eta_i = \ln\!\left(\dfrac{\pi_i}{1-\pi_i}\right) \iff \pi_i = \dfrac{e^{\eta_i}}{1 + e^{\eta_i}}$.

                    
**Probito (Simétrica em $\pi=0,5$):** $\eta_i = \Phi^{-1}(\pi_i) \iff \pi_i = \Phi(\eta_i)$ (FDA da Normal Padrão, com caudas mais leves que a logística).

                    
**Complemento Log-Log (Assimétrica):** $\eta_i = \ln\!\big(-\ln(1-\pi_i)\big) \iff \pi_i = 1 - \exp\!\big(-\exp(\eta_i)\big)$. É assimétrica: parte devagar próximo a $0$ e sobe abruptamente em direção a $1$.

                  

                

                

                  
**c) Por que apenas a Logito entrega OR Constante via $\exp(\beta_1)$:**

                  
Na ligação logito, $\ln(\text{Chance}(x)) = \beta_0 + \beta_1 x$, logo a chance é $\frac{\pi(x)}{1-\pi(x)} = e^{\beta_0 + \beta_1 x}$. Calculando a Razão de Chances para um incremento unitário de $x$ para $x+1$:

                  
$$OR = \frac{\pi(x+1)/[1-\pi(x+1)]}{\pi(x)/[1-\pi(x)]} = \frac{e^{\beta_0 + \beta_1(x+1)}}{e^{\beta_0 + \beta_1 x}} = e^{\beta_1}$$

                  
Note que o ponto de partida $x$ e o intercepto $\beta_0$ cancelam-se perfeitamente! Em Probito e Cloglog essa razão não cancela $\Phi(\cdot)$ nem $\exp(-e^{\cdot})$, logo o OR varia conforme $x$ (em Cloglog, o que permanece constante é a razão entre $\ln(1-\pi)$, ligada à razão de taxas instantâneas / *hazard ratio*).

                

                

                  
**d) Escolha Estatística da Ligação na Tabela do R:**

                  

                    **📝 Como redigir na prova:** “Comparando os modelos binomiais ($gl = 8$, crítico $\chi^2_{8;0,95} = 15,507$), tanto a ligação Logito ($D = 16,557 > 15,507$, $p = 0,035$; $\text{AIC} = 70,03$) quanto a Probito ($D = 17,765 > 15,507$, $p = 0,023$; $\text{AIC} = 71,24$) rejeitam a hipótese nula de bom ajuste ao nível de 5% devido ao padrão assimétrico de adesão nos descontos elevados. Em contraste, a ligação **Complemento Log-Log** reduz drasticamente o desvio para $D = 1,102$ ($p = 0,997 \gg 0,05$; $X^2 = 1,164$) e apresenta $\text{AIC} = 54,57$ ($15,46$ pontos inferior ao Logito). Portanto, escolhe-se a ligação `cloglog` como a mais adequada para descrever a taxa de resgate de cupons (sacrificando-se o OR constante em troca de excelente bondade de ajuste).”

</details>

---

## Questão 10 — Odds Ratio com Salto c = 10, Contraste entre Categorias Não-Referência (vcov) e Armadilha da Codificação {-1, +1}
**Bloco:** Bloco III • Logística, OR, RR & Interações | **Inspiração:** Inspirada na Lista 4 (Q2) e Atividade 7 • Profa. Rosemeire

Em um estudo cardiológico com $n = 450$ adultos, modelou-se a presença de obstrução coronariana ($Y=1$) via regressão logística em função de `Idade` (contínua, anos), `Colesterol` (Normal [ref.], Limítrofe, Alto) e `Sedentarismo`. Para explorar parametrizações distintas, ajustou-se o **Modelo I** usando variável dummy binária usual `Sed_01` $\in \{0, 1\}$ e o **Modelo II** substituindo-a pelo contraste soma `Sed_pm1` $\in \{-1, +1\}$ ($-1$ para ativos, $+1$ para sedentários).

```r
=== MODELO I: Codificacao usual dummy (0 = Ativo, 1 = Sedentario) ===

Call:
glm(formula = DC ~ Idade + Colesterol + Sed_01, family = binomial(link = "logit"), 
    data = df10)

Coefficients:
                     Estimate Std. Error z value Pr(>|z|)    
(Intercept)         -3.140428   0.527885  -5.949 2.70e-09 ***
Idade                0.044218   0.008458   5.228 1.72e-07 ***
ColesterolLimitrofe  0.651881   0.230290   2.831  0.00464 ** 
ColesterolAlto       1.097192   0.267563   4.101 4.12e-05 ***
Sed_01               0.621659   0.202310   3.073  0.00212 ** 
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 622.09  on 449  degrees of freedom
Residual deviance: 566.39  on 445  degrees of freedom
AIC: 576.39

Number of Fisher Scoring iterations: 4


=== MODELO II: Mesmos dados, apenas trocando Sedentarismo para (-1 = Ativo, +1 = Sedentario) ===
                   Estimate Std. Error z value Pr(>|z|)
Sed_pm1            0.310829   0.101155   3.073  0.00212 **

=== Submatriz de Covariancias Estimadas vcov(mod_I) para Colesterol ===
                    ColesterolLimitrofe ColesterolAlto
ColesterolLimitrofe            0.053034       0.027115
ColesterolAlto                 0.027115       0.071590
```

### Subitens Discursivos
- **a)** A partir do Modelo I, calcule e interprete o **Odds Ratio (OR)** e o respectivo **IC de 95%** da covariável contínua `Idade` para um acréscimo de **$1$ ano** e para um acréscimo de **$c = 10$ anos** ($z_{0,975} = 1,96$). Explique a armadilha de multiplicar apenas o $\text{OR}$ unitário por $10$ em vez de elevar à 10ª potência.
- **b)** Explique matematicamente por que, no Modelo II (codificação $\{-1, +1\}$), a estimativa e o erro-padrão de `Sed_pm1` equivalem exatamente à **metade** dos obtidos no Modelo I. Calcule o $\text{OR}$ e o IC 95% comparando Sedentários ($+1$) vs. Ativos ($-1$) usando exclusivamente os números do Modelo II, provando a equivalência exata.
- **c)** Compare diretamente a chance de obstrução coronariana entre indivíduos com **Colesterol Alto vs. Colesterol Limítrofe** (ambas fora da categoria de referência `Normal`), obtendo a estimativa pontual de $\text{OR}_{\text{Alto vs. Lim}}$ e seu **IC de 95% exato** por meio da submatriz `vcov`.
- **d)** Redija um parágrafo conclusivo completo no padrão epidemiológico exigido em prova interpretando todos os achados de `Idade`, `Sedentarismo` e `Colesterol`.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Para incremento $\Delta x = c$, primeiro multiplicamos o coeficiente na escala do logito ($c \cdot \hat{\beta}$ e erro-padrão $c \cdot \text{EP}(\hat{\beta})$) e só depois exponenciamos! Logo $\text{OR}_{10\text{ anos}} = \exp(10\hat{\beta}) = (\text{OR}_{1\text{ ano}})^{10}$.

                
**Item b):** Quando passamos de $Z = -1$ (Ativo) para $Z = +1$ (Sedentário), o salto na covariável codificada é $\Delta Z = (+1) - (-1) = \mathbf{2}$ unidades! Logo o contraste de logitos é $\gamma(+1) - \gamma(-1) = 2\gamma$.

                
**Item c):** A diferença de logitos entre `Alto` e `Limitrofe` é $\hat{\delta} = \hat{\beta}_{\text{Alto}} - \hat{\beta}_{\text{Lim}}$. Atenção à variância da diferença de estimadores correlacionados: $\text{Var}(A - B) = \text{Var}(A) + \text{Var}(B) - 2\text{Cov}(A, B)$! Tire a covariância $0,027115$ da submatriz `vcov`!

                
**Item d):** Lembre os 3 pilares de interpretação na frase da Profa. Rosemeire: (1) ajustar pelas demais variáveis (“mantendo constantes...”); (2) quantificar em “vezes mais chance” ou “% a mais de chance”; (3) citar o IC 95% e verificar se contém $1$.

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Armadilha clássica 1 (Codificação {-1,+1}):** Um erro fatal de prova é calcular $\exp(\hat{\gamma}) = \exp(0,3108) = 1,36$ achando que esse é o OR de sedentário vs. ativo! Errado: $1,36$ compara apenas sedentário com a *média grand-mean* ($Z=0$). Para comparar $+1$ vs. $-1$, é obrigatório dobrar coeficiente e erro-padrão: $\exp(2\hat{\gamma})$ e $\exp[2(\hat{\gamma} \pm 1,96\cdot\text{EP}(\hat{\gamma}))]$!

                
**Armadilha clássica 2 (Variância de contraste):** Jamais some as variâncias sem subtrair $2\text{Cov}(\hat{\beta}_A, \hat{\beta}_B)$! Como duas dummies do mesmo fator categórico compartilham a mesma categoria de referência (`Normal`), suas estimativas têm forte covariância positiva ($+0,027115$); ignorar a covariância superestima grosseiramente a largura do IC!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Odds Ratio de `Idade` ($1$ ano vs. salto $c = 10$ anos):**

                  

                    
**1 ano:** $\widehat{\text{OR}}_1 = \exp(0.044218) = \mathbf{1.0452}$ (+4,52% na chance por ano).

                    
**10 anos ($c = 10$):** Primeiro multiplicamos $\hat{\beta}$ e o erro-padrão por $10$: $\hat{\Delta} = 10 \times 0.044218 = 0.4422$ e $\text{EP}(10\hat{\beta}) = 10 \times 0.008458 = 0.08458$.

                    $\widehat{\text{OR}}_{10} = \exp(0.4422) = (1,0452)^{10} = \mathbf{1.5561}$ (note que multiplicar $10 \times 1,0452 = 10,45$ seria absurdo!).

                    **IC 95% ($10$ anos):** $\exp(0.4422 \pm 1,96 \times 0.08458) = \exp([0.2764;\,0.6080]) = \mathbf{[1.318;\,1.837]}$ (excluindo $1$, logo altamente significativo).

                  

                

                

                  
**b) Equivalência Matemática da Codificação $\{-1, +1\}$:**

                  
Na codificação $X \in \{0,1\}$, a diferença entre categorias vale $\Delta X = 1 - 0 = 1$. Na codificação contraste soma $Z \in \{-1,+1\}$, $Z = 2X - 1$, de modo que o salto entre Ativo ($-1$) e Sedentário ($+1$) passa a medir $\Delta Z = 2$ unidades. Por isso, o modelo preserva exatamente os mesmos valores preditos $\hat{\pi}_i$, obrigando $2\hat{\gamma} = \hat{\beta}$, ou seja:

                  
$\hat{\gamma} = 0.621659/2 = \mathbf{0.310829}$ e $\text{EP}(\hat{\gamma}) = 0.202310/2 = \mathbf{0.101155}$ (enquanto $z = \hat{\gamma}/\text{EP} = 3.073$ permanece rigorosamente idêntico!).

                  Calculando pelo Modelo II: $\widehat{\text{OR}} = \exp(2 \times 0.310829) = \exp(0.621659) = \mathbf{1.8620}$, e IC 95% $= \exp[2(0.310829 \pm 1,96 \times 0.101155)] = \mathbf{[1.252;\,2.768]}$, coincidente até a última casa com o Modelo I!

                

                

                  
**c) Contraste Direto Colesterol Alto vs. Colesterol Limítrofe:**

                  
$\hat{\delta} = \hat{\beta}_{\text{Alto}} - \hat{\beta}_{\text{Lim}} = 1.097192 - 0.651881 = \mathbf{0.44531}$, logo $\widehat{\text{OR}}_{\text{Alto/Lim}} = \exp(0.44531) = \mathbf{1.561}$.

                  
Usando a matriz de covariâncias para o erro-padrão do contraste:

                  
$\widehat{\text{Var}}(\hat{\beta}_{\text{Alto}} - \hat{\beta}_{\text{Lim}}) = 0.071590 + 0.053033 - 2(0.027115) = 0.070394$

                  $\text{EP}(\hat{\delta}) = \sqrt{0.070394} = \mathbf{0.26532}$

                  $\text{IC}_{95\%}(\text{OR}_{\text{Alto/Lim}}) = \exp(0.44531 \pm 1,96 \times 0.26532) = \mathbf{[0.928;\,2.626]}$.

                  
(Observe que como o IC 95% de $[0,928;\,2,626]$ tangencia/contém o $1$ por pouco, $z = 1.678$, $p = 0,080$, indicando diferença limítrofe entre Alto e Limítrofe quando comparados diretamente).

                

                

                  
**d) Texto Discursivo Padrão Profa. Rosemeire:**

                  

                    **📝 Como redigir na prova:** “Ajustando-se pelas demais covariáveis do modelo, cada acréscimo de 10 anos na idade multiplica a chance de obstrução coronariana por $1,56$ (IC 95%: $[1,32;\,1,84]$, aumento de $55,6\%$ na chance por década). Indivíduos sedentários apresentam uma chance estimada $86,2\%$ maior de obstrução em comparação aos ativos ($\text{OR} = 1,862$; IC 95%: $[1,25;\,2,77]$), resultado invariante à codificação escolhida. Quanto ao colesterol, pacientes com nível Alto possuem chance $2,996\times$ maior que o grupo Normal ($e^{1,0972} = 3,00$; $p < 0,001$) e $1,561\times$ maior quando comparados ao grupo Limítrofe (IC 95%: $[0,93;\,2,63]$).”

</details>

---

## Questão 11 — Regressão Logística com Interação e Sinais Opostos: As 4 Hipóteses de Modificação de Efeito e OR Estratificado
**Bloco:** Bloco III • Logística, OR, RR & Interações | **Inspiração:** Inspirada na Lista 5 e Atividade 8 • Profa. Rosemeire

Em um estudo longitudinal com $n = 620$ pacientes pós-alta cardíaca, ajustou-se um modelo logístico para o risco de re-hospitalização em 90 dias ($Y=1$) em função de `Diabetes` ($X_1 \in \{0=\text{Não}, 1=\text{Sim}\}$), programa de `Acomp`anhamento domiciliar ($X_2 \in \{0=\text{Não}, 1=\text{Sim}\}$), `Idade_c50` ($X_3 = \text{Idade} - 50$, centrada aos 50 anos) e as interações `Diabetes:Acomp` e `Diabetes:Idade_c50`.

```r

Call:
glm(formula = Rehosp ~ Diabetes + Acomp + Idade_c50 + Diabetes:Acomp + 
    Diabetes:Idade_c50, family = binomial(link = "logit"), data = df11)

Coefficients:
                   Estimate Std. Error z value Pr(>|z|)    
(Intercept)        -1.96401    0.22153  -8.866  < 2e-16 ***
Diabetes            1.11347    0.33312   3.343  0.00083 ***
Acomp              -0.09102    0.28040  -0.325  0.74549    
Idade_c50           0.02541    0.01042   2.439  0.01475 *  
Diabetes:Acomp     -0.74077    0.42889  -1.727  0.08414 .  
Diabetes:Idade_c50  0.04005    0.01642   2.439  0.01474 *  
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 652.35  on 619  degrees of freedom
Residual deviance: 572.45  on 614  degrees of freedom
AIC: 584.45

Number of Fisher Scoring iterations: 4


=== Teste Omnibus da Inclusao das Interacoes (Modelo Aditivo vs. Com Interacoes) ===
Analysis of Deviance Table

Model 1: Rehosp ~ Diabetes + Acomp + Idade_c50
Model 2: Rehosp ~ Diabetes + Acomp + Idade_c50 + Diabetes:Acomp + Diabetes:Idade_c50
  Resid. Df Resid. Dev Df Deviance Pr(>Chi)  
1       616     580.99                       
2       614     572.45  2   8.5436  0.01396 *
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

### Subitens Discursivos
- **a)** Considerando um modelo genérico com interação entre duas binárias $\eta = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \beta_{12} X_1 X_2$, enuncie e interprete com precisão clínica as **quatro hipóteses estatísticas fundamentais** em modelos com interação: (i) $H_0: \beta_{12} = 0$; (ii) $H_0: \beta_1 = \beta_{12} = 0$; (iii) $H_0: \beta_1 = 0$; e (iv) $H_0: \beta_1 + \beta_{12} = 0$.
- **b)** Calcule a **Razão de Chances (OR) de re-hospitalização atribuível à Diabetes** em um paciente de **50 anos** (`Idade_c50 = 0`): (1) na ausência de acompanhamento domiciliar ($X_2 = 0$); e (2) na presença de acompanhamento domiciliar ($X_2 = 1$).
- **c)** Analise o fenômeno dos **sinais opostos** encontrado entre o efeito principal de `Diabetes` ($\hat{\beta}_1 = +1.1135$) e a interação `Diabetes:Acomp` ($\hat{\beta}_{12} = -0.7408$): o acompanhamento domiciliar transformou a diabetes em um “fator protetor” ou reduziu parcialmente o excesso de risco? Prove numericamente.
- **d)** Com os termos de idade, escreva a expressão geral do $\text{OR}_{\text{Diabetes}}(X_2, \text{Idade})$ e calcule especificamente quanto vale o OR da Diabetes para pacientes **sem acompanhamento** aos **50 anos** e aos **70 anos**.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Lembre que o efeito da variável $X_1$ dentro do preditor linear é dado pelo coeficiente efetivo $( \beta_1 + \beta_{12} X_2 ) \cdot X_1$. Substitua $X_2 = 0$ e $X_2 = 1$ para entender imediatamente o significado das hipóteses (iii) e (iv)!

                
**Item b):** Quando `Idade_c50 = 0` (50 anos), o termo `Diabetes:Idade_c50` zera! Então o log-OR da diabetes reduz-se a $\hat{\beta}_{\text{Diab}} + \hat{\beta}_{\text{Diab:Acomp}} \cdot X_2$.

                
**Item c):** Não olhe o sinal isolado da interação $-0,7408$ achando que tudo ficou protetor! Some $+1,1135 + (-0,7408)$: o resultado final $+0,3727$ ainda é positivo (ou seja, $OR > 1$)! Logo, o sinal negativo representa uma **atenuação / redução de danos**!

                
**Item d):** Isole tudo que multiplica $X_1 = \text{Diabetes}$ na equação do logito: $\text{log-OR}_{\text{Diab}} = \hat{\beta}_{\text{Diab}} + \hat{\beta}_{\text{Diab:Acomp}} X_2 + \hat{\beta}_{\text{Diab:Idade}} (\text{Idade} - 50)$. Para 70 anos, faça $(\text{Idade} - 50) = +20$!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Erro fatal na presença de interação:** Na existência de interação significativa ($p = 0,01396$ no teste ANODEV global!), é **estatisticamente errado** interpretar `exp(coef[Diabetes]) = exp(1,1135) = 3,045` como “o OR global da diabetes para todos os pacientes”! Ele vale *exclusivamente* para o estrato em que todas as variáveis interagentes valem zero (ou seja, sem acompanhamento e exatamente com 50 anos)!

                
**Princípio da Hierarquia:** Mesmo que o efeito principal isolado não atingisse $5\%$ ou que apenas uma subcomponente pareça marginal ($p = 0,077$ para `Diabetes:Acomp` e $p = 0,049$ para `Diabetes:Idade_c50`), ao reter qualquer interação no modelo devemos obrigatoriamente manter todos os respectivos termos principais (`Diabetes`, `Acomp`, `Idade_c50`).

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) As 4 Hipóteses Fundamentais em Modelos com Interação:**

                  

                    
**(i) $H_0: \beta_{12} = 0$:** Testa *ausência de interação* (homogeneidade de efeito). Sob $H_0$, o OR de $X_1$ é exatamente o mesmo quer $X_2 = 0$ quer $X_2 = 1$ (efeito aditivo na escala logito).

                    
**(ii) $H_0: \beta_1 = \beta_{12} = 0$ (teste conjunto com 2 gl):** Testa *ausência total de efeito de $X_1$* em qualquer estrato de $X_2$. Só quando não rejeitamos essa hipótese podemos eliminar $X_1$ completamente.

                    
**(iii) $H_0: \beta_1 = 0$:** Testa *apenas* se $X_1$ tem efeito no grupo de referência $X_2 = 0$ ($\text{OR}_{X_1 | X_2=0} = e^{\beta_1} = 1$).

                    
**(iv) $H_0: \beta_1 + \beta_{12} = 0$:** Testa se $X_1$ ainda exerce algum efeito dentro do segundo estrato $X_2 = 1$ ($\text{OR}_{X_1 | X_2=1} = e^{\beta_1 + \beta_{12}} = 1$).

                  

                

                

                  
**b) OR da Diabetes aos 50 anos ($\text{Idade\_c50} = 0$) por Estrato de Acompanhamento:**

                  

                    
**Sem acompanhamento ($X_2 = 0$):** $\text{log-OR} = 1.1135 \implies \widehat{\text{OR}}_{\text{Diab} | X_2=0, 50} = \exp(1.1135) = \mathbf{3.045}$. (Aos 50 anos sem programa, diabéticos têm chance **204,5% maior** de re-hospitalização).

                    
**Com acompanhamento ($X_2 = 1$):** $\text{log-OR} = 1.1135 + (-0.7408) = \mathbf{0.3727} \implies \widehat{\text{OR}}_{\text{Diab} | X_2=1, 50} = \exp(0.3727) = \mathbf{1.452}$. (O excesso de chance cai para $+45,2\%$).

                  

                

                

                  
**c) Interpretação dos Sinais Opostos ($+1,1135$ vs. $-0,7408$):**

                  
O acompanhamento domiciliar **não transformou a diabetes em fator protetor**! Embora a interação tenha sinal negativo ($-0,7408$), sua magnitude é inferior em módulo ao efeito principal positivo ($+1,1135$), de modo que o log-OR líquido permanece positivo ($+0,3727 > 0$) e o OR permanece maior que $1$ ($\text{OR} = 1,452$). O sinal oposto significa **atenuação / efeito tampão (modificação de efeito antagônica parcial)**: o acompanhamento reduz a disparidade causada pela diabetes de $3,045\times$ para $1,452\times$.

                

                

                  
**d) Efeito Modificado pela Idade ($50$ vs. $70$ anos, $X_2 = 0$):**

                  
Expressão geral: $\text{OR}_{\text{Diab}}(X_2, \text{Idade}) = \exp\!\big(1.1135 -0.7408\,X_2 + 0.0400(\text{Idade} - 50)\big)$.

                  

                    
Aos **50 anos** ($\text{Idade\_c50} = 0$): $\widehat{\text{OR}} = \mathbf{3.045}$.

                    
Aos **70 anos** ($\text{Idade\_c50} = +20$): $\text{log-OR} = 1.1135 + 20 \times 0.0400 = 1.9144 \implies \widehat{\text{OR}} = \exp(1.9144) = \mathbf{6.783}$.

                  

                  

                    **📝 Como redigir na prova:** “A inclusão das interações é conjuntamente significativa ($\Delta D = 8,544$; $p = 0,0140$), proibindo a interpretação isolada do efeito principal da diabetes. Existe forte modificação de efeito: entre pacientes sem acompanhamento domiciliar, o impacto da diabetes intensifica-se com a idade, elevando o OR de $3,04$ aos 50 anos para $6,78$ aos 70 anos. Em contrapartida, a interação negativa `Diabetes:Acomp` ($-0,741$) atua como redutora de danos, amortecendo o OR aos 50 anos de $3,04$ para $1,45$ sem anular ou inverter o risco basal positivo da doença.”

</details>

---

## Questão 12 — Epidemiologia em Tabelas 2×2: Coorte vs. Caso-Controle, OR vs. Risco Relativo (RR), Risco Atribuível (d) e Condição de Cornfield
**Bloco:** Bloco III • Logística, OR, RR & Interações | **Inspiração:** Inspirada nos Slides de Epidemiologia & Lista 4 (Q5) • Profa. Rosemeire

Um epidemiologista conduziu dois desenhos observacionais distintos para avaliar o impacto da exposição ocupacional a solventes:
• **Estudo A (Coorte prospectiva — dermatite frequente):** Acompanhou $200$ trabalhadores expostos ($80$ desenvolveram dermatite, $120$ sadios) e $200$ não-expostos ($40$ desenvolveram dermatite, $160$ sadios).
• **Estudo B (Caso-Controle — neoplasia rara, incidência populacional basal $p_0 = 0,004 = 0,4\%$):** Selecionou $150$ casos diagnosticados ($90$ expostos, $60$ não-expostos) e $300$ controles pareados ($90$ expostos, $210$ não-expostos).

```r
=== ESTUDO A (Coorte: glm(cbind(Doentes, Sadios) ~ Exposicao, family = binomial)) ===
Coefficients:
             Estimate Std. Error z value Pr(>|z|)
(Intercept)   -1.386294   0.176777  -6.982 2.91e-12 ***
ExposicaoSim  0.980829   0.228218   4.307 1.66e-05 ***

=== ESTUDO B (Caso-Controle: glm(cbind(Casos, Controles) ~ Exposicao, family = binomial)) ===
Coefficients:
             Estimate Std. Error z value Pr(>|z|)
(Intercept)   -1.252763   0.146385  -8.210  < 2e-16 ***
ExposicaoSim  1.252763   0.208928   5.974 2.31e-09 ***
```

### Subitens Discursivos
- **a)** No **Estudo A (Coorte)**, monte a tabela de contingência $2\times 2$ e calcule manualmente: o risco nos expostos ($\hat{p}_1$), o risco nos não-expostos ($\hat{p}_0$), o **Risco Relativo ($\widehat{RR}$)**, o **Risco Atribuível ($\hat{d}$)** e a **Razão de Chances ($\widehat{OR}$)**, verificando numericamente que $\exp(\hat{\beta}_1)$ no R coincide com $\widehat{OR}$.
- **b)** Por que no Estudo A o $\widehat{OR} = 2,67$ é substancialmente maior que o verdadeiro Risco Relativo ($\widehat{RR} = 2,00$)? Demonstre usando a identidade exata $OR = RR \cdot \frac{1 - p_0}{1 - p_1}$ e indique o perigo clínico de confundir $OR$ com $RR$ em desfechos frequentes.
- **c)** No **Estudo B (Caso-Controle)**, mostre por que **não é possível** estimar diretamente a probabilidade $p_1$, $p_0$, o $RR$ nem o risco atribuível $d$ (demonstrando o que o intercepto $\hat{\beta}_0 = -1.2528$ realmente mede nesse delineamento), e prove matematicamente pelo Teorema de Bayes / razão de produtos cruzados que $\exp(\hat{\beta}_1)$ continua estimando o $\text{OR}$ populacional sem viés.
- **d)** Enuncie a **condição de Cornfield (aproximação da doença rara)** e explique por que, especificamente no Estudo B ($p_0 = 0,4\%$), o $\widehat{OR} = 3,50$ (calcule também seu IC 95%) pode ser validamente interpretado como excelente aproximação do Risco Relativo populacional.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Lembre das três definições fundamentais em coorte: $\hat{p}_1 = a/(a+b)$, $\hat{p}_0 = c/(c+d)$, $\widehat{RR} = \hat{p}_1 / \hat{p}_0$, $\hat{d} = \hat{p}_1 - \hat{p}_0$, e $\widehat{OR} = \frac{a\cdot d}{b\cdot c}$. Verifique também que $\exp(\hat{\beta}_0) = \frac{40}{160} = 0,25 = \text{Chance}_0$!

                
**Item b):** Compare $1 - \hat{p}_0 = 0,80$ com $1 - \hat{p}_1 = 0,60$. Como $\frac{0,80}{0,60} = 1,333 > 1$, o $OR$ sempre se afasta de $1$ mais rapidamente do que o $RR$ quando o risco basal não é pequeno!

                
**Item c):** Num caso-controle, a proporção de casos na amostra foi fixada arbitrariamente pelo pesquisador ($150$ casos vs. $300$ controles = $1:2$, não a prevalência real da população!). Logo $\hat{\beta}_0 = \ln(60/210) = \ln(\text{Odds de amostragem})$. Contudo, pela simetria da razão cruzada $\frac{P(D=1|E=1)/P(D=0|E=1)}{P(D=1|E=0)/P(D=0|E=0)} = \frac{P(E=1|D=1)/P(E=0|D=1)}{P(E=1|D=0)/P(E=0|D=0)}$, a razão de chances é idêntica!

                
**Item d):** Qual o limiar prático da condição de Cornfield para $p_0$ e $p_1$? Abaixo de $10\%$ (ou idealmente $< 5\%$), $\frac{1 - p_0}{1 - p_1} \approx 1 \implies OR \approx RR$!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Regra de Ouro Epidemiológica da Profa. Rosemeire:**
                  
• *Desenho Coorte / Transversal:* podemos estimar $p_1$, $p_0$, $RR$, $d = p_1 - p_0$ e $OR$. Se o desfecho for comum ($p_0 \ge 10\%$), **nunca** leia $OR$ como “risco $2,67\times$ maior” — diga “*chance* $2,67\times$ maior”, pois o *risco* real é apenas $2,00\times$ maior!
                  
• *Desenho Caso-Controle:* $RR$, $d$ e probabilidades absolutas preditas estão **proibidos** sem conhecimento externo da taxa de amostragem ou prevalência! Somente $\beta_1$ ($\text{OR} = e^{\beta_1}$) é estimável de forma consistente.

                
**Verifique a conta de Cornfield no Estudo B:** Com $p_0 = 0,004$ e $RR \approx 3,5$, temos $p_1 \approx 3,5 \times 0,004 = 0,014$, de modo que o fator de distorção vale $\frac{1 - 0,004}{1 - 0,014} = \frac{0,996}{0,986} = 1,010$ (erro relativo de apenas $1\%$!).

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Métricas da Coorte (Estudo A):**

                  

                    
Riscos absolutos: $\hat{p}_1 = \frac{80}{200} = \mathbf{0,40\,(40\%)}$ e $\hat{p}_0 = \frac{40}{200} = \mathbf{0,20\,(20\%)}$.

                    
**Risco Relativo:** $\widehat{RR} = \dfrac{0,40}{0,20} = \mathbf{2,00}$ (expostos têm o *dobro* do risco).

                    
**Risco Atribuível ($d$):** $\hat{d} = \hat{p}_1 - \hat{p}_0 = 0,40 - 0,20 = \mathbf{0,20}$ (excesso absoluto de **20 casos a cada 100 trabalhadores expostos**).

                    
**Razão de Chances:** $\widehat{OR} = \dfrac{80 \times 160}{120 \times 40} = \dfrac{12.800}{4.800} = \mathbf{2,6667}$. Verificando no R: $\exp(0.980829) = \mathbf{2.6667}$.

                  

                

                

                  
**b) Discrepância $OR = 2,67$ vs. $RR = 2,00$:**

                  
Algebricamente, $OR = \dfrac{p_1/(1-p_1)}{p_0/(1-p_0)} = \dfrac{p_1}{p_0} \cdot \dfrac{1 - p_0}{1 - p_1} = RR \cdot \dfrac{1 - p_0}{1 - p_1}$. Substituindo os valores da coorte: $2,00 \times \dfrac{1 - 0,20}{1 - 0,40} = 2,00 \times \dfrac{0,80}{0,60} = 2,6667$. Sempre que o fator de risco aumenta a doença ($RR > 1$) e o desfecho é frequente ($p_0 = 20\%$, $p_1 = 40\%$), o termo $(1-p_0)/(1-p_1) > 1$ faz o $OR$ afastar-se de $1$ mais que o $RR$, superestimando a magnitude da associação em $33,3\%$ caso fosse erroneamente lido como risco relativo.

                

                

                  
**c) Restrições e Invariância no Caso-Controle (Estudo B):**

                  
No desenho Caso-Controle, o pesquisador fixa arbitrariamente as frações amostrais de doentes ($150$) e controles ($300$), logo as proporções amostrais $60/(60+210)$ ou $90/(90+90)$ não representam incidências reais! Na saída do R, $\hat{\beta}_0 = \ln(60/210) = -1,2528$ reflete apenas a razão de amostragem dos não-expostos (impedindo calcular $\hat{p}_1, \hat{p}_0, RR$ ou $d$ diretamente). Todavia, pelo Teorema de Bayes (propriedade de simetria da razão de produtos cruzados), a razão de chances de exposição entre casos e controles iguala exatamente a razão de chances da doença entre expostos e não-expostos:

                  
$\widehat{OR}_B = \dfrac{90 \times 210}{60 \times 90} = \dfrac{18.900}{5.400} = \mathbf{3,500} = \exp(1.252763)$.

                

                

                  
**d) Condição de Cornfield e IC 95% no Estudo B:**

                  
A **condição de Cornfield (1951)** estabelece que, quando o desfecho é raro na população (usualmente $p_0 < 10\%$, e tanto melhor quanto menor que $5\%$), tanto $1 - p_0 \approx 1$ quanto $1 - p_1 \approx 1$, implicando $\frac{1 - p_0}{1 - p_1} \approx 1 \implies OR \approx RR$. No Estudo B ($p_0 = 0,004$, $p_1 \approx 0,014$), o fator de correção é $\frac{0,996}{0,986} = 1,010$ (discrepância inferior a $1\%$).

                  
IC 95% no Estudo B: $\exp(1.252763 \pm 1,96 \times 0.208928) = \mathbf{[2.324;\,5.271]}$.

                  

                    **📝 Como redigir na prova:** “Na coorte A ($p_0 = 20\%$, doença comum), expostos apresentam o dobro do risco de dermatite ($\widehat{RR} = 2,00$) e excesso absoluto de $20$ pontos percentuais ($\hat{d} = 0,20$); o $\widehat{OR} = 2,67$ não deve ser chamado de risco relativo pois o superestima. No caso-controle B, não é possível estimar $RR$, $d$ ou risco absoluto porque $\hat{\beta}_0$ está contaminado pela fração amostral; contudo, como a neoplasia é muito rara ($p_0 = 0,4\% < 10\%$), aplica-se a condição de Cornfield, permitindo interpretar $\widehat{OR} = 3,50$ (IC 95%: $[2,32;\,5,27]$) como uma aproximação acurada de que expostos têm cerca de $3,5$ vezes o risco populacional dos não-expostos.”

</details>

---

## Questão 13 — Predição de Probabilidade Individual e Construção do IC 95% via Escala do Preditor Linear (predict(..., type='link')) e vcov()
**Bloco:** Bloco IV • Predição, HL & ROC | **Inspiração:** Inspirada na Lista 5 e Atividade 7 • Profa. Rosemeire

Um banco ajustou uma regressão logística em $n = 350$ empréstimos para estimar a probabilidade de inadimplência ($Y=1$) em função do `Renda_comp` (% da renda comprometida pela parcela) e `Atraso` prévio ($0=$Não, $1=$Sim). Deseja-se prever a probabilidade pontual e o intervalo de confiança de 95% para um candidato específico com perfil **35% de comprometimento de renda e histórico prévio de atraso ($\mathbf{x}_0^\top = [1,\, 35,\, 1]$)**.

```r

Call:
glm(formula = Inadimp ~ Renda_comp + Atraso, family = binomial(link = "logit"), 
    data = df13)

Coefficients:
             Estimate Std. Error z value Pr(>|z|)    
(Intercept) -2.214745   0.348017  -6.364 1.97e-10 ***
Renda_comp   0.064092   0.009667   6.630 3.36e-11 ***
Atraso       0.795915   0.247332   3.218  0.00129 ** 
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 484.47  on 349  degrees of freedom
Residual deviance: 426.07  on 347  degrees of freedom
AIC: 432.07

Number of Fisher Scoring iterations: 4


=== Matriz de Variancias e Covariancias Estimadas: vcov(mod_cred) ===
            (Intercept) Renda_comp    Atraso
(Intercept)    0.121116  -0.003059 -0.033155
Renda_comp    -0.003059   0.000093  0.000372
Atraso        -0.033155   0.000372  0.061173

=== Saida parcial de predict(mod_cred, newdata, type = "link", se.fit = TRUE) ===
$fit      = 0.824383
$se.fit   = 0.205850
--- Comparativo para alerta do item d: predict(..., type = "response", se.fit = TRUE) ---
$fit      = 0.695166
$se.fit   = 0.043622
```

### Subitens Discursivos
- **a)** Reproduza manualmente a estimativa pontual do preditor linear $\hat{\eta}_0 = 0.8244$ a partir da coluna `Estimate` e calcule a **probabilidade pontual estimada de inadimplência** $\hat{\pi}_0$ para esse candidato.
- **b)** Demonstre passo a passo como o R obteve o erro-padrão $\text{EP}(\hat{\eta}_0) = 0.20585$ a partir da fórmula matricial $\sqrt{\mathbf{x}_0^\top \, \widehat{\text{Cov}}(\hat{\boldsymbol{\beta}}) \, \mathbf{x}_0}$, expandindo os produtos com os termos da matriz `vcov`.
- **c)** Construa o **intervalo de confiança de 95%** primeiro na escala do preditor linear ($[\eta_{\text{inf}}, \eta_{\text{sup}}]$) usando $z_{0,975} = 1,96$ e, em seguida, aplique a transformação logística inversa para obter o **IC de 95% da probabilidade $[\pi_{\text{inf}}, \pi_{\text{sup}}]$**.
- **d)** Explique teoricamente por que o procedimento correto utiliza `type = "link"` antes de transformar, mostrando o defeito do método ingênuo direto $\hat{\pi}_0 \pm 1,96 \cdot \text{EP}(\hat{\pi}_0)$ (compare a simetria vs. assimetria dos intervalos e o risco próximo às bordas $0$ e $1$).

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Calcule $\hat{\eta}_0 = \hat{\beta}_0 + 35\hat{\beta}_1 + 1\hat{\beta}_2$ e substitua em $\hat{\pi}_0 = \frac{\exp(\hat{\eta}_0)}{1 + \exp(\hat{\eta}_0)}$.

                
**Item b):** Escreva a variância de uma combinação linear: $\text{Var}(\hat{\beta}_0 + 35\hat{\beta}_1 + \hat{\beta}_2) = \text{Var}(\hat{\beta}_0) + 35^2 \text{Var}(\hat{\beta}_1) + \text{Var}(\hat{\beta}_2) + 2(35)\text{Cov}_{01} + 2(1)\text{Cov}_{02} + 2(35)\text{Cov}_{12}$.

                
**Item c):** Calcule $[\eta_{\text{inf}}, \eta_{\text{sup}}] = 0.8244 \pm 1,96 \times 0.20585$ e depois aplique $f(\eta) = \frac{e^\eta}{1+e^\eta}$ em cada extremo!

                
**Item d):** Compare a distância da estimativa $\hat{\pi}_0 = 0.6952$ até $\pi_{\text{inf}}$ e até $\pi_{\text{sup}}$ no intervalo transformado vs. no ingênuo $\hat{\pi}_0 \pm 1,96 \times 0.04362$: qual garante sempre limites dentro de $(0, 1)$?

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Erro clássico na construção de $\text{Var}(\hat{\eta}_0)$:** Esquecer os termos de covariância cruzada $2x_{0j}x_{0k}\text{Cov}(\hat{\beta}_j, \hat{\beta}_k)$! Se você somar apenas as variâncias da diagonal ($0,1211 + 1225 \times 0,000093 + 0,0612 = 0,296$), obteria $\text{EP} \approx 0,54$ — quase o triplo do erro-padrão real ($0,2059$), porque as covariâncias negativas ($-0,003059$ e $-0,033155$) reduzem drasticamente a variância!

                
**Ordem obrigatória das operações:** Na escala $\eta \in (-\infty, +\infty)$, a distribuição assintótica de $\hat{\eta}_0$ é aproximadamente Normal e irrestrita. Por isso, constrói-se o IC simétrico **no logito primeiro** e só depois aplica-se a função logística crescente (que preserva a cobertura exata de 95% e respeita obrigatoriamente $(0,1)$).

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Preditor Linear Manual e Probabilidade Pontual:**

                  
$\hat{\eta}_0 = -2.214745 + 35 \times (0.064092) + 1 \times (0.795915) = \mathbf{0.8244}$.

                  
$\hat{\pi}_0 = \dfrac{\exp(0.8244)}{1 + \exp(0.8244)} = \dfrac{2.2805}{3.2805} = \mathbf{0.6952}$ (ou **69,52%** de probabilidade estimada de inadimplência).

                

                

                  
**b) Decomposição Matricial de $\text{EP}(\hat{\eta}_0)$:**

                  
Com $\mathbf{x}_0^\top = (1,\, 35,\, 1)$, expandimos $\widehat{\text{Var}}(\hat{\eta}_0) = \mathbf{x}_0^\top \hat{\Sigma} \mathbf{x}_0$:

                  

                    Var(η₀) = 0,121116 + 35²(0,000093) + 0,061173 + 2(35)(-0,003059) + 2(1)(-0,033155) + 2(35)(1)(0,000372)

                            = 0,121116 + 0,113925 + 0,061173 − 0,214130 − 0,066310 + 0,026040 ≈ 0,04181 (com precisão plena: 0,042374)

                    EP(η₀) = √(0,042374) = **0.20585** (coincidindo com `$se.fit = 0.205850`).
                  

                

                

                  
**c) IC de 95% via Escala Logito Transformada:**

                  

                    
**Escala do Preditor Linear ($\eta$):**

                    $\eta_{\text{inf}} = 0.8244 - 1,96 \times 0.20585 = \mathbf{0.4209}$

                    $\eta_{\text{sup}} = 0.8244 + 1,96 \times 0.20585 = \mathbf{1.2278}$

                    
**Escala de Probabilidade ($\pi$):**

                    $\pi_{\text{inf}} = \dfrac{e^{0.4209}}{1 + e^{0.4209}} = \mathbf{0.6037}$ (60,37%)

                    $\pi_{\text{sup}} = \dfrac{e^{1.2278}}{1 + e^{1.2278}} = \mathbf{0.7734}$ (77,34%)

                    Portanto, $\text{IC}_{95\%}(\pi_0) = \mathbf{[0.604;\,0.773]}$ (de $60,4\%$ a $77,3\%$).

                  

                

                

                  
**d) Por que não usar `type = "response"` diretamente no IC:**

                  

                    **📝 Como redigir na prova:** “Na escala da probabilidade $\hat{\pi}_0$, a distribuição amostral de $\hat{\pi}_0$ é assimétrica e restrita ao intervalo $(0,1)$; aplicar o método Wald direto $\hat{\pi}_0 \pm 1,96\cdot\text{EP}(\hat{\pi}_0)$ produziria aqui um intervalo simétrico $[0,610;\,0,781]$ que não respeita a curvatura sigmoidal e, para perfis com risco alto ou baixo, facilmente extrapola $0$ ou $1$. Ao calcular o IC primeiro na escala do preditor linear (`type="link"`, onde $\hat{\eta}_0 \sim \mathcal{N}$ sem fronteiras) e aplicar a transformação logística inversa monótona, garante-se obrigatoriamente que $[0,604;\,0,773] \subset (0,1)$, preservando a assimetria correta ($\Delta_{\text{inf}} = 0,091$ vs. $\Delta_{\text{sup}} = 0,078$) e a taxa de cobertura de 95%.”

</details>

---

## Questão 14 — Falha Teórica da Deviance Residual quando m_i = 1 (Dados Desagrupados) e Teste de Qualidade de Ajuste de Hosmer-Lemeshow
**Bloco:** Bloco IV • Predição, HL & ROC | **Inspiração:** Inspirada nos Slides de Calibração/HL e Lista 5 • Profa. Rosemeire

Um hospital ajustou uma regressão logística em dados individuais não-agrupados ($n = 500$ pacientes, ou seja, $m_i = 1$ para cada linha, $p = 4$ parâmetros) para prever óbito intra-hospitalar ($Y_i \in \{0,1\}$) a partir das covariáveis contínuas `Idade` e `PAM` (pressão arterial média) e da binária `Urgencia`. Analise os resultados da deviance residual e do agrupamento em $g = 10$ decis do teste de Hosmer-Lemeshow.

```r
=== Ajuste Global no Modelo Logistico Individual (n = 500, m_i = 1, p = 4) ===
Residual deviance: 237.72  on 496 degrees of freedom

=== Tabela dos 10 Decis de Risco — Teste de Hosmer-Lemeshow ===
    Grupo n_k O1     E1 O0     E0 Contrib
  Decil 1  50  0  0.336 50 49.664  0.3383
  Decil 2  50  1  0.582 49 49.418  0.3037
  Decil 3  50  1  0.898 49 49.102  0.0118
  Decil 4  50  0  1.321 50 48.679  1.3568
  Decil 5  50  2  1.880 48 48.120  0.0080
  Decil 6  50  2  2.802 48 47.198  0.2432
  Decil 7  50  7  4.128 43 45.872  2.1780
  Decil 8  50  7  6.132 43 43.868  0.1400
  Decil 9  50  6  8.474 44 41.526  0.8697
 Decil 10  50 14 13.446 36 36.554  0.0312

=== Resultado de ResourceSelection::hoslem.test(y, fitted(mod), g = 10) ===
X-squared = 5.4807, df = 8, p-value = 0.7052
```

### Subitens Discursivos
- **a)** Um colega olhou a saída `Residual deviance: 237.72 on 496 df` e concluiu: “Como $D = 237,72 < \chi^2_{496; 0,95} \approx 548,9$, o modelo está bem ajustado!”. Refute matematicamente esse argumento explicando por que a deviance residual **falha** como teste global de bondade de ajuste em dados binários individuais ($m_i = 1$) com covariáveis contínuas (mostrando a relação de $D$ apenas com $\hat{\pi}_i$).
- **b)** Explique o algoritmo do teste de **Hosmer-Lemeshow** ($g = 10$ decis), escreva a fórmula completa de sua estatística $\hat{C}_{\text{HL}}$ e justifique por que sua distribuição qui-quadrado de referência possui **$gl = g - 2 = 8$** graus de liberdade (e não $g$ ou $g - 1$).
- **c)** Formule explicitamente $H_0$ e $H_1$ do teste de Hosmer-Lemeshow (destacando o cuidado com a inversão de sinal em relação aos testes de regressão) e interprete o resultado obtido ($\hat{C}_{\text{HL}} = 5,4807$; $p = 0,7052$). Além disso, **reproduza manualmente a contribuição do Decil 10** (`0.0312`) usando tanto a soma das duas parcelas ($O_1, O_0$) quanto a fórmula simplificada com $n_k = 50$.
- **d)** Cite e explique **duas limitações clássicas do teste de Hosmer-Lemeshow** que motivam o uso complementar de gráficos de calibração, envelopes ou curvas ROC na prática aplicada.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Escreva a deviance para $m_i = 1$: $D = -2 \sum_{i=1}^n \big[y_i \ln(\hat{\pi}_i) + (1 - y_i)\ln(1 - \hat{\pi}_i)\big]$. Ao aplicar a equação de verossimilhança $\sum y_i x_{ij} = \sum \hat{\pi}_i x_{ij}$ (que vale quando há intercepto), mostre que $\mathbb{E}[D] \approx -2 \sum [\hat{\pi}_i \ln\hat{\pi}_i + (1-\hat{\pi}_i)\ln(1-\hat{\pi}_i)]$ depende **exclusivamente de $\hat{\pi}_i$** e não mais das realizações $y_i$!

                
**Item b):** Os $n = 500$ pacientes são ordenados em ordem crescente de probabilidade predita $\hat{\pi}_{(1)} \le \dots \le \hat{\pi}_{(500)}$ e fatiados em $g = 10$ grupos iguais ($n_k = 50$ por decil). Como são estimados parâmetros no modelo original, simulações de Hosmer & Lemeshow mostram que $gl = g - 2$ captura a perda aproximada de 2 gl ao comparar frequências binomiais observadas e preditas sem reestimar sobre os grupos.

                
**Item c):** Aplique $\frac{(O_{1,10} - E_{1,10})^2}{E_{1,10}} + \frac{(O_{0,10} - E_{0,10})^2}{E_{0,10}}$ para o Decil 10 ($O_1=14, E_1=13,446, O_0=36, E_0=36,554$) e depois verifique a equivalência algébrica $\frac{(O_{1,10} - E_{1,10})^2}{E_{1,10}(1 - E_{1,10}/50)}$!

                
**Item d):** Pense em: (1) dependência arbitrária do corte $g$ (ex.: $g=8$ vs. $g=10$ vs. $g=12$ pode inverter o p-valor em casos limítrofes!); e (2) efeito do tamanho amostral ($n$ pequeno $\to$ baixo poder; $n$ gigantesco $\to$ rejeita até discrepâncias irrelevantes; além de $E_k < 5$ nas caudas de risco extremo!).

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Condição de Assintótica de McCullagh & Nelder:** A distribuição $\chi^2_{N-p}$ da deviance residual exige que o tamanho de *cada subpopulação* $m_i$ cresça ($m_i \to \infty$ com $N$ fixo), *não* apenas que $n$ total cresça com $m_i = 1$! Quando há covariáveis contínuas (`Idade`, `PAM`), cada pessoa forma sua própria subpopulação com $m_i = 1$, violando a assintótica da $\chi^2_{n-p}$.

                
**Inversão clássica de $H_0$:** Em testes de bondade de ajuste (`Hosmer-Lemeshow`), $H_0$ é sempre “o modelo está bem calibrado / bem ajustado”. Logo, **queremos $p > 0,05$**! Um $p = 0,7052$ é um ótimo resultado indicando excelente concordância entre óbitos observados e preditos em todas as faixas de risco!

                
**Identidade algébrica útil no item c:** Como $O_0 = n_k - O_1$ e $E_0 = n_k - E_1$, temos $O_0 - E_0 = -(O_1 - E_1)$! Tirando o fator comum $(O_1 - E_1)^2$, soma $\frac{1}{E_1} + \frac{1}{n_k - E_1} = \frac{n_k}{E_1(n_k - E_1)} = \frac{1}{E_1(1 - E_1/n_k)}$.

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Refutação do Uso da Deviance Residual para $m_i = 1$:**

                  
O colega cometeu um erro clássico. Quando $m_i = 1$ (dados binários individuais com covariáveis contínuas como `Idade` e `PAM`, de modo que quase toda observação tem um perfil único), a deviance residual vale $D = -2\sum_{i=1}^n [y_i\ln(\hat{\pi}_i) + (1-y_i)\ln(1-\hat{\pi}_i)]$. Expandindo e aplicando as equações normais de máxima verossimilhança ($\sum y_i \eta_i = \sum \hat{\pi}_i \eta_i$), constata-se que $D$ reduz-se a $-2\sum [\hat{\pi}_i \ln\hat{\pi}_i + (1-\hat{\pi}_i)\ln(1-\hat{\pi}_i)]$: uma função pura das probabilidades ajustadas $\hat{\pi}_i$, sem informação independente sobre a discrepância dos $y_i$! Além disso, a assintótica $\chi^2_{n-p}$ exige $m_i \to \infty$ por perfil (células densas), o que nunca ocorre com $m_i=1$. Portanto, $D = 237.72$ vs. $496$ gl **não** serve como teste de bondade de ajuste (serve apenas para calcular $\Delta D$ entre modelos encaixados!).

                

                

                  
**b) Construção da Estatística $\hat{C}_{\text{HL}}$ e $gl = g - 2$:**

                  
Como $m_i=1$ não forma grupos naturais, Hosmer & Lemeshow propuseram criar agrupamentos artificiais ordenando os $n$ indivíduos pelas probabilidades preditas $\hat{\pi}_i$ e particionando-os em $g = 10$ decis de tamanho aproximado $n_k = n/10 = 50$. Em cada decil $k = 1, \dots, g$, comparam-se as contagens observadas de eventos ($O_{1k} = \sum_{i \in k} y_i$) e não-eventos ($O_{0k} = n_k - O_{1k}$) com as esperadas ($E_{1k} = \sum_{i \in k} \hat{\pi}_i$ e $E_{0k} = n_k - E_{1k}$):

                  
$$\hat{C}_{\text{HL}} = \sum_{k=1}^g \left[ \frac{(O_{1k} - E_{1k})^2}{E_{1k}} + \frac{(O_{0k} - E_{0k})^2}{E_{0k}} \right] = \sum_{k=1}^g \frac{(O_{1k} - E_{1k})^2}{n_k \, \bar{\pi}_k (1 - \bar{\pi}_k)} \sim \chi^2_{g - 2}$$

                  
Por extensivos estudos de simulação com modelos ajustados corretamente nos $n$ pontos individuais (e apenas $1$ restrição global $\sum O_{1k} = \sum E_{1k}$ além da ordenação induzida pelas estimativas $\hat{\boldsymbol{\beta}}$), mostrou-se que a distribuição nula aproxima-se bem por $\chi^2$ com $gl = g - 2 = 10 - 2 = \mathbf{8}$ graus de liberdade.

                

                

                  
**c) Hipóteses, Decisão e Cálculo Manual no Decil 10:**

                  

                    
**Hipóteses:** $H_0$: O modelo logístico está corretamente especificado e calibrado (as probabilidades preditas concordam com os riscos observados nos 10 decis) vs. $H_1$: Há má calibração / falta de ajuste.

                    
**Contribuição do Decil 10 ($n_{10} = 50$, $O_1 = 14$, $E_1 = 13,446$, $O_0 = 36$, $E_0 = 36,554$):**

                    Pela soma das duas parcelas: $\dfrac{(14 - 13,446)^2}{13,446} + \dfrac{(36 - 36,554)^2}{36,554} = \dfrac{0,3069}{13,446} + \dfrac{0,3069}{36,554} = 0,0228 + 0,0084 = \mathbf{0,0312}$.

                    Pela fórmula compacta ($\bar{\pi}_{10} = 13,446/50 = 0,26892$): $\dfrac{(14 - 13,446)^2}{13,446 \times (1 - 0,26892)} = \dfrac{0,3069}{9,830} = \mathbf{0,0312}$.

                  

                  

                    **📝 Como redigir na prova:** “Em dados binários individuais ($m_i=1$) com covariáveis contínuas, a deviance residual não segue $\chi^2_{n-p}$, recorrendo-se ao teste de Hosmer-Lemeshow. Sob $H_0$ de bom ajuste/calibração, obteve-se $\hat{C}_{\text{HL}} = 5,4807$ com $gl = 8$ e $p = 0,7052 > 0,05$. Logo, não rejeitamos $H_0$: os óbitos observados acompanham fielmente os óbitos esperados do 1º ao 10º decil de risco (por exemplo, $14$ observados contra $13,45$ esperados no estrato mais grave), evidenciando boa calibração do modelo.”
                  

                

                

                  
**d) Duas Limitações Clássicas do Teste de Hosmer-Lemeshow:**

                  

                    
**Dependência arbitrária do agrupamento ($g$) e frequências esperadas baixas:** Mudar o número de grupos (por ex., $g = 8$ vs. $g = 10$ vs. $g = 15$) ou o método de desempate pode alterar o p-valor; além disso, note nos Decis 1 a 6 que vários $E_1 < 5$, o que degrada a aproximação $\chi^2$ em amostras moderadas.

                    
**Dependência extrema do tamanho amostral ($n$) e cegueira direcional:** Possui baixo poder para detectar erros sutis de não-linearidade quando $n$ é pequeno/moderado e, inversamente, rejeita modelos clinicamente excelentes por desvios irrisórios quando $n$ é muito grande, além de testar apenas calibração sem indicar *qual* covariável falhou nem medir discriminação (papel da Curva ROC).

</details>

---

## Questão 15 — Curva ROC, AUC, Ponto de Corte (0,50 vs. Prevalência), Métricas de Classificação 2×2 e Efeito Hauck-Donner em Eventos Raros
**Bloco:** Bloco IV • Predição, HL & ROC | **Inspiração:** Inspirada na Lista 5, Desafio 3 e Slides de ROC/Separação • Profa. Rosemeire

**Parte I (Triagem Diagnóstica):** Em um estudo de validação com $n = 400$ pacientes (94 doentes [$Y=1$] e 306 sadios [$Y=0$], prevalência de $23,5\%$), ajustou-se um escore logístico preditivo e avaliaram-se dois pontos de corte de probabilidade predita: o corte padrão $\pi_0 = 0,50$ e um corte ajustado à prevalência $\pi_0 = 0,20$.
**Parte II (Subestudo de Eventos Raros):** Em outra coorte paralela com $n = 300$ pacientes e apenas **$6$ óbitos**, todos os $6$ óbitos ocorreram dentro do grupo `Risco_ext = 1` ($80$ expostos e $0$ óbitos nos $220$ não-expostos), produzindo a saída anômala abaixo.

```r
=== PARTE I: Matrizes de Confusao (n = 400: 94 doentes [VP+FN=94] e 306 sadios [VN+FP=306]) ===
[Corte pi_0 = 0,50]                     [Corte pi_0 = 0,20]
             Observado                               Observado
Predito     0    1                      Predito     0    1
      0   291   58                            0   201   20
      1    15   36                            1   105   74

AUC estimada = 0.800  (IC 95%: 0.743 - 0.857)

=== PARTE II: Subestudo de Eventos Raros (n = 300, 6 eventos) ===
Warning message: glm.fit: fitted probabilities numerically 0 or 1 occurred

Call:
glm(formula = Y ~ Risco_ext + X_cov, family = binomial(link = "logit"), 
    data = df15_rare)

Coefficients:
             Estimate Std. Error z value Pr(>|z|)  
(Intercept)  -21.7761  1906.0974  -0.011   0.9909  
Risco_ext     19.1258  1906.0974   0.010   0.9920  
X_cov         -0.8053     0.4894  -1.646   0.0999 .
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 58.823  on 299  degrees of freedom
Residual deviance: 39.881  on 297  degrees of freedom
AIC: 45.881

Number of Fisher Scoring iterations: 20


--- anova(m15_rare, test = "Chisq") ---
Analysis of Deviance Table

Model: binomial, link: logit

Response: Y

Terms added sequentially (first to last)


          Df Deviance Resid. Df Resid. Dev  Pr(>Chi)    
NULL                        299     58.823              
Risco_ext  1  16.2020       298     42.622 5.694e-05 ***
X_cov      1   2.7401       297     39.881   0.09786 .  
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

### Subitens Discursivos
- **a)** A partir das duas matrizes de confusão da **Parte I**, calcule todas as métricas diagnósticas para os cortes $\pi_0 = 0,50$ e $\pi_0 = 0,20$: **Sensibilidade**, **Especificidade**, **Valor Preditivo Positivo (VPP)**, **Valor Preditivo Negativo (VPN)** e **Acurácia Global**.
- **b)** Analise o *trade-off* observado entre os dois pontos de corte. Por que adotar cegamente $\pi_0 = 0,50$ em dados com prevalência moderada-baixa derrubou a Sensibilidade para apenas 38,3% apesar da acurácia alta (81,8%), e qual estratégia é usada para definir o ponto de corte ótimo (como o Índice de Youden)?
- **c)** Defina os eixos da **Curva ROC**, dê a interpretação probabilística exata de $\text{AUC} = 0.800$ e classifique o poder discriminatório deste escore segundo a escala de Hosmer & Lemeshow.
- **d)** Na **Parte II (Subestudo de Eventos Raros)**, explique o paradoxo grotesco ocorrido com `Risco_ext`: apesar de TODOS os $6$ óbitos estarem em `Risco_ext = 1`, a tabela de Wald reporta $\hat{\beta}_1 = 19,13$, $\text{EP} = 1906,10$ e $p = 0,9920$ (não significativo!), enquanto a ANODEV acusa $\Delta D = 16,202$ ($p = 5,69 \times 10^{-5}$)! Identifique o nome desses dois fenômenos (separação e **efeito Hauck-Donner**), indique qual teste deve prevalecer e cite o método de correção de viés em eventos raros (Firth).

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Identifique nas colunas (Observado) e linhas (Predito): `Pred=1, Obs=1` é $VP$; `Pred=0, Obs=0` é $VN$; `Pred=1, Obs=0` é $FP$; e `Pred=0, Obs=1` é $FN$. Note que o total de doentes na coluna $1$ é $VP+FN = 94$ e de sadios na coluna $0$ é $VN+FP = 306$.

                
**Item b):** Compare $\text{Sens}_{0,50} = 38.3\%$ contra $\text{Sens}_{0,20} = 78.7\%$, e o preço pago na $\text{Espec}_{0,50} = 95.1\%$ caindo para $\text{Espec}_{0,20} = 65.7\%$. Note também o impacto no VPP quando os falsos positivos sobem de 15 para 105!

                
**Item c):** A interpretação clássica de AUC: sorteie um par aleatório formado por $1$ paciente doente e $1$ sadio — AUC é a probabilidade de o modelo atribuir $\hat{\pi}_{\text{doente}} > \hat{\pi}_{\text{sadio}}$! E na escala de Hosmer-Lemeshow, note o valor $\text{AUC} = 0.800$ (no limite superior de $[0,7; 0,8]$ entre aceitável e excelente!).

                
**Item d):** Como nenhum óbito ocorreu quando `Risco_ext = 0` (célula vazia $=$ zero na tabela $2\times 2$!), a verossimilhança cresce monotonicamente em direção a $\beta_1 \to +\infty$ (**quase-separação completa**). Isso faz o algoritmo parar em 20 iterações com erro-padrão explodindo muito mais rápido que $\hat{\beta}_1$ (**efeito Hauck-Donner**), zerando a estatística de Wald $z = 19,1258 / 1906,0974 = 0,010$!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Memorize os denominadores da matriz 2×2:**
                  
• **Sensibilidade** = $VP / (VP + FN)$ (lê-se na *coluna dos doentes*!).
                  
• **Especificidade** = $VN / (VN + FP)$ (lê-se na *coluna dos sadios*!).
                  
• **VPP** = $VP / (VP + FP)$ (lê-se na *linha dos classificados como positivos* — depende fortemente da prevalência!).
                  
• **VPN** = $VN / (VN + FN)$ (lê-se na *linha dos classificados como negativos*!).

                
**Regra da Prova para Separação & Hauck-Donner:** Sempre que surgir `fitted probabilities numerically 0 or 1 occurred` com $\hat{\beta} \approx \pm 19$ e $\text{EP} > 1000$, o teste $z$ de **Wald está completamente corrompido**! Confie exclusivamente no Teste da Razão de Verossimilhanças ($\Delta D$ na ANODEV / `drop1`, aqui $p = 5,69 \times 10^{-5}$) e recomende a **regressão logística penalizada de Firth** (`logistf`), que garante estimativas finitas e reduz o viés de pequena amostra / eventos raros.

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Métricas das Matrizes de Confusão ($n = 400$, 94 doentes e 306 sadios):**

                  

                    

                      
Corte padrão $\pi_0 = 0,50$ ($VP=36, FN=58, VN=291, FP=15$):

                      

                        
• Sensibilidade = $36/94 = \mathbf{0.3830}$ (38,3%)

                        
• Especificidade = $291/306 = \mathbf{0.9510}$ (95,1%)

                        
• VPP = $36/(36+15) = 36/51 = \mathbf{0.7059}$ (70,6%)

                        
• VPN = $291/(291+58) = 291/349 = \mathbf{0.8338}$ (83,4%)

                        
• Acurácia = $(36+291)/400 = \mathbf{0.8175}$ (81,8%)

                      

                    

                    

                      
Corte ajustado $\pi_0 = 0,20$ ($VP=74, FN=20, VN=201, FP=105$):

                      

                        
• Sensibilidade = $74/94 = \mathbf{0.7872}$ (78,7%)

                        
• Especificidade = $201/306 = \mathbf{0.6569}$ (65,7%)

                        
• VPP = $74/(74+105) = 74/179 = \mathbf{0.4134}$ (41,3%)

                        
• VPN = $201/(201+20) = 201/221 = \mathbf{0.9095}$ (91,0%)

                        
• Acurácia = $(74+201)/400 = \mathbf{0.6875}$ (68,8%)

                      

                    

                  

                

                

                  
**b) Trade-off, Armadilha do Corte $0,50$ e Critério de Youden:**

                  
Como a prevalência da doença é moderada-baixa ($23,5\%$), muitas observações doentes têm probabilidades ajustadas entre $0,20$ e $0,49$. Adotar $\pi_0 = 0,50$ ignora esses casos, fazendo a Sensibilidade despencar para apenas **38,3%** ($58$ falsos negativos perdidos!), embora a acurácia aparente seja enganosamente alta (81,8%) por causa dos numerosos sadios corretamente classificados. Ao baixar o corte para próximo à prevalência ($\pi_0 = 0,20$), a Sensibilidade salta para **78,7%** ($VPN = 91,0\%$), excelente para triagem clínica, ao custo previsível de queda na Especificidade (65,7%) e no VPP (41,3%). O corte ótimo equilibrado costuma maximizar o **Índice de Youden** $J = \max_c(\text{Sens}(c) + \text{Espec}(c) - 1)$ (ponto mais distante da diagonal na curva ROC).

                

                

                  
**c) Eixos da Curva ROC e Interpretação de $\text{AUC} = 0.800$:**

                  
A Curva ROC plota na ordenada (eixo Y) a **Sensibilidade (TVP)** contra a abscissa (eixo X) **$1 - \text{Especificidade}$ (TFP)** à medida que o corte varia de $1$ a $0$. O valor $\text{AUC} = 0.800$ significa que, ao sortear aleatoriamente um paciente doente e um sadio, há **80,0% de probabilidade** de o modelo logístico atribuir um escore de risco mais alto ao paciente doente do que ao sadio. Pela classificação clássica de Hosmer & Lemeshow ($0,70 \le \text{AUC} < 0,80$ aceitável; $\ge 0,80$ excelente), o poder discriminatório alcança a fronteira de **bom / excelente** (note que o IC 95% $[0,743;\,0,857]$ supera amplamente a linha do acaso $0,50$).

                

                

                  
**d) Separação, Efeito Hauck-Donner e Regressão Penalizada de Firth:**

                  

                    **📝 Como redigir na prova:** “Na Parte II, como nenhum dos $220$ pacientes com `Risco_ext = 0` veio a óbito ($0$ casos numa célula da tabela $2\times 2$), ocorre **quase-separação completa**: o estimador de máxima verossimilhança de $\beta_1$ diverge para $+\infty$, fazendo o algoritmo de Fisher parar precocemente após 20 iterações com estimativa inflada ($\hat{\beta}_1 = 19,13$) e erro-padrão explosivo ($\text{EP} = 1.906,10$). Como o EP cresce muito mais rápido que $\hat{\beta}_1$, instala-se o **efeito Hauck-Donner**, comprimindo artificialmente a estatística de Wald para $z = 0,010$ ($p = 0,992$) e gerando um falso negativo absurdo! Na presença de separação, **deve-se descartar o teste de Wald** e adotar o Teste da Razão de Verossimilhanças (ANODEV: $\Delta D = 16,202$; $p = 5,69\times 10^{-5}$, altamente significativo). Para obter estimativas pontuais e ICs finitos de $\text{OR}$ em amostras pequenas ou com eventos raros, recomenda-se ajustar a **regressão logística com verossimilhança penalizada de Firth**.”

</details>

---

## Questão 16 — HorseCrab (Lista 5 — Itens a, b): Análise Exploratória, Multicolinearidade Largura × Peso e Seleção do Modelo Principal via ANODEV & AIC
**Bloco:** Bloco V • Especial Lista 5 (Logística HorseCrab a–j) | **Inspiração:** Lista 5 Completa (HorseCrab, n = 173) — Itens a) e b) + Seleção ANODEV • Profa. Rosemeire

No clássico estudo com $n = 173$ fêmeas de caranguejo-ferradura (*HorseCrab*), definiu-se a variável resposta binária $Y = 1$ se a fêmea atrai pelo menos um macho satélite ($\text{satélite} \ge 1$, observado em $111$ fêmeas, $64,16\%$) e $Y = 0$ caso contrário ($62$ fêmeas, $35,84\%$). Ajustou-se inicialmente o modelo logístico múltiplo completo (tomando `Clara` e `AmbasBoas` como referências basais), seguido da análise sequencial de deviance para seleção de variáveis.

```r
=== TABELAS 1 & 2: Analise Exploratoria (n = 173) ===
Variavel / Grupo      | Media   |   DP   | Mediana |   IQR  |   Min  |   Max
Largura (cm) Y=0 (62) |   25,17 |   1,67 |   25,45 |   2,35 |  21,00 |  28,70
Largura (cm) Y=1 (111)|   26,93 |   2,07 |   27,00 |   2,90 |  22,50 |  33,50
Peso (g)     Y=0 (62) | 2139,11 | 448,39 | 2125,00 | 550,00 | 1200,00 | 3200,00
Peso (g)     Y=1 (111)| 2603,68 | 575,39 | 2600,00 | 812,50 | 1475,00 | 5200,00

Categoria              | Total | Y=0 | Y=1 | Prop(Y=1)
Cor: Clara (5 - Ref)   |    22 |  15 |   7 |  31,82%
Cor: Media (4)         |    44 |  18 |  26 |  59,09%
Cor: Escura-Fraca (3)  |    95 |  26 |  69 |  72,63%
Cor: Escura-Forte (2)  |    12 |   3 |   9 |  75,00%
Cond: AmbasBoas (1-Ref)|    37 |  11 |  26 |  70,27%
Cond: UmaQuebrada (2)  |    15 |   8 |   7 |  46,67%
Cond: AmbasQuebradas(3)|   121 |  43 |  78 |  64,46%

cor.test(largura, peso): r = 0,8869 (p < 2,2e-16)

=== TABELA 3: Modelo Logistico Completo (y ~ cor + cond + largura + peso) ===
Deviance Nula: 225,76 em 172 gl | Deviance Residual: 185,20 em 165 gl | AIC: 201,20
(Intercept)           -9.67368   3.86456  -2.503  p=0.01231  OR=0.0001  IC95%=[0.000; 0.122]
corMedia               1.11980   0.59329   1.887  p=0.0591   OR=3.0642  IC95%=[0.958; 9.802]
corEscura-Fraca        1.50576   0.56667   2.657  p=0.007879  OR=4.5076  IC95%=[1.485; 13.687]
corEscura-Forte        1.60867   0.93553   1.720  p=0.08552  OR=4.9961  IC95%=[0.799; 31.258]
condUmaQuebrada       -0.09598   0.70337  -0.136  p=0.8915   OR=0.9085  IC95%=[0.229; 3.606]
condAmbasQuebradas     0.40029   0.50270   0.796  p=0.4259   OR=1.4923  IC95%=[0.557; 3.997]
largura                0.26313   0.19530   1.347  p=0.1779   OR=1.3010  IC95%=[0.887; 1.908]
peso                   0.00083   0.00070   1.173  p=0.2407   OR=1.0008  IC95%=[0.999; 1.002]

=== ANODEV Sequencial & Selecao do Modelo Principal ===
Analysis of Deviance Table

Model: binomial, link: logit

Response: y

Terms added sequentially (first to last)


        Df Deviance Resid. Df Resid. Dev  Pr(>Chi)    
NULL                      172     225.76              
cor      3  13.6977       169     212.06  0.003347 ** 
largura  1  24.6038       168     187.46 7.041e-07 ***
cond     2   0.8451       166     186.61  0.655362    
peso     1   1.4099       165     185.20  0.235073    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

=== Modelo Principal Adotado (y ~ cor + largura) | AIC = 197,46 ===

Call:
glm(formula = y ~ cor + largura, family = binomial, data = df16)

Coefficients:
                Estimate Std. Error z value Pr(>|z|)    
(Intercept)     -12.7151     2.7617  -4.604 4.14e-06 ***
corMedia          1.1061     0.5921   1.868   0.0617 .  
corEscura-Fraca   1.4023     0.5484   2.557   0.0106 *  
corEscura-Forte   1.3299     0.8525   1.560   0.1188    
largura           0.4680     0.1055   4.434 9.26e-06 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 225.76  on 172  degrees of freedom
Residual deviance: 187.46  on 168  degrees of freedom
AIC: 197.46

Number of Fisher Scoring iterations: 4

```

### Subitens Discursivos
- **a)** **(Item a — Exploratória)** Analise as medidas descritivas de `largura` e `peso` por $Y$, bem como as proporções por `cor` e `condição da carapaça`. Qual o impacto da correlação $r = 0,8869$ ($p < 2,2 \times 10^{-16}$) e quais variáveis aparentam ser realmente discriminatórias (há sinais de não-linearidade em "U"?)?
- **b)** **(Item b — Modelo Completo & Multicolinearidade)** No modelo completo (Tabela 3), apesar de `largura` e `peso` diferenciarem fortemente os grupos $Y=1$ vs. $Y=0$ na análise exploratória, **nenhum dos dois** atinge significância individual de Wald ($p = 0,1779$ e $p = 0,2407$). Explique rigorosamente por que esse aparente paradoxo ocorre, e interprete as razões de chances ($\text{OR} = e^{\hat{\beta}}$) e ICs $95\%$ para `cor` (`Media`, `Escura-Fraca` e `Escura-Forte` vs. `Clara`).
- **c)** **Seleção por ANODEV e AIC:** Usando a ANODEV sequencial e o $\text{AIC}$, justifique numericamente a exclusão de `peso` ($\Delta D = 1,41;\, p = 0,2351$) e de `condição` ($\Delta D = 0,85;\, p = 0,6554$), estabelecendo o modelo parcimonioso `y ~ cor + largura`.
- **d)** Compare a estimativa, o erro-padrão, a significância e o $\text{OR}$ da `largura` no **modelo completo** ($\hat{\beta} = 0,2631$, $p = 0,1779$) versus no **modelo principal selecionado** ($\hat{\beta} = 0,4680$, $z = 4,434$, $p = 9,26 \times 10^{-6}$). Redija a interpretação biológica final para $+1\text{ cm}$ de largura.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Compare médias e medianas: fêmeas com satélites são maiores ($26,93$ vs. $25,17$ cm) e mais pesadas ($2603,68$ vs. $2139,11$ g). Observe o gradiente monotônico crescente de escurecimento em `cor` ($31,82\% \to 59,09\% \to 72,63\% \to 75,00\%$), em contraste com a oscilação errática de `condição` ($70,27\% \to 46,67\% \to 64,46\%$).

                
**Item b):** Quando duas covariáveis contêm praticamente a mesma informação biológica ($r = 0,8869$), seus erros-padrão assintóticos na matriz $(X^TWX)^{-1}$ ficam **inflacionados**! Por isso Wald individual ($\hat{\beta}/\text{EP}$) perde potência ao testar uma mantendo a outra fixa.

                
**Item c):** Na ANODEV sequencial após incluir `cor` ($\Delta D = 13,70;\, p=0,0033$) e `largura` ($\Delta D = 24,60;\, p=7,04\times 10^{-7}$), adicionar `cond` ($+2$ gl) reduz apenas $0,85$ na deviance ($p = 0,6554$) e adicionar `peso` ($+1$ gl) reduz apenas $1,41$ ($p = 0,2351$). Note também que o AIC cai de $201,20$ para $197,46$!

                
**Item d):** Verifique o cálculo do $\text{OR}$ da largura no modelo final: $\text{OR} = e^{0,4680} = 1,5968 \approx 1,597$. Sem a concorrência do peso, o erro-padrão de `largura` cai de $0,1953$ para $0,1055$ (redução de $46\%$)!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Dica de Ouro — Multicolinearidade vs. Concorrência:** Nunca conclua que uma variável sem significância no modelo completo é irrelevante sem antes checar correlações entre preditoras! O teste de Wald condiciona na permanência das demais covariáveis no modelo: “qual o efeito adicional de largura *mantendo o peso fixo*?”. Como caranguejos mais largos são quase inevitavelmente mais pesados ($r=0,89$), há pouca variação independente residual.

                
**Como comparar modelos por AIC e Deviance:** Ao remover $3$ parâmetros irrelevantes (`cond` + `peso`), a deviance residual sobe muito pouco ($185,20 \to 187,46$, apenas $+2,26$ em $3\text{ gl}$, $p > 0,50$), enquanto o corte na penalização $2p$ ($-6,0$) garante um AIC claramente menor ($197,46 < 201,20$).

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Síntese Exploratória e Associação com Satélites ($n = 173$):**

                  
• **Largura e Peso:** Fêmeas com satélites ($Y=1$, $64,16\%$) têm carapaças mais largas (média $26,93$ cm, $\text{DP}=2,07$ vs. $25,17$ cm, $\text{DP}=1,67$) e são substancialmente mais pesadas ($2603,68$ g vs. $2139,11$ g). A chance cresce monotonicamente com o porte físico (sem padrão em "U"). Contudo, a correlação de Pearson de $r = 0,8869$ ($p < 2,2 \times 10^{-16}$) antecipa forte redundância (multicolinearidade) se ambas forem mantidas juntas.

                  • **Cor vs. Condição:** A pigmentação exibe nítido gradiente biológico monotônico: carapaças `Claras` têm apenas $31,82\%$ de atração ($7/22$), saltando para $59,09\%$ (`Média`), $72,63\%$ (`Escura-Fraca`) e $75,00\%$ (`Escura-Forte`). Já a `condição` oscila sem padrão monotônico ($70,27\% \to 46,67\% \to 64,46\%$). Assim, `largura`, `peso` e `cor` são relevantes na etapa exploratória, ao passo que `condição` parece irrelevante.

                

                

                  
**b) Modelo Completo — Multicolinearidade e Interpretação de $\text{OR}$ da Cor:**

                  
No modelo completo ($D_p = 185,20$ em $165\text{ gl}$; $D_0 = 225,76$ em $172\text{ gl}$; $\text{AIC}=201,20$), apenas `cor: Escura-Fraca` atinge significância individual a $5\%$ ($p = 0,0079$). Nem `largura` ($p = 0,1779$; $\text{OR}=1,301$, IC $95\%: [0,887;\,1,908]$) nem `peso` ($p = 0,2407$) atingem $5\%$. Isso ocorre diretamente pela **multicolinearidade intensa** ($r = 0,8869$), que inflaciona os erros-padrão da matriz de variância-covariância $(X^T W X)^{-1}$, alargando os intervalos de confiança a ponto de incluírem $1$. Para a cor (basal `Clara`):

                  • `Média`: $\text{OR} = e^{1,1198} = \mathbf{3,064}$ (IC $95\%: [0,958;\,9,802]$, $p=0,0591$ — chance $3,06\times$ maior);

                  • `Escura-Fraca`: $\text{OR} = e^{1,5058} = \mathbf{4,508}$ (IC $95\%: [1,485;\,13,686]$, $p=0,0079^*$ — chance $4,51\times$ maior e significativa);

                  • `Escura-Forte`: $\text{OR} = e^{1,6087} = \mathbf{4,996}$ (IC $95\%: [0,799;\,31,258]$, $p=0,0855$ — chance $5,00\times$ maior, mas imprecisa pois $n=12$).

                

                

                  
**c) e d) Seleção ANODEV/AIC e Modelo Principal Final ($y \sim \text{cor} + \text{largura}$):**

                  

                    **📝 Como redigir na prova:** “Na Análise de Deviance sequencial, após a entrada de `cor` ($\Delta D = 13,70;\, gl=3;\, p=0,0033$) e `largura` ($\Delta D = 24,60;\, gl=1;\, p=7,04\times 10^{-7}$), a adição de `condição` ($\Delta D = 0,85;\, gl=2;\, p=0,6554$) e de `peso` ($\Delta D = 1,41;\, gl=1;\, p=0,2351$) não traz ganho explicativo significativo. Pelo critério de parcimônia e menor AIC ($197,46$ contra $201,20$ do completo), adota-se o modelo principal:

                    $$\widehat{\text{logit}}(\pi) = -12,7151 + 1,1061\,\text{cor}_{\text{Media}} + 1,4023\,\text{cor}_{\text{EFra}} + 1,3299\,\text{cor}_{\text{EFor}} + 0,4680\,\text{largura}$$

                    Removida a colinearidade com o peso, o erro-padrão de `largura` reduz-se de $0,1953$ para $0,1055$, tornando-a altamente significativa ($z = 4,434;\, p = 9,26\times 10^{-6}$). Mantendo a pigmentação constante, cada centímetro adicional de carapaça multiplica a chance (*odds*) de a fêmea atrair satélites por $\text{OR} = e^{0,4680} = \mathbf{1,597}$ (IC $95\%: [1,299;\,1,964]$), ou seja, um aumento de **$+59,7\%$ na chance por centímetro adicional**.”

</details>

---

## Questão 17 — HorseCrab (Lista 5 — Item c): Investigação da Linearidade entre Largura e Logito por Dois Métodos (TRV Quadrático e Logitos Empíricos de Haldane)
**Bloco:** Bloco V • Especial Lista 5 (Logística HorseCrab a–j) | **Inspiração:** Lista 5 Completa (HorseCrab, n = 173) — Item c) + Tabela 4 + Figura 2 • Profa. Rosemeire

Uma premissa estrutural fundamental do modelo logístico `y ~ cor + largura` é que a covariável contínua `largura` se relacione **linearmente** com o logito $\eta = \ln(\pi/(1-\pi))$. Para auditar essa suposição no conjunto *HorseCrab* ($n = 173$), aplicaram-se dois métodos complementares: (1) expansão polinomial com termo quadrático `I(largura^2)` avaliada por TRV, e (2) cálculo dos logitos empíricos corrigidos por Haldane em $8$ decis homogêneos de largura.

```r
=== METODO 1: TRV — Modelo Linear vs. Modelo Quadratico ===
Modelo Linear    (y ~ cor + largura):                D = 187,4570  em 168 gl
Modelo Quadratico(y ~ cor + largura + I(largura^2)): D = 186,6062  em 167 gl

Analysis of Deviance Table

Model 1: y ~ cor + largura
Model 2: y ~ cor + largura + I(largura^2)
  Resid. Df Resid. Dev Df Deviance Pr(>Chi)
1       168     187.46                     
2       167     186.61  1  0.85085   0.3563

=== METODO 2: TABELA 4 — Classes de Largura (8 Decis) e Logitos Empiricos (Haldane) ===
Formula: logit*(p_k) = ln((y_k + 0,5) / (m_k - y_k + 0,5))
Faixa Largura (cm) | x_bar  | Total(m) | Sucessos(y) | Fracassos | Logito Haldane
[21,23.9]          |  23,05 |       22 |           6 |        16 |        -0.9316
(23.9,24.9]        |  24,48 |       24 |          11 |        13 |        -0.1603
(24.9,25.6]        |  25,25 |       19 |          13 |         6 |        +0.7309
(25.6,26.1]        |  25,86 |       22 |          13 |         9 |        +0.3514
(26.1,26.9]        |  26,45 |       21 |          12 |         9 |        +0.2744
(26.9,27.7]        |  27,32 |       22 |          18 |         4 |        +1.4137
(27.7,28.7]        |  28,24 |       22 |          17 |         5 |        +1.1575
(28.7,33.5]        |  29,93 |       21 |          21 |         0 |        +3.7612
Reta OLS ajustada aos 8 pontos (Figura 2): inclinacao = +0,5918 | R^2 = 0,8498
```

### Subitens Discursivos
- **a)** **(Método 1 — Teste da Razão de Verossimilhanças)** Formule as hipóteses nula ($H_0$) e alternativa ($H_1$), calcule manualmente a estatística $\Delta D^*$ e sua variação nos graus de liberdade ($\Delta gl$) a partir das deviances acima, e interprete o valor-$p = 0,3563$.
- **b)** **(Método 2 — Logitos Empíricos & Correção de Haldane)** Usando a equação $\text{logit}^*(p_k) = \ln\!\left(\frac{y_k + 0,5}{m_k - y_k + 0,5}\right)$, confira os cálculos do $1^\circ$ decil ($[21,0;\,23,9]$: $m_1=22, y_1=6$) e do $8^\circ$ decil ($(28,7;\,33,5]$: $m_8=21, y_8=21$), explicando **por que** a adição de $+0,5$ é indispensável no último intervalo.
- **c)** Analise criticamente a **Figura 2** (Logito Empírico vs. Largura Média da Classe): o comportamento empírico sustenta a forma retilínea assumida pelo logito ou denuncia necessidade de termos quadráticos/splines?
- **d)** Em conjunto, o que os dois métodos determinam quanto à especificação final de `largura` no modelo principal?

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Compare dois modelos encaixados com $\phi = 1$ conhecido: $\Delta D^* = D_{\text{linear}} - D_{\text{quad}} = 187,457 - 186,606 = 0,8508$ com $\Delta gl = 168 - 167 = 1$, comparado contra $\chi^2_{1;\,0,95} = 3,841$.

                
**Item b):** No último decil $(28,7;\,33,5]$, todas as $21$ fêmeas possuem satélites ($y_8 = 21$ e fracassos $= 0$!). O logito ingênuo $\ln(21/0)$ explodiria para $+\infty$, tornando impossível plotar o ponto!

                
**Item c):** Observe na Figura 2 os $8$ pontos em relação à linha vermelha de tendência: há alinhamento crescente contínuo ($R^2 = 0,850$, inclinação $+0,592$), sem formato de parábola côncava ou convexa sistemática.

                
**Item d):** Ambos convergem para a mesma conclusão: não há violação da linearidade no logito. Manter `largura` linear evita sobreajuste e perda desnecessária de $1$ grau de liberdade.

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Atenção à Pegadinha de Prova:** Linearidade no modelo logístico significa relação retilínea entre $X$ e o *logito* $\eta = \ln(\pi/(1-\pi))$ — e NÃO linearidade entre $X$ e a probabilidade $\pi$ (que sempre será uma curva sigmoidal em "S")!

                
**Correção de Haldane-Anscombe:** Sempre que uma faixa ou célula tiver $0$ sucessos ($y_k = 0$) ou $0$ fracassos ($m_k - y_k = 0$), soma-se $0,5$ tanto ao numerador quanto ao denominador para estabilizar o logito empírico em valores finitos ($\ln(21,5/0,5) = \ln(43) = +3,7612$).

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Método 1 — Teste da Razão de Verossimilhanças (TRV):**

                  
Testa-se $H_0: \beta_{\text{quad}} = 0$ (efeito linear suficiente no logito) contra $H_1: \beta_{\text{quad}} \ne 0$ (curvatura quadrática necessária). Como a Binomial tem $\phi = 1$ fixo:

                  $$\Delta D^* = D_{\text{lin}} - D_{\text{quad}} = 187,4570 - 186,6062 = \mathbf{0,8508} \quad (\Delta gl = 168 - 167 = 1)$$

                  Como $0,8508 \ll \chi^2_{1;\,0,95} = 3,841$ e o valor-$p = P(\chi^2_1 \ge 0,8508) = \mathbf{0,3563} \gg 0,05$, **não se rejeita $H_0$**: o termo quadrático `I(largura^2)` não adiciona poder explicativo estatisticamente relevante.

                

                

                  
**b) Método 2 — Conferência Numérica da Tabela 4 e Função de $+0,5$:**

                  
• **$1^\circ$ Decil $[21,0;\,23,9]$ ($y_1=6$, fracassos $= 16$):**

                  $\text{logit}^*(p_1) = \ln\!\left(\frac{6 + 0,5}{16 + 0,5}\right) = \ln\!\left(\frac{6,5}{16,5}\right) = \ln(0,3939) = \mathbf{-0,9316}$.

                  • **$8^\circ$ Decil $(28,7;\,33,5]$ ($y_8=21$, fracassos $= 0$):**

                  $\text{logit}^*(p_8) = \ln\!\left(\frac{21 + 0,5}{0 + 0,5}\right) = \ln\!\left(\frac{21,5}{0,5}\right) = \ln(43) = \mathbf{+3,7612}$.

                  • **Importância da correção de Haldane ($+0,5$):** Sem esse acréscimo, como há zero fracassos no último decil ($m_8 - y_8 = 0$), a razão $21/0$ divergiria para $+\infty$, inviabilizando tanto o cálculo tabular quanto o ajuste gráfico da reta de tendência.

                

                

                  
**c) e d) Interpretação da Figura 2 e Decisão Final:**

                  

                    **📝 Como redigir na prova:** “Na Figura 2, os logitos empíricos corrigidos por Haldane exibem um alinhamento essencialmente retilíneo e estritamente ascendente ao longo de toda a amplitude de largura ($23$ a $30$ cm; $R^2 = 0,850$), sem apresentar inflexões em 'U' ou patamares quadráticos acentuados. Como o diagnóstico gráfico confirma integralmente o TRV formal ($\Delta D^* = 0,8508;\, p = 0,3563$), valida-se a premissa de linearidade no logito e mantém-se exclusivamente o termo linear `largura` no modelo principal.”

</details>

---

## Questão 18 — HorseCrab (Lista 5 — Itens d, e, f): Medidas de Bondade de Ajuste e Limitações Teóricas, Envelope hnp (0/173), Diagnóstico de Influência e Sensibilidade LOO
**Bloco:** Bloco V • Especial Lista 5 (Logística HorseCrab a–j) | **Inspiração:** Lista 5 Completa (HorseCrab, n = 173) — Itens d), e) e f) + Figuras 3–4 + Tabela 5 • Profa. Rosemeire

Para validar globalmente o modelo principal selecionado (`y ~ cor + largura`, $p = 5$ parâmetros e $n = 173$ fêmeas com dados binários desagrupados, $m_i = 1$), calcularam-se quatro medidas de qualidade de ajuste, o gráfico meio-normal com envelope simulado (99 réplicas de Monte Carlo via pacote `hnp`), o painel diagnóstico 2×2 (Figura 4) e a análise de sensibilidade por deleção *Leave-One-Out* (LOO).

```r
=== ITEM d): Medidas Globais de Qualidade de Ajuste (y ~ cor + largura) ===
Deviance Nula (D0)        = 225,7585  em 172 gl
Deviance Residual (Dp)    = 187,4570  em 168 gl  (Pseudo-R2 McFadden = 16,97%)
AIC                       = 197,4570
Qui-Quadrado Pearson (X2) = 168,6590  em 168 gl  (X2/gl = 1,0039 | p = 0,4712)
Hosmer-Lemeshow (10 decis)= C_hat = 4,4575  em 8 gl  (p = 0,8137)

=== ITEM e): Envelope Simulado hnp (95%, 99 replicas Monte Carlo) ===
Observacoes fora do envelope: 0 de 173 (0,00%)

=== ITEM f): Destaques Diagnosticos (2p/n = 0,0578 | 3p/n = 0,0867) ===
Obs   | h_ii    | Cook Di | Res.Dev | Larg(cm) | Cor          | Y | DFBETAS_larg
#171  | 0.0659  | 0.0842  |  -1.940 |     28.0 | Escura-Forte | 0 |      -0.1190
#7    | 0.0943  | 0.0635  |  -1.628 |     26.5 | Escura-Forte | 0 |      -0.0066
#128  | 0.0324  | 0.0616  |  +2.141 |     22.5 | Clara        | 1 |      -0.2189
#32   | 0.1084  | 0.0543  |  -1.480 |     25.8 | Escura-Forte | 0 |      +0.0485
#131  | 0.1308  | 0.0320  |  +1.144 |     24.5 | Escura-Forte | 1 |      -0.1260
#87   | 0.0141  | 0.0220  |  -2.073 |     28.5 | Escura-Fraca | 0 |      -0.1768
#94   | 0.0143  | 0.0244  |  -2.112 |     28.7 | Escura-Fraca | 0 |      -0.1872
#157  | 0.0139  | 0.0197  |  -2.033 |     28.3 | Escura-Fraca | 0 |      -0.1657
#23   | 0.0410  | 0.0195  |  +1.523 |     22.5 | Escura-Fraca | 1 |      -0.2514
#54   | 0.0371  | 0.0145  |  +1.439 |     22.9 | Escura-Fraca | 1 |      -0.2151

=== Analise de Sensibilidade Leave-One-Out (LOO) ===
Sem Obs #171: Variacao Intercepto = -3.44% | Variacao Largura = +3.63% (p_larg = 6.72e-06)
Sem Obs #7  : Variacao Intercepto = -0.26% | Variacao Largura = +0.28% (p_larg = 9.84e-06)
Sem Obs #131: Variacao Intercepto = -2.43% | Variacao Largura = +2.57% (p_larg = 7.16e-06)
```

### Subitens Discursivos
- **a)** **(Item d — Valores & Interpretação)** Interprete cada uma das quatro medidas reportadas: (1) a redução $D_0 \to D_p$ ($\text{Pseudo-}R^2$), (2) o $\text{AIC}$, (3) a razão $X^2/gl = 1,0039$, e (4) o Teste de Hosmer-Lemeshow ($\hat{C} = 4,4575, gl = 8, p = 0,8137$).
- **b)** **(Item d — Limitações Críticas!)** Explique claramente a **limitação teórica específica de cada uma das 4 medidas** em regressão logística para dados binários individuais ($m_i = 1$). Por que $D_p$ e $X^2$ NÃO seguem assintoticamente $\chi^2_{n-p}$ aqui, e qual a limitação do próprio $\hat{C}$ de Hosmer-Lemeshow?
- **c)** **(Itens e & f — Envelope & Painel Diagnóstico)** O que significa encontrar **0 de 173 observações ($0,00\%$)** fora do envelope simulado de $95\%$ (`hnp`)? Analisando a **Figura 4** e a tabela de destaques ($2p/n = 0,0578$, $3p/n = 0,0867$), explique por que observações como `#131` ($h_{ii}=0,1308$), `#171` ($D_i=0,0842$) e `#128` ($d_i=+2,141$) aparecem destacadas.
- **d)** **(Análise Leave-One-Out)** Com base nas variações LOO ao remover `#171` ($+3,63\%$ na largura), `#7` e `#131` ($<2,6\%$), decida se alguma observação é influente prejudicial e se deve ser removida da amostra.

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Calcule o Pseudo-$R^2$ de McFadden: $1 - 187,46/225,76 = 16,97\%$. Em Hosmer-Lemeshow, note que como $\hat{C} = 4,4575$ em $g-2 = 8$ gl gera $p = 0,8137 \gg 0,05$, $H_0$ de boa calibração **não é rejeitada**!

                
**Item b):** Lembre-se da regra fundamental de dados binários desagrupados ($m_i = 1$, $y_i \in \{0, 1\}$): a deviance $D_p$ passa a ser função exclusiva das probabilidades preditas $\hat{\pi}_i$, não convergindo para $\chi^2_{n-p}$ porque o número de parâmetros do modelo saturado cresce junto com $n$!

                
**Item c):** Olhe os perfis biológicos na tabela de destaques! `#171` é uma fêmea larga ($28,0$ cm) e escura-forte sem satélite ($Y=0$), indo contra a tendência; `#131` pertence ao estrato raro `Escura-Forte` ($n=12$, peso grande na matriz $H$) e tem alta alavancagem $h_{ii} = 0,1308 > 3p/n = 0,0867$; `#128` é uma fêmea `Clara` estreita ($22,5$ cm) mas com satélite ($Y=1$), gerando resíduo positivo $+2,141$.

                
**Item d):** Compare variações máximas LOO: no pior caso (`#171`, maior Cook), $\hat{\beta}_{\text{larg}}$ muda apenas $+3,63\%$ e mantém $p < 10^{-5}$ — mudança totalmente desprezível!

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Checklist das 4 Limitações (Cai Muito na Prova da Profa. Rosemeire!):**
                  
1. **Deviance Residual ($D_p$):** Com $m_i = 1$, NÃO segue distribuição $\chi^2_{n-p}$ — logo NÃO serve como teste formal de bondade de ajuste! (Nota: na Binomial, como $\phi=1$ é fixo e $m_i=1$, também não existe teste formal de sobredispersão por $D_p/gl$ — embora $X^2/gl = 1,004$ mostre ausência prática de anomalias).
                  
2. **AIC:** Não é um teste de hipótese (não possui valor-$p$ nem escala absoluta de "bom/ruim"); só ordena modelos comparáveis na mesma amostra.
                  
3. **Pearson ($X^2$):** Sofre da mesma falha assintótica de $D_p$ quando $m_i = 1$ e explode artificialmente caso existam probabilidades preditas muito próximas de $0$ ou $1$ (denominadores $\hat{\pi}(1-\hat{\pi}) \approx 0$).
                  
4. **Hosmer-Lemeshow ($\hat{C}$):** Embora seja o teste correto para $m_i=1$, depende da partição arbitrária em $g=10$ grupos (decis), podendo variar conforme empates/cortes, e tem baixo poder para detectar desvios locais isolados nas caudas.

                
**Corte prático de Cook e Alavancagem:** Nenhuma observação supera $D_i = 0,5$ ou $1,0$ (o máximo é apenas $0,0842$!), indicando ausência de influência global grave.

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Interpretação das 4 Medidas Numéricas de Ajuste:**

                  
• **Deviance ($D_0=225,76 \to D_p=187,46$):** As variáveis `cor` e `largura` explicam $16,97\%$ do desvio nulo ($\text{Pseudo-}R^2 = 1 - 187,457/225,759$).

                  • **AIC ($197,46$):** Menor dentre os modelos concorrentes, demonstrando equilíbrio ótimo entre verossimilhança e parcimônia ($p=5$).

                  • **Qui-Quadrado de Pearson ($X^2 = 168,66;\, X^2/gl = 1,0039$):** Razão praticamente unitária ($\approx 1$), sinalizando boa dispersão empírica compatível com a variância binomial.

                  • **Hosmer-Lemeshow ($\hat{C} = 4,4575;\, gl=8;\, p = 0,8137$):** Como $p = 0,8137 \gg 0,05$, **não se rejeita** $H_0$ de que as probabilidades preditas concordam com as frequências observadas nos $10$ decis de risco — atestando **excelente calibração preditiva**.

                

                

                  
**b) Limitações Teóricas de Cada Medida em Dados Binários Individuais ($m_i = 1$):**

                  

                    
1. **Deviance ($D_p$):** Com dados desagrupados ($y_i \in \{0,1\}, m_i=1$), a deviance depende apenas dos $\hat{\pi}_i$ e **NÃO segue distribuição $\chi^2_{n-p}$** (o número de parâmetros saturados cresce com $n$). Assim, não pode testar bondade de ajuste absoluta.

                    
2. **AIC:** É apenas um índice comparativo relativo (sem valor-$p$); não avalia se o melhor modelo dentre os candidatos é realmente adequado em termos absolutos.

                    
3. **Pearson ($X^2$):** Padece da mesma invalidade assintótica da deviance quando $m_i = 1$ e é altamente sensível a probabilidades ajustadas extremas próximas de $0$ ou $1$.

                    
4. **Hosmer-Lemeshow ($\hat{C}$):** Embora supere a falha assintótica via agrupamento em decis, sofre sensibilidade à escolha arbitrária de $g$ ($g=10$) e possui poder reduzido para detectar falta de ajuste localizada em sub-regiões pequenas.

                  

                

                

                  
**c) e d) Envelope `hnp`, Painel Diagnóstico (Figura 4) e Análise LOO:**

                  

                    **📝 Como redigir na prova:** “No gráfico meio-normal simulado (`hnp`, 99 réplicas), observa-se que exatamente **0 de 173 observações (0,00%)** ultrapassam as bandas empíricas de $95\%$, validando rigorosamente a distribuição Binomial, a função de ligação logito e a ausência de *outliers* discrepantes. Na Figura 4, algumas observações destacam-se pontualmente: `#131` ($h_{ii}=0,1308 > 3p/n=0,0867$) e `#32` ($h_{ii}=0,1084$) apresentam alavancagem elevada por pertencerem à categoria rara `Escura-Forte` ($n=12$); `#128` ($d_i=+2,141$) e `#94` ($d_i=-2,112$) excedem discretamente $\pm 2$ no resíduo deviance; `#171` detém a maior Distância de Cook ($0,0842$, ainda muito abaixo de $0,5$); e `#23` registra o maior $|\text{DFBETAS}|$ para largura ($-0,2514$). Contudo, a análise de sensibilidade *Leave-One-Out* demonstra total robustez: removendo `#171`, $\hat{\beta}_{\text{larg}}$ varia apenas $+3,63\%$ e o intercepto $+3,44\%$; removendo `#7` ou `#131`, a variação na largura fica abaixo de $2,6\%$, sem alterar sinais nem conclusões inferenciais ($p < 10^{-5}$). Conclui-se que **não há observações influentes prejudiciais**, devendo todas as $173$ observações permanecer no modelo final.”

</details>

---

## Questão 19 — HorseCrab (Lista 5 — Itens g, h): Matriz de Confusão em c = 0,50, Curva ROC (AUC = 0,771) e Otimização pelo Índice de Youden (c = 0,685)
**Bloco:** Bloco V • Especial Lista 5 (Logística HorseCrab a–j) | **Inspiração:** Lista 5 Completa (HorseCrab, n = 173) — Itens g) e h) + Tabelas 6–7 + Figura 5 • Profa. Rosemeire

Avaliou-se a capacidade discriminatória e preditiva do modelo principal *HorseCrab* (`y ~ cor + largura`, $n = 173$ fêmeas: $111$ com satélites [$Y=1$, prevalência $64,16\%$] e $62$ sem satélites [$Y=0$]). Inicialmente classificou-se como Caso ($\hat{Y}=1$) se $\hat{\pi} \ge 0,50$; em seguida construiu-se a Curva ROC e identificou-se o ponto de corte ótimo pelo Índice de Youden em $c_{\text{ótimo}} = 0,685$.

```r
=== ITEM g): TABELA 6 — Matriz de Confusao no Corte Padrao c0 = 0,50 ===
                    Observado
Classif. Predita    Sem Satelite (Y=0)   Com Satelite (Y=1)   Total Predito
Predito Nao (Y^=0)         VN = 31               FN = 15             46
Predito Caso(Y^=1)         FP = 31               VP = 96            127
Total Real                     62                  111            173

=== ITEM h): Curva ROC, AUC e Comparacao (Tabela 7) ===
AUC = 0,7714  (EP = 0,0351 | IC 95% = [0,703; 0,840])

Metrica Preditiva   | Corte Padrao (c=0,50) | Corte Otimo Youden (c=0,685) | Diferenca Abs.
Sensibilidade (SE)  | 86,49% (96/111)      | 63,06% (70/111)            | -23,43 p.p.
Especificidade (SP) | 50,00% (31/62)       | 80,65% (50/62)             | +30,65 p.p.
Acuracia Global     | 73,41% (127/173)    | 69,36% (120/173)          | -4,05 p.p.
Indice Youden (J)   | 0,3649                | 0,4371                     | +0,0722
```

### Subitens Discursivos
- **a)** **(Item g — Métricas em $c_0 = 0,50$)** A partir da Tabela 6 ($VN=31, FN=15, FP=31, VP=96$), calcule e interprete a **Sensibilidade**, a **Especificidade**, o **VPP**, o **VPN** e a **Acurácia Global**.
- **b)** Explique por que o limiar padrão $c_0 = 0,50$ privilegiou a Sensibilidade ($86,49\%$), mas sofreu Especificidade modesta de exatos $50,00\%$ ($31$ falsos positivos — metade das fêmeas sem satélites mal classificada!), relacionando com a prevalência amostral de $64,16\%$.
- **c)** **(Item h — Curva ROC & AUC)** Analisando a Figura 5, interprete probabilisticamente $\text{AUC} = 0,7714$ e classifique o poder discriminatório do modelo na escala de Hosmer & Lemeshow.
- **d)** **(Ponto de Corte Ótimo & Trade-off)** Defina o **Índice de Youden ($J$)**, confira o valor de $J$ em $c = 0,50$ vs. $c_{\text{ótimo}} = 0,685$, e discuta o *trade-off* completo da Tabela 7 ao adotar $c = 0,685$ (redução de $31$ para quantos falsos positivos?).

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Confira os 5 quocientes no corte $0,50$: $\text{SE} = 96/111$, $\text{SP} = 31/62$, $\text{VPP} = 96/127$, $\text{VPN} = 31/46$ e $\text{Acc} = (96+31)/173$.

                
**Item b):** Quando a prevalência de $Y=1$ é alta ($111/173 = 64,16\%$), as probabilidades preditas deslocam-se naturalmente para cima — muitas fêmeas $Y=0$ acabam recebendo $\hat{\pi}_i$ entre $0,50$ e $0,68$, caindo acima de $c_0=0,50$ como Falsos Positivos!

                
**Item c):** Na tabela de Hosmer-Lemeshow: $0,50 =$ acaso; $0,50$–$0,70 =$ fraco; $0,70$–$0,80 =$ **aceitável/razoável (Classificação C)**; $0,80$–$0,90 =$ excelente; $\ge 0,90 =$ excepcional.

                
**Item d):** Calcule os $FP$ em $c = 0,685$: como existem $62$ fêmeas $Y=0$ e $VN = 50$ ($\text{SP} = 80,65\%$), restam $FP = 62 - 50 = 12$ falsos positivos (queda de $31 \to 12$!). E $J(0,685) = 0,6306 + 0,8065 - 1 = 0,4371 > J(0,50) = 0,3649$.

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Simetria com o Corte Baixo (Q15 vs. Q19):** Note o contraste didático precioso para a prova! Quando a prevalência é *baixa* ($23,5\%$ na Q15), o corte $0,50$ peca por *baixa Sensibilidade* (precisa baixar $c$ para $\sim 0,20$). Já quando a prevalência é *alta* ($64,2\%$ aqui em HorseCrab), o corte $0,50$ peca por *baixa Especificidade* ($50\%$) e precisa ser **elevado** para $c_{\text{ótimo}} = 0,685$!

                
**Geometria do Índice de Youden:** $J = \text{Sensibilidade} + \text{Especificidade} - 1 = \text{TVP} - \text{TFP}$ mede exatamente a *distância vertical* entre a Curva ROC e a diagonal aleatória de $45^\circ$!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) e b) Métricas em $c_0 = 0,50$ e Efeito da Prevalência ($64,16\%$):**

                  

                    
• **Sensibilidade (SE):** $96 / 111 = \mathbf{0,8649}$ ($86,49\%$) — alta detecção de fêmeas com satélites (apenas $15$ falsos negativos);

                    
• **Especificidade (SP):** $31 / 62 = \mathbf{0,5000}$ ($50,00\%$) — baixa capacidade de descartar fêmeas sem satélites ($31$ falsos positivos!);

                    
• **VPP:** $96 / 127 = \mathbf{0,7559}$ ($75,59\%$) — dado que o modelo previu $Y=1$, acerta em $75,6\%$ dos casos;

                    
• **VPN:** $31 / 46 = \mathbf{0,6739}$ ($67,39\%$) — dado que previu $Y=0$, acerta em $67,4\%$;

                    
• **Acurácia Global:** $(96 + 31)/173 = 127/173 = \mathbf{0,7341}$ ($73,41\%$).

                  

                  
**Por que $\text{SP} = 50,00\%$?** Como a prevalência amostral de casos ($64,16\%$) já ultrapassa $0,50$, o intercepto ajusta probabilidades basais relativamente altas. Muitas fêmeas sem satélites atingem $\hat{\pi}_i \in [0,50;\,0,68]$ e são classificadas erroneamente como positivas pelo limiar fixo $0,50$.

                

                

                  
**c) e d) Curva ROC, AUC e Corte Ótimo de Youden ($c = 0,685$):**

                  

                    **📝 Como redigir na prova:** “A Curva ROC (Figura 5) revela $\text{AUC} = \mathbf{0,7714}$ ($\approx 0,771$; IC $95\%: [0,699;\,0,844]$). Isso significa que, ao sortear aleatoriamente uma fêmea com satélite e uma sem satélite, há $77,14\%$ de probabilidade de o modelo atribuir $\hat{\pi}_{\text{com}} > \hat{\pi}_{\text{sem}}$, classificando a capacidade discriminatória como **aceitável / razoável (Classificação C de Hosmer & Lemeshow)**.

                    Pelo **Índice de Youden** $J = \text{SE} + \text{SP} - 1$, o corte padrão apresenta $J(0,50) = 0,8649 + 0,5000 - 1 = 0,3649$, enquanto a busca na curva ROC maximiza $J$ em $c_{\text{ótimo}} = \mathbf{0,685}$ ($J = 0,6306 + 0,8065 - 1 = \mathbf{0,4371}$). Ao elevar o corte de $0,50$ para $0,685$, ganha-se expressivamente em Especificidade (**$+30,65$ p.p.**, subindo de $50,00\%$ para $80,65\%$ e cortando os falsos positivos de $31$ para apenas **$12$**) ao custo controlado de redução na Sensibilidade ($-23,43$ p.p., indo para $63,06\%$) e leve variação na acurácia global ($73,41\% \to 69,36\%$). O limiar $0,685$ equilibra os erros entre as duas classes quando se busca discriminação simétrica.”

</details>

---

## Questão 20 — HorseCrab (Lista 5 — Itens i, j): Curvas Sigmoidais de Probabilidade Predita, Defasagem Fisiológica (x₀,₅₀) e Efeito Marginal Condicional
**Bloco:** Bloco V • Especial Lista 5 (Logística HorseCrab a–j) | **Inspiração:** Lista 5 Completa (HorseCrab, n = 173) — Itens i) e j) + Figuras 6–7 • Profa. Rosemeire

Na etapa final da Lista 5, avaliou-se o comportamento do modelo principal `y ~ cor + largura` na escala natural da **probabilidade** $\hat{\pi}(x)$ para cada pigmentação de carapaça (Item i) e deduziu-se a curva do **efeito marginal condicional** da largura da carapaça via regra da cadeia (Item j): $\frac{\partial \pi(x)}{\partial \text{largura}} = \hat{\beta}_{\text{larg}} \cdot \pi(x)\big(1 - \pi(x)\big)$.

```r
=== Equacao Predita do Modelo Principal ===
eta(cor, largura) = -12.7151 + 1,1061*corMedia + 1,4023*corEscura-Fraca + 1,3299*corEscura-Forte + 0.4680*largura
pi_hat(x)         = [1 + exp(-eta(cor, largura))]^(-1)

=== Pontos de Inflexao (pi_hat = 0,50), Probabilidades e Efeitos Marginais por Cor ===
Cor          | Offset Cor | x_0.50(cm) | pi(22) | pi(26) | pi(30) | Marg(22) | Marg(32)
Clara        |    +0.0000 |      27,17 | 0.0817 | 0.3663 | 0.7898 |   3.51%  |   4.01%
Media        |    +1.1061 |      24,81 | 0.2118 | 0.6360 | 0.9191 |   7.81%  |   1.51%
Escura-Fraca |    +1.4023 |      24,17 | 0.2655 | 0.7014 | 0.9385 |   9.12%  |   1.14%
Escura-Forte |    +1.3299 |      24,33 | 0.2516 | 0.6860 | 0.9342 |   8.81%  |   1.22%

Efeito Marginal Maximo Teorico: beta_larg * 0,25 = 0.4680 * 0,25 = 0.1170 (11,70% por cm)
Defasagem fisiologica Clara vs. Escura-Fraca em pi = 0,50: 3,00 cm
```

### Subitens Discursivos
- **a)** **(Item i — Curvas Sigmoidais $\hat{\pi}(x)$)** Observando a **Figura 6**, descreva o formato das curvas preditas na escala de probabilidade ($g^{-1}(\eta) = (1+e^{-\eta})^{-1}$): ele é linear ou não-linear? Qual o impacto do termo de `cor` sobre a geometria das $4$ curvas (há alteração de inclinação/interação ou translação horizontal)?
- **b)** **(Ponto de Inflexão $x_{0,50}$ e Defasagem Fisiológica)** Sabendo que $\hat{\pi}(x) = 0,50 \iff \hat{\eta}(x) = 0$, calcule algebricamente a largura exata $x_{0,50}$ necessária para atingir $50\%$ de chance de atrair satélites nas fêmeas `Clara` e `Escura-Fraca`, demonstrando a defasagem fisiológica de $3,00\text{ cm}$ visível na Figura 6.
- **c)** **(Item j — Derivada & Efeito Marginal Condicional)** Partindo da ligação logística $\pi(x) = (1+e^{-\eta})^{-1}$, explique por que o efeito marginal $\frac{\partial \pi}{\partial \text{largura}} = \hat{\beta}_{\text{larg}} \cdot \pi(x)\big(1 - \pi(x)\big)$ **não é constante** (Figura 7, formato de sino), e demonstre algebricamente seu valor máximo teórico ($0,1170$, ou $11,70\%/\text{cm}$).
- **d)** **(Saturação e Coincidência de Picos)** Por que os picos das $4$ curvas de efeito marginal na Figura 7 coincidem exatamente com as larguras $x_{0,50}$ encontradas no item b ($24,17;\, 24,33;\, 24,81;\, 27,17$ cm), e o que acontece com o efeito marginal de $+1\text{ cm}$ nas extremidades ($\text{largura} \le 22$ cm ou $\ge 32$ cm)?

<details>
<summary>👣 Primeiro Passo (Por onde começar)</summary>

**Item a):** Lembre que o modelo é aditivo (sem termo `cor:largura`): portanto na escala do *logito* as retas são paralelas, e na escala da *probabilidade* (Figura 6) a diferença de cor provoca uma **translação horizontal rígida** em forma de "S"!

                
**Item b):** Resolva $\hat{\eta} = 0$: para `Clara` (offset $0$), $-12,7151 + 0,4680\,x_{0,50} = 0 \implies x_{0,50} = 12,7151/0,4680 = 27,17\text{ cm}$. Para `Escura-Fraca` ($+1,4023$), $(-12,7151 + 1,4023) + 0,4680\,x_{0,50} = 0 \implies x_{0,50} = 11,3128/0,4680 = 24,17\text{ cm}$!

                
**Item c):** Pela regra da cadeia, $\frac{d}{dx}(1+e^{-\eta})^{-1} = \beta \cdot \frac{e^{-\eta}}{(1+e^{-\eta})^2} = \beta\,\pi(1-\pi)$. A parábola $f(\pi) = \pi(1-\pi)$ atinge seu máximo em $\pi = 0,50$, onde $f(0,50) = 0,50 \times 0,50 = 0,25$!

                
**Item d):** Compare as extremidades na tabela de saída: para `Escura-Fraca` aos $32\text{ cm}$, $\hat{\pi}$ já saturou perto de $97,5\%$, logo $\pi(1-\pi) = 0,024$ e o efeito marginal cai para meros $1,14\%/\text{cm}$ ($< 2\%$).

</details>

<details>
<summary>💡 Dica & Armadilhas de Prova</summary>

**Distinção Crucial na Prova (OR vs. Efeito Marginal):**
                  
• Na escala do **Odds (Chance)**, o efeito multiplicativo é *constante* ($\text{OR} = e^{0,4680} = 1,597$, sempre $+59,7\%$ na chance por cm, qualquer que seja a cor ou a largura inicial!).
                  
• Na escala da **Probabilidade ($\pi$)**, porém, o efeito aditivo em pontos percentuais ($\%$) *varia conforme $x$ e conforme a cor* ($\partial\pi/\partial x = \beta\pi(1-\pi)$), atingindo o ápice exato de $\beta/4$ ($11,70\text{ p.p./cm}$) quando a fêmea está exatamente em $50\%$ de chance!

                
**Regra Prática (Regra do "Divide por 4" de Gelman & Hill):** A inclinação máxima de qualquer curva logística na escala da probabilidade vale exatamente $\hat{\beta} / 4$!

</details>

<details>
<summary>✅ Gabarito Completo (Padrão Profa. Rosemeire)</summary>

**a) Formato das Curvas Sigmoidais e Translação entre Cores (Figura 6):**

                  
Na escala da probabilidade, as curvas são **estritamente não-lineares (formato sigmoidal em "S")** devido à inversa do logito $g^{-1}(\eta) = (1+e^{-\eta})^{-1} \in [0, 1]$, sendo estritamente crescentes com a largura (abaixo de $23\text{ cm}$, $\hat{\pi} 90\%$). Como o modelo é aditivo sem interação, o termo $\hat{\beta}_{\text{cor}}$ provoca apenas **translações horizontais** entre as cores, mantendo inalterada a forma da sigmoide e a razão de chances.

                

                

                  
**b) Cálculo de $x_{0,50}$ e Defasagem Fisiológica:**

                  
Como $\hat{\pi}(x) = 0,50 \iff \text{logit}(0,50) = \ln(1) = 0$, igualamos $\hat{\eta}(x) = 0$:

                  • **Clara (referência, $+0$):** $-12,7151 + 0,4680\,x_{0,50} = 0 \implies x_{0,50} = 12,7151 / 0,4680 = \mathbf{27,17\text{ cm}}$;

                  • **Escura-Fraca ($+1,4023$):** $(-12,7151 + 1,4023) + 0,4680\,x_{0,50} = 0 \implies x_{0,50} = 11,3128 / 0,4680 = \mathbf{24,17\text{ cm}}$;

                  *(Média: $24,81\text{ cm}$; Escura-Forte: $24,33\text{ cm}$).*

                  Portanto, há uma **defasagem fisiológica de $27,17 - 24,17 = \mathbf{3,00\text{ cm}}$**: uma fêmea escura-fraca atinge $50\%$ de probabilidade com carapaça $3\text{ cm}$ menor do que uma fêmea clara!

                

                

                  
**c) e d) Efeito Marginal Condicional, Máximo Teórico $\hat{\beta}/4$ e Saturação (Figura 7):**

                  

                    **📝 Como redigir na prova:** “Não, o efeito marginal da largura sobre a probabilidade $\pi(x)$ **não é constante**. Pela regra da cadeia, $\frac{\partial \pi(x)}{\partial \text{largura}} = \hat{\beta}_{\text{larg}} \cdot \pi(x)\big(1 - \pi(x)\big) = 0,4680 \cdot \pi(x)\big(1 - \pi(x)\big)$, produzindo curvas em formato de sino (*bell-shaped*) moduladas pela variância binomial $\pi(1-\pi)$. Como a função côncava $\pi(1-\pi)$ atinge máximo absoluto em $\pi = 0,50$ (onde vale $0,25$), o **efeito marginal máximo teórico** é idêntico para todas as cores e vale $\max(\partial\pi/\partial x) = 0,4680 \times 0,25 = \mathbf{0,1170}$ (isto é, um aumento de até **$11,70\text{ pontos percentuais por centímetro adicional}$**). Esse pico ocorre exatamente no ponto de inflexão $\hat{\pi} = 0,50$ de cada sigmoide ($24,17\text{ cm}$ nas Escuras-Fracas, $24,33\text{ cm}$ nas Escuras-Fortes, $24,81\text{ cm}$ nas Médias e deslocado para $27,17\text{ cm}$ nas Claras). Nas extremidades (carapaças muito estreitas ou muito largas), $\pi(x) \to 0$ ou $1$, fazendo $\pi(1-\pi) \to 0$ (saturação nas assíntotas) e reduzindo o ganho marginal para menos de $2\%/\text{cm}$ ($<0,02$).”

</details>

---
