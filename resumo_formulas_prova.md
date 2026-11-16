# GUIA DEFINITIVO & FOLHA DE FÓRMULAS PARA A PROVA 2
## Modelos Lineares Generalizados (MATD50 — UFBA) | Profa. Rosemeire L. Fiaccone
*Material estruturado em etapas didáticas: conceitos, quando usar, passo a passo, fórmulas, diagnósticos, gráficos, testes com $H_0$, limitações e modelos prontos de resposta para a folha da prova.*

---

# ETAPA 1: GUIA DEFINITIVO DA ANODEV, DEVIANCE, $\hat{\phi}$ E GRAUS DE LIBERDADE

### 1.1 O Que É e em Qual Contexto Usar a ANODEV
* **O que é:** A **Análise do Desvio (ANODEV)** é a generalização da Análise de Variância (ANOVA) clássica para os Modelos Lineares Generalizados (MLGs). Em vez de decompor somas de quadrados ($SQ$), ela decompõe e compara as **log-verossimilhanças maximizadas** (via função desvio) de modelos concorrentes encaixados.
* **Em qual contexto usar:**
  1. Para avaliar a **qualidade global do ajuste** (*goodness of fit*) comparando o modelo proposto ($M_p$) ao modelo saturado ($M_n$) — válido diretamente apenas quando $\phi$ é conhecido e as contagens/subpopulações são adequadas.
  2. Para testar a **significância da inclusão ou exclusão de variáveis, fatores (categóricos) e interações** entre **modelos encaixados** ($M_{\text{reduzido}} \subset M_{\text{completo}}$).
  3. Para orientar estratégias de **seleção de variáveis** (*Forward*, *Backward* e *Stepwise*).
* **⚠️ Quando NÃO usar ANODEV / TRV:** Modelos **não encaixados** (ex.: comparar ligação `logit` vs `probit` ou preditores totalmente distintos) **NÃO** podem ser comparados por ANODEV/TRV! Nesses casos, utilize **AIC** ($\Delta AIC > 2 \implies$ menor AIC fortemente preferível) ou **BIC**.

---

### 1.2 Os Três Modelos de Referência & Reta das Log-Verossimilhanças (Slides 2, 7)
Para entender qualquer saída de MLG, domine a hierarquia:
$$\ell_0 \le \ell_m \le \ell_q \le \ell_n \qquad \Longleftrightarrow \qquad D_0^* \ge D_m^* \ge D_q^* \ge D_n^* = 0$$
1. **Modelo Nulo ($M_0$):** Possui apenas o intercepto comum ($p = 1$). Assume que nenhuma covariável afeta $Y$ ($\mu_i = \mu$ constante). Tem a **menor** log-verossimilhança ($\ell_0$) e a **maior** deviance (`Null deviance` $D_0$).
2. **Modelos Propostos ($M_m \subset M_q$):** Modelos intermediários com $m$ e $q$ parâmetros ($1 < m < q < n$). O modelo $M_m$ é **encaixado** em $M_q$ quando pode ser obtido de $M_q$ zerando $q-m$ coeficientes.
3. **Modelo Saturado ($M_n$ ou $M_s$):** Possui $n$ parâmetros (ou $N$ em dados agrupados: 1 parâmetro por observação/subpopulação). Ajusta perfeitamente os dados ($\hat{\mu}_i = y_i$), atingindo a **máxima** log-verossimilhança possível ($\ell_n = \ell(\mathbf{y}; \mathbf{y})$) e deviance nula ($D_n = 0$).

---

### 1.3 Deviance Não-Escalonada ($D_p$ / $D$) vs. Deviance Escalonada ($D^*$)
A família exponencial tem log-verossimilhança $\ell(\boldsymbol{\theta}, \phi; \mathbf{y}) = \sum_{i=1}^n \left[ \frac{y_i \theta_i - b(\theta_i)}{a(\phi)} + c(y_i, \phi) \right]$ com $\text{Var}(Y_i) = a(\phi) V(\mu_i)$.

#### A. Deviance Escalonada (*Scaled Deviance*, $D^*$)
Mede o dobro da perda de log-verossimilhança do modelo proposto em relação ao modelo saturado:
$$D^* = 2(\ell_n - \ell_p) = 2\left[\ell(\mathbf{y}; \mathbf{y}) - \ell(\hat{\boldsymbol{\mu}}; \mathbf{y})\right] = a^{-1}(\phi) D_p = a^{-1}(\phi) \sum_{i=1}^n d_i^2$$
* **Características principais:**
  * É **adimensional** (livre da unidade da variável resposta $Y$).
  * Sob o modelo correto e condições de regularidade, $D^* \sim_a \chi^2_{n - p}$ (distribuição **exata** no modelo Normal com $\sigma^2$ conhecido).
  * **Alerta do Slide 7:** *"Para modelos com parâmetros de dispersão que não foram estimados (ou desconhecidos), a deviance escalonada $D^*$ não pode ser determinada."*

#### B. Deviance Não-Escalonada (*Unscaled Deviance*, $D_p$ ou $D$)
É a discrepância puramente geométrica/verossímil sem dividir pelo parâmetro de dispersão $\phi$:
$$D_p = D(\mathbf{y}; \hat{\boldsymbol{\mu}}) = \sum_{i=1}^n d_i^2 = 2 a(\phi) (\ell_n - \ell_p)$$
onde $d_i$ é o **componente do desvio não escalonado** da $i$-ésima observação (e o resíduo deviance bruto é $t_{Di} = \text{sinal}(y_i - \hat{\mu}_i)\sqrt{d_i^2}$).

* **🚨 Por que a diferença entre $D_p$ e $D^*$ cai na prova?**
  1. **$D_p$ NÃO depende de $\phi$:** Por isso pode sempre ser calculada pela função de média $\hat{\boldsymbol{\mu}}$.
  2. **O R imprime $D_p$!** Os valores `Null deviance` e `Residual deviance` exibidos em `summary(glm)` e `anova(glm)` são **deviances NÃO-escalonadas ($D_p$)**.
  3. **Consequência prática crucial:** Quando $\phi \ne 1$ (Normal, Gama, Normal Inversa), $D_p$ carrega a escala de $Y$ (na Normal, $D_p = SQRes$ em unidade de $Y^2$). Logo, você **NÃO PODE** testar `Residual deviance` ($D_p$) contra $\chi^2_{n-p}$ diretamente nem usar $D_p/(n-p) \approx 1$! Para converter em $D^*$, é obrigatório dividir pela estimativa $\hat{\phi}$: $D^* = D_p / \hat{\phi}$.
  4. Quando $\phi = 1$ (Binomial, Poisson): $D_p = D^*$ coincidem numericamente!

#### C. 🚨 Atenção Máxima às Duas Notações de $\phi$ nos Slides (`mlgp2.pdf`)
Na disciplina convivem duas convenções algebricamente equivalentes — aprenda a reconhecê-las na hora da prova:
* **Convenção 1 — Parâmetro de Dispersão ($a(\phi) = \phi$ — Slides 2, 42–50 e software R):**
  $$D^* = \frac{D_p}{\phi} \quad \Longleftrightarrow \quad D_p = \phi D^*$$
  Aqui, na Normal $\phi = \sigma^2$, na Gama $\phi = \nu^{-1}$, na Normal Inversa $\phi = \sigma^2$. É o valor impresso no R em `Dispersion parameter ... taken to be ...` e calculado por $\hat{\phi}_D = \frac{D_p}{n-p}$.
* **Convenção 2 — Notação de Gilberto Paula (Slides 2 e 8 — parâmetro de *precisão*):**
  $$D^*(\mathbf{y}; \hat{\boldsymbol{\mu}}) = \phi D(\mathbf{y}; \hat{\boldsymbol{\mu}}) = 2\{L(\mathbf{y}; \mathbf{y}) - L(\hat{\boldsymbol{\mu}}; \mathbf{y})\}$$
  Aqui $\phi$ é o inverso da dispersão (precisão), logo na Normal $\phi = \sigma^{-2}$. Em testes comparativos ($\Delta D^*$, TRV ou $F$), o $\phi$ cancela no Teste $F$ e o resultado numérico da decisão é **idêntico** nas duas convenções!

#### D. Fórmulas Explícitas da Deviance Não-Escalonada ($D_p$) por Distribuição
| Distribuição | Parâmetro $\phi$ (Conv. 1) | Função de Variância $V(\mu)$ | Deviance Não-Escalonada ($D_p$) | Relação com $D^*$ |
| :--- | :---: | :---: | :--- | :---: |
| **Normal** | $\phi = \sigma^2$ (desconhecido) | $1$ | $\sum_{i=1}^n (y_i - \hat{\mu}_i)^2 = SQRes$ | $D^* = SQRes/\sigma^2 \sim \chi^2_{n-p}$ (exata) |
| **Poisson** | $\phi = 1$ (conhecido) | $\mu$ | $2\sum_{i=1}^n \left[y_i \ln\!\left(\frac{y_i}{\hat{\mu}_i}\right) - (y_i - \hat{\mu}_i)\right]$ | $D^* = D_p$ |
| **Binomial (agrupada)** | $\phi = 1$ (conhecido) | $\frac{\pi(1-\pi)}{n_i}$ | $2\sum_{i=1}^N \left[y_i \ln\!\left(\frac{y_i}{n_i\hat{\pi}_i}\right) + (n_i-y_i)\ln\!\left(\frac{n_i-y_i}{n_i-n_i\hat{\pi}_i}\right)\right]$ | $D^* = D_p$ |
| **Bernoulli ($m_i=1$)** | $\phi = 1$ (conhecido) | $\pi(1-\pi)$ | $-2\sum_{i=1}^n \left[y_i \ln(\hat{\pi}_i) + (1-y_i)\ln(1-\hat{\pi}_i)\right]$ | $D^* = D_p$ (**NÃO** $\sim \chi^2_{n-p}$!) |
| **Gama** | $\phi = \nu^{-1}$ (desconhecido) | $\mu^2$ | $2\sum_{i=1}^n \left[-\ln\!\left(\frac{y_i}{\hat{\mu}_i}\right) + \frac{y_i - \hat{\mu}_i}{\hat{\mu}_i}\right]$ | $D^* = D_p / \phi$ |
| **Normal Inversa** | $\phi = \sigma^2$ (desconhecido) | $\mu^3$ | $\sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{y_i \hat{\mu}_i^2}$ | $D^* = D_p / \phi$ |

---

### 1.4 Estimação Completa do Parâmetro de Dispersão ($\hat{\phi}$) (Slides 42–49)

#### Quando $\phi$ é conhecido vs. desconhecido?
* **$\phi = 1$ fixo e conhecido:** **Binomial** (e Bernoulli), **Poisson** e **Binomial Negativa**. Não se estima $\phi$ para o TRV ($\Delta D$ usa $\chi^2$). *(Contudo, pode-se calcular $\hat{\phi}_{\text{checado}}$ depois para diagnosticar superdispersão em Poisson/Binomial agrupada!)*
* **$\phi$ desconhecido (precisa ser estimado):** **Normal** ($\phi = \sigma^2$), **Gama** ($\phi = \nu^{-1}$) e **Normal Inversa** ($\phi = \sigma^2$).

#### Os 3 Métodos de Estimação de $\hat{\phi}$ (Slides 43–48)
1. **Método do Desvio ($\hat{\phi}_D$):**  
   Baseia-se em $D^* = D_p/\phi \sim_a \chi^2_{n-p} \implies E(D^*) = n - p$. Igualando à estatística observada ($n - p = D_p / \hat{\phi}$):
   $$\hat{\phi}_D = \frac{D_p}{n - p}$$
   * **No modelo Normal Linear:** Coincide com o estimador não viesado clássico $\hat{\sigma}^2 = \frac{\sum (y_i-\hat{\mu}_i)^2}{n-p} = QMRes$.
   * *Limitação:* Em modelos assimétricos (como Gama), observações muito próximas de zero tornam $D_p$ muito sensível, podendo distorcer $\hat{\phi}_D$.

2. **Método de Pearson ($\hat{\phi}_P$ — Mais recomendado e padrão do R):**  
   Baseia-se na Estatística de Pearson Generalizada $\chi_p^2 = \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{V(\hat{\mu}_i)}$, onde $\frac{\chi_p^2}{\phi} \sim_a \chi^2_{n-p}$:
   $$\hat{\phi}_P = \frac{\chi_p^2}{n - p} = \frac{1}{n - p} \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{V(\hat{\mu}_i)}$$
   * **No modelo Normal:** Como $V(\mu) = 1$, também coincide exatamente com $QMRes = \hat{\phi}_D$.
   * **Por que é o favorito na prática (e usado pelo `summary.glm()` no R)?** Depende apenas da média e da função de variância $V(\hat{\mu}_i)$, sendo bem menos sensível a valores próximos de zero ou arredondamentos do que $\hat{\phi}_D$.

3. **Método da Máxima Verossimilhança ($\hat{\phi}_{MV}$):**  
   Resolve a equação escore para $\phi$: $U_\phi = \frac{\partial \ell(\boldsymbol{\beta}, \phi)}{\partial \phi} = 0$.
   * **Para Normal e Normal Inversa (Slide 47):** O EMV exato é:
     $$\hat{\phi}_{MV} = \frac{D_p}{n}$$
     *(Atenção: divide por $n$ em vez de $n - p$, portanto é **viesado** para menos em amostras finitas!)*
   * **Para Gama — Aproximação de Cordeiro & McCullagh (1991) (Slide 47)** para pequenos valores de $\phi$:
     $$\hat{\phi}_{CM} = \frac{2 D_p}{n \left[ 1 + \left( 1 + \frac{2 D_p}{3 n} \right)^{1/2} \right]}$$

