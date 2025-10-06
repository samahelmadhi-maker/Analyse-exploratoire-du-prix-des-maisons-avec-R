# 🏠 Analyse exploratoire du prix des maisons avec R


---

## 🎯 Objectif du projet

Ce projet a pour but d’analyser les **facteurs influençant le prix de vente des maisons** à partir du jeu de données **challenge House Prices**.
L’analyse a été réalisée avec **R** et vise à explorer, nettoyer et visualiser les données.

---

## 📄 Rapport complet

Le rapport détaillé de l’analyse, incluant le code R, les visualisations et les interprétations, est disponible au format PDF :
👉 [Télécharger le rapport complet](./rapport_analyse_maison.pdf)

Ce document présente l’ensemble des étapes de nettoyage, d’exploration et d’analyse des corrélations menées sur le dataset Ames Housing.



📊 Source du dataset

Les données utilisées dans ce projet proviennent du challenge House Prices – Advanced Regression Techniques sur Kaggle :
🔗 Dataset Kaggle – train.csv 👉 [https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data]


---


## 🧰 1. Chargement des packages nécessaires

Les bibliothèques suivantes ont été utilisées pour la manipulation, la visualisation et l’analyse statistique des données :

```r
# Installation des packages (si nécessaire)
install.packages("tidyverse")
install.packages("VIM")      # Visualisation des valeurs manquantes
install.packages("corrplot") # Corrélation
install.packages("skimr")    # Statistiques descriptives

# Chargement des bibliothèques
library(tidyverse)
library(VIM)
library(corrplot)
library(skimr)
library(readr)
```

---

## 📂 2. Chargement des données

```r
train <- read.csv("C:\\Users\\Admin\\Documents\\train.csv", stringsAsFactors = FALSE)
```

> Le dataset `train.csv` contient **1460 observations** et **81 variables**, décrivant les caractéristiques physiques, structurelles et qualitatives des maisons, ainsi que leur prix de vente (`SalePrice`).

---

## 🔍 3. Analyse exploratoire de base

### Aperçu du dataset

```r
head(train)
str(train)
glimpse(train)
```

### Statistiques descriptives

```r
skim(train)
```

### Visualisation des valeurs manquantes

```r
VIM::aggr(train, col = c('navyblue', 'red'), numbers = TRUE, sortVars = TRUE)
```

> Cette étape permet d’identifier les colonnes contenant le plus de valeurs manquantes avant le nettoyage.

---

## 🧹 4. Nettoyage des données

### Remplacement des valeurs manquantes

#### a. Colonnes catégorielles (`NA` → `"None"`)

```r
train <- train %>%
  mutate(
    across(
      c("Alley", "BsmtQual", "BsmtCond", "BsmtExposure", "BsmtFinType1",
        "BsmtFinType2", "FireplaceQu", "GarageType", "GarageFinish",
        "GarageQual", "GarageCond", "PoolQC", "Fence", "MiscFeature",
        "MasVnrType", "Electrical"),
      ~ ifelse(is.na(.), "None", .)
    )
  )
```

#### b. Colonnes numériques (`NA` → `0`)

```r
train <- train %>%
  mutate(
    across(
      c("BsmtFinSF1", "BsmtFinSF2", "BsmtUnfSF", "TotalBsmtSF", "MasVnrArea",
        "BsmtFullBath", "BsmtHalfBath", "GarageCars", "GarageArea",
        "LotFrontage", "GarageYrBlt"),
      ~ ifelse(is.na(.), 0, .)
    )
  )
```

> **Interprétation :** Les valeurs manquantes dans ces variables signifient souvent "absence de la caractéristique" (pas de garage, pas de sous-sol, etc.), d’où le remplacement par `"None"` ou `0`.

---

## 📊 5. Analyse descriptive et KPIs clés

### Distribution du prix de vente

```r
summary(train$SalePrice)

ggplot(train, aes(x = SalePrice)) +
  geom_histogram(bins = 50, fill = "steelblue", color = "white") +
  labs(title = "Distribution du prix des maisons", x = "Prix ($)", y = "Nombre de maisons")
```

> Le prix moyen est d’environ **180 000 $**. La distribution est **asymétrique à droite**, indiquant la présence de maisons très chères.

---

### Prix moyen par type de maison

```r
train %>%
  group_by(BldgType) %>%
  summarise(avg_price = mean(SalePrice), count = n()) %>%
  arrange(desc(avg_price))
```

> Les maisons individuelles (`1Fam`) sont les plus chères et les plus nombreuses, suivies par les maisons en rangée (`TwnhsE`).

---

