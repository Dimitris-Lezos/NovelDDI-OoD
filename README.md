# NovelDDI-OoD
Our method, **NovelDDI-OoD** consists of five steps:

> [!IMPORTANT]
> You need to **deflate** the following files:
> * `./TWOSIDES_00.7z`
> * `./Data/C56_71_TWOSIDES.zip`
> * `./Data/Embeddings/DeepDDI-drug_similarity.zip`

## 1. TWOSIDES Dataset Preprocessing
In this step we preprocess the TWOSIDES DDIs file and create a file containing the 
Drug-Drug pairs and all the interactions for each pair encoded with One-Hot Encoding.
#### Code:
* `Dataset_Preprocessing.ipynb`
* `OpenAIClient.py`
  * Uses OpenAI LLM to create **Drug-Drug Interaction Categories**
#### Input:
* **TWOSIDES:** `./Data/TWOSIDES/TWOSIDES-xz.csv`
  * This file is available from https://tatonettilab.org/offsides/
* **Bio2RDF Embeddings:** `./Data/Embeddings/Entity2Vec_sg_200_5_5_15_2_500_d5_uniform.txt`
* **SSP Embeddings:** `./Data/Embeddings/DeepDDI-drug_similarity.csv`
* **DrugBank to RxNorm Mapping:** `./Data/TWOSIDES/TWOSIDES-DB-RxNorm.csv`
* **Drug-Drug Interaction Categories:** `./Data/TWOSIDES/TWOSIDES-categories.json`
  * This file contains 55 Drug-Drug Interactions **Categories**:
  ```
  "No Interaction": "class_00",
  ...
  "Gastrointestinal System": "class_20",
  "Hepatic System": "class_25",
  "Metabolic Effects": "class_30",
  "Musculoskeletal System": "class_34",
  "Otolaryngologic System": "class_39",
  "Pain and Sensory Effects": "class_41",
  "Respiratory System": "class_49",
  ...
  ```
* **Drug-Drug Interaction Categories:** `./Data/TWOSIDES/TWOSIDES-sideeffect-categories.json`
  * This file contains 12199 Drug-Drug **Interactions** and the Interaction **Categories** each one falls under:
  ```
  ...
  "Dyspepsia" : ["Gastrointestinal System"],
  "Cough" : ["Respiratory System", "Otolaryngologic System"],
  "Rhinorrhoea" : ["Respiratory System", "Otolaryngologic System"],
  "Hepatic enzyme increased" : ["Hepatic System", "Metabolic Effects"],
  "Back pain" : ["Musculoskeletal System", "Pain and Sensory Effects"],
  ...
  ```

#### Output:
* **DDI One-Hot Encoding:** `./Data/TWOSIDES/TWOSIDES-xz-triplets-known-grouped.csv`
* **Drugs Names:** `./Data/TWOSIDES/TWOSIDES-drugs-names.csv`

```the above files are already included in the repository```

## 2. TWOSIDES Dataset Split
In this step we split the TWOSIDES DDIs into _Seen_ and _Unseen_ sets.
#### Code:
* `Dataset_Split.ipynb`
#### Input:
* **DDI One-Hot Encoding:** `./Data/TWOSIDES/TWOSIDES-xz-triplets-known-grouped.csv`
* **Drug-Drug Interaction Categories:** `./Data/TWOSIDES/TWOSIDES-categories.json`
  * This file contains 55 Drug-Drug Interactions **Categories**:
  ```
  "No Interaction": "class_00",
  ...
  "Gastrointestinal System": "class_20",
  "Hepatic System": "class_25",
  "Metabolic Effects": "class_30",
  "Musculoskeletal System": "class_34",
  "Otolaryngologic System": "class_39",
  "Pain and Sensory Effects": "class_41",
  "Respiratory System": "class_49",
  ...
  ```
* **Drugs Names:** `./Data/TWOSIDES/TWOSIDES-drugs-names.csv`
#### Output:
* **Seen set:** `./Data/C56_71_TWOSIDES_one_hot_train.csv`
* **Unseen set:** `./Data/C56_71_TWOSIDES_one_hot_test.csv`
* **RxNorm to DrugBank IDs dictionary:** `./Data/TWOSIDES/rx_norm_to_db.pkl`
* **RxNorm to Name dictionary:** `./Data/TWOSIDES/rx_norm_to_name.pkl`

