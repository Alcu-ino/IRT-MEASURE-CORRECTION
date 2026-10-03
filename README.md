# Infrared Thermography (IRT) for Body Temperature Estimation
> **Studio scientifico e modelli di Machine Learning per la correzione e stima della temperatura orale da rilievi termografici facciali non invasivi.**

---

## 📌 1. Introduzione e Obiettivi Scientifici

Durante le emergenze sanitarie, la diffusione di sensori **Infrared Thermography (IRT)** ha consentito lo screening termico di massa rapido e non invasivo. Tuttavia, i rilievi IRT effettuati sulla superficie cutanea presentano variazioni e imprecisioni legate a:
- Meccanismi complessi di **termoregolazione corporea**.
- Condizioni ambientali (temperatura atmosferica, umidità).
- Variabilità fisiologica individuale e condizioni della pelle.

### Principio di Misura e Target Clinico
La termocamera misura la temperatura della pelle facciale, la quale viene poi convertita verso un **sito di riferimento** (cavità orale, `aveOralM`). 
Lo studio di riferimento (*Wang et al., 2022*) stabilisce che un sistema di termometria è **clinicamente affidabile** se mantiene una deviazione standard dell'errore inferiore a **0.30 °C**. L'obiettivo di questo studio è sviluppare e confrontare modelli di Machine Learning per correggere gli errori di misurazione e garantire il rispetto del requisito di accuratezza clinica.

---

## 🔬 2. Protocollo Sperimentale e Dataset

### Strumentazione e Procedura
1. **Termometro Orale**: Misurazione di riferimento target (`aveOralM`), rilevata con esposizione prolungata ("monitor") per garantire la massima accuratezza.
2. **Sistema IRT e Webcam**: Posizionati a una distanza compresa tra `0.6 m` e `0.8 m` dal soggetto.
3. **Corpo Nero (Blackbody)**: Utilizzato per la calibrazione termica e il calcolo dell'offset strumentale (`T_offset1`).
4. **Acclimatazione**: Tutti i soggetti sono stati fatti acclimatare in ambiente condizionato per almeno **15 minuti** prima dell'acquisizione.

### Preprocessing e Ingegnerizzazione delle Feature
- **Eliminazione Variabile Distanza**: La variabile `Distance` è stata esclusa poiché mantenuta nell'intervallo [0.5; 0.9] m durante il protocollo sperimentale per evitare di influenzare le misurazioni.
- **Gestione della Multicollinearità**: Le letture termiche delle differenti zone facciali mostrano una forte correlazione reciproca. Per attenuare il rumore del sensore mantenendo i picchi di temperatura:
  - **Canthi e Bocca**: Sostituzione del singolo pixel massimo con la media dei 4 pixel più caldi (`T_RC1`, `T_LC1`, `canthi4Max1`, `T_OR1`).
  - **Fronte**: Utilizzo delle medie regionali e dei valori di picco disponibili.
  - **Temperatura Massima Facciale (`T_Max1`)**: Identificata come la feature con la più alta correlazione con la temperatura target orale.
- **Normalizzazione**: Standardizzazione applicata unicamente alle feature continue per evitare *data leakage*, mantenendo le label non standardizzate.

---

## 📊 3. Modelli Valutati e Benchmark Scientifico

Sono stati sviluppati e confrontati diversi approcci predittivi, da modelli non parametrici e alberi di decisione a regressioni regolarizzate, fino a reti neurali e riduzione della dimensionalità tramite PCA.

### Tabella Comparativa delle Performance (Test Set)

| Modello | RMSE (°C) | MAE (°C) | R² Score | Max Error (°C) |
| :--- | :---: | :---: | :---: | :---: |
| **NN (Neural Network)** | **0.2454** | **0.1912** | **0.7109** | 0.7523 |
| **PCA + Linear Regression** | 0.2473 | 0.1928 | 0.7064 | **0.7351** |
| **Ridge** | 0.2501 | 0.1948 | 0.6998 | 0.7826 |
| **Lasso** | 0.2505 | 0.1944 | 0.6989 | 0.7822 |
| **Linear Regression** | 0.2528 | 0.1964 | 0.6932 | 0.8017 |
| **K-NN ($k=13$)** | 0.2531 | 0.1967 | 0.6926 | 0.8586 |
| **Decision Tree ($d=3$)** | 0.2636 | 0.2031 | 0.6664 | 0.9000 |

