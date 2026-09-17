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
