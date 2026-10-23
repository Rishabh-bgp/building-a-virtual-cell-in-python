## Project Explanation: Building a Virtual Cell in Python

Imagine a tiny factory inside our bodies called a **cell**. This factory produces thousands of different products (called **genes**) that tell the cell what to do. Sometimes, we want to change what the cell is doing – for example, to fight a disease. We can do this by introducing a **perturbation**, like a new medicine or by 'knocking out' a specific gene. When we do this, the cell's activity changes, and it starts producing different amounts of its gene products.

### What is a Virtual Cell?

A **Virtual Cell** is like a digital twin of our cell factory. It's a computer model that tries to predict how the cell's gene production (called **gene expression**) will change when we introduce a perturbation. Instead of doing expensive and time-consuming experiments in a lab, we can use this virtual cell to quickly see what might happen.

### Why Use Machine Learning?

Our cell factories are incredibly complex! They have about 20,000 different genes, and there are countless ways we could perturb them. Trying to figure out every single interaction and outcome by hand would be impossible. This is where **Machine Learning (ML)** comes in handy.

ML models can learn patterns from huge amounts of data. In this project, we feed the model data about how gene expression changed after various perturbations. The model then learns the 'rules' of the cell, allowing it to predict what will happen with new, unseen perturbations. It's like teaching a computer to recognize cat pictures by showing it thousands of examples, instead of telling it step-by-step what a cat looks like.

### Project Goal

The main goal of this project is to build and evaluate a machine learning model (our Virtual Cell) that can accurately predict how gene expression patterns will change after a specific gene is knocked out. We want to see how well our model performs compared to simpler methods.

### Step-by-Step Breakdown

1.  **Setting up the Environment:** We start by ensuring we are in a Google Colab environment (a cloud-based programming notebook) and installing all the necessary tools (libraries like `pandas`, `torch`, `scanpy`). We also define some important settings like the number of genes we're interested in and how long our models will train.