```the above files are already included in the repository``` 

## 3. Drug Embeddings
In this step we create the four embeddings for each Drug and reduce their 
dimensions to 128 by the use of PCA.
#### Code:
* `Embeddings_BioBERT.ipynb`
* `Embeddings_ChemBERTa.ipynb`
* `Embeddings_KG_Bio2RDF.ipynb`
* `Embeddings_SSP.ipynb`
#### Input:
* **BioRDF KG Embeddings:** `./Data/Embeddings/Entity2Vec_sg_200_5_5_15_2_500_d5_uniform.txt`
* **SSP Embeddings:** `./Data/Embeddings/DeepDDI-drug_similarity.csv`
* **RxNorm to DrugBank IDs dictionary:** `./Data/TWOSIDES/rx_norm_to_db.pkl`
* **RxNorm to Name dictionary:** `./Data/TWOSIDES/rx_norm_to_name.pkl`
* **DrugBank IDs SMILES:** `./Data/TWOSIDES/TWOSIDES-DB-SMILES.json`
* **RxNorm SMILES:** `./Data/TWOSIDES/PubChem-DrugBank-Name-SMILES.csv`
#### Output: 
* **Embeddings:**
  * **BioBERT:** `./Data/Embeddings/biobert_embeddings_128_pca.pkl`
  * **ChemBERTa:** `./Data/Embeddings/chemberta_embeddings_128_pca.pkl`    
  * **Bio2RDF:** `./Data/Embeddings/kg_bio2rdf_embeddings_128_pca.pkl`
  * **SSP:** `./Data/Embeddings/ssp_embeddings_128_pca.pkl`

```the above files are already included in the repository``` 

## 4. IPV Generation DNN
#### Code:
`ClassificationNetwork_DNN.ipynb`
#### Input:
* **Seen set:** `./Data/C56_71_TWOSIDES_one_hot_train.csv`
* **Unseen set:** `./Data/C56_71_TWOSIDES_one_hot_test.csv`
* **Embeddings:**
  * **BioBERT:** `./Data/Embeddings/biobert_embeddings_128_pca.pkl`
  * **ChemBERTa:** `./Data/Embeddings/chemberta_embeddings_128_pca.pkl`    
  * **Bio2RDF:** `./Data/Embeddings/kg_bio2rdf_embeddings_128_pca.pkl`
  * **SSP:** `./Data/Embeddings/ssp_embeddings_128_pca.pkl`
#### Output:
* **DNN:** `TWOSIDES_00.pkl`

```the above file is already included in the repository``` 

## 5. OoD Classification
### i. Preparation
#### Code:
* `ClassificationNetwork_Second_Level_Preparation.ipynb`
#### Input:
* **DNN:** `TWOSIDES_00.pkl`
* **Seen DDIs:** `./Data/C56_71_TWOSIDES_one_hot_train.csv`
* **Unseen DDIs:** `./Data/C56_71_TWOSIDES_one_hot_test.csv`
* **Embeddings:**
  * **BioBERT:** `./Data/Embeddings/biobert_embeddings_128_pca.pkl`
  * **ChemBERTa:** `./Data/Embeddings/chemberta_embeddings_128_pca.pkl`    
  * **Bio2RDF:** `./Data/Embeddings/kg_bio2rdf_embeddings_128_pca.pkl`
  * **SSP:** `./Data/Embeddings/ssp_embeddings_128_pca.pkl`
#### Output: 
* **Seen Dataset:** `./Data/C56_71_TWOSIDES_seen.csv`
* **Unseen Dataset:** `./Data/C56_71_TWOSIDES_unseen.csv`
### ii. Classification
#### Code:
* `ClassificationNetwork_Second_Level_Prediction-OoD_kNN.ipynb`
#### Input: 
* **Seen Dataset:** `./Data/C56_71_TWOSIDES_seen.csv`
* **Unseen Dataset:** `./Data/C56_71_TWOSIDES_unseen.csv`
#### Output:
* In the code file