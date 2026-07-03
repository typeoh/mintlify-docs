# Source: https://rasa.com/docs/reference/changelogs/compatibility-matrix

On this page

## Platform Feature Availability[​](https://rasa.com/docs/reference/changelogs/compatibility-matrix/#platform-feature-availability 'Direct link to Platform Feature Availability')

This table describes the ability of features across the platform.

| **Features** | **Studio** | **Pro** | **Comment** |
| --- | --- | --- | --- |
| Flow: slot persistence | ✅ | ✅ | |
| Flow: NLU trigger | ✅ | ✅ | |
| Flow: flow guard | ✅ | ✅ | |
| Flow: flow guard context namespace | ❌ | ✅ | |
| Flow: call step | ✅ | ✅ | |
| Flow: set slots | ✅ | ✅ | |
| Flow: button payload | ✅ | ✅ | |
| Flow: Next | ✅ | ✅ | |
| Flow: Noop | ❌ | ✅ | |
| Flow: Slot comparisons | ❌ | ✅ | |
| Flow: Nested logics | ✅ | ✅ | |
| Flow: action trigger search | ❌ | ✅ | |
| System flow/ Repair patterns: pattern\_user\_silence | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_collect\_info | ❌ | ✅ | |
| System flow/ Repair patterns: pattern\_continue | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_correction | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_cancel\_flow | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_skip\_question | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_chitchat | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_completed | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_clarification | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_internal\_error | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_cannot\_handle | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_human\_handoff | ✅ | ✅ | |
| System flow/ Repair patterns: pattern\_customer\_satisfaction | ✅ | ✅ | |
| System flow response editing | ✅ | ✅ | |
| Multi-LLM Routing | ✅ | ✅ | |
| Slot Custom | ❌ | ✅ | |
| Slot List | ❌ | ✅ | |
| Slot Boolean | ✅ | ✅ | |
| Slot Categorical | ✅ | ✅ | |
| Slot Float | ✅ | ✅ | |
| Slot Any | ✅ | ✅ | |
| Slot mapping | ✅ | ✅ | |
| **Advanced Slot mapping** | ❌ | ✅ | |
| Conditional responses variations | ✅ | ✅ | |
| Custom responses | ✅ | ✅ | |
| Images support in responses | ❌ | ✅ | |
| Assistant Domain | ✅ | ✅ | |
| Assistant Endpoints | ✅ | ✅ | |
| Assistant Credentials | ❌ | ✅ | |
| Assistant Configuration | ✅ | ✅ | |
| Assistant Language settings | ✅ | ✅ | |
| Inspector | ✅ | ✅ | |
| Inspector: model metadata (token, consumption) | ✅ | ❌ | |
| Inspector: Replay | ✅ | ❌ | |
| Inspector: Restart | ✅ | ✅ | |
| Mock custom actions | ❌ | ✅ | |
| E2E testing: generate | ✅ | ✅ | |
| E2E testing: Run | ❌ | ✅ | |
| Custom Action | ✅ | ✅ | Studio can read but not create Custom Action |
| Enterprise Search (RAG) config | ✅ | ✅ | |
| Enterprise Search (RAG): Custom Info Retrievers | ❌ | ✅ | |
| Enterprise Search (RAG): Vector Store Connectors | ❌ | ✅ | |
| Enterprise Search (RAG): Extractive Search | ❌ | ✅ | |
| Custom Components | ❌ | ✅ | |
| Command generator prompt editing | ✅ | ✅ | |
| Rephraser prompt editing | ✅ | ✅ | |
| Rephraser prompt editing per response | ✅ | ✅ | |
| Enterprise Search (RAG) prompt adjust | ✅ | ✅ | |
| Consistent project folder structure | ❌ | ✅ | |
| Conversation review tagging API | ✅ | ❌ | |
| Flow builder | ✅ | ✅ | Pro users use IDE |
| Conversation review | ✅ | ❌ | |
| State management | ✅ | ✅ | Pro users use Git |
| Flow history | ✅ | ✅ | Pro users use Git |
| Model management | ✅ | ✅ | Pro users use Git |
| CMS: Intents | ✅ | ✅ | Pro users use IDE |
| CMS: Entities | ✅ | ✅ | Pro users use IDE |
| CMS: Responses | ✅ | ✅ | Pro users use IDE |
| CMS: Slots | ✅ | ✅ | Pro users use IDE |
| CMS: Buttons & Links | ✅ | ✅ | Pro users use IDE |
| Fine tune recipe | ❌ | ✅ | |
| Custom Information Retrievers | ❌ | ✅ | |
| Channel connectors | ❌ | ✅ | |

## Rasa Pro Services compatibility[​](https://rasa.com/docs/reference/changelogs/compatibility-matrix/#rasa-pro-services-compatibility 'Direct link to Rasa Pro Services compatibility')

When choosing a version of Rasa Pro Services to deploy, you can refer to the table below to see which one is compatible with your installation of Rasa Pro.

Rasa Pro Services version is independent of Rasa Pro version, except that they share the same major version number.

| Rasa Pro Services | Rasa Pro |
| --- | --- |
| 3.0.x | 3.3.x, 3.4.x, 3.5.x |
| 3.1.x | 3.6.x |
| 3.2.x | 3.7.x, 3.8.x |
| 3.3.x | 3.8.x, 3.9.x, 3.10.x |
| 3.4.x | 3.8.x, 3.9.x, 3.10.x, 3.11.x, 3.12.x |
| 3.5.x | 3.8.x, 3.9.x, 3.10.x, 3.11.x, 3.12.x, 3.13.x |
| 3.6.x | 3.8.x, 3.9.x, 3.10.x, 3.11.x, 3.12.x, 3.13.x, 3.14.x |
| 3.7.x | 3.8.x, 3.9.x, 3.10.x, 3.11.x, 3.12.x, 3.13.x, 3.14.x, 3.15.x |
| 3.8.x | 3.14.x, 3.15.x, 3.16.x |

## Studio/Pro Versions Compatibility[​](https://rasa.com/docs/reference/changelogs/compatibility-matrix/#studiopro-versions-compatibility 'Direct link to Studio/Pro Versions Compatibility')

Rasa Studio uses Rasa Pro to train and run models. The following table shows the compatibility between Rasa Studio and Rasa Pro.

| Rasa Studio | Rasa Pro |
| --- | --- |
| 1.0.x, 1.1.x | 3.7.x |
| 1.2.x | 3.8.x |
| 1.3.0 | 3.8.4 |
| 1.3.1, 1.4.x, 1.5.x | 3.8.7 |
| 1.6.x | 3.9.9 |
| 1.7.x, 1.8.x, 1.9.x | 3.10.x |
| 1.10.x | 3.11.x |
| 1.11.x | 3.11.x |
| 1.12.x | 3.12.x |
| 1.13.x | 3.13.x |
| 1.14.1, 1.14.2, 1.14.3 | 3.13.x |
| 1.14.4 & above | 3.14.x |
| 1.15.x | 3.15.x |
| 1.16.x | 3.16.x |

- [Platform Feature Availability](https://rasa.com/docs/reference/changelogs/compatibility-matrix/#platform-feature-availability)
- [Rasa Pro Services compatibility](https://rasa.com/docs/reference/changelogs/compatibility-matrix/#rasa-pro-services-compatibility)
- [Studio/Pro Versions Compatibility](https://rasa.com/docs/reference/changelogs/compatibility-matrix/#studiopro-versions-compatibility)