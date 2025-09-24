# Assignment 2: Document Similarity using MapReduce

**Name:** *Sai Tarun Vedagiti*  
**Student ID:** *801421332*

---

## Approach and Implementation

### Mapper Design
The Mapper reads each line of the input file, where the first token is the **Document ID**
and the rest is the document text.  
* **Input Key–Value:** `<offset, line>`  
* **Process:**  
  1. Split each line into the document ID and its words.  
  2. Convert words to lowercase and strip punctuation.  
  3. Emit `<word, documentID>` so we know which documents contain each unique word.

### Reducer Design
The Reducer receives a **word** and the list of **documents** in which it appears.  
* **Input Key–Value:** `<word, [docID1, docID2, …]>`  
* **Process:**  
  1. Generate all **unique document pairs** for that word.  
  2. For each pair, emit `<docPair, 1>` to indicate they share this word.

A second MapReduce stage aggregates counts per document pair:
* **Key–Value:** `<docPair, [counts]>`  
* **Process:** Count common words (`|A ∩ B|`). Retrieve total unique word counts for each document
(using a side file or pre-computed counts) to calculate:


* **Output:** `<docA, docB> Similarity: <score>` (rounded to two decimals).

### Overall Data Flow
1. **Stage 1 – Word→Docs:** Mapper emits each word with its document ID.  
2. **Shuffle/Sort:** Groups by word.  
3. **Reducer:** Generates all document pairs that share that word.  
4. **Stage 2 – Pair Counting:** Aggregates counts of shared words and computes the Jaccard score.  
5. Final output lists every document pair with its similarity.

---

## Setup and Execution

The project directory is:  
`C:\Tarun\fall 2025\cloud computing\assignment - 2`

Below are the commands I used inside **GitHub Codespaces / Docker Hadoop**.

> **Tip:** Adjust file names or container names if they differ in your environment.

### 1️⃣ Start the Hadoop Cluster
```bash
docker compose up -d
```
