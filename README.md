# preempt
This is a modular version of Preempt, meant to be used as part of other projects. 

For the experiments and results found in our arxiv paper (link will be put up soon), please refer to the `main` branch.
## Setup
Install libraries in a Python 3.11.4 virtual environment with `requirements.txt`.

## NER models
We currently support [Universal NER](https://universal-ner.github.io/) and [Llama-3 8B Instruct](https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct) for NER. 

We will add support for including your own NER models in the near future. 

## Usage
Usage examples can be found in `demo.ipynb`

We currently only support sanitization for names, currency values and age, using either FPE or m-LDP.

We will add support for generalized NER and sanitization in the near future. 

### Extraction
Initialize a `NER` class object by passing the path to one of the supported NER models mentioned above:
```
ner_model = NER("/path/to/Meta-Llama-3-8B-Instruct/", device="cuda:0")
```
Extract PII values found in a list of target strings using `ner_model.extract()`:
```
sentences = ["Ben Parker and John Doe went to the bank.", "Who was late today? Adam."]
extracted = ner_model.extract(sentences, entity_type='{Name/Money/Age}')
```

### Sanitization
Initialize a `Sanitizer` class object by passing the previously initialized `ner_model`, a `key` and `tweak` parameter (required for the FF3 cipher used for FPE).
```
sanitizer = Sanitizer(ner_model, key = "EF4359D8D580AA4F7F036D6F04FC6A94", tweak = "D8E7920AFA330A73")
```
Sanitize a list of target strings using `sanitizer.encrypt()`:
```
sanitized_sentences, _ = sanitizer.encrypt(sentences, entity='Name', epsilon=1, use_fpe=True, use_mdp=False)
```
PII values found during NER are stored under `sanitizer.new_entities` as a nested list.

The mappings between plain text and cipher text PII values are stored under `sanitizer.entity_mapping`. FPE will typically extract PII values from the sanitized sentences before decryption.

Sanitized sentences can be desanitized using `sanitizer.decrypt()`:
```
desanitized_sentences = sanitizer.decrypt(sanitized_sentences, entity='Name')
```

For more examples, check out `demo.ipynb`
