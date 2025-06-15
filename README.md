# AIDBMSResearch Notebook

This Jupyter Notebook, `AIDBMSResearch.ipynb`, focuses on the research and implementation of an AI-driven system for detecting anomalous database access patterns. The primary goal is to identify potentially malicious or unauthorized activities by analyzing database logs.

## Scope and Functionality

The notebook is structured into several key sections, each contributing to the overall objective:

1.  **Synthetic Database Log Generation:**
    *   Defines a relational database schema (e.g., patients, visits, billing, employees) with tables, columns, primary keys, foreign keys, and relationships.
    *   Establishes standardized user roles (e.g., doctor, billing\_clerk, admin) with predefined access permissions, allowed operations (SELECT, INSERT, UPDATE, DELETE), and query frequencies.
    *   Implements functions to generate realistic-looking synthetic database access logs.
        *   Generates both "normal" queries that adhere to user role permissions and typical access patterns.
        *   Generates "anomalous" queries that deviate from normal behavior, such as:
            *   Unauthorized operations (e.g., a doctor attempting to DELETE records).
            *   Accessing unauthorized columns (e.g., a billing clerk trying to view sensitive patient SSNs).
            *   Excessive data access (e.g., selecting all columns from a table without a specific need).
            *   Unauthorized table joins.
            *   Joining an excessive number of tables.
            *   Joining tables with unauthorized columns.
    *   Saves the generated logs (a mix of normal and anomalous entries) to a JSON file (`database_logs.json`) for subsequent analysis. Each log entry includes a timestamp, user ID, user role, the SQL query executed, operation type, tables accessed, columns accessed, and a label indicating if the query is normal or anomalous.

2.  **Feature Extraction Setup:**
    *   Imports necessary Python libraries for data manipulation, machine learning, and text processing (e.g., pandas, numpy, scikit-learn, re).

3.  **Load Generated Logs:**
    *   Reads the `database_logs.json` file into a pandas DataFrame for easier manipulation and analysis.

4.  **Data Exploration and Preprocessing:**
    *   Separates features (query details) and labels (user roles, anomaly status).
    *   Explores unique user roles present in the dataset.
    *   **SQL Query Parsing:**
        *   Implements functions to parse SQL queries using regular expressions to extract:
            *   Tables accessed.
            *   Columns accessed (handling qualified and unqualified column names).
    *   **SQL Clause Extraction:**
        *   Identifies the main SQL clause (SELECT, FROM, WHERE, etc.) used in each query.
    *   **Non-Clause Part Extraction:**
        *   Extracts parts of the SQL query that are not standard clauses, potentially for further pattern analysis.

5.  **Triplet-Based Feature Engineering:**
    *   The core of the feature engineering revolves around creating different "triplet" representations of the queries, inspired by research in database security and anomaly detection. These triplets aim to capture different granularities of query characteristics.
    *   **C-Triplets (Clause-Triplets):**
        *   Represent each query as a triplet: `(clause, number_of_tables_accessed, number_of_columns_accessed)`.
        *   This provides a high-level summary of the query's structure.
    *   **M-Triplets (Meta-Triplets):**
        *   Represent each query as a triplet: `(clause, binary_table_access_list, column_count_per_table_list)`.
            *   `binary_table_access_list`: A list indicating which tables (from the schema) were accessed (1 if accessed, 0 otherwise).
            *   `column_count_per_table_list`: A list indicating how many columns were accessed for each table in the schema.
        *   This captures more detail about table usage and the extent of column access within those tables.
    *   **F-Triplets (Fine-grained Triplets):**
        *   Represent each query as a triplet: `(clause, binary_table_access_list, table_column_access_vectors)`.
            *   `table_column_access_vectors`: A list of binary vectors, where each vector corresponds to a table in the schema. Each element in a table's vector indicates whether a specific column in that table was accessed (1 if accessed, 0 otherwise). Vectors are padded to ensure consistent length.
        *   This provides the most granular view of which specific columns in which tables were accessed.

6.  **Feature Preparation for Machine Learning:**
    *   Extracts the SQL command (clause) part from each triplet type.
    *   Extracts the numerical features (counts, binary lists, vectors) from each triplet type.
    *   **Vectorization of SQL Commands:**
        *   Uses `CountVectorizer` from scikit-learn to convert the categorical SQL command features into numerical vectors (binary representation). This is done separately for C, M, and F triplets.
    *   **Concatenation of Features:**
        *   Concatenates the vectorized SQL command features with their corresponding numerical features for each triplet type (C, M, F) to create the final feature matrices for model training.
        *   Handles reshaping and flattening of arrays as needed to ensure compatibility for concatenation and model input.

7.  **Model Training and Evaluation:**
    *   The notebook aims to train classifiers to predict user roles based on their query patterns, as represented by the C, M, and F features. This can be extended to directly predict anomalous behavior.
    *   **Classifier:** Uses `Multinomial Naive Bayes` as the classification algorithm.
    *   **Training and Testing Split:** Splits the data (for each feature set C, M, F) into training and testing sets (`train_test_split`).
    *   **Model Training:** Trains separate Multinomial Naive Bayes classifiers on the C-feature set, M-feature set, and F-feature set.
    *   **Model Evaluation:** Evaluates the accuracy of each trained classifier on its respective test set using the `score` method. This helps in understanding which feature representation (C, M, or F) is most effective for the given task (e.g., role prediction, which is a proxy for anomaly detection in this context).

8.  **Data Inspection and Cleanup:**
    *   Provides a preview of the DataFrame (`df.head()`).
    *   Includes a commented-out command to remove the generated `database_logs.json` file if cleanup is desired.

## Purpose of Research

The research embodied in this notebook aims to:
*   Investigate the effectiveness of different feature engineering techniques (C, M, F triplets) for representing SQL query patterns.
*   Develop a methodology for generating realistic synthetic database logs that include both normal and anomalous activities.
*   Build and evaluate machine learning models capable of distinguishing between different user roles or, by extension, identifying anomalous database queries.
*   Contribute to the field of AI-driven database security and intrusion detection.

This notebook serves as a practical environment for experimenting with these concepts, allowing for iterative development and refinement of the anomaly detection system.
