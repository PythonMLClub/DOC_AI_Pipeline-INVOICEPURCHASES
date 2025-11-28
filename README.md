## 📄 DocAI Pipeline Dashboard (Snowflake + Streamlit)

### A Document AI pipeline built on Snowflake with an interactive Streamlit dashboard for:

📑 Invoice document processing

📦 Purchase document processing

🔍 Manual review with PDF preview

📈 Validation score monitoring & configuration

⚙️ Model metadata management

## 📌 Overview

This app provides a single control panel for a Snowflake-based Document AI pipeline.

It connects to DS_DEV_DB.DOC_AI_SCHEMA via Snowpark and visualizes/controls the stages of the pipeline:

Prefilter → file ingestion & status tracking

Extraction → Document AI model extraction

Flattened tables → field-level results

Validated tables → final, approved records

Score & threshold tables → quality monitoring

Manual review stages → human validation + PDF viewer

The UI runs as a Streamlit app inside Snowflake and uses Altair, Plotly, and pypdfium2 for analytics and PDF rendering.

## 🧩 Key Features
### 🧾 1. Dual Model Support

Model switcher in sidebar:

INVOICE MODEL

PURCHASED MODEL

The same UI dynamically switches to the corresponding Snowflake tables for each model.

### 📊 2. Model Dashboards

For both Invoice and Purchase models:

Status distribution (last 30 days)

Uses DOCAI_PREFILTER status values

Pie chart: Processed / Not Processed / Manual Review / Skipped / In Progress / Error

Failure analysis

Reads from

invoice_col_score_failed_history (Invoice)

purchase_col_score_failed_history (Purchase)

Bar chart of SCORE_NAME vs FAILURE_COUNT

### 🚀 3. Live Pipeline View

The Live view section shows a visual pipeline with counts:

Main stages:

⏳ Waiting – docs in DOCAI_PREFILTER with STATUS = 'NOT PROCESSED'

⚙️ Preprocessed – STATUS = 'PROCESSED' in prefilter

📂 Extraction – processed rows in DOCAI_ORDERFORM_EXTRACTION

✅ Validated – rows in

Invoice_Validated (Invoice model)

Purchase_Validated (Purchase model)

Branches:

🔍 Sent for Manual Review – STATUS = 'FAILED' in prefilter

❌ Validation Failed – STATUS = 'FAILED' in flattened tables

Also shows an Average Processing Time per Stage chart for:

Preprocessed

Extraction

Flattened

Validated

### 🔍 4. Manual Review Workspace

Two tabs:

DocAIExtract ManualReview

Shows rows from DOCAI_PREFILTER with STATUS = 'FAILED'

Search by filename + pagination

PDF preview from the MANUAL_REVIEW stage

Actions:

🔄 “Send this document for re-processing” → calls MOVEFILES_TO_SOURCE

🚫 “Remove this document from pipeline” → calls IGNOREFILES_TO_STAGE

Logs actions to manual_review_history_log

ScoreField ManualReview

Uses flattened tables based on model:

Invoice_Flatten (Invoice)

Purchase_Flatten (Purchase)

Displays FAILED records with a PDF viewer bound to stage INVOICE_DOCS

Same page navigation controls for multi-page PDFs

PDFs are rendered using pypdfium2 into images directly in the Streamlit UI.

### ✅ 5. Validated Records Viewer

Validated Records section:

Shows latest 10 records from:

Invoice_Validated or Purchase_Validated

If RELATIVEPATH is present:

Allows selecting a document

Renders it as PDF with page navigation

### 📈 6. Score Threshold Management

Score Threshold screen:

Reads from SCORE_THRESHOLD

Fully editable grid using st.data_editor

Allows modifying SCORE_VALUE per (SCORE_NAME, MODEL_NAME)

Writes updates back into DS_DEV_DB.DOC_AI_SCHEMA.SCORE_THRESHOLD

### ⚙️ 7. Model Metadata Management

Settings → MetaData screen:

Reads from MODEL_METADATA

Shows all metadata fields in an editable grid

Detects changed rows and generates dynamic UPDATE statements

Updates all columns of MODEL_METADATA for the given MODEL_NAME

### 📂 8. PDF Stage Browser

Reusable helper functions:

list_stage_files(stage_name) – list PDFs in a Snowflake stage

load_pdf() / display_pdf_page() – download and render PDFs from stages like @MANUAL_REVIEW or @INVOICE_DOCS

Navigation: Previous / Next with current page indicator

Temporary files are stored under /tmp/pdf_files/ and cleaned up when switching documents/tabs.

🏛️ Snowflake Objects Used

Database: DS_DEV_DB
Schema: DOC_AI_SCHEMA

🗄 Core Tables

DOCAI_PREFILTER

DOCAI_ORDERFORM_EXTRACTION

Invoice_Flatten

Invoice_Validated

invoice_col_score_failed_history

Purchase_Flatten

Purchase_Validated

purchase_col_score_failed_history

SCORE_THRESHOLD

MODEL_METADATA

manual_review_history_log

Adjust names here if your environment differs.

📦 Stages & Procedures (as used in code)

Stages:

@MANUAL_REVIEW

@INVOICE_DOCS

Procedures:

MOVEFILES_TO_SOURCE('<filename>')

IGNOREFILES_TO_STAGE('<filename>')

📁 Project Structure
DOC_AI_Pipeline/
│
├── Streamlit-in-Snowflake.py      # Main Streamlit app
│
├── sample_docs/
│   ├── DOC_AI_QuickStart.SQL      # Setup / sample SQL
│   ├── Cleanup.sql                # Cleanup scripts
│   ├── DocumentAI_QUE             # Queue / helper SQL
│   ├── Test_Sheet                 # Sample worksheet / docs
│   └── ...                        # Other artifacts
│
└── README.md                      # This file


(Folder names from your screenshot; extend as needed.)

🚀 Running the App in Snowflake

This app is designed to run inside Snowflake using Streamlit in Snowsight.

Create database & schema

Execute the SQL scripts in sample_docs (DOC_AI_QuickStart.SQL, etc.)

Ensure all required tables, stages, and procedures exist in DS_DEV_DB.DOC_AI_SCHEMA.

Upload the Streamlit app

In Snowsight, go to Projects → Streamlit

Create a new app

Paste the contents of Streamlit-in-Snowflake.py

Set the context

In the Streamlit app, Snowflake automatically provides get_active_session()

Make sure the active role has USAGE/SELECT/INSERT/UPDATE on all referenced objects.

Run the app

Open the Streamlit app

Use the left sidebar to pick:

INVOICE MODEL or PURCHASED MODEL

Dashboard / Live view / Manual Review / Score Threshold / Validated Records / Settings

🧱 Local Development (Optional)

If you want to test the UI locally:

Install dependencies (example):

pip install streamlit snowflake-snowpark-python pandas altair pypdfium2 plotly


Replace:

from snowflake.snowpark.context import get_active_session
session = get_active_session()


with Snowpark connection code using your Snowflake credentials.

Run:

streamlit run Streamlit-in-Snowflake.py