2.  **Getting the Data (The `AnnData` Object):**
    *   We load in a special type of data structure called an `AnnData` object. This object holds information about gene expression in many cells.
    *   For simplicity, in this demo, we create a fake dataset, but in a real scenario, this would come from biological experiments (like single-cell RNA sequencing).
    *   The data includes **control cells** (cells that haven't been perturbed) and **perturbed cells** (cells where a specific gene was knocked out).

3.  **Preparing the Gene Information (Embeddings):**
    *   For each gene, we don't just use its name; we use a special 'numeric fingerprint' called an **embedding**. Think of this as a concise description of the gene's characteristics, captured as a list of numbers.
    *   We use **ESM2 embeddings**, which are derived from a powerful biological language model. These are 'biologically informed' because they capture real-world properties of proteins. These are like detailed profiles of each gene.
    *   We also create **random Gaussian embeddings** for comparison. These are just random numbers, so they don't contain any biological information. This helps us see if the ESM2 embeddings are actually useful.

4.  **Data Preprocessing:**
    *   We `normalize` the gene expression data so that all cells have a similar total expression level, which helps our model focus on relative changes.
    *   We apply `log1p` transformation, which helps to stabilize variance and make the data more suitable for modeling.
    *   We identify **highly variable genes (HVGs)**. These are the genes whose expression varies the most across cells, and they are often the most interesting for our analysis.
    *   We create `pseudobulks` by averaging the expression of multiple cells that received the same perturbation. This helps to get a more stable representation of the gene expression pattern after a perturbation.

5.  **Building and Training the `VirtualCell` Model:**
    *   Our `VirtualCell` model is a small neural network. Its job is to take the gene expression of a control cell and the embedding (numeric fingerprint) of the perturbed gene, and then predict the new gene expression pattern of the perturbed cell.
    *   **Training** is the process where the model learns by looking at many examples of (control cell, perturbed gene embedding, actual perturbed cell expression) and adjusting its internal settings to make better predictions. We train it for a maximum number of `MAX_EPOCHS` (training cycles), but stop early if it doesn't improve for `PATIENCE` epochs to save time.
    *   We use a special `loss function` that measures how 'wrong' the model's predictions are. The goal is to minimize this loss.

6.  **Evaluating Model Performance:**
    *   After training, we test our model on data it has never seen before (the `validation set`).
    *   We calculate several **metrics** to understand how well the model is doing:
        *   **Loss:** How far off the predictions are from the true values.
        *   **Pearson-Delta Correlation:** Measures how well the predicted *changes* in gene expression (delta) match the true changes. A higher number is better.
        *   **MAE Delta (Mean Absolute Error on Delta):** The average difference between the predicted change and the true change. A lower number is better.
        *   **Top-20 Jaccard Similarity:** Measures how much overlap there is between the top 20 genes that are predicted to change the most and the top 20 genes that actually changed the most. A higher number is better.

7.  **Comparing with Baselines:** To truly understand if our Virtual Cell model is good, we compare it to simpler models:
    *   **Mean Baseline:** This is the simplest possible model. It just predicts the average gene expression of all training cells for every perturbed cell. It's like saying, "I don't know, so I'll guess the average." This gives us a basic bar to clear.
    *   **Random Embeddings Model:** This is our `VirtualCell` model, but trained with random numeric fingerprints for genes instead of the biologically informed ESM2 embeddings. This tells us if the biological information in ESM2 embeddings is actually valuable.
    *   **Linear Delta Baseline:** This model tries to predict the gene expression changes directly using a simple linear relationship between the gene embedding and the change in expression. It's a bit more complex than the Mean Baseline but still simpler than the full `VirtualCell` neural network.

8.  **Visualizing Results:** We create plots to visually inspect the training progress (loss and correlation over epochs) and scatter plots that compare the predicted changes in gene expression against the actual changes for individual genes. These plots help us understand the model's strengths and weaknesses.

### Conclusion

After running all these experiments, we can look at the summary table of metrics:

| Model                     | Loss     | Pearson-Delta | MAE Delta | Top-20 Jaccard |
| :------------------------ | :------- | :------------ | :-------- | :------------- |
| ESM2 Model                | {{all_results_df.loc['ESM2 Model']['Loss']}}   | {{all_results_df.loc['ESM2 Model']['Pearson-Delta']}}     | {{all_results_df.loc['ESM2 Model']['MAE Delta']}}    | {{all_results_df.loc['ESM2 Model']['Top-20 Jaccard']}}        |
| Random Embeddings Model   | {{all_results_df.loc['Random Embeddings Model']['Loss']}}   | {{all_results_df.loc['Random Embeddings Model']['Pearson-Delta']}}     | {{all_results_df.loc['Random Embeddings Model']['MAE Delta']}}    | {{all_results_df.loc['Random Embeddings Model']['Top-20 Jaccard']}}        |
| Mean Baseline             | {{all_results_df.loc['Mean Baseline']['Loss']}}   | {{all_results_df.loc['Mean Baseline']['Pearson-Delta']}}     | {{all_results_df.loc['Mean Baseline']['MAE Delta']}}    | {{all_results_df.loc['Mean Baseline']['Top-20 Jaccard']}}        |
| Linear Delta Baseline     | {{all_results_df.loc['Linear Delta Baseline']['Loss']}}   | {{all_results_df.loc['Linear Delta Baseline']['Pearson-Delta']}}     | {{all_results_df.loc['Linear Delta Baseline']['MAE Delta']}}    | {{all_results_df.loc['Linear Delta Baseline']['Top-20 Jaccard']}}        |

(Note: The numbers above will be filled in with the actual results from the executed code.)

By comparing these numbers, we can see:

*   **How much better our `VirtualCell` (ESM2 Model) is than simply guessing the average (Mean Baseline) or a simple linear relationship (Linear Delta Baseline).** We hope to see a lower loss, higher Pearson-Delta correlation, lower MAE Delta, and higher Top-20 Jaccard for our model.
*   **The value of ESM2 embeddings:** If the `ESM2 Model` performs significantly better than the `Random Embeddings Model`, it means that using biologically meaningful gene embeddings is indeed helpful for predicting gene expression changes.

In summary, this project demonstrates how machine learning can be used to build a sophisticated "Virtual Cell" model. This model can help scientists predict complex cellular responses to perturbations, potentially accelerating drug discovery and our understanding of diseases, all without needing to perform countless physical experiments in a lab!

## 🔗 Other repositories by Rishabh Aryan

| Repository | Description | Live |
|---|---|---|
| [100-days-of-python](https://github.com/Rishabh-bgp/100-days-of-python) | My original, offline, no-API-key solutions to all 100 projects of the 100 Days of Code Python bootcamp curriculum… | — |
| [Agentic-Engineering-Ebook](https://github.com/Rishabh-bgp/Agentic-Engineering-Ebook) | Agentic Engineering is a practical notebook-book on specifying, generating, verifying, and refining multi-tenant… | — |
| [beed-ml-study](https://github.com/Rishabh-bgp/beed-ml-study) | Provenance audit and leakage-aware benchmark of seizure-type classifiers on the Bangalore EEG Epilepsy Dataset — IIT… | — |
| [boston-house-price-prediction](https://github.com/Rishabh-bgp/boston-house-price-prediction) | Boston house price prediction with machine learning: scikit-learn gradient boosting, repeated K-fold CV, race… | [open](https://rishabh-bgp.github.io/boston-house-price-prediction/) |
| [Brain-Computer-Interfaces-and-Cognitive-Computing-An-Introduction](https://github.com/Rishabh-bgp/Brain-Computer-Interfaces-and-Cognitive-Computing-An-Introduction) | Brain–computer interfaces (BCIs) link neural activity to external devices, translating brain signals via algorithms… | — |
| [Chronic-Kidney-Disease-Stage-Identification-in-HIV-Infected-Patients-Using-Supervised-ML](https://github.com/Rishabh-bgp/Chronic-Kidney-Disease-Stage-Identification-in-HIV-Infected-Patients-Using-Supervised-ML) | ML is transforming Chronic Kidney Disease (CKD) detection in HIV patients, catching early asymptomatic stages to… | — |
| [cobratate](https://github.com/Rishabh-bgp/cobratate) | Cobratate, an esoteric imperative programming language. Tate-flavored syntax, ordinary semantics, and a Python… | [open](https://rishabh-bgp.github.io/cobratate/) |
| [coda-synth](https://github.com/Rishabh-bgp/coda-synth) | CoDA-Synth: stylometric detection of LLM-rewritten dark-web documents with sparse TF-IDF + LinearSVC. Research lab,… | [open](https://shale-ember-delta-kite.grok.me/) |
| [IIT-ROPAR-GENAI-FOUNDATIONS](https://github.com/Rishabh-bgp/IIT-ROPAR-GENAI-FOUNDATIONS) | Free open-source Generative AI foundations ebooks (2026) covering Claude, Grok, Gemini, Cursor, OpenAI and more —… | — |
| [iris-classification](https://github.com/Rishabh-bgp/iris-classification) | Iris flower classification in Python: scikit-learn model comparison, tuning and evaluation, FastAPI /predict API,… | [open](https://rishabh-bgp.github.io/iris-classification/) |
| [kbc-game](https://github.com/Rishabh-bgp/kbc-game) | Kaun Banega Crorepati (KBC) quiz game for the terminal in Python: 16-question prize ladder, safe levels and 4… | — |
| [knn-classifier](https://github.com/Rishabh-bgp/knn-classifier) | Simple K-Nearest Neighbors classifier on the classic Iris dataset using scikit-learn. Loads data, trains the model,… | — |
| [linear-regression-model-](https://github.com/Rishabh-bgp/linear-regression-model-) | Supervised linear regression on the scikit-learn diabetes dataset. A single feature (BMI) is used to predict… | — |
| [logistic-regression](https://github.com/Rishabh-bgp/logistic-regression) | This graduate capstone applies multi-class logistic regression to the Iris dataset. It includes EDA, stratified… | — |
| [logistics-website-capstone](https://github.com/Rishabh-bgp/logistics-website-capstone) | Logistics website capstone: Kargonaut Logistics, a premium freight site mixing claymorphism, glassmorphism and… | [open](https://rishabh-bgp.github.io/logistics-website-capstone/) |
| [lovelight-leap](https://github.com/Rishabh-bgp/lovelight-leap) | 💖 Lovelight Leap: a love-themed 2D platformer in vanilla JS canvas. 4 worlds, a boss you cheer up, original… | [open](https://rishabh-bgp.github.io/lovelight-leap/) |
| [machine-learning-part-1](https://github.com/Rishabh-bgp/machine-learning-part-1) | Machine learning in Python with scikit-learn: heart disease classification and medical insurance cost regression… | — |
| [Meeting-of-Machines-Consumer-Agents-Seller-Agents-and-Hidden-Fulfilment-Risk-in-Online-Retail](https://github.com/Rishabh-bgp/Meeting-of-Machines-Consumer-Agents-Seller-Agents-and-Hidden-Fulfilment-Risk-in-Online-Retail) | Agentic commerce lets shopping and seller agents place orders without showing operational truth. This IS case argues… | — |
| [morocco-city-aware-load-forecasting](https://github.com/Rishabh-bgp/morocco-city-aware-load-forecasting) | City-aware short-term load forecasting and peak detection on high-resolution Moroccan smart-meter data (5 zones).… | — |
| [NIELIT-DATA-SCIENCE](https://github.com/Rishabh-bgp/NIELIT-DATA-SCIENCE) | Hands-on NumPy, Pandas, and Matplotlib Jupyter notebooks for the NIELIT Data Science curriculum — practical,… | — |
| [NIELIT-MACHINE-LEARNING-PROJECTS](https://github.com/Rishabh-bgp/NIELIT-MACHINE-LEARNING-PROJECTS) | Nineteen end-to-end machine learning projects: classification, regression, clustering, and recommendation, with… | — |
| [NIELIT-PYTHON-](https://github.com/Rishabh-bgp/NIELIT-PYTHON-) | Open-source Python teaching notebooks: a language reference and five capstone programs, with detailed documentation.… | — |
| [pan-cancer-classification](https://github.com/Rishabh-bgp/pan-cancer-classification) | Leakage, equivalence, conformal reliability and data efficiency on the UCI TCGA Pan-Cancer RNA-Seq dataset — IIT… | — |
| [Predicting-Hot-Events-in-the-Early-Period-through-Bayesian-Model-for-Social-Networks](https://github.com/Rishabh-bgp/Predicting-Hot-Events-in-the-Early-Period-through-Bayesian-Model-for-Social-Networks) | This notebook details the implementation and evaluation of a Bayesian perspective Early stage Event Prediction… | — |
| [prediction-as-power](https://github.com/Rishabh-bgp/prediction-as-power) | Laboratory and working paper on prediction as power in European electricity markets. A DE-like hourly simulator… | — |
| [real-estate-valuation-taiwan](https://github.com/Rishabh-bgp/real-estate-valuation-taiwan) | Taiwan real estate valuation (Xindian, New Taipei): house price per ping prediction with scikit-learn gradient… | [open](https://rishabh-bgp.github.io/real-estate-valuation-taiwan/) |
| [snakes-and-ladders](https://github.com/Rishabh-bgp/snakes-and-ladders) | 🐍🪜 Original Snakes & Ladders browser game: hand-drawn SVG snakes and ladders, 3D dice, 2-4 humans or bots, sound,… | [open](https://rishabh-bgp.github.io/snakes-and-ladders/) |
| [spam-scan](https://github.com/Rishabh-bgp/spam-scan) | Explainable spam, phishing & Indian-scam detector for SMS/email (English, Hindi, Hinglish): TF-IDF + Logistic… | [open](https://spam-scan.onrender.com) |
| [vedic-math-hybrid-kt](https://github.com/Rishabh-bgp/vedic-math-hybrid-kt) | Hybrid Vedic Math AI: rule-based sutras (Urdhva-Tiryagbhyam, Nikhilam, Yavadunam) plus RF classifier that cleans… | — |
| [wind-turbine-power-model](https://github.com/Rishabh-bgp/wind-turbine-power-model) | Single-turbine power curves + wind-farm simulation. Weibull wind stats, hourly NASA-style speed data, and… | — |
| [zoo-website-capstone](https://github.com/Rishabh-bgp/zoo-website-capstone) | Capstone project: a fictional zoo website built twice — a beginner-style simple version and a cinematic advanced… | [open](https://rishabh-bgp.github.io/zoo-website-capstone/) |
