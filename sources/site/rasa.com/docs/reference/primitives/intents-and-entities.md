# Source: https://rasa.com/docs/reference/primitives/intents-and-entities

On this page

##### NLU-based assistants

This section refers to building NLU-based assistants. If you are working with [Conversational AI with Language Models (CALM)](https://rasa.com/docs/calm), this content may not apply to you.

The goal of NLU (Natural Language Understanding) is to extract structured information from user messages. This usually includes the user's [intent](https://rasa.com/docs/reference/primitives/intents-and-entities/#intents) and any [entities](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities) their message contains. You can add extra information such as [regular expressions](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions) and [lookup tables](https://rasa.com/docs/reference/primitives/intents-and-entities/#lookup-tables) to your training data to help the model identify intents and entities correctly.

## Intents[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#intents 'Direct link to Intents')

### What is a Intent?

An intent represents the goal or purpose behind a user’s message. For example, a user might express the intent to book a ticket, ask for a weather update, or say hello. Intents help the assistant determine what the user wants to achieve.

NLU training data consists of example user utterances categorized by intent. To make it easier to use your intents, give them names that relate to what the user wants to accomplish with that intent, keep them in lowercase, and avoid spaces and special characters.

note

The `/` symbol is reserved as a delimiter to separate [retrieval intents](https://legacy-docs-oss.rasa.com/docs/rasa/glossary#retrieval-intent) from response text identifiers. Make sure not to use it in the name of your intents.

## Entities[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities 'Direct link to Entities')

### What is a Entity?

An entity is a specific piece of information extracted from a user’s message. They provide additional context to the intent. For example, in the message "Book a flight to Paris," the intent might be "book\_flight," and the entity would be "Paris" (destination)

Entities are structured pieces of information inside a user message. For entity extraction to work, you need to either specify training data to train an ML model or you need to define [regular expressions](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-entity-extraction) to extract entities using the [`RegexEntityExtractor`](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor) based on a character pattern.

When deciding which entities you need to extract, think about what information your assistant needs for its user goals. The user might provide additional pieces of information that you don't need for any user goal; you don't need to extract these as entities.

See the [training data format](https://rasa.com/docs/reference/primitives/training-data-format/) for details on how to annotate entities in your training data.

## Synonyms[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#synonyms 'Direct link to Synonyms')

Synonyms map extracted entities to a value other than the literal text extracted in a case-insensitive manner. You can use synonyms when there are multiple ways users refer to the same thing. Think of the end goal of extracting an entity, and figure out from there which values should be considered equivalent.

Let's say you had an entity `account` that you use to look up the user's balance. One of the possible account types is "credit". Your users also refer to their "credit" account as "credit account" and "credit card account".

In this case, you could define "credit card account" and "credit account" as synonyms to "credit":

nlu.yml

```
nlu:- synonym: credit  examples: |    - credit card account    - credit account
```

Then, if either of these phrases is extracted as an entity, it will be mapped to the value `credit`. Any alternate casing of these phrases (e.g. `CREDIT`, `credit ACCOUNT`) will also be mapped to the synonym.

Provide Training Examples

Synonym mapping only happens **after** entities have been extracted. That means that your training examples should include the synonym examples (`credit card account` and `credit account`) so that the model will learn to recognize these as entities and replace them with `credit`.

See the [training data format](https://rasa.com/docs/reference/primitives/training-data-format/) for details on how to include synonyms in your training data.

## Regular Expressions[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions 'Direct link to Regular Expressions')

You can use regular expressions to improve intent classification and entity extraction in combination with the [`RegexFeaturizer`](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer) and [`RegexEntityExtractor`](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor) components in the pipeline.

### Regular Expressions for Intent Classification[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-intent-classification 'Direct link to Regular Expressions for Intent Classification')

You can use regular expressions to improve intent classification by including the `RegexFeaturizer` component in your pipeline. When using the `RegexFeaturizer`, a regex does not act as a rule for classifying an intent. It only provides a feature that the intent classifier will use to learn patterns for intent classification. Currently, all intent classifiers make use of available regex features.

The name of a regex in this case is a human readable description. It can help you remember what a regex is used for, and it is the title of the corresponding pattern feature. It does not have to match any intent or entity name. A regex for a "help" request might look like this:

nlu.yml

```
nlu:- regex: help  examples: |    - \bhelp\b
```

The intent being matched could be `greet`,`help_me`, `assistance` or anything else.

Try to create your regular expressions in a way that they match as few words as possible. E.g. using `\bhelp\b` instead of `help.*`, as the later one might match the whole message whereas the first one only matches a single word.

Provide Training Examples

The `RegexFeaturizer` provides features to the intent classifier, but it doesn't predict the intent directly. Include enough examples containing the regular expression so that the intent classifier can learn to use the regular expression feature.

### Regular Expressions for Entity Extraction[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-entity-extraction 'Direct link to Regular Expressions for Entity Extraction')

If your entity has a deterministic structure, you can use regular expressions in one of two ways:

#### Regular Expressions as Features[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-as-features 'Direct link to Regular Expressions as Features')

You can use regular expressions to create features for the [`RegexFeaturizer`](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer) component in your NLU pipeline.

When using a regular expression with the `RegexFeaturizer`, the name of the regular expression does not matter. When using the `RegexFeaturizer`, a regular expression provides a feature that helps the model learn an association between intents/entities and inputs that fit the regular expression.

Provide Training Examples

The `RegexFeaturizer` provides features to the entity extractor, but it doesn't predict the entity directly. Include enough examples containing the regular expression so that the entity extractor can learn to use the regular expression feature.

Regex features for entity extraction are currently only supported by the `CRFEntityExtractor` component. Other entity extractors, like `MitieEntityExtractor` or `SpacyEntityExtractor`, won't use the generated features and their presence will not improve entity recognition for these extractors.

#### Regular Expressions for Rule-based Entity Extraction[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-rule-based-entity-extraction 'Direct link to Regular Expressions for Rule-based Entity Extraction')

You can use regular expressions for rule-based entity extraction using the [`RegexEntityExtractor`](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor) component in your NLU pipeline.

When using the `RegexEntityExtractor`, the name of the regular expression should match the name of the entity you want to extract. For example, you could extract account numbers of 10-12 digits by including this regular expression and at least two annotated examples in your training data:

nlu.yml

```
nlu:- regex: account_number  examples: |    - \d{10,12}- intent: inform  examples: |    - my account number is [1234567891](account_number)    - This is my account number [1234567891](account_number)
```

Whenever a user message contains a sequence of 10-12 digits, it will be extracted as an `account_number` entity. `RegexEntityExtractor` doesn't require training examples to learn to extract the entity, but you do need at least two annotated examples of the entity so that the NLU model can register it as an entity at training time.

## Lookup Tables[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#lookup-tables 'Direct link to Lookup Tables')

Lookup tables are lists of words used to generate case-insensitive regular expression patterns. They can be used in the same ways as [regular expressions](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions) are used, in combination with the [`RegexFeaturizer`](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer) and [`RegexEntityExtractor`](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor) components in the pipeline.

You can use lookup tables to help extract entities which have a known set of possible values. Keep your lookup tables as specific as possible. For example, to extract country names, you could add a lookup table of all countries in the world:

nlu.yml

```
nlu:- lookup: country  examples: |    - Afghanistan    - Albania    - ...    - Zambia    - Zimbabwe
```

When using lookup tables with `RegexFeaturizer`, provide enough examples for the intent or entity you want to match so that the model can learn to use the generated regular expression as a feature. When using lookup tables with `RegexEntityExtractor`, provide at least two annotated examples of the entity so that the NLU model can register it as an entity at training time.

## Entities Roles and Groups[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities-roles-and-groups 'Direct link to Entities Roles and Groups')

Annotating words as custom entities allows you to define certain concepts in your training data. For example, you can identify cities by annotating them:

```
I want to fly from [Berlin]{"entity": "city"} to [San Francisco]{"entity": "city"} .
```

However, sometimes you want to add more details to your entities.

For example, to build an assistant that should book a flight, the assistant needs to know which of the two cities in the example above is the departure city and which is the destination city. `Berlin` and `San Francisco` are both cities, but they play different **roles** in the message. To distinguish between the different roles, you can assign a role label in addition to the entity label.

```
- I want to fly from [Berlin]{"entity": "city", "role": "departure"} to [San Francisco]{"entity": "city", "role": "destination"}.
```

You can also group different entities by specifying a **group** label next to the entity label. The group label can, for example, be used to define different orders. In the following example, the group label specifies which toppings go with which pizza and what size each pizza should be.

```
Give me a [small]{"entity": "size", "group": "1"} pizza with [mushrooms]{"entity": "topping", "group": "1"} anda [large]{"entity": "size", "group": "2"} [pepperoni]{"entity": "topping", "group": "2"}
```

See the [Training Data Format](https://rasa.com/docs/reference/primitives/training-data-format/#entities) for details on how to define entities with roles and groups in your training data.

The entity object returned by the extractor will include the detected role/group label.

```
{  "text": "Book a flight from Berlin to SF",  "intent": "book_flight",  "entities": [    {      "start": 19,      "end": 25,      "value": "Berlin",      "entity": "city",      "role": "departure",      "extractor": "CRFEntityExtractor"    },    {      "start": 29,      "end": 31,      "value": "San Francisco",      "entity": "city",      "role": "destination",      "extractor": "CRFEntityExtractor"    }  ]}
```

note

Entity roles and groups are currently only supported by the [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor).

In order to properly train your model with entities that have roles and groups, make sure to include enough training examples for every combination of entity and role or group label. To enable the model to generalize, make sure to have some variation in your training examples. For example, you should include examples like `fly TO y FROM x`, not only `fly FROM x TO y`.

To fill slots from entities with a specific role/group, you need to define a `from_entity` [slot mapping](https://legacy-docs-oss.rasa.com/docs/rasa/domain/#slot-mappings) for the slot and specify the role/group that is required. For example:

domain.ymml

```
entities:   - city:       roles:       - departure       - destinationslots:  departure:    type: any    mappings:    - type: from_entity      entity: city      role: departure  destination:    type: any    mappings:    - type: from_entity      entity: city      role: destination
```

## BILOU Entity Tagging[​](https://rasa.com/docs/reference/primitives/intents-and-entities/#bilou-entity-tagging 'Direct link to BILOU Entity Tagging')

The [CRFEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#crfentityextractor) has the option `BILOU_flag`, which refers to a tagging schema that can be used by the machine learning model when processing entities. `BILOU` is short for Beginning, Inside, Last, Outside, and Unit-length.

For example, the training example

```
[Alex]{"entity": "person"} is going with [Marty A. Rick]{"entity": "person"} to [Los Angeles]{"entity": "location"}.
```

is first split into a list of tokens. Then the machine learning model applies the tagging schema as shown below depending on the value of the option `BILOU_flag`:

| token | `BILOU_flag = true` | `BILOU_flag = false` |
| --- | --- | --- |
| alex | U-person | person |
| is | O | O |
| going | O | O |
| with | O | O |
| marty | B-person | person |
| a | I-person | person |
| rick | L-person | person |
| to | O | O |
| los | B-location | location |
| angeles | L-location | location |

The BILOU tagging schema is richer compared to the normal tagging schema. It may help to improve the performance of the machine learning model when predicting entities.

inconsistent BILOU tags

When the option `BILOU_flag` is set to `True`, the model may predict inconsistent BILOU tags, e.g. `B-person I-location L-person`. Rasa uses some heuristics to clean up the inconsistent BILOU tags. For example, `B-person I-location L-person` would be changed into `B-person I-person L-person`.

- [Intents](https://rasa.com/docs/reference/primitives/intents-and-entities/#intents)
- [Entities](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities)
- [Synonyms](https://rasa.com/docs/reference/primitives/intents-and-entities/#synonyms)
- [Regular Expressions](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions)
 - [Regular Expressions for Intent Classification](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-intent-classification)
 - [Regular Expressions for Entity Extraction](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions-for-entity-extraction)
- [Lookup Tables](https://rasa.com/docs/reference/primitives/intents-and-entities/#lookup-tables)
- [Entities Roles and Groups](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities-roles-and-groups)
- [BILOU Entity Tagging](https://rasa.com/docs/reference/primitives/intents-and-entities/#bilou-entity-tagging)