#### Propriedade Teórica Crucial: Ortogonalidade entre $\boldsymbol{\beta}$ e $\phi$ (Slide 49)
$$\mathfrak{I}_{\beta\phi}(\theta) = E\left[ \frac{\partial^2 \ell(\beta, \phi)}{\partial \beta \, \partial \phi} \right] = 0 \implies \mathfrak{I}_\theta = \text{diag}\{\mathfrak{I}_{\beta\beta}, \, \mathfrak{I}_{\phi\phi}\}$$
Como a matriz de informação de Fisher é **bloco-diagonal**, $\hat{\boldsymbol{\beta}}$ e $\hat{\phi}$ são **assintoticamente independentes**, e a estimação de $\hat{\boldsymbol{\beta}}$ (via IWLS / Escore de Fisher) pode ser feita sem conhecer $\phi$!

---

### 1.5 Regras Completas de $n$ ($N$), $p$, $m$, $q$, Graus de Liberdade ($gl$) e Comparações (Slides 4–9, 74)

#### 1. Quem é $n$ (ou $N$) e quem é $gl_{\text{Nulo}}$?
* **Dados individuais (desagrupados):** $n$ é o total de observações/sujeitos (linhas do banco). Ex.: `kyphosis` tem $n = 81$ crianças $\implies gl_0 = 81 - 1 = 80$.
* **Dados agrupados em tabela/subpopulações ($N$ ou $s$, Slides 8 e 74):** $N$ é o **número de subpopulações (combinações de covariáveis / linhas da tabela)**, e **NÃO** a soma das frequências de indivíduos!
  * *Exemplo dos Slides (Cupons):* $1400$ cupons distribuídos em $7$ valores de desconto $\implies N = 7 \implies gl_{\text{Nulo}} = 7 - 1 = \mathbf{6}$.
  * *Regra do Slide 8:* $$gl_N = \text{g.l. do modelo nulo} = \text{número de subpopulações } - 1$$

#### 2. Como contar $p$ (total de parâmetros) e $gl = n - p$ em qualquer modelo:
Cada termo no preditor linear consome parâmetros **além do intercepto ($\beta_0 \implies +1$)**:
* **Covariável contínua linear ($X$):** $+1$ parâmetro.
* **Termo polinomial ($X + X^2$):** $+2$ parâmetros.
* **Fator categórico $A$ com $n(A)$ níveis (Slide 8–9):** $+[n(A) - 1]$ parâmetros (dummies em relação à categoria de referência).
* **Interação entre fatores $A * B$ (Slide 9):** $+[n(A) - 1][n(B) - 1]$ parâmetros.
* **Interação contínua $\times$ fator $A$:** $+[n(A) - 1]$ parâmetros.
* **Fórmula mestra dos Graus de Liberdade:**
  $$gl_{\text{modelo}} = n - p \qquad (\text{ou } N - p \text{ em dados agrupados})$$
  * **Nulo:** $p = 1 \implies gl_0 = n - 1$
  * **Saturado:** $p = n \implies gl_S = 0$ e $D_S = 0$
  * **Diferença entre encaixados ($M_m \subset M_q$):** $\Delta gl = gl_m - gl_q = (n - m) - (n - q) = q - m$

#### 3. As Regras de Ouro sobre Deviance, $gl$, Dados Faltantes e Razão $D/(n-p)$
1. **Regra dos Dados Faltantes / `NA` (Alerta do Slide 8):** Em modelos sequenciais, a presença de `NA` em uma covariável $X_k$ faz o R excluir linhas somente quando $X_k$ entra, alterando $n$ entre modelos. **Se `n` mudar, `Null deviance` muda e a ANODEV/TRV fica INVÁLIDA!** *(Na prova, verifique sempre se `Resid. Df` reduz exatamente pelo nº de parâmetros adicionados!)*
2. **Monotonicidade (Slide 2):** Adicionar parâmetros SEMPRE faz $D$ **decrescer ou permanecer igual** ($D_m \ge D_q$) e $gl$ diminuir ($gl_m > gl_q$).
3. **Regra da Razão $D_p / (n - p)$ (Quando $\phi = 1$ — Poisson e Binomial Agrupada):**
   * **$\frac{D_p}{n - p} \approx 1.0$** (e $P(\chi^2_{n-p} \ge D_p) \ge 0.05$): Modelo bem ajustado (equidispersão).
   * **$\frac{D_p}{n - p} \gg 1.0$** (ex.: $> 1.5$ ou $> 2.0$): **Superdispersão** (*overdispersion*) ou falha de especificação (faltam variáveis/interações, ligação errada, excesso de zeros, dependência). Soluções: Quasi-Poisson/Quasi-Binomial ou Binomial Negativa.
   * **$\frac{D_p}{n - p} \ll 1.0$**: **Subdispersão** (rara; possível sobreajuste ou correlação negativa).
   * **⚠️ Exceção Binomial Pura ($m_i=1, y_i \in \{0,1\}$):** $D_p$ NÃO segue $\chi^2_{n-p}$! Não use $D_p/(n-p)$ para checar superdispersão ou bondade de ajuste (use Hosmer-Lemeshow); mas **$\Delta D$ entre modelos encaixados CONTINUA válido!**
   * **⚠️ Exceção $\phi$ Desconhecido (Normal, Gama, NI):** Nunca compare $D_p$ bruto com $(n-p)$! Em $\phi$ desconhecido, $D_p / (n - p) = \hat{\phi}_D$ é o **estimador da dispersão**, e NÃO indica falta de ajuste por si só!
4. **Comparação com Pearson ($X^2$) vs. Deviance (Slide 3):** A função desvio é **aditiva** para modelos encaixados e sempre **decresce** ao incluir variáveis; já $X^2 = \sum \frac{(y_i-\hat{\mu}_i)^2}{\widehat{\text{Var}}(y_i)}$ não é aditiva e não decresce necessariamente ao adicionar termos.

---

### 1.6 Como Fazer uma ANODEV: Passo a Passo Completo

#### Passo 1: Definir Modelos Encaixados ($M_m \subset M_q$, $m < q < n$) e Hipóteses
* **$H_0$:** $\beta_{m+1} = \beta_{m+2} = \cdots = \beta_q = 0$ (os $q-m$ parâmetros extras são nulos; adota-se o modelo parcimonioso $M_m$).
* **$H_1$:** Pelo menos um dos $\beta_j \ne 0$ ($j = m+1, \dots, q$) (melhora significante; adota-se $M_q$).

#### Passo 2: Calcular $\Delta D$, $\Delta gl$ e Pseudo-$R^2$ / AIC
* **Diferença de Deviances:** $\Delta D = D_m - D_q = 2(\ell_q - \ell_m)$ *(sempre menor menos maior $\ge 0$!)*.
* **Diferença de g.l.:** $\Delta gl = gl_m - gl_q = q - m$.
* **Pseudo-$R^2$ (Slide 4):** $R^2 = \frac{D_0 - D_p}{D_0} = 1 - \frac{D_p}{D_0}$ *(proporção da deviance do modelo nulo explicada pelo modelo; independe de $\phi$ pois o fator cancela na divisão!)*.
* **AIC (Slide 4):** $AIC = -2\ell(\hat{\boldsymbol{\mu}}; \mathbf{y}) + 2p$. Menor é melhor ($\Delta AIC > 2 \implies$ forte preferência).

#### Passo 3: Escolher a Estatística de Teste ($\chi^2$ vs. Teste $F$)

##### CASO A: Dispersão ($\phi$) Conhecida ($\phi = 1$ — Binomial e Poisson)
* **Estatística TRV:** $\Delta D^* = \Delta D = D_m - D_q \sim_a \chi^2_{q - m}$
* **Decisão:** Se $\Delta D > \chi^2_{q-m;\, 1-\alpha}$ (ou $p\text{-valor} < \alpha$, no R `test="Chisq"`): **Rejeita-se $H_0$** $\implies$ termo(s) significante(s), adota-se $M_q$. Caso contrário, mantém-se $M_m$ por parcimônia.

##### CASO B: Dispersão ($\phi$) Desconhecida (Normal, Gama, Normal Inversa — Slides 3 e 50)
* **Estatística $F$:**
  $$F = \frac{(D_m - D_q) / (q - m)}{\hat{\phi}_{\text{maximal}}} = \frac{\Delta D / \Delta gl}{D_{\text{maximal}} / (n - q_{\text{maximal}})} \sim_a F_{q - m,\, n - q_{\text{maximal}}}$$
  *(Nota de notação: o Slide 50 escreve $(D_m^* - D_q^*)$ dividido por $D_q^*/(n-q)$, onde $\phi$ cancela entre numerador e denominador; na saída do R, usa-se diretamente as deviances não-escalonadas $D_p$ com `test="F"`).*
* **🚨 Regra de Ouro da Professora Rosemeire (Slide 50):** Numa sequência de modelos encaixados, utilize em **TODOS os testes $F$** o denominador $\hat{\phi}$ do **modelo maximal** ($M_{\text{maximal}}$, com o maior número de termos considerado). Se você usasse $\hat{\phi}$ de um modelo intermediário sub-ajustado, variáveis omitidas inflariam $\hat{\phi}$, diminuindo $F$ e mascarando efeitos significantes!
* **Decisão:** Rejeita-se $H_0$ se $F_{\text{calc}} > F_{q-m,\, n-q_{\text{max}};\, 1-\alpha}$ (ou $p\text{-valor} < \alpha$).

---

### 1.7 Como Interpretar Tabelas ANODEV no R: Tipo I (`anova`) vs. Tipo II (`Anova`)
1. **Tabela Sequencial — Tipo I (`anova(modelo, test="Chisq")` ou `"F"`):**  
   Avalia cada termo **condicional apenas aos termos que entraram antes dele** (a **ordem na fórmula importa**!).
   * Exemplo: linha 1 testa $X_1$ vs Nulo; linha 2 testa $X_2 \mid X_1$; linha 3 testa $X_1\!:\!X_2 \mid X_1, X_2$.
   * **Colunas:** `Df` $= \Delta gl = q - m$; `Deviance` $= \Delta D = \text{Resid. Dev}_{\text{ant}} - \text{Resid. Dev}_{\text{atual}}$; `Resid. Df` $= n - q$; `Resid. Dev` $= D_q$; `Pr(>Chi)` / `Pr(>F)` $= p$-valor.
2. **Tabela Parcial — Tipo II (`car::Anova(modelo)`):**  
   Testa cada efeito principal ajustado por **todos os demais efeitos principais** (como se entrasse por último antes das interações). **Independe da ordem** na fórmula! Coluna é `LR Chisq` e $p$-valor `Pr(>Chisq)`.
3. **Comparação Direta de Dois Modelos (`anova(m1, m2, test="Chisq")`):**  
   Testa simultaneamente o bloco completo de $\Delta gl$ termos adicionados de `m1` ($M_m$) para `m2` ($M_q$).

---

### 1.8 Exercícios Resolvidos Completos de ANODEV (Estilo Prova — Sem Código)

#### 📝 Exercício Resolvido 1 ($\phi = 1$ Conhecido — Dados Binomiais Agrupados, Lista 4 Gestantes)
Um estudo classificou recém-nascidos de gestantes fumantes quanto à **sobrevivência** em função de $3$ fatores binários: **Tempo de gestação** ($\le 260$ vs $>260$ dias), **Idade da mãe** ($<30$ vs $\ge 30$ anos) e **Cigarros/dia** ($<5$ vs $\ge 5$), formando $2 \times 2 \times 2 = 8$ subpopulações (total de $6789$ nascimentos). Abaixo está a saída parcial da ANODEV sequencial (`family = binomial(link="logit")`):

```text
                       Df Deviance Resid. Df Resid. Dev Pr(>Chi)
NULL                                     [A]    352.271         
Tempo_gestacao          1  342.334         6        [B]  < 0.001 ***
Idade_mae               1    6.038       [C]      3.898    0.014 *
Quantidade_cigarros     1      [D]         4      1.488    0.120  
```
**(a) Preencha `[A]`, `[B]`, `[C]`, `[D]` e explique por que `[A]` não vale $6788$:**
* Como os dados são **agrupados**, $N = 8$ subpopulações (regra do Slide 8: $gl_N = N - 1$). Logo, **`[A]` $= 8 - 1 = 7$**.
* Por aditividade da deviance: **`[B]` $= 352.271 - 342.334 = 9.937$**.
* A cada covariável binária entra $+1$ parâmetro: **`[C]` $= 6 - 1 = 5$**.
* Diferença do passo 3 para 4: **`[D]` $= 3.898 - 1.488 = 2.410$** (na tabela da Lista 4 arredondada: `2.411`).

