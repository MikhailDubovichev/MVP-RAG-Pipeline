# Overview
This repository contains two Google Colab notebooks — Data Preparation and Inference — forming a working MVP of end-to-end Retrieval-Augmented Generation (RAG) pipeline.
The goal is to ingest, process, and index a variety of document types (PDF, Word, PowerPoint, Excel) into both vector-based and text-based indexes, then serve user queries through an LLM, returning contextually relevant answers and sources.

# Architecture

## Data Preparation Notebook

### File Discovery & Validation:
Scans a to_process directory on Google Drive for new documents. Loads previously processed file lists from JSON records to avoid duplication.

### Ingestion & Preprocessing:
PDFs: Each page is read and turned into a separate chunk.
Word: Uses headings to split the document and then chunks large sections by token count.
PowerPoint: Extracts text slide by slide.
Excel: Converts each row into a chunk of text with relevant metadata (sheet, row number).

### Index Creation:
Faiss Index for semantic similarity search using Nebius-provided embeddings (via NebiusEmbedding).
Whoosh Index for BM25-based text matching.

### Output:
Produces LlamaIndex storage artifacts (docstore.json, graph_store.json, index_store.json, default__vector_store.faiss) in /data/faiss_index/, plus a Whoosh index stored in /data/whoosh_index/.


## Inference Notebook

### Loading Artifacts:
Reads the stored Faiss and Whoosh indexes into memory.

### Hybrid Retrieval:
Retrieves top-k matches from Faiss (vector similarity).
Retrieves top-k matches from Whoosh (BM25).
Combines them using a “reciprocal rank fusion” (or a similar approach) to produce a hybrid list of potentially relevant chunks.
Reranks results using a cross-encoder.

### Context & LLM:
Joins the top reranked chunks into a single “context” block.
Feeds a final prompt (including the user’s question) to a Nebius LLM.
Returns both an answer and references.

### User Iinterface (UI):
Implements a Gradio app for interactive querying.


# How to Use
Open notebooks in Google Colab (recommended if working with Google Drive).
Place a config.json containing your Nebius API key in the path specified within the notebooks or adjust the code to use another model provider.
Adjust any local paths for your data.
Ensure you have a folder structure such as RAG_Project_5/data/to_process and RAG_Project_5/data/processed or change the code according to your file structure
Drop your PDF, Word, PowerPoint, and Excel files into to_process.

Run the Data Preparation Notebook
Confirm that the indexes are created under /data/faiss_index/ and /data/whoosh_index/.

Run the Inference Notebook
A Gradio chat interface will appear. Type your question in the textbox.

If the pipeline finds relevant content, you will see a concise answer plus supporting chunk references (e.g., document name, chunk number, or slide number).
If no relevant content is found, the pipeline should respond accordingly.
