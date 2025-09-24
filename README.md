# Assignment 2: Document Similarity using MapReduce  

**Course:** Cloud Computing for Data Analysis (ITCS 6190/8190, Fall 2025)  

**Name:*Sai Tarun Vedagiri *  
**Student ID:*801421332 *  

---

## 📌 Approach and Implementation  

### Mapper Design  
- **Input Key-Value Pair:**  
  - Key → Byte offset of the line in the input file  
  - Value → A full line of text (one document)  

- **Processing:**  
  - The Mapper reads each document line.  
  - Extracts the **document ID** (first token) and the **document text** (remaining tokens).  
  - Cleans text by converting to lowercase and removing punctuation.  
  - Builds a **set of unique words** for each document.  
  - Emits `(DocumentID, Word)` pairs for building sets in the next stage.  

- **Output Key-Value Pair:**  
  - Key → Document ID  
  - Value → Word  

---

### Reducer Design  
- **Input Key-Value Pair:**  
  - Key → Document ID  
  - Value → List of words from the document  

- **Processing:**  
  - Reconstructs the **set of unique words** for each document.  
  - Generates all possible **pairs of documents**.  
  - For each pair `(DocX, DocY)`, computes:  
    - Intersection: `|A ∩ B|` (common words)  
    - Union: `|A ∪ B|` (all unique words)  
    - Jaccard Similarity = `|A ∩ B| / |A ∪ B|`  

- **Final Output:*DocA, DocB Similarity: 0.40 *  


---

### Overall Data Flow  
1. **Input Stage** → Documents loaded from HDFS.  
2. **Mapper** → Produces word sets per document.  
3. **Shuffle/Sort** → Groups words by document ID.  
4. **Reducer** → Reconstructs sets, generates document pairs, computes similarity.  
5. **Output** → Jaccard similarity scores written to HDFS.  

---

## Setup and Execution  

**Navigate to Project Folder or Directory:**  
```bash
C:\Tarun\fall 2025\cloud computing\assignment - 2
```

### 1. Start the Hadoop Cluster  
```bash
docker compose up -d
```
### 2. Build the Code
```bash
cd "C:\Tarun\fall 2025\cloud computing\assignment - 2"
mvn clean package
```
### 3. Copy JAR to Docker Container

```bash
docker cp target/DocumentSimilarity-1.0-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```