**(b) Qual o modelo selecionado ao nível $\alpha = 5\%$? Calcule seu Pseudo-$R^2$ e avalie sua bondade de ajuste:**
* Na sequência: `Tempo_gestacao` ($p < 0.001$) e `Idade_mae` ($p = 0.014 < 0.05$) reduzem significativamente a deviance (ambas $\Delta D > \chi^2_{1; 0.95} = 3.841$). Já `Quantidade_cigarros` tem $\Delta D = 2.411 < 3.841$ ($p = 0.120 \ge 0.05$), não sendo significante dado que Tempo e Idade já estão no modelo $\implies$ **Seleciona-se o modelo `Tempo_gestacao + Idade_mae`** ($p = 3$ parâmetros: intercepto $+ 2$ fatores).
* **Pseudo-$R^2$:** $R^2 = 1 - \frac{3.898}{352.271} = \mathbf{98.89\%}$ da deviance do modelo nulo explicada!
* **Bondade de ajuste global (válida pois os dados são agrupados em $N=8$ subpopulações grandes e $\phi = 1$):**  
  Razão $\frac{D}{N - p} = \frac{3.898}{5} = \mathbf{0.78} \approx 1$ (e $3.898 < \chi^2_{5; 0.95} = 11.07$, com $p = 0.564 > 0.05$). Conclui-se que **não há evidência de falta de ajuste nem de superdispersão**!

---

#### 📝 Exercício Resolvido 2 ($\phi$ Desconhecido — Gama com 4 Modelos Encaixados, $\hat{\phi}$ e Teste $F$)
Deseja-se modelar o tempo de resposta hospitalar $Y > 0$ ($n = 60$ pacientes) usando distribuição **Gama** ($\phi = \nu^{-1}$ desconhecido). Ajustaram-se 4 modelos encaixados:
* $M_0$ (Nulo): $D_p^{(0)} = 42.00$
* $M_1$ ($+ X_1$ contínua): $D_p^{(1)} = 39.80$
* $M_2$ ($+ X_2$ fator categórico com $3$ níveis): $D_p^{(2)} = 18.50$
* $M_3$ (Maximal: $+ X_3$ contínua): $D_p^{(3)} = 17.90$ e estatística de Pearson generalizada $X_3^2 = 16.50$.
*(Valores críticos: $F_{1, 55; 0.95} = 4.016$; $F_{2, 55; 0.95} = 3.165$.)*

**(a) Calcule $p$ e $gl = n - p$ de cada modelo e estime $\hat{\phi}$ pelos três métodos no modelo maximal:**
* Parâmetros $p$ e $gl$:
  * $M_0$: $p=1 \implies gl_0 = 60 - 1 = \mathbf{59}$
  * $M_1$: $p=2 \implies gl_1 = 60 - 2 = \mathbf{58}$
  * $M_2$: fator $X_2$ tem $3$ níveis $\implies +(3-1)=2$ parâmetros $\implies p = 4 \implies gl_2 = 60 - 4 = \mathbf{56}$
  * $M_3$: $p = 5 \implies gl_3 = 60 - 5 = \mathbf{55}$
* Estimativas de $\hat{\phi}$ no **modelo maximal ($M_3$)**:
  1. **Método do Desvio:** $\hat{\phi}_D = \frac{D_p^{(3)}}{n - p_{\text{max}}} = \frac{17.90}{55} = \mathbf{0.3255}$
  2. **Método de Pearson:** $\hat{\phi}_P = \frac{X_3^2}{n - p_{\text{max}}} = \frac{16.50}{55} = \mathbf{0.3000}$
  3. **Aproximação EMV Cordeiro & McCullagh (1991):** $\hat{\phi}_{CM} = \frac{2(17.90)}{60\left[1 + \left(1 + \frac{2(17.90)}{3(60)}\right)^{1/2}\right]} = \frac{35.80}{60[1 + \sqrt{1.1989}]} \approx \mathbf{0.2848}$

**(b) Realize os Testes $F$ sequenciais (usando $\hat{\phi}_D = 0.3255$ do maximal) a $\alpha = 5\%$ e escolha o modelo final:**
* **$M_1$ vs $M_0$ ($\Delta gl = 1$):** $F_1 = \frac{(42.00 - 39.80)/1}{0.3255} = \frac{2.20}{0.3255} = \mathbf{6.760} > 4.016$ ($p = 0.0119 < 0.05$) $\implies$ **Rejeita $H_0$** ($X_1$ é significante!).
  *(🚨 Armadilha: se usássemos $\hat{\phi}_1 = 39.80/58 = 0.6862$ de $M_1$, obteríamos $F_{\text{errado}} = 2.20/0.6862 = 3.206 < 4.016$, mascarando o efeito de $X_1$ porque $X_2$ ainda faltava no modelo! Por isso o Slide 50 exige $\hat{\phi}_{\text{max}}$ em TODOS os passos!)*
* **$M_2$ vs $M_1$ ($\Delta gl = 2$):** $F_2 = \frac{(39.80 - 18.50)/2}{0.3255} = \frac{10.65}{0.3255} = \mathbf{32.723} > 3.165$ ($p < 0.0001$) $\implies$ **Rejeita $H_0$** ($X_2$ é altamente significante!).
* **$M_3$ vs $M_2$ ($\Delta gl = 1$):** $F_3 = \frac{(18.50 - 17.90)/1}{0.3255} = \frac{0.60}{0.3255} = \mathbf{1.844} < 4.016$ ($p = 0.1801 \ge 0.05$) $\implies$ **Não rejeita $H_0$** ($X_3$ não contribui significativamente).
* **Conclusão e Verificação de Deviance Escalonada:** Seleciona-se o **modelo parcimonioso $M_2$ ($X_1 + X_2$)**, com Pseudo-$R^2 = 1 - \frac{18.50}{42.00} = \mathbf{55.95\%}$. Note que $D_p^{(2)} = 18.50$ **não** pode ser comparado diretamente a $gl_2 = 56$; após escalonar por $\hat{\phi}_{\text{max}}$, tem-se $D^*_2 = 18.50 / 0.3255 = \mathbf{56.84} \approx gl_2 = 56$!

---

#### 📝 Exercício Resolvido 3 (Saídas `anova` Tipo I vs `Anova` Tipo II + Dados Binários Individuais — Lista 4 Kyphosis)
Um modelo logístico (`binomial(logit)`) para presença de cifose após cirurgia ($n = 81$ crianças) forneceu:
* Modelo 1 (`~ Age + Number + Start`): `Null dev = 83.234 (80 df)`, `Resid dev = 61.380 (77 df)`, `AIC = 69.38`
* Modelo 2 (`~ Start`): `Null dev = 83.234 (80 df)`, `Resid dev = 68.072 (79 df)`, `AIC = 72.072`
* `anova(ajuste1)` (Tipo I): `Age` ($\Delta D = 1.302$), `Number` ($\Delta D = 10.306$), `Start` ($\Delta D = 10.247$).
* `Anova(ajuste1)` (Tipo II): `Age` (`LR Chisq = 3.1565, p = 0.0756`), `Number` (`3.9191, p = 0.0477 *`), `Start` (`10.2466, p = 0.0014 **`).

**Pontos críticos de interpretação na prova:**
1. **Tipo I vs Tipo II:** Na tabela sequencial `anova`, `Number` é testada sem `Start` ($\Delta D = 10.306$); na `Anova` Tipo II, `Number` é avaliada **controlando por `Age` e `Start`** (`LR Chisq = 3.9191, p = 0.0477`). Já `Start` aparece com $\approx 10.247$ em ambas pois é a última variável a entrar na Tipo I!
2. **Comparação Modelo 1 vs Modelo 2 ($H_0: \beta_{\text{Age}} = \beta_{\text{Number}} = 0$):** $\Delta D = 68.072 - 61.380 = \mathbf{6.692}$ com $\Delta gl = 79 - 77 = \mathbf{2}$. Como $6.692 > \chi^2_{2; 0.95} = 5.991$ ($p = 0.0352 < 0.05$) e $\Delta AIC = 72.072 - 69.38 = 2.692 > 2$, **rejeita-se $H_0$**: conjuntamente, `Age` e `Number` melhoram o ajuste em relação a usar só `Start`.
3. **🚨 Pegadinha de Prova:** Alguém pode dizer que `Residual deviance = 61.380` com `77 df` ($D/gl = 0.80$) prova bom ajuste? **NÃO!** Como os dados são individuais binários ($m_i = 1$), $D_p$ não converge para $\chi^2_{77}$. A bondade de ajuste exige **Hosmer-Lemeshow** (mas o TRV $\Delta D = 6.692$ entre modelos encaixados continua 100% válido!).

---

### 1.9 O Que Escrever na Folha de Resposta (Modelos Prontos para a Prova)
* **Modelo 1 ($\phi = 1$ conhecido — Teste $\chi^2$):**
  > *"Para testar a significância do(s) termo(s) adicionado(s), realizou-se a Análise do Desvio (ANODEV) via Teste da Razão de Verossimilhanças entre os modelos encaixados, testando $H_0: \beta_{\text{adicionais}} = 0$ contra $H_1: \text{pelo menos um } \beta_j \ne 0$. Como o parâmetro de dispersão é conhecido ($\phi = 1$), utilizou-se $\Delta D^* = \Delta D = D_m - D_q = [\text{valor}]$ com $\Delta gl = [\text{valor}]$ graus de liberdade, que segue $\chi^2_{\Delta gl}$ sob $H_0$. Como $\Delta D > \chi^2_{\Delta gl; 0.95}$ ($p\text{-valor} = [\text{valor}] < 0.05$), rejeita-se $H_0$ ao nível de 5%, concluindo que o(s) termo(s) melhora(m) significativamente o ajuste e deve(m) permanecer no modelo."*
* **Modelo 2 ($\phi$ desconhecido — Teste $F$ com $\hat{\phi}_{\text{max}}$):**
  > *"Como $\phi$ é desconhecido (distribuição [Normal/Gama]), as deviances impressas são não-escalonadas ($D_p$). Estimou-se a dispersão a partir do modelo maximal ($M_{\text{max}}$ com $gl = [\text{valor}]$) pelo método [de Pearson / do Desvio], obtendo $\hat{\phi}_{\text{max}} = [\text{valor}]$. Aplicou-se a estatística $F = \frac{(D_m - D_q)/\Delta gl}{\hat{\phi}_{\text{max}}} = [\text{valor}]$ sob distribuição $F_{\Delta gl,\, gl_{\text{max}}}$. Como $F_{\text{calc}} [< \text{ou} >]\; F_{\text{tab}}$ ($p = [\text{valor}]$), [não rejeita-se / rejeita-se] $H_0$, optando-se pelo modelo [reduzido / completo] segundo o princípio da parcimônia."*

---

# ETAPA 2: GUIA COMPLETO DE DIAGNÓSTICOS DO MLG

### 2.0 Robustez Assintótica, Tríade de Testes, IC pela Deviance e Convenção de $\phi$ (Slides 20–27, 90–95)

1. **Robustez à Má Especificação de $\phi$ vs. Função de Ligação:**
   - **Média bem especificada ($\eta_i = x_i^T\beta$, ligação $g(\mu_i)$ correta):** O estimador de Máxima Verossimilhança $\hat{\beta}$ permanece consistente mesmo se houver erro na variância assumida ($\phi \neq 1$ em Binomial/Poisson). **PORÉM**, os erros-padrão ingênuos ($EP_{\text{ingênuo}}$ com $\phi=1$) ficam subestimados na superdispersão ($\implies z$ inflado $\implies$ **falsos positivos**!).
   - **Ligação ou preditor errados:** $\hat{\beta}$ torna-se **viciado e inconsistente**; corrigir $\phi$ não salva o modelo!
2. **Tríade Assintótica de Testes para $H_0: C\beta = d$ (ou $\beta_j = 0$):**
   - **Teste de Wald:** $W = (C\hat{\beta}-d)^T [C\,\widehat{\text{Cov}}(\hat{\beta})\,C^T]^{-1}(C\hat{\beta}-d) \sim \chi^2_q$ (para 1 parâmetro: $z = \hat{\beta}_j / EP(\hat{\beta}_j)$). Usa apenas o modelo completo. **Armadilha (Hauck-Donner):** sob separação completa ou $\hat{\pi} \approx 0$ ou $1$, $EP(\hat{\beta}_j) \to \infty$ e $W \to 0$, gerando falso negativo mesmo para variável fortíssima!
   - **Teste da Razão de Verossimilhanças (TRV / $\Delta D$):** Compara modelo completo ($M_1$) vs. reduzido ($M_0$) via $RV = 2(\ell_1 - \ell_0) = D^*_0 - D^*_1$. É **invariante a reparametrizações** e **não sofre do efeito Hauck-Donner**.
   - **Teste do Escore de Rao:** Avalia o gradiente $U(\beta)$ apenas sob $H_0$: $S_R = U(\hat{\beta}_0)^T \mathcal{I}(\hat{\beta}_0)^{-1} U(\hat{\beta}_0) \sim \chi^2_q$. Não exige ajustar $M_1$; para variáveis categóricas/contagens equivale ao clássico $\chi^2$ de Pearson.
3. **IC de Wald vs. IC Baseado na Deviance (`confint()` no R):**
   - **IC de Wald:** $\hat{\beta}_j \pm z_{\alpha/2}EP(\hat{\beta}_j)$ — sempre simétrico em $\beta$; ruim em amostras pequenas ou assimetria forte da log-verossimilhança.
   - **IC da Deviance (Perfil de Verossimilhança):** $\{\beta_j : D^*(\beta_j, \hat{\beta}_{-j}(\beta_j)) - D^*(\hat{\beta}) \le \chi^2_{1; 1-\alpha}\}$ — captura a **assimetria natural** pelas duas raízes da curva de deviance perfilhada e respeita melhor regiões próximas a separação/fronteira.

