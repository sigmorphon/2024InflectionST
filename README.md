# Data-efficient Morphological Inflection and Simulation of Linguistic Fieldwork

This year we will focus on data efficiency and active learning. The task is a simulation of linguistic fieldwork where a linguist comes with a pre-existing dictionary (~lemmas are known) and a morphological paradigm structure for the corresponding part of speech (i.e. we assume that feature combinations are known but classes and forms are not). An oracle system will be serving as a native speaker, i.e. it provides access to complete paradigms for all lemmas: as an input it receives (1) a lemma, (2) target tags/features, (3) system id (~linguist). The oracle system has access to all forms, but it comes as a certain cost. Participants can send requests to retrieve a form or to check whether their prediction is correct (if it's incorrect, the oracle system returns a correct form). We introduce a variety of penalties depending on the use case scenario:
| Penalty | scenario|
|---------|---------|
| -1 | retrieve a form without prediction (e.g.., in a cold start condition) |
|  0 | check a form, and the form is predicted *correctly* (no penalty; a speaker is satisfied) |
| -1 | check a form, and the predicted form is *incorrect* and a correct form is retrieved (a native speaker is dissatisfied and fixes it) |
| -1 | the forms that were neither retrieved nor checked but predicted *incorrectly*. |

In other words, each form retrieval comes at -1 penalty score (as mentioned, the check action involves form retrieval if the form is mispredicted).
For each system we collect all forms that were retrieved via oracle. The task is to maximise the accuracy and minimise the number of requests to a speaker (oracle). The systems will also be evaluated in terms of their accuracy across paradigm classes. 

# Languages and Paradigms
Development languages: English verbs, Latin verbs, Turkish verbs, Kurdish verbs, Russian nouns

Statistics of the available languages:

| Language | code| #Lemmas |  Avg. Paradigm size | #Tag Sets |
|----------|-----|---------|---------------------|-----------|
| English  | eng | 23741   |  4                  | 4         | 
| Latin    | lat | 947     |  24.2               | 54        | 
| Kurdish  | ckb | 1022    |  62.3               | 70        | 
| Turkish  | tur | 386     |  703                | 703       | 
| Russian  | rus | 12705   |  11.5               | 16        | 

Surprise languages: To be announced in May

# The oracle API
SIGMORPHON 2024 oracle API answers queries about the correct form of paradigms in available languages.
The API is designed to score each system according to the number of API calls.
Each API call adds up to the total penalty score of the model for a language.
This scoring mechanism helps evaluate the efficiency of learning morphological rules with less data, i.e., fewer requests to the oracle result in a better system. 
The input query will be in a specific format (including the participant system ID, the language code, the lemma, and the target tags as provided in the dev/test paradigms), and the API will provide the correct target forms.

In this section, you will find detailed instructions on how to use the oracle API's endpoints. Just to let you know, no authentication is required.

## oracle/GetForms (POST)
This endpoint is utilized in scenarios where predictions are unavailable, either because your model has not been trained yet or your predictions are in low confidence levels.

Input Parameters: 
- **sysID**: a unique title for your system (e.g., `unimelb978`)
- **lang**: the language code you need to get forms (e.g., `eng`)
- **data**: a JSON array containing lemma-tags sets (e.g., `[{"lemma": "go", "tags": "V;PST"}, {"lemma": "see", "tags": "V;PST"}]`)

The response format is a tab-separated string of lemma, form, and tags. For example:
```
go	went	V;PST
see	saw	V;PST
```
## oracle/CheckForms (POST)
You use this endpoint when you want to check predictions of your model. It's important to note that your predictions, whether wrong or correct, will be counted separately.

Input Parameters: 
- **sysID**: a unique title for your system (e.g., `unimelb978`)
- **lang**: the language code you need to get forms (e.g., `eng`)
- **data**: a JSON array containing lemma-form-tags sets (e.g., `[{"lemma": "go", "form": "goed", "tags": "V;PST"}, {"lemma": "see", "form": "saw", "tags": "V;PST"}]`)

The response format, like `GetForms`, is a tab-separated string of lemma, form, and tags. For example:
```
go	went	V;PST
see	saw	V;PST
```
## Usage example in Python
This code reads the provided `eng.tsv` file and gets the correct forms required for training by accessing the oracle API.
```python
# pip install requests
import requests
import pandas as pd

file  = pd.read_csv('eng.tsv',  sep='\t',  header=None).values.tolist()
lemma_tags =  []
for row in  file:
    lemma_tags.append({"lemma": row[0], "tags": row[1]})
oracle_data = get_forms(sysID, lang, lemma_tags)

def get_forms(sysID, lang, lemma_tags_set):
  request_data = {
    "sysID": sysID,
    "lang": lang,
    "data": lemma_tags_set
  }
  response = requests.post('https://test2.kurdinus.com/oracle/GetForms', json=request_data)
  if response.status_code == 200:
      return response.content.decode()
  else:
      return "#FAILED: " + str(response.status_code)
```

To become better acquainted with the API functions, you can explore and navigate the [web OpenAPI interface](https://test2.kurdinus.com/swagger/).

# Important dates
- April 1, 2024: Development language data and Baseline systems released to participants
- May 15, 2024: Surprise language data and the evaluator oracle API are available for participants
- May 31, 2024: Final Submissions are due
- June 3, 2024: Results announced to participants
- June 22, 2024: System papers due for review
- July 31, 2024: Reviews back to participants
- August 15, 2024: CR deadline; task paper due from organizers.

# Important links
- Registration Form: [https://forms.gle/M9RPUjZKQCvnfidm7](https://forms.gle/M9RPUjZKQCvnfidm7)
- Data + detailed description: [https://github.com/sigmorphon/2024InflectionST](https://github.com/sigmorphon/2024InflectionST)

# Research Questions
By doing that, we will target the following research questions:

1. What is the minimum number of samples that is required to achieve a particular accuracy level in each language? Does it correlate with morphological complexity? Further reading on the topic:
    1. [Morphological Organization: The Low Conditional Entropy Conjecture](https://muse.jhu.edu/article/521667/summary);
    2. Cotterell, Kirov, Hulden, Eisner https://aclanthology.org/Q19-1021.pdf;
    3. Çöltekin and Rama https://www.degruyter.com/document/doi/10.1515/lingvan-2021-0007/html?lang=en
2. What is the best strategy to sample selection? Further reading:
    1. Muradoglu and Hulden https://aclanthology.org/2022.emnlp-main.492.pdf;
    2. Goldman and Tsarfaty https://aclanthology.org/2021.emnlp-main.159.pdf
    3. Kodner, Payne, Khalifa and Liu https://aclanthology.org/2023.acl-long.335/
    4. Kodner, Khalifa, Payne https://aclanthology.org/2023.emnlp-main.552/
3. What are the most essential paradigm parts (principal parts of the paradigm)? How do we learn them automatically? Will the samples selected by systems resemble the principal parts? Further reading on the topic:
    1. Finkel & Stump https://link.springer.com/article/10.1007/s11525-007-9115-9;
    2. Cotterell, Glassman, Kirov https://aclanthology.org/E17-2120.pdf 
4. How well do the systems learn syncretic forms? Further reading:
    1. Muradoglu, Evans, Vylomova: https://aclanthology.org/2020.alta-1.5.pdf

# Task organizers
- Aso Mahmudi, School of Computing and Information Systems, University of Melbourne
- Demian Inostroza Améstica
- Borja Herce, Department of Comparative Language Science, UZH Zurich
- Andreas Scherbakov, Constraint Technology International
- Khuyagbaatar Batsuren, School of Computing and Information Systems, University of Melbourne
- Duygu Ataman, Courant Institute of New York University
- Ryan Cotterell, Department of Computer Science, ETH Zurich
- Ekaterina Vylomova, School of Computing and Information Systems, University of Melbourne
 
# Contact details
- Aso: amahmudi@student.unimelb.edu.au
- Kat: ekaterina.vylomova@unimelb.edu.au
