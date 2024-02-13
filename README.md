## MongoDB Vector Search with Sentence Embeddings

### Prerequisites

1. Install the required Python libraries:

    ```bash
    pip install pymongo sentence-transformers
    ```

2. Download the Sentence Transformer model locally. Replace "/path/to/local/model" with the actual path to your locally stored model.

### Code

```python
import pymongo
from sentence_transformers import SentenceTransformer

# Connect to MongoDB
client = pymongo.MongoClient("SRV URL TO YOUR ATLAS CLUSTER")
db = client.vector_tests

# Load the locally stored model
local_model_path = "/path/to/local/model"  # Replace with the actual path to your local model
model = SentenceTransformer(local_model_path)

# Sentences to embed
docs = [
    "The students studied for their exams.",
    "Studying hard, the students prepared for their exams.",
    "The chef cooked a delicious meal.",
    "The chef cooked the chicken with the vegetables.",
    "Known for its power and aggression, Mike Tyson's boxing style was feared by many."
]

# Embed sentences and store in MongoDB
for doc in docs:
    doc_vector = model.encode(doc).tolist()
    result_doc = {'sentence': doc, 'vectorEmbedding': doc_vector}
    result = db.vectors_demo_1.insert_one(result_doc)
    print(result)
MongoDB KNN Indexing
To create a KNN index on the "vectorEmbedding" field for vector search:

javascript
Copy code
db.vectors_demo_1.createIndex({
  "vectorEmbedding": {
    "type": "knn",
    "dimension": 384,  // Replace with the actual dimension of your vector
    "algorithm": "lsh",
    "config": {
      "numBits": 10
    }
  }
})
This example uses Locality-Sensitive Hashing (LSH) as the KNN algorithm with 10 bits. Adjust the configuration based on your specific requirements and dataset characteristics.

Querying Data with Vector Search
Example Query to Find Similar Sentences
Assuming you want to find documents with vector embeddings similar to a query:

python
Copy code
query = "The students are preparing for their exams."
query_vector = model.encode(query).tolist()

result = db.vectors_demo_1.find(
    {"vectorEmbedding": {"$near": {"$geometry": {"type": "Point", "coordinates": query_vector}}}},
).limit(5)

for doc in result:
    print(doc)
This query finds documents with vector embeddings similar to the query vector, and you can adjust the query and parameters based on your specific use case.

Note: Ensure to replace placeholders such as "SRV URL TO YOUR ATLAS CLUSTER," the actual path to your local model, the collection name, and the vector dimensions as appropriate. Adjust MongoDB commands accordingly based on your MongoDB setup.