> #### 🚨 Alerta de Dupla Convenção de $\phi$ (Paula vs. Williams 1987 / R)
> O parâmetro de escala $\phi$ aparece em **duas convenções inversas** no curso — confira sempre qual delas está sendo usada antes de aplicar uma fórmula:
> - **Convenção Precisão (Paula, Slides 31–39):** $\text{Var}(Y_i) = \phi^{-1} V(\mu_i)$. Aqui $\phi$ é **precisão**, portanto $\hat{\phi}$ aparece no **numerador** ($D^* = \hat{\phi} D$, $r_{Di} = \hat{\phi}^{1/2}d_i/\sqrt{1-h_{ii}}$, $r_{Pi} = \hat{\phi}^{1/2}(y_i-\hat{\mu}_i)/\sqrt{V(\hat{\mu}_i)(1-h_{ii})}$, $LD_i \propto \hat{\phi}$, Cook $D_i \propto \hat{\phi}$).
> - **Convenção Dispersão (Williams 1987, Slide 40 e saída `Dispersion parameter` do R):** $\text{Var}(Y_i) = \phi_{\text{disp}} V(\mu_i)$, em que $\phi_{\text{disp}} = \phi_{\text{prec}}^{-1}$. Portanto $\hat{\phi}_{\text{disp}}$ aparece no **denominador** ($D^* = D / \hat{\phi}_{\text{disp}}$, $r_{Di} = d_i / \sqrt{\hat{\phi}_{\text{disp}}(1-h_{ii})}$, $r_{Pi} = (y_i-\hat{\mu}_i)/\sqrt{\hat{\phi}_{\text{disp}} V(\hat{\mu}_i)(1-h_{ii})}$).
> - **Regra prática na prova:** Em **Binomial e Poisson padrão**, $\phi = 1 \implies \phi_{\text{prec}} = \phi_{\text{disp}} = 1$ e todas as expressões coincidem numericamente! Em **Quasi-Binomial, Gama ou Normal**, olhe o output do R: `Dispersion parameter` é sempre $\hat{\phi}_{\text{disp}}$.

### 2.1 A Relação entre Residual Deviance e os Graus de Liberdade ($D$ vs $gl$)
Uma das perguntas conceituais mais cobradas em provas de MLG é a relação direta entre o valor numérico da deviance residual e seus graus de liberdade:
* **Teoria Assintótica (somente se $\phi$ for conhecido ou após dividir por $\hat{\phi}$):** Se o modelo estiver corretamente especificado e o tamanho amostral for adequado, temos:
  $$E(D^*) \approx n - p \implies \frac{D^*}{n - p} = \frac{D_p}{\phi(n - p)} \approx 1$$
* **Diagnóstico da Razão $\frac{D_p}{n - p}$ (quando $\phi = 1$, ex.: Poisson e Binomial agrupada):**
  1. **Ajuste Ideal:** $\frac{D_p}{n - p} \approx 1.0$. O modelo é parcimonioso e reproduz fielmente a variabilidade esperada.
  2. **Sobredispersão (*Overdispersion*):** $\frac{D_p}{n - p} \gg 1.0$ (ex.: $> 1.5$ ou $> 2.0$). A variabilidade observada é substancialmente maior do que a teórica permitida pelo modelo. *Causas:* omissão de preditoras relevantes, excesso de zeros, correlação entre observações (clusters) ou má especificação da função de variância.
  3. **Subdispersão (*Underdispersion*):** $\frac{D_p}{n - p} \ll 1.0$. Rara na prática; pode indicar superajuste (*overfitting*) ou correlação negativa entre observações.
* **⚠️ Lembrete quando $\phi$ é desconhecido (Normal, Gama, Normal Inversa):** Como o R imprime $D_p$ (não-escalonada), $\frac{D_p}{n-p}$ **NÃO deve valer 1** — esse quociente é na verdade $\hat{\phi}_D$ (a estimativa da dispersão, como $QMRes$)!

#### 🚨 O Paradoxo e Falha Fatal em Dados Binários Puros ($y_i \in \{0, 1\}$ / $m_i = 1$)
* **Por que a relação $D \approx n - p$ falha em dados individuais?**
  Em dados binários puros ($m_i = 1$), a deviance residual tem a expressão:
  $$D = -2 \sum_{i=1}^n \left[ y_i \ln(\hat{\pi}_i) + (1 - y_i) \ln(1 - \hat{\pi}_i) \right]$$
  Quando a ligação canônica + intercepto estão presentes, ela reduz-se algebricamente a $-2\sum [\hat{\pi}_i\ln\hat{\pi}_i + (1-\hat{\pi}_i)\ln(1-\hat{\pi}_i)]$, dependendo apenas dos $\hat{\pi}_i$ ajustados e **NÃO converge para a distribuição $\chi^2_{n-p}$** quando $n \to \infty$!
* **Conclusão Obrigatória de Prova:** Em dados binários individuais, a razão $\frac{D}{n - p}$ **não mede sobredispersão** e a deviance residual **NÃO serve como teste de qualidade de ajuste global**. A ferramenta correta é o **Teste de Hosmer-Lemeshow**.

---

### 2.2 Todos os Tipos de Resíduos em MLG

*(Abaixo apresentamos as fórmulas na convenção de dispersão do R/Williams $\text{Var}(Y_i) = \hat{\phi}_{\text{disp}} V(\mu_i)$; na notação de precisão de Paula $\phi_{\text{prec}} = 1/\phi_{\text{disp}}$, substitua $1/\sqrt{\hat{\phi}_{\text{disp}}}$ por $\hat{\phi}_{\text{prec}}^{1/2}$ no numerador.)*

| Tipo de Resíduo | Fórmula Matemática | Por que / Quando Fazer | Limitações |
| :--- | :---: | :--- | :--- |
| **Ordinário** | $e_i = y_i - \hat{\mu}_i$ | Medida ingênua de erro observada menos predita. | **Falha em MLG:** heterocedasticidade intrínseca ($\text{Var}(Y_i) = \phi\, V(\mu_i)$) e assimetria da distribuição. **Não usar.** |
| **Pearson Bruto** | $r_{P,i} = \frac{y_i - \hat{\mu}_i}{\sqrt{V(\hat{\mu}_i)}}$ | Pondera o erro pelo desvio padrão da função de variância. Base de $X^2 = \sum r_{P,i}^2$. | Ignora alavancagem e assimetria; afasta-se muito da Normalidade em Binomial/Poisson assimétricos. |
| **Pearson Padronizado** | $r_{P,i}^s = \frac{y_i - \hat{\mu}_i}{\sqrt{\hat{\phi}_{\text{disp}} \hat{V}_i (1 - h_{ii})}}$ | Corrige por dispersão e alavancagem $h_{ii}$ para variância $\approx 1$. | Distribuição ainda assimétrica nas caudas frente ao deviance padronizado. |
| **Anscombe (Slide 35)** | $r_i^A = \frac{\psi(y_i) - \psi(\hat{\mu}_i)}{\sqrt{\hat{\phi}_{\text{disp}}\hat{V}_i}\,\psi'(\hat{\mu}_i)}$, $\psi(\mu)=\int_0^\mu V^{-1/3}(t)\,dt$ | Transforma $y_i$ para aproximar simetria/normalidade ($\psi$: Poisson $y^{2/3}$, Gama $y^{1/3}$, Norm. Inv. $\ln y$, Binomial $B(y/m; 2/3,2/3)$). | Exige função especial na Binomial e, sem $(1-h_{ii})^{-1/2}$, não corrige alavancagem. |
| **Deviance Bruto** | $d_i = \text{sign}(y_i - \hat{\mu}_i) \sqrt{d(y_i; \hat{\mu}_i)}$ | Contribuição individual com sinal para a deviance ($D_p = \sum d_i^2$). | Variância menor que 1 em pontos com alta alavancagem ($h_{ii}$ alto). |
| **Deviance Padronizado** | $r_{D,i}^s = \frac{d_i}{\sqrt{\hat{\phi}_{\text{disp}}(1 - h_{ii})}}$ | **Favorito padrão (Williams, 1987).** Corrige assimetria e alavancagem $h_{ii}$. | Requer cálculo da matriz chapéu $\mathbf{H}$ em cada ponto. |
| **Verossimilhança ($r_{Gi}$, Williams 1987)** | $r_{Gi} = \text{sign}(y_i - \hat{\mu}_i)\sqrt{(1-h_{ii})(r_{D,i}^s)^2 + h_{ii}(r_{P,i}^s)^2}$ | Mede diretamente a redução na deviance escalonada sob deleção ($r_{Gi}^2 \approx G_i$); fica entre $r_{D,i}^s$ e $r_{P,i}^s$. | Uso principal na detecção de *outliers* e *Half-Normal Plots*; menos impresso por padrão. |

* **Por que o Resíduo Deviance Padronizado ($r_{D,i}^s$) e $r_{Gi}$ são Superiores? (Williams, 1987 — Slide 40 de `mlgp2.pdf`):**
  Williams demonstrou que $r_{D,i}^s$ e $r_{Gi}$ levam em conta a forma assimétrica da distribuição de $Y_i$ (ignorada por Pearson), apresentando distribuição empírica muito mais próxima da Normal Padrão $N(0, 1)$, o que legitima a referência $[-2, +2]$ ou $[-3, +3]$ em inspeções gráficas.

---

### 2.3 Todos os Gráficos Diagnósticos em MLG

#### 1. Gráfico de Resíduos ($t_{Di}$) versus Índice da Observação
* **Quando fazer / Por que fazer:** Após ajustar o modelo, para detectar *outliers* pontuais, erros de digitação e dependência serial ao longo da ordem de coleta.
* **Interpretação & Limitações:** Pontos aleatórios em torno de $0$, quase todos em $[-2, +2]$ (pontos fora de $\pm 3$ são suspeitos). Não detecta má forma funcional de $\eta$ ou $g(\mu)$.

#### 2. Gráfico de Resíduos ($t_{Di}$) versus Valores Ajustados ($\hat{\eta}_i$ ou $\hat{\mu}_i$) e versus $X_k$ (Slide 34)
* **Por que fazer ($t_{Di}$ vs $\hat{\eta}_i$ / $\hat{\mu}_i$):** Detecta simultaneamente **dispersão não constante** (funil/megafone $\implies V(\mu)$ errada), **observações correlacionadas**, **erro na função de ligação** (curvatura) e ***outliers***.
* **Por que fazer ($t_{Di}$ vs $X_k$):** Avalia se falta algum termo não linear ou covariável no componente sistemático (ex.: curvatura parabólica pede $X_k^2$).
* **Limitação em Dados Binários Individuais ($y \in \{0, 1\}$):** Formam-se **duas curvas distintas** ($y=1$ acima, $y=0$ abaixo), tornando padrões sutis difíceis de ler a olho nu (o foco recai sobre pontos isolados fora de $[-2, 2]$).

#### 3. Gráfico Meio-Normal com Envelope Simulado (`hnp` / Atkinson — Slides 36–38 & 102)
* **Quando e por que fazer:** Como resíduos de MLG não têm normalidade exata, gera-se via Monte Carlo (ex.: $m=19$ ou **$m = 99$ amostras simuladas com percentis 5%, 50% e 95%**) uma banda empírica sob a hipótese de que o modelo ajustado é verdadeiro.
* **Estatística de Ordem Meio-Normal (Slide 38):** Plota-se $|r_{(i)}|$ contra $\Phi^{-1}\!\left(\frac{i + 1 - 1/8}{2n + 1/2}\right)$.
* **🎯 Diagnóstico Diferencial Obrigatório na Prova:**
  * **Pontos dentro das bandas simuladas:** Modelo adequado (não exige reta de 45°!).
  * **Apenas os maiores resíduos absolutos acima da banda superior:** indica presença de ***outliers* isolados** inflando a deviance.
  * **TODOS os resíduos (pequenos e grandes) sistematicamente fora da banda:** indica **superdispersão** (ex.: caso *Rotifers*), **distribuição inadequada** ou **ligação incorreta**!

#### 4. Gráficos de Alavancagem ($h_{ii}$) e Distância de Cook ($D_i$)
* **Cook vs Índice:** Destaca observações cujo afastamento de parâmetros é desproporcional ($D_i > 2p/n$, $>4/n$ ou pico isolado no gráfico).
* **Resíduos Padronizados vs Alavanca ($h_{ii}$):** Decompõe se o ponto é *outlier* puro em $Y$, ponto extremo em $\mathbf{X}$ ou ambos.

---

### 2.4 Medidas e Testes de Influência (*Leave-One-Out* e Pregibon 1981)

#### 1. Ponto de Alavancagem (*Leverage* — $h_{ii}$, Slide 32 & Williams 1987)
* **Fórmula geral:** $\hat{\mathbf{H}} = \hat{\mathbf{W}}^{1/2} \mathbf{X} (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X})^{-1} \hat{\mathbf{W}}^{1/2}$, com **peso geral**:
  $$w_i = \left[ V(\mu_i)\left(\frac{d\eta_i}{d\mu_i}\right)^2 \right]^{-1}$$
  *(Só reduz a $w_i = V(\mu_i)$ sob ligação **canônica**!)*
