# Source: https://rasa.com/docs/reference/primitives/training-data-format

On this page

##### NLU-based assistants

This section refers to building NLU-based assistants. If you are working with [Conversational AI with Language Models (CALM)](https://rasa.com/docs/calm), this content may not apply to you.

## Overview[​](https://rasa.com/docs/reference/primitives/training-data-format/#overview 'Direct link to Overview')

Rasa uses [YAML](https://yaml.org/spec/1.2/spec.html) as a unified and extendable way to manage all NLU training data; intents and entities. [Rasa Studio](https://rasa.com/docs/studio/) provides an additional layer on top of that, enabling the management of training data through a web-based interface.

You can split the training data over any number of YAML files, and each file can contain any combination of NLU data. The training data parser determines the training data type using top level keys.

The [domain](https://rasa.com/docs/reference/config/domain/) uses the same YAML format as the training data and can also be split across multiple files or combined in one file. The domain includes the definitions for [responses](https://rasa.com/docs/reference/primitives/responses/). See the [documentation for the domain](https://rasa.com/docs/reference/config/domain/) for information on how to format your domain file.

### High-Level Structure[​](https://rasa.com/docs/reference/primitives/training-data-format/#high-level-structure 'Direct link to High-Level Structure')

Each file can contain one or more **keys** with corresponding training data. One file can contain multiple keys, but each key can only appear once in a single file. The available keys are:

- `version`
- `nlu`

You should specify the `version` key in all YAML training data files. If you don't specify a version key in your training data file, Rasa will assume you are using the latest training data format specification supported by the version of Rasa you have installed. Training data files with a Rasa version greater than the version you have installed on your machine will be skipped. Currently, the latest training data format specification for Rasa 3.x is 3.1.

### Example[​](https://rasa.com/docs/reference/primitives/training-data-format/#example 'Direct link to Example')

Here's a short example which keeps all training data in a single file:

nlu.yml

```
version: "3.1"nlu:- intent: greet  examples: |    - Hey    - Hi    - hey there [Sara](name)- intent: faq/language  examples: |    - What language do you speak?    - Do you only handle english?
```

The `|` symbol

As shown in the above examples, the `user` and `examples` keys are followed by `|` (pipe) symbol. In YAML `|` identifies multi-line strings with preserved indentation. This helps to keep special symbols like `"`, `'` and others still available in the training examples.

## NLU Training Data[​](https://rasa.com/docs/reference/primitives/training-data-format/#nlu-training-data 'Direct link to NLU Training Data')

NLU training data consists of example user utterances categorized by [intent](https://rasa.com/docs/reference/primitives/intents-and-entities/#intents). Training examples can also include [entities](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities). Entities are structured pieces of information that can be extracted from a user's message. You can also add extra information such as regular expressions and lookup tables to your training data to help the model identify intents and entities correctly.

NLU training data is defined under the `nlu` key. Items that can be added under this key are:

- [Training examples](https://rasa.com/docs/reference/primitives/training-data-format/#training-examples) grouped by user intent e.g. optionally with annotated [entities](https://rasa.com/docs/reference/primitives/training-data-format/#entities)

nlu.yml

```
nlu:- intent: check_balance  examples: |    - What's my [credit](account) balance?    - What's the balance on my [credit card account]{"entity":"account","value":"credit"}
```

- [Synonyms](https://rasa.com/docs/reference/primitives/training-data-format/#synonyms)

nlu.yml

```
nlu:- synonym: credit  examples: |    - credit card account    - credit account
```

- [Regular expressions](https://rasa.com/docs/reference/primitives/training-data-format/#regular-expressions)

nlu.yml

```
nlu:- regex: account_number  examples: |    - \d{10,12}
```

- [Lookup tables](https://rasa.com/docs/reference/primitives/training-data-format/#lookup-tables)

nlu.yml

```
nlu:- lookup: banks  examples: |    - JPMC    - Comerica    - Bank of America
```

### Training Examples[​](https://rasa.com/docs/reference/primitives/training-data-format/#training-examples 'Direct link to Training Examples')

Training examples are grouped by [intent](https://rasa.com/docs/reference/primitives/intents-and-entities/#intents) and listed under the `examples` key. Usually, you'll list one example per line as follows:

nlu.yml

```
nlu:- intent: greet  examples: |    - hey    - hi    - whats up
```

However, it's also possible to use an extended format if you have a custom NLU component and need metadata for your examples:

nlu.yml

```
nlu:- intent: greet  examples:  - text: |      hi    metadata:      sentiment: neutral  - text: |      hey there!
```

The `metadata` key can contain arbitrary key-value data that is tied to an example and accessible by the components in the NLU pipeline. In the example above, the sentiment metadata could be used by a custom component in the pipeline for sentiment analysis.

You can also specify this metadata at the intent level:

nlu.yml

```
nlu:- intent: greet  metadata:    sentiment: neutral  examples:  - text: |      hi  - text: |      hey there!
```

In this case, the content of the `metadata` key is passed to every intent example.

If you want to specify [retrieval intents](https://legacy-docs-oss.rasa.com/docs/rasa/glossary#retrieval-intent), then your NLU examples will look as follows:

nlu.yml

```
nlu:- intent: chitchat/ask_name  examples: |    - What is your name?    - May I know your name?    - What do people call you?    - Do you have a name for yourself?- intent: chitchat/ask_weather  examples: |    - What's the weather like today?    - Does it look sunny outside today?    - Oh, do you mind checking the weather for me please?    - I like sunny days in Berlin.
```

All retrieval intents have a suffix added to them which identifies a particular response key for your assistant. In the above example, `ask_name` and `ask_weather` are the suffixes. The suffix is separated from the retrieval intent name by a `/` delimiter.

Special meaning of `/`

As shown in the above examples, the `/` symbol is reserved as a delimiter to separate retrieval intents from their associated response keys. Make sure not to use it in the name of your intents.

### Entities[​](https://rasa.com/docs/reference/primitives/training-data-format/#entities 'Direct link to Entities')

[Entities](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities) are structured pieces of information that can be extracted from a user's message.

Entities are annotated in training examples with the entity's name. In addition to the entity name, you can annotate an entity with [synonyms](https://rasa.com/docs/reference/primitives/intents-and-entities/#synonyms), [roles, or groups](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities-roles-and-groups).

In training examples, entity annotation would look like this:

nlu.yml

```
nlu:- intent: check_balance  examples: |    - how much do I have on my [savings](account) account    - how much money is in my [checking]{"entity": "account"} account    - What's the balance on my [credit card account]{"entity":"account","value":"credit"}
```

The full possible syntax for annotating an entity is:

```
[<entity-text>]{"entity": "<entity name>", "role": "<role name>", "group": "<group name>", "value": "<entity synonym>"}
```

The keywords `role`, `group`, and `value` are optional in this notation. The `value` field refers to synonyms. To understand what the labels `role` and `group` are for, see the section on [entity roles and groups](https://rasa.com/docs/reference/primitives/intents-and-entities/#entities-roles-and-groups).

### Synonyms[​](https://rasa.com/docs/reference/primitives/training-data-format/#synonyms 'Direct link to Synonyms')

Synonyms normalize your training data by mapping an extracted entity to a value other than the literal text extracted. You can define synonyms using the format:

nlu.yml

```
nlu:- synonym: credit  examples: |    - credit card account    - credit account
```

You can also define synonyms in-line in your training examples by specifying the `value` of the entity:

nlu.yml

```
nlu:- intent: check_balance  examples: |    - how much do I have on my [credit card account]{"entity": "account", "value": "credit"}    - how much do I owe on my [credit account]{"entity": "account", "value": "credit"}
```

Read more about synonyms on the [NLU Training Data page](https://rasa.com/docs/reference/primitives/intents-and-entities/#synonyms).

### Regular Expressions[​](https://rasa.com/docs/reference/primitives/training-data-format/#regular-expressions 'Direct link to Regular Expressions')

You can use regular expressions to improve intent classification and entity extraction using the [`RegexFeaturizer`](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer) and [`RegexEntityExtractor`](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor) components.

The format for defining a regular expression is as follows:

nlu.yml

```
nlu:- regex: account_number  examples: |    - \d{10,12}
```

Here `account_number` is the name of the regular expression. When used as features for the `RegexFeaturizer` the name of the regular expression does not matter. When using the `RegexEntityExtractor`, the name of the regular expression should match the name of the entity you want to extract.

Read more about when and how to use regular expressions with each component on the [NLU Training Data page](https://rasa.com/docs/reference/primitives/intents-and-entities/#regular-expressions).

### Lookup Tables[​](https://rasa.com/docs/reference/primitives/training-data-format/#lookup-tables 'Direct link to Lookup Tables')

Lookup tables are lists of words used to generate case-insensitive regular expression patterns. The format is as follows:

nlu.yml

```
nlu:- lookup: banks  examples: |    - JPMC    - Bank of America
```

When you supply a lookup table in your training data, the contents of that table are combined into one large regular expression. This regex is used to check each training example to see if it contains matches for entries in the lookup table.

Lookup table regexes are processed identically to the regular expressions directly specified in the training data and can be used either with the [RegexFeaturizer](https://rasa.com/docs/reference/config/components/nlu-components/#regexfeaturizer) or with the [RegexEntityExtractor](https://rasa.com/docs/reference/config/components/nlu-components/#regexentityextractor). The name of the lookup table is subject to the same constraints as the name of a regex feature.

Read more about using lookup tables on the [NLU Training Data page](https://rasa.com/docs/reference/primitives/intents-and-entities/#lookup-tables).

- [Overview](https://rasa.com/docs/reference/primitives/training-data-format/#overview)
 - [High-Level Structure](https://rasa.com/docs/reference/primitives/training-data-format/#high-level-structure)
 - [Example](https://rasa.com/docs/reference/primitives/training-data-format/#example)
- [NLU Training Data](https://rasa.com/docs/reference/primitives/training-data-format/#nlu-training-data)
 - [Training Examples](https://rasa.com/docs/reference/primitives/training-data-format/#training-examples)
 - [Entities](https://rasa.com/docs/reference/primitives/training-data-format/#entities)
 - [Synonyms](https://rasa.com/docs/reference/primitives/training-data-format/#synonyms)
 - [Regular Expressions](https://rasa.com/docs/reference/primitives/training-data-format/#regular-expressions)
 - [Lookup Tables](https://rasa.com/docs/reference/primitives/training-data-format/#lookup-tables)