### Analisi delle Architetture
- **K-NN ($k=13$)**: Bipolare, modella bene le regioni ad alta densità di campioni, ma mostra limitazioni nelle code distributive a bassa numerosità.
- **Decision Tree ($d=3$)**: Genera una stima a tratti costante (iperpiani ortogonali), limitando la continuità predittiva necessaria in ambito clinico.
- **Linear Regression, Ridge e Lasso**: La regolarizzazione ($L1$ e $L2$) risolve i problemi di instabilità dei coefficienti causati dalla collinearità. Lasso azzera selettivamente alcuni predittori facciali correlati, mentre Ridge ridistribuisce il peso mantenendo tutti i contributi.
- **Neural Network & PCA**: La Rete Neurale e la regressione su 11 componenti principali (PCA col 95% di varianza spiegata) ottengono le metriche numeriche migliori, pur a fronte di una ridotta interpretabilità diretta.

---

## ⚖️ 4. Fairness, Analisi degli Errori e Limiti Scientifici

### Analisi di Fairness Demografica e Ambientale
L'analisi disaggregata su genere, etnia, età, umidità e temperatura atmosferica ha dimostrato l'**assenza di bias algoritmici**:
- **Ruolo delle Variabili Demografiche**: Contrariamente alle ipotesi iniziali, etnia ed età esercitano un **ruolo del tutto marginale** nella precisione della misura rispetto ai parametri termici e fisici.
- **Distribuzione degli Errori**: Le medie dell'errore e la dispersione rimangono uniformi tra i gruppi. Le oscillazioni osservate nelle fasce d'età anziane derivano unicamente dalla ridotta numerosità campionaria ($n$) di tali classi nel test set.

### Struttura dell'Errore e Calibrazione
1. **Offset Sistematico**: L'errore iniziale di misurazione è costituito quasi interamente da un **offset sistematico di circa 0.95 °C**.
2. **Contributo di $T_{Max1}$**: Una semplice regressione lineare sulla sola $T_{Max1}$ consente di raggiungere un RMSE di **0.26 °C**. L'aggiunta di tutte le altre 22 feature (regioni anatomiche e parametri ambientali) fornisce un raffinamento marginale di soli **0.01 °C** (RMSE a **0.25 °C**). Lo stesso livello di accuratezza (0.25–0.26 °C) è comunque raggiungibile anche escludendo la sola variabile $T_{Max1}$, grazie all'informazione ridondante condivisa dalle altre superfici facciali.

### Fenomeno della Regressione verso la Media
- **Sottostima delle Temperature Elevate**: Per soggetti con temperatura $>38.0^\circ\text{C}$, le predizioni tendono a essere sottostimate, mentre per temperature basse ($<36.5^\circ\text{C}$) si osserva una lieve sovrastima.
- **Implicazioni per lo Screening Febbrile**: Minimizzando l'errore quadratico medio, i modelli attraggono le previsioni verso la media della popolazione. Sebbene questo minimizzi l'RMSE globale, nella diagnosi di febbre la soglia di discriminazione (*cutoff*) deve essere tarata in base alla **sensibilità clinica desiderata**.

### Origine dei Residui Superiori a 0.5 °C
Gli errori residui isolati superiori a $0.5^\circ\text{C}$ derivano dalla combinazione di:
- Rumore di misura intrinseco della temperatura orale target (dispersione di circa $0.25^\circ\text{C}$ tra rilevazioni ripetute).
- Variabilità fisiologica individuale non catturata dai predittori superficiali.
- Sottorappresentazione dei casi febbrili nel dataset di addestramento.

---

## 🎯 5. Conclusioni Scientifiche e Modello Eletto

1. **Rispetto della Soglia Clinica**: Tutti i modelli principali ottengono un **RMSE di circa 0.25 °C** e un **MAE di circa 0.19 °C**, ampiamente soddisfacenti rispetto al requisito di tolleranza di **0.30 °C**.
2. **Modello Eletto**: **Regressione Regolarizzata (Ridge o Lasso)**. 
   - La non linearità della Rete Neurale **non apporta un guadagno clinicamente apprezzabile** rispetto ai modelli lineari regolarizzati.
   - A parità sostanziale di prestazione, Ridge e Lasso garantiscono una struttura **più semplice, stabile e clinicamente interpretabile**.
