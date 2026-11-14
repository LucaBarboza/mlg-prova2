# GUIA COMPLETO E DEFINITIVO PARA A PROVA: MODELOS LINEARES GENERALIZADOS (MLG)
**Universidade Federal da Bahia (UFBA) — MATD50 | Profa. Rosemeire L. Fiaccone**  
*Material compilado e estruturado exclusivamente a partir dos slides e notas de aula oficiais do curso.*

---

## SUMÁRIO GERAL

1. [PARTE 1: ANÁLISE DO DESVIO (ANODEV) E QUALIDADE GLOBAL DO AJUSTE](#parte-1-análise-do-desvio-anodev-e-qualidade-global-do-ajuste)
   - 1.1 Função Desvio (*Deviance*) e Decomposição da Verossimilhança
   - 1.2 Deviance Escalonada vs. Não Escalonada
   - 1.3 Estatística de Pearson Generalizada ($X^2$) vs. Deviance
   - 1.4 Medidas de Qualidade: Pseudo-$R^2$ e Critério de Akaike (AIC)
   - 1.5 Tabela ANODEV e Teste da Razão de Verossimilhanças (TRV)
   - 1.6 Estimação do Parâmetro de Dispersão ($\phi$) e Inferência com $\phi$ Desconhecido (Estatística $F$)
   - 1.7 Algoritmos de Seleção de Covariáveis (*Forward*, *Backward*, Combinado)

2. [PARTE 2: INFERÊNCIA COMPLEMENTAR E DIAGNÓSTICOS GERAIS DO MLG](#parte-2-diagnósticos-gerais-do-mlg)
   - 2.0 Robustez dos MLGs, Tríade Assintótica Completa e IC via Deviance
   - 2.1 Por que o Resíduo Ordinário não é adequado em MLG? (e Convenções de $\phi$)
   - 2.2 Resíduos de Pearson, Anscombe ($r_i^A$) e a Matriz Chapéu ($\mathbf{H}$)
   - 2.3 Resíduos do Desvio ($r_i^{DP}$) e Resíduo de Verossimilhança ($r_{Gi}$, Williams 1987)
   - 2.4 Análise Gráfica dos Resíduos: Padrões e Falhas Estruturais
   - 2.5 Gráficos com Envelopes Simulados e Gráfico Meio-Normal (*Half-Normal Plot*)
   - 2.6 Pontos de Alavanca (*Leverage*) e Medidas de Influência (*Leave-One-Out*, $LD_i$, Distância de Cook, DFBETAS)

3. [PARTE 3: REGRESSÃO LOGÍSTICA](#parte-3-regressão-logística)
   - 3.1 Fundamentos, Estrutura dos Dados e Funções de Ligação
   - 3.2 Regressão Logística Simples: Equação, Estimação e Dados Agrupados
   - 3.3 Interpretação dos Parâmetros, Codificação ($\{0,1\}$ vs $\{-1,+1\}$), Linearidade no Logito e *Odds Ratio* ($OR$)
   - 3.4 Medidas de Efeito: Risco Atribuível ($d$), Risco Relativo ($RR$) vs. *Odds Ratio* ($OR$) em Tabelas $2 \times 2$
   - 3.5 Regressão Logística Múltipla, Variáveis Categóricas, Interações (Slide 71) e EPV
   - 3.6 Inferência: Teste de Wald, TRV e Intervalos de Confiança (Coeficiente, OR e $\pi(\mathbf{x})$)
   - 3.7 Padrões de Covariáveis, Dados Agrupados ($\chi^2_{N-p}$) e o Teste de Hosmer-Lemeshow (Limitações)
   - 3.8 Poder Preditivo, Matriz de Confusão (Sensibilidade, Especificidade, Acurácia)
   - 3.9 Curva ROC (*Receiver Operating Characteristic*) e a Área sob a Curva ($AUC$)
   - 3.10 Diagnósticos Específicos em Regressão Logística e Eventos Raros (`relogit`, Firth)

4. [PARTE 4: CADERNO DE INTERPRETAÇÃO DE SAÍDAS DO R PARA A PROVA](#parte-4-caderno-de-interpretação-de-saídas-do-r-para-a-prova)
   - 4.1 Anatomia da Saída `summary(glm(...))` (Hospital Heliópolis & Cupons)
   - 4.2 Leitura da Tabela `anova(ajuste, test = "Chisq")` e Seleção Stepwise
   - 4.3 Saída do Teste de Hosmer-Lemeshow (`hoslem.test`)
   - 4.4 Saída da Curva ROC e Discriminação (`lroc`, `logistic.display`)
   - 4.5 Estudos de Caso Completos Resolvidos dos Slides (Koch et al. & Paciente #172)

5. [PARTE 5: FORMULÁRIO DE BOLSO E GUIA DE DECISÃO RÁPIDO](#parte-5-formulário-de-bolso-e-guia-de-decisão-rápido)

---

# PARTE 1: ANÁLISE DO DESVIO (ANODEV) E QUALIDADE GLOBAL DO AJUSTE

## 1.1 Função Desvio (*Deviance*) e Decomposição da Verossimilhança

O objetivo principal da função desvio é analisar a **adequação do modelo como um todo**, avaliando a "bondade do ajuste" (*goodness of fit*) baseada na diferença de log-verossimilhanças maximizadas.

Em modelagem estatística, busca-se um modelo que resuma os dados de forma **parcimoniosa**:
- Não deve perder informação relevante (evitar subajuste ou viés de omissão).
- Não deve ser desnecessariamente complexo (evitar sobreajuste e perda de precisão/graus de liberdade).

Estatisticamente, isso equivale a comparar o modelo sob estudo com o **modelo saturado** e verificar se a discrepância entre eles é estatisticamente significativa.

### Os Três Modelos de Referência
1. **Modelo Nulo ($M_0$):** Contém apenas o intercepto comum ($\beta_0$) para todas as observações. Representa a ausência total de efeito das covariáveis ($p = 1$).
2. **Modelo Proposto / Intermediário ($M_p$):** Contém os $p$ parâmetros de interesse ($p < n$).
3. **Modelo Saturado ($M_s$ ou $M_n$):** Possui tantos parâmetros quantos forem as observações ou subpopulações ($p = n$). O modelo ajusta perfeitamente os dados observados ($\hat{\mu}_i = y_i$), fornecendo o limite superior teórico da log-verossimilhança.

---

## 1.2 Deviance Escalonada ($D^*$) vs. Deviance Não Escalonada ($D_p$ ou $D$)

A professora Rosemeire e a literatura de Gilberto Paula adotam a decomposição clássica da log-verossimilhança da família exponencial:
$$\ell(\boldsymbol{\theta}, \phi; \mathbf{y}) = \sum_{i=1}^n \left[ \frac{y_i \theta_i - b(\theta_i)}{a(\phi)} + c(y_i, \phi) \right]$$
com $\text{Var}(Y_i) = a(\phi)V(\mu_i)$. Nos slides do curso convivem duas notações equivalentes:

### Deviance Escalonada (*Scaled Deviance*, $D^*$)
Definida pelo dobro da diferença entre a log-verossimilhança do modelo saturado ($\ell_n$) e a do modelo proposto com $p$ parâmetros ($\ell_p$):
$$D^* = 2(\ell_n - \ell_p) = a^{-1}(\phi) D_p = a^{-1}(\phi) \sum_{i=1}^n d_i^2$$
* É **adimensional** (livre da unidade da resposta $Y$) e segue assintoticamente $D^* \sim_a \chi^2_{n-p}$ (exata na Normal com $\sigma^2$ conhecido).
* **Alerta do Slide 7:** *"Para modelos com parâmetros de dispersão que não foram estimados, a deviance escalonada $D^*$ não pode ser determinada."*

### Deviance Não Escalonada (*Unscaled Deviance*, $D_p$ ou $D$)
Mede a discrepância dependendo apenas de $\mathbf{y}$ e $\hat{\boldsymbol{\mu}}$, **sem dividir pelo parâmetro de dispersão $\phi$**:
$$D_p = D(\mathbf{y}; \hat{\boldsymbol{\mu}}) = \sum_{i=1}^n d_i^2 = 2 a(\phi) (\ell_n - \ell_p)$$
onde $d_i$ é o **componente do desvio não escalonado** da $i$-ésima observação.
* **🚨 O que o software R imprime:** As linhas `Null deviance` e `Residual deviance` em `summary(glm)` e `anova(glm)` são **deviances NÃO-escalonadas ($D_p$)**!
* Quando $\phi = 1$ (Poisson e Binomial): $D_p = D^*$ coincidem numericamente.
* Quando $\phi \ne 1$ (Normal, Gama, Normal Inversa): $D_p$ carrega a escala de $Y$ (na Normal, $D_p = SQRes$ em unidade de $Y^2$). Logo, **nunca compare $D_p$ diretamente com $\chi^2_{n-p}$ nem use $D_p/(n-p) \approx 1$** — antes divida pela estimativa $\hat{\phi}$ ($D^* = D_p/\hat{\phi}$).

### Conciliação das Duas Notações nos Slides (`mlgp2.pdf`)
1. **Convenção 1 — Parâmetro de dispersão $\phi$ (Slides 42–50 e software R):** $a(\phi) = \phi \implies D^* = \frac{D_p}{\phi}$. Aqui Normal tem $\phi = \sigma^2$, Gama tem $\phi = \nu^{-1}$ e Normal Inversa tem $\phi = \sigma^2$. É o padrão do R (`Dispersion parameter`) e do estimador $\hat{\phi}_D = D_p/(n-p)$.
2. **Convenção 2 — Notação Gilberto Paula (Slides 2 e 8 — parâmetro de *precisão*):** escreve $\phi$ no numerador:
   $$D^*(\mathbf{y}; \hat{\boldsymbol{\mu}}) = \phi D(\mathbf{y}; \hat{\boldsymbol{\mu}}) = 2 \{ L(\mathbf{y}; \mathbf{y}) - L(\hat{\boldsymbol{\mu}}; \mathbf{y}) \}$$
   Aqui $\phi = \sigma^{-2}$ na Normal. Em testes comparativos ($\Delta D^*$ ou Teste $F$), $\phi$ cancela entre numerador e denominador e a decisão estatística é **idêntica**.

### Fórmulas Explícitas da Deviance Não-Escalonada ($D_p$) por Distribuição
| Distribuição | $\phi$ (Conv. 1) | $V(\mu)$ | Deviance Não-Escalonada ($D_p$) | Comparar $D_p$ direto com $\chi^2_{n-p}$? |
| :--- | :---: | :---: | :--- | :---: |
| **Normal** | $\sigma^2$ (desconhecido) | $1$ | $\sum_{i=1}^n (y_i - \hat{\mu}_i)^2 = SQRes$ | **Não!** Só $D^* = SQRes/\sigma^2$ (exata) |
| **Poisson** | $1$ (conhecido) | $\mu$ | $2\sum_{i=1}^n \left[y_i\ln\!\left(\frac{y_i}{\hat{\mu}_i}\right) - (y_i-\hat{\mu}_i)\right]$ | **Sim** ($D_p = D^*$) |
| **Binomial (agrup.)** | $1$ (conhecido) | $\frac{\pi(1-\pi)}{n_i}$ | $2\sum_{i=1}^N \left[y_i\ln\!\left(\frac{y_i}{n_i\hat{\pi}_i}\right) + (n_i-y_i)\ln\!\left(\frac{n_i-y_i}{n_i-n_i\hat{\pi}_i}\right)\right]$ | **Sim** (se contagens adequadas) |
| **Bernoulli ($m_i=1$)** | $1$ (conhecido) | $\pi(1-\pi)$ | $-2\sum_{i=1}^n \left[y_i\ln\hat{\pi}_i + (1-y_i)\ln(1-\hat{\pi}_i)\right]$ | **Não!** Falha $\to$ Hosmer-Lemeshow |
| **Gama** | $\nu^{-1}$ (desconhecido) | $\mu^2$ | $2\sum_{i=1}^n \left[-\ln\!\left(\frac{y_i}{\hat{\mu}_i}\right) + \frac{y_i - \hat{\mu}_i}{\hat{\mu}_i}\right]$ | **Não!** Só $D^* = D_p / \hat{\phi}$ |
| **Normal Inversa** | $\sigma^2$ (desconhecido) | $\mu^3$ | $\sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{y_i \hat{\mu}_i^2}$ | **Não!** Só $D^* = D_p / \hat{\phi}$ |

### Propriedades Cruciais da Função Desvio para a Prova:
1. **Não negatividade:** $D^* \ge 0$ e $D_p \ge 0$.
2. **Monotonicidade decrescente:** O desvio **sempre decresce** (ou permanece igual) à medida que novas variáveis explicativas são acrescentadas ao modelo ($\ell_0 \le \ell_m \le \ell_q \le \ell_n \implies D_0^* \ge D_m^* \ge D_q^* \ge 0$).
3. **Sentido de qualidade:** Quanto melhor o ajuste aos dados, mais $\ell_p$ se aproxima de $\ell_n$, e portanto **menor será o valor de $D^*$**.
4. **Distribuição assintótica:** Se o modelo estiver corretamente especificado, $D^* \sim_a \chi^2_{n-p}$.
5. **Casos especiais:** No modelo linear normal com variância conhecida, a deviance escalonada é exatamente $SQRes/\sigma^2$, e sua distribuição qui-quadrado é **exata**.

---

## 1.3 Estatística de Pearson Generalizada ($X^2$) vs. Deviance

Uma alternativa clássica ao teste de qualidade de ajuste baseado na deviance é a **Estatística de Pearson Generalizada**:
$$X^2 = \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{\widehat{\text{Var}}(y_i)} = \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{\hat{\phi}^{-1} V(\hat{\mu}_i)}$$

Assim como a deviance escalonada, a estatística $X^2$ tem distribuição assintótica qui-quadrado:
$$X^2 \sim_a \chi^2_{n-p}$$
se o modelo proposto for correto e o parâmetro de dispersão for conhecido.

### Comparativo Exigido em Provas: Deviance vs. Pearson Generalizada
| Critério | Função Desvio ($D^*$) | Estatística de Pearson ($X^2$) |
| :--- | :--- | :--- |
| **Definição** | Baseada na razão de verossimilhanças | Baseada na discrepância quadrática ponderada |
| **Aditividade** | **É aditiva** para modelos encaixados ($\Delta D^* = D_m^* - D_q^*$) | **Não é aditiva** |
| **Comportamento ao incluir variáveis** | **Sempre decresce** com o acréscimo de preditores | **Não necessariamente decresce**; pode oscilar |
| **Distribuição no modelo normal** | Distribuição $\chi^2_{n-p}$ exata | Distribuição $\chi^2_{n-p}$ exata |
| **Uso preferencial** | Comparação e seleção de modelos encaixados (ANODEV) | Detecção de superdispersão e cálculo de $\hat{\phi}$ |

---

## 1.4 Medidas de Qualidade: Pseudo-$R^2$ e Critério de Akaike (AIC)

### Pseudo-$R^2$
Sejam $D_0$ o desvio do modelo nulo (apenas com o intercepto) e $D_1$ o desvio de um modelo intermediário com covariáveis. Define-se o pseudo-$R^2$ como:
$$R^2 = \frac{D_0 - D_1}{D_0} = 1 - \frac{D_1}{D_0}$$
- **Interpretação:** Representa a **proporção do desvio do modelo nulo que foi explicada pelas covariáveis** incluídas no modelo intermediário.
- Varia no intervalo $[0, 1]$. Quanto mais próximo de 1, maior foi a redução do desvio.

### Critério de Informação de Akaike (AIC)
Mede a qualidade do ajuste penalizando a complexidade (número de parâmetros), promovendo a **parcimônia**:
$$AIC = -2 \ell(\hat{\boldsymbol{\mu}}; \mathbf{y}) + 2p$$
onde $p$ é o número total de parâmetros estimados no modelo.

- **Regra de Decisão:** Modelos com **menor AIC** são preferíveis.
- **Regra de Ouro dos Slides:** Se a diferença entre os AICs de dois modelos concorrentes for maior do que 2 ($\Delta AIC > 2$), o modelo com menor AIC é considerado **fortemente preferível**.
- No software R, o AIC é extraído diretamente via função `AIC(objeto)` ou na última linha do `summary(objeto)`.

---

## 1.5 Tabela ANODEV e Teste da Razão de Verossimilhanças (TRV)

A Análise do Desvio (**ANODEV**) é a extensão direta da Análise de Variância (ANOVA) para a classe dos Modelos Lineares Generalizados.
- Baseia-se na comparação de modelos **encaixados** (*nested models*).
- Dois modelos são encaixados quando o modelo mais simples ($M_m$, com $m$ parâmetros) pode ser obtido como um caso particular do modelo mais complexo ($M_q$, com $q$ parâmetros, onde $m < q$), fixando certos parâmetros em zero:
  $$H_0: \boldsymbol{\beta} = \boldsymbol{\beta}_m = [\beta_1, \dots, \beta_m]^\top \quad \text{vs.} \quad H_1: \boldsymbol{\beta} = \boldsymbol{\beta}_q = [\beta_1, \dots, \beta_m, \beta_{m+1}, \dots, \beta_q]^\top$$

### Diferença de Deviances e o TRV
Para testar $H_0$ contra $H_1$, calcula-se a diferença entre suas estatísticas de deviance:
$$\Delta D^* = D^*_m - D^*_q = 2[\ell_n - \ell_m] - 2[\ell_n - \ell_q] = 2(\ell_q - \ell_m) = 2 \ln\left( \frac{L_q}{L_m} \right)$$
A diferença entre deviances escalonadas é **exatamente igual à estatística do Teste da Razão de Verossimilhanças (TRV)**.

Sob $H_0$ e condições de regularidade:
$$\Delta D^* \sim_a \chi^2_{q - m}$$
com graus de liberdade dados pela diferença no número de parâmetros: $\Delta gl = q - m$.

- **Critério de Decisão:**
  - Se $\Delta D^* > \chi^2_{q-m; 1-\alpha}$ (ou se $p\text{-valor} < \alpha$): **Rejeita-se $H_0$**. Conclui-se que os parâmetros adicionais são estatisticamente significativos; o modelo mais complexo ($M_q$) descreve os dados significativamente melhor.
  - Se $p\text{-valor} \ge \alpha$: **Não se rejeita $H_0$**. Pelo princípio da parcimônia, mantém-se o modelo mais simples ($M_m$).

### Estrutura Formal da Tabela ANODEV (Notação dos Slides)
Considere dois preditores $X_1$ e $X_2$:

| Modelo | g.l. | Deviances | TRV | $\Delta$ g.l. | $p$-valor |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Nulo** | $gl_N$ | $D_N$ | — | — | — |
| **$X_1$** | $gl_N - 1$ | $D_1$ | $D_N - D_1$ | 1 | $P(\chi^2_1 \ge D_N - D_1)$ |
| **$X_2 \mid X_1$** | $gl_N - 2$ | $D_2$ | $D_1 - D_2$ | 1 | $P(\chi^2_1 \ge D_1 - D_2)$ |
| **$X_1 * X_2 \mid X_1, X_2$** | $gl_N - 3$ | $D_3$ | $D_2 - D_3$ | 1 | $P(\chi^2_1 \ge D_2 - D_3)$ |

> **Atenção para a Prova (Regras de $n$, $p$, $gl$, `NA` e Tipo I vs Tipo II — Slides 8–9):**  
> 1. **Dados agrupados vs individuais:** Em dados individuais, $n$ = nº de indivíduos ($gl_0 = n-1$). Em **dados agrupados** (subpopulações), $N$ = nº de **linhas/subpopulações** da tabela, e NÃO o total de indivíduos somados ($\implies gl_N = N - 1$, Slide 8!).  
> 2. **Contagem de parâmetros $p$:** Intercepto ($+1$) + contínua ($+1$) + polinômio quadrático ($+2$) + fator $A$ ($+[n(A)-1]$) + interação $A\!:\!B$ ($+[n(A)-1][n(B)-1]$). Logo $gl = n - p$ e $\Delta gl = q - m$.  
> 3. **Armadilha dos dados faltantes (`NA`):** Na presença de `NA`s em alguma covariável $X_k$, o R remove linhas automaticamente apenas quando $X_k$ entra, alterando $n$ entre modelos sequenciais e **invalidando o TRV/ANODEV**! Confira sempre se `Resid. Df` reduz exatamente em $\Delta p$.  
> 4. **Regra de $D_p/(n-p) \approx 1$ (somente se $\phi=1$):** $\approx 1$ indica ajuste ideal; $\gg 1$ superdispersão; $\ll 1$ subdispersão. **Duas exceções:** falha em Bernoulli individual ($m_i=1 \to$ Hosmer-Lemeshow) e em $\phi$ desconhecido (pois $D_p/(n-p) = \hat{\phi}_D$!).  
> 5. **`anova(m)` (Tipo I) vs `car::Anova(m)` (Tipo II):** Tipo I é **sequencial** (a ordem das variáveis altera todas as linhas intermediárias); Tipo II é **parcial/marginal** (independe da ordem!).

---

## 1.6 Estimação Completa de $\hat{\phi}$ e Inferência com $\phi$ Desconhecido (Slides 42–50)

Para algumas distribuições da família exponencial, o parâmetro de dispersão é fixo e conhecido por definição teórica:
- **$\phi = 1$:** Distribuição Binomial (e Bernoulli), Distribuição de Poisson e Distribuição Binomial Negativa.

Para outras distribuições, $\phi$ é desconhecido e precisa ser estimado:
- **Normal:** $\phi = \sigma^2$
- **Gama:** $\phi = \nu^{-1}$ (onde $\nu$ é o parâmetro de forma)
- **Normal Inversa:** $\phi = \sigma^2$

### Os 3 Métodos para Estimar $\hat{\phi}$ (Slides 43–48)
1. **Método do Desvio ($\hat{\phi}_D$):**  
   Como $D^* = \frac{D_p}{\phi} \sim_a \chi^2_{n-p}$, sob bom ajuste $E(D^*) = n - p$. Igualando a estatística à sua esperança ($n - p = D_p / \hat{\phi}$):
   $$\hat{\phi}_D = \frac{D_p}{n - p}$$
   *(Limitação: em modelos assimétricos como Gama, observações muito próximas de zero tornam $D_p$ sensível.)*

2. **Método de Pearson ($\hat{\phi}_P$ — Mais utilizado e padrão no `summary.glm()` do R):**  
   Baseado na estatística de Pearson generalizada: $\frac{\chi_p^2}{\phi} \sim_a \chi^2_{n-p}$.
   $$\hat{\phi}_P = \frac{1}{n - p} \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{V(\hat{\mu}_i)} = \frac{\chi_p^2}{n - p}$$
   *(Menos sensível a observações próximas de zero na Gama, usando apenas $\hat{\mu}_i$ e $V(\hat{\mu}_i)$.)*

3. **Método da Máxima Verossimilhança ($\hat{\phi}_{MV}$):**  
   Obtido resolvendo a equação escore $U_\phi = \frac{\partial \ell(\boldsymbol{\beta}, \phi)}{\partial \phi} = 0$:
   - **Normal e Normal Inversa (Slide 47):** $\hat{\phi}_{MV} = \frac{D_p}{n}$ *(viesado em amostras pequenas por dividir por $n$ e não por $n-p$)*.
   - **Gama — Aproximação de Cordeiro & McCullagh (1991) (Slide 47)** para pequenos valores de $\phi$:
     $$\hat{\phi}_{CM} = \frac{2 D_p}{n \left[ 1 + \left( 1 + \frac{2 D_p}{3n} \right)^{1/2} \right]}$$

> **Nota dos Slides:** No caso do **Modelo Normal Linear**, o Método do Desvio e o Método de Pearson coincidem exatamente com o Quadrado Médio dos Resíduos ($QMRes$):
> $$\hat{\phi} = \hat{\sigma}^2 = \frac{\sum_{i=1}^n (y_i - \hat{\mu}_i)^2}{n - p} = QMRes$$

### Propriedade Teórica Crucial — Ortogonalidade $\boldsymbol{\beta} \perp \phi$ (Slide 49)
$$\mathfrak{I}_{\beta\phi}(\theta) = E\left[ \frac{\partial^2 \ell(\beta, \phi)}{\partial \beta \, \partial \phi} \right] = 0 \implies \mathfrak{I}_\theta = \text{diag}\{\mathfrak{I}_{\beta\beta}, \, \mathfrak{I}_{\phi\phi}\}$$
Como a matriz de informação de Fisher é **bloco-diagonal**, $\hat{\boldsymbol{\beta}}$ e $\hat{\phi}$ são assintoticamente independentes e o algoritmo de estimação de $\hat{\boldsymbol{\beta}}$ não depende de conhecer $\phi$.

### ANODEV com Parâmetro de Dispersão Desconhecido: O Teste $F$ (Slides 3 e 50)
Quando $\phi$ é desconhecido, não se compara $\Delta D_p$ com $\chi^2$. Utiliza-se a **Estatística $F$**:
$$F = \frac{(D_m - D_q) / (q - m)}{\hat{\phi}_{\text{maximal}}} = \frac{(D_m - D_q) / (q - m)}{D_{\text{maximal}} / (n - q_{\text{maximal}})} \sim_a F_{q - m, \, n - q_{\text{maximal}}}$$
*(Nota: no Slide 50 aparece $(D^*_m - D^*_q)$ dividido por $D^*_q/(n-q)$, onde $\phi$ cancela igualmente; na saída do R, usa-se diretamente $D_p$.)*

### Vantagens e Regra de Ouro do Teste $F$ (Slide 3 e Slide 50):
1. **Independe de $\phi$:** A razão elimina o parâmetro de dispersão desconhecido.
2. **Regra do Modelo Maximal:** Quando se testa uma sequência hierárquica de modelos encaixados, **recomenda-se aplicar em todos os testes a estimativa $\hat{\phi}$ fornecida pelo modelo maximal (com maior número de termos)**. Modelos intermediários sub-ajustados inflam $D_m$ por omitir variáveis importantes, o que infla $\hat{\phi}$, derruba $F$ e pode esconder termos significantes!

### Exercícios Numéricos Resolvidos Completos (Estilo Prova — Sem Código)

#### Exercício Resolvido 1 ($\phi=1$, Dados Binomiais Agrupados — Gestantes, Lista 4)
Fatores binários *Tempo de gestação*, *Idade da mãe* e *Cigarros/dia* combinam-se em $N = 2\times2\times2 = 8$ subpopulações (6789 nascimentos). A tabela `anova()` traz:
- `NULL`: `Resid. Df = [A]`, `Resid. Dev = 352.271`
- `+ Tempo_gestacao`: `Df = 1`, `Deviance = 342.334`, `Resid. Df = 6`, `Resid. Dev = [B]`, `Pr(>Chi) < 0.001`
- `+ Idade_mae`: `Df = 1`, `Deviance = 6.038`, `Resid. Df = [C]`, `Resid. Dev = 3.898`, `Pr(>Chi) = 0.014`
- `+ Qtde_cigarros`: `Df = 1`, `Deviance = [D]`, `Resid. Df = 4`, `Resid. Dev = 1.488`, `Pr(>Chi) = 0.120`

**Resolução:** `[A] = N - 1 = 8 - 1 = 7` (não 6788!); `[B] = 352.271 - 342.334 = 9.937`; `[C] = 6 - 1 = 5`; `[D] = 3.898 - 1.488 = 2.410` (na lista: $2.411$). Ao nível $\alpha=5\%$, `Tempo` e `Idade` entram ($p < 0.05$) e `Cigarros` fica fora ($p=0.12$). Para o modelo selecionado `Tempo + Idade`: Pseudo-$R^2 = 1 - 3.898/352.271 = \mathbf{98.89\%}$, e como os dados são agrupados ($N=8$ subpopulações grandes) com $\phi=1$, $\frac{D}{N-p} = \frac{3.898}{5} = \mathbf{0.78} \approx 1$ ($p=0.564 > 0.05$), confirmando ótimo ajuste e ausência de superdispersão.

#### Exercício Resolvido 2 ($\phi$ Desconhecido — Gama $n = 60$, Estimação de $\hat{\phi}$ e Teste $F$)
Gama ($\phi=\nu^{-1}$) com $n=60$: $M_0$ (Nulo): $D_p=42.00$; $M_1$ ($+X_1$): $D_p=39.80$; $M_2$ ($+X_2$, fator 3 níveis): $D_p=18.50$; $M_3$ (Maximal, $+X_3$): $D_p=17.90$ e Pearson $X^2_3=16.50$. Críticos: $F_{1,55;0.95}=4.016$; $F_{2,55;0.95}=3.165$.

**Resolução:**
1. **g.l.:** $M_0: gl=59$ ($p=1$); $M_1: gl=58$ ($p=2$); $M_2: gl=56$ ($p=4$, pois fator de 3 níveis soma 2 gl); $M_3: gl=55$ ($p=5$).
2. **Estimativas $\hat{\phi}$ no Maximal ($M_3$):** $\hat{\phi}_D = 17.90/55 = \mathbf{0.3255}$; $\hat{\phi}_P = 16.50/55 = \mathbf{0.3000}$; Cordeiro & McCullagh $\hat{\phi}_{CM} = \frac{35.80}{60[1+\sqrt{1+35.80/180}]} \approx \mathbf{0.2848}$.
3. **Testes $F$ usando sempre $\hat{\phi}_{\text{max}} = 0.3255$:**
   - $M_1$ vs $M_0$: $F = \frac{(42.00-39.80)/1}{0.3255} = \mathbf{6.760} > 4.016$ ($p=0.0119 < 0.05$) $\implies$ $X_1$ entra! *(Se errasse usando $\hat{\phi}_1 = 39.80/58 = 0.6862$, daria $F=3.206, p=0.0786$, escondendo $X_1$!)*
   - $M_2$ vs $M_1$: $F = \frac{(39.80-18.50)/2}{0.3255} = \mathbf{32.723} > 3.165$ ($p < 0.0001$) $\implies$ $X_2$ entra!
   - $M_3$ vs $M_2$: $F = \frac{(18.50-17.90)/1}{0.3255} = \mathbf{1.844} < 4.016$ ($p=0.1801$) $\implies$ $X_3$ sai.
4. **Final ($M_2$):** Pseudo-$R^2 = 1 - 18.50/42.00 = \mathbf{55.95\%}$; após escalonar por $\hat{\phi}_{\text{max}}$, $D^*_2 = 18.50/0.3255 = \mathbf{56.84} \approx gl_2 = 56$.

---

## 1.7 Algoritmos de Seleção de Covariáveis (*Forward*, *Backward*, Combinado)

Os métodos automatizados são ferramentas para encontrar modelos parcimoniosos, comparando passos via TRV, AIC ou BIC:

1. **Método Forward (Para a frente):**
   - Inicia no **Modelo Nulo** (apenas intercepto).
   - A cada passo, avalia todas as variáveis fora do modelo e insere aquela que proporciona o maior ganho de ajuste (ex.: menor AIC ou maior redução significativa de deviance).
   - Encerra quando nenhuma variável restante produzir ganho estatisticamente significativo.

2. **Método Backward (Para trás):**
   - Inicia no **Modelo Maximal** (com todos os regressores, fatores e interações de interesse).
   - A cada passo, exclui o termo com menor contribuição para o ajuste (cuja exclusão resulte no menor impacto ou menor AIC).
   - Encerra quando a retirada de qualquer termo piorar significativamente o ajuste.

3. **Método Combinado / Stepwise:**
   - Inicia com o modelo com todos os termos (ou a partir de um modelo nulo com inclusão).
   - Combina as duas abordagens: a cada nova exclusão/inclusão, reavalia a possibilidade de reverter decisões anteriores (variáveis que entraram podem se tornar redundantes e sair; variáveis que saíram podem voltar a ser relevantes).
   - Encerra quando nenhum termo fora tiver força para entrar e nenhum dentro for fraco para sair.

---

# PARTE 2: DIAGNÓSTICOS GERAIS DO MLG

## 2.0 Robustez dos MLGs, Tríade Assintótica Completa e IC via Deviance (Slides 20–27, 90–95)

Antes de avaliar resíduos, três pilares teóricos de inferência do curso (`mlgp2.pdf`, Slides 20–27) costumam aparecer em questões discursivas:

1. **Robustez dos MLGs quanto à especificação incorreta (Slide 20):**
   - Os estimadores $\hat{\boldsymbol{\beta}}$ permanecem **consistentes** ainda que a distribuição especificada para $Y_i$ esteja incorreta, **desde que o preditor linear $\eta_i$ e a função de ligação $g(\mu_i)$ estejam corretos**.
   - **Consequência crítica:** Ao assumir uma distribuição incorreta, a função de variância $V(\mu_i)$ também estará errada; logo, $\widehat{\text{Var}}(\hat{\boldsymbol{\beta}}) = (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X})^{-1}\hat{\phi}$ (erros-padrão, intervalos de confiança e $p$-valores dos testes) **estará incorreta**!
2. **A Tríade Assintótica de Testes de Hipóteses ($H_0: \boldsymbol{\beta}_r = \mathbf{0}$, Slides 21–25):**
   - **Teste da Razão de Verossimilhanças (TRV / $\Delta D^*$):** compara o modelo restrito (sob $H_0$) e o irrestrito ($H_1$); é invariante a reparametrizações, mas exige ajustar ambos os modelos.
   - **Teste de Wald ($W$):** exige apenas o modelo irrestrito; $W = \hat{\boldsymbol{\beta}}_r^\top [\widehat{\text{Var}}(\hat{\boldsymbol{\beta}}_r)]^{-1}\hat{\boldsymbol{\beta}}_r \sim_a \chi^2_r$. Pode ser usado em modelos com sobredispersão, mas **não é invariante** a transformações não lineares dos parâmetros.
   - **Teste Escore de Rao ($SR$):** avalia a inclinação da log-verossimilhança sob $H_0$; **grande vantagem prática:** requer apenas o estimador **restrito** (não precisa ajustar o modelo completo!), e assim como Wald pode ser aplicado em modelos com sobredispersão pois depende apenas dos momentos de 1ª e 2ª ordem.
3. **Intervalo de Confiança Baseado na Deviance vs. IC de Wald (Slides 27, 90–95):**
   - Enquanto o IC de Wald assume aproximação quadrática simétrica ($\hat{\theta} \pm z_{1-\alpha/2} EP(\hat{\theta})$), o **IC baseado na Deviance (Perfil de Verossimilhança)** reúne todos os valores de $\theta$ tais que:
     $$L(\theta) \ge r \, L(\hat{\theta}) \iff D(\theta) = -2[\ell(\theta) - \ell(\hat{\theta})] \le c^* = -2\ln(r) = \chi^2_{1; 1-\alpha}$$
   - **Vantagens:** é invariante a transformações monótonas de $\theta$, capta assimetria natural da verossimilhança e tem melhor cobertura em amostras pequenas ou parâmetros próximos à fronteira. **Desvantagem:** em geral não possui expressão fechada, exigindo métodos numéricos para encontrar as raízes.

---

## 2.1 Por que o Resíduo Ordinário não é adequado em MLG?

O resíduo ordinário clássico da regressão linear simples é dado por:
$$e_i = y_i - \hat{\mu}_i$$
Em MLG, **o resíduo ordinário NÃO é de interesse diagnóstico** pelas seguintes razões:
1. **Heterocedasticidade intrínseca:** Na família exponencial, a variância não é constante; ela depende da média através da função de variância: $\text{Var}(Y_i) = \phi\,V(\mu_i)$ (ou $\phi^{-1}V(\mu_i)$ na notação de precisão). Portanto, mesmo em um modelo perfeito, os resíduos $e_i$ terão variâncias desiguais para cada observação.
2. **Assimetria amostral:** A distribuição da variável resposta (ex.: Binomial, Poisson, Gama) costuma ser marcadamente assimétrica. Resíduos ordinários herdam essa assimetria, impedindo a inspeção visual padrão baseada na normalidade.

### 🚨 Atenção Crítica à Dupla Notação de $\phi$ em Diagnósticos (`mlgp2.pdf`):
- **Slides 31–39 (Notação Gilberto Paula — $\phi$ como *precisão*):** escreve $\text{Var}(Y_i) = \phi^{-1} V(\mu_i)$, logo $\phi = 1/\phi_{\text{R}}$ e $\hat{\phi}^{1/2}$ multiplica o **numerador** dos resíduos padronizados ($r_i^{P*}, r_i^{DP}, r_i^A$) e $\phi$ multiplica $LD_i$.
- **Slide 40 (Williams 1987) e software R ($\phi$ como *dispersão*):** escreve $\text{Var}(Y_i) = \phi\, V(\mu_i)$, logo $\hat{\phi}$ entra no **denominador** ($r_{Pi} = \frac{y_i-\hat{\mu}_i}{\sqrt{\hat{\phi} V(\hat{\mu}_i)(1-h_{ii})}}$ e $D_i = \frac{(\hat{\boldsymbol{\beta}}-\hat{\boldsymbol{\beta}}_{(i)})^\top(\mathbf{X}^\top\hat{\mathbf{W}}\mathbf{X})(\hat{\boldsymbol{\beta}}-\hat{\boldsymbol{\beta}}_{(i)})}{p\,\hat{\phi}}$).
- *Para Binomial ($\phi=1$) e Poisson ($\phi=1$), as duas convenções são numericamente idênticas!* Para Gama e Normal, observe sempre em qual notação a questão definiu $\phi$.

### Propriedades Ideais de um Resíduo em MLG:
- Esperança nula: $E(r_i) = 0$.
- Variância constante (homocedasticidade): $\text{Var}(r_i) \approx \text{constante}$ (idealmente igual a 1).
- Distribuição aproximadamente simétrica / normal.
- Covariância nula entre observações distintas: $\text{Cov}(r_i, r_j) = 0, \forall i \ne j$.

---

## 2.2 Resíduos de Pearson, Anscombe ($r_i^A$) e a Matriz Chapéu ($\mathbf{H}$)

### Resíduo de Pearson Bruto ($r_i^P$)
Divide o erro ordinário pelo desvio padrão da distribuição teórica associada (a menos de $\phi$):
$$r_i^P = \frac{y_i - \hat{\mu}_i}{\sqrt{V(\hat{\mu}_i)}}$$
Na forma matricial:
$$\mathbf{r}^P = \hat{\mathbf{V}}^{-1/2} (\mathbf{y} - \hat{\boldsymbol{\mu}})$$

- **Para Poisson:** $V(\mu) = \mu \implies r_i^P = \frac{y_i - \hat{\mu}_i}{\sqrt{\hat{\mu}_i}}$
- **Para Binomial com $n_i$ ensaios (proporção):** $V(\mu) = \frac{\pi(1-\pi)}{n_i} \implies r_i^P = \frac{y_i - \hat{\pi}_i}{\sqrt{\hat{\pi}_i(1 - \hat{\pi}_i)/n_i}}$  
  (Se $y_i$ for a contagem absoluta de sucessos: $r_i^P = \frac{y_i - n_i \hat{\pi}_i}{\sqrt{n_i \hat{\pi}_i(1 - \hat{\pi}_i)}}$).

A soma dos quadrados dos resíduos de Pearson brutos resulta na estatística de Pearson: $\sum_{i=1}^n (r_i^P)^2 = X^2$.

### A Matriz de Projeção / Chapéu ($\hat{\mathbf{H}}$) em MLG (Slide 32 & Williams 1987)
No algoritmo de estimação por Mínimos Quadrados Reponderados Iterativos (MQRP / Escore de Fisher):
$$\hat{\mathbf{H}} = \hat{\mathbf{W}}^{1/2} \mathbf{X} (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X})^{-1} \mathbf{X}^\top \hat{\mathbf{W}}^{1/2}$$
onde $\hat{\mathbf{W}} = \text{diag}(w_1, \dots, w_n)$ possui **peso geral**:
$$w_i = \left[ V(\mu_i) \left( \frac{d\eta_i}{d\mu_i} \right)^2 \right]^{-1} = \left( \frac{d\mu_i}{d\eta_i} \right)^2 \frac{1}{V(\mu_i)}$$
*(Apenas quando a função de ligação é **canônica**, vale $\frac{d\mu_i}{d\eta_i} = V(\mu_i)$ e a fórmula simplifica para $w_i = V(\mu_i)$!)*

O $i$-ésimo elemento da diagonal principal da matriz $\hat{\mathbf{H}}$, denotado por $\hat{h}_{ii}$, é a **alavanca (*leverage*)** da $i$-ésima observação.
Propriedades fundamentais de $\hat{\mathbf{H}}$:
- **Limites:** Com intercepto no modelo, $\frac{1}{n} \le h_{ii} \le 1$.
- **Traço e média:** $\text{tr}(\hat{\mathbf{H}}) = \sum_{i=1}^n h_{ii} = p \implies \bar{h} = \frac{p}{n}$.
- **Ponto de alta alavanca:** observações onde $h_{ii} > \frac{2p}{n}$ (ou $\frac{3p}{n}$).
- **🚨 Diferença crucial vs. Modelo Linear Normal (Williams, 1987):** Na regressão normal clássica, $\mathbf{H} = \mathbf{X}(\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top$ depende **apenas de $\mathbf{X}$**. Em MLG, como $w_i$ depende de $\hat{\mu}_i$, a alavanca **$h_{ii}$ depende tanto de $\mathbf{X}$ quanto da resposta observada $Y$**! Conforme mostra a Tabela 1 de Williams (dados de Irwin, 1937), uma observação em um extremo de $X$ (ex.: menor dose $x_1$) pode ter alavanca $h_{11}$ *baixa* se seu peso $w_1$ for muito pequeno (em Logística, $w_i = m_i\hat{\pi}_i(1-\hat{\pi}_i) \to 0$ quando $\hat{\pi}_i \to 0$ ou $1$).

### Variância do Resíduo de Pearson e Resíduo de Pearson Padronizado ($r_i^{P*}$)
Na convenção de precisão ($\text{Var}(Y_i)=\phi^{-1}V(\mu_i)$): $\text{Var}(\mathbf{r}^P) = \phi^{-1}(\mathbf{I} - \hat{\mathbf{H}}) \implies \text{Var}(r_i^P) = \phi^{-1}(1 - h_{ii})$.  
Na convenção de dispersão ($\text{Var}(Y_i)=\phi\,V(\mu_i)$): $\text{Var}(r_i^P) = \phi(1 - h_{ii})$.  
Padronizando para média $0$ e variância aproximada $1$:
$$r_i^{P*} = \underbrace{\frac{\hat{\phi}_{\text{prec}}^{1/2} (y_i - \hat{\mu}_i)}{\sqrt{\hat{V}_i (1 - \hat{h}_{ii})}}}_{\text{Slides 31–39 (Paula)}} = \underbrace{\frac{y_i - \hat{\mu}_i}{\sqrt{\hat{\phi}_{\text{disp}} \hat{V}_i (1 - \hat{h}_{ii})}}}_{\text{Slide 40 (Williams) e R}}$$

### Resíduo de Anscombe ($r_i^A$, Slide 35)
Proposto para tornar a distribuição do resíduo o mais simétrica/normal possível sem calcular diretamente a deviance, transformando $y_i$ e $\hat{\mu}_i$ por $\psi(\mu) = \int_0^\mu V^{-1/3}(t)\,dt$:
$$r_i^A = \frac{\psi(y_i) - \psi(\hat{\mu}_i)}{\hat{V}_i^{1/2} \, \psi'(\hat{\mu}_i)} \cdot \phi_{\text{prec}}^{1/2} \qquad \left(\text{ou } \div \sqrt{\phi_{\text{disp}}}\right)$$

| Distribuição | $V(\mu)$ | Transformação $\psi(y)$ | Forma do Resíduo de Anscombe |
| :--- | :---: | :---: | :--- |
| **Normal** | $1$ | $y$ | $y_i - \hat{\mu}_i$ |
| **Binomial** | $\mu(1-\mu)$ | $B\!\left(\frac{y}{m}; \frac{2}{3}, \frac{2}{3}\right)$ | Baseado na função Beta incompleta |
| **Poisson** | $\mu$ | $y^{2/3}$ | $\frac{3}{2}\frac{y_i^{2/3} - \hat{\mu}_i^{2/3}}{\hat{\mu}_i^{1/6}}$ |
| **Gama** | $\mu^2$ | $y^{1/3}$ | $3\left[(y_i/\hat{\mu}_i)^{1/3} - 1\right]$ (vezes $\hat{\nu}^{1/2}$) |
| **Normal Inversa** | $\mu^3$ | $\ln(y)$ | $\frac{\ln(y_i) - \ln(\hat{\mu}_i)}{\hat{\mu}_i^{1/2} \hat{\sigma}}$ |

*(Nota: o resíduo de Anscombe numérico coincide frequentemente com o resíduo deviance, mas requer funções especiais na Binomial e na forma bruta não incorpora $1-h_{ii}$.)*

---

## 2.3 Resíduos do Desvio ($r_i^{DP}$) e Resíduo de Verossimilhança ($r_{Gi}$, Williams 1987)

Os resíduos mais recomendados na prática de MLG baseiam-se na contribuição individual de cada observação para a função desvio e para a redução da deviance sob deleção de caso único.

### Resíduo Deviance Bruto ($r_i^D$ ou $d_i$)
Lembrando que a deviance não-escalonada satisfaz $D_p = \sum_{i=1}^n d_i^2$, define-se:
$$d_i = r_i^D = \text{sinal}(y_i - \hat{\mu}_i) \sqrt{d(y_i; \hat{\mu}_i)}$$
onde $d(y_i; \hat{\mu}_i)$ é o componente individual do desvio não-escalonado. O termo $\text{sinal}(y_i - \hat{\mu}_i)$ garante desvios positivos acima da média ajustada e negativos abaixo dela.

### Resíduo Deviance Padronizado ($r_i^{DP}$, $t_{Di}$ em Paula / $r_{Di}$ em Williams)
Ajusta pela alavanca $h_{ii}$ e pelo parâmetro de dispersão $\phi$:
$$r_i^{DP} = \text{sinal}(y_i - \hat{\mu}_i) \frac{\sqrt{\hat{\phi}_{\text{prec}} \, d(y_i; \hat{\mu}_i)}}{\sqrt{1 - \hat{h}_{ii}}} = \text{sinal}(y_i - \hat{\mu}_i) \sqrt{\frac{d(y_i; \hat{\mu}_i)}{\hat{\phi}_{\text{disp}}(1 - \hat{h}_{ii})}}$$

### Resíduo de Verossimilhança ($r_{Gi}$, Williams 1987 — Slides 101–102)
Ao deletar uma observação, a redução aproximada na deviance escalonada ($G_i$) combina os **resíduos padronizados** de deviance e de Pearson. Tomando a raiz com sinal obtém-se o **Resíduo de Verossimilhança ($r_{Gi}$)**:
$$r_{Gi} = \text{sinal}(y_i - \hat{\mu}_i)\sqrt{(1 - h_{ii})(r_i^{DP})^2 + h_{ii}(r_i^{P*})^2}$$
- **Por que $r_{Gi}$ e $r_i^{DP}$ superam $r_i^{P*}$?** Williams (1984, 1987) demonstrou via simulações de Monte Carlo que:
  1. $r_i^{P*}$ permanece assimétrico mesmo em grandes amostras;
  2. $r_i^{DP}$ incorpora a forma da distribuição da família exponencial e aproxima-se muito melhor de $N(0, 1)$;
  3. $r_{Gi}$ situa-se entre $r_i^{DP}$ e $r_i^{P*}$ (ficando próximo de $r_i^{DP}$ pois $\bar{h}=p/n$ é pequeno) e mede diretamente a contribuição do caso $i$ na detecção de *outliers* ($r_{Gi}^2 \approx G_i$).

---

## 2.4 Análise Gráfica dos Resíduos: Padrões e Falhas Estruturais

A professora Rosemeire destaca quatro gráficos fundamentais com resíduos deviance padronizados ($t_{Di}$ ou $r_i^{DP}$, Slides 34–35):

1. **$t_{Di}$ vs. Índice / Ordem das Observações ($i = 1, \dots, n$):**
   - **Objetivo:** Identificar independência dos erros, ordem de coleta e *outliers* isolados.
   - **Padrão Adequado:** Dispersão aleatória em torno de $0$, com quase todos os pontos contidos na faixa $[-2, 2]$.
   - **Falhas:** Pontos além de $\pm 2$ ou $\pm 3$ indicam *outliers*; tendências cíclicas ou em onda denunciam **autocorrelação / dependência serial**.

2. **$t_{Di}$ vs. Valores Ajustados ($\hat{\eta}_i$ ou $\hat{\mu}_i$, Slide 34 sub-slide 21):**
   - **Objetivo:** Verificar simultaneamente a adequação da **função de ligação**, da **função de variância $V(\mu)$**, **constância da dispersão**, **ausência de correlação** e presença de ***outliers***.
   - **Padrão Adequado:** Faixa de amplitude constante em torno de zero (sem curvatura nem abertura em cone).
   - **Falhas:** Padrão em funil $\implies$ dispersão não constante ou $V(\mu)$ incorreta; curvatura em "U" $\implies$ função de ligação incorreta ou termos omissos.
   - **🚨 Caso Particular de Dados Binários ($y_i \in \{0, 1\}$):** Como $y_i$ só assume $0$ ou $1$, o gráfico exibe **duas curvas paralelas distintas** ($y_i=1$ no arco superior positivo e $y_i=0$ no arco inferior negativo — vide Slide 82!); isso é normal em regressão logística binária e busca-se apenas pontos extremos longe de $[-2, 2]$.

3. **$t_{Di}$ vs. Cada Covariável Explicativa ($X_k$):**
   - **Objetivo:** Checar especificamente a **inclusão/forma funcional de termos no componente sistemático** ($\eta$).
   - **Falhas:** Curvaturas parabólicas revelam efeito não linear ou ausência de $X_k^2$, $\ln(X_k)$ ou *splines*.

4. **Gráfico de Probabilidade Normal / Meio-Normal com Envelopes Simulados:**
   - Avalia conjuntamente a distribuição assumida para $Y_i$, superdispersão e presença de observações discrepantes.

---

## 2.5 Gráficos com Envelopes Simulados e Gráfico Meio-Normal (*Half-Normal Plot*)

Em MLGs, os resíduos geralmente não possuem distribuição normal exata, tornando o gráfico Q-Q Normal convencional difícil de interpretar. A solução metodológica desenvolvida por Atkinson (1981) e recomendada no curso (`envelr_*` / `hnp`) é o uso de **Envelopes Simulados**.

### Algoritmo do Envelope Simulado (Slides 36–38 & 102):
1. Ajusta-se o modelo aos dados originais, obtendo $\hat{\boldsymbol{\beta}}$ e $\hat{\phi}$.
2. Simulam-se $m$ amostras artificiais da variável resposta (tipicamente $m = 19$ ou **$m = 99$ amostras simuladas** nos slides) sob a hipótese de que o modelo ajustado é o verdadeiro gerador dos dados.
3. Para cada uma das $m$ réplicas simuladas, reajusta-se exatamente o mesmo modelo e calculam-se os resíduos padronizados ($r_i^{DP}$ ou $r_{Gi}$).
4. Para cada réplica $j$, ordenam-se os resíduos (ou seus valores absolutos $|r_i|$) em ordem crescente.
5. Para cada posição de ordem $i = 1, \dots, n$, extraem-se os limites do envelope (Slide 38 para 99 simulações: **percentis 5%, 50% [mediana] e 95%**; ou mínimo e máximo em $m=19$).
6. Plota-se o resíduo observado ordenado contra os quantis teóricos sobrepostos às bandas simuladas.

### Gráfico Meio-Normal com Envelope (*Half-Normal Plot*, Slide 38):
Plota os valores absolutos ordenados da medida diagnóstica ($|r_{(i)}|$) contra as esperanças aproximadas das estatísticas de ordem da distribuição meio-normal:
$$\Phi^{-1}\left( \frac{i + 1 - \frac{1}{8}}{2n + \frac{1}{2}} \right), \quad i = 1, 2, \dots, n$$
Como toma módulos, todas as discrepâncias negativas e positivas concentram-se no primeiro quadrante, destacando desvios extremos.

### 🎯 Diagnóstico Visual Obrigatório no Envelope Simulado (Williams 1987, Slide 102):
- **Bom Ajuste:** Os pontos observados permanecem **dentro das bandas simuladas**, mesmo que o traçado não forme uma reta perfeita de $45^\circ$!
- **Apenas os pontos extremos (maiores valores absolutos) escapam acima da banda superior:** indica presença de ***outliers* isolados** (ex.: poucas observações atípicas inflando a deviance).
- **Praticamente TODOS os pontos (pequenos e grandes resíduos) saem sistematicamente da banda:** indica **superdispersão** (ex.: caso *Rotifers* da Lista 5!), **distribuição errada** ou **função de ligação / preditor incorretos**.

---

## 2.6 Pontos de Alavanca (*Leverage*) e Medidas de Influência

Uma observação é dita **influente** quando a sua eliminação isolada do banco de dados provoca **mudanças substanciais nas estimativas dos parâmetros ($\hat{\boldsymbol{\beta}}$)** ou na inferência global (alteração de significância estatística ou sinal).

A abordagem metodológica clássica é a estratégia de **deleção de caso único (*leave-one-out*)**:
- $\hat{\boldsymbol{\beta}}$: estimador com toda a amostra ($n$); $\hat{\boldsymbol{\beta}}_{(i)}$: estimador após excluir a $i$-ésima observação ($n - 1$).
- Para evitar reajustar o modelo $n$ vezes, Pregibon (1981) e Williams (1987) derivaram a **aproximação analítica de um passo (*one-step approximation*)**:
  $$\hat{\boldsymbol{\beta}}_{(i)}^1 = \hat{\boldsymbol{\beta}} - \frac{r_{i}^{P*}}{1 - h_{ii}} \cdot \frac{\sqrt{w_i}}{\hat{\phi}_{\text{disp}}^{-1/2}} \, (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X})^{-1} \mathbf{x}_i \quad \Big(\text{Paula, Slide 39: } \hat{\boldsymbol{\beta}} - \frac{\hat{\phi}_{\text{prec}}^{-1/2} r_i^P \sqrt{w_i}}{1 - h_{ii}} (\mathbf{X}^\top\hat{\mathbf{W}}\mathbf{X})^{-1}\mathbf{x}_i\Big)$$

### 1. Afastamento pela Verossimilhança (*Likelihood Displacement*, $LD_i$, Slide 39)
Avalia a perda de log-verossimilhança ao remover a $i$-ésima observação:
$$LD_i = 2 [ L(\hat{\boldsymbol{\beta}}) - L(\hat{\boldsymbol{\beta}}_{(i)}) ]$$
Expansão de Taylor em torno de $\hat{\boldsymbol{\beta}}$:
- **Notação Precisão (Slide 39):** $LD_i \cong \phi_{\text{prec}} (\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})^\top (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X}) (\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})$.
- **Notação Dispersão (R / Williams):** $LD_i \cong \frac{(\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})^\top (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X}) (\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})}{\phi_{\text{disp}}}$.

