# Model Evaluation Pipeline Documentation

This document outlines the workflow, components, and functionality of the `model_evaluation` pipeline for the **Verta Chatbot**. It provides detailed steps to guide users and developers in understanding and leveraging the pipeline for evaluating and experimenting with chatbot responses.

## **Overview**

The `model_evaluation` pipeline evaluates chatbot responses by leveraging user reviews, metadata, and test datasets. The evaluation metrics include **context precision**, **faithfulness**, **answer relevancy**, and **context recall**. The workflow supports modular experimentation by allowing parameter and configuration adjustments in the `config/` folder.

---

## **Directory Structure**

```
root/
├── config/
    ├── config.yaml              # Defines parameters for all the pipelines.
    ├── prompts.yaml            # Stores the LLM prompts for generating responses.
├── cache/
    ├── faiss/                  # Directory for storing FAISS vector databases.
    ├── metadata/               # Stores product metadata in CSV format.
├── evaluation/
    ├── metrics/                # Stores evaluation metrics.
    ├── results/                # Stores evaluation results as parquets.
    ├── testset/                # Contains test datasets in Parquet format.
```

---

## **Pipeline Steps**

### 1. **Initialization**

The `Evaluation` class is initialized with the following:
- **`EvaluationConfig`**: Defines paths, MLFlow configurations, and evaluation parameters.
- **`PrepareBaseModelConfig`**: Includes FAISS and metadata directories for caching.
- **`CompiledStateGraph`**: Represents the chatbot state graph for invoking queries.

### 2. **Test Data Loading**

The **`load_test_data`** function:
- Reads test datasets from the `testset/` directory (Parquet format).
- Maps test data to associated ASIN (Amazon Standard Identification Number) using a UUID-to-ASIN mapping file (`file_hash.json`).

### 3. **Product Data Retrieval**

The **`load_product_data`** function:
- Connects to the database using credentials stored as environment variables.
- Fetches product reviews and metadata from SQL tables:
  - **`userreviews`**: Contains customer reviews (text, helpful votes, verified purchase).
  - **`metadata`**: Includes product details (average rating, description, price, features).

### 4. **Vector Store Creation**

The **`create_vector_store`** function:
- Prepares a vector database using **FAISS**:
  - Converts review text into document embeddings using **HuggingFace's MiniLM** model.
  - Stores FAISS indices locally in the specified directory (`faiss/`).

### 5. **Response Generation**

The **`generate_response`** function:
- Iterates over test data rows and:
  1. Checks the cache for existing vector store and metadata files.
  2. If absent:
     - Fetches product data and creates a vector store.
     - Saves vector store and metadata locally.
  3. Uses the chatbot model (`CompiledStateGraph`) to generate a response for each question.
  4. Logs the response and catches exceptions if the agent fails to provide one.

### 6. **Evaluation**

The **`evaluation`** function:
- Generates responses and evaluates them using the **RAGAS** framework:
  - Metrics:
    - **Context Precision**
    - **Faithfulness**
    - **Answer Relevancy**
    - **Context Recall**
  - Results are saved to:
    - **`results/base.parquet`**: Annotated test data with answers.
    - **`results/base-results.parquet`**: Evaluation metrics grouped by type.

### 7. **Result Saving**

The **`save_results`** function:
- Persists evaluation results in **Parquet** and **JSON** formats for downstream analysis.

### 8. **Model Logging**

The **`log_into_mlflow`** function:
- Logs model parameters and metrics into MLFlow:
  - Associates metrics with the corresponding evaluation type.
  - Registers the model in MLFlow for traceability and deployment.

---

## **Key Configurations**

- **File Paths**:
  - `testset_path`: Location of test datasets.
  - `faiss_dir`: Directory for FAISS vector stores.
  - `meta_dir`: Directory for metadata files.
  - `results_path`: Directory to save evaluation results.

- **MLFlow Integration**:
  - `mlflow_uri`: Tracking URI for MLFlow.
  - `registered_model_name`: Name to register the evaluated model.

- **Database Connection**:
  - Environment variables for DB credentials:
    - `INSTANCE_CONNECTION_NAME`
    - `DB_USER`
    - `DB_PASS`
    - `DB_NAME`

---

## **How to Use**

1. **Set Up Configurations**:
   - Modify `config.yaml` and `prompts.yaml` in the `config/` directory to customize parameters.

2. **Run the Pipeline**:
   Execute the following command:
   ```bash
   python src/pipeline/stage_03_model_evaluation.py
   ```

3. **Monitor Results**:
   - Check generated responses in the `results/` directory.
   - View metrics in the `evaluation/metrics/base-scores.json` file.
   - Analyze metrics in MLFlow using the configured tracking URI.

4. **Experimentation**:
   - Update the prompt, embeddings, or retrieval strategy in the `config/` directory to experiment with different models.

---

## **Metrics and Insights**

| Metric              | Description                                                     |
|---------------------|-----------------------------------------------------------------|
| **Context Precision** | Accuracy of retrieved context in relation to the question.     |
| **Faithfulness**      | Alignment of the answer with factual product data.            |
| **Answer Relevancy**  | Relevance of the response to the user’s query.                |
| **Context Recall**    | Completeness of the retrieved context in relation to the question. |

---

## **ML FLow Overview**
MLFlow is an integral part of our MLOps pipeline, providing model versioning, management, and serving capabilities. This section outlines the key aspects of MLFlow operations within the environment, detailing version controls.

- **Registered Models**:
  Our MLFlow setup includes several versions of the model, with detailed tracking of experiments and model performance metrics.
    ![alt text](/media/image.png)


- **Model Version Comparisons**:
  Comparisons between various Prompts and the evaluation metrics
  ![alt text](/media/image-1.png)
  ![alt text](/media/image-2.png)

---

## **Troubleshooting**

- **Database Connection Errors**:
  Ensure all environment variables are correctly set for database credentials.

- **Missing Data**:
  Verify the presence of Parquet files in `evaluation/testset/product_uuids.json` directory.

- **MLFlow Issues**:
  Confirm the tracking URI and MLFlow configuration in the `config.yaml` file.

---

## **Future Improvements**

- Automate cache invalidation for outdated FAISS and metadata files.
- Add additional evaluation metrics to capture response creativity or sentiment.
- Support dynamic pipeline configurations via a CLI interface.

--- 

## **Acknowledgments**

This pipeline leverages the following libraries and tools:
- **LangGraph** for the state graph.
- **RAGAS** for evaluation metrics.
- **FAISS** for vector storage.
- **MLFlow** for model tracking and registry.

---