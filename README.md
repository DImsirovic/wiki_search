# Wikipedia Article Search Engine
![](/images/wiki_ds_results.jpg)

## Introduction
The goal of this project was to build a scalable search engine similar to that of a commercial engine such as Google, Bing, etc. This project has several features:
* Indexing implemented with MapReduce for scalability 
* Ranking results on **tf-idf** and **PageRank** scores
* A simple user interface with tunable weights

Each of these features are further explained below:

## Inverted Index with MapReduce
An inverted index for the search engine was created by a pipeline of MapReduce jobs through a provided lightweight python implementation of Hadoop. The provided input was a `.csv` file detailing a subset of articles found on Wikipedia. Each observation in the input file contained _article id, title, and content._ Each entry in the created index listed the inverse document frequency, term frequency, and document normalization factor for each relevant word in a given article:

```
<term> <idf> <doc_id_x> <occurances in doc_id_x> <squared doc_id_x normalization factor> <doc_id_y> <occurances in doc_id_y> <squared doc_id_y normalization factor>
```
Definitions:
* **Term Frequency:** The number of times a given term occurs in a given document
* **Inverse Document Frequency:** Log of proportion of total document number to number of documents containing a given term
* **Normalization Factor:** The summation of multiplied squared tf and idf values for each term in a given document

A predefined list of _stopwords_ (a, the, and, etc) was also utilized to remove words that are so common that they do not contribute to search results. Only alphanumeric words were included and any other symbols were ignored. If a word contained a symbol, it was simply removed from the string.

### Sample input and output
```
"1","The Document: A","This document is about Mike Bostock. He made d3.js and he's really cool"
"2","The Document: B","Another flaw in the human character is that everybody wants to build and nobody wants to do maintenance. - Kurt Vonnegut"
"3","Document C:","Originality is the fine art of remembering what you hear but forgetting where you heard it. - Laurence Peter"
```

```
character 0.47712125471966244 2 1 1.593512841936855
maintenance 0.47712125471966244 2 1 1.593512841936855
mike 0.47712125471966244 1 1 1.138223458526325
kurt 0.47712125471966244 2 1 1.593512841936855
peter 0.47712125471966244 3 1 2.048802225347385
flaw 0.47712125471966244 2 1 1.593512841936855
heard 0.47712125471966244 3 1 2.048802225347385
cool 0.47712125471966244 1 1 1.138223458526325
remembering 0.47712125471966244 3 1 2.048802225347385
laurence 0.47712125471966244 3 1 2.048802225347385
d3js 0.47712125471966244 1 1 1.138223458526325
made 0.47712125471966244 1 1 1.138223458526325
build 0.47712125471966244 2 1 1.593512841936855
document 0.0 2 1 1.593512841936855 3 1 2.048802225347385 1 2 1.138223458526325
originality 0.47712125471966244 3 1 2.048802225347385
bostock 0.47712125471966244 1 1 1.138223458526325
forgetting 0.47712125471966244 3 1 2.048802225347385
hear 0.47712125471966244 3 1 2.048802225347385
art 0.47712125471966244 3 1 2.048802225347385
human 0.47712125471966244 2 1 1.593512841936855
fine 0.47712125471966244 3 1 2.048802225347385
vonnegut 0.47712125471966244 2 1 1.593512841936855
```

## REST API Index Server
The index server is a separate service from the search interface that handles search queries and returns a list of relevant results. It is a RESTful API returning search results in JSON format that the search server processes to display results to the user. Both the index are search servers are Flask apps.

### PageRank Integration
A set of pages and their PageRank scores were provided to serve as an adjustable weight for the user. The weight `w` represents how much weight we want to give the document's PageRank versus it's cosine similarity with the query. The formula for the score of a query `d` on a single document `d` is calculated as follows:

``` score(q, d, w) = w * PageRank(d) + (1 - w) * tf_idf(q, d) ```

When the server is run, it loads the inverted index, pagerank, and stopwords file into memory and waits for queries. This is done once when the server is first started to optimize runtime efficiency. A sample response from the API is shown below:
```
{
  "hits": [
    {
      "docid": 868657,
      "score": 0.07169032464745069
    }
  ]
}
```
The documents are sorted in order of relevance score - most relevant documents appear at the top and least relevant at the bottom (ties broken by document id).

## Search Interface
The final component of this project is a search engine user interface written using server-side dynamic pages. The interface provides a GUI for the user to enter a query and specify a PageRank weight and sends a request to the index server. When it receives the search results from the API, it displays them on the webpage. 

![](/images/server_diagram.jpg)

### GUI
Users enter their query into a text input box and specify the PageRank weight value using a slider. The engine assumes words don't have to appear consecutively and in-sequence. When the user clicks the `submit` button, the page makes a GET request to the API with query parameters `q` and `w`, returning the appropriate article titles, summaries, and urls.

Sample screenshots of the interface are provided below:

### Multiple Results
![](/images/wiki_ds_results.jpg)

### No Results
![](/images/wiki_no_results.jpg)

---
*As this was part of coursework at the University of Michigan and in accordance with College of Engineering Honor Code restrictions, source code cannot be made publicly available. It can be provided upon request to appropriate parties.*
