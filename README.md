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
| Kurdish  | ckb | 1013    |  56.4               | 63        | 
| Turkish  | tur | 380     |  211.2              | 295       | 
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
- **Note**: To avoid server errors, please post your request in 100 lemma-tags sets bunches.

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
## oracle/GetStat (GET)
To check the current penalty score of your system, simply call this API with your system ID and the language code. For instance, this [link](https://test2.kurdinus.com/Oracle/GetStat?sysID=q1&lang=eng) gets the number of the times the system ''q1'' asked for target forms of the English data.
- **Note**: There is no oracle usage reset function. If you are training a new model, just use a different unique `sysID`.

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

# Baseline
The baseline settings are:
- We have used [neural character-level transformer](https://github.com/shijie-wu/neural-transducer/) described in [Wu and Cotterell, 2019](https://arxiv.org/abs/2005.10213)
- In the initial iteration, we employed a random selection process to extract 500 samples from the provided data (lemma and tags sets).
- Subsequently, utilizing the oracle API, we retrieved the forms of the chosen samples and partitioned them into a 90% training set and a 10% development set.
- Our model training started, and after 2,000 epochs, we stopped the training phase. The model then predicted the target form for the remaining pool data.
- In subsequent iterations, we refined our approach by first sorting the pool data based on the confidence level (negative log likelihood loss) of predictions. We then selected the 100 least confident predictions to request the correct forms from the oracle. Given the likelihood of incorrect predictions, acquiring these correct forms significantly enhances the quality of our training data. Additionally, we chose the 300 most confident predictions to cross-check with the oracle, as these are expected to be correct, thus incurring no penalty from the oracle.
- Following this, we incorporated these 400 new samples into our training data and removed them from the pool. Subsequently, we re-trained the model. These steps were iterated for four additional cycles to further refine our model's performance.
- Finally, we queried the oracle API for the statistics of our requests. 

The baseline results are:
| Measure |  eng | lat | rus | tur | ckb |
|---|---|---|---|---|---|
| Asked without prediction | 900 | 900 | 900 | 900 | 900 |
| Asked with wrong prediction | 874 | 118 | 86 | 149 | 79 |
| Asked with correct prediction | 326 | 1082 | 1114 | 1051 | 1121 |
| Total forms | 94964 | 20742 | 146287 | 80264 | 57103 |
| Asked from oracle | 2.2% | 10.1% | 1.4% | 2.6% | 3.7% |

Then, with the full data with target forms and the predictions of the final models, we calculated the accuracy of the models:
| Measure |  eng | lat | rus | tur | ckb |
|---|---|---|---|---|---|
| Count of correct predictions | 84309 | 16645 | 121858 | 75952 | 49510 |
| Count of wrong predictions | 8555 | 1997 | 22329 | 2215 | 5493 |
| Penalty | 10,329 | 3,015 | 23,315 | 3,264 | 6,472 |
| **Normalized Penalty** | 89.1% | 85.5% | 84.1% | 95.9% | 88.7% |
| Count of predictions | 92864 | 18642 | 144187 | 78167 | 55003 |
| **Accuracy of predictions** | 90.8% | 89.3% | 84.5% | 97.2% | 90.0% |

We calculate Penalty and Normalized Penalty as follows:
- Penalty = #retrieved_no_prediction + #retrieved_wrong_prediction + #wrong_prediction
- Normalised Penalty = (#total_forms - penalty)/#total_forms

# Important dates
- April 1, 2024: Development language data and Baseline systems released to participants
- (~~May 15~~) June 5, 2024: Surprise language data and the evaluator oracle API are available for participants
- (~~May 31~~) June 14, 2024: Final Submissions are due
- (~~June 3~~) June 17, 2024: Results announced to participants
- (~~June 22~~) July 6, 2024: System papers due for review
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
- John Mansfield, UZH Zurich
- Borja Herce, Department of Comparative Language Science, UZH Zurich
- Andreas Scherbakov, Constraint Technology International
- Khuyagbaatar Batsuren, School of Computing and Information Systems, University of Melbourne
- Duygu Ataman, Courant Institute of New York University
- Ryan Cotterell, Department of Computer Science, ETH Zurich
- Ekaterina Vylomova, School of Computing and Information Systems, University of Melbourne
 
# Contact details
- Aso: amahmudi@student.unimelb.edu.au
- Kat: ekaterina.vylomova@unimelb.edu.au
