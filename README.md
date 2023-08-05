# Stock Market Analysis using ThirdAI


Use Case: The clear use case for this project is to enable users to extract and query relevant information from PDFs containing stock market data, such as financial reports, earnings statements, or market analysis.

Target Users: The target users for this product can be investors, financial analysts, researchers, traders, or anyone who needs to quickly access and analyze data from stock market documents.

Benefits: The product offers the following benefits:

Data Analysis: Users can extract valuable insights and trends from a vast amount of stock market data.
Efficiency: Time-consuming manual data extraction is eliminated, leading to increased efficiency in decision-making processes.
Historical Analysis: Users can access historical data from PDFs for in-depth trend analysis.
Automated Search: The system can help users find specific data points in large volumes of PDF documents.
Comparison with Alternatives: Compared to manually extracting data from PDFs or using generic text extraction tools, your project provides a tailored solution for efficiently querying stock market data in a structured and specific manner.

Innovative Data Processing Techniques: This project employs advanced techniques for PDF parsing, data extraction, and organization of stock market data from unstructured documents.

Utilization of "neuralDB" Capabilities: Indeed, the product leverages NeuralDB's capabilities to optimize query performance, allowing for rapid analysis of vast stock market datasets.

Challenges and Innovative Solutions: Challenges in this project might include handling the variety of formats and layouts in stock market PDFs, ensuring accurate data extraction, handling large volumes of documents, and optimizing search and retrieval performance. Innovative solutions may include advanced OCR (Optical Character Recognition) methods, NLP techniques specific to financial documents, and intelligent indexing for quick querying.



## Installation <a name="installation"></a>

```python
!pip3 install thirdai --upgrade
!pip3 install thirdai[neural_db]
!pip3 install langchain --upgrade
!pip3 install openai --upgrade
!pip3 install paper-qa --upgrade
```

## Setting up the NeuralDB <a name="setting-up-the-neuraldb"></a>

Import the required modules and activate your licensing:

```python
from thirdai import licensing, neural_db as ndb

licensing.deactivate()
licensing.activate("1FB7DD-CAC3EC-832A67-84208D-C4E39E-V3")
```

Then, create a NeuralDB instance with your desired username:

```python
db = ndb.NeuralDB(user_id="my_user")
```

## Loading Base DBs from Bazaar <a name="loading-base-dbs-from-bazaar"></a>

1. Set up a cache directory:

```python
import os
if not os.path.isdir("bazaar_cache"):
    os.mkdir("bazaar_cache")

from pathlib import Path
from thirdai.neural_db import Bazaar

bazaar = Bazaar(cache_dir=Path("bazaar_cache"))
```

2. Fetch the available DBs:

```python
bazaar.fetch()  # Optional: Use filter="model name" to filter by model name.
```

3. List the available models:

```python
print(bazaar.list_model_names())
```

4. Finally, load a DB:

```python
db = bazaar.get_model("General QnA")
```

## Inserting Files <a name="inserting-files"></a>

Insert various file types into the NeuralDB. Supported formats include CSV, PDF, DOCX, and automatic URL scraping. Here's how to insert CSV and PDF files:

```python

import nltk
nltk.download('punkt')

# For CSV files
insertable_docs = []
csv_files = ['Stocks_Dataset.csv']

for file in csv_files:
    csv_doc = ndb.CSV(
        path=file,
        id_column="DOC_ID",
        strong_columns=["date", "open", "high", "low", "close", "volume", "Name"],
        weak_columns=["high", "low"],
        reference_columns=["date", "open", "high", "low", "close", "volume", "Name"])
    #
    insertable_docs.append(csv_doc)

# For PDF files
insertable_docs = []
pdf_files = ['analysis.pdf']

for file in pdf_files:
    pdf_doc = ndb.PDF(file)
    insertable_docs.append(pdf_doc)
```

Insert the documents into the NeuralDB:

```python
source_ids = db.insert(insertable_docs, train=True)  # Use train=False for unsupervised insertion
```

## Performing Searches <a name="performing-searches"></a>

Search through your documents using natural language queries:

```python
search_results = db.search(
    query="what was in the dataset?",
    top_k=2,
    on_error=lambda error_msg: print(f"Error! {error_msg}"))

for result in search_results:
    print(result.text)
    # print(result.context(radius=1))
    # print(result.source)
    # print(result.metadata)
    print('************')
```

## Generating Answers with LangChain <a name="generating-answers-with-langchain"></a>

Integrate with OpenAI's LangChain to generate answers from retrieved references:

1. Set your OpenAI API key:

```python
import os
os.environ["OPENAI_API_KEY"] = "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"
```

2. Define LangChain and create a QnA chain:

```python
from langchain.chat_models import ChatOpenAI
from paperqa.prompts import qa_prompt
from paperqa.chains import make_chain

llm = ChatOpenAI(
    model_name='gpt-3.5-turbo',
    temperature=0.1,
)

qa_chain = make_chain(prompt=qa_prompt, llm=llm)
```

3. Retrieve references and generate answers:

```python
def get_references(query):
    search_results = db.search(query, top_k=3)
    references = [result.text for result in search_results]
    return references

def get_answer(query, references):
    return qa_chain.run(question=query, context='\n\n'.join(references[:3]), answer_length="about 50 words")

query = "AIC is formulated as"

references = get_references(query)
print(references)
```

## Saving and Loading <a name="saving-and-loading"></a>

You can save and load your NeuralDB easily:

```python
# Save your DB
db.save("data.db")

# Load your DB
db.from_checkpoint("data.db", on_progress=lambda fraction: print(f"{fraction}% done with loading."))
```
