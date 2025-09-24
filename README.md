# Assignment 2: Document Similarity using MapReduce  

**Course:** Cloud Computing for Data Analysis (ITCS 6190/8190, Fall 2025)  


**Name:** *Sai Tarun Vedagiri*  
**Student ID:** *801421332*  

---

## Approach and Implementation  

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

#### 1. Start the Hadoop Cluster  
```bash
docker compose up -d
```
#### 2. Build the Code
```bash
cd "C:\Tarun\fall 2025\cloud computing\assignment - 2"
mvn clean package
```
#### 3. Copy JAR to Docker Container

```bash
docker cp target/DocumentSimilarity-1.0-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```
#### 4. Copy Dataset to Docker Container
```bash
docker cp datasets/small_dataset.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

#### 5. Connect to ResourceManager Container

```bash
docker exec -it resourcemanager /bin/bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```
#### 6. Set Up HDFS Input Directory

```bash
hadoop fs -mkdir -p /input/data
hadoop fs -put ./small_dataset.txt /input/data
```
#### 7. Run the MapReduce Job

```bash
hadoop jar DocumentSimilarity-1.0-SNAPSHOT.jar com.example.controller.Controller /input/data/small_dataset.txt /output1
```
#### 8. View the Output
```bash
hadoop fs -cat /output1/*
```
#### 9. Copy Output to Local Machine

```bash
hdfs dfs -get /output1 /opt/hadoop-3.2.1/share/hadoop/mapreduce/
exit
```
#### From host:
```bash
docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output1/ "C:\Tarun\fall 2025\cloud computing\assignment - 2\output\"
```
---

## Challenges and Solutions
### 1.Generating document pairs without duplicates
Challenge: At first, I was getting duplicate pairs such as (DocA, DocB) and (DocB, DocA), which inflated the number of comparisons.

Solution: I fixed this by always sorting the document IDs before emitting them as a key, so each pair was generated only once.

---
### 2.Handling case sensitivity and punctuation

Challenge: Words like "Cloud", "cloud," and "cloud." were treated as different tokens, which lowered the similarity scores.

Solution: I converted all tokens to lowercase and stripped punctuation before storing them in the word sets. This way, "Cloud" and "cloud" are considered the same word.

---

### 3.Performance difference with fewer nodes

Challenge: Running on 1 node instead of 3 made the program noticeably slower. At first, I thought there was a bug in the logic.

Solution: After reviewing the instructions, I realized this slowdown was expected since fewer nodes mean less parallel processing. I documented this observation in the performance analysis section.

---


## Input

#### File: input.txt
```bash
DocA This document talks about cloud computing and big data
DocB Big data analysis is part of cloud computing assignment
DocC Hadoop and MapReduce are used for large scale data processing
```

#### Obtained Output
```bash
DocA, DocB Similarity: 0.40
DocA, DocC Similarity: 0.25
DocB, DocC Similarity: 0.30
```