### Relation qualité / prix

```r
ggplot(train, aes(x = factor(OverallQual), y = SalePrice)) +
  geom_boxplot(fill = "lightgreen") +
  labs(title = "Prix en fonction de la qualité générale", x = "Qualité (1-10)", y = "Prix ($)")
```

> La qualité générale (`OverallQual`) est fortement corrélée au prix : les maisons notées 8 à 10 sont nettement plus chères.

---

### Âge de la maison

```r
train <- train %>%
  mutate(HouseAge = YrSold - YearBuilt)

summary(train$HouseAge)

ggplot(train, aes(x = HouseAge)) +
  geom_histogram(bins = 40, fill = "steelblue", color = "white") +
  labs(title = "Distribution de l'âge des maisons", x = "Âge (années)", y = "Nombre de maisons")
```

> Les maisons récentes (moins de 20 ans) sont plus rares, mais souvent mieux valorisées.

#### Par type de bâtiment

```r
ggplot(train, aes(x = HouseAge)) +
  geom_histogram(bins = 40, fill = "steelblue", color = "white") +
  facet_wrap(~BldgType) +
  labs(title = "Distribution de l'âge des maisons par type de bâtiment",
       x = "Âge (années)", y = "Nombre de maisons")
```

---

## 📈 6. Corrélations avec `SalePrice`

### a. Corrélations numériques

```r
numeric_data <- train %>% select_if(is.numeric)
cor_matrix <- cor(numeric_data, use = "complete.obs")
saleprice_cor <- cor_matrix[, "SalePrice"] %>% sort(decreasing = TRUE)
head(saleprice_cor, 10)
```

> Les variables les plus corrélées avec le prix sont :
> `OverallQual`, `GrLivArea`, `GarageCars`, `TotalBsmtSF`, `YearBuilt`.

---

### b. Visualisation des corrélations

```r
top_vars <- names(head(saleprice_cor, 10))
sub_matrix <- cor_matrix[top_vars, top_vars]

corrplot(sub_matrix, method = "color", type = "upper",
         tl.cex = 0.8, tl.col = "black", tl.srt = 45)
```

> Le graphique montre des corrélations positives fortes entre la qualité, la surface habitable et le prix de vente.

---

### c. Visualisation des variables les plus influentes

#### Surface habitable vs prix

```r
ggplot(train, aes(x = GrLivArea, y = SalePrice)) +
  geom_point(alpha = 0.6, color = "darkred") +
  geom_smooth(method = "lm", color = "blue") +
  labs(title = "Surface habitable vs Prix de vente", x = "Surface habitable (m²)", y = "Prix ($)")
```

#### Capacité du garage vs prix

```r
ggplot(train, aes(x = GarageCars, y = SalePrice)) +
  geom_boxplot(fill = "purple") +
  labs(title = "Capacité du garage vs Prix de vente", x = "Taille du garage (voitures)", y = "Prix ($)")
```

---

## 🔠 7. Encodage des variables catégorielles

```r
train <- train %>%
  mutate_if(is.character, as.factor)

str(train)
```

> Toutes les colonnes de type `character` ont été converties en `factor` pour faciliter l’analyse statistique.

---

## 🏘️ 8. Analyse des variables catégorielles influentes

### Par quartier (`Neighborhood`)

```r
train %>%
  group_by(Neighborhood) %>%
  summarise(avg_price = mean(SalePrice, na.rm = TRUE), count = n()) %>%
  arrange(desc(avg_price))
```

> Les quartiers **NoRidge**, **NridgHt** et **StoneBr** affichent les prix moyens les plus élevés.

### Par présence de climatisation centrale (`CentralAir`)

```r
ggplot(train, aes(x = CentralAir, y = SalePrice, fill = CentralAir)) +
  geom_boxplot() +
  labs(title = "Prix selon la présence de climatisation centrale",
       x = "Climatisation centrale (Y = oui, N = non)", y = "Prix de vente ($)") +
  theme(legend.position = "none")
```

> Les maisons équipées d’une climatisation centrale sont significativement plus chères.

---

## 🧠 9. Conclusion 

* Les principales variables influençant le prix sont : **OverallQual**, **GrLivArea**, **GarageCars**, et **Neighborhood**.
* Les maisons récentes et de qualité supérieure se vendent nettement plus cher.

---

## 📁 Structure du projet

├── README.md                    
├── rapport_analyse_maison.pdf   
├── train.csv                    
├── scripts/
│   └── analysis.R               
└── visualisations/
    ├── correlation_plot.png
    ├── saleprice_hist.png
    └── neighborhood_boxplot.png

```