* **Propriedades e Lição de Williams:** Com intercepto, $\frac{1}{n} \le h_{ii} \le 1$, $\sum h_{ii} = p$ e corte $h_{ii} > \frac{2p}{n}$. **Crucial:** diferente da regressão normal, em MLG $h_{ii}$ depende tanto de $\mathbf{X}$ quanto de $Y$ (via $w_i$): um ponto extremo em $X$ pode ter $h_{ii}$ *baixo* se $w_i$ for pequeno!

#### 2. Aproximação de 1 Passo, $LD_i$ (Slide 39) e Distância de Cook Generalizada ($D_i$, Slide 40)
* **Um passo de Pregibon (1981):** evita reajustar $n$ vezes para estimar $\hat{\boldsymbol{\beta}}_{(i)}^1$.
* **Afastamento pela Verossimilhança ($LD_i$):** $LD_i = 2[L(\hat{\boldsymbol{\beta}}) - L(\hat{\boldsymbol{\beta}}_{(i)})] \cong \frac{(\hat{\boldsymbol{\beta}}-\hat{\boldsymbol{\beta}}_{(i)})^\top(\mathbf{X}^\top\hat{\mathbf{W}}\mathbf{X})(\hat{\boldsymbol{\beta}}-\hat{\boldsymbol{\beta}}_{(i)})}{\phi_{\text{disp}}}$ *(na notação precisão de Paula, $\phi_{\text{prec}}$ multiplica o numerador)*.
* **Distância de Cook Generalizada (Williams, 1987):**
  $$D_i = \frac{(\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})^\top (\mathbf{X}^\top \hat{\mathbf{W}} \mathbf{X}) (\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})}{p \, \hat{\phi}_{\text{disp}}} = \frac{h_{ii}}{p(1 - h_{ii})} (r_{P,i}^s)^2$$
* **DFBETAS ($DFB_{j,i}$):** $\frac{\hat{\beta}_j - \hat{\beta}_{j(i)}}{\text{EP}(\hat{\beta}_{j(i)})}$ — avalia o impacto isolado de $i$ sobre um parâmetro específico $\beta_j$. (Conduta obrigatória: reajustar sem o caso $i$ para checar estabilidade!)

---

# ETAPA 3: GUIA COMPLETO DO MODELO DE REGRESSÃO LOGÍSTICA

### 3.1 Quando Usar e Por Que a Regressão Linear Falha
* **Quando usar:** Quando a variável resposta $Y$ é qualitativa binária (sucesso/fracasso, doente/sadio, comprou/não comprou), ou seja, $Y_i \sim \text{Bernoulli}(\pi_i)$, ou proporções em dados agrupados $Y_i \sim \text{Binomial}(m_i, \pi_i)$.
* **Por que a regressão linear clássica ($Y = X\beta + \epsilon$) falha completamente:**
  1. **Probabilidades Fora do Intervalo:** A reta linear prediz valores $\hat{Y} < 0$ e $\hat{Y} > 1$, o que viola os axiomas de probabilidade.
  2. **Heterocedasticidade Estrutural:** Como a variância de uma v.a. Bernoulli é $\text{Var}(Y) = \pi(1-\pi)$, ela depende diretamente da média. A homocedasticidade linear é violada.
  3. **Erros Não-Normais:** Os resíduos só podem assumir dois valores: $1 - \hat{\pi}$ ou $-\hat{\pi}$.

---

### 3.2 O Passo a Passo Matemático da Regressão Logística
1. **Definição da Média e Chance:**
   $$\pi_i = P(Y_i = 1 \mid x_i), \quad \text{odds}(\pi_i) = \frac{\pi_i}{1 - \pi_i}$$
2. **Função de Ligação Canônica (Logito):**
   $$\eta_i = \text{logit}(\pi_i) = \ln\left( \frac{\pi_i}{1 - \pi_i} \right) = \beta_0 + \beta_1 x_{i1} + \cdots + \beta_p x_{ip}$$
3. **Função Média Inversa (Probabilidade Predita):**
   $$\pi_i = \frac{e^{\eta_i}}{1 + e^{\eta_i}} = \frac{1}{1 + e^{-\eta_i}}$$
4. **Estimação dos Parâmetros:** Realizada pelo método da Máxima Verossimilhança através do algoritmo iterativo de Mínimos Quadrados Re-ponderados (IRLS / Fisher-scoring), resolvendo o sistema de equações de escore:
   $$U(\beta) = X^T (y - \mu) = 0$$

---

### 3.3 Interpretação Completa dos Coeficientes, Codificação, Linearidade & Odds Ratio (OR)
* **Intercepto ($\beta_0$):** Log da chance de sucesso quando todas as variáveis estão em $0$: $\text{odds}_{\text{base}} = e^{\beta_0}$ ($\pi(0) = \frac{e^{\beta_0}}{1+e^{\beta_0}}$).
* **Coeficiente $\beta_j$:** Variação no *log da chance* para aumento de 1 unidade em $X_j$, **mantendo as demais covariáveis fixas**.
* **Odds Ratio ($OR$):**
  * **Variável Dicotômica Usual ($X \in \{0, 1\}$):** $OR = e^{\beta_1}$.
  * **🚨 Armadilha de Codificação ($X \in \{-1, +1\}$, Slide 62):** Como o salto de $-1$ até $+1$ tem distância $2$, $OR = e^{2\beta_1}$!
  * **Comparação entre Dummies Não-Referência (Slide 87):** $OR_{B \mid A} = e^{\hat{\beta}_B - \hat{\beta}_A}$.
  * **Contínua com Incremento de $c$ Unidades (Slide 63):**
    $$OR_c = e^{c \cdot \beta_j} \qquad (\text{🚨 nunca multiplique } c \times OR!)$$
  * **Variação Percentual na Chance:** $\Delta \% = (OR - 1) \times 100\%$ (ex.: $OR = 1.40 \implies +40\%$; $OR = 0.70 \implies -30\%$).
  * **Intervalo de Confiança ($1-\alpha$) para o $OR_c$:**
    $$\text{IC}_{95\%}(OR_c) = \left[ e^{c\hat{\beta}_j - 1.96 \cdot c \cdot \text{EP}(\hat{\beta}_j)}, \; e^{c\hat{\beta}_j + 1.96 \cdot c \cdot \text{EP}(\hat{\beta}_j)} \right]$$
    *Regra de Ouro: Se o IC para $OR$ contiver $1.0$, o efeito NÃO é significativo a 5%.*
* **Verificação de Linearidade no Logito (Slides 63–64):** Checa-se por **dois métodos**: (1) incluir termos polinomiais ($X^2, X^3$) e testar significância; (2) categorizar $X$ contínuo e plotar o **logito amostral (empírico)** contra os pontos médios das classes.
* **Variância e IC da Probabilidade Predita $\pi(\mathbf{x}_0)$ (Slide 65):**
  $$\widehat{\text{Var}}(\hat{\alpha}+\hat{\beta}x_0) = \widehat{\text{Var}}(\hat{\alpha}) + x_0^2\widehat{\text{Var}}(\hat{\beta}) + 2x_0\widehat{\text{Cov}}(\hat{\alpha},\hat{\beta})$$
  Construa o IC sempre na escala link ($\hat{\eta}_0 \pm 1.96\,EP(\hat{\eta}_0)$) e aplique $\frac{e^\eta}{1+e^\eta}$ nos extremos!

---

### 3.4 Risco Atribuível ($d$), Risco Relativo ($RR$) vs. Odds Ratio ($OR$) (Slides 67–68)
* **Em Tabela $2\times 2$ ($a$: ex./doente; $b$: ex./sadio; $c$: não-ex./doente; $d_{\text{cel}}$: não-ex./sadio):**
  $$\pi_1 = \frac{a}{a+b}, \quad \pi_0 = \frac{c}{c+d_{\text{cel}}}, \quad d = \pi_1 - \pi_0 \text{ (Risco Atribuível)}, \quad RR = \frac{\pi_1}{\pi_0}, \quad OR = \frac{\pi_1/(1-\pi_1)}{\pi_0/(1-\pi_0)} = \frac{a \cdot d_{\text{cel}}}{b \cdot c}$$
* **Hipótese da Doença Rara (Regra de Cornfield):** Só quando o desfecho é raro **em ambos os grupos** ($\max(\pi_1, \pi_0) < 0.10$, melhor $< 0.05$) vale $1-\pi_1 \approx 1-\pi_0 \approx 1 \implies \mathbf{OR \approx RR}$. Em eventos comuns (como o Titanic!), o $OR$ amplifica fortemente o $RR$.
* **Caso-Controle vs. Coorte:** Em **Caso-Controle**, fixam-se os totais de doentes/sadios, logo **não se estimam $\pi_1, \pi_0$ nem $RR$**, mas o $OR = \frac{ad}{bc}$ é invariante ao delineamento!

---

### 3.5 Construção, Hipóteses de Interação, Diagnóstico & Eventos Raros (Slides 71–88)
* **Tabela de Hipóteses com Interação ($\text{logit}\,\pi = \beta_0 + \beta_1\text{Idade} + \beta_2\text{Sexo} + \beta_3\,\text{Idade}\!\times\!\text{Sexo}$, Slide 71):**
  * $H_0: \beta_3 = 0 \implies$ Efeitos aditivos (retas paralelas no logito; sem interação).
  * $H_0: \beta_1 = \beta_3 = 0 \implies$ Idade não está associada a $Y$.
  * $H_0: \beta_2 = \beta_3 = 0 \implies$ Sexo não está associado a $Y$.
  * $H_0: \beta_1 = \beta_2 = \beta_3 = 0 \implies$ Nem Idade nem Sexo estão associados a $Y$.
* **Stepwise & Regra do EPV (Slides 72 & 77):** Stepwise inicia com $P_E \in [0.15, 0.25]$ (ex.: $0.20$) para não perder confundidores antes de testar interações; exigir **EPV $\ge 10\text{–}15$ sucessos por variável** e reagrupar categorias esparsas.
* **Peso na Logística e Caso #172 (Slides 82–84):** Peso $w_i = m_i\hat{\pi}_i(1-\hat{\pi}_i)$ (máximo em $\hat{\pi}_i = 0.5$ e próximo de zero nos extremos). Sem o paciente #172, `sexo:ff` saltou de $\hat{\beta}=-3.3494$ ($EP=1.480, p=0.0236$) para $\hat{\beta}=-19.3711$ ($EP=1357.18, p=0.9886$) $\implies$ remover a interação instável!
* **Eventos Raros & Separação (Slides 84 & 88):** Com pouquíssimos sucessos (ex.: 20/1.000), ocorre viés e não-convergência ($\hat{\beta}\to\pm\infty$, $EP$ explode). Soluções dos slides: **Logística Exata** ($N<200$, preditores dicotômicos), **Correção de viés** (`relogit` no Stata) e **Verossimilhança Penalizada de Firth** (`firthlogit` / pacote `logistf` no R).

---

# ETAPA 4: TODOS OS TESTES DO MODELO LOGÍSTICO (COM $H_0$, ESTATÍSTICA E INTERPRETAÇÃO)

### Teste 1: Teste Global do Modelo (TRV entre Modelo Nulo e Proposto)
* **Objetivo:** Verificar se o conjunto das covariáveis incluídas no modelo explica significativamente a probabilidade de ocorrência do evento.
* **Hipóteses Formais:**
  $$H_0: \beta_1 = \beta_2 = \cdots = \beta_p = 0 \quad \text{vs} \quad H_1: \text{pelo menos um } \beta_j \ne 0$$
* **Estatística do Teste:**
  $$\Delta D = D_{\text{nula}} - D_{\text{residual}} \sim \chi^2_{gl_{\text{nulo}} - gl_{\text{residual}}} = \chi^2_p$$
* **Interpretação para a Folha de Resposta (Dados Agrupados — Exemplo Cupons, Slide 43):**
  > *"No exemplo dos cupons de desconto ($N = 7$ níveis de desconto, 200 cupons cada), a deviance caiu de $D_0 = 442.24$ ($gl_0 = 6$) no modelo nulo para $D_1 = 2.1571$ ($gl_1 = 5$) ao incluir `desconto`. A estatística do TRV é $\Delta D = 440.08$ com $\Delta gl = 1$ ($p < 0.0001$), rejeitando $H_0$ e comprovando efeito altamente significativo do desconto (com $R^2 = 99.51\%$ e $p(\chi^2_5 \ge 2.1571) = 0.83$ indicando ótimo ajuste global)."*

---

### Teste 2: Teste de Wald para Coeficientes Individuais
* **Objetivo:** Testar se o efeito de uma covariável específica $X_j$ é estatisticamente significante na presença das demais variáveis.
* **Hipóteses Formais:**
  $$H_0: \beta_j = 0 \quad \text{vs} \quad H_1: \beta_j \ne 0$$
* **Estatística do Teste:**
  $$z = \frac{\hat{\beta}_j}{\text{EP}(\hat{\beta}_j)} \sim N(0, 1) \quad \text{ou} \quad W = z^2 \sim \chi^2_1$$
