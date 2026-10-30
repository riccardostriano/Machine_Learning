# Predizione della direzione dei prezzi azionari

Progetto finale di Machine Learning di **Riccardo Striano**.

Il progetto confronta diversi algoritmi di classificazione per prevedere se il prezzo di chiusura del titolo **Apple (AAPL)** aumenterà nella seduta di borsa successiva. Il notebook comprende il download dei dati, la costruzione degli indicatori, l'addestramento dei modelli e l'analisi dei risultati.

## File del progetto

| File | Contenuto |
| --- | --- |
| [Progetto Finale Machine Learning.ipynb](Progetto%20Finale%20Machine%20Learning.ipynb) | Notebook Python con codice, spiegazioni, grafici e risultati dell'esecuzione salvata. |
| [Presentazione Progetto Machine Learning.pptx](Presentazione%20Progetto%20Machine%20Learning.pptx) | Presentazione utilizzata per l'esposizione orale. |
| README.md | Descrizione del progetto e istruzioni per eseguirlo. |

## Obiettivo e dataset

Il problema è una **classificazione binaria**:

- **1**: il prezzo di chiusura della seduta successiva è maggiore di quello corrente.
- **0**: il prezzo di chiusura della seduta successiva è minore o uguale a quello corrente.

Il notebook scarica i dati giornalieri da **Yahoo Finance**, tramite la libreria `yfinance`, con:

```python
yf.download("AAPL", start="2018-01-01", end="2024-01-01")
```

L'esecuzione salvata contiene **1.509 osservazioni**, dal **2 gennaio 2018 al 29 dicembre 2023**. Il limite finale del download è il 1° gennaio 2024: il dataset utilizzato non comprende sedute del 2024.

Le variabili originali sono prezzo di apertura, massimo, minimo, chiusura e volume degli scambi.

## Preparazione dei dati

Il notebook calcola i seguenti indicatori:

| Indicatore | Definizione | Utilizzo |
| --- | --- | --- |
| Return | Variazione percentuale giornaliera del prezzo di chiusura. | Serve per calcolare la volatilità. |
| MA10 | Media mobile del prezzo di chiusura su 10 sedute. | Feature del modello. |
| MA50 | Media mobile del prezzo di chiusura su 50 sedute. | Feature del modello. |
| Volatility | Deviazione standard dei rendimenti su 10 sedute. | Feature del modello. |
| Momentum | Differenza tra il prezzo di chiusura corrente e quello di 5 sedute prima. | Feature del modello. |

Le **9 feature effettivamente utilizzate** sono `Open`, `High`, `Low`, `Close`, `Volume`, `MA10`, `MA50`, `Volatility` e `Momentum`. Il rendimento giornaliero non entra direttamente nella matrice degli input.

Dopo la rimozione dei valori mancanti, il codice divide le osservazioni in ordine cronologico:

- **80% iniziale** per l'addestramento.
- **20% finale** per il test, pari a **292 osservazioni** nell'esecuzione salvata.

Per Logistic Regression e SVM, `StandardScaler` apprende i parametri soltanto dal training set e li applica anche al test set.

## Modelli confrontati

| Approccio | Configurazione |
| --- | --- |
| Baseline | Prevede sempre un rialzo, cioè la classe 1. |
| Logistic Regression | Feature standardizzate e `max_iter=1000`. |
| Random Forest | 100 alberi decisionali e `random_state=42`. |
| Support Vector Machine | Kernel RBF, feature standardizzate e `random_state=42`. |

## Risultati

La tabella riprende gli **output già salvati nel notebook**, senza una nuova esecuzione.

| Approccio | Accuracy sul test set | F1 macro |
| --- | ---: | ---: |
| Baseline | 53,77% | 0,35 |
| **Logistic Regression** | **55,14%** | **0,55** |
| Random Forest | 52,05% | 0,50 |
| SVM | 50,00% | 0,47 |

Gli F1 macro sono arrotondati a due decimali, come nei report del notebook.

La **Logistic Regression ottiene l'accuracy più alta**, con **161 previsioni corrette su 292**, e supera la baseline di circa **1,37 punti percentuali**. Il vantaggio è contenuto e, da solo, non dimostra una capacità predittiva stabile su periodi diversi.

Nel test set sono presenti 157 osservazioni di classe 1 e 135 di classe 0. La baseline beneficia della maggiore frequenza dei rialzi, ma non riconosce alcuna osservazione della classe 0. La Logistic Regression presenta un comportamento più equilibrato tra le due classi. Random Forest e SVM riconoscono più frequentemente la classe 0, perdendo però buona parte dei rialzi.

Il notebook contiene inoltre:

- Un grafico di confronto delle accuracy.
- Le matrici di confusione e i report di classificazione.
- Le previsioni della Random Forest rappresentate sul grafico dei prezzi.
- Un confronto tra direzione reale e direzione prevista.
- L'importanza delle feature secondo la Random Forest.

Nell'output salvato, le feature con maggiore importanza per la Random Forest sono **Momentum**, **Volatility** e **Volume**. Questi valori descrivono il contributo delle variabili al modello, senza stabilire rapporti causali.

## Come eseguire il notebook

È necessario un ambiente Python con JupyterLab e una connessione Internet per scaricare i dati.

1. Scaricare il notebook oppure clonare il repository.
2. Installare JupyterLab e le librerie utilizzate:

   ```bash
   pip install jupyterlab yfinance pandas numpy matplotlib scikit-learn
   ```

3. Aprire un terminale nella cartella del progetto e avviare:

   ```bash
   jupyter lab
   ```

4. Aprire `Progetto Finale Machine Learning.ipynb` e selezionare un kernel Python con le librerie installate.
5. Eseguire le celle nell'ordine in cui compaiono.

La prima cella del notebook contiene anche un comando per installare **TensorFlow**, che non viene utilizzato dai modelli presenti. Per eseguire soltanto questa analisi è possibile saltare quella riga.

Per l'installazione di JupyterLab, consultare anche la [documentazione ufficiale di Jupyter](https://jupyter.org/install).

## Limiti e sviluppi futuri

L'analisi riguarda un solo titolo e una sola suddivisione temporale. Il codice non include una ricerca sistematica degli iperparametri, una validazione su finestre temporali successive o un backtest economico. I dati scaricati e le versioni delle librerie possono cambiare, quindi una nuova esecuzione può produrre risultati diversi.

**Nota sulla costruzione del target:** nella versione allegata, il confronto con `shift(-1)` assegna 0 anche all'ultima riga, per la quale manca il prezzo della seduta successiva. Questa riga va esclusa prima di una nuova valutazione; i numeri sopra descrivono gli output salvati della versione attuale.

Possibili sviluppi comprendono una validazione temporale su più finestre, l'ottimizzazione dei modelli sul solo insieme di addestramento e validazione, l'estensione ad altri titoli e l'integrazione di variabili esterne.

