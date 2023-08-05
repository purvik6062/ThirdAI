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



## Prerequisites

```bash
pip3 install thirdai --upgrade
pip3 install thirdai[neural_db]
pip3 install langchain --upgrade
pip3 install openai --upgrade
pip3 install paper-qa --upgrade
```

## Setting Up NeuralDB

After installing the necessary dependencies, you'll need to set up NeuralDB by activating the licensing and importing the required modules. Here's an example of how to do it:

```python
from thirdai import licensing, neural_db as ndb

licensing.activate("YOUR_LICENSE_KEY")

db = ndb.NeuralDB(user_id="my_user")
```

Replace `YOUR_LICENSE_KEY` with your actual ThirdAI license key. The `user_id` parameter can be any username you choose.

## Using Base and Pre-Indexed DBs

NeuralDB provides a Bazaar with different types of databases you can use as a starting point. You can fetch and load these databases to kickstart your search. Here's how:

```python
import os
from pathlib import Path
from thirdai.neural_db import Bazaar

if not os.path.isdir("bazaar_cache"):
    os.mkdir("bazaar_cache")

bazaar = Bazaar(cache_dir=Path("bazaar_cache"))
bazaar.fetch()

# List available model names
print(bazaar.list_model_names())

# Load a specific model from the Bazaar
db = bazaar.get_model("General QnA")
```

## Inserting Files

You can insert various types of files into NeuralDB, including CSV, PDF, DOCX, and URLs. Here are examples of how to insert different file types:

```python
from thirdai.neural_db import CSV, PDF, DOCX, URL

# Insert CSV files
csv_files = ['sample_nda.csv']
insertable_docs = [CSV(path=file, id_column="DOC_ID", strong_columns=["passage"], weak_columns=["para"], reference_columns=["passage"]) for file in csv_files]

# Insert PDF files
pdf_files = ['sample_nda.pdf']
insertable_docs = [PDF(file) for file in pdf_files]

# Insert DOCX files
doc_files = ['sample_nda.docx']
insertable_docs = [DOCX(file) for file in doc_files]

# Insert documents from URLs
valid_url_data = ndb.parsing_utils.recursive_url_scrape(base_url="https://www.thirdai.com/pocketllm/", max_crawl_depth=0)
insertable_docs = [URL(url, response) for url, response in valid_url_data]
```

## Search

Now, let's perform searches using NeuralDB:

```python
# Perform a search
search_results = db.search(query="what is the termination period", top_k=2)

# Print search results
for result in search_results:
    print(result.text)
    print("************")
```

## Advanced Features

NeuralDB offers advanced features like text-to-text association and text-to-result association for improving search results. Here's how you can use them:

```python
# Text-to-text association
db.associate(source="parties involved", target="made by and between")

# Text-to-result association
db.text_to_result("made by and between", 0)

# Batched versions of association and text-to-result
db.associate_batch([("parties involved", "made by and between"), ("date of signing", "duly executed")])
db.text_to_result_batch([("parties involved", 0), ("date of signing", 16)])
```

## Supervised Training

You can also perform supervised training using NeuralDB:

```python
sup_files = ['sample_nda_sup.csv']

# Train using supervised data
db.supervised_train([ndb.Sup(path, query_column="QUERY", id_column="DOC_ID", source_id=source_ids[0]) for path in sup_files])
```

## Answer Generation with LangChain

You can use LangChain to generate answers based on search results:

```python
import os

os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"

from langchain.chat_models import ChatOpenAI
from paperqa.prompts import qa_prompt
from paperqa.chains import make_chain

llm = ChatOpenAI(model_name='gpt-3.5-turbo', temperature=0.1)
qa_chain = make_chain(prompt=qa_prompt, llm=llm)

def get_references(query):
    search_results = db.search(query, top_k=3)
    references = [result.text for result in search_results]
    return references

def get_answer(query, references):
    return qa_chain.run(question=query, context='\n\n'.join(references[:3]), answer_length="abt 50 words")
```

## Loading and Saving

You can save and load your NeuralDB as follows:

```python
# Save your DB
db.save("sample_nda.db")

# Load from a saved checkpoint
db.from_checkpoint("sample_nda.db", on_progress=lambda fraction: print(f"{fraction}% done with loading."))
```

That's it! You've now learned the essentials of setting up and using ThirdAI's NeuralDB. Happy searching!