* **Interpretação para a Folha de Resposta:**
  > *"Para a covariável `sexo`, o coeficiente estimado é $\hat{\beta} = 1.4023$ com erro padrão $\text{EP} = 0.5837$, resultando na estatística de Wald $z = 2.403$. Como a probabilidade associada é $p = 0.0163 < 0.05$, rejeitamos a hipótese nula $H_0: \beta_{\text{sexo}} = 0$ ao nível de significância de 5%. Conclui-se que a variável sexo possui associação estatisticamente significante com a ocorrência do processo maligno."*
* **🚨 Limitação (Efeito Hauck-Donner):** Quando o efeito real é muito grande ou há separação de dados, o erro padrão infla excessivamente, fazendo a estatística de Wald $z$ cair para próximo de zero. Nessa situação, Wald falha e deve-se recorrer obrigatoriamente ao TRV!

---

### Teste 3: Teste de Bondade de Ajuste de Hosmer-Lemeshow (`hoslem.test`) e Limitações (Slides 74–76)
* **Por que fazer:** Porque em dados binários individuais ($m_i=1$) a deviance residual e Pearson falham. *(Nota: em dados agrupados em $N$ subpopulações, $\chi^2_{N-p}$ só vale se a maioria dos $n_i > 10$, $\ge 80\%$ das frequências esperadas $\ge 5$ e todas $> 2$.)*
* **Mecanismo e Estatística:** Ordenam-se os $\hat{\pi}_i$ em $g = 10$ decis de risco (ou pontos fixos):
  $$\hat{C} = \sum_{k=1}^g \frac{(O_k - n'_k \bar{\pi}_k)^2}{n'_k \bar{\pi}_k (1 - \bar{\pi}_k)} \sim_a \chi^2_{g - 2} \qquad (g=10 \implies df = 8)$$
* **Hipóteses Formais (Decore com Atenção!):**
  $$H_0: \text{O modelo logístico ajusta-se adequadamente aos dados observados (Bom Ajuste)}$$
  $$H_1: \text{O modelo apresenta falta de ajuste (Mau Ajuste)}$$
* **Interpretação para a Folha de Resposta:**
  > *"O teste de Hosmer-Lemeshow com $g=10$ grupos apresentou $\hat{C} = 6.8216$ ($df = 8, p = 0.556$). Como $p = 0.556 > 0.05$, **NÃO rejeitamos $H_0$** a 5%, concluindo por calibração adequada (o mesmo ocorreu no modelo principal: $\hat{C} = 7.6435, p = 0.469$)."*
* **🚨 Limitações do Teste (Provas Discursivas):**
  1. Escolha arbitrária de $g = 10$ grupos (e sensibilidade a empates de $\hat{\pi}_i$);
  2. **Baixa potência estatística em amostras pequenas** ($n$ pequeno pode resultar em $p > 0.05$ mesmo com falta de ajuste);
  3. Avalia apenas **calibração** — deve sempre acompanhar a **Curva ROC (discriminação)** e gráficos de resíduos/influência.

### Leituras Complementares Rápidas (`logistic.display` — Slide 36):
* Na função `epiDisplay::logistic.display(modelo)`, lê-se `OR`, `lower95ci`, `upper95ci` e `Pr(>|Z|)`. Se o $IC_{95\%}(OR)$ **contiver $1.0$** (ex.: `sexo` $[0.92, 5.31]$, $p=0.075$; `ff` $[0.22, 1.34]$, $p=0.186$), o efeito **não é significativo** a 5%; caso contrário (`idade` $[1.040, 1.094]$, `hl` $[0.090, 0.449]$), há associação significativa.

---

### Teste 4: Tabela de Classificação $2 \times 2$ e Métricas Diagnósticas
Fixado o limiar $c = 0.5$:
* **Matriz de Confusão:**
  * $VP$ (Verdadeiro Positivo): Doente classificado como Doente.
  * $FP$ (Falso Positivo): Sadio classificado como Doente (Erro Tipo I).
  * $FN$ (Falso Negativo): Doente classificado como Sadio (Erro Tipo II).
  * $VN$ (Verdadeiro Negativo): Sadio classificado como Sadio.
* **Fórmulas e Interpretações:**
  * **Sensibilidade ($S$):** $S = \frac{VP}{VP + FN} = P(\hat{Y}=1 \mid Y=1)$. Capacidade do modelo de detectar os verdadeiramente positivos.
  * **Especificidade ($E$):** $E = \frac{VN}{VN + FP} = P(\hat{Y}=0 \mid Y=0)$. Capacidade do modelo de descartar os verdadeiramente negativos.
  * **Taxa de Falso Positivo ($1 - E$):** $1 - E = \frac{FP}{VN + FP} = P(\hat{Y}=1 \mid Y=0)$.
  * **Taxa de Falso Negativo ($1 - S$):** $1 - S = \frac{FN}{VP + FN} = P(\hat{Y}=0 \mid Y=1)$.
  * **Acurácia Global:** $\text{Acc} = \frac{VP + VN}{N}$. Proporção total de acertos do modelo.
  * **Valor Preditivo Positivo (VPP):** $\frac{VP}{VP + FP}$. Probabilidade de um indivíduo com teste positivo ser de fato doente.
  * **Valor Preditivo Negativo (VPN):** $\frac{VN}{VN + FN}$. Probabilidade de um indivíduo com teste negativo ser sadio.

---

### Teste 5: Curva ROC e Área sob a Curva (AUC)
* **Construção:** Gráfico da **Sensibilidade ($S$)** no eixo das ordenadas contra **$1 - \text{Especificidade}$ ($1-E$)** no eixo das abscissas, calculado para todos os possíveis pontos de corte $c \in [0, 1]$.
* **Significado Probabilístico da AUC:**
  $$AUC = P(\hat{\pi}_A > \hat{\pi}_B \mid Y_A = 1, Y_B = 0)$$
  Representa a probabilidade de o modelo atribuir uma probabilidade predita mais elevada a um indivíduo positivo sorteado aleatoriamente do que a um indivíduo negativo sorteado aleatoriamente.
* **Tabela de Classificação de Hosmer & Lemeshow para a AUC:**
  * $AUC = 0.50$: Capacidade discriminativa **Nula** (equivalente ao lançamento de moeda).
  * $0.50 < AUC < 0.70$: Discriminação **Pobre / Fraca**.
  * $0.70 \le AUC < 0.80$: Discriminação **Aceitável**.
  * $0.80 \le AUC < 0.90$: Discriminação **Excelente / Boa** (exemplo dos slides: $AUC = 0.8539$).
  * $AUC \ge 0.90$: Discriminação **Excepcional / Quase Perfeita**.
* **Interpretação para a Folha de Resposta:**
  > *"A Área sob a Curva ROC calculada foi $AUC = 0.8539$ (85,39%). De acordo com os critérios clínicos de Hosmer e Lemeshow, o modelo enquadra-se na faixa $0.80 \le AUC < 0.90$, possuindo uma **excelente capacidade de discriminação**. Isso significa que em 85,39% dos pares de pacientes (um maligno e um benigno) selecionados ao acaso, o modelo atribuirá maior probabilidade de malignidade ao paciente verdadeiramente doente."*

---

# ETAPA 5: MODELOS PRONTOS DE RESPOSTA PARA PROVAS ("COPIE E COMPLETE")

### 5.1 Roteiro para "Interprete o Coeficiente da Variável Contínua $X_1$"
> *"O coeficiente estimado para a variável $X_1$ é $\hat{\beta}_1 = 0.0387$ ($p = 0.0251 < 0.05$). Para um aumento de $c = 10$ unidades em $X_1$, o Odds Ratio é calculado por $OR = e^{c \cdot \hat{\beta}_1} = e^{10 \times 0.0387} = e^{0.387} \approx 1.472$. Mantendo todas as demais variáveis constantes, para cada 10 unidades a mais em $X_1$, a chance de ocorrência do evento aumenta em $47.2\%$ ($[1.472 - 1] \times 100\%$)."*

### 5.2 Roteiro para "Interprete a Variável Categórica $X_2$ (Dummy)"
> *"A categoria de referência é $X_2 = 0$. O coeficiente para o nível $X_2 = 1$ é $\hat{\beta}_2 = 1.4023$ ($p = 0.0163 < 0.05$). O Odds Ratio associado é $OR = e^{1.4023} \approx 4.06$. Conclui-se que indivíduos com a característica $X_2 = 1$ possuem chance 4,06 vezes maior (ou um acréscimo de 306% na chance) de apresentar o desfecho em comparação aos indivíduos do grupo de referência ($X_2 = 0$), fixadas as demais variáveis."*

### 5.3 Roteiro para "Avalie a Bondade de Ajuste em Dados Binários Individuais"
> *"Tratando-se de dados binários individuais ($y_i \in \{0, 1\}$ e $m_i = 1$), a deviance residual e a estatística de Pearson não possuem distribuição assintótica qui-quadrado, sendo teoricamente inadequadas para o teste global de bondade de ajuste. Para esse propósito, emprega-se o Teste de Hosmer-Lemeshow. Com $g=10$ decis de risco, obteve-se $p\text{-valor} = 0.556 > 0.05$. Portanto, não se rejeita a hipótese nula $H_0$ de adequação, concluindo-se que o modelo apresenta calibração satisfatória e bom ajuste aos dados observados."*

### 5.4 Roteiro para "Decisão sobre Eliminação de Termo com Base em Ponto Influente"
> *"A análise da Distância de Cook revelou que a observação #172 é um ponto altamente influente. A comparação dos modelos demonstrou que a significância estatística da interação dependia exclusivamente desse único indivíduo (com ele, $p = 0.0236$; sem ele, o erro padrão explodiu para 1357 e $p = 0.9886$). Trata-se de um artefato de significância. Portanto, para assegurar a estabilidade e a generalização estatística, a interação deve ser removida do modelo final."*


---

# ETAPA 6: BANCO DE QUESTÕES TÍPICAS DA PROFESSORA ROSEMEIRE (GABARITOS & PADRÕES DE RESPOSTA SEM PROGRAMAÇÃO)

Esta seção compila exatamente a estrutura das atividades em sala (Atividades 4, 5, 7, 8 e Listas 4 e 5), fornecendo os cálculos manuais e os padrões textuais exigidos pela professora.

---

### 6.1 CASO 1: Regressão Logística em Transporte Urbano (Atividade 5)

#### Contexto e Modelo Ajustado
Estudo de fatores associados a acidentes com transporte urbano ($Y=1$ acidente, $Y=0$ não):
$$\log\left(\frac{\pi_i}{1-\pi_i}\right) = \beta_0 + \beta_1 \text{Horas}_i + \beta_2 \text{Treinamento}_i + \beta_3 \text{Experiência}_i$$

Tabela de Estimativas do Modelo:
| Parâmetro | Estimativa ($\hat{\beta}$) | Erro-Padrão ($EP$) |
| :--- | :--- | :--- |
| Intercepto ($\beta_0$) | -1.80 | 0.50 |
| HorasDirigidas ($\beta_1$) | +0.030 | 0.008 |
| Treinamento ($\beta_2$) | -0.70 | 0.25 |
| Experiência ($\beta_3$) | -0.015 | 0.010 |

---

#### Item (a): Significado dos Coeficientes no Contexto
* **$\beta_1 = +0.030$ (Horas Dirigidas):**
  $$OR = e^{0.030} \approx 1.0304 \implies (1.0304 - 1) \times 100\% = +3.04\%$$
  *Interpretação de Prova:* "Mantendo fixas as demais variáveis, a cada 1 hora a mais dirigida por semana, a chance de ocorrência de acidente aumenta em aproximadamente $3,0\%$ ($OR = 1.03$)."
* **$\beta_2 = -0.70$ (Treinamento Adicional):**
  $$OR = e^{-0.70} \approx 0.4966 \implies (0.4966 - 1) \times 100\% = -50.34\%$$
  *Interpretação de Prova:* "Fixadas as horas e a experiência, motoristas com treinamento adicional possuem uma chance de sofrer acidentes $50,3\%$ menor (ou $0.50$ vezes a chance) em comparação aos motoristas sem treinamento ($OR = 0.50$)."
* **$\beta_3 = -0.015$ (Experiência em Anos):**
  $$OR = e^{-0.015} \approx 0.9851 \implies (0.9851 - 1) \times 100\% = -1.49\%$$
  *Interpretação de Prova:* "Para cada ano adicional de experiência profissional na direção, a chance de acidente reduz-se em cerca de $1,5\%$ ($OR = 0.985$)."

---

#### Item (b): Estatística de Wald para a Variável Treinamento ($\alpha = 0.05$)
1. **Hipóteses:**
   $$H_0: \beta_2 = 0 \quad \text{vs} \quad H_1: \beta_2 \ne 0$$
2. **Cálculo da Estatística $z$ de Wald:**
   $$z_{\text{calc}} = \frac{\hat{\beta}_2}{EP(\hat{\beta}_2)} = \frac{-0.70}{0.25} = -2.80$$
   *(Ou na escala $\chi^2$: $W = z^2 = (-2.80)^2 = 7.84$)*.
3. **Decisão:**
   * Como $|z_{\text{calc}}| = 2.80 > 1.96$ (ou $W = 7.84 > \chi^2_{1; 0.95} = 3.84$, com $p\text{-valor} = 0.0051 < 0.05$):
   * **Rejeitamos $H_0$ ao nível de significância de 5%**.
4. **Conclusão no Contexto:**
   > *"Há evidências estatísticas altamente significativas de que a realização de treinamento adicional reduz o risco de acidentes entre os motoristas de transporte urbano."*

---

#### Item (c): Estimação da Probabilidade de Acidente
Perfil do motorista: $45$ horas dirigidas, $12$ anos de experiência, com treinamento ($\text{Treinamento} = 1$).
1. **Preditor Linear ($\hat{\eta}$):**
   $$\hat{\eta} = -1.80 + 0.030(45) - 0.70(1) - 0.015(12)$$
   $$\hat{\eta} = -1.80 + 1.35 - 0.70 - 0.18 = -1.33$$
2. **Probabilidade Predita ($\hat{\pi}$):**
   $$\hat{\pi} = \frac{e^{-1.33}}{1 + e^{-1.33}} = \frac{0.26447}{1 + 0.26447} = \frac{0.26447}{1.26447} \approx 0.2091 \quad (20,91\%)$$

---

#### Item (d): Classificação com Limiar de Decisão $c = 0.40$
* **Regra de Decisão:** Classificar como $\hat{Y} = 1$ (Risco de acidente) se $\hat{\pi} \ge 0.40$, e $\hat{Y} = 0$ caso contrário.
* **Aplicação:** Como $\hat{\pi} = 0.2091 < 0.40$, o motorista é classificado na **Classe 0 (Sem risco de acidente)**.

---

#### Item (e): Matriz de Confusão & Métricas com Limiar $c = 0.40$
Dados do teste ($N = 300$ motoristas):
| | Predito 1 (Alto Risco) | Predito 0 (Baixo Risco) | Total Real |
| :--- | :---: | :---: | :---: |
| **Real 1 (Acidente)** | $VP = 45$ | $FN = 30$ | $75$ |
| **Real 0 (Sem Acidente)** | $FP = 25$ | $VN = 200$ | $225$ |
| **Total Predito** | $70$ | $230$ | $N = 300$ |

1. **Acurácia Global:**
   $$\text{Acurácia} = \frac{VP + VN}{N} = \frac{45 + 200}{300} = \frac{245}{300} \approx 0.8167 \quad (81,67\%)$$
2. **Sensibilidade (*Recall*):**
   $$\text{Sensibilidade} = \frac{VP}{VP + FN} = \frac{45}{45 + 30} = \frac{45}{75} = 0.6000 \quad (60,00\%)$$
3. **Especificidade:**
   $$\text{Especificidade} = \frac{VN}{VN + FP} = \frac{200}{200 + 25} = \frac{200}{225} \approx 0.8889 \quad (88,89\%)$$
4. **Comentário de Prova:**
   > *"O modelo apresenta boa acurácia geral ($81.67\%$) e elevada especificidade ($88.89\%$), descartando corretamente os motoristas seguros. Contudo, sua sensibilidade é moderada ($60.0\%$), deixando de identificar $40\%$ dos motoristas que de fato se envolveram em acidentes (30 falsos negativos). Em transporte público, onde o custo humano de um acidente é crítico, recomenda-se reduzir o limiar de corte para aumentar a sensibilidade."*

---

#### Item (f): Capacidade Discriminatória da AUC ($0.84$) e Superioridade sobre a Acurácia
1. **Interpretação da $AUC = 0.84$:**
   > *"Segundo os critérios de Hosmer e Lemeshow ($0.80 \le AUC < 0.90$), o modelo possui uma **excelente capacidade de discriminação**. Em 84% das vezes em que selecionarmos ao acaso um motorista que sofreu acidente e outro que não sofreu, o modelo atribuirá uma probabilidade de risco maior àquele que realmente sofreu acidente."*
2. **Por que a AUC é Mais Robusta que a Acurácia?**
   * **Invariância ao Limiar de Corte:** A acurácia depende exclusivamente de um limiar arbitrário fixado previamente ($c = 0.40$ ou $0.50$). A AUC avalia o desempenho do classificador em **todos os possíveis pontos de corte simultaneamente** ($c \in [0, 1]$).
   * **Robustez ao Desbalanceamento de Classes:** Quando uma classe é majoritária (neste exemplo, 225 motoristas sem acidente contra 75 com acidente, proporção $3:1$), um modelo ingênuo que classifique todos como 0 já teria $75\%$ de acurácia sem qualquer valor preditivo. A AUC não é inflada pela prevalência das classes, pois pondera sensibilidade e especificidade de forma equilibrada.

---

#### Item (g): Modelo com Interações e Coeficientes com Sinais Opostos
Modelo expandido:
$$\eta = \beta_0 + \beta_1 \text{Horas} + \beta_2 \text{Exp} + \beta_3 \text{Trein} + \beta_4 \text{Turno} + \beta_5(\text{Horas} \times \text{Exp}) + \beta_6(\text{Trein} \times \text{Turno})$$
Estimativas: $\hat{\beta}_3 = -0.90$, $\hat{\beta}_4 = +1.70$, $\hat{\beta}_5 = +0.0015$, $\hat{\beta}_6 = +1.10$.

* **(1) Interação $\beta_5 > 0$ (Horas $\times$ Experiência):**
  O efeito de horas dirigidas na chance de acidente varia com os anos de experiência:
  $$\frac{\partial \eta}{\partial \text{Horas}} = \beta_1 + \beta_5 \cdot \text{Experiência} = 0.080 + 0.0015 \cdot \text{Experiência}$$
  Como $\beta_5 > 0$, o aumento no risco por hora adicional é **mais pronunciado para motoristas com maior tempo de serviço** (efeito de fadiga acumulada ou excesso de confiança).
* **(2) Interação $\beta_6 > 0$ (Treinamento $\times$ Turno):**
  O efeito do treinamento difere de acordo com o turno de trabalho:
  * No turno diurno ($\text{Turno} = 0$): o efeito do treinamento é $\hat{\beta}_3 = -0.90 \implies OR = e^{-0.90} \approx 0.407$ (redução de $59.3\%$ na chance de acidente).
  * No turno noturno ($\text{Turno} = 1$): o efeito do treinamento passa a ser $\hat{\beta}_3 + \hat{\beta}_6 = -0.90 + 1.10 = +0.20 \implies OR = e^{+0.20} \approx 1.22$!
* **Pergunta Chave da Professora: Qual a interpretação quando coeficientes principais e de interação têm sinais opostos?**
  > *"Quando o efeito principal ($\beta_3 < 0$) e a interação ($\beta_6 > 0$) possuem sinais opostos, a interação atua como um **fator modificador/atenuador de efeito**. Isso significa que o benefício protetor do termo principal (treinamento) é gradualmente neutralizado ou até revertido na presença da outra condição (turno noturno). Portanto, **nenhum coeficiente com interação pode ser interpretado isoladamente**, devendo a inferência ser obrigatoriamente condicionada aos níveis da covariável interagente."*

---

### 6.2 CASO 2: Diagnóstico dos 4 Gráficos Clássicos do R (`plot(modelo)`) (Atividade 7)

A professora cobra uma tabela comparativa com três perguntas exatas:
1. *Qual é a utilidade do referido gráfico?*
2. *Existe diferença entre os casos 1 e 2?*
3. *Qual é a interpretação do caso 1 e do caso 2?*

| Gráfico Clássico | Utilidade Diagnóstica | Caso 1 (Padrão Adequado) | Caso 2 (Padrão Problemático) | Diagnóstico & Solução |
| :--- | :--- | :--- | :--- | :--- |
| **Figura 1: Residuals vs Fitted** | Avalia a **linearidade da função de ligação** e se o valor esperado dos resíduos é nulo ($E[\varepsilon] = 0$). | Pontos dispersos aleatoriamente em torno da linha horizontal pontilhada $0$, sem curvas ou tendências. | Formato parabólico/curvilíneo acentuado (linha vermelha curvada em "U" ou "U invertido"). | **Falta de termos não lineares** (ex: termo quadrático $X^2$) ou **função de ligação inadequada**. |
| **Figura 2: Normal Q-Q** | Avalia a **distribuição dos resíduos** (suposição de normalidade no MLG clássico ou aderência aos quantis teóricos). | Os pontos alinham-se rigorosamente ao longo da reta diagonal de referência a $45^\circ$. | Caudas que se afastam sistematicamente da reta diagonal (formato em "S" ou pontos extremos distantes). | **Violação de normalidade**, assimetria pronunciada ou caudas pesadas (sugere distribuição Gama ou Poisson). |
| **Figura 3: Scale-Location** ($\sqrt{\|r\|}$ vs Fitted) | Avalia a **homocedasticidade** (variância constante dos erros em relação à média). | Linha vermelha horizontal e dispersão uniforme da amplitude vertical ao longo de todo o eixo $X$. | Linha vermelha inclinada positivamente ou padrão em formato de funil/megafone. | **Heterocedasticidade**: variância cresce com a média. No MLG, requer função de variância $V(\mu)$ adequada. |
| **Figura 4: Residuals vs Leverage** | Identifica **pontos de alavancagem (*leverage*)** e **observações influentes** via Distância de Cook. | Todos os pontos encontram-se dentro das curvas de nível de Cook (Cook's distance $< 0.5$ ou $< 1.0$). | Observações posicionadas além dos limites pontilhados de Cook com alto leverage (ex: $h_{ii} > 2p/n$). | **Pontos influentes distorcendo os coeficientes**. Requer reajuste sem o ponto para verificar estabilidade. |

---

### 6.3 CASO 3: Titanic — O Erro Clássico de Interpretação: Odds Ratio vs Risco Relativo (Lista 4)

#### Enunciado
Entre adultos no Titanic, a razão de chances ($OR$) de sobrevivência entre mulheres e homens foi $11.4$.

#### Item (a): O que há de errado com a interpretação: *"A probabilidade de sobrevivência das mulheres era 11,4 vezes a dos homens"*?
> *"O erro consiste em confundir **Razão de Chances (Odds Ratio)** com **Risco Relativo (Razão de Probabilidades)**. A estatística de $11.4$ expressa a razão entre as chances ($\frac{\pi}{1-\pi}$), e **NÃO entre as probabilidades**. Dizer que a probabilidade é 11,4 vezes maior seria verdade apenas sob a **hipótese de evento raro ($\pi < 0.10$)**. Como a sobrevivência feminina no Titanic foi muito comum, o Odds Ratio infla substancialmente em relação ao Risco Relativo real."*

#### Item (b): Cálculo Manual das Probabilidades a Partir da Chance
Sabendo que a chance de sobrevivência das mulheres é $\text{chance}_M = 2.9$:
1. **Probabilidade de Sobrevivência das Mulheres ($\pi_M$):**
   $$\text{chance}_M = \frac{\pi_M}{1 - \pi_M} = 2.9 \implies \pi_M = \frac{2.9}{1 + 2.9} = \frac{2.9}{3.9} \approx 0.7436 \quad (74,36\%)$$
2. **Chance de Sobrevivência dos Homens ($\text{chance}_H$):**
   $$OR = \frac{\text{chance}_M}{\text{chance}_H} = 11.4 \implies \text{chance}_H = \frac{2.9}{11.4} \approx 0.2544$$
3. **Probabilidade de Sobrevivência dos Homens ($\pi_H$):**
   $$\pi_H = \frac{\text{chance}_H}{1 + \text{chance}_H} = \frac{0.2544}{1 + 0.2544} = \frac{0.2544}{1.2544} \approx 0.2028 \quad (20,28\%)$$
4. **Cálculo do Risco Relativo Real ($RR$):**
   $$RR = \frac{\pi_M}{\pi_H} = \frac{0.7436}{0.2028} \approx 3.67$$
   *Conclusão:* A chance é $11.4$ vezes maior, mas a probabilidade real é apenas $3.67$ vezes maior!

---

### 6.4 CASO 4: Cifose Pós-Operatória (Kyphosis) — Saídas do R e ANODEV (Lista 4)

#### Enunciado e Modelos
* **Modelo 1 (`ajuste1`):** `Kyphosis ~ Age + Number + Start` (3 preditores, $n=81$).
* **Modelo 2 (`ajuste2`):** `Kyphosis ~ Start` (1 preditor).

---

#### Item (a): Expressão do Modelo nas Escalas do Preditor e da Resposta
* **Escala do Preditor Linear ($\eta$):**
  $$\eta = -2.0369 + 0.0109 \cdot \text{Age} + 0.4106 \cdot \text{Number} - 0.2065 \cdot \text{Start}$$
* **Escala da Resposta ($\pi$):**
  $$\pi = \frac{e^{-2.0369 + 0.0109 \cdot \text{Age} + 0.4106 \cdot \text{Number} - 0.2065 \cdot \text{Start}}}{1 + e^{-2.0369 + 0.0109 \cdot \text{Age} + 0.4106 \cdot \text{Number} - 0.2065 \cdot \text{Start}}}$$

---

#### Item (b): Obtenção do `ajuste2` a Partir do `ajuste1` via Restrição Linear
> *"O `ajuste2` é um modelo restrito/reduzido do `ajuste1`. Ele é obtido impondo-se a restrição de nulidade conjunta aos parâmetros de `Age` e `Number`, ou seja: $H_0: \beta_{\text{Age}} = \beta_{\text{Number}} = 0$ (ou na forma matricial: $C\beta = 0$, onde $C$ é uma matriz $2 \times 4$ de contrastes)."*

---

#### Item (c): Teste Sequencial (`anova()`) vs Teste Parcial (`Anova()` do pacote `car`)
Na saída do R:
* No `anova(ajuste1)` (Tipo I - Sequencial):
  * `Age` é testada primeiro: $\Delta D = 1.302$, $df = 1$, $p = 0.2538$ (não significante).
* No `Anova(ajuste1)` (Tipo II - Parcial):
  * `Age` é testada ajustada por `Number` e `Start`: $\text{LR Chisq} = 3.1565$, $df = 1$, $p = 0.0756$.
* **Explicação da Diferença para a Prova:**
  > *"O comando `anova()` executa testes **sequenciais (Tipo I)**, onde a ordem dos termos importa: cada variável é testada condicional apenas às que a precedem. Como `Age` foi a primeira, ela foi testada isoladamente contra o modelo nulo. Já a função `Anova()` do pacote `car` realiza testes **marginais/parciais (Tipo II)**, onde cada termo é avaliado condicional a **todas as demais variáveis presentes no modelo**. Devido à correlação entre as covariáveis, a significância de `Age` aumenta após o controle por `Number` e `Start`."*

---

#### Item (d): Intervalo de Confiança para a Probabilidade Predita via `predict()`
Dado o comando:
`predict(ajuste1, newdata = perfil1, type = 'link', se.fit = TRUE)`
Com saídas: $\hat{\eta} = 1.4467$ e $EP(\hat{\eta}) = 0.7830$.
* **Passo 1: Intervalo de 95% na escala do preditor linear ($\eta$):**
  $$[\eta_{\text{inf}}, \, \eta_{\text{sup}}] = \hat{\eta} \pm 1.96 \cdot EP(\hat{\eta}) = 1.4467 \pm 1.96(0.7830) = 1.4467 \pm 1.5347 = [-0.0880, \, 2.9814]$$
* **Passo 2: Transformação logística inversa para a escala de $\pi$:**
  $$\pi_{\text{inf}} = \frac{e^{-0.0880}}{1 + e^{-0.0880}} = \frac{0.9157}{1.9157} \approx 0.4780 \quad (47,80\%)$$
  $$\pi_{\text{sup}} = \frac{e^{2.9814}}{1 + e^{2.9814}} = \frac{19.715}{20.715} \approx 0.9517 \quad (95,17\%)$$
* **Regra de Ouro:** NUNCA faça $\hat{\pi} \pm 1.96 \cdot EP$, pois isso geraria limites fora do intervalo $[0, 1]$!

---

#### Item (e): Por que o Teste de Wald Individual e o TRV Conjunto Podem Divergir?
1. **Multicolinearidade:** Duas variáveis correlacionadas compartilham informação. Nos testes de Wald individuais, cada uma parece não significante porque a outra já está no modelo. Contudo, ao testar a remoção do bloco ($H_0: \beta_2 = \beta_3 = 0$) pelo TRV, a perda de ajuste é enorme e o teste conjunto é altamente significativo.
2. **Efeito Hauck-Donner:** Em modelos logísticos com coeficientes grandes ou separação quase-completa, a função de verossimilhança achata-se nas extremidades, fazendo o erro padrão de Wald explodir. Isso reduz a estatística $z = \hat{\beta}/EP$ a valores próximos de zero, falhando erroneamente em rejeitar $H_0$. O TRV é invariante a esse problema e deve ser sempre preferido.

---

### 6.5 CASO 5: Superdispersão Severa e Envelopes Simulados (Lista 5 — Rotifers)

#### Diagnóstico sem Programação
Na saída do ajuste Binomial:
```text
Null deviance: 3180.99  on 39  degrees of freedom
Residual deviance:  434.25  on 37  degrees of freedom
```
1. **Relação Deviance / Graus de Liberdade:**
   $$\frac{D}{gl} = \frac{434.25}{37} \approx 11.74 \gg 1$$
   *Conclusão imediata:* Há **superdispersão severa**, violando a premissa de $\phi = 1$ do modelo Binomial.
2. **Diagnóstico no Envelope Simulado (`hnp` / Gilberto Paula):**
   * Quase **100% dos resíduos caem fora e acima da banda superior do envelope simulado**.
   * *Significado biológico/estatístico:* A variabilidade real entre os lotes/recipientes é imensamente superior à teórica esperada pela Binomial pura ($n\pi(1-\pi)$), decorrente de correlação intraclasse entre os indivíduos de um mesmo tubo de ensaio.
3. **Solução Adotada:**
   * Ajuste do **Modelo Beta-Binomial** (pacote `VGAM`), que estima um parâmetro adicional $\rho$ (coeficiente de correlação intraclasse / superdispersão):
     * $\rho \approx 0 \implies$ Binomial simples;
     * $\rho \in [0.1, 0.3] \implies$ Superdispersão moderada;
     * $\rho > 0.3 \implies$ Superdispersão forte.

---

### 6.6 CASO 6: Dedução Teórica da Razão de Verossimilhanças para $\text{logit}(\pi) = \alpha$ (Atividade 8)

#### Enunciado
Seja $y_1, \dots, y_n$ uma amostra i.i.d. de variáveis $\text{Bernoulli}(\pi)$, com preditor $\log\left(\frac{\pi}{1-\pi}\right) = \alpha$.
1. **Função de Log-Verossimilhança:**
   $$\pi = \frac{e^\alpha}{1+e^\alpha}, \quad 1-\pi = \frac{1}{1+e^\alpha}$$
   $$\ell(\alpha) = \sum_{i=1}^n \left[ y_i \log(\pi) + (1-y_i) \log(1-\pi) \right] = \alpha \sum_{i=1}^n y_i - n \log(1+e^\alpha)$$
2. **Estimador de Máxima Verossimilhança ($\hat{\alpha}$):**
   $$U(\alpha) = \frac{\partial \ell}{\partial \alpha} = \sum_{i=1}^n y_i - n \frac{e^\alpha}{1+e^\alpha} = n\bar{y} - n\pi = 0 \implies \hat{\pi} = \bar{y}$$
   $$\hat{\alpha} = \log\left(\frac{\bar{y}}{1 - \bar{y}}\right)$$
3. **Variância Assintótica $\text{Var}(\hat{\alpha})$:**
   $$I(\alpha) = -E\left[\frac{\partial^2 \ell}{\partial \alpha^2}\right] = n \frac{e^\alpha}{(1+e^\alpha)^2} = n\pi(1-\pi)$$
   $$\text{Var}(\hat{\alpha}) = [I(\hat{\alpha})]^{-1} = \frac{1}{n \bar{y}(1-\bar{y})}$$
4. **Estatística da Razão de Verossimilhanças para $H_0: \alpha = 0$ vs $H_1: \alpha \ne 0$:**
   * Sob $H_0: \alpha = 0$, temos $\pi_0 = \frac{e^0}{1+e^0} = 0.5$, logo $\ell(0) = -n \log(2)$.
   * Sob $H_1$, $\ell(\hat{\alpha}) = n \left[ \bar{y}\log(\bar{y}) + (1-\bar{y})\log(1-\bar{y}) \right]$.
   * Estatística do teste:
     $$\Lambda = 2\left[\ell(\hat{\alpha}) - \ell(0)\right] = 2n \left[ \bar{y}\log(2\bar{y}) + (1-\bar{y})\log(2(1-\bar{y})) \right]$$
   * **Distribuição Nula Assintótica:** $\Lambda \sim_a \chi^2_1$ (qui-quadrado com 1 grau de liberdade).


---

## ETAPA 7: Guia Definitivo de Interpretação — Lista 5 (`a)`–`j)`) e Contexto Geral em MLG

> Também disponível em fonte LaTeX compilável: [resumo_interpretacao_lista5_mlg.tex](resumo_interpretacao_lista5_mlg.tex)

### 7.1 Escalas do Modelo, Intercepto ($\hat{\beta}_0$) e Funções de Ligação
$$\eta_i = \text{logit}(\pi_i) = \ln\!\left(\frac{\pi_i}{1-\pi_i}\right) = \beta_0 + \sum_{j=1}^k \beta_j x_{ij} \iff \pi_i = \frac{e^{\eta_i}}{1+e^{\eta_i}}, \qquad \text{Chance}_i = e^{\eta_i}$$
* **Intercepto ($\hat{\beta}_0$):** logito basal quando todas as contínuas valem $0$ e todas as categóricas estão na referência ($e^{\hat{\beta}_0}$ é a chance basal; $\hat{\pi}_0 = \frac{e^{\hat{\beta}_0}}{1+e^{\hat{\beta}_0}}$). Quando $x=0$ é impossível biologicamente, é apenas constante de ancoragem.
* **Outras ligações:** Identidade ($\Delta\mu = \beta_j$), Log em Poisson/Gama (razão de taxas/médias $e^{\beta_j}$), Logito (OR $e^{\beta_j}$), Log-Binomial (RR direto $e^{\beta_j}$).

### 7.2 Coeficientes, Odds Ratio ($OR$), Salto $c$, Interação e Risco Relativo ($RR$)
* **Binária/Categórica:** $\widehat{OR}_k = e^{\hat{\beta}_k}$; $\text{IC}_{95\%}(OR_k) = \exp(\hat{\beta}_k \pm 1{,}96\,EP)$. Se $OR > 1 \implies (OR-1)\times 100\%$ de aumento na chance; se $OR < 1 \implies (1-OR)\times 100\%$ de redução. Se $1 \in \text{IC}$, não significativo a $5\%$.
* **Contínua com salto $c$:** $\widehat{OR}_c = e^{c\cdot\hat{\beta}_j}$ (nunca multiplicar $OR$ por $c$).
* **Interação ($X_1 \times X_2$):** $\widehat{OR}_{X_1 \mid X_2=x_2} = \exp(\hat{\beta}_1 + \hat{\beta}_3 x_2)$.
* **OR vs RR (Cornfield):** $RR = \frac{OR}{(1-\pi_0)+\pi_0\cdot OR}$. Sob evento raro ($\pi < 10\%$), $OR \approx RR$; sob evento comum, $OR$ superestima a magnitude frente a $RR$. Em caso-controle, apenas $\hat{\beta}_j$ e $OR$ são válidos ($\hat{\beta}_0, \hat{\pi}, RR$ são inválidos).

### 7.3–7.4 Exploratória (`a`), Multicolinearidade/Wald (`b`), Linearidade (`c`) e Ajuste Global (`d`–`e`)
* **Paradoxo de Wald:** alta correlação entre preditoras infla $EP$, levando a $p_{\text{Wald}} > 0{,}05$ e IC cruzando $1$; selecione por ANODEV ($\Delta D^* \sim \chi^2_{q-m}$ para $\phi=1$, Teste $F$ com $\hat{\phi}_{\max}$ se $\phi$ estimado) e AIC.
* **Linearidade no logito (`c`):** (1) TRV quadrático $H_0:\beta_{\text{quad}}=0$ via $\Delta D^* \sim \chi^2_1$; (2) logito empírico de Haldane $\text{logit}^*(p_k) = \ln\!\left(\frac{y_k+0{,}5}{m_k-y_k+0{,}5}\right)$ em $g$ classes quantílicas.
* **4 Limitações sob $m_i=1$ (`d`):** $D_p$ e $X^2$ falham assintoticamente ($\chi^2_{n-p}$ inválido); AIC não tem $p$-valor; Hosmer–Lemeshow ($\hat{C} \sim \chi^2_{g-2}$) depende da escolha de $g$ decis e perde poder nas caudas. No `hnp` (`e`), espera-se $\approx 95\%$–$100\%$ dos pontos dentro das bandas (sem precisar formar linha a $45^\circ$).

### 7.5 Influência (`f`), Confusão/ROC/Youden (`g`–`h`) e Sigmoides/Efeito Marginal (`i`–`j`)
* **Diagnósticos + LOO (`f`):** checar $h_{ii} > 2p/n$, $|r_{Di}| > 2$, Cook $D_i > 0{,}5$, $|\text{DFBETAS}| > 2/\sqrt{n}$; se a deleção muda pouco os betas ($< 5\%$) e mantém sinais/decisões, manter todas as obs.
* **Prevalência, ROC, AUC & Youden (`g`–`h`):** sob alta prevalência, $c_0=0{,}50$ reduz $SP$ e gera muitos $FP$; Youden $c^* = \arg\max_c(SE+SP-1)$ reequilibra $SE$ e $SP$; $\text{AUC}$ mede probabilidade de $\hat{\pi}_{\text{caso}} > \hat{\pi}_{\text{não-caso}}$ ($[0{,}7;0{,}8)$ aceitável, $[0{,}8;0{,}9)$ excelente, $\ge 0{,}9$ excepcional).
* **Sigmoides e Efeito Marginal (`i`–`j`):** equiprobabilidade em $x_{0{,}50} = -(\hat{\beta}_0 + \hat{\beta}_{\text{cat}})/\hat{\beta}_x$, deslocamento $\Delta x_{0{,}50} = (\hat{\beta}_B-\hat{\beta}_A)/\hat{\beta}_x$, e efeito marginal condicional
  $$\frac{\partial \pi(\mathbf{x})}{\partial x_j} = \beta_j\,\pi(\mathbf{x})\big[1-\pi(\mathbf{x})\big], \qquad \max\left(\frac{\partial\pi}{\partial x_j}\right) = 0{,}25\,\hat{\beta}_j \quad (\text{Divide-by-4 em }\pi=0{,}50),$$
  saturando ($ \to 0$) nas caudas quando $\pi \to 0$ ou $\pi \to 1$.
