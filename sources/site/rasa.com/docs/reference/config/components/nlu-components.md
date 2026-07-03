# Source: https://rasa.com/docs/reference/config/components/nlu-components

On this page

##### NLU-based assistants

This section refers to building NLU-based assistants. If you are working with [Conversational AI with Language Models (CALM)](https://rasa.com/docs/calm), this content may not apply to you.

Installation Requirements

To use NLU components, you need to install the `nlu` dependency group:

```
pip install 'rasa-pro[nlu]'
```

For more information about dependency groups, see our [Python Versions and Dependencies](https://rasa.com/docs/reference/python-versions-and-dependencies/) reference page.

## Tokenizers[​](https://rasa.com/docs/reference/config/components/nlu-components/#tokenizers 'Direct link to Tokenizers')

Tokenizers split text into tokens. If you want to split intents into multiple labels, e.g. for predicting multiple intents or for modeling hierarchical intent structure, use the following flags with any tokenizer:

- `intent_tokenization_flag` indicates whether to tokenize intent labels or not. Set it to `True`, so that intent labels are tokenized.
 
- `intent_split_symbol` sets the delimiter string to split the intent labels, default is underscore (`_`).
 

### WhitespaceTokenizer[​](https://rasa.com/docs/reference/config/components/nlu-components/#whitespacetokenizer 'Direct link to WhitespaceTokenizer')

- **Short**
 
 Tokenizer using whitespaces as a separator
 
- **Outputs**
 
 `tokens` for user messages, responses (if present), and intents (if specified)
 
- **Requires**
 
 Nothing
 
- **Description**
 
 Creates a token for every whitespace separated character sequence.
 
 Any character not in: `a-zA-Z0-9_#@&` will be substituted with whitespace before splitting on whitespace if the character fulfills any of the following conditions:
 
 - the character follows a whitespace: `" !word"` → `"word"`
 - the character precedes a whitespace: `"word! "` → `"word"`
 - the character is at the beginning of the string: `"!word"` → `"word"`
 - the character is at the end of the string: `"word!"` → `"word"`
 
 Note that:
 
 - `"wo!rd"` → `"wo!rd"`
 
 In addition, any character not in: `a-zA-Z0-9_#@&.~:\/?[]()!$*+,;=-` will be substituted with whitespace before splitting on whitespace if the character is not between numbers:
 
 - `"twenty\{one"` → `"twenty"`, `"one"` ("{"\` is not between numbers)
    -   `"20\{1"` → `"20\{1"` ("{"\` _is_ between numbers)
 
 Note that:
 
 - `"name@example.com"` → `"name@example.com"`
 - `"10,000.1"` → `"10,000.1"`
 - `"1 - 2"` → `"1"`,`"2"`
- **Configuration**
 
 config.yml
 
 ```
    pipeline:- name: "WhitespaceTokenizer"  # Flag to check whether to split intents  "intent_tokenization_flag": False  # Symbol on which intent should be split  "intent_split_symbol": "_"  # Regular expression to detect tokens  "token_pattern": None
    ```
 

## Featurizers[​](https://rasa.com/docs/reference/config/components/nlu-components/#featurizers 'Direct link to Featurizers')

Text featurizers are divided into two different categories: sparse featurizers and dense featurizers. Sparse featurizers are featurizers that return feature vectors with a lot of missing values, e.g. zeros. As those feature vectors would normally take up a lot of memory, we store them as sparse features. Sparse features only store the values that are non zero and their positions in the vector. Thus, we save a lot of memory and are able to train on larger datasets.

All featurizers can return two different kind of features: sequence features and sentence features. The sequence features are a matrix of size `(number-of-tokens x feature-dimension)`. The matrix contains a feature vector for every token in the sequence. This allows us to train sequence models. The sentence features are represented by a matrix of size `(1 x feature-dimension)`. It contains the feature vector for the complete utterance. The sentence features can be used in any bag-of-words model. The corresponding classifier can therefore decide what kind of features to use. Note: The `feature-dimension` for sequence and sentence features does not have to be the same.

### LanguageModelFeaturizer[​](https://rasa.com/docs/reference/config/components/nlu-components/#languagemodelfeaturizer 'Direct link to LanguageModelFeaturizer')

- **Short**
 
 Creates a vector representation of user message and response (if specified) using a pre-trained language model.
 
- **Outputs**
 
 `dense_features` for user messages and responses
 
- **Type**
 
 Dense featurizer
 
- **Description**
 
 Creates features for entity extraction, intent classification, and response selection. Uses a pre-trained language model to compute vector representations of input text.
 
 note
 
 Please make sure that you use a language model which is pre-trained on the same language corpus as that of your training data.
 
- **Configuration**
 
 Include a [Tokenizer](https://rasa.com/docs/reference/config/components/nlu-components/#tokenizers) component before this component.
 
 You should specify what language model to load via the parameter `model_name`. See the below table for the currently supported language models. The weights to be loaded can be specified by the additional parameter `model_weights`. If left empty, it uses the default model weights listed in the table.
 
 ```
    +----------------+--------------+-------------------------+| Language Model | Parameter    | Default value for       ||                | "model_name" | "model_weights"         |+----------------+--------------+-------------------------+| BERT           | bert         | rasa/LaBSE              |+----------------+--------------+-------------------------+| GPT            | gpt          | openai-gpt              |+----------------+--------------+-------------------------+| GPT-2          | gpt2         | gpt2                    |+----------------+--------------+-------------------------+| XLNet          | xlnet        | xlnet-base-cased        |+----------------+--------------+-------------------------+| DistilBERT     | distilbert   | distilbert-base-uncased |+----------------+--------------+-------------------------+| RoBERTa        | roberta      | roberta-base            |+----------------+--------------+-------------------------+| camemBERT      | camembert    | camembert-base          |+----------------+--------------+-------------------------+
    ```
 
 Apart from the default pretrained model weights, further models can be used from [HuggingFace models](https://huggingface.co/models) provided the following conditions are met (the mentioned files can be found in the "Files and versions" section of the model website):
 
 - The model architecture is one of the supported language models (check that the `model_type` in `config.json` is listed in the table's column `model_name`)
 - The model has pretrained Tensorflow weights (check that the file `tf_model.h5` exists, at this time Safetensors are not supported.)
 - The model uses the default tokenizer (`config.json` should not contain a custom `tokenizer_class` setting)
 
 note
 
 While the `LaBSE` weights are loaded by default for the `bert` architecture offering a multi-lingual model trained on 112 languages (see our [tutorial](https://www.youtube.com/watch?v=7tAWk_Coj-s) and the original [paper](https://arxiv.org/pdf/2007.01852.pdf)), we now recommend using `MiniLM` model for better performance.
 
 The `LaBSE` weights can still serve as a baseline for initial testing and development. After establishing this baseline, we strongly encourage exploring optimization with the `MiniLM` to improve your assistant effectiveness, before trying to optimize this component with other weights/architectures.
 
 The following configuration loads the language model BERT with `rasa/LaBSE` weights, which can be found [here](https://huggingface.co/rasa/LaBSE/tree/main):
 
 config.yml
 
 ```
    pipeline:  - name: LanguageModelFeaturizer    # Name of the language model to use    model_name: "bert"    # Pre-Trained weights to be loaded    model_weights: "rasa/LaBSE"    # An optional path to a directory from which    # to load pre-trained model weights.    # If the requested model is not found in the    # directory, it will be downloaded and    # cached in this directory for future use.    # The default value of `cache_dir` can be    # set using the environment variable    # `TRANSFORMERS_CACHE`, as per the    # Transformers library.    cache_dir: null
    ```
 
 For enhanced performance, we recommend the `sentence-transformers/all-MiniLM-L6-v2` weights, which can be found [here](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2):
 
 config.yml
 
 ```
    pipeline:- name: LanguageModelFeaturizer  model_name: "bert"  model_weights: "sentence-transformers/all-MiniLM-L6-v2"  cache_dir: null
    ```
 

### RegexFeaturizer[​](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer 'Direct link to RegexFeaturizer')

- **Short**
 
 Creates a vector representation of user message using regular expressions.
 
- **Outputs**
 
 `sparse_features` for user messages and `tokens.pattern`
 
- **Requires**
 
 `tokens`
 
- **Type**
 
 Sparse featurizer
 
- **Description**
 
 Creates features for entity extraction and intent classification. During training the `RegexFeaturizer` creates a list of regular expressions defined in the training data format. For each regex, a feature will be set marking whether this expression was found in the user message or not. All features will later be fed into an intent classifier / entity extractor to simplify classification (assuming the classifier has learned during the training phase, that this set feature indicates a certain intent / entity). Regex features for entity extraction are currently only supported by the [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor).
 
- **Configuration**
 
 Make the featurizer case insensitive by adding the `case_sensitive: False` option, the default being `case_sensitive: True`.
 
 To correctly process languages such as Chinese that don't use whitespace for word separation, the user needs to add the `use_word_boundaries: False` option, the default being `use_word_boundaries: True`.
 
 config.yml
 
 ```
    pipeline:- name: "RegexFeaturizer"  # Text will be processed with case sensitive as default  "case_sensitive": True  # use match word boundaries for lookup table  "use_word_boundaries": True
    ```
 

### CountVectorsFeaturizer[​](https://rasa.com/docs/reference/config/components/nlu-components/#countvectorsfeaturizer 'Direct link to CountVectorsFeaturizer')

- **Short**
 
 Creates bag-of-words representation of user messages, intents, and responses.
 
- **Outputs**
 
 `sparse_features` for user messages, intents, and responses
 
- **Requires**
 
 `tokens`
 
- **Type**
 
 Sparse featurizer
 
- **Description**
 
 Creates features for intent classification and response selection. Creates bag-of-words representation of user message, intent, and response using [sklearn's CountVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html). All tokens which consist only of digits (e.g. 123 and 99 but not a123d) will be assigned to the same feature.
 
- **Configuration**
 
 See [sklearn's CountVectorizer docs](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html) for detailed description of the configuration parameters.
 
 This featurizer can be configured to use word or character n-grams, using the `analyzer` configuration parameter. By default `analyzer` is set to `word` so word token counts are used as features. If you want to use character n-grams, set `analyzer` to `char` or `char_wb`. The lower and upper boundaries of the n-grams can be configured via the parameters `min_ngram` and `max_ngram`. By default both of them are set to `1`. By default the featurizer takes the lemma of a word instead of the word directly if it is available. You can disable this behavior by setting `use_lemma` to `False`.
 
 note
 
 Option `char_wb` creates character n-grams only from text inside word boundaries; n-grams at the edges of words are padded with space. This option can be used to create [Subword Semantic Hashing](https://arxiv.org/abs/1810.07150).
 
 note
 
 For character n-grams do not forget to increase `min_ngram` and `max_ngram` parameters. Otherwise the vocabulary will contain only single letters.
 
 Handling Out-Of-Vocabulary (OOV) words:
 
 note
 
 Enabled only if `analyzer` is `word`.
 
 Since the training is performed on limited vocabulary data, it cannot be guaranteed that during prediction an algorithm will not encounter an unknown word (a word that were not seen during training). In order to teach an algorithm how to treat unknown words, some words in training data can be substituted by generic word `OOV_token`. In this case during prediction all unknown words will be treated as this generic word `OOV_token`.
 
 For example, one might create separate intent `outofscope` in the training data containing messages of different number of `OOV_token` s and maybe some additional general words. Then an algorithm will likely classify a message with unknown words as this intent `outofscope`.
 
 You can either set the `OOV_token` or a list of words `OOV_words`:
 
 - `OOV_token` set a keyword for unseen words; if training data contains `OOV_token` as words in some messages, during prediction the words that were not seen during training will be substituted with provided `OOV_token`; if `OOV_token=None` (default behavior) words that were not seen during training will be ignored during prediction time;
 
 - `OOV_words` set a list of words to be treated as `OOV_token` during training; if a list of words that should be treated as Out-Of-Vocabulary is known, it can be set to `OOV_words` instead of manually changing it in training data or using custom preprocessor.
 
 
 note
 
 This featurizer creates a bag-of-words representation by **counting** words, so the number of `OOV_token` in the sentence might be important.
 
 note
 
 Providing `OOV_words` is optional, training data can contain `OOV_token` input manually or by custom additional preprocessor. Unseen words will be substituted with `OOV_token` **only** if this token is present in the training data or `OOV_words` list is provided.
 
 If you want to share the vocabulary between user messages and intents, you need to set the option `use_shared_vocab` to `True`. In that case a common vocabulary set between tokens in intents and user messages is build.
 
 config.yml
 
 ```
    pipeline:- name: "CountVectorsFeaturizer"  # Analyzer to use, either 'word', 'char', or 'char_wb'  "analyzer": "word"  # Set the lower and upper boundaries for the n-grams  "min_ngram": 1  "max_ngram": 1  # Set the out-of-vocabulary token  "OOV_token": "_oov_"  # Whether to use a shared vocab  "use_shared_vocab": False
    ```
 
 **Configuring for incremental training**
 
 To ensure that `sparse_features` are of fixed size during [incremental training](https://rasa.com/docs/reference/config/components/nlu-components/#incremental-training), the component should be configured to account for additional vocabulary tokens that may be added as part of new training examples in the future. To do so, configure the `additional_vocabulary_size` parameter while training the base model from scratch:
 
 config.yml
 
 ```
    pipeline:- name: CountVectorsFeaturizer  additional_vocabulary_size:    text: 1000    response: 1000    action_text: 1000
    ```
 
 As in the above example, you can define additional vocabulary size for each of `text` (user messages), `response` (bot responses used by `ResponseSelector`) and `action_text` (bot responses not used by `ResponseSelector`). If you are building a shared vocabulary (`use_shared_vocab=True`), you only need to define a value for the `text` attribute. If any of the attribute is not configured by the user, the component takes half of the current vocabulary size as the default value for the attribute's `additional_vocabulary_size`. This number is kept at a minimum of 1000 in order to avoid running out of additional vocabulary slots too frequently during incremental training. Once the component runs out of additional vocabulary slots, the new vocabulary tokens are dropped and not considered during featurization. At this point, it is advisable to retrain a new model from scratch.
 

The above configuration parameters are the ones you should configure to fit your model to your data. However, additional parameters exist that can be adapted.

More configurable parameters

```
+---------------------------+-------------------------+--------------------------------------------------------------+| Parameter                 | Default Value           | Description                                                  |+===========================+=========================+==============================================================+| use_shared_vocab          | False                   | If set to 'True' a common vocabulary is used for labels      ||                           |                         | and user message.                                            |+---------------------------+-------------------------+--------------------------------------------------------------+| analyzer                  | word                    | Whether the features should be made of word n-gram or        ||                           |                         | character n-grams. Option 'char_wb' creates character        ||                           |                         | n-grams only from text inside word boundaries;               ||                           |                         | n-grams at the edges of words are padded with space.         ||                           |                         | Valid values: 'word', 'char', 'char_wb'.                     |+---------------------------+-------------------------+--------------------------------------------------------------+| strip_accents             | None                    | Remove accents during the pre-processing step.               ||                           |                         | Valid values: 'ascii', 'unicode', 'None'.                    |+---------------------------+-------------------------+--------------------------------------------------------------+| stop_words                | None                    | A list of stop words to use.                                 ||                           |                         | Valid values: 'english' (uses an internal list of            ||                           |                         | English stop words), a list of custom stop words, or         ||                           |                         | 'None'.                                                      |+---------------------------+-------------------------+--------------------------------------------------------------+| min_df                    | 1                       | When building the vocabulary ignore terms that have a        ||                           |                         | document frequency strictly lower than the given threshold.  |+---------------------------+-------------------------+--------------------------------------------------------------+| max_df                    | 1                       | When building the vocabulary ignore terms that have a        ||                           |                         | document frequency strictly higher than the given threshold  ||                           |                         | (corpus-specific stop words).                                |+---------------------------+-------------------------+--------------------------------------------------------------+| min_ngram                 | 1                       | The lower boundary of the range of n-values for different    ||                           |                         | word n-grams or char n-grams to be extracted.                |+---------------------------+-------------------------+--------------------------------------------------------------+| max_ngram                 | 1                       | The upper boundary of the range of n-values for different    ||                           |                         | word n-grams or char n-grams to be extracted.                |+---------------------------+-------------------------+--------------------------------------------------------------+| max_features              | None                    | If not 'None', build a vocabulary that only consider the top ||                           |                         | max_features ordered by term frequency across the corpus.    |+---------------------------+-------------------------+--------------------------------------------------------------+| lowercase                 | True                    | Convert all characters to lowercase before tokenizing.       |+---------------------------+-------------------------+--------------------------------------------------------------+| OOV_token                 | None                    | Keyword for unseen words.                                    |+---------------------------+-------------------------+--------------------------------------------------------------+| OOV_words                 | []                      | List of words to be treated as 'OOV_token' during training.  |+---------------------------+-------------------------+--------------------------------------------------------------+| alias                     | CountVectorFeaturizer   | Alias name of featurizer.                                    |+---------------------------+-------------------------+--------------------------------------------------------------+| use_lemma                 | True                    | Use the lemma of words for featurization.                    |+---------------------------+-------------------------+--------------------------------------------------------------+| additional_vocabulary_size| text: 1000              | Size of additional vocabulary to account for incremental     ||                           | response: 1000          | training while training a model from scratch                 ||                           | action_text: 1000       |                                                              |+---------------------------+-------------------------+--------------------------------------------------------------+
```

### LexicalSyntacticFeaturizer[​](https://rasa.com/docs/reference/config/components/nlu-components/#lexicalsyntacticfeaturizer 'Direct link to LexicalSyntacticFeaturizer')

- **Short**
 
 Creates lexical and syntactic features for a user message to support entity extraction.
 
- **Outputs**
 
 `sparse_features` for user messages
 
- **Requires**
 
 `tokens`
 
- **Type**
 
 Sparse featurizer
 
- **Description**
 
 Creates features for entity extraction. Moves with a sliding window over every token in the user message and creates features according to the configuration (see below). As a default configuration is present, you don't need to specify a configuration.
 
- **Configuration**
 
 You can configure what kind of lexical and syntactic features the featurizer should extract. The following features are available:
 
 ```
    ==============  ==========================================================================================Feature Name    Description==============  ==========================================================================================BOS             Checks if the token is at the beginning of the sentence.EOS             Checks if the token is at the end of the sentence.low             Checks if the token is lower case.upper           Checks if the token is upper case.title           Checks if the token starts with an uppercase character and all remaining characters are                lowercased.digit           Checks if the token contains just digits.prefix5         Take the first five characters of the token.prefix2         Take the first two characters of the token.suffix5         Take the last five characters of the token.suffix3         Take the last three characters of the token.suffix2         Take the last two characters of the token.suffix1         Take the last character of the token.==============  ==========================================================================================
    ```
 
 As the featurizer is moving over the tokens in a user message with a sliding window, you can define features for previous tokens, the current token, and the next tokens in the sliding window. You define the features as a \[before, token, after\] array. If you want to define features for the token before, the current token, and the token after, your features configuration would look like this:
 
 config.yml
 
 ```
    pipeline:- name: LexicalSyntacticFeaturizer  "features": [    ["low", "title", "upper"],    ["BOS", "EOS", "low", "upper", "title", "digit"],    ["low", "title", "upper"],  ]
    ```
 
 This configuration is also the default configuration.
 

## Intent Classifiers[​](https://rasa.com/docs/reference/config/components/nlu-components/#intent-classifiers 'Direct link to Intent Classifiers')

Intent classifiers assign one of the intents defined in the domain file to incoming user messages.

### LogisticRegressionClassifier[​](https://rasa.com/docs/reference/config/components/nlu-components/#logisticregressionclassifier 'Direct link to LogisticRegressionClassifier')

- **Short**
 
 Logistic regression intent classifier, using the [scikit-learn implementation](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html).
 
- **Outputs**
 
 `intent` and `intent_ranking`
 
- **Requires**
 
 Either `sparse_features` or `dense_features` need to be present.
 
- **Output-Example**
 

```
{  "intent": { "name": "greet", "confidence": 0.78 },  "intent_ranking": [    {      "confidence": 0.78,      "name": "greet"    },    {      "confidence": 0.14,      "name": "goodbye"    },    {      "confidence": 0.08,      "name": "restaurant_search"    }  ]}
```

- **Description**
 
 This classifier uses scikit-learn's logistic regression implementation to perform intent classification. It's able to use only sparse features, but will also pick up any dense features that are present. In general, DIET should yield higher accuracy results, but this classifier should train faster and may be used as a lightweight benchmark. Our implementation uses the base settings from scikit-learn, with the exception of the `class_weight` parameter where we assume the `"balanced"` setting.
 
- **Configuration**
 

An example configuration with all the defaults can be found below.

```
pipeline:  - name: LogisticRegressionClassifier    max_iter: 100    solver: lbfgs    tol: 0.0001    random_state: 42    ranking_length: 10
```

There configuration parameters are briefly explained below.

- `max_iter`: Maximum number of iterations taken for the solvers to converge.
- `solver`: Solver to be used. For very small datasets you might consider `liblinear`.
- `tol`: Tolerance for stopping criteria of the optimizer.
- `random_state`: Used to shuffle the data before training.
- `ranking_length`: Number of top intents to report. Set to 0 to report all intents

More details on the parameters can be found on the [scikit-learn documentation page](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html).

### SklearnIntentClassifier[​](https://rasa.com/docs/reference/config/components/nlu-components/#sklearnintentclassifier 'Direct link to SklearnIntentClassifier')

- **Short**
 
 Sklearn intent classifier
 
- **Outputs**
 
 `intent` and `intent_ranking`
 
- **Requires**
 
 `dense_features` for user messages
 
- **Output-Example**
 
 ```
    {  "intent": { "name": "greet", "confidence": 0.78 },  "intent_ranking": [    {      "confidence": 0.78,      "name": "greet"    },    {      "confidence": 0.14,      "name": "goodbye"    },    {      "confidence": 0.08,      "name": "restaurant_search"    }  ]}
    ```
 
- **Description**
 
 The sklearn intent classifier trains a linear SVM which gets optimized using a grid search. It also provides rankings of the labels that did not “win”. The `SklearnIntentClassifier` needs to be preceded by a dense featurizer in the pipeline. This dense featurizer creates the features used for the classification. For more information about the algorithm itself, take a look at the [GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html) documentation.
 
- **Configuration**
 
 During the training of the SVM a hyperparameter search is run to find the best parameter set. In the configuration you can specify the parameters that will get tried.
 
 config.yml
 
 ```
    pipeline:- name: "SklearnIntentClassifier"  # Specifies the list of regularization values to  # cross-validate over for C-SVM.  # This is used with the ``kernel`` hyperparameter in GridSearchCV.  C: [1, 2, 5, 10, 20, 100]  # Specifies the kernel to use with C-SVM.  # This is used with the ``C`` hyperparameter in GridSearchCV.  kernels: ["linear"]  # Gamma parameter of the C-SVM.  "gamma": [0.1]  # We try to find a good number of cross folds to use during  # intent training, this specifies the max number of folds.  "max_cross_validation_folds": 5  # Scoring function used for evaluating the hyper parameters.  # This can be a name or a function.  "scoring_function": "f1_weighted"
    ```
 

### KeywordIntentClassifier[​](https://rasa.com/docs/reference/config/components/nlu-components/#keywordintentclassifier 'Direct link to KeywordIntentClassifier')

- **Short**
 
 Simple keyword matching intent classifier, intended for small, short-term projects.
 
- **Output s**
 
 `intent`
 
- **Requires**
 
 Nothing
 
- **Output-Example**
 
 ```
    {  "intent": { "name": "greet", "confidence": 1.0 }}
    ```
 
- **Description**
 
 This classifier works by searching a message for keywords. The matching is case sensitive by default and searches only for exact matches of the keyword-string in the user message. The keywords for an intent are the examples of that intent in the NLU training data. This means the entire example is the keyword, not the individual words in the example.
 
- **Configuration**
 
 config.yml
 
 ```
    pipeline:- name: "KeywordIntentClassifier"  case_sensitive: True
    ```
 

## Entity Extractors[​](https://rasa.com/docs/reference/config/components/nlu-components/#entity-extractors 'Direct link to Entity Extractors')

Entity extractors extract entities, such as person names or locations, from the user message.

note

If you use multiple entity extractors, we advise that each extractor targets an exclusive set of entity types. For example, use [Duckling](https://rasa.com/docs/reference/config/components/nlu-components/#ducklingentityextractor) to extract dates and times, and [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor) to extract person names. Otherwise, if multiple extractors target the same entity types, it is very likely that entities will be extracted multiple times.

For example, if you use two or more general purpose extractors like [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor), the entity types in your training data will be found and extracted by all of them. If the [slots](https://rasa.com/docs/reference/primitives/slots/) you are filling with your entity types are of type `text`, then the last extractor in your pipeline will win. If the slot is of type `list`, then all results will be added to the list, including duplicates.

Another, less obvious case of duplicate/overlapping extraction can happen even if extractors focus on different entity types. Imagine a food delivery bot and a user message like `I would like to order the Monday special`. Hypothetically, if your time extractor's performance isn't very good, it might extract `Monday` here as a time for the order, and your other extractor might extract `Monday special` as the meal.

### CRFEntityExtractor[​](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor 'Direct link to CRFEntityExtractor')

- **Short**
 
 Conditional random field (CRF) entity extraction
 
- **Outputs**
 
 `entities`
 
- **Requires**
 
 `tokens` and `dense_features` (optional)
 
- **Output-Example**
 
 ```
    {  "entities": [    {      "value": "New York City",      "start": 20,      "end": 33,      "entity": "city",      "confidence": 0.874,      "extractor": "CRFEntityExtractor"    }  ]}
    ```
 
- **Description**
 
 This component implements a conditional random fields (CRF) to do named entity recognition. CRFs can be thought of as an undirected Markov chain where the time steps are words and the states are entity classes. Features of the words (capitalization, POS tagging, etc.) give probabilities to certain entity classes, as are transitions between neighbouring entity tags: the most likely set of tags is then calculated and returned.
 
 If you want to pass custom features, such as pre-trained word embeddings, to `CRFEntityExtractor`, you can add any dense featurizer to the pipeline before the `CRFEntityExtractor` and subsequently configure `CRFEntityExtractor` to make use of the dense features by adding `"text_dense_feature"` to its feature configuration. `CRFEntityExtractor` automatically finds the additional dense features and checks if the dense features are an iterable of `len(tokens)`, where each entry is a vector. A warning will be shown in case the check fails. However, `CRFEntityExtractor` will continue to train just without the additional custom features. In case dense features are present, `CRFEntityExtractor` will pass the dense features to `sklearn_crfsuite` and use them for training.
 
- **Configuration**
 
 `CRFEntityExtractor` has a list of default features to use. However, you can overwrite the default configuration. The following features are available:
 
 ```
    ===================  ==========================================================================================Feature Name         Description===================  ==========================================================================================low                  word identity - use the lower-cased token as a feature.upper                Checks if the token is upper case.title                Checks if the token starts with an uppercase character and all remaining characters are                     lowercased.digit                Checks if the token contains just digits.prefix5              Take the first five characters of the token.prefix2              Take the first two characters of the token.suffix5              Take the last five characters of the token.suffix3              Take the last three characters of the token.suffix2              Take the last two characters of the token.suffix1              Take the last character of the token.pattern              Take the patterns defined by ``RegexFeaturizer``.bias                 Add an additional "bias" feature to the list of features.text_dense_features  Adds additional features from a dense featurizer.===================  ==========================================================================================
    ```
 
 As the featurizer is moving over the tokens in a user message with a sliding window, you can define features for previous tokens, the current token, and the next tokens in the sliding window. You define the features as \[before, token, after\] array.
 
 Additional you can set a flag to determine whether to use the BILOU tagging schema or not.
 
 - `BILOU_flag` determines whether to use BILOU tagging or not. Default `True`.
 
 config.yml
 
 ```
    pipeline:- name: "CRFEntityExtractor"  # BILOU_flag determines whether to use BILOU tagging or not.  "BILOU_flag": True  # features to extract in the sliding window  "features": [    ["low", "title", "upper"],    [      "bias",      "low",      "prefix5",      "prefix2",      "suffix5",      "suffix3",      "suffix2",      "upper",      "title",      "digit",      "pattern",      "text_dense_features"    ],    ["low", "title", "upper"],  ]  # The maximum number of iterations for optimization algorithms.  "max_iterations": 50  # weight of the L1 regularization  "L1_c": 0.1  # weight of the L2 regularization  "L2_c": 0.1  # Name of dense featurizers to use.  # If list is empty all available dense features are used.  "featurizers": []  # Indicated whether a list of extracted entities should be split into individual entities for a given entity type  "split_entities_by_comma":      address: False      email: True
    ```
 
 note
 
 If `pattern` features are used, you need to have `RegexFeaturizer` in your pipeline.
 
 note
 
 If `text_dense_features` features are used, you need to have a dense featurizer (e.g. `LanguageModelFeaturizer`) in your pipeline.
 

### DucklingEntityExtractor[​](https://rasa.com/docs/reference/config/components/nlu-components/#ducklingentityextractor 'Direct link to DucklingEntityExtractor')

- **Short**
 
 Duckling lets you extract common entities like dates, amounts of money, distances, and others in a number of languages.
 
- **Outputs**
 
 `entities`
 
- **Requires**
 
 Nothing
 
- **Output-Example**
 
 ```
    {  "entities": [    {      "end": 53,      "entity": "time",      "start": 48,      "value": "2017-04-10T00:00:00.000+02:00",      "confidence": 1.0,      "extractor": "DucklingEntityExtractor"    }  ]}
    ```
 
- **Description**
 
 To use this component you need to run a duckling server. The easiest option is to spin up a docker container using `docker run -p 8000:8000 rasa/duckling`.
 
 Alternatively, you can [install duckling directly on your machine](https://github.com/facebook/duckling#quickstart) and start the server.
 
 Duckling allows to recognize dates, numbers, distances and other structured entities and normalizes them. Please be aware that duckling tries to extract as many entity types as possible without providing a ranking. For example, if you specify both `number` and `time` as dimensions for the duckling component, the component will extract two entities: `10` as a number and `in 10 minutes` as a time from the text `I will be there in 10 minutes`. In such a situation, your application would have to decide which entity type is be the correct one. The extractor will always return 1.0 as a confidence, as it is a rule based system.
 
 The list of supported languages can be found in the [Duckling GitHub repository](https://github.com/facebook/duckling/tree/master/Duckling/Dimensions).
 
- **Configuration**
 
 Configure which dimensions, i.e. entity types, the duckling component should extract. A full list of available dimensions can be found in the [duckling project readme](https://github.com/facebook/duckling). Leaving the dimensions option unspecified will extract all available dimensions.
 
 config.yml
 
 ```
    pipeline:- name: "DucklingEntityExtractor"  # url of the running duckling server  url: "http://localhost:8000"  # dimensions to extract  dimensions: ["time", "number", "amount-of-money", "distance"]  # allows you to configure the locale, by default the language is  # used  locale: "de_DE"  # if not set the default timezone of Duckling is going to be used  # needed to calculate dates from relative expressions like "tomorrow"  timezone: "Europe/Berlin"  # Timeout for receiving response from http url of the running duckling server  # if not set the default timeout of duckling http url is set to 3 seconds.  timeout : 3
    ```
 

### RegexEntityExtractor[​](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor 'Direct link to RegexEntityExtractor')

- **Short**
 
 Extracts entities using the lookup tables and/or regexes defined in the training data
 
- **Outputs**
 
 `entities`
 
- **Requires**
 
 Nothing
 
- **Description**
 
 This component extract entities using the [lookup tables](https://rasa.com/docs/reference/primitives/intents-and-entities/#lookup-tables) and [regexes](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-entity-extraction) defined in the training data. The component checks if the user message contains an entry of one of the lookup tables or matches one of the regexes. If a match is found, the value is extracted as entity.
 
 This component only uses those regex features that have a name equal to one of the entities defined in the training data. Make sure to annotate at least one example per entity.
 
 note
 
 When you use this extractor in combination with [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor), it can lead to multiple extraction of entities. Especially if many training sentences have entity annotations for the entity types for which you also have defined regexes. See the big info box at the start of the [entity extractor section](https://rasa.com/docs/reference/config/components/nlu-components/#entity-extractors) for more info on multiple extraction.
 
 In the case where you seem to need both this RegexEntityExtractor and another of the aforementioned statistical extractors, we advise you to consider one of the following two options.
 
 Option 1 is advisable when you have exclusive entity types for each type of extractor. To make the sure the extractors don't interfere with one another annotate only one example sentence for each regex/lookup entity type, but not more.
 
 Option 2 is useful when you want to use regexes matches as additional signal for your statistical extractor, but you don't have separate entity types. In this case you will want to 1) add the [RegexFeaturizer](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer) before the extractors in your pipeline 2) annotate all your entity examples in the training data and 3) remove the RegexEntityExtractor from your pipeline. This way, your statistical extractors will receive additional signal about the presence of regex matches and will be able to statistically determine when to rely on these matches and when not to.
 
- **Configuration**
 
 Make the entity extractor case sensitive by adding the `case_sensitive: True` option, the default being `case_sensitive: False`.
 
 To correctly process languages such as Chinese that don't use whitespace for word separation, the user needs to add the `use_word_boundaries: False` option, the default being `use_word_boundaries: True`.
 
 config.yml
 
 ```
        pipeline:    - name: RegexEntityExtractor      # text will be processed with case insensitive as default      case_sensitive: False      # use lookup tables to extract entities      use_lookup_tables: True      # use regexes to extract entities      use_regexes: True      # use match word boundaries for lookup table      use_word_boundaries: True
    ```
 

### EntitySynonymMapper[​](https://rasa.com/docs/reference/config/components/nlu-components/#entitysynonymmapper 'Direct link to EntitySynonymMapper')

- **Short**
 
 Maps synonymous entity values to the same value.
 
- **Outputs**
 
 Modifies existing entities that previous entity extraction components found.
 
- **Requires**
 
 An extractor from [Entity Extractors](https://rasa.com/docs/reference/config/components/nlu-components/#entity-extractors)
 
- **Description**
 
 If the training data contains defined synonyms, this component will make sure that detected entity values will be mapped to the same value. For example, if your training data contains the following examples:
 
 ```
    [  {    "text": "I moved to New York City",    "intent": "inform_relocation",    "entities": [      {        "value": "nyc",        "start": 11,        "end": 24,        "entity": "city"      }    ]  },  {    "text": "I got a new flat in NYC.",    "intent": "inform_relocation",    "entities": [      {        "value": "nyc",        "start": 20,        "end": 23,        "entity": "city"      }    ]  }]
    ```
 
 This component will allow you to map the entities `New York City` and `NYC` to `nyc`. The entity extraction will return `nyc` even though the message contains `NYC`. When this component changes an existing entity, it appends itself to the processor list of this entity.
 
- **Configuration**
 
 config.yml
 
 ```
    pipeline:- name: "EntitySynonymMapper"
    ```
 
 note
 
 When using the `EntitySynonymMapper` as part of an NLU pipeline, it will need to be placed below any entity extractors in the configuration file.
 

## Incremental training[​](https://rasa.com/docs/reference/config/components/nlu-components/#incremental-training 'Direct link to Incremental training')

New in 2.2

This feature is experimental. We introduce experimental features to get feedback from our community, so we encourage you to try it out! However, the functionality might be changed or removed in the future. If you have feedback (positive or negative) please share it with us on the [Rasa Forum](https://forum.rasa.com).

In order to improve the performance of an assistant, it's helpful to practice CDD and add new training examples based on how your users have talked to your assistant. You can use `rasa train --finetune` to initialize the pipeline with an already trained model and further finetune it on the new training dataset that includes the additional training examples. This will help reduce the training time of the new model.

By default, the command picks up the latest model in the `models/` directory. If you have a specific model which you want to improve, you may specify the path to this by running `rasa train --finetune <path to model to finetune>`. Finetuning a model usually requires fewer epochs to train machine learning components like `DIETClassifier`, `ResponseSelector` and `TEDPolicy` compared to training from scratch. Either use a model configuration for finetuning which defines fewer epochs than before or use the flag `--epoch-fraction`. `--epoch-fraction` will use a fraction of the epochs specified for each machine learning component in the model configuration file. For example, if `DIETClassifier` is configured to use 100 epochs, specifying `--epoch-fraction 0.5` will only use 50 epochs for finetuning.

You can also finetune an NLU-only or dialogue management-only model by using `rasa train nlu --finetune` and `rasa train core --finetune` respectively.

To be able to fine tune a model, the following conditions must be met:

1. The configuration supplied should be exactly the same as the configuration used to train the model which is being finetuned. The only parameter that you can change is `epochs` for the individual machine learning components and policies.
 
2. The set of labels(intents, actions, entities and slots) for which the base model is trained should be exactly the same as the ones present in the training data used for finetuning. This means that you cannot add new intent, action, entity or slot labels to your training data during incremental training. You can still add new training examples for each of the existing labels. If you have added/removed labels in the training data, the pipeline needs to be trained from scratch.
 
3. The model to be finetuned is trained with `MINIMUM_COMPATIBLE_VERSION` of the currently installed rasa version.
 

- [Tokenizers](https://rasa.com/docs/reference/config/components/nlu-components/#tokenizers)
 - [WhitespaceTokenizer](https://rasa.com/docs/reference/config/components/nlu-components/#whitespacetokenizer)
- [Featurizers](https://rasa.com/docs/reference/config/components/nlu-components/#featurizers)
 - [LanguageModelFeaturizer](https://rasa.com/docs/reference/config/components/nlu-components/#languagemodelfeaturizer)
 - [RegexFeaturizer](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer)
 - [CountVectorsFeaturizer](https://rasa.com/docs/reference/config/components/nlu-components/#countvectorsfeaturizer)
 - [LexicalSyntacticFeaturizer](https://rasa.com/docs/reference/config/components/nlu-components/#lexicalsyntacticfeaturizer)
- [Intent Classifiers](https://rasa.com/docs/reference/config/components/nlu-components/#intent-classifiers)
 - [LogisticRegressionClassifier](https://rasa.com/docs/reference/config/components/nlu-components/#logisticregressionclassifier)
 - [SklearnIntentClassifier](https://rasa.com/docs/reference/config/components/nlu-components/#sklearnintentclassifier)
 - [KeywordIntentClassifier](https://rasa.com/docs/reference/config/components/nlu-components/#keywordintentclassifier)
- [Entity Extractors](https://rasa.com/docs/reference/config/components/nlu-components/#entity-extractors)
 - [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor)
 - [DucklingEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#ducklingentityextractor)
 - [RegexEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor)
 - [EntitySynonymMapper](https://rasa.com/docs/reference/config/components/nlu-components/#entitysynonymmapper)
- [Incremental training](https://rasa.com/docs/reference/config/components/nlu-components/#incremental-training)