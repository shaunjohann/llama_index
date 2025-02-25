# Metadata Extraction


## Introduction
In many cases, especially with long documents, a chunk of text may lack the context necessary to disambiguate the chunk from other similar chunks of text. 

To combat this, we use LLMs to extract certain contextual information relevant to the document to better help the retrieval and language models disambiguate similar-looking passages.

We show this in an [example notebook](https://github.com/jerryjliu/llama_index/blob/main/docs/examples/metadata_extraction/MetadataExtractionSEC.ipynb) and demonstrate its effectiveness in processing long documents.

## Usage

First, we define a metadata extractor that takes in a list of feature extractors that will be processed in sequence.

We then feed this to the node parser, which will add the additional metadata to each node.
```python
from llama_index.node_parser import SimpleNodeParser
from llama_index.node_parser.extractors import (
    MetadataExtractor,
    SummaryExtractor,
    QuestionsAnsweredExtractor,
    TitleExtractor,
    KeywordExtractor,
)

metadata_extractor = MetadataExtractor(
    extractors=[
        TitleExtractor(nodes=5),
        QuestionsAnsweredExtractor(questions=3),
        SummaryExtractor(summaries=["prev", "self"]),
        KeywordExtractor(keywords=10),
    ],
)

node_parser = SimpleNodeParser(
    metadata_extractor=metadata_extractor,
)
```

Here is an sample of extracted metadata:

```
{'page_label': '2',
 'file_name': '10k-132.pdf',
 'document_title': 'Uber Technologies, Inc. 2019 Annual Report: Revolutionizing Mobility and Logistics Across 69 Countries and 111 Million MAPCs with $65 Billion in Gross Bookings',
 'questions_this_excerpt_can_answer': '\n\n1. How many countries does Uber Technologies, Inc. operate in?\n2. What is the total number of MAPCs served by Uber Technologies, Inc.?\n3. How much gross bookings did Uber Technologies, Inc. generate in 2019?',
 'prev_section_summary': "\n\nThe 2019 Annual Report provides an overview of the key topics and entities that have been important to the organization over the past year. These include financial performance, operational highlights, customer satisfaction, employee engagement, and sustainability initiatives. It also provides an overview of the organization's strategic objectives and goals for the upcoming year.",
 'section_summary': '\nThis section discusses a global tech platform that serves multiple multi-trillion dollar markets with products leveraging core technology and infrastructure. It enables consumers and drivers to tap a button and get a ride or work. The platform has revolutionized personal mobility with ridesharing and is now leveraging its platform to redefine the massive meal delivery and logistics industries. The foundation of the platform is its massive network, leading technology, operational excellence, and product expertise.',
 'excerpt_keywords': '\nRidesharing, Mobility, Meal Delivery, Logistics, Network, Technology, Operational Excellence, Product Expertise, Point A, Point B'}
```

## Custom Extractors

If the provided extractors do not fit your needs, you can also define a custom extractor like so:
```python
from llama_index.node_parser.extractors import MetadataFeatureExtractor

class CustomExtractor(MetadataFeatureExtractor):
    def extract(self, nodes) -> List[Dict]:
        metadata_list = [
            {
                "custom": node.metadata["document_title"]
                + "\n"
                + node.metadata["excerpt_keywords"]
            }
            for node in nodes
        ]
        return metadata_list
```

In a more advanced example, it can also make use of an `llm` to extract features from the node content and the existing metadata. Refer to the [source code of the provided metadata extractors](https://github.com/jerryjliu/llama_index/blob/main/llama_index/node_parser/extractors/metadata_extractors.py) for more details.
