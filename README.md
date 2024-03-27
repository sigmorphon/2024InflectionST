# Data-efficient Morphological Inflection and Simulation of Linguistic Fieldwork

This year we will focus on data efficiency and active learning. The task is a simulation of linguistic fieldwork where a linguist comes with a pre-existing dictionary (~lemmas are known) and a morphological paradigm structure for the corresponding part of speech (i.e. we assume that feature combinations are known but classes and forms are not). An oracle system will be serving as a native speaker, i.e. it provides access to complete paradigms for all lemmas: as an input it receives (1) a lemma, (2) target tags/features, (3) system id (~linguist). The oracle system has access to all forms, but it comes as a certain cost. Participants can send requests to retrieve a form or to check whether their prediction is correct (if it's incorrect, the oracle system returns a correct form). We introduce a variety of penalties depending on the use case scenario:
1. form retrieval without prediction (e.g.., in a cold start condition);
2. form check and the form is predicted correctly (e.g., lowest penalty; a speaker is satisfied);
3. form check, the predicted form is wrong and a correct form is provided (a native speaker is dissatisfied and fixes it);
4. the forms that were neither retrieved nor checked by the speaker but predicted incorrectly (the highest).

For the latter, you may imagine a situation where a linguist publishes a book and makes an error there (hence the highest penalty).
For each system we collect all forms that were retrieved via oracle. The task is to maximise the accuracy and minimise the number of requests to a speaker (oracle). The systems will also be evaluated in terms of their accuracy across paradigm classes. 

# Languages and Paradigms
Development languages: English verbs, Latin verbs, Turkish verbs, Kurdish verbs, Russian nouns

Statistics of the available languages:

| Language | #Lemmas |  Avg. Paradigm size | #Tag Sets | #Classes |
|----------|---------|--------------------|-----------|-------------|
| English  | 23741   |  4                  | 4         | 12          |
| Latin    | 947     |  24.2               | 54        | 9           |
| Kurdish  | 1022    |  62.3               | 70        | 46          |
| Turkish  | 386     |  703                | 703       | 46          |
| Russian  | 12705   |  11.5               | 16        | 306         |

Surprise languages: To be announced in May

# Oracle API
SIGMORPHON 2024 oracle API answers queries about the correct form of paradigms in available languages.
The input query will be in a specific format (including the participant system ID, the language code, the lemma, and the target tags as provided in the dev/test paradigms), and the API will provide the correct target form. 

The API is designed to score each system according to the number of API calls. Each API call adds up to the total penalty score of the model for a language.
This scoring mechanism helps evaluate the efficiency of learning morphological rules with less data, i.e., fewer requests to the oracle result in a better system. 

To obtain the correct target form via Direct API **GetForm**, you must provide the system ID, language code, lemma, and tags.
The output will be a string representation of the target form.
For example, this [link](https://test2.kurdinus.com/Oracle/GetForm?sysID=q1&lang=eng&lemma=go&tags=V;PST), for the system ''q1'', gets the past form of English verb ''go'' which is ''went''.

If you want to check a prediction of your model, you can use **CheckForm** command. This involves providing the system ID, language code, lemma, tags, and the prediction. The output will be a string representation of the correct target form. It's important to note that your prediction, whether wrong or correct, will be counted separately.
This [link](https://test2.kurdinus.com/Oracle/CheckForm?sysID=q1&lang=eng&lemma=go&tags=V;PST&predicted=goed) checks the wrong predidiction ''goed'' for the past form of ''go''.

To check the current penalty score of your system, simply call this API with your system ID and language code. For instance, this [link](https://test2.kurdinus.com/Oracle/GetStat?sysID=q1&lang=eng) gets the number of the times the system ''q1'' asked for target forms of the English data.

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
