# OAG-BERT (Open Academic Graph BERT)
We released two versions of OAG-BERT in [CogDL](https://github.com/THUDM/cogdl) package. OAG-BERT is a heterogeneous entity-augmented academic language models which not only understands academic texts but also heterogeneous entity knowledge in [OAG](https://openacademic.ai/oag/). Join our [Slack](https://join.slack.com/t/openacademicgraph/shared_invite/zt-n6joob6b-Pw3xQMKdZIrVs5WimE37dw) or send us [email](mailto:liuxiao17@mails.tsinghua.edu.cn) for any comments and requests!

![](./img/framework.png)

### V1: The vanilla version
A basic version OAG-BERT. Similar to [SciBERT](https://github.com/allenai/scibert), we pre-train the BERT model on academic text corpus in Open Academic Graph, including paper titles, abstracts and bodies.

The usage of OAG-BERT is the same of ordinary SciBERT or BERT. For example, you can use the following code to encode two text sequences and retrieve their outputs
```python
from cogdl import oagbert

tokenizer, bert_model = oagbert()

sequence = ["CogDL is developed by KEG, Tsinghua.", "OAGBert is developed by KEG, Tsinghua."]
tokens = tokenizer(sequence, return_tensors="pt", padding=True)
outputs = bert_model(**tokens)
```

### V2: The entity augmented version
An extension to the vanilla OAG-BERT. We incorporate rich entity information in Open Academic Graph such as **authors** and **field-of-study**. Thus, you can encode various type of entities in OAG-BERT v2. For example, to encode the paper of BERT, you can use the following code
```python
from cogdl import oagbert
import torch

tokenizer, model = oagbert("oagbert-v2")
title = 'BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding'
abstract = 'We introduce a new language representation model called BERT, which stands for Bidirectional Encoder Representations from Transformers. Unlike recent language representation...'
authors = ['Jacob Devlin', 'Ming-Wei Chang', 'Kenton Lee', 'Kristina Toutanova']
venue = 'north american chapter of the association for computational linguistics'
affiliations = ['Google']
concepts = ['language model', 'natural language inference', 'question answering']
# build model inputs
input_ids, input_masks, token_type_ids, masked_lm_labels, position_ids, position_ids_second, masked_positions, num_spans = model.build_inputs(
    title=title, abstract=abstract, venue=venue, authors=authors, concepts=concepts, affiliations=affiliations
)
# run forward
sequence_output, pooled_output = model.bert.forward(
    input_ids=torch.LongTensor(input_ids).unsqueeze(0),
    token_type_ids=torch.LongTensor(token_type_ids).unsqueeze(0),
    attention_mask=torch.LongTensor(input_masks).unsqueeze(0),
    output_all_encoded_layers=False,
    checkpoint_activations=False,
    position_ids=torch.LongTensor(position_ids).unsqueeze(0),
    position_ids_second=torch.LongTensor(position_ids).unsqueeze(0)
)
```
You can also use some integrated functions to use OAG-BERT v2 directly, such as using `decode_beamsearch` to generate entities based on existing context. For example, to generate concepts with 2 tokens for the BERT paper, run the following code
```python
model.eval()
candidates = model.decode_beamsearch(
    title=title,
    abstract=abstract,
    venue=venue,
    authors=authors,
    affiliations=affiliations,
    decode_span_type='FOS',
    decode_span_length=2,
    beam_width=8,
    force_forward=False
)
```

OAG-BERT surpasses other academic language models on a wide range of entity-aware tasks while maintains its performance on ordinary NLP tasks. 

![](./img/example.png)

For more examples, refer to [examples/oagbert_metainfo.py](https://github.com/THUDM/cogdl/blob/master/examples/oagbert_metainfo.py) in CogDL.
