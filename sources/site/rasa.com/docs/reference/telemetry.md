# Source: https://rasa.com/docs/reference/telemetry

On this page

Rasa utilizes telemetry to gather usage data, helping us continuously improve Rasa Pro's performance, reliability, and feature set for all users. This data allows our team to make informed decisions about the product roadmap and enhance the overall experience for organizations using Rasa.

When you run Rasa for the first time, you'll be notified about telemetry reporting and will have the option to disable it if you prefer. Before making a decision, let us explain the value of telemetry. It provides essential insights that help us continually improve product performance. However, we want to stress that data collection is fully configurable and remains under your control. We hope this transparency reassures you, as telemetry allows us to collaborate more effectively and mutually enhance the product experience.

## Why do we use telemetry reporting?[​](https://rasa.com/docs/reference/telemetry/#why-do-we-use-telemetry-reporting 'Direct link to Why do we use telemetry reporting?')

Telemetry data enables us to improve Rasa based on real-world usage, ensuring that it meets the operational needs of organizations and continues to evolve in line with industry demands. This helps us design future features and prioritize current work, so you can benefit directly from our reporting.

So how will we use the reported telemetry data? Here are some examples of what we use the data for:

- **Feature Optimization and Development**: By understanding which languages, pipelines, and policies are most frequently used, we can prioritize research and development in areas that will provide the greatest benefit to our enterprise clients. This helps us improve text, voice and dialogue handling, ensuring Rasa supports the most relevant features for your business needs.
- **Performance Testing and Scalability**: Telemetry data on dataset sizes and structure (e.g., the number of intents or actions) allows us to better test Rasa under various conditions. This ensures Rasa performs optimally across different types of enterprise applications, from small-scale deployments to large, complex systems.
- **Error Identification and Resolution**: Insights into common error patterns (e.g., during initialization or training) enable us to improve Rasa's stability and reliability. This helps us address recurring issues more effectively, reducing operational friction for your team.

We do not use the data collected for other purposes than to improve our products and services. For instance, we will not use this data for marketing purposes.

## What about privacy and sensitive data?[​](https://rasa.com/docs/reference/telemetry/#what-about-privacy-and-sensitive-data 'Direct link to What about privacy and sensitive data?')

We designed our telemetry to ensure that we collect only the data necessary to help us optimize our products and services, identify issues, and improve the overall user experience.

### No Data Sharing with Third Parties[​](https://rasa.com/docs/reference/telemetry/#no-data-sharing-with-third-parties 'Direct link to No Data Sharing with Third Parties')

Regardless of whether telemetry is enabled or disabled, Rasa does not share telemetry data with third parties. Your organization's data remains confidential and is used solely for the purposes of improving our Pro and providing better support.

### Data Collection[​](https://rasa.com/docs/reference/telemetry/#data-collection 'Direct link to Data Collection')

The data points aggregated for each subscription include usage details, command invocations, performance measures and errors.

Here are some data points that are collected:

- Event Type: Information about actions performed in Rasa (e.g., "Training Started", "Error Encountered").
- System Information: Information about the environment in which Rasa is running, such as the operating system, number of CPUs/GPUs, and whether the system is running in a cloud or containerized environment.
- Versions: Information on the versions of Rasa and Python in use, which helps us ensure compatibility and performance across different setups.
- Licence Information: A hash of your organization's license key and the name of the company associated with the license.
- Machine ID: A UUID that is stored locally and sent as part of the telemetry data to uniquely identify the environment where Rasa is deployed.

The contextual information about the use of Rasa, the environment in which it's deployed and the licences applicable help us identify trends about how Rasa is used across various environments, and diagnose and resolve problems more effectively. However, it's important to understand that sensitive data (see examples below) never leaves your machine.

We do not report on:

- Your training data
- Any messages your assistant receives or sends
- The techniques that you used to improve your assistant

We also ensure that we have proper security measures in place: Data Encryption: All telemetry data transmitted to Rasa servers is encrypted using industry-standard protocols (e.g., HTTPS), ensuring that data is secure in transit.

Minimal Data Collection: We limit telemetry data collection to what is strictly necessary for the purposes explained in this document. No data from your assistant's interactions, conversations, or training data is collected or transmitted. Access Control and Internal Handling: Access to telemetry data is restricted to authorized personnel within Rasa who need it to improve Rasa and resolve issues. We enforce strict access controls to ensure that data is handled appropriately and securely. For more information about how we process personal data, click here to read the privacy notice applicable to our commercial products and services.

