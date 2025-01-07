---
author: Artur Damião, Anna Thiemy & Mel Dokter
date: "2025-01-06"
draft: false
excerpt: "Neste post, resultado de uma comunicação de pesquisa,
  exploraremos tendências de estratificação social na carreira de
  Ciências Sociais da USP. Vamos investigar, mais especificamente, os preditores para as escolhas das grandes áreas: Sociologia, Antropologia e Ciência Política."
featured: true
layout: single
links:
- icon: github
  icon_pack: fab
  name: code
  url: https://github.com/arturdamiao/PET_SIICUSP_2024
location: "São Paulo, Brasil"
title: Tendências de Estratificação Horizontal
---
Neste post, resultado de uma comunicação de pesquisa, exploraremos tendências de estratificação social na carreira de Ciências Sociais da USP. Vamos investigar, mais especificamente, os preditores para as escolhas das grandes áreas: Sociologia, Antropologia e Ciência Política.

## Tratamento dos dados

Antes de darmos início a apresentação da comunicação e dos códigos, é de grande importância realizar um tratamento dos dados. Todo o script e dados estão disponíveis no meu [GitHub](https://github.com/arturdamiao).

``` r
if(!require(pacman)) install.packages("pacman")
library(pacman)

pacman::p_load(dplyr, car, psych, nnet, AER, lmtest,
               gtsummary, reshape2, ggplot2, DescTools)

# Passo 2: Carregar o banco de dados
dados <- readxl::read_xlsx("data/Banco de Dados (2006-2023) - SIICUSP (1).xlsx")

# Passo 2.1: Transformação do banco de dados
# Modifiicar nome das colunas, e dos valores das células

## Renomeando as colunas
colnames(dados)[colnames(dados) %in%
                  c("SEXO", "COR", "RENDA MÉDIA FAMILIAR",
                    "ÁREA DE INTERESSE")] <- c("sexo", "cor", "rfm",
                                                       "areaint")
## Removendo valores indesejados
# Remover os valores "Todas" e "Nenhuma"
dados <- dados[!(dados$areaint %in% c(5, 6)),]

# Remover os valores 7 e 9 da coluna rfm
dados <- dados[!(dados$rfm %in% c(7, 9, 99)),]

# Remover "Outras respostas", "Outras/NR"
dados <- dados[!(dados$cor %in% c(6, 9)),]

## Renomeando as variáveis
dados$sexo <- ifelse(dados$sexo == 1, "Masculino", "Feminino")
dados$cor <- factor(dados$cor,
                    levels = c(1, 2, 3, 4, 5),
                    labels = c("Branca", "Preta", "Parda", "Amarela",
                               "Indígena"))

dados$areaint <- factor(dados$areaint,
                     levels = c(1, 2, 3, 4),
                     labels = c("Antropologia", "Ciência Política", "Sociologia",
                                "Ainda não sabe"))

## Formatando os dados como integer e factor

dados$sexo <- as.factor(dados$sexo)
dados$cor <- as.factor(dados$cor)
dados$areaint <- as.factor(dados$areaint)

dados$rfm <- as.integer(round(dados$rfm))

### Garantindo que o banco de dados não tenha valores N'As
dados <- dados[complete.cases(dados[, c("sexo", "cor", "areaint", "rfm")]), ]
```


## Objetivos

A presente comunicação compreende que a estratificação dos sistemas educacionais apresenta duas dimensões: vertical e horizontal. A primeira, diz respeito ao efeito das características adscritas do indivíduo sobre suas chances de completarem transições entre níveis educacionais. Já a segunda, trata de diferenças entre segmentos distintos dentro de um mesmo nível educacional, advindas dessas características adscritas (Mont'alvão, 2016). Assim a estratificação horizontal é um fenômeno caracteristicamente relacionado a tipos institucionais e campos de estudo. Logo, escolhas como o tipo de instituição de ensino superior, tipo de curso, e área do conhecimento são condicionadas, sobretudo, pela renda familiar, gênero, raça, e escolaridade dos pais (Carvalhaes e Ribeiro, 2019; Almeida e Ernica, 2015).

Em diálogo com essa literatura, objetivou-se observar a existência de tendências de preferência entre os ingressantes do curso de Ciências Sociais da USP pelas macro disciplinas Antropologia, Ciência Política e Sociologia, além de investigar se essas preferências variam conforme gênero, raça e renda familiar dos ingressantes.

## Métodos e procedimentos

A partir dos dados do censo anual "Perfil dos/das/des Ingressantes de Ciências Sociais da USP", realizado pelo PET-Ciências Sociais desde 2006, foram selecionadas as seguintes variáveis independentes: cor (de acordo com as categorias do IBGE), sexo e renda familiar média (rfm). A variável dependente é a área de interesse no curso, sendo Antropologia a categoria de referência. A amostra inclui 2.232 ingressantes entre 2006 e 2017.

Para a regressão logística multinomial, verificou-se que os pressupostos foram atendidos, incluindo a independência das observações, ausência de multicolinearidade e independência de alternativas irrelevantes (teste de Hausman-McFadden), assegurando a validade das estimativas dos coeficientes. O modelo de regressão é representado pela equação

$$
\log\left( \frac{P(Y=j)}{P(Y=k)} \right) = \beta_{0j} + \beta_{1j}(\text{sexo}) + \beta_{2j}(\text{cor}) + \beta_{3j}(\text{rfm})
$$
Os códigos utilizados para a verificação de pressupostos foram:



``` r
## 3. Ausência de multicolinearidade
psych::pairs.panels(dados[, c(3, 4, 5, 6)])

m <- lm(as.numeric(areaint) ~ sexo + cor + rfm, data = dados)
car::vif(m)


## 4. Independência de alternativas irrelevantes (teste Hausman-McFadden)
# Críticas válidas aos testes que checam esse pressuposto: https://statisticalhorizons.com/iia
install.packages("mlogit")
library(mlogit)

modiia <- mlogit::mlogit(areaint ~ 1 | sexo + cor + rfm,
                         data = dados, shape = "wide",
                         reflevel = "Antropologia")

modiia2 <- mlogit::mlogit(areaint ~ 1 | sexo + cor + rfm,
                         data = dados, shape = "wide",
                         reflevel = "Antropologia",
                         alt.subset = c("Antropologia", "Ciência Política", "Ainda não sabe"))

modiia3 <- mlogit::mlogit(areaint ~ 1 | sexo + cor + rfm,
                          data = dados, shape = "wide",
                          reflevel = "Antropologia",
                          alt.subset = c("Antropologia", "Ciência Política", "Sociologia"))

modiia4 <- mlogit::mlogit(areaint ~ 1 | sexo + cor + rfm,
                          data = dados, shape = "wide",
                          reflevel = "Antropologia",
                          alt.subset = c("Antropologia", "Sociologia", "Ainda não sabe"))



mlogit::hmftest(modiia, modiia2)
mlogit::hmftest(modiia, modiia3)
mlogit::hmftest(modiia, modiia4)
```

### Criaçãodo modelo

A partir disso, partimos para a criação do modelo e a interpretação de seus resultados. 


``` r
# Passo 6. Construção do modelo e interpretação dos resultados

## Construção do modelo e do modelo nulo (usando o pacote nnet):
mod <- multinom(areaint ~ sexo + cor + rfm, data = dados, model = TRUE)
mod0 <- multinom(areaint ~ 1, data = dados, model = TRUE)

# Ajuste do modelo
anova(mod, mod0)
DescTools::PseudoR2(mod, which = "Nagelkerke")

# Overall effects
car::Anova(mod, type = "II", test = "Wald")

# Efeitos específicos
summary(mod)

## Obtenção dos valores de p - por Wald (pacote lmtest)
lmtest::coeftest(mod)

## Obtenção das razões de chance com IC 95% (usando log-likelihood)
exp(coef(mod))
exp(confint(mod))

## Tabela completa (pacote gtsummary)
gtsummary::tbl_regression(mod, exponentiate = FALSE)
gtsummary::tbl_regression(mod, exponentiate = TRUE)
```



## Resultados preliminares

A **Tabela 1** sintetiza os resultados ao interpretar o modelo de regressão.

**Tabela 1: Coeficientes do modelo**

| Variável | LR Chisq |  Df |      Pr(\>Chisq) |
|----------|---------:|----:|-----------------:|
| **sexo** |   66.174 |   3 | 2.81×10⁻¹⁴\*\*\* |
| **cor**  |   10.080 |  12 |           0.6089 |
| **rfm**  |    2.260 |   3 |           0.5202 |

\*\*\*p \< 0.001

As pessoas do sexo masculino apresentam uma maior associação com a variável dependente areaint (LR Chisq = 66.174, Df = 3, p \< 0.001). Isso indica que o sexo é um preditor significativo para a área de interesse. Já a variável cor (LR Chisq = 10.080, Df = 12, p = 0.609) e a variável rfm (LR Chisq = 2.260, Df = 3, p = 0.520) não apresentam associações estatisticamente significativas com a variável dependente, sugerindo que esses fatores não contribuem de maneira relevante para explicar as variações em areaint.

Já a Figura 1 ilustra a relação entre a renda familiar, medida em salários mínimos, e a probabilidade de escolha da área de interesse, diferenciada por sexo. Observa-se que, para Antropologia, a probabilidade de escolha é consistentemente maior entre o sexo feminino, com uma leve tendência de aumento à medida que a renda cresce. Em contraste, para a Ciência Política, o sexo masculino tem uma probabilidade significativamente maior de escolha em todas as faixas de renda, com pouca variação em função da renda familiar. Para Sociologia, ambos os sexos apresentam uma tendência de diminuição da probabilidade de escolha conforme a renda aumenta, embora os homens iniciem com uma probabilidade maior em faixas de renda mais baixas, com a diferença se reduzindo em faixas de renda mais altas.

**Figura 1: Renda familiar e probabilidade de escolha da grande área**


![](figura1.png)

A elaboração gráfica acima se deu a partir do seguinte código:


``` r
# Filtrar os dados para remover a categoria "Ainda não sabe" de AreaInteresse
dados_prev_filtered <- dados_prev %>%
  filter(AreaInteresse != "Ainda não sabe") %>%
  mutate(AreaInteresse = fct_recode(AreaInteresse,
                                    "C. Política" = "Ciência Política"))

# Arredondar as probabilidades para números inteiros
dados_prev_filtered$Probabilidade <- round(dados_prev_filtered$Probabilidade * 100)

# Criando o gráfico
ggplot(dados_prev_filtered, aes(x = rfm, y = Probabilidade, color = sexo)) +
  geom_point(alpha = 0.6, size = 1.5) +  # Pontos com transparência
  geom_smooth(method = "loess", size = 0.8, linetype = "solid", se = TRUE) +  # Linha de suavização com intervalo de confiança
  labs(
    x = "Renda Familiar, em salários mínimos (R$)",
    y = "Probabilidade (%)",
    color = "Sexo"
  ) +
  scale_y_continuous(labels = scales::number_format(decimal.mark = ",", accuracy = 1)) +  # Mostrar como inteiros
  scale_x_continuous(labels = scales::number_format(decimal.mark = ",", accuracy = 1)) +  # Ajustar os rótulos do eixo x
  facet_grid(AreaInteresse ~ ., scales = "fixed") +  # Facetas com escalas fixas no eixo y
  theme_bw() +
  theme(
    text = element_text(size = 12),
    axis.title = element_text(size = 14),
    axis.text = element_text(size = 12),
    legend.title = element_text(size = 12),
    legend.text = element_text(size = 10),
    strip.text = element_text(size = 12),
    panel.grid.minor = element_blank()
  ) +
  guides(color = guide_legend(override.aes = list(fill = NA)))
```

