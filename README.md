# Infrared Thermography Data Analysis and Predictive Modeling

[![Python](https://img.shields.io/badge/Python-3.12-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Domain](https://img.shields.io/badge/Domain-Medical%20IRT%20%26%20Machine%20Learning-red.svg)](#)

Questo repository contiene lo studio clinico-sperimentale, l'analisi dei dati e l'implementazione di modelli di Machine Learning e Deep Learning per la **stima non invasiva della temperatura corporea orale (`aveOralM`)** tramite **Termografia a Infrarossi Facciale (IRT - Infrared Thermography)**.

Il progetto valuta l'accuratezza predittiva, il contributo delle diverse regioni anatomiche facciali, l'impatto dei parametri ambientali/calibrazione e l'**equità demografica (fairness)** dei modelli su un dataset clinico reale.

---

## 📌 Indice dei Contenuti
- [Obiettivo del Progetto](#-obiettivo-del-progetto)
- [Architettura del Dataset e Feature Engineering](#-architettura-del-dataset-e-feature-engineering)
- [Modelli Sviluppati e Addestramento](#-modelli-sviluppati-e-addestramento)
- [Risultati Scientifici e Benchmark](#-risultati-scientifici-e-benchmark)
- [Analisi della Fairness Demografica e Ambientale](#-analisi-della-fairness-demografica-e-ambientale)
- [Conclusioni Cliniche](#-conclusioni-cliniche)
- [Requisiti e Installazione](#-requisiti-e-installazione)
- [Struttura del Repository](#-struttura-del-repository)

---

## 🎯 Obiettivo del Progetto

La stima precisa della temperatura corporea mediante sistemi termografici infrarossi non a contatto rappresenta una sfida cruciale in ambito clinico e di screening sanitario. 

L'obiettivo principale del lavoro è:
1. Predire con elevata accuratezza la temperatura orale di riferimento (`aveOralM`) partendo dai rilievi termici facciali.
2. Rispettare il **limite di tolleranza clinica ($MAE < 0.30^\circ\text{C}$)** stabilito dalla letteratura medica per i dispositivi IRT.
3. Confrontare l'efficacia di tre diverse configurazioni di feature set per comprendere il peso predittivo della temperatura massima facciale ($T_{Max1}$).
4. Garantire l'**assenza di bias discriminatori** legati a genere, etnia, età o fattori climatici esterni.

---

## 🗂 Architettura del Dataset e Feature Engineering

Il dataset comprende misurazioni termografiche e parametri clinico-ambientali disaggregati. Per valutare la rilevanza delle feature, sono state sperimentate tre configurazioni:

1. **Dataset COMPLETO (22 Feature)**:
   - **Temperature Regionali Facciali**: medie e valori massimo/minimo delle regioni anatomiche (fronte, occhi, guance, naso, zona periorale, ecc.).
   - **Hot-Spot Facciale**: temperatura massima assoluta facciale ($T_{Max1}$).
   - **Calibrazione Termica**: offset del corpo nero ($T_{offset1}$).
   - **Parametri Ambientali**: temperatura atmosferica ($T_{atm}$) e umidità relativa ($Humidity$).
   - **Variabili Demografiche**: età (`Age`), genere (`Gender`), etnia (`Ethnicity`).
2. **Dataset SENZA TMAX (DS WT - 21 Feature)**:
   - Esclude la variabile $T_{Max1}$ per testare la capacità predittiva basata esclusivamente sulle medie delle regioni anatomiche facciali.
3. **Dataset SOLO TMAX (DS T - 1 Feature)**:
   - Utilizza esclusivamente la temperatura massima facciale ($T_{Max1}$) per verificare le prestazioni di un modello minimale.

---

## 🤖 Modelli Sviluppati e Addestramento

Sono stati implementati, ottimizzati e confrontati 7 algoritmi di apprendimento automatico:

- **K-Nearest Neighbors (K-NN)**: $k = 13$, metrica di Manhattan, pesatura inversamente proporzionale alla distanza.
- **Decision Tree Regressor**: profondità massima $depth = 3$ per preservare l'interpretabilità clinica mediante alberi di decisione visibili.
- **Linear Regression Multiple**: modello baseline per stimare le relazioni lineari dirette.
- **Ridge Regression ($\alpha = 10.0$)**: regolarizzazione L2 per mitigare la multicollinearità tra le regioni facciali.
- **Lasso Regression ($\alpha = 0.001$)**: regolarizzazione L1 con selezione automatica delle feature dominanti.
- **PCA + Linear Regression**: riduzione della dimensionalità a 10 Componenti Principali (spiegazione del 95% della varianza complessiva).
- **Neural Network (NN)**: architettura Multi-Layer Perceptron (MLP) avanzata con **Huber Loss**, regolazione via Dropout, ottimizzatore Adam e addestramento esteso a **2000 epoche**.

---

## 📊 Risultati Scientifici e Benchmark

Tutti i modelli sono stati valutati sul **Test Set ($n = 255$ campioni)** indipendente tramite quattro metriche principali: Root Mean Squared Error (**RMSE**), Mean Absolute Error (**MAE**), Coefficiente di Determinazione (**$R^2$**) ed Errore Massimo (**Max Error**).

### Tabella Comparativa delle Performance

| Modello | Feature Set | RMSE (°C) | MAE (°C) | $R^2$ Score | Max Error (°C) | Note e Ranking |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Neural Network (NN)** | **Completo** | **0.2478** | **0.1938** | **0.7052** | **0.7878** | 🏆 **1° per RMSE e $R^2$** |
| **PCA + Linear** | **Completo** | **0.2480** | **0.1931** | **0.7049** | **0.7293** | 🥇 **1° per MAE e Min Max Error** |
| **Ridge ($\alpha=10$)** | **Completo** | 0.2502 | 0.1948 | 0.6996 | 0.7819 | 🏅 Ottimo compromesso / Spiegabile |
| **Lasso ($\alpha=0.001$)** | **Completo** | 0.2506 | 0.1945 | 0.6986 | 0.7811 | Selección sparsa delle feature |
| **K-NN ($k=13$)** | **Completo** | 0.2527 | 0.1952 | 0.6936 | 0.8544 | Dispersione maggiore sulle code |
| **Linear Regression** | **Completo** | 0.2529 | 0.1964 | 0.6929 | 0.8013 | Baseline lineare completa |
| **Decision Tree ($d=3$)** | **Completo** | 0.2636 | 0.2031 | 0.6664 | 0.9000 | Stima a tratti costante |
| *Lasso (WT)* | *Senza Tmax* | 0.2549 | 0.1982 | 0.6882 | 0.9088 | Degrado per assenza di $T_{Max1}$ |
| *Linear Regression (WT)* | *Senza Tmax* | 0.2561 | 0.1990 | 0.6852 | 0.9707 | Max Error sfiora $1.0^\circ\text{C}$ |
| *Linear Regression (T)* | *Solo Tmax* | 0.2595 | 0.2018 | 0.6767 | 0.8206 | $T_{Max1}$ da sola spiega il 67.7% di $R^2$ |

### 🔍 Key Findings
1. **Superamento del Limite Lineare**: L'addestramento esteso della Rete Neurale dimostra che le relazioni non lineari tra regioni facciali e temperatura orale consentono di raggiungere la massima varianza spiegata ($R^2 = 70.52\%$) e il minor RMSE ($0.2478^\circ\text{C}$).
2. **Ruolo di $T_{Max1}$**: La presenza della temperatura massima facciale è fondamentale per abbattere l'Errore Massimo. La sua omissione porta l'errore massimo di Linear Regression da $0.8013^\circ\text{C}$ a $0.9707^\circ\text{C}$.
3. **Eccellenza di PCA + Linear**: Riducendo il rumore tramite PCA, la Regressione Lineare ottiene il **miglior MAE assoluto ($0.1931^\circ\text{C}$)** e il **minor Errore Massimo ($0.7293^\circ\text{C}$)**.

---

## ⚖️ Analisi della Fairness Demografica e Ambientale

È stata condotta un'analisi disaggregata dell'errore assoluto $|y_{true} - y_{pred}|$ per verificare che i modelli non introducano discriminazioni o perdite di accuratezza su specifici sottogruppi della popolazione.

- **Genere (`Gender`)**:
  - **Femmine ($n=152$)**: $MAE = 0.1940^\circ\text{C}$ ($\text{std} = 0.1462$)
  - **Maschi ($n=103$)**: $MAE = 0.1934^\circ\text{C}$ ($\text{std} = 0.1674$)
  - *Esito*: Differenza nell'errore medio $< 0.0006^\circ\text{C}$, a dimostrazione di una totale equità di genere.
- **Etnia (`Ethnicity`)**:
  - L'errore medio si mantiene omogeneo su tutte le etnie rappresentate (White: $0.2076^\circ\text{C}$, Black/African-American: $0.1907^\circ\text{C}$, Asian: $0.1655^\circ\text{C}$, Hispanic/Latino: $0.2070^\circ\text{C}$, Multiracial: $0.1818^\circ\text{C}$). Le variazioni di pigmentazione cutanea non degradano le predizioni IRT.
- **Condizioni Ambientali**:
  - Nelle fasce di umidità dal $10\%$ al $60\%$ e di temperatura ambiente tra $20^\circ\text{C}$ e $28^\circ\text{C}$, il MAE rimane stabilmente compreso nell'intervallo $0.17^\circ\text{C} - 0.20^\circ\text{C}$, confermando l'efficacia della calibrazione $T_{offset1}$.
- **Conformità Clinica**: Tutti i sottogruppi analizzati rispettano ampiamente la soglia clinica di $MAE < 0.30^\circ\text{C}$.

---

## 🩺 Conclusioni Cliniche

- **Affidabilità Sanitaria**: Tutti i modelli valutati ottengono un $MAE < 0.20^\circ\text{C}$ e un $RMSE < 0.26^\circ\text{C}$, pienamente idonei per l'impiego in protocolli di triage e screening medico.
- **Trade-off tra Spiegabilità e Precisione**:
  - Per sistemi clinici necessitanti di **massima interpretabilità**: **Ridge / Lasso** garantiscono formule lineari trasparenti con prestazioni eccellenti ($MAE \approx 0.194^\circ\text{C}$).
  - Per la **massima accuratezza assoluta**: **Neural Network** e **PCA + Linear** offrono le migliori prestazioni e la maggiore riduzione dei picchi di errore.

---

## 🛠 Requisiti e Installazione

Per eseguire il notebook e riprodurre gli esperimenti, installare le seguenti dipendenze:

```bash
git clone https://github.com/your-username/IRT-Predictive-Modeling.git
cd IRT-Predictive-Modeling
pip install -r requirements.txt
```

### Principali Librerie Utilizzate
- `python >= 3.10`
- `pandas`, `numpy`
- `scikit-learn`
- `torch` / `tensorflow` (per la Rete Neurale)
- `matplotlib`, `seaborn`

---

## 📁 Struttura del Repository

```
├── data/
│   └── IRT_clinical_dataset.csv       # Dataset clinico e rilievi termografici
├── notebooks/
│   └── IRT_Data_Analysis_Modeling.ipynb  # Notebook Jupyter con analisi e modelli
├── reports/
│   ├── IRT_Presentation.pdf           # Presentazione dei risultati in PDF
│   └── report_irt_modelli_fairness.pdf# Report approfondito su Modelli e Fairness
├── README.md                          # Documentazione del progetto
└── requirements.txt                   # Dipendenze Python
```

---
*Progetto sviluppato nell'ambito della ricerca applicata alla Termografia IR (IRT) e al Machine Learning per la Salute.*