For more information about how we process personal data, [click here](https://rasa.com/product-privacy/) to read the privacy notice applicable to our commercial products and services.

#### Inspecting Telemetry Data[​](https://rasa.com/docs/reference/telemetry/#inspecting-telemetry-data 'Direct link to Inspecting Telemetry Data')

We believe in full transparency. If you prefer to inspect the telemetry data before deciding whether to opt out, you can enable telemetry debug mode. This will allow you to view all the data that would be transmitted without actually sending it to our servers:

```
RASA_TELEMETRY_DEBUG=true rasa train
```

This logs telemetry data locally, so you can review exactly what is collected, including details like system information, event types, and machine identifiers, ensuring full visibility into the process.

Here is an example report that shows the data reported to Rasa after running `rasa train`:

```
{  "userId": "38d23c36c9be443281196080fcdd707d",  "event": "Training Started",  "properties": {    "language": "en",    "training_id": "311f4dfbbea64c3592f7d626bb169e36",    "type": "rasa",    "pipeline": [      {"name": "KeywordIntentClassifier"},      {"name": "NLUCommandAdapter"},      {"name": "CompactLLMCommandGenerator",       "llm": {         "model_name": "gpt-5.1-2025-11-13",         "request_timeout": 7         }       }     ],    "policies": [      {"name": "rasa.core.policies.flow_policy.FlowPolicy"},      {"name": "rasa.core.policies.intentless_policy.IntentlessPolicy"}    ],    "train_schema": "None",    "predict_schema": "None",    "num_intent_examples": 14,    "num_entity_examples": 0,    "num_actions": 171,    "num_templates": 113,    "num_conditional_response_variations": 3,    "num_slot_mappings": 45,    "num_custom_slot_mappings": 45,    "num_conditional_slot_mappings": 0,    "num_slots": 48,    "num_forms": 0,    "num_intents": 8,    "num_entities": 0,    "num_story_steps": 6,    "num_lookup_tables": 0,    "num_synonyms": 0,    "num_regexes": 0,    "is_finetuning": false,    "recipe": "default.v1",    "num_flows": 25,    "num_flows_with_nlu_trigger": 1,    "num_flows_with_flow_guards": 0,    "num_flows_with_not_startable_flow_guards": 0,    "num_collect_steps": 31,    "num_collect_steps_with_separate_utter": 1,    "num_collect_steps_with_rejections": 1,    "num_collect_steps_with_not_reset_after_flow_ends": 1,    "num_set_slot_steps": 1,    "max_depth_of_if_construct": 2,    "num_call_steps": 3,    "num_link_steps": 1,    "num_shared_slots_between_flows": 0,    "llm_command_generator_model_name": "gpt-4-0613",    "llm_command_generator_custom_prompt_used": false,    "multi_step_llm_command_generator_custom_handle_flows_prompt_used": false,    "multi_step_llm_command_generator_custom_fill_slots_prompt_used": false,    "flow_retrieval_enabled": true,    "flow_retrieval_embedding_model_name": "text-embedding-3-large",    "agents": {      "usage": [        {          "flow": "car_shopping",          "agent": "shopping_agent"        },        {          "flow": "schedule_new_appointment",          "agent": "appointment_selector",          "exit_if": [            "slots.selected_appointment_slot is not null"          ]        },        {          "flow": "schedule_new_appointment",          "mcp_tool": "book_appointment",          "mcp_server": "appointment_booking",          "mapping": {            "input": [              {                "param": "appointment_slot",                "slot": "selected_appointment_slot"              }            ],            "output": [              {                "slot": "appointment_confirmed",                "value": "result.structuredContent.appointment_confirmed"              }            ]          }        }      ],      "mcp_servers": [        {          "name": "appointment_booking",          "url": "http://localhost:8002/mcp",          "type": "http",          "additional_params": {}        }      ],      "agents": [        {          "shopping_agent": {            "agent": {              "name": "shopping_agent",              "protocol": "A2A",              "description": "Helps users shop for cars by connecting them with dealers and facilitating purchases"            },            "configuration": {              "module": "custom.car_shopping_agent.CarShoppingAgent",              "agent_card": "./sub_agents/shopping_agent/agent_card.json"            }          }        },        {          "appointment_selector": {            "agent": {              "name": "appointment_selector",              "protocol": "RASA",              "description": "Helps users select an appointment slot for seeing a car dealer."            },            "configuration": {              "prompt_template": "./sub_agents/appointment_selector/prompt_template.jinja2",              "module": "custom.appointment_booking_agent.AppointmentBookingAgent"            },            "connections": {              "mcp_servers": [                {                  "name": "appointment_booking",                  "exclude_tools": [                    "book_appointment"                  ]                }              ]            }          }        },      ]    },    "metrics_id": "36e8e5e43fef4429a2a01ad239d0081d"  },  "context": {    "os": {      "name": "Darwin",      "version": "19.4.0"    },    "ci": false,    "project": "a0a7178e6e5f9e6484c5cfa3ea4497ffc0c96d0ad3f3ad8e9399a1edd88e3cf4",    "python": "3.7.5",    "rasa_pro": "3.8.0",    "cpu": 16,    "docker": false,    "license_hash": "t1a7170e6e5f9e6484c5cfa3ea4497ffc0c96a0ad3f3ad8e9399adadd88e3cf5",    "company": "Rasa"  }}
```

## How to opt-out[​](https://rasa.com/docs/reference/telemetry/#how-to-opt-out 'Direct link to How to opt-out')

Rasa Developer Edition License

For users of the [Developer Edition License](https://rasa.com/rasa-pro-developer-edition-license-key-request/), telemetry can't be disabled. Please refer to the license [terms](https://rasa.com/developer-terms) for more information.

We understand that some organizations may prefer to disable telemetry data collection. You can opt out of telemetry reporting at any time without impacting the core functionality of Rasa. Disabling telemetry ensures that no data will be sent to Rasa, but you will still retain full access to Rasa's features and performance.

To opt out of telemetry, use one of the following methods:

### Opt out with the Command Line[​](https://rasa.com/docs/reference/telemetry/#opt-out-with-the-command-line 'Direct link to Opt out with the Command Line')

You can disable telemetry directly through the command line by running the following command:

```
rasa telemetry disable
```

This command immediately stops all telemetry reporting and prevents any further data from being sent to Rasa.

### Opt out with environment variables[​](https://rasa.com/docs/reference/telemetry/#opt-out-with-environment-variables 'Direct link to Opt out with environment variables')

Alternatively, you can disable telemetry by setting the `RASA_TELEMETRY_ENABLED` environment variable. This approach allows you to manage telemetry through your system's configuration settings:

```
export RASA_TELEMETRY_ENABLED=false
```

When you run Rasa for the first time, you will be notified about telemetry collection and provided with an option to disable it during the initial setup.

### Impact of opting out[​](https://rasa.com/docs/reference/telemetry/#impact-of-opting-out 'Direct link to Impact of opting out')

By opting out of telemetry, your organization will no longer contribute usage data to help improve Rasa. However, this will not affect your organization's ability to use Rasa fully, and all core features will continue to function as expected. Opting out may limit our ability to provide tailored support and optimization based on your specific deployment environment.

### Re-Enable Telemetry[​](https://rasa.com/docs/reference/telemetry/#re-enable-telemetry 'Direct link to Re-Enable Telemetry')

If you change your mind and wish to enable telemetry reporting again, you can do so by running the following command:

```
rasa telemetry enable
```

- [Why do we use telemetry reporting?](https://rasa.com/docs/reference/telemetry/#why-do-we-use-telemetry-reporting)
- [What about privacy and sensitive data?](https://rasa.com/docs/reference/telemetry/#what-about-privacy-and-sensitive-data)
 - [No Data Sharing with Third Parties](https://rasa.com/docs/reference/telemetry/#no-data-sharing-with-third-parties)
 - [Data Collection](https://rasa.com/docs/reference/telemetry/#data-collection)
- [How to opt-out](https://rasa.com/docs/reference/telemetry/#how-to-opt-out)
 - [Opt out with the Command Line](https://rasa.com/docs/reference/telemetry/#opt-out-with-the-command-line)
 - [Opt out with environment variables](https://rasa.com/docs/reference/telemetry/#opt-out-with-environment-variables)
 - [Impact of opting out](https://rasa.com/docs/reference/telemetry/#impact-of-opting-out)
 - [Re-Enable Telemetry](https://rasa.com/docs/reference/telemetry/#re-enable-telemetry)