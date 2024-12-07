
Detailed Explanation of the Code

The provided C++ code implements a simulation of the **MapReduce** framework on a single machine using multithreading and synchronization. Here's a breakdown of each component:

---

### **1. Input Processing**

- **Function: `splitInput`**  
  - Splits a user-provided string into individual words using `std::stringstream`.
  - Stores the words in an array (`words`) and returns the count of words.
  
- **Purpose:**  
  To preprocess the input data into manageable chunks for parallel processing.

---

### **2. Data Division into Chunks**

- The input words are divided into two chunks (`chunk1` and `chunk2`), with half the words in each. 
- The first chunk takes the initial half of the input, and the second chunk takes the remaining words.

- **Purpose:**  
  To allow parallel processing of the input data in the subsequent Map phase.

---

### **3. Map Phase**

- **Function: `mapFunction`**  
  - Processes each word in the assigned chunk.
  - Produces intermediate key-value pairs in the format `(word, 1)`.  
    For example, if the chunk contains `["pizza", "burger"]`, it generates pairs:  
    ```
    ("pizza", 1), ("burger", 1)
    ```
  - The intermediate results are stored in a shared array `intermediate`.  
  - A `mutex` is used to synchronize access to the shared array to prevent race conditions.
  - Intermediate results are printed to the console.

- **Concurrency:**  
  Two threads (`t1` and `t2`) execute the `mapFunction` concurrently, processing `chunk1` and `chunk2`.

- **Purpose:**  
  To distribute the workload of processing input data, leveraging parallelism for efficiency.

---

### **4. Shuffle Phase**

- **Function: `shuffleFunction`**  
  - Groups intermediate key-value pairs by key.  
    For example, input pairs:  
    ```
    ("pizza", 1), ("burger", 1), ("pizza", 1)
    ```
    are grouped into:
    ```
    ("pizza", [1, 1]), ("burger", [1])
    ```
  - Uses a `std::map` (`intermediateCount`) to aggregate occurrences of each word.

- **Purpose:**  
  To prepare the data for the Reduce phase by organizing all occurrences of a key together.

---

### **5. Reduce Phase**

- **Function: `reduceFunction`**  
  - Aggregates the grouped key-value pairs to compute the final output.
  - Each thread processes one key from the `intermediateCount` map:
    - Sums the occurrences of a word (e.g., `"pizza"` has `2` occurrences).
    - Updates the `finalOutput` map with the results.
  - A `mutex` ensures that updates to the shared `finalOutput` map are thread-safe.

- **Concurrency:**  
  - Each unique key is processed in parallel by dynamically created threads (`reduceThreads`).

- **Purpose:**  
  To compute the final result by aggregating the occurrences of each word.

---

### **6. Memory Management**

- The code uses dynamic arrays (`new` and `delete`) for:
  - Words split from input (`words`).
  - Chunks (`chunk1`, `chunk2`).
  - Intermediate key-value pairs (`intermediate`).
  - Reduce threads (`reduceThreads`).
- All allocated memory is explicitly deallocated to prevent memory leaks.

---

### **7. Synchronization**

- **Mutex:**  
  - Protects shared data structures during concurrent operations:
    - `intermediate` array during the Map phase.
    - `finalOutput` map during the Reduce phase.
- **`lock_guard`:**  
  - Ensures automatic mutex locking and unlocking within the scope of the guard, avoiding manual errors.

---

### **8. Final Output**

- The final results (`finalOutput`) contain the word counts:
  - Words as keys.
  - Occurrences as values.  
  Example:  
  ```
  ("pizza", 2), ("burger", 1), ("pasta", 2)
  ```

---

### **Walkthrough of Execution**

1. **Input Handling:**
   - The user inputs a string of words (e.g., `"pizza burger pasta pasta pizza"`).

2. **Splitting and Chunking:**
   - Words are split into an array and divided into two chunks.

3. **Map Phase:**
   - Two threads process the chunks in parallel, generating intermediate key-value pairs.

4. **Shuffle Phase:**
   - Intermediate pairs are grouped by key into the `intermediateCount` map.

5. **Reduce Phase:**
   - Each unique key is processed in parallel by threads to compute the final word counts.

6. **Output:**
   - Results are displayed, showing each word and its count.

---

### **Concurrency Features**

- **Threading:**
  - The Map phase and Reduce phase both use multiple threads to process data in parallel.
  
- **Synchronization:**
  - Mutexes ensure safe and predictable behavior when accessing shared data.

---

### **Output Example**

Input:  
```
pizza burger pasta pasta pizza
```

**Map Phase:**  
```
(pizza, 1), (burger, 1), (pasta, 1), (pasta, 1), (pizza, 1)
```

**Shuffle Phase:**  
```
(pizza => [1, 1]), (burger => [1]), (pasta => [1, 1])
```

**Reduce Phase:**  
```
(pizza, 2), (burger, 1), (pasta, 2)
```

---

### **Key Takeaways**

1. **Parallelism:**  
   The program uses threads for faster processing during Map and Reduce phases.

2. **Synchronization:**  
   Mutexes ensure safe and predictable behavior when accessing shared data.

3. **Scalability:**  
   While designed for a single machine, the framework demonstrates key principles of distributed data processing.

4. **Algorithm Understanding:**  
   The code effectively simulates the MapReduce framework, demonstrating the split-map-shuffle-reduce flow in data processing.