### 2. Distância de Cook Generalizada ($D_i$ ou $C_i$) — Williams (1987, Slide 40 & 101–102)
Padroniza a forma quadrática de influência pelo número de parâmetros $p$:
$$D_i = \frac{(\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})^\top (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X}) (\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})}{p \, \phi_{\text{disp}}} \approx \frac{LD_i}{p}$$
Substituindo a aproximação de um passo de Pregibon, obtém-se a relação fundamental com alavanca e resíduo de Pearson padronizado:
$$D_i = \frac{h_{ii}}{p(1 - h_{ii})} (r_i^{P*})^2$$

> **Destaque para a Prova:** A Distância de Cook combina dois fatores multiplicativos:
> 1. **Discrepância na resposta $Y_i$:** $(r_i^{P*})^2$.
> 2. **Posição extrema no espaço das covariáveis ponderadas:** $\frac{h_{ii}}{1 - h_{ii}}$.
> - Pontos de referência usuais de alerta: $D_i > \frac{2p}{n}$ (ou $D_i > \frac{4}{n}$, ou destaque visual isolado no gráfico / $D_i > 0.5$–$1.0$). Toda observação suspeita deve ter sua remoção testada (**refit sem o caso $i$**) para verificar mudança inferencial (ex.: caso **#172**!).

### 3. DFBETAS
Avalia o impacto da retirada da observação $i$ sobre **cada coeficiente individual $\beta_j$**:
$$\text{DFBETAS}_{j(i)} = \frac{\hat{\beta}_j - \hat{\beta}_{j(i)}}{SE(\hat{\beta}_{j(i)})}$$
Permite diagnosticar exatamente qual parâmetro teve sua estimativa ou sinal puxado pelo ponto influente.

---

# PARTE 3: REGRESSÃO LOGÍSTICA

## 3.1 Fundamentos, Estrutura dos Dados e Funções de Ligação

A Regressão Logística é o MLG destinado a modelar uma **variável resposta binária** ($Y \in \{0, 1\}$) ou **binomial** ($Y \sim \text{Binomial}(m_i, \pi_i)$).

### Por que não usar o Modelo Linear de Probabilidade via MCO ($y_i = \mathbf{x}_i^\top \boldsymbol{\beta} + \varepsilon_i$)?
1. **Probabilidades fora de $[0, 1]$:** O preditor linear $\mathbf{x}_i^\top \boldsymbol{\beta}$ varia de $-\infty$ a $+\infty$. O modelo linear comum inevitavelmente produz probabilidades estimadas menores que zero ou maiores que 1.
2. **Heterocedasticidade severa:** $\text{Var}(Y_i \mid \mathbf{x}_i) = \pi(x_i)[1 - \pi(x_i)]$, dependendo criticamente de $\mathbf{x}_i$.
3. **Distribuição não normal:** Os erros só podem assumir dois valores discretos ($1 - \mathbf{x}^\top \boldsymbol{\beta}$ ou $-\mathbf{x}^\top \boldsymbol{\beta}$).

### A Solução via Função de Ligação
Adota-se uma função de distribuição acumulada estritamente crescente $F(\cdot)$ mapeando $(-\infty, +\infty) \to (0, 1)$:
$$\pi(x_i) = F(\mathbf{x}_i^\top \boldsymbol{\beta}) \iff g(\pi_i) = F^{-1}(\pi_i) = \mathbf{x}_i^\top \boldsymbol{\beta}$$

### As Três Funções de Ligação para Dados Binários
1. **Logit (Canônica):**
   $$g(\pi) = \ln\left( \frac{\pi}{1 - \pi} \right) = \mathbf{x}^\top \boldsymbol{\beta} \implies \pi = \frac{e^{\mathbf{x}^\top \boldsymbol{\beta}}}{1 + e^{\mathbf{x}^\top \boldsymbol{\beta}}} = \frac{1}{1 + e^{-\mathbf{x}^\top \boldsymbol{\beta}}}$$
   - Vantagem: Coeficientes diretamente interpretáveis em termos de **Razão de Chances (*Odds Ratio*)**.
   - Simétrica em torno de $\pi = 0.5$.

2. **Probit:**
   $$g(\pi) = \Phi^{-1}(\pi) = \mathbf{x}^\top \boldsymbol{\beta} \implies \pi = \Phi(\mathbf{x}^\top \boldsymbol{\beta}) = \int_{-\infty}^{\mathbf{x}^\top \boldsymbol{\beta}} \frac{1}{\sqrt{2\pi}} e^{-u^2/2} \, du$$
   - Baseada na Normal Padrão. Muito similar ao logit, diferindo apenas nas caudas.

3. **Complemento Log-Log (*Cloglog*):**
   $$g(\pi) = \ln( -\ln(1 - \pi) ) = \mathbf{x}^\top \boldsymbol{\beta} \implies \pi = 1 - e^{-e^{\mathbf{x}^\top \boldsymbol{\beta}}}$$
   - **Assimétrica:** Aproxima-se de 0 muito lentamente e de 1 muito rapidamente.
   - Indicada para modelar eventos com taxas de ocorrência raras ou associada a modelos de tempo até o evento (riscos proporcionais de Cox).

---

## 3.2 Regressão Logística Simples: Equação e Estimação

Para uma única covariável explicativa $X$:
$$\pi(x) = P(Y = 1 \mid X = x) = \frac{e^{\beta_0 + \beta_1 x}}{1 + e^{\beta_0 + \beta_1 x}}$$
Aplicando a transformação logito:
$$\text{logit}[\pi(x)] = \ln\left( \frac{\pi(x)}{1 - \pi(x)} \right) = \beta_0 + \beta_1 x$$

### Verossimilhança e Log-Verossimilhança
Para uma amostra de $n$ observações com $m_i$ ensaios e $y_i$ sucessos:
$$L(\beta_0, \beta_1) = \prod_{i=1}^n \binom{m_i}{y_i} \pi_i^{y_i} (1 - \pi_i)^{m_i - y_i}$$
Tomando o logaritmo natural e omitindo o termo binomial constante:
$$\ell(\beta_0, \beta_1) = \sum_{i=1}^n \left[ y_i \ln\left( \frac{\pi_i}{1 - \pi_i} \right) + m_i \ln(1 - \pi_i) \right] = \sum_{i=1}^n \left[ y_i (\beta_0 + \beta_1 x_i) - m_i \ln(1 + e^{\beta_0 + \beta_1 x_i}) \right]$$

### Equações de Verossimilhança (Não Lineares)
Derivando em relação aos parâmetros e igualando a zero:
$$\frac{\partial \ell}{\partial \beta_0} = \sum_{i=1}^n y_i - \sum_{i=1}^n m_i \frac{e^{\beta_0 + \beta_1 x_i}}{1 + e^{\beta_0 + \beta_1 x_i}} = \sum_{i=1}^n (y_i - m_i \pi_i) = 0$$
$$\frac{\partial \ell}{\partial \beta_1} = \sum_{i=1}^n y_i x_i - \sum_{i=1}^n m_i x_i \frac{e^{\beta_0 + \beta_1 x_i}}{1 + e^{\beta_0 + \beta_1 x_i}} = \sum_{i=1}^n x_i (y_i - m_i \pi_i) = 0$$
Como as equações não possuem solução analítica fechada, a maximização é obtida pelo algoritmo iterativo de **Newton-Raphson / Escore de Fisher**.

---

## 3.3 Interpretação dos Parâmetros e *Odds Ratio* ($OR$)

Este é o ponto mais cobrado em provas teóricas e práticas!

### 1. Chance (*Odds*)
A chance de ocorrência do evento de interesse é a razão entre a probabilidade de ocorrência e a probabilidade de não ocorrência:
$$\text{Chance}(x) = \frac{\pi(x)}{1 - \pi(x)} = e^{\beta_0 + \beta_1 x}$$

### 2. O Intercepto ($\beta_0$)
- $\beta_0$ representa o logaritmo da chance quando $x = 0$: $\ln(\text{Chance} \mid x = 0) = \beta_0$.
- A **chance de base** (para $x=0$) é dada por $e^{\beta_0}$.
- A **probabilidade de base** quando $x=0$ é: $\pi(0) = \frac{e^{\beta_0}}{1 + e^{\beta_0}}$.

### 3. O Coeficiente de Inclinação ($\beta_1$) e o *Odds Ratio* ($OR$)
Se aumentarmos a covariável $x$ em 1 unidade ($x + 1$):
$$\text{Chance}(x + 1) = e^{\beta_0 + \beta_1 (x + 1)} = e^{\beta_0 + \beta_1 x} \cdot e^{\beta_1} = \text{Chance}(x) \cdot e^{\beta_1}$$
A razão de chances (*Odds Ratio*) para um incremento de 1 unidade em $X$ é:
$$OR = \frac{\text{Chance}(x + 1)}{\text{Chance}(x)} = \frac{e^{\beta_0 + \beta_1(x+1)}}{e^{\beta_0 + \beta_1 x}} = e^{\beta_1}$$

- Se $\beta_1 > 0 \implies OR > 1$: O aumento em $X$ **aumenta a chance** do evento. A chance de sucesso é multiplicada por $e^{\beta_1}$ (aumento percentual de $(e^{\beta_1} - 1) \times 100\%$).
- Se $\beta_1 < 0 \implies OR < 1$: O aumento em $X$ **diminui a chance** do evento (redução percentual de $(1 - e^{\beta_1}) \times 100\%$).
- Se $\beta_1 = 0 \implies OR = 1$: Não há associação entre $X$ e a chance de ocorrência do evento.

### 4. Incremento de $c$ unidades, Armadilha de Codificação $\{-1, +1\}$ e Verificação da Linearidade (Slides 62–64)
- **Incremento de $c$ unidades ($X$ contínua):**
  $$OR_c = \frac{\text{Chance}(x + c)}{\text{Chance}(x)} = e^{c \, \beta_1}$$
- **🚨 Armadilha de Codificação Dicotômica (Slide 62):**
  - Se $X \in \{0, 1\}$ (codificação usual *dummy*): $\Delta X = 1 \implies OR = e^{\beta_1}$.
  - Se $X \in \{-1, +1\}$ (codificação por desvio/*effect*): o salto entre grupos é $\Delta X = (+1) - (-1) = 2 \implies OR = e^{2\beta_1}$!
- **Verificação da Premissa de Linearidade no Logito (Slides 63–64 e Lista 4):**
  Como o $OR = e^{c\beta_1}$ assume que a variação de chance por $c$ unidades é a mesma em qualquer faixa de $X$ (ex.: dos 30 aos 40 anos igual aos 50 aos 60 anos), deve-se verificar se a relação é de fato linear no logito por **dois métodos**:
  1. Incluir termos polinomiais ($X^2, X^3$) ou transformações ($\ln X$) e testar sua significância;
  2. Categorizar $X$ contínuo e construir o gráfico do **logito amostral (empírico)** contra os pontos médios das categorias (ou recodificar $X$ em variáveis *dummies*).

> **Exemplo Clássico dos Slides (Doença Coronariana e Idade, Slides 63–64):**  
> Modelo ilustrativo: $\text{logit}[\pi(x)] = -5.13 + 0.11 \times \text{Idade}$.
> - **Incremento unitário ($c = 1$ ano):** $OR = e^{0.11} \approx 1.12$ (a chance aumenta cerca de 12% por ano adicional de idade).
> - **Incremento de 10 anos ($c = 10$):** $OR_{10} = e^{10 \times 0.11} = e^{1.10} \approx 3.00$ (a chance triplica — aumento de ~200% para um salto de 10 anos; nos slides originais há arredondamento ilustrativo com `0.12` resultando em `3.03`).
> - **Sintaxe R para Dados Agrupados (Exemplo Cupons, Slide 65):** $\text{glm}(\text{cbind}(y, t-y) \sim \text{desconto}, \text{family}=\text{binomial})$ é equivalente a $\text{glm}(y/t \sim \text{desconto}, \text{weights}=t, \text{family}=\text{binomial})$.

---

## 3.4 Medidas de Efeito: Risco Atribuível ($d$), Risco Relativo ($RR$) vs. *Odds Ratio* ($OR$) em Tabelas $2 \times 2$

Considere um estudo epidemiológico com um fator de exposição dicotômico ($X = 1$ para Exposto, $X = 0$ para Não Exposto) e a ocorrência de doença ($Y = 1$ para Doente, $Y = 0$ para Sadio):

| Fator | Doença Presente ($Y=1$) | Doença Ausente ($Y=0$) | Total |
| :--- | :---: | :---: | :---: |
| **Exposto ($X=1$)** | $a$ | $b$ | $a + b$ |
| **Não Exposto ($X=0$)** | $c$ | $d$ | $c + d$ |
| **Total** | $a + c$ | $b + d$ | $N = a+b+c+d$ |

### Definições Matemáticas (Slides 67–68)
1. **Probabilidades Condicionais / Riscos:**
   - Risco entre expostos: $\pi_1 = P(Y = 1 \mid X = 1) = \frac{a}{a + b}$
   - Risco entre não expostos: $\pi_0 = P(Y = 1 \mid X = 0) = \frac{c}{c + d}$
2. **Risco Atribuível (diferença absoluta de risco):**
   $$d = \pi_1 - \pi_0 \quad \text{(incremento absoluto no risco atribuível à exposição)}$$
3. **Risco Relativo ($RR$):**
   $$RR = \frac{P(Y = 1 \mid X = 1)}{P(Y = 1 \mid X = 0)} = \frac{\pi_1}{\pi_0} = \frac{a / (a + b)}{c / (c + d)}$$
4. **Razão de Chances (*Odds Ratio*, $OR$):**
   $$OR = \frac{\pi_1 / (1 - \pi_1)}{\pi_0 / (1 - \pi_0)} = \frac{a / b}{c / d} = \frac{a \cdot d}{b \cdot c}$$

### Comparação Fundamental para a Prova: $RR$ vs. $OR$

1. **Dependência do Delineamento Amostral:**
   - Em **Estudos Prospectivos (Coorte / Ensaio Clínico)**, fixam-se os totais de expostos e não expostos ($a+b$ e $c+d$), permitindo estimar diretamente $\pi_1$, $\pi_0$, $d$, $RR$ e $OR$.
   - Em **Estudos Retrospectivos (Caso-Controle)**, fixam-se os totais de doentes e sadios ($a+c$ e $b+d$). **Não é possível estimar diretamente $\pi_1$, $\pi_0$ nem o $RR$**. Contudo, o **$OR = \frac{ad}{bc}$ é simétrico/invariante ao delineamento**, estimando perfeitamente a razão de chances populacional!

2. **Relação Matemática entre $OR$ e $RR$:**
   $$OR = RR \cdot \left( \frac{1 - \pi_0}{1 - \pi_1} \right)$$

3. **A Aproximação de Cornfield / Hipótese do Evento Raro:**
   - Quando a doença é **rara em ambos os grupos** ($\max(\pi_1, \pi_0) < 0.10$, e melhor ainda $< 0.05$):
     $$1 - \pi_1 \approx 1 \quad \text{e} \quad 1 - \pi_0 \approx 1 \implies \frac{1 - \pi_0}{1 - \pi_1} \approx 1 \implies \mathbf{OR \approx RR}$$
   - Se o evento for comum: $RR > 1 \implies OR > RR$ (o $OR$ **amplifica/superestima** a magnitude do $RR$; ex.: caso Titanic com $\pi_M = 74.4\%$, onde $OR = 11.4$ mas $RR = 3.67$).

> **Exemplo Numérico Resolvido dos Slides (Tabagismo e Câncer, Slide 68):**  
> - Não Fumante: Sem Câncer = 80, Com Câncer = 20 (Total = 100) $\implies \pi_0 = \frac{20}{100} = 0.20$
> - Fumante: Sem Câncer = 35, Com Câncer = 15 (Total = 50) $\implies \pi_1 = \frac{15}{50} = 0.30$
> 
> Cálculos:
> - **Risco Atribuível:** $d = 0.30 - 0.20 = 0.10$ (aumento de 10 pontos percentuais no risco absoluto).
> - **Risco Relativo:** $RR = \frac{0.30}{0.20} = 1.50$ (risco 1,5 vezes maior em fumantes).
> - **Chances:** Fumantes $= \frac{0.30}{0.70} \approx 0.43$; Não Fumantes $= \frac{0.20}{0.80} = 0.25$.
> - **Odds Ratio:** $OR = \frac{0.4286}{0.25} = 1.714 \approx 1.72$ (chance de câncer 1,72 vezes maior em fumantes).

---

## 3.5 Regressão Logística Múltipla, Variáveis Categóricas, Interações (Slide 71) e EPV

Com $p$ regressores $X_1, X_2, \dots, X_p$:
$$\text{logit}[\pi(\mathbf{x})] = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \dots + \beta_p x_p$$
- **Interpretação de $\beta_j$:** Variação no log da chance por aumento de 1 unidade em $X_j$, **mantendo todas as demais variáveis constantes (ajuste simultâneo para confundidores)**. $OR_{\text{ajust}} = e^{\beta_j}$.

### Variáveis Categóricas e Fatores com $K$ Níveis
Para um fator com $K$ categorias, constroem-se $K - 1$ variáveis indicadoras (*dummies*), elegendo uma **categoria de referência**.
- **Comparação entre duas categorias não-referência (Slide 87):** Se A tem coeficiente $\beta_2$ e B tem $\beta_3$ (ambas contra C):
  $$\widehat{OR}_{B \mid A} = \frac{e^{\hat{\beta}_3}}{e^{\hat{\beta}_2}} = e^{\hat{\beta}_3 - \hat{\beta}_2}$$

### Modelos com Interação e Tabela de Hipóteses (Slide 71)
Considere Idade ($X_1$) e Sexo ($X_2 = 1$ se Feminino, 0 se Masculino):
$$\text{logit}[\pi] = \beta_0 + \beta_1 \text{Idade} + \beta_2 \text{Sexo} + \beta_3 (\text{Idade} \times \text{Sexo})$$
- Homens ($\text{Sexo} = 0$): $\text{logit}[\pi] = \beta_0 + \beta_1 \text{Idade}$ (inclinação $=\beta_1$).
- Mulheres ($\text{Sexo} = 1$): $\text{logit}[\pi] = (\beta_0 + \beta_2) + (\beta_1 + \beta_3) \text{Idade}$ (inclinação $=\beta_1 + \beta_3$).

| Pergunta / Hipótese Nula ($H_0$) | Interpretação Substantiva | Hipótese Alternativa ($H_1$) |
| :--- | :--- | :--- |
| **$H_0: \beta_3 = 0$** | Efeito da idade independe do sexo (efeitos aditivos / retas paralelas no logito) | $H_1: \beta_3 \ne 0$ (há interação / sinergia) |
| **$H_0: \beta_1 = \beta_3 = 0$** | Idade não está associada à resposta $Y$ | $H_1: \beta_1 \ne 0$ ou $\beta_3 \ne 0$ |
| **$H_0: \beta_2 = \beta_3 = 0$** | Sexo não está associado à resposta $Y$ | $H_1: \beta_2 \ne 0$ ou $\beta_3 \ne 0$ |
| **$H_0: \beta_1 = \beta_2 = \beta_3 = 0$** | Nem idade nem sexo estão associados à resposta $Y$ | $H_1:$ pelo menos um $\beta_j \ne 0$ |

### Cuidados na Construção do Modelo Logístico (Slides 72 & 77)
- **Stepwise com $P_E$ liberal (Slide 72):** Na triagem inicial, usam-se níveis de entrada $P_E \in [0.15, 0.25]$ (ex.: $P_E = P_S = 0.20$ na Lista 4) para não eliminar confundidores relevantes antes da avaliação conjunta e das interações.
- **Regra do EPV (*Events Per Variable*, Slide 77):** Verificar as classificações cruzadas entre resposta e preditores garantindo **pelo menos 10 a 15 sucessos (eventos)** por parâmetro/variável incluída no modelo. Categorias com frequências muito baixas devem ser **reagrupadas** para evitar instabilidade e separação.

---

## 3.6 Inferência: Teste de Wald, TRV e Intervalos de Confiança

### 1. Teste de Wald
Testa a significância individual de um parâmetro $H_0: \beta_j = 0$ vs. $H_1: \beta_j \ne 0$:
$$Z = \frac{\hat{\beta}_j}{SE(\hat{\beta}_j)} \sim N(0, 1) \quad \text{ou} \quad W = Z^2 = \frac{\hat{\beta}_j^2}{\text{Var}(\hat{\beta}_j)} \sim \chi^2_1$$
onde $SE(\hat{\beta}_j)$ é o erro padrão obtido da diagonal da inversa da matriz de informação de Fisher.
- Rejeita-se $H_0$ se $|Z| > z_{1-\alpha/2}$ ou $p\text{-valor} < \alpha$.

### 2. Intervalo de Confiança para $\beta$ e para o $OR$
- **Para o coeficiente $\beta$:**
  $$IC(\beta; 1-\alpha) = \left[ \hat{\beta} - z_{1-\alpha/2} SE(\hat{\beta}), \;\; \hat{\beta} + z_{1-\alpha/2} SE(\hat{\beta}) \right]$$
- **Para o $OR$ ($e^\beta$) com incremento de 1 unidade:**
  $$IC(OR; 1-\alpha) = \left[ e^{\hat{\beta} - z_{1-\alpha/2} SE(\hat{\beta})}, \;\; e^{\hat{\beta} + z_{1-\alpha/2} SE(\hat{\beta})} \right]$$
- **Para o $OR$ ($e^{c\beta}$) com incremento de $c$ unidades:**
  $$IC(OR_c; 1-\alpha) = \left[ e^{c\hat{\beta} - z_{1-\alpha/2} \cdot c \cdot SE(\hat{\beta})}, \;\; e^{c\hat{\beta} + z_{1-\alpha/2} \cdot c \cdot SE(\hat{\beta})} \right]$$

> **Regra Prática de Decisão no IC do OR:**  
> Se o intervalo de confiança de $OR$ **inclui o valor 1.0** (ex.: $IC_{95\%} = [0.92, 5.31]$), conclui-se que o efeito da covariável **não é estatisticamente significativo** ao nível adotado ($p \ge 0.05$).

### 3. Inferência para a Probabilidade Predita $\pi(\mathbf{x})$
Para um indivíduo com características $\mathbf{x}_0$, calcula-se:
$$\hat{\eta}_0 = \mathbf{x}_0^\top \hat{\boldsymbol{\beta}} \implies \text{Var}(\hat{\eta}_0) = \mathbf{x}_0^\top \widehat{\text{Cov}}(\hat{\boldsymbol{\beta}}) \mathbf{x}_0$$
No modelo simples com intercepto $\hat{\alpha}$ e inclinação $\hat{\beta}$:
$$\text{Var}(\hat{\alpha} + \hat{\beta} x_0) = \text{Var}(\hat{\alpha}) + x_0^2 \text{Var}(\hat{\beta}) + 2 x_0 \text{Cov}(\hat{\alpha}, \hat{\beta})$$
A probabilidade pontual é $\hat{\pi}_0 = \frac{e^{\hat{\eta}_0}}{1 + e^{\hat{\eta}_0}}$. O intervalo de confiança de $\pi(\mathbf{x}_0)$ é obtido calculando o IC na escala do logito e aplicando a transformação logística inversa nos limites inferior e superior.

---

## 3.7 Padrões de Covariáveis, Dados Agrupados ($\chi^2_{N-p}$) e o Teste de Hosmer-Lemeshow

### O Conceito de Padrão de Covariáveis (*Covariate Patterns*, Slide 89)
- Observações com exatamente os mesmos valores de todas as covariáveis compartilham um mesmo padrão de covariáveis e mesma probabilidade ajustada $\hat{\pi}_k$.
- **Condições de Validade do Teste $\chi^2_{N-p}$ em Dados Agrupados (Slide 74):** A aproximação assintótica $D \sim \chi^2_{N-p}$ e $X^2 \sim \chi^2_{N-p}$ só é válida quando:
  1. O tamanho da maioria das subpopulações satisfaz $n_i > 10$;
  2. Pelo menos **$80\%$ das frequências esperadas** são $\ge 5$ e **todas** são $> 2$.

### Por que a Deviance e Pearson Falham em Dados Binários Puros ($y \in \{0, 1\}$)?
Esta é uma **pergunta clássica de prova**:
> Quando os dados são **binários individuais** ($m_i = 1$) ou contêm covariáveis contínuas (de modo que $N \approx n$ e $m_i = 1$ para quase todas as linhas), as condições de regularidade assintótica ($m_i \to \infty$ com $N$ fixo) são violadas. Nesses casos, **nem a deviance residual nem a estatística de Pearson seguem distribuição qui-quadrado**, tornando-se inválidas como testes globais de bondade de ajuste!

### O Teste de Hosmer-Lemeshow (1982, 1989 — Slides 75–76)
Criado para contornar essa falha agrupando observações por probabilidades preditas:
1. Ordenam-se as $n$ probabilidades ajustadas: $\hat{\pi}_{(1)} \le \hat{\pi}_{(2)} \le \dots \le \hat{\pi}_{(n)}$.
2. Agrupam-se os indivíduos em $g$ decis de risco (padrão $g = 10$, ou pontos de corte fixos). *(Quando há empates nos valores preditos, observações idênticas podem exigir agrupamento adjacente para evitar grupos muito desiguais.)*
3. Em cada grupo $k$ ($n'_k$ elementos): computam-se $O_k = \sum y_{(j)}$, $\bar{\pi}_k = \frac{1}{n'_k}\sum\hat{\pi}_{(j)}$ e $E_k = n'_k \bar{\pi}_k$.
4. Estatística $\hat{C}$ de Hosmer-Lemeshow:
   $$\hat{C} = \sum_{k=1}^g \frac{(O_k - n'_k \bar{\pi}_k)^2}{n'_k \bar{\pi}_k (1 - \bar{\pi}_k)} \sim_a \chi^2_{g - 2} \qquad (g = 10 \implies gl = 8)$$
- **Hipótese Nula ($H_0$):** O modelo ajusta-se bem aos dados (calibração adequada).
- **Decisão:** $p\text{-valor} > 0.05 \implies$ **não rejeita $H_0$** (bom ajuste / modelo calibrado); $p\text{-valor} \le 0.05 \implies$ falta de ajuste.
- **🚨 Limitações Importantes para a Prova:**
  1. **Arbitrariedade de $g$:** A escolha $g = 10$ decis é convencional e pode alterar levemente o $p$-valor conforme empates ou número de grupos;
  2. **Baixa potência em amostras pequenas:** Em bancos com $n$ pequeno, $\hat{C}$ frequentemente resulta em $p > 0.05$ mesmo havendo má especificação de forma funcional;
  3. Avalia apenas **calibração**, devendo sempre ser complementado pela **Curva ROC / AUC** (discriminação) e pelos **gráficos de resíduos e influência**.

---

## 3.8 Poder Preditivo, Matriz de Confusão (Sensibilidade, Especificidade, Acurácia)

Fixando um limiar de corte (*cut-off* $p_c$, tipicamente $p_c = 0.5$):
- Se $\hat{\pi}_i \ge p_c \implies$ Classifica como Sucesso / Positivo ($\hat{Y}_i = 1$).
- Se $\hat{\pi}_i < p_c \implies$ Classifica como Fracasso / Negativo ($\hat{Y}_i = 0$).

### Matriz de Confusão / Tabela de Classificação

| Observado | Classificado $\hat{Y} = 1$ ($+$) | Classificado $\hat{Y} = 0$ ($-$) | Total Observado |
| :--- | :---: | :---: | :---: |
| **Real $Y = 1$ ($+$)** | **VP** (Verdadeiro Positivo) | **FN** (Falso Negativo) | $VP + FN = n_1$ |
| **Real $Y = 0$ ($-$)** | **FP** (Falso Positivo) | **VN** (Verdadeiro Negativo) | $FP + VN = n_0$ |
| **Total Predito** | $VP + FP$ | $FN + VN$ | $N$ |

### Métricas Fundamentais e Fórmulas:
1. **Sensibilidade ($S$ ou $Se$, Taxa de Verdadeiros Positivos - TVP):**  
   Capacidade do modelo de classificar corretamente quem realmente teve o evento:
   $$Se = \frac{VP}{VP + FN} = \frac{VP}{n_1}$$
2. **Especificidade ($E$ ou $Sp$, Taxa de Verdadeiros Negativos - TVN):**  
   Capacidade do modelo de classificar corretamente quem não teve o evento:
   $$Sp = \frac{VN}{FP + VN} = \frac{VN}{n_0}$$
3. **Taxa de Falsos Positivos ($1 - Sp$):**
   $$1 - Sp = \frac{FP}{FP + VN} = \frac{FP}{n_0}$$
4. **Taxa de Falsos Negativos ($1 - Se$):**
   $$1 - Se = \frac{FN}{VP + FN} = \frac{FN}{n_1}$$
5. **Acurácia (Valor Preditivo Global do Modelo):**  
   Proporção total de classificações corretas:
   $$\text{Acurácia} = \frac{VP + VN}{N}$$

> **Exemplo Numérico Resolvido dos Slides (Slide 78):**  
> Tabela com $p_c = 0.5$:
> - Observado $Y=1$: 31 classificados como 1, 10 classificados como 0 (Total = 41)
> - Observado $Y=0$: 12 classificados como 1, 25 classificados como 0 (Total = 37)
> - Total de indivíduos: $41 + 37 = 78$
> 
> Resultados:
> - $\text{Sensibilidade} = \frac{31}{41} = 0.7561$ (75,61%)
> - $\text{Especificidade} = \frac{25}{37} = 0.6757$ (67,57%)
> - $\text{Falsos Positivos} = \frac{12}{37} = 0.3243$ ($1 - Sp$)
> - $\text{Falsos Negativos} = \frac{10}{41} = 0.2439$ ($1 - Se$)
> - $\text{Acurácia} = \frac{31 + 25}{78} = \frac{56}{78} = 0.7179$ (71,79%)

### O Trade-off do Ponto de Corte (Slide 78, Sub-slide 29):
- **Limiar baixo ($p_c \to 0$):** O modelo classifica quase todos como sucesso. A **sensibilidade sobe para 1**, mas a **especificidade cai para 0** (alta taxa de falsos positivos).
- **Limiar alto ($p_c \to 1$):** O modelo classifica quase todos como fracasso. A **especificidade sobe para 1**, mas a **sensibilidade cai para 0** (alta taxa de falsos negativos).
- O ponto de corte ótimo equilibra sensibilidade e especificidade conforme o custo de cometer erros falsos positivos versus falsos negativos.

---

## 3.9 Curva ROC (*Receiver Operating Characteristic*) e a Área sob a Curva ($AUC$)

A **Curva ROC** é o gráfico contínuo que traça a **Sensibilidade** (eixo Y) contra **$1 - \text{Especificidade}$** (eixo X) para **todos os possíveis pontos de corte $p_c \in [0, 1]$**.

- **Eixo Y:** Taxa de Verdadeiros Positivos ($Se$)
- **Eixo X:** Taxa de Falsos Positivos ($1 - Sp$)
- **Ponto Ideal:** Coordenada $(0, 1)$ no canto superior esquerdo (100% de sensibilidade e 0% de falsos positivos).
- **Linha Diagonal de 45°:** Representa um modelo aleatório sem nenhum poder preditivo ($AUC = 0.50$).

### Área sob a Curva ROC ($AUC$ / Estatística $c$)
Mede a probabilidade de que o modelo atribua uma probabilidade prevista maior para um indivíduo sorteado ao acaso que teve o evento ($Y=1$) do que para um indivíduo sorteado ao acaso que não teve o evento ($Y=0$).

### Tabela de Classificação do AUC (Grading Guidelines, Slide 79, Sub-slide 32):
| Faixa de AUC | Poder de Discriminação | Conceito |
| :---: | :---: | :---: |
| **$0.90 \le AUC \le 1.00$** | **Excelente discriminação** | **A** |
| **$0.80 \le AUC < 0.90$** | **Boa discriminação** | **B** |
| **$0.70 \le AUC < 0.80$** | **Discriminação aceitável / razoável** | **C** |
| **$0.60 \le AUC < 0.70$** | **Discriminação pobre / fraca** | **D** |
| **$0.50 \le AUC < 0.60$** | **Falha de discriminação** (não melhor que o acaso) | **F** |

---

## 3.10 Diagnósticos Específicos em Regressão Logística e Eventos Raros

### Gráficos Diagnósticos Típicos em Logística (Slide 82):
1. **Alavanca ($h_{ii}$) vs. Valor Ajustado ($\hat{\pi}_i$):**  
   Identifica observações com combinações raras de covariáveis.
2. **Resíduo do Desvio ($t_{Di}$) vs. Valor Ajustado ($\hat{\pi}_i$):**  
   Gera **duas curvas distintas características** decorrentes da resposta ser 0 ou 1:
   - A curva superior corresponde aos casos com $y_i = 1$.
   - A curva inferior corresponde aos casos com $y_i = 0$.
   - Observações fora da faixa $[-2, 2]$ representam casos em que o modelo previu alta probabilidade e ocorreu fracasso, ou previu baixa probabilidade e ocorreu sucesso.
3. **Distância de Cook vs. Índice:**  
   Destaca pontos que puxam o hiperplano logístico de forma isolada.

### O Problema dos Eventos Raros (Slides 84 e 88):
Quando o evento de interesse tem um número ínfimo de sucessos (ex.: 20 sucessos em 1.000 observações, 200 em 10.000, ou 2.000 em 100.000):
- **Consequências:** Ocorrência de **viés nas estimativas** e/ou **não convergência** (separação completa ou quase-completa, onde $\hat{\beta} \to \pm\infty$ e erros-padrão explodem).
- **As Três Abordagens Corretivas dos Slides:**
  1. **Logística Exata:** Computacionalmente intensiva; indicada para tamanho amostral $N < 200$ e preferencialmente preditoras dicotômicas.
  2. **Correção de Viés (*Rare Events Logistic*):** Reduz o viés de pequenas amostras/eventos raros; implementada no Stata sob o comando `relogit`.
  3. **Máxima Verossimilhança Penalizada (Firth):** Adiciona a penalidade de Jeffreys para gerar estimativas finitas e consistentes; implementada no Stata (`firthlogit` de Coveney) e no R (pacote `logistf`).

---

# PARTE 4: CADERNO DE INTERPRETAÇÃO DE SAÍDAS DO R PARA A PROVA

Como a prova enfatiza a interpretação de saídas de computador, abaixo estão as saídas **reais e literais dos slides e atividades do curso** e o roteiro de decodificação de cada número.

---

## 4.1 Anatomia da Saída `summary(glm(...))`

### Exemplo A — Modelo Logístico Selecionado (Hospital Heliópolis, $n=175$, Slide 39 / pág. 81):
```text
glm(formula = tipo ~ sexo + idade + ff + hl + idade * hl + ff * hl + sexo * ff,
    family = binomial(link = logit), data = dados)

Coefficients:
            Estimate Std. Error z value Pr(>|z|)    
(Intercept) -1.40858    0.93725  -1.503  0.13287    
sexo         1.40234    0.58366   2.403  0.01628 *  
idade        0.03871    0.01728   2.240  0.02507 *  
ff          -1.97754    0.88752  -2.228  0.02587 *  
hl          -5.52155    1.68622  -3.275  0.00106 ** 
idade:hl     0.06165    0.02944   2.094  0.03626 *  
ff:hl        2.90786    1.10370   2.635  0.00842 ** 
sexo:ff     -3.34942    1.47998  -2.263  0.02363 *  

Residual deviance: 146.22  on 167 degrees of freedom
```

### Exemplo B — Dados Agrupados com $N=7$ Subpopulações (Cupons de Desconto, Slide 43 / pág. 66):
```text
Coefficients:
             Estimate Std. Error z value Pr(>|z|)    
(Intercept) -2.535516   0.157355  -16.11   <2e-16 ***
desconto     0.131587   0.007346   17.91   <2e-16 ***
---
Dispersion parameter for binomial family taken to be 1

    Null deviance: 442.2377  on 6  degrees of freedom
Residual deviance:   2.1571  on 5  degrees of freedom
AIC: 43.728
```

### Como Interpretar cada Elemento na Prova:

1. **Coluna `Estimate` ($\hat{\beta}_j$):**  
   - Coeficiente estimado na escala do preditor linear ($\eta = \ln(\text{chance})$).
   - Para variáveis sem interação ou na categoria basal das demais: $OR = e^{\text{Estimate}}$. Exemplo B (`desconto`): $e^{0.131587} \approx 1.14$ (cada unidade de desconto aumenta a chance de uso do cupom em ~14%).
   - **🚨 Cuidado no Exemplo A (`sexo:ff` no modelo):** Como existem interações `idade:hl`, `ff:hl` e `sexo:ff`, **nunca interprete `sexo`, `ff`, `hl` ou `idade` isoladamente** sem especificar o nível da variável que interage (ex.: $\hat{\beta}_{\text{sexo}} = 1.40234 \implies OR = e^{1.40234} \approx 4.06$ é válido apenas quando `ff = 0`!).
2. **Coluna `Std. Error` ($EP(\hat{\beta}_j)$):**  
   - Raiz quadrada da variância assintótica na diagonal de $(\mathbf{X}^\top\hat{\mathbf{W}}\mathbf{X})^{-1}\hat{\phi}$.
3. **Coluna `z value` e `Pr(>|z|)` (Teste de Wald):**  
   - $z = \text{Estimate} / \text{Std. Error}$ (no exemplo A para `idade`: $0.03871 / 0.01728 = 2.240$). Se $p < 0.05$, rejeita-se $H_0: \beta_j = 0$.
4. **Leitura de `Residual deviance` e Graus de Liberdade:**
   - **No Exemplo A (dados binários individuais, $n=175$, $p=8$):** $gl = 175 - 8 = 167$. Mesmo com `Residual deviance = 146.22` e $gl = 167$, **NÃO se usa `146.22` contra $\chi^2_{167}$ como teste global** pois $m_i=1$! Deve-se usar o `hoslem.test`.
   - **No Exemplo B (dados binomiais agrupados em $N=7$ níveis com $200$ cupons cada):** $gl_0 = N - 1 = 7 - 1 = 6$ e $gl = N - p = 7 - 2 = 5$. Como as contagens são grandes e $\phi = 1$, vale testar $D = 2.1571$ contra $\chi^2_5$: $p = P(\chi^2_5 \ge 2.1571) = 0.83 > 0.05$, indicando excelente ajuste; e $R^2 = 1 - 2.1571/442.2377 = \mathbf{99.51\%}$!

---

## 4.2 Leitura da Tabela `anova(ajuste, test = "Chisq")` e Seleção Stepwise

Nos modelos binomiais agrupados da **Lista 4 (Gestantes Fumantes, $N=8$ subpopulações)** e do **Stepwise (Slide 38 / pág. 81)** vemos exatamente os dois tipos de saída cobrados em prova:

### A) Tabela Sequencial ANODEV (Gestantes Fumantes, Lista 4):
```text
Terms added sequentially (first to last)
                 Df Deviance Resid. Df Resid. Dev Pr(>Chi)
NULL                                 7    352.271
Tempo_gestacao    1  342.334         6      9.937  < 0.001 ***
Idade_mae         1    6.038         5      3.898    0.014 *
Qtde_cigarros     1    2.411         4      1.488    0.120
```
- **Como Responder na Prova:**
  - `Resid. Df` do `NULL` é $N - 1 = 8 - 1 = 7$ (nº de subpopulações $- 1$).
  - Cada linha mede $\Delta D = D_{\text{ant}} - D_{\text{atual}}$ ao adicionar um termo sobre os anteriores (`Tempo_gestacao` reduz 342.334; `Idade_mae` reduz 6.038, $p=0.014 < 0.05$; `Qtde_cigarros` reduz apenas 2.411, $p=0.120 \ge 0.05$ e é descartado).
  - Na tabela do **Slide 38 (Etapa 2 do Stepwise — Hospital Heliópolis)**, analisam-se os $p$-valores de inclusão de interações passo a passo: entram `IDA*HL` ($p=0.013$), `HL*FF` ($p=0.027 \to 0.005$) e `SEX*FF` ($p=0.012$ no passo 3), parando no passo 4 quando nenhum $p$-valor restante atinge $P_E$ (`0.208`, `0.403`, `0.794`).

---

## 4.3 Saída do Teste de Hosmer-Lemeshow (`hoslem.test`)

```text
        Hosmer and Lemeshow goodness of fit (GOF) test

data:  ajuste6$y, fitted(ajuste6)
X-squared = 6.8216, df = 8, p-value = 0.556
```
*(E para o modelo principal aditivo `modelComp` no Slide 25 / pág. 77: `X-squared = 7.6435, df = 8, p-value = 0.469`.)*

### Roteiro Obrigatório de Interpretação:
1. **Identificar as Hipóteses:**
   - $H_0$: O modelo ajusta-se bem aos dados observados (calibração adequada).
   - $H_1$: O modelo não se ajusta bem aos dados (falta de ajuste).
2. **Avaliar a Estatística e Graus de Liberdade:**
   - Foram utilizados $g = 10$ decis de risco, gerando $df = g - 2 = 8$ graus de liberdade.
   - A estatística obtida foi $\hat{C} = X^2 = 6.8216$.
3. **Conclusão (e Ressalva Crítica):**
   - Como o $p\text{-valor} = 0.556 > 0.05$, **não rejeitamos $H_0$** ao nível de 5%. Logo, não há evidências estatísticas de falta de calibração (o modelo com 3 interações parece bem ajustado globalmente — embora a análise de influência no paciente #172 ainda vá revelar instabilidade em `sexo:ff`!).

---

## 4.4 Saída da Curva ROC e `logistic.display` (`epiDisplay`, Slides 36–37)

```text
> library(epiDisplay)
> logistic.display(modelComp)
                OR  lower95ci upper95ci    Pr(>|Z|)
factor(sexo)1 2.2128593 0.92260923 5.3074978 7.515743e-02
idade         1.0667060 1.04042273 1.0936533 3.913684e-07
hl            0.2006313 0.08971806 0.4486601 9.158257e-05
ff            0.5411889 0.21777058 1.3449264 1.861897e-01
```
```text
> lroc(modelComp, title=TRUE, auc.coords=c(.5,.1))
$model.description
[1] "tipo ~ factor(sexo) + idade + hl + ff"
$auc
[1] 0.8538732
```

### Roteiro Obrigatório de Interpretação:
- **`logistic.display`:** A coluna `OR` já traz $e^{\hat{\beta}}$ junto ao $IC_{95\%}$. `idade` ($OR=1.067$, IC $[1.040, 1.094]$) e `hl` ($OR=0.201$, IC $[0.090, 0.449]$) são estatisticamente significativos porque seus intervalos **não contêm o valor $1.0$** ($p < 0.05$). Já `factor(sexo)1` ($IC = [0.92, 5.31]$, $p=0.075$) e `ff` ($IC = [0.22, 1.34]$, $p=0.186$) **não são significativos** a 5% porque ambos os intervalos contêm $1.0$!
- **`lroc` ($AUC = 0.8539$):** Segundo Hosmer & Lemeshow ($0.80 \le AUC < 0.90$, grau B), o modelo possui **boa capacidade discriminativa** (85,39% de probabilidade de ranquear um caso maligno com probabilidade predita maior que um benigno sorteados ao acaso).

---

## 4.5 Estudos de Caso Completos Resolvidos dos Slides

### Estudo de Caso 1: Ensaio Clínico de Infecção Urinária (Koch et al., 1985 — Slides 85–87)

**Descrição:** 3 tratamentos (A, B, C) e 2 níveis de diagnóstico de infecção (Complicada vs. Não complicada) com desfecho de Cura (Sim vs. Não).  
- $N = 6$ subpopulações (padrões de covariáveis, também denotado $s = 6$).
- $X_1$: Infecção ($x_1 = 1$ se complicada, 0 se não complicada).
- $X_2$: Tratamento com 3 níveis ($X_{21}$ para A, $X_{22}$ para B, e referência Tratamento C).

#### Tabela 1. Diferenças de Deviances (ANODEV):
| Modelos | g.l. | Deviances | $\Delta$ Deviances | $\Delta$ g.l. | $p$-valor |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Nulo | 5 | 44,473 | — | — | — |
| $X_1$ (Infecção) | 4 | 30,628 | 13,844 | 1 | 0,0002 |
| $X_2 \mid X_1$ (Tratamento) | 2 | 2,515 | 28,114 | 2 | < 0,0001 |
| $X_1 * X_2 \mid X_1, X_2$ | 0 | 0,000 | 2,515 | 2 | 0,2843 |

#### Questões Típicas de Prova sobre este Estudo:
1. **A interação entre diagnóstico e tratamento é significativa?**
   - *Resposta:* Não. O teste da inclusão da interação apresentou $\Delta D = 2.515$ com $\Delta gl = 2$ e $p\text{-valor} = 0.2843 > 0.05$. Portanto, adota-se o modelo aditivo sem interação.
2. **O modelo aditivo final ($X_1 + X_2$) se ajusta bem aos dados?**
   - *Resposta:* Sim. A deviance residual do modelo aditivo é $D = 2.515$ com $2$ graus de liberdade. Como os dados estão agrupados por subpopulações e o teste qui-quadrado de bondade de ajuste fornece $p\text{-valor} = P(\chi^2_2 \ge 2.515) = 0.2844 > 0.05$, não rejeitamos o modelo. As estatísticas $Q_L = 2.515$ ($p = 0.2844$) e $Q_P = 2.757$ ($p = 0.2519$) confirmam excelente ajuste aos dados.
3. **Estimativas e Interpretação dos Parâmetros:**
   - $\hat{\beta}_0 = 1.4184$ (Constante / Não complicada / Tratamento C)
   - $\hat{\beta}_1 = -0.9616$ (Infecção complicada)
   - $\hat{\beta}_2 = 0.5847$ (Tratamento A)
   - $\hat{\beta}_3 = 1.5608$ (Tratamento B)
4. **Cálculo das Razões de Chances:**
   - **Efeito da Infecção:** $\widehat{OR}_{\text{Comp} \mid \text{NãoComp}} = e^{-0.9616} = 0.38$. A chance de cura em pacientes com infecção complicada é de apenas 0,38 vezes a de pacientes com infecção não complicada. Invertendo: $\widehat{OR}_{\text{NãoComp} \mid \text{Comp}} = e^{0.9616} = 2.62$.
   - **Tratamento A vs. C:** $\widehat{OR}_{A \mid C} = e^{0.5847} = 1.79$.
   - **Tratamento B vs. C:** $\widehat{OR}_{B \mid C} = e^{1.5608} = 4.76$. O Tratamento B aumenta a chance de cura em 4,76 vezes em relação ao C.
   - **Comparação direta B vs. A:**  
     $$\widehat{OR}_{B \mid A} = e^{\hat{\beta}_3 - \hat{\beta}_2} = e^{1.5608 - 0.5847} = e^{0.9761} = 2.65$$
     O Tratamento B proporciona uma chance de cura 2,65 vezes maior do que o Tratamento A.

---

### Estudo de Caso 2: Análise de Influência e Exclusão da Observação #172 (Slides 82–84)

**Contexto:** Estudo de caso-controle com 175 pacientes com tumor pulmonar.
- No gráfico da Distância de Cook, a **observação #172** se destacou isoladamente com valor extremo ($D_{172} > 1.5$).
- Perfil do paciente #172: Sexo feminino, 55 anos, processo maligno presente ($Y=1$), mas níveis altos para HL e FF (que correspondiam a um perfil com baixa probabilidade prevista, gerando resíduo deviance negativo acentuado).

#### Comparação do Modelo Com e Sem a Observação #172 (Slide 83, Sub-slide 44):
- **Com a observação #172:**  
  O termo de interação `sexo:ff` tinha $\hat{\beta} = -3.3494$, $SE = 1.47998$, $z = -2.263$ e **$p\text{-valor} = 0.02363$ (significativo)**.
- **Sem a observação #172:**  
  O termo `sexo:ff` passa para $\hat{\beta} = -19.3711$, $SE = 1357.18$, $z = -0.014$ e **$p\text{-valor} = 0.98861$ (não significativo!)**.

#### Conclusão Diagnóstica Mandatória de Prova (Slide 83, Sub-slide 45):
1. A observação #172 é um **ponto altamente influente**.
2. A significância estatística da interação `sexo:ff` era artificialmente sustentada por uma única observação isolada.
3. Ao remover a observação influente, a interação perde totalmente a significância ($p \approx 0.99$).
4. **Decisão:** A interação `sexo:ff` deve ser **eliminada do modelo**, pois sua presença refletia a instabilidade provocada por um único caso atípico.

---

# PARTE 5: FORMULÁRIO DE BOLSO E GUIA DE DECISÃO RÁPIDO

### Tabela-Resumo de Fórmulas Fundamentais

| Conceito | Notação / Fórmula | Distribuição / Uso |
| :--- | :--- | :--- |
| **Deviance Escalonada** | $D^* = 2(\ell_n - \ell_p) = D_p/\phi_{\text{disp}} = \phi_{\text{prec}} D_p$ | $\sim_a \chi^2_{n-p}$ sob $H_0$ |
| **Deviance Não Escalonada** | $D_p = \sum_{i=1}^n d_i^2 = 2\phi_{\text{disp}}(\ell_n - \ell_p)$ | Saída do R! Comparar direto com $\chi^2_{n-p}$ só se $\phi=1$ |
| **Estatística de Pearson** | $X^2 = \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{\widehat{\text{Var}}(y_i)}$ | $\sim_a \chi^2_{n-p}$ sob $\phi$ conhecido (não aditiva) |
| **Diferença de Deviances (TRV)** | $\Delta D^* = D^*_m - D^*_q = 2(\ell_q - \ell_m)$ | $\sim_a \chi^2_{q-m}$ para modelos encaixados |
| **Teste Escore / Wald** | $W = \hat{\boldsymbol{\beta}}_r^\top [\widehat{\text{Var}}(\hat{\boldsymbol{\beta}}_r)]^{-1}\hat{\boldsymbol{\beta}}_r \sim \chi^2_r$ | Escore usa só modelo restrito $H_0$; ambos aceitam $\phi$ |
| **Estatística $F$ ($\phi$ desconhecido)** | $F = \frac{(D_m - D_q)/(q-m)}{D_{\text{max}} / (n-q_{\text{max}})}$ | $\sim F_{q-m, \, n-q_{\text{max}}}$ (sempre usar $\hat{\phi}$ do modelo maximal) |
| **Estimador $\hat{\phi}$ (Pearson)** | $\hat{\phi}_P = \frac{1}{n-p} \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{V(\hat{\mu}_i)}$ | Padrão do `summary.glm()` (dispersão) |
| **Pseudo-$R^2$ / AIC** | $R^2 = 1 - \frac{D_1}{D_0}$; $\quad AIC = -2\ell + 2p$ | Menor AIC é melhor ($\Delta AIC > 2$ forte evidência) |
| **Peso MQRP ($w_i$) & Alavanca** | $w_i = \left[V(\mu_i)\left(\frac{d\eta_i}{d\mu_i}\right)^2\right]^{-1}$; $\frac{1}{n}\le h_{ii}\le 1$ | $\sum h_{ii} = p$; depende de $X$ e $Y$ (Williams 1987) |
| **Resíduo Pearson Padronizado** | $r_i^{P*} = \frac{\hat{\phi}_{\text{prec}}^{1/2}(y_i - \hat{\mu}_i)}{\sqrt{\hat{V}_i(1 - h_{ii})}} = \frac{y_i - \hat{\mu}_i}{\sqrt{\hat{\phi}_{\text{disp}}\hat{V}_i(1 - h_{ii})}}$ | Média 0 e Variância $\approx 1$ |
| **Resíduo Deviance / Verossimilhança** | $r_i^{DP} = \text{sinal}(y_i-\hat{\mu}_i)\sqrt{\frac{d(y_i;\hat{\mu}_i)}{\hat{\phi}_{\text{disp}}(1-h_{ii})}}$ | $r_{Gi} = \text{sinal}(e_i)\sqrt{(1-h_{ii})(r_i^{DP})^2 + h_{ii}(r_i^{P*})^2}$ |
| **Distância de Cook em MLG** | $D_i = \frac{h_{ii}}{p(1 - h_{ii})} (r_i^{P*})^2$ | Medida de influência global (Williams 1987) |
| **Odds Ratio ($OR$)** | $e^{\beta_1}$ ($X\in\{0,1\}$), $e^{2\beta_1}$ ($X\in\{-1,+1\}$), $e^{c\beta_1}$ ($+c$ unid.) | Entre dummies B vs A: $e^{\hat{\beta}_B - \hat{\beta}_A}$ |
| **Risco Atribuível ($d$) e $RR$** | $d = \pi_1 - \pi_0$; $\quad RR = \frac{\pi_1}{\pi_0}$ | $OR \approx RR$ só se $\max(\pi_1,\pi_0) < 0.10$ (evento raro) |
| **IC para o $OR$** | $\exp[\hat{\beta} \pm z_{1-\alpha/2} SE(\hat{\beta})]$ | Não significativo se contiver $1.0$ |
| **Hosmer-Lemeshow ($\hat{C}$)** | $\hat{C} = \sum_{k=1}^g \frac{(O_k - n'_k \bar{\pi}_k)^2}{n'_k \bar{\pi}_k (1 - \bar{\pi}_k)}$ | $\sim_a \chi^2_{g-2}$ ($p > 0.05 \implies$ bom ajuste; $EPV \ge 10\text{–}15$) |
| **Sensibilidade / Especificidade / Acc** | $Se = \frac{VP}{VP + FN}$; $\quad Sp = \frac{VN}{FP + VN}$; $\quad \text{Acc} = \frac{VP+VN}{N}$ | Curva ROC ($Se$ vs $1-Sp$) independe do corte $c$ |

---

### Algoritmo Mental para Resolução de Questões na Prova:

1. **A questão pede teste de qualidade global em dados binários puros ($y \in \{0, 1\}$)?**
   - *Atenção:* NÃO use o $p$-valor da Deviance Residual do `summary`! Responda que a deviance não segue $\chi^2$ para $m_i=1$, e aponte o **Teste de Hosmer-Lemeshow** ou a **Área sob a Curva ROC**.
2. **A questão forneceu uma Tabela ANODEV de modelos encaixados?**
   - Calcule $\Delta D = D_m - D_q$ e $\Delta gl = gl_m - gl_q$.
   - Se $\phi$ for conhecido (Binomial/Poisson), use $\Delta D \sim \chi^2_{\Delta gl}$.
   - Se $\phi$ for desconhecido (Normal/Gama), use a estatística $F = \frac{\Delta D / \Delta gl}{D_{\text{maximal}} / gl_{\text{maximal}}}$.
3. **A questão pede a interpretação de um coeficiente $\beta_j$?**
   - Não se esqueça de frisar: *"Mantendo as demais covariáveis constantes"* (ajuste simultâneo).
   - Especifique a escala: $\beta_j$ é a mudança no **log da chance**, enquanto $e^{\beta_j}$ é a **razão de chances (*Odds Ratio*)**.
4. **A questão traz um fator categórico com dummies?**
   - Identifique quem é a referência (o nível que não aparece na saída com coeficiente próprio).
   - Para comparar duas dummies entre si ($B$ vs. $A$), faça $e^{\beta_B - \beta_A}$.
5. **A questão traz gráficos diagnósticos?**
   - Procure pontos além de $\pm 2$ ou $\pm 3$ no gráfico de resíduos (*outliers*).
   - Procure pontos com $h_{ii} > 2p/n$ no gráfico de alavancas.
   - Verifique se a remoção de pontos com alta distância de Cook altera o sinal ou a significância ($p$-valor) dos coeficientes.


---

# CAPÍTULO 8: BANCO DE EXERCÍCIOS RESOLVIDOS DAS ATIVIDADES DA PROFESSORA ROSEMEIRE (PADRÃO DE RESPOSTA PARA PROVA SEM PROGRAMAÇÃO)

Este capítulo reproduz fielmente as atividades aplicadas pela Profa. Rosemeire L. Fiaccone (Atividade 4, Atividade 5, Atividade 7, Atividade 8 e Listas 4 e 5), apresentando o passo a passo algébrico e a resposta textual rigorosa exigida em avaliações teóricas.

---

### 8.1 CASO 1: Regressão Logística em Transporte Urbano (Atividade em Sala 5)

#### Contexto e Variáveis
Estudo de fatores associados a acidentes com transporte coletivo urbano ($N=300$ motoristas):
* $Y$: Acidente ($1 = \text{sofreu acidente no ano}$; $0 = \text{não sofreu}$)
* $X_1$: $\text{Horas}$ semanais dirigidas (contínua)
* $X_2$: $\text{Treinamento}$ ($1 = \text{fez treinamento adicional}$; $0 = \text{não fez}$)
* $X_3$: $\text{Experiência}$ profissional na função (em anos)

Modelo Logístico Postulado:
$$\log\left(\frac{\pi_i}{1 - \pi_i}\right) = \beta_0 + \beta_1 \text{Horas}_i + \beta_2 \text{Treinamento}_i + \beta_3 \text{Experiência}_i$$

Tabela de Estimativas e Erros-Padrão:
| Covariável | Estimativa ($\hat{\beta}$) | Erro-Padrão ($EP$) | $z_{\text{calc}}$ | $p\text{-valor}$ |
| :--- | :--- | :--- | :--- | :--- |
| Intercepto ($\beta_0$) | -1.800 | 0.500 | -3.60 | 0.0003 |
| HorasDirigidas ($\beta_1$) | +0.030 | 0.008 | +3.75 | 0.0002 |
| Treinamento ($\beta_2$) | -0.700 | 0.250 | -2.80 | 0.0051 |
| Experiência ($\beta_3$) | -0.015 | 0.010 | -1.50 | 0.1336 |

---

#### Questão (a): Significado dos Coeficientes no Contexto do Estudo
* **HorasDirigidas ($\hat{\beta}_1 = +0.030$):**
  $$OR_1 = e^{0.030} \approx 1.0304 \implies (1.0304 - 1) \times 100\% = +3.04\%$$
  *Resposta:* "Fixadas as demais covariáveis, para cada 1 hora a mais dirigida por semana, a chance de ocorrência de acidente com o veículo de transporte urbano aumenta em aproximadamente 3,0%."
* **Treinamento ($\hat{\beta}_2 = -0.700$):**
  $$OR_2 = e^{-0.700} \approx 0.4966 \implies (0.4966 - 1) \times 100\% = -50.34\%$$
  *Resposta:* "Mantendo-se iguais a carga horária e os anos de experiência, os motoristas que realizaram treinamento adicional apresentam uma chance de sofrer acidentes 50,3% menor (ou aproximadamente metade da chance, $OR = 0.50$) em relação aos motoristas sem treinamento."
* **Experiência ($\hat{\beta}_3 = -0.015$):**
  $$OR_3 = e^{-0.015} \approx 0.9851 \implies -1.49\%$$
  *Resposta:* "Para cada ano adicional de experiência como motorista profissional, a chance de envolvimento em acidente reduz-se em cerca de 1,5%."

---

#### Questão (b): Teste de Wald para a Variável Treinamento ($\alpha = 0.05$)
1. **Hipóteses Estatísticas:**
   $$H_0: \beta_2 = 0 \quad \text{vs} \quad H_1: \beta_2 \ne 0$$
2. **Cálculo da Estatística $z$ de Wald:**
   $$z_{\text{calc}} = \frac{\hat{\beta}_2}{EP(\hat{\beta}_2)} = \frac{-0.70}{0.25} = -2.80$$
3. **Decisão ao Nível de 5%:**
   * O valor crítico bicaudal da distribuição Normal Padrão ao nível de 5% é $z_{\text{tab}} = 1.96$.
   * Como $|z_{\text{calc}}| = |-2.80| = 2.80 > 1.96$ (com $p\text{-valor} = 0.0051 < 0.05$), **rejeita-se categoricamente a hipótese nula $H_0$**.
4. **Conclusão no Contexto:**
   > *"Há fortes evidências estatísticas de que a realização de treinamento adicional exerce efeito protetor significativo, reduzindo a chance de acidentes na frota urbana."*

---

#### Questão (c): Estimação da Probabilidade de Acidente
Motorista com $45$ horas semanais dirigidas, $12$ anos de experiência e que realizou treinamento ($\text{Treinamento} = 1$).
1. **Preditor Linear ($\hat{\eta}$):**
   $$\hat{\eta} = -1.80 + 0.030(45) - 0.70(1) - 0.015(12)$$
   $$\hat{\eta} = -1.80 + 1.35 - 0.70 - 0.18 = -1.33$$
2. **Probabilidade Predita ($\hat{\pi}$):**
   $$\hat{\pi} = \frac{e^{-1.33}}{1 + e^{-1.33}} = \frac{0.26447}{1.26447} \approx \mathbf{0.2091} \quad (20,91\%)$$

---

#### Questão (d): Classificação com Limiar de Risco $c = 0.40$
* **Regra:** Se $\hat{\pi} \ge 0.40 \implies \hat{Y} = 1$ (Alto Risco); se $\hat{\pi} < 0.40 \implies \hat{Y} = 0$ (Baixo Risco).
* **Decisão:** Como $\hat{\pi} = 0.2091 < 0.40$, o motorista é classificado como **Classe 0 (Sem Risco de Acidente)**.

---

#### Questão (e): Matriz de Confusão & Métricas Clínicas
Tabela observada em amostra de validação ($N=300$ motoristas):
| | Predito $\hat{Y}=1$ | Predito $\hat{Y}=0$ | Total Real |
| :--- | :---: | :---: | :---: |
| **Real $Y=1$ (Acidente)** | $VP = 45$ | $FN = 30$ | $75$ |
| **Real $Y=0$ (Sem Acidente)** | $FP = 25$ | $VN = 200$ | $225$ |
| **Total Predito** | $70$ | $230$ | $N=300$ |

1. **Acurácia Global:**
   $$\text{Acurácia} = \frac{VP + VN}{N} = \frac{45 + 200}{300} = \frac{245}{300} \approx \mathbf{81.67\%}$$
2. **Sensibilidade (*Recall*):**
   $$\text{Sensibilidade} = \frac{VP}{VP + FN} = \frac{45}{45 + 30} = \frac{45}{75} = \mathbf{60.00\%}$$
3. **Especificidade:**
   $$\text{Especificidade} = \frac{VN}{VN + FP} = \frac{200}{200 + 25} = \frac{200}{225} \approx \mathbf{88.89\%}$$
4. **Comentário Crítico:**
   > *"O modelo exibe acurácia global satisfatória ($81.67\%$) e elevada especificidade ($88.89\%$). No entanto, sua sensibilidade é moderada ($60.00\%$), gerando 30 falsos negativos (40% dos acidentes deixaram de ser previstos pelo modelo com o corte em 0.40). Em gestão de segurança rodoviária, o custo de não prever um acidente é desproporcionalmente elevado, justificando a adoção de um limiar de corte mais rigoroso (menor que 0.40) para elevar a sensibilidade."*

---

#### Questão (f): Avaliação da AUC ($0.84$) e Comparação com a Acurácia
1. **Capacidade Discriminatória:**
   > *"Conforme a classificação de Hosmer e Lemeshow ($0.80 \le AUC < 0.90$), o valor $AUC = 0.84$ confere ao modelo uma **excelente capacidade de discriminação**. Há 84% de probabilidade de que, sorteando-se ao acaso um motorista acidentado e um não acidentado, o modelo atribua uma probabilidade de risco mais elevada àquele que realmente sofreu o acidente."*
2. **Por que a AUC é Mais Robusta que a Acurácia?**
   * **Independência de Limiar Arbitrário:** A acurácia varia drasticamente conforme o pesquisador escolhe $c = 0.30, 0.40$ ou $0.50$. A AUC integra a curva de sensibilidade versus $1-\text{especificidade}$ ao longo de **todos os infinitos limiares possíveis** ($c \in [0, 1]$), constituindo uma medida global e pura da capacidade de separação das classes.
   * **Invariância ao Desbalanceamento Amostral:** Na amostra avaliada, a proporção de não acidentes para acidentes é de $3:1$ ($225$ vs $75$). Um classificador nulo que previsse 'não acidente' para todos os motoristas alcançaria $75\%$ de acurácia com utilidade nula. A AUC não sofre essa distorção porque trata a taxa de verdadeiros positivos e a taxa de falsos positivos com pesos intrinsecamente equilibrados.

---

#### Questão (g): Interações e Interpretação de Sinais Opostos
Modelo com termos interativos:
$$\eta = \beta_0 + \beta_1 \text{Horas} + \beta_2 \text{Exp} + \beta_3 \text{Trein} + \beta_4 \text{Turno} + \beta_5(\text{Horas} \times \text{Exp}) + \beta_6(\text{Trein} \times \text{Turno})$$
Estimativas: $\hat{\beta}_1 = +0.080$, $\hat{\beta}_2 = -0.060$, $\hat{\beta}_3 = -0.900$, $\hat{\beta}_4 = +1.700$, $\hat{\beta}_5 = +0.0015$, $\hat{\beta}_6 = +1.100$.

* **(1) Interação Horas $\times$ Experiência ($\beta_5 = +0.0015 > 0$):**
  $$\frac{\partial \eta}{\partial \text{Horas}} = \beta_1 + \beta_5 \cdot \text{Experiência} = 0.080 + 0.0015 \cdot \text{Experiência}$$
  *Interpretação:* "O impacto de horas dirigidas no aumento da chance de acidente é maior para motoristas com maior experiência acumulada (indicativo de desgaste físico ou tolerância excessiva ao risco entre motoristas veteranos)."
* **(2) Interação Treinamento $\times$ Turno ($\beta_3 = -0.900$ vs $\beta_6 = +1.100$):**
  * Para motoristas diurnos ($\text{Turno} = 0$): $\text{Efeito} = \hat{\beta}_3 = -0.900 \implies OR = e^{-0.90} \approx 0.407$ (redução de 59% na chance de acidente).
  * Para motoristas noturnos ($\text{Turno} = 1$): $\text{Efeito} = \hat{\beta}_3 + \hat{\beta}_6 = -0.900 + 1.100 = +0.200 \implies OR = e^{+0.20} \approx 1.22$.
* **Padrão de Resposta para "Sinais Opostos":**
  > *"Quando o efeito principal ($\beta_3 < 0$) e o termo de interação ($\beta_6 > 0$) possuem sinais opostos, a interação opera como um **modificador e atenuador de efeito**. O benefício protetor do treinamento observado no turno diurno é completamente anulado e ligeiramente invertido no turno noturno em razão de adversidades severas não remediadas pelo treinamento (como fadiga biológica, visibilidade precária e reflexos diminuídos). Por essa razão, **é estatisticamente incorreto interpretar o efeito principal isoladamente** quando um termo interativo estatisticamente significante integra o modelo."*

---

### 8.2 CASO 2: Diagnóstico Comparativo dos 4 Gráficos Clássicos (Atividade 7)

A Profa. Rosemeire cobra a comparação sistemática entre gráficos bem-comportados (Caso 1) e problemáticos (Caso 2) gerados pela função `plot(modelo)`:

#### 1. Residuals vs Fitted ($r_i$ vs $\hat{\mu}_i$)
* **Utilidade:** Validação da premissa de linearidade da função de ligação e da hipótese de esperança matemática nula dos erros ($E[\varepsilon_i] = 0$).
* **Caso 1 (Adequado):** Nuvem de resíduos uniformemente distribuída em torno da linha horizontal $0$, sem concentração sistemática de pontos acima ou abaixo da reta de referência.
* **Caso 2 (Inadequado):** Curvatura acentuada em forma de parábola ou "U".
* **Diagnóstico e Conduta:** Falta de especificação de preditores não-lineares (como termos quadráticos $X^2$) ou uso de função de ligação inadequada ao fenômeno.

#### 2. Normal Q-Q (Quantis dos Resíduos vs Quantis Teóricos)
* **Utilidade:** Avaliação da hipótese distributiva da resposta (normalidade dos resíduos padronizados ou aderência à distribuição teórica postulada).
* **Caso 1 (Adequado):** Pontos rigorosamente alinhados sobre a bissetriz diagonal a $45^\circ$.
* **Caso 2 (Inadequado):** Desvios acentuados nas extremidades em formato sinusoidal ("S").
* **Diagnóstico e Conduta:** Evidência de caudas pesadas ou assimetria pronunciada na variável dependente, sugerindo abandono do modelo normal em favor de distribuições da família exponencial mais flexíveis (como Gama ou Normal Inversa).

#### 3. Scale-Location ($\sqrt{|r_{\text{std}}|} \text{ vs } \hat{\mu}_i$)
* **Utilidade:** Verificação da homocedasticidade (constância da variância ao longo dos valores esperados).
* **Caso 1 (Adequado):** Linha suavizada vermelha praticamente horizontal com amplitude de dispersão vertical homogênea em toda a extensão do eixo das abscissas.
* **Caso 2 (Inadequado):** Linha vermelha com inclinação positiva acentuada ou abertura em formato de leque/funil.
* **Diagnóstico e Conduta:** Heterocedasticidade confirmada (a variância cresce sistematicamente com a média $\mu$), justificando a modelagem por MLG com função de variância $V(\mu) = \mu^2$ (Gama) ou $V(\mu) = \mu$ (Poisson).

#### 4. Residuals vs Leverage ($r_{\text{std}} \text{ vs } h_{ii}$)
* **Utilidade:** Identificação conjunta de pontos de alta alavancagem (*leverage*) e observações desproporcionalmente influentes por meio da Distância de Cook ($D_i$).
* **Caso 1 (Adequado):** Todas as observações situadas dentro das curvas tracejadas de nível de Cook ($D_i < 0.50$).
* **Caso 2 (Inadequado):** Presença de observações posicionadas além dos limites críticos de Cook ($D_i > 0.50$ ou $D_i > 1.0$) associadas a alavancagem superior a $2p/n$.
* **Diagnóstico e Conduta:** Indivíduos influentes distorcendo os coeficientes e a inferência. Recomenda-se ajustar o modelo sem as observações para avaliar a robustez das conclusões.

---

### 8.3 CASO 3: Titanic — O Erro Clássico de Interpretação: OR vs RR (Lista 4)

#### Enunciado
Estudo de sobrevivência no Titanic reportou $OR = 11.4$ entre mulheres e homens. A chance de sobrevivência de mulheres foi $\text{chance}_M = 2.9$.

#### Item (a): Por que a afirmação "A probabilidade de sobrevivência de mulheres foi 11.4 vezes a de homens" está incorreta?
> *"A afirmação incorre no equívoco clássico de equiparar a **Razão de Chances (Odds Ratio)** à **Razão de Probabilidades (Risco Relativo)**. A estatística de 11,4 mensura a razão entre as razões $\frac{\pi}{1-\pi}$, e **não entre as probabilidades brutas $\pi$**. A equivalência matemática $OR \approx RR$ é válida única e exclusivamente sob a **premissa de evento raro ($\pi < 0.10$)**. Como a sobrevivência feminina no naufrágio foi um desfecho comum ($\pi_M \approx 74.4\%$), o Odds Ratio superestima consideravelmente o Risco Relativo."*

#### Item (b): Cálculo Manual Rigoroso das Probabilidades
1. **Probabilidade Feminina ($\pi_M$):**
   $$\text{chance}_M = \frac{\pi_M}{1 - \pi_M} = 2.9 \implies \pi_M = \frac{2.9}{1 + 2.9} = \frac{2.9}{3.9} \approx \mathbf{0.7436} \quad (74,36\%)$$
2. **Chance Masculina ($\text{chance}_H$):**
   $$OR = \frac{\text{chance}_M}{\text{chance}_H} = 11.4 \implies \text{chance}_H = \frac{2.9}{11.4} \approx \mathbf{0.2544}$$
3. **Probabilidade Masculina ($\pi_H$):**
   $$\pi_H = \frac{\text{chance}_H}{1 + \text{chance}_H} = \frac{0.2544}{1 + 0.2544} = \frac{0.2544}{1.2544} \approx \mathbf{0.2028} \quad (20,28\%)$$
4. **Cálculo do Risco Relativo Verdadeiro ($RR$):**
   $$RR = \frac{\pi_M}{\pi_H} = \frac{0.7436}{0.2028} \approx \mathbf{3.67}$$
   *Conclusão:* A chance de sobrevivência é 11,4 vezes maior, mas a probabilidade real é apenas 3,67 vezes maior!

---

### 8.4 CASO 4: Cifose (Kyphosis) — Decodificação das Saídas do R (Lista 4)

#### Modelos em Confronto
* `ajuste1`: `Kyphosis ~ Age + Number + Start` ($n=81, p=4$)
* `ajuste2`: `Kyphosis ~ Start` ($n=81, p=2$)

#### 1. Relação entre os Modelos via Restrição Linear
> *"O `ajuste2` é obtido impondo-se a restrição de nulidade conjunta aos coeficientes de idade e número de vértebras no `ajuste1`, ou seja, sob a hipótese linear $H_0: \beta_{\text{Age}} = \beta_{\text{Number}} = 0$ (em notação matricial: $C\beta = 0$, com $C$ sendo uma matriz de contrastes $2 \times 4$)."*

#### 2. Teste Sequencial (`anova()`) vs Teste Parcial (`Anova()`)
* **Saída de `anova(ajuste1)`:**
  `Age` entra na 1ª linha: $\Delta D = 1.302, df = 1, p = 0.2538$.
* **Saída de `Anova(ajuste1)`:**
  `Age` ajustada por `Number` e `Start`: $\Delta D = 3.1565, df = 1, p = 0.0756$.
* **Padrão de Resposta para Prova:**
  > *"O comando nativo `anova()` calcula deviances **sequenciais (Tipo I)**, de modo que cada variável é avaliada unicamente contra as variáveis adicionadas antes dela. Como `Age` foi introduzida primeiro, foi testada isoladamente contra o modelo nulo, sem benefício do ajuste pelas demais. Já a função `Anova()` do pacote `car` calcula deviances **parciais/marginais (Tipo II)**, testando cada covariável condicional a **todas as outras presentes no modelo final**. Em decorrência da correlação entre idade e número de vértebras, o controle simultâneo reduziu o ruído residual e evidenciou a relevância de `Age` ($p$ reduziu de $0.25$ para $0.075$)."*

#### 3. Intervalo de Confiança para $\pi$ via `predict(..., type = "link")`
* Dados fornecidos: $\hat{\eta} = 1.4467$ e $EP(\hat{\eta}) = 0.7830$.
* **Passo 1 (Escala do Link):**
  $$[\eta_{\text{inf}}, \, \eta_{\text{sup}}] = 1.4467 \pm 1.96(0.7830) = [-0.0880, \, 2.9814]$$
* **Passo 2 (Transformação Inversa Logística):**
  $$\pi_{\text{inf}} = \frac{e^{-0.0880}}{1 + e^{-0.0880}} \approx \mathbf{0.4780} \quad (47,80\%)$$
  $$\pi_{\text{sup}} = \frac{e^{2.9814}}{1 + e^{2.9814}} \approx \mathbf{0.9517} \quad (95,17\%)$$
* *Atenção de Prova:* Jamais somar e subtrair $1.96 \times EP$ diretamente sobre $\hat{\pi}$, sob risco de obter probabilidades fora do intervalo $[0, 1]$.

---

### 8.5 CASO 5: Superdispersão Severa e o Modelo Beta-Binomial (Lista 5 — Rotifers)

#### Diagnóstico Imediato sem Linhas de Código
1. **Critério da Razão Deviance / Graus de Liberdade:**
   $$D_{\text{res}} = 434.25, \quad gl_{\text{res}} = 37 \implies \frac{D}{gl} = \frac{434.25}{37} = \mathbf{11.74} \gg 1$$
   *Conclusão:* Violação frontal da premissa de equi-dispersão ($\phi = 1$) do modelo Binomial.
2. **Critério do Gráfico de Envelope Simulado (`hnp` / Paula):**
   * Praticamente a totalidade dos pontos amostrais encontra-se **fora e acima da banda superior do envelope simulado**.
   * *Interpretação Biológica:* Presença de correlação intra-amostral decorrente de animais expostos conjuntamente nos mesmos tubos de ensaio, gerando variância amostral excessiva frente à variância binomial teórica $m_i \pi_i (1-\pi_i)$.
3. **Solução Adotada:**
   * Ajuste do **Modelo Beta-Binomial**, que incorpora um parâmetro adicional $\rho$ (coeficiente de correlação intraclasse):
     $$\hat{\rho} = \text{logit}^{-1}(-2.0432) = \frac{1}{1 + e^{2.0432}} \approx \mathbf{0.115}$$
   * *Efeito Corretivo:* O modelo Beta-Binomial amplia os erros-padrão dos coeficientes para seu tamanho real (que eram subestimados pelo modelo binomial comum), eliminando significâncias espúrias e garantindo confiabilidade aos testes de hipótese.

---

### 8.6 CASO 6: Dedução Analítica da Estatística da Razão de Verossimilhanças (Atividade 8)

#### Enunciado Teórico
Seja $Y_1, \dots, Y_n$ uma amostra aleatória simples tal que $Y_i \sim \text{Bernoulli}(\pi)$, com função de ligação logística canônica dada por $\log\left(\frac{\pi}{1-\pi}\right) = \alpha$.
Obtenha a EMV $\hat{\alpha}$, $\text{Var}(\hat{\alpha})$ e a estatística da Razão de Verossimilhanças para testar $H_0: \alpha = 0$ contra $H_1: \alpha \ne 0$.

#### 1. Função de Log-Verossimilhança
Como $\pi = \frac{e^\alpha}{1 + e^\alpha}$ e $1 - \pi = \frac{1}{1 + e^\alpha}$:
$$\ell(\alpha) = \sum_{i=1}^n \left[ y_i \log(\pi) + (1 - y_i)\log(1 - \pi) \right]$$
$$\ell(\alpha) = \sum_{i=1}^n \left[ y_i \log\left(\frac{\pi}{1-\pi}\right) + \log(1 - \pi) \right] = \alpha \sum_{i=1}^n y_i - n \log(1 + e^\alpha)$$

#### 2. Estimador de Máxima Verossimilhança ($\hat{\alpha}$)
Derivando em relação a $\alpha$ (função escore):
$$U(\alpha) = \frac{\partial \ell}{\partial \alpha} = \sum_{i=1}^n y_i - n \frac{e^\alpha}{1 + e^\alpha} = n\bar{y} - n\pi = 0$$
$$\hat{\pi} = \bar{y} \implies \frac{\hat{\pi}}{1 - \hat{\pi}} = \frac{\bar{y}}{1 - \bar{y}} \implies \mathbf{\hat{\alpha} = \log\left(\frac{\bar{y}}{1 - \bar{y}}\right)}$$

#### 3. Informação de Fisher e Variância Assintótica
Derivada de segunda ordem:
$$\frac{\partial^2 \ell}{\partial \alpha^2} = -n \frac{e^\alpha (1 + e^\alpha) - e^\alpha(e^\alpha)}{(1 + e^\alpha)^2} = -n \frac{e^\alpha}{(1 + e^\alpha)^2} = -n \pi(1 - \pi)$$
A Informação de Fisher é:
$$I(\alpha) = -E\left[\frac{\partial^2 \ell}{\partial \alpha^2}\right] = n \pi(1 - \pi)$$
Portanto, a variância assintótica da EMV é:
$$\mathbf{\text{Var}(\hat{\alpha}) = [I(\hat{\alpha})]^{-1} = \frac{1}{n \bar{y}(1 - \bar{y})}}$$

#### 4. Estatística da Razão de Verossimilhanças para $H_0: \alpha = 0$
* Sob $H_0: \alpha = 0$, temos $\pi_0 = \frac{e^0}{1+e^0} = \frac{1}{2} = 0.5$.
  $$\ell(0) = 0 \cdot \sum y_i - n \log(1 + 1) = -n \log(2)$$
* Sob o modelo irrestrito ($H_1$):
  $$\ell(\hat{\alpha}) = n \bar{y} \log\left(\frac{\bar{y}}{1-\bar{y}}\right) - n \log\left(1 + \frac{\bar{y}}{1-\bar{y}}\right) = n \left[ \bar{y}\log(\bar{y}) + (1-\bar{y})\log(1-\bar{y}) \right]$$
* Estatística da Razão de Verossimilhanças:
  $$\Lambda = 2\left[\ell(\hat{\alpha}) - \ell(0)\right] = 2n \left[ \bar{y}\log(2\bar{y}) + (1-\bar{y})\log(2(1-\bar{y})) \right]$$
* **Distribuição Assintótica:**
  Como há apenas 1 parâmetro sob restrição ($\alpha = 0$), sob regularidades assintóticas:
  $$\mathbf{\Lambda \sim_a \chi^2_1}$$
  Rejeita-se $H_0$ se $\Lambda > \chi^2_{1; 1-\alpha}$ (ao nível $\alpha = 0.05$, $\chi^2_{1; 0.95} = 3.84$).
