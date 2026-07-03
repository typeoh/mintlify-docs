# Source: https://rasa.com/docs/reference/testing/test-case-conversion

On this page

## End-To-End Test Conversion[​](https://rasa.com/docs/reference/testing/test-case-conversion/#end-to-end-test-conversion 'Direct link to End-To-End Test Conversion')

New Beta Feature in 3.10

To facilitate a conversation-driven development approach and ensure the reliability of Rasa assistants, Rasa Pro 3.10 introduces automated end-to-end test case conversion. This feature allows you to convert your sample conversations into end-to-end test cases that use the new assertion format introduced in the same version. This feature is currently in beta (experimental) and may change in future Rasa versions.

We have introduced a new feature that converts sample conversations provided in CSV or Excel format into end-to-end YAML test cases. This allows you to automatically generate test cases from your existing conversational data, ensuring that your assistant behaves as expected from the very beginning of the development process.

We recommend running the [CLI command](https://rasa.com/docs/reference/testing/test-case-conversion/#cli-command) against 30-50 sample conversations per skill or use case that the assistant should handle. These sample conversations should be representative conversations of what the assistant should be able to handle.

It's essential to review the generated test cases and augment them with additional assertions as needed. This human review ensures the test cases are accurate, comprehensive, and tailored to meet specific requirements, further enhancing the robustness of your testing framework.

The new feature is released as a beta version, and we would love to hear your feedback, particularly on the usability and the value it brings to your testing workflow. We also have a list of [questions](https://rasa.com/docs/reference/testing/test-case-conversion/#beta-feedback-questions) that we would like to get feedback on. Please reach out to us through the Customer Office team to share your feedback.

To enable the feature, you need to set the environment variable `RASA_PRO_BETA_E2E_CONVERSION` to `true` in your testing environment.

### Benefits of End-to-End Test Conversion[​](https://rasa.com/docs/reference/testing/test-case-conversion/#benefits-of-end-to-end-test-conversion 'Direct link to Benefits of End-to-End Test Conversion')

- **Conversation-Driven Development (CDD) and Test-Driven Development (TDD) Workflow**: This feature enables CDD by employing TDD principle, which guides your assistant's development based on real conversations rather than pre-defined flows. This approach ensures it handles actual user interactions from the start.
- **Efficiency and Reduced Manual Effort**: Automates the generation of comprehensive test cases from conversational data, significantly reducing manual effort and speeding up development.
- **Iterative Improvement and Customization**: Enables continuous refinement of test cases, which should be reviewed by a human and augmented with extra assertions as needed.

### How It Works[​](https://rasa.com/docs/reference/testing/test-case-conversion/#how-it-works 'Direct link to How It Works')

You can use the `rasa data convert e2e <path>` CLI command to perform this conversion. Below is a detailed breakdown of the command and its functionality.

#### CLI Command[​](https://rasa.com/docs/reference/testing/test-case-conversion/#cli-command 'Direct link to CLI Command')

```
rasa data convert e2e <path>
```

#### Arguments[​](https://rasa.com/docs/reference/testing/test-case-conversion/#arguments 'Direct link to Arguments')

- `path` (required): Path to the input CSV or XLS/XLSX file containing the sample conversational data.
- `-o` or `--output` (optional): Output directory to store the generated YAML test cases. This must be a relative path pointing to a location inside the assistant project. The default value is `e2e_tests`.
- `--sheet-name` (required if using XLS/XLSX): Name of the sheet within the XLS/XLSX file that contains the relevant data.

#### Input Parsing[​](https://rasa.com/docs/reference/testing/test-case-conversion/#input-parsing 'Direct link to Input Parsing')

The command reads your input data file (CSV or XLS/XLSX) and prepares it for processing. A conversation can be stored in a single row or across multiple rows. You must add an empty row between each conversation, this enables reliable conversation gathering from your data.

Each conversation message should be associated with a certain actor, user or assistant. For instance, in a separate column within the same row, the message is contained:

```
user, "I want to book a flight"bot, "Where would you like to fly to?",user, "I am hungry, I want to order the food."bot, "What would you like to eat?"
```

As a prefix of a message:

```
"user: I want to book a flight""bot: Where would you like to fly to?","user: I am hungry, I want to order the food.""bot: "What would you like to eat?"
```

Alternatively, any other method that suits the use case can be utilized.

#### Data Conversion[​](https://rasa.com/docs/reference/testing/test-case-conversion/#data-conversion 'Direct link to Data Conversion')

Each sample conversation is sent concurrently to a Large Language Model (LLM) making the execution time relatively short. The LLM generates the YAML test cases based on the conversational data provided. Please see our LLM cost and performance analysis conducted using this feature.

#### Costs and Performance[​](https://rasa.com/docs/reference/testing/test-case-conversion/#costs-and-performance 'Direct link to Costs and Performance')

The `GPT-4o-mini` generally performs well, generating accurate test cases for most conversations. However, it may occasionally struggle with more complex test cases. On the other hand, `GPT-4o` consistently delivers correct test cases across all conversations, demonstrating high accuracy and reliability. In terms of costs, a typical conversation uses approximately 500 input tokens and 300 output tokens. This translates to a per-conversation cost of around $0.0003 with GPT-40-mini and $0.0070 with GPT-4o.

#### Storage of Test Cases[​](https://rasa.com/docs/reference/testing/test-case-conversion/#storage-of-test-cases 'Direct link to Storage of Test Cases')

The generated test cases are written into a YAML file. If the output path is not specified using the CLI argument, the default directory `e2e_tests` will be used. It is important to note that if the output path is specified, it must be a relative path pointing to a location inside the assistant project.

#### LLM Configuration[​](https://rasa.com/docs/reference/testing/test-case-conversion/#llm-configuration 'Direct link to LLM Configuration')

By default, the LLM model is configured to use the OpenAI `gpt-4.1-mini-2025-04-14`. If you want to use a different model, you can configure the LLM model in the `conftest.yml` file which can be discoverable by Rasa as long as it is in the root directory of your assistant project.

LLM can be configured based on the preferred provider:

- **OpenAI**
 
 ```
    llm_e2e_test_conversion:  provider: openai  model: ...
    ```
 
- **Azure**
 
 ```
    llm_e2e_test_conversion:  provider: azure  deployment: ...  api_base: ...
    ```
 
- **Any other LLM provider**
 
 ```
    llm_e2e_test_conversion:  model: ...  provider: ...
    ```
 

### Example Usage[​](https://rasa.com/docs/reference/testing/test-case-conversion/#example-usage 'Direct link to Example Usage')

To convert your sample conversational data into E2E test cases, run the following command:

```
rasa data convert e2e /path/to/your/conversations.csv -o /path/to/output
```

Or for an XLSX file with a specific sheet:

```
rasa data convert e2e /path/to/your/conversations.xlsx --sheet-name "Sheet1" -o /path/to/output
```

### Example Test Case Generation[​](https://rasa.com/docs/reference/testing/test-case-conversion/#example-test-case-generation 'Direct link to Example Test Case Generation')

Given a CSV file with sample conversations:

```
user, "I want to book a flight"bot, "Where would you like to fly to?"user, "New York"bot, "When would you like to travel?"user, "Tomorrow"bot, "Please wait while I check the available flights."...
```

After running the conversion command, an example YAML test case would look like this:

```
test_cases:- test_case: flight_booking  steps:    - user: "I want to book a flight"      assertions:        - bot_uttered:            text_matches: "Where would you like to fly to?"    - user: "New York"      assertions:        - bot_uttered:            text_matches: "When would you like to travel?"    - user: "Tomorrow"      assertions:        - bot_uttered:            text_matches: "Please wait while I check the available flights."
```

### Beta Feedback Questions[​](https://rasa.com/docs/reference/testing/test-case-conversion/#beta-feedback-questions 'Direct link to Beta Feedback Questions')

Among the questions we would like to get feedback on are:

- How valuable did you find the conversion of sample conversations into E2E test cases for your development workflow?
- Was the input format (CSV/XLS/XLSX) for sample conversations suitable for your needs?
- Did you encounter any difficulties when preparing your conversational data for conversion?
- How could we improve the usability of the CLI command and its arguments?
- How satisfied are you with the performance and accuracy of the generated YAML test cases by the LLM?
- Did you find the default `gpt-4o-mini` model adequate for your needs?
- Did you find the process of configuring the LLM model in the `conftest.yml` file straightforward?
- How useful are the generated YAML test cases in ensuring the reliability of your Rasa assistant?
- What challenges did you face when using the E2E test conversion feature?
- What additional features or improvements would make the E2E test conversion more effective for you?

- [End-To-End Test Conversion](https://rasa.com/docs/reference/testing/test-case-conversion/#end-to-end-test-conversion)
 - [Benefits of End-to-End Test Conversion](https://rasa.com/docs/reference/testing/test-case-conversion/#benefits-of-end-to-end-test-conversion)
 - [How It Works](https://rasa.com/docs/reference/testing/test-case-conversion/#how-it-works)
 - [Example Usage](https://rasa.com/docs/reference/testing/test-case-conversion/#example-usage)
 - [Example Test Case Generation](https://rasa.com/docs/reference/testing/test-case-conversion/#example-test-case-generation)
 - [Beta Feedback Questions](https://rasa.com/docs/reference/testing/test-case-conversion/#beta-feedback-questions)