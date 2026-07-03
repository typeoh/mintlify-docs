# Source: https://www.rasa.com/docs/reference/changelogs/studio-changelog

All notable changes to Rasa Studio will be documented in this page.

## Important Notice: Database Migration Changes in Studio v1.13.x+[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#important-notice-database-migration-changes-in-studio-v113x 'Direct link to Important Notice: Database Migration Changes in Studio v1.13.x+')

caution

Starting with version `1.13.x`, Studio supports deployments using non-superuser database roles. If you're upgrading from an earlier version, manual steps are required before the upgrade can be completed successfully. Please refer to [Rasa Studio 1.12.x to Rasa Studio 1.13.x](https://www.rasa.com/docs/reference/changelogs/rasa-pro-migration-guide/#rasa-pro-311-to-rasa-pro-312) for full instructions. Skipping these steps will cause migration issues during deployment.

## \[1.16.2\] - 2026-05-25[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1162---2026-05-25 'Direct link to [1.16.2] - 2026-05-25')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes 'Direct link to Bugfixes')

- Fixed copy for the "Containment insights" table in Analytics: replaced "highest human handoff rates" with "highest containment rates".
- Fixed problem with RAG and subagent utterances not being displayed in Conversation View.

## \[1.16.1\] - 2026-05-19[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1161---2026-05-19 'Direct link to [1.16.1] - 2026-05-19')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-1 'Direct link to Bugfixes')

- Fixed all npm vulnerabilities

## \[1.16.0\] - 2026-04-07[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1160---2026-04-07 'Direct link to [1.16.0] - 2026-04-07')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features 'Direct link to Features')

- **Out of the box Analytics:** Rasa Studio is now equipped with Analytics to help you quickly understand how your agents are performing and identify areas for improvement.
 
 It currently covers the most requested metrics and includes common filters to help you drill down into your data.
 
 Both existing and new customers will benefit from it.
 
 ![image](https://www.rasa.com/docs/assets/images/analytics-overview-2ff9cc2ad8e4d87b481a0c02022e3b16.png)
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-volume-973c5d4b29115e2e5a3587334ee792a6.png)
 
 While we default to a sensible definition for each of the metrics, you still have the freedom to customize them if needed. Once the new criteria is saved, you’ll start seeing the results the next day.
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-volume0-f7739d1ae460e6166241c87fe19d8b2f.png)
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-volume1-6519168ee1ced965f71d6435c2dc7766.png)
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-volume2-aa16954eeecbabbc30fd1061bdef3b56.png)
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-volume3-bf7af3cff628d89dba8eff8f27a300c1.png)
 
 If you need to diagnose an issue, click the chart, then click view conversation to open the Conversation Review page with the filters preselected.
 
 ![image](https://www.rasa.com/docs/assets/images/select-conversations-b51a08d11b2d5c9436e83f94fe064507.png)
 
 You have existing BI tools? We have good news! You can access the Analytics data in your own dashboard. Read more in the [docs](https://www.rasa.com/docs/reference/api/analytics-data-structure-reference/).
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements 'Direct link to Improvements')

- **Conversation Review Improvements:** We've improved conversation review usability and enhanced existing features.
 
 We've added highly requested filters to the page, including filters by tag, language, and channel.
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-review-improvements-1-6260b035da0aef7658b878fbc6bdb033.png)
 
 When you apply these filters, they appear in the conversation list so you can easily access and refine them.
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-review-improvements-2-ca7e2d4693e31cc744708f98fbd0a84c.png)
 
 The conversation review page now supports read-only access so you can manage various accounts more safely and securely. Read more in the [docs](https://www.rasa.com/docs/studio/installation/setup-guides/authorization-guide/).
 
- **Session management:** Rasa now introduces session management, giving builders the tools to create ChatGPT-like conversation history experiences. Each user gets a stable `user_id` that ties their conversations together, while individual interaction windows get their own `session_id` — so end users can pick up where they left off or start fresh, and builders no longer have to hack around Rasa's single-tracker-per-sender model.
 
 - You can now link multiple conversations to the same person using a `user_id`, while each individual interaction window gets its own `session_id`.
 - Explicit lifecycle events mark when sessions start and end, and a new inactivity timer closes sessions proactively and persists across server restarts.
 
 ![image](https://www.rasa.com/docs/assets/images/session-management-1-d5e41e230bd1533a45edab4051616925.png)
 
 ![image](https://www.rasa.com/docs/assets/images/session-management-2-af17492b0de73466febc94f4f0b63572.png)
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-2 'Direct link to Bugfixes')

- Fixed **Copy conversation URL** on the conversation detail view so it copies a permalink to that conversation instead of the Conversation view list URL with the current filters.
- Fixed **Mark as reviewed** on the transcript viewer returning you to page 1 of Conversation view instead of the paginated list page you opened the conversation from.
- Fixed **Number of messages** on Conversation View counting the initial `/session_start` as a user message; the count now reflects user-authored messages only (sessions with no user input show 0).

## \[1.15.4\] - 2026-05-22[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1154---2026-05-22 'Direct link to [1.15.4] - 2026-05-22')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-3 'Direct link to Bugfixes')

- Fixed problem with RAG and subagent utterances not being displayed in Conversation View.

## \[1.15.3\] - 2026-03-09[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1153---2026-03-09 'Direct link to [1.15.3] - 2026-03-09')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-1 'Direct link to Improvements')

- Make model polling intervals configurable and optimize database transactions while polling.

## \[1.15.2\] - 2026-01-13[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1152---2026-01-13 'Direct link to [1.15.2] - 2026-01-13')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-4 'Direct link to Bugfixes')

- Fixed collision errors during Studio upload when default custom actions in `domain.yml` were defined.

## \[1.15.0\] - 2025-12-09[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1150---2025-12-09 'Direct link to [1.15.0] - 2025-12-09')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-1 'Direct link to Features')

- **Slots CMS:** Studio users can now create and manage their assistant slots from a centralised CMS. [Learn more about Slots here](https://www.rasa.com/docs/studio/build/content-management/slots/)
 
 ![image](https://www.rasa.com/docs/assets/images/slot-cms-14960251abd048fbb45ba7c013be2963.png)
 
- **Buttons & Links CMS:** Studio users can now create and manage the buttons and links used on their assistant responses from a centralised CMS. [Learn more about Buttons & Links here](https://www.rasa.com/docs/studio/build/content-management/buttons-and-links/)
 
 ![image](https://www.rasa.com/docs/assets/images/buttons-and-links-cms-6ad526fca95146fd9e5f1ec036fe4864.png)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-2 'Direct link to Improvements')

- **Conversation Review Improvements:** Studio users can now filter conversations on Conversation Review by channel and assistant response.
 
 ![image](https://www.rasa.com/docs/assets/images/conversation-review-filters-f5110125b0b8be18bf51d1df4074f178.png)
 

## \[1.14.7\] - 2025-03-09[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1147---2025-03-09 'Direct link to [1.14.7] - 2025-03-09')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-3 'Direct link to Improvements')

- Make model polling intervals configurable and optimize database transactions while polling.

## \[1.14.6\] - 2026-01-13[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1146---2026-01-13 'Direct link to [1.14.6] - 2026-01-13')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-5 'Direct link to Bugfixes')

- Fixed collision errors during Studio upload when default custom actions in `domain.yml` were defined.

## \[1.14.4\] - 2025-11-27[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1144---2025-11-27 'Direct link to [1.14.4] - 2025-11-27')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-4 'Direct link to Improvements')

- Add support for Rasa Pro 3.14.1

## \[1.14.3\] - 2025-11-24[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1143---2025-11-24 'Direct link to [1.14.3] - 2025-11-24')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-6 'Direct link to Bugfixes')

- Fixed an issue where user messages with date/time entities (Duckling) were failing to process correctly, causing errors in conversation handling.

## \[1.14.2\] - 2025-11-13[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1142---2025-11-13 'Direct link to [1.14.2] - 2025-11-13')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-5 'Direct link to Improvements')

- Improve the assistant upload time.

## \[1.14.1\] - 2025-10-28[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1141---2025-10-28 'Direct link to [1.14.1] - 2025-10-28')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-2 'Direct link to Features')

- **Supporting Controlled Slots:** Studio now supports controlled slots; these can be used to define slots that should be filled by a custom action, [response button payload](https://www.rasa.com/docs/reference/primitives/responses/#payload-syntax), or a [`set_slots` flow step](https://www.rasa.com/docs/reference/primitives/flow-steps/#set-slots).
 
 ![image](https://www.rasa.com/docs/assets/images/mapping-7aab001018d42160c594b9e98351ce86.jpg)
 
- **Response latency:** Response latency per turn on text and on voice with color coding with Rasa latency and TTS.
 
 ![image](https://www.rasa.com/docs/assets/images/latency-a2cf8cf4255626efb74415ed12b43bbf.jpg)
 
- **Support for IAM auth in AWS:** Studio now supports AWS IAM database authentication for enhanced security and simplified credential management.
 
 More details on how to set it up and impacts can be found [here](https://www.rasa.com/docs/reference/changelogs/studio-version-migration-guide/#rasa-studio-v113x--v114x)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-6 'Direct link to Improvements')

- **Deployment improvements:** The Rasa Studio deployment experience has undergone drastic improvements. We have released new Rasa Studio Deployment Playbooks, reducing the time and effort required to deploy Rasa Studio.
 
- **Upload license:** You can now upload your extended license from Studio.
 

## \[1.14.0.edge1\] - 2025-08-29[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1140edge1---2025-08-29 'Direct link to [1.14.0.edge1] - 2025-08-29')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-3 'Direct link to Features')

- **Streamlined UX for flow versioning:**
 
 - Flows can now be marked as Draft or Published to clearly communicate their status to your team.
 
 ![image](https://www.rasa.com/docs/assets/images/Status1-0c467b0c1591124be0eba8c981dda885.png)
 
 - You can add comments to flow versions to describe changes, making it easier to track progress and revert to a specific version.
 
 ![image](https://www.rasa.com/docs/assets/images/Status2-6a6d03eb3605009590d502c6f0074fb9.png)
 
 - The history log is now streamlined to highlight key changes and who made them, with the option to view a detailed breakdown when needed.
 
 ![image](https://www.rasa.com/docs/assets/images/Status3-72976d178989dfb06c8d9d39ca10862a.png)
 
- **Editing system responses:** You can now directly edit responses in Rasa system flows, making it easy to adjust how your assistant handles unexpected turns and to align the tone of voice with your brand.
 
 ![image](https://www.rasa.com/docs/assets/images/CMS-1-2e9e536c59397dd4eded7f0682ce5d92.jpg)
 
- **Keycloak upgrade:**
 
 - The Keycloak Docker image has been upgraded from version `23` to `26.3.2`
 - This update includes security patches, performance improvements, and API updates from the upstream release

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-7 'Direct link to Improvements')

- **Vulnerability fixes**
 - All Docker images are now upgraded from node `v20.11.0` to node `v.22.18.0`
 - Update all package dependencies to fix security vulnerabilities

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-7 'Direct link to Bugfixes')

- Silence timeout in Collect step shows an inaccurate value in the UI
- Voice Inspector does not hang up the call after executing the silence pattern

## \[1.13.8\] - 2025-12-09[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1138---2025-12-09 'Direct link to [1.13.8] - 2025-12-09')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-8 'Direct link to Bugfixes')

- Fix validation so Studio no longer blocks assistant uploads on incorrect persisted-slot warnings when training succeeds.

## \[1.13.7\] - 2025-11-24[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1137---2025-11-24 'Direct link to [1.13.7] - 2025-11-24')

### Improvement[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvement 'Direct link to Improvement')

- Introduce enhanced debug logging across the backend and Keycloak.

## \[1.13.6\] - 2025-10-31[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1136---2025-10-31 'Direct link to [1.13.6] - 2025-10-31')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-9 'Direct link to Bugfixes')

- Fixed event ingestion issue which prevented entities from `DucklingEntityExtractor` to be ingested

## \[1.13.5\] - 2025-10-16[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1135---2025-10-16 'Direct link to [1.13.5] - 2025-10-16')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-10 'Direct link to Bugfixes')

- Fixed a Prisma transaction handling issue that prevented assistants from being downloaded in Studio.

## \[1.13.4\] - 2025-09-25[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1134---2025-09-25 'Direct link to [1.13.4] - 2025-09-25')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-11 'Direct link to Bugfixes')

- Improved the previous flow node validation fix to ensure it continues working correctly after multiple training runs.

## \[1.13.3\] - 2025-09-15[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1133---2025-09-15 'Direct link to [1.13.3] - 2025-09-15')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-12 'Direct link to Bugfixes')

- Fixed an issue where training in Studio failed with an “invalid step” error unless the condition value was manually re-selected.

## \[1.13.2\] - 2025-08-25[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1132---2025-08-25 'Direct link to [1.13.2] - 2025-08-25')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-13 'Direct link to Bugfixes')

- Fixed all npm vulnerabilities

## \[1.13.1\] - 2025-07-28[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1131---2025-07-28 'Direct link to [1.13.1] - 2025-07-28')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-14 'Direct link to Bugfixes')

- Fixed prisma dependency installation script
- Fixes a timeout issue in viewing conversation in Studio with bigger datasets

## \[1.13.0\] - 2025-07-14[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1130---2025-07-14 'Direct link to [1.13.0] - 2025-07-14')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-4 'Direct link to Features')

- **Supporting Loops in Flows:** Studio now supports looping flows, making it easier to build conversations with more natural and flexible control. You can now:
 
 - **Loop back** to earlier steps in the same flow — useful for retries, clarifications, or repeated prompts.
 - **Loop forward** by connecting multiple branches to the same ending, simplifying flows that converge to a common result.
 
 [Learn more](https://www.rasa.com/docs/studio/build/flow-building/linking-flows/#connecting-steps-within-a-flow)
 
 ![image](https://www.rasa.com/docs/assets/images/nodeconnections-96f73a45779e2817a2776f2955b4f3b3.png)
 
- **Custom Responses:** You can now add and edit any custom response components — carousels, cards, date pickers, or anything else your chat widget supports — directly in Studio using the inline YAML editor.
 
 ![image](https://www.rasa.com/docs/assets/images/customresponses-a40eeb7d90c0661ceca5adeda75b1102.png)
 
 Custom response code will also be visible in the Inspector during testing, so you can review exactly what was sent during conversations.
 
 ![image](https://www.rasa.com/docs/assets/images/customresponses2-0fa8405a7986560791e7114f0e215312.png)
 
- **Custom Prompts:** This release introduces the ability to edit assistant prompts directly in the UI:
 
 - Fine-tune your assistant’s tone of voice by editing the global rephraser prompt and individual response prompts directly in Studio.
 
 - For more low-level control over your assistant’s behaviour, you can now edit Command Generator and Enterprise Search prompts.
 
 
 _**⚠️ Note:** Modifying the command generator prompt can significantly impact assistant behavior. We recommend validating changes with end-to-end testing before deploying to production._
 
 ![image](https://www.rasa.com/docs/assets/images/PrompEditing-bc56760404eb12b99b1f18901f8fa622.png)
 
- **Testing in Voice:** You can now test your assistant not just in chat, but also in voice — directly in Studio **using the Inspector view:**
 
 - Choose whether you want to test with chat or voice
 - Start a call with your assistant, grant microphone access, and begin speaking
 - All events and responses are transcribed and visible in the Inspector, just like with chat
 - It uses Rasa’s default voice setup (STT from Deepgram and TTS from Cartesia), so there’s nothing to configure — defaults work out of the box.
 - For now, voice input is limited to English.
 
 ⚠️ Only available if voice is included in your Rasa license
 
 [Learn more](https://www.rasa.com/docs/studio/test/try-your-assistant/#switching-between-chat-and-voice)
 
 ![image](https://www.rasa.com/docs/assets/images/voice-c475265a3d5322a2e1b7ef996aad236c.png)
 
- **Silence Handling for Voice Interactions:** You can now configure how long your assistant should wait for a user response during voice interactions before treating it as silence. Default is 7 seconds, you can change it in endpoints.yml:
 
 ![image](https://www.rasa.com/docs/assets/images/Silence-15a07a6d95edf524fdae68d06ebe8a33.png)
 
 Or override it per Collect step to fine-tune behavior for specific questions:
 
 ![image](https://www.rasa.com/docs/assets/images/Silence2-8e5c1f142a16e616822f6251a664801f.png)
 
 When the timeout is reached, the assistant triggers the `pattern_user_silence`, allowing you to define how it should respond to user silence — for example, by repeating the question or offering help.
 
 ⚠️ Only available if voice is included in your Rasa license
 
- **Supporting All Logical Operators and OR in Conditional Responses:** Studio now supports both AND and OR operators in conditional response variations, along with all comparison operators available in the Rasa Framework (e.g. `==`, `!=`, `>`, `<`). This brings more flexibility when defining when a response should be shown.
 
 ![image](https://www.rasa.com/docs/assets/images/operators-5cd56cfa22669b5aae8505b017eda92d.png)
 
- **Preventing Digressions:** You can now keep conversations on track while collecting multiple pieces of user input by turning on a setting that prevents users from switching flows until specific required slots are filled.
 
 ![image](https://www.rasa.com/docs/assets/images/Prevent_digressions-87734fb7e403f395d2f479eab4f2c9bc.png)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-8 'Direct link to Improvements')

- Allow deleting primitives and flows used in old assistant versions
 
- Allow nested Logic steps in flows
 
- Automatically open a flow on creation
 

## \[1.13.0.edge1\] - 2025-05-19[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1130edge1---2025-05-19 'Direct link to [1.13.0.edge1] - 2025-05-19')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-5 'Direct link to Features')

- **Duplicating flows:** Need to reuse existing flow logic? Now you can duplicate any flow in one click — no need to rebuild from scratch. A simple way to speed up editing and keep things consistent. [Learn more](https://www.rasa.com/docs/studio/build/content-management/manage-flows/#duplicating-flows)
 
 ![image](https://www.rasa.com/docs/assets/images/DuplicateFlow-51d814b022f9e374fd0e6c1080231084.png)
 
- **Reordering buttons and links in responses:** You can now drag and drop buttons and links directly in the response editor — giving you full control over the order in which options are shown to users. Learn more about [buttons](https://www.rasa.com/docs/studio/build/flow-building/collect/#reordering-buttons) and [links](https://www.rasa.com/docs/studio/build/flow-building/sending-messages/#reordering-links)
 
 ![image](https://www.rasa.com/docs/assets/images/ReorderButtons-ece6d67709280293bcab37b2e1da3988.png)
 
- **Upload & download improvements:** Granular upload and download of Assistant Settings is now available, enabling users to maintain consistent and synchronized `config.yml` and `endpoints.yaml` files across Studio and Pro.
 
 ![image](https://www.rasa.com/docs/assets/images/Configuration-68fddf06e0ac82c9965f47d730118766.jpg)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-9 'Direct link to Improvements')

- **Training panel Improvements:** Managing training just got easier
 
 1. Newly created flows are selected for training by default. If you adjust the selection, the assistant will remember your choice.
 2. Along with your flows, the panel now shows only _customized_ system flows — keeping things cleaner.
 3. Selecting a version is only required if you’re using flow versioning. [Learn More](https://www.rasa.com/docs/studio/test/train-your-assistant/#training-in-studio)
 
 ![image](https://www.rasa.com/docs/assets/images/TrainingPanel-a34f45facb4af10808348f67c551ac5b.png)
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-15 'Direct link to Bugfixes')

- Fixed incorrect error message shown when deleting intents with examples but no annotations.
- Fixed unexpected navigation from “Try your assistant” when clicking empty canvas areas in inspector mode.
- Fixed hidden initial values for boolean slots in slot configuration modal.
- Fixed issue where intent and entity dropdowns were empty when editing slot mappings.
- Other minor fixes for Studio trial improvement

## \[1.12.13\] - 2025-08-25[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#11213---2025-08-25 'Direct link to [1.12.13] - 2025-08-25')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-16 'Direct link to Bugfixes')

- Fixed all dependency vulnerabilities

## \[1.12.12\] - 2025-07-29[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#11212---2025-07-29 'Direct link to [1.12.12] - 2025-07-29')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-17 'Direct link to Bugfixes')

- Fixes a timeout issue in viewing conversation in Studio with bigger datasets

## \[1.12.10\] - 2025-06-27[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#11210---2025-06-27 'Direct link to [1.12.10] - 2025-06-27')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-18 'Direct link to Bugfixes')

- Fixed a bug on uploading condition on many categorical values

## \[1.12.9\] - 2025-06-17[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1129---2025-06-17 'Direct link to [1.12.9] - 2025-06-17')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-10 'Direct link to Improvements')

- Updated latest Rasa-pro version
- Other minor bugfixes

## \[1.12.8\] - 2025-06-03[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1128---2025-06-03 'Direct link to [1.12.8] - 2025-06-03')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-19 'Direct link to Bugfixes')

- Defaulted new assistants to use `CompactLLMCommandGenerator` and `gpt-4o-2024-11-20` model since the older gpt-4 model will soon be deprecated.
- Added selected language to test case metadata to ensure that the generated e2e tests can be run for specific language configuration.

## \[1.12.7\] - 2025-05-23[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1127---2025-05-23 'Direct link to [1.12.7] - 2025-05-23')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-20 'Direct link to Bugfixes')

- Fixed a bug where re-adding a language caused duplicates and broken translations.

## \[1.12.5\] - 2025-05-15[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1125---2025-05-15 'Direct link to [1.12.5] - 2025-05-15')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-21 'Direct link to Bugfixes')

- Fixed a crash on the Versions page caused by a Rust-to-NAPI string conversion error.
- Fixed an issue causing the Flows page to render blank due to unexpected local storage state.

## \[1.12.4\] - 2025-05-08[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1124---2025-05-08 'Direct link to [1.12.4] - 2025-05-08')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-22 'Direct link to Bugfixes')

- Fixed an issue due to duplication of button translations during import.
- Fixed an issue related to incorrect parsing of slot conditions during import.
- Removed unwanted validation of model names during import.
- `pattern completed` and `session start` system flows are now displayed in `Inspector mode`.
- Other minor bug fixes.

## \[1.12.3\] - 2025-04-29[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1123---2025-04-29 'Direct link to [1.12.3] - 2025-04-29')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-11 'Direct link to Improvements')

- Users can download selected flows from the Flows page.
 
 ![image](https://www.rasa.com/docs/assets/images/download-flows-78d9054bc89c22693e3da92f59d5ca28.png)
 
- New non-empty flows are automatically selected for training.
 
- Users are directed to the `Try your assistant` page when selecting an assistant project from the welcome page.
 
- Users can edit custom languages in the assistant settings.
 
- Multiple UX improvements to the CMS page.
 
- Renamed `Custom Flows` to `My Flows`.
 
- Updated T&C links.
 

## \[1.12.2\] - 2025-04-14[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1122---2025-04-14 'Direct link to [1.12.2] - 2025-04-14')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-23 'Direct link to Bugfixes')

- Fixed CVE-2025-32812
- Fixed an issue where duplicate links could be added in the 'Create response' modal.
- Fixed a bug where localized button payload settings were not saved correctly.

## \[1.12.1\] - 2025-04-08[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1121---2025-04-08 'Direct link to [1.12.1] - 2025-04-08')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-24 'Direct link to Bugfixes')

- Fixed an issue where the Flows page rendered blank due to stale data in local storage.

## \[1.12.0\] - 2025-04-08[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1120---2025-04-08 'Direct link to [1.12.0] - 2025-04-08')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-6 'Direct link to Features')

- **Reusable Links and Buttons in Responses:**
 
 1. Native link support is now available within responses. You can add links to external URLs, within your application, or phone/email links.
 
 [Learn More about links in responses](https://www.rasa.com/docs/studio/build/flow-building/sending-messages/#links)
 
 ![image](https://www.rasa.com/docs/assets/images/LInks-dde893858d0149eb8b1b2fcd04490b62.jpg)
 
 2. Links and buttons can now be reused within a project. Create a button or link once, and simply select it in another response for reuse. ![image](https://www.rasa.com/docs/assets/images/ButtonReusability-3d457584863d66cf70206fe7902b80a5.jpg)
 
- **Integrated Multi-Language Support:** This release introduces multi-language support natively in Rasa to make multilingual assistants easier to manage and scale. With this update, you can now configure multiple languages directly within your assistant’s settings. This means you can pre-translate responses, buttons, links, and even flow names, ensuring a consistent and brand-aligned experience across different languages.
 
 **Language Configuration:** Define your assistant’s default language and add additional languages in the `config.yml` file or through the Assistant Settings in Studio. ![image](https://www.rasa.com/docs/assets/images/assistant-language-settings-fd2e3def8ae9904966a90cae35ff78cb.png)
 
 **Content Translation:** Manage your translations for responses, buttons, links, and flow names. ![image](https://www.rasa.com/docs/assets/images/content-translation-ad8b25a943ec6c71271d3d47af2addc3.png)
 
 **Try your assistant in all your supported languages:** Test out your assistant in different languages to ensure that your user experience is seamless for any market. ![image](https://www.rasa.com/docs/assets/images/TYA-languages-c58c0769591f35cd80f705214a2d7bb2.png)
 
- **Train & Test in Isolation :** With this release, customers can create and include different versions of flows in training, allowing them to test the model in isolation without disrupting others' work.
 
 1. **Save stable flow versions for training :** store reliable versions of flows that teams can use consistently for training and testing. ![image](https://www.rasa.com/docs/assets/images/Save-stable-flow-version-c99733b9502377555fc4079978b8609a.jpg)
 
 2. **Train assistant with specific flow versions :** choose between stable or current versions of a flow when training a model, and create a dedicated model version. ![image](https://www.rasa.com/docs/assets/images/Train-with-specific-flow-versions-266942ac7351c81e93ae328b67627b6a.jpg)
 
 3. **Test in isolation :** test newly created assistant versions without affecting others’ testing. ![image](https://www.rasa.com/docs/assets/images/Test-in-isolation-e68f6e8e3d554e8bcf5b3e67039f4e2b.jpg)
 
 4. **Assistant version management :** manage multiple assistant versions, test them anytime, and update their name and description as needed. ![image](https://www.rasa.com/docs/assets/images/Manage-assistant-versions-0d2c1368b2aed755f3726408db454e53.jpg)
 
- **Persisted Slot Values :** You now have more control over the bot’s memory. Choose whether to persist or clear a specific slot value after the flow ends. ![image](https://www.rasa.com/docs/assets/images/Persist-slots-1d544f74e39b440fb85a337e5fde3847.png)
 
- **Welcome Page for Improved Studio Navigation :** Studio now has a homepage, so you could navigate between projects and explore educational materials. ![image](https://www.rasa.com/docs/assets/images/Homepage-88a5efe889a426b0df89b4dd9267a788.jpg)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-12 'Direct link to Improvements')

- **Form to Leave Feedback About the Product** You can now share your thoughts and insights about Studio using a dedicated Feedback button in the menu.
 
 ![image](https://www.rasa.com/docs/assets/images/Feedback-569f72458161b45a5a46dfc592f0a9c3.jpg)
 
- **Documentation Links for Contextual Help** Rasa now has new documentation! To make it easier to discover, we’ve added contextual links to it in various places across the UI where help might be needed.
 
 ![image](https://www.rasa.com/docs/assets/images/linksdocs-30246c385a29775a8de0d93404df2341.jpg)
 
- **Navigation Enhancements**
 
 1. In-Flow Search: Quickly find and jump to specific nodes within your flow, reducing time spent scrolling and faster editing of complex assistants. ![image](https://www.rasa.com/docs/assets/images/inflow-search-5d59d78e176ca017b891cdca68da53a7.png)
 
 2. View Linked Flows in CMS: You can now preview which flows are using a response and jump to the response in the flow builder directly from the CMS ![image](https://www.rasa.com/docs/assets/images/linked-flows-in-cms-1f7f8267686b08bdcd30eb3171ed5a43.png)
 
- **Enable Delete Assistant :** Users can now delete an assistant.
 
 ![image](https://www.rasa.com/docs/assets/images/delete-assistant-4f6aac550fd4017f141787b4aefdafab.png)
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-25 'Direct link to Bugfixes')

- Other minor bugfixes

## \[1.12.0.dev2\] - 2025-02-20[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1120dev2---2025-02-20 'Direct link to [1.12.0.dev2] - 2025-02-20')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-26 'Direct link to Bugfixes')

- Fixed an issue around flow version validation where conditions linked to a slot now apply only to draft flows, preventing validation issues caused by historical flow versions.

## \[1.12.0.dev1\] - 2025-02-17[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1120dev1---2025-02-17 'Direct link to [1.12.0.dev1] - 2025-02-17')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-7 'Direct link to Features')

- **Inspector mode:** Studio now includes Inspector Mode—a detailed debugging view that displays flow events, slot updates, and response details in real time. Users can easily switch between a simplified view and Inspector Mode to track their assistant’s behavior and resolve issues without leaving Studio.

[Learn more about Inspector mode](https://www.rasa.com/docs/studio/test/try-your-assistant/)

![image](https://www.rasa.com/docs/assets/images/inspector-mode-61c3eee94bdd9b2edbe1ff3a52c2e5b0.png)

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-13 'Direct link to Improvements')

- **Support OR operator between logical conditions**: Users can now use the OR operator in conditions throughout Studio. This enhancement brings Studio closer to the Rasa Framework in functionality, giving you more flexibility in designing your flows. ![image](https://www.rasa.com/docs/assets/images/or-operator-support-45a5751bad8b36c8f1381c3c970b065d.png)
- **Improved navigation between invalid nodes in a flow**: Users can now quickly identify and fix validation errors, even in large flows. ![image](https://www.rasa.com/docs/assets/images/invalid-nodes-navigation-3ec9e750c7c223c1836d8cb788d06bb0.png)
- **System Flow Reset Improvement**: Resetting system flows no longer clears their change log, ensuring the flow history remains intact.
- **Improved Flow List Ordering**: The Flow list is now ordered based on the last update made by users, ensuring it reflects direct user actions rather than system-driven changes.
- **Introduction of Superuser Group**: A new Superuser user group has been introduced. Members of this group have full access to all features provided by Studio.

## \[1.11.4\] - 2025-07-24[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1114---2025-07-24 'Direct link to [1.11.4] - 2025-07-24')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-27 'Direct link to Bugfixes')

- Fixes a timeout issue in viewing conversation in Studio with bigger datasets

## \[1.11.2\] - 2025-04-17[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1112---2025-04-17 'Direct link to [1.11.2] - 2025-04-17')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-28 'Direct link to Bugfixes')

- Fixed CVE-2025-32812

## \[1.11.1\] - 2025-02-05[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1111---2025-02-05 'Direct link to [1.11.1] - 2025-02-05')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-29 'Direct link to Bugfixes')

- Fixed an issue where the slots list was not updating after creating new slots.
- Corrected the behavior to ensure landing on the correct response in the responses page when creating or editing responses.
- Other minor bug fixes.

## \[1.11.0\] - 2025-01-15[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1110---2025-01-15 'Direct link to [1.11.0] - 2025-01-15')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-8 'Direct link to Features')

- **Responses in CMS:** We’re introducing a centralized hub for content managers and flow builders to manage all the responses used in an assistant project in one place.

[Learn more about Responses in CMS](https://www.rasa.com/docs/reference/primitives/responses/)

![image](https://www.rasa.com/docs/assets/images/responses-00-40cb7d29987730ed6c4dc05d563a44f7.png)

- **Conditional Response Variations:** Conditional responses let your assistant choose specific response variations based on one or more slot values, enhancing personalization and flexibility in replies. This feature allows users to easily create multi-lingual assistants or segment responses based on different criteria.

[Learn more about Conditional Response Variations](https://www.rasa.com/docs/studio/build/content-management/responses/#conditional-response-variations)

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-30 'Direct link to Bugfixes')

- Fixed an issue on import of categorical slot type.
- Aligned Validation Rules between Rasa Studio and the Rasa Framework for configuring conditions where values are surrounded by double quotes.
- Other minor bug fixes.

## \[1.10.2\] - 2025-04-17[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1102---2025-04-17 'Direct link to [1.10.2] - 2025-04-17')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-31 'Direct link to Bugfixes')

- Fixed CVE-2025-32812

## \[1.10.1\] - 2024-12-19[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1101---2024-12-19 'Direct link to [1.10.1] - 2024-12-19')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-32 'Direct link to Bugfixes')

- Remove unwanted `model-service` database creation step from `db migration job` during Studio deployment.

## \[1.10.0\] - 2024-12-17[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#1100---2024-12-17 'Direct link to [1.10.0] - 2024-12-17')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-9 'Direct link to Features')

- **Simpler Training in Studio:** Enhanced Studio’s training architecture and performance to improve reliability, reduce failures, and simplify Deployment
 - The old setup involved multiple services for model running and training, leading to more points of failure. These have been consolidated into a single “Simple Model Service”, reducing moving parts and making training more reliable. This brings down the number of container images in Studio to `5`.
 - The new architecture allows for faster training times, with a `85%` improvement in training time.
 - We’ve replaced NFS with new storage options to provide greater flexibility for storing trained models:
 - **Disk Storage**: Simplified local storage for using Kubernetes Persistent Volume.
 - **Cloud Integration**: Seamless integration with your preferred cloud storage solution.

![image](https://www.rasa.com/docs/assets/images/old-training-architecture-e18cf3fb4cec1ad5e19802cdeefd75a7.png)

Old training architecture.

![image](https://www.rasa.com/docs/assets/images/new-training-architecture-1ead875b0c40bcec0c4fae43c37b95ae.png)

New training architecture.

- **Error UX Improvements:** Various training and deployment errors are now handled directly in the UI, allowing users to view errors in the popover without needing to download the training log.

![image](https://www.rasa.com/docs/assets/images/training-error1-f9ea394360dc78c655125a1c11762379.png)

Error during training.

![image](https://www.rasa.com/docs/assets/images/training-error2-b9dc7748e877e7d7473f0e3861bd04cd.png)

Error during Deployment, which goes right after successful training.

- **Model Download**: Model input data which was used to train a model can be downloaded from UI in YAML format.
 
 ![image](https://www.rasa.com/docs/assets/images/model-input-download-0a781ef1b82c749d321cb3f66133394f.png)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-14 'Direct link to Improvements')

- **Performance Improvements**: Improved performance of the Studio application to support large assistants and complex flows. Performance improved for following features:
 - Assistant pre-training validation
 - Assistant export and training
 - Toggling “ready for training” for flows linked from other flows
 - Creating a Logic node
 - System flows page
 - Small performance improvements in a few other areas
- **Model Download API for CI Integration**: Updates to the Model Download API to support CI integration to support the new model service architecture.

### Migration Guide[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#migration-guide 'Direct link to Migration Guide')

- Make sure to use the latest Helm chart version `2.0.1` & above for deploying this release.
- Make sure to provision a Persistent Volume for storing trained models in the new architecture. Alternatively you can use cloud storage for storing trained models. More details can be found in the [deployment guide](https://www.rasa.com/docs/reference/deployment/studio-hardware-requirements/).
- If you are migrating from an older version of Studio, you can safely delete the below pods and services from your Kubernetes cluster after deploying the latest version of Studio:
 - Pods starting with `dsj-` for example `dsj-0f436992-895c-4fb3-92cb-045ea06c41df-4q7tx`
 - Pods starting with `tsj-` for example `tsj-0f436992-895c-4fb3-92cb-045ea06c41df-4q7tx`
 - Services starting with `dss-` for example `dss-048671c6-3086-49c0-afa7-a17043df2287`

## \[1.9.0\] - 2024-11-08[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#190---2024-11-08 'Direct link to [1.9.0] - 2024-11-08')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-10 'Direct link to Features')

- **Conversation Tagging API:** Expose API for tagging conversations for our clients. Key updates include:
 
 - Keycloak client: Introduce `studio-external` Keycloak client ID to authenticate the API calls
 - Simplified conversation query: Fetch conversations filtered by time range
 - Delete conversations: A new API to delete conversations by ID
 - Tag management: Tag IDs will be visible in the Studio UI to support integrations that use conversation tagging
 - Assistant ID: Assistant UUID id is now visible for customers using API to leverage it
 
 [Learn more about conversation tagging API](https://www.rasa.com/docs/reference/api/studio/conversation-api/)
 
- **API for CI integration:** API for CI integration enables Studio users to automate the download of their assistant data via an API endpoint. Instead of manually exporting and pushing data to their infrastructure, technical users can now pull Studio's data directly into any CI platform. This is accomplished by exposing a query that returns raw YAML files and the trained model file (`model.tar.gz`). It is possible to download `latest` and all trained assistant versions.
 
 [Learn more about API for CI integration](https://www.rasa.com/docs/reference/api/studio/ci-integration-api/)
 

## \[1.8.1\] - 2024-10-30[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#181---2024-10-30 'Direct link to [1.8.1] - 2024-10-30')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-33 'Direct link to Bugfixes')

- Fixed an issue to prevent steps from being wrongly referenced in flow during export/download

## \[1.8.0\] - 2024-10-23[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#180---2024-10-23 'Direct link to [1.8.0] - 2024-10-23')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-11 'Direct link to Features')

- **Flow Version Control:** Flow Version Control allows you to save the flow version at any point, which will be logged in the flow history. From the flow history, you can restore previously saved versions, allowing you to revert any breaking or unwanted changes, giving you further control over the changes that you or your teammates have made to flows.
 
 [Learn more about flow version control](https://www.rasa.com/docs/studio/build/content-management/version-control/)
 
 ![image](https://www.rasa.com/docs/assets/images/primitive-vs-flow-changelog-revert-9177ea3dcf54a4a969454210a8d1e744.png)
 
- **Button Payloads:** Button payloads allow you to assign commands to buttons, so when a user clicks on them, your assistant knows exactly what to do next. This can include:
 
 - Setting slot values
 - Triggering intents
 - Passing entities to the assistant
 - Sending predefined messages to guide the conversation
 
 While Rasa Framework users have been enjoying this feature, we’re bringing this capability to Studio, making it even more powerful and flexible for building your conversational experiences.
 
 [Learn more about buttons in Studio](https://www.rasa.com/docs/studio/build/flow-building/collect/#buttons)
 
 ![image](https://www.rasa.com/docs/assets/images/Payloads-9e948956eac87fed7c3a271d263ccd09.jpg)
 
- **Download your assistant project** You can now download your entire assistant project, including all flows marked as "Ready for training," along with the NLU data and configuration files, all in one click.
 
 ![image](https://www.rasa.com/docs/assets/images/Download-36532aba0f1362cfec98717773a3ef28.jpg)
 

## \[1.7.2\] - 2024-10-04[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#172---2024-10-04 'Direct link to [1.7.2] - 2024-10-04')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-34 'Direct link to Bugfixes')

- Fixed a couple of issues with conditions of if in logic node.
- Fixed an issue where empty page was rendered if slot used in flow guard condition was deleted

## \[1.7.1\] - 2024-10-01[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#171---2024-10-01 'Direct link to [1.7.1] - 2024-10-01')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-35 'Direct link to Bugfixes')

- Fixed an issue where assistant couldn't be trained when a flow uses flow guards.

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-15 'Direct link to Improvements')

- Added support for `jinga templates`.

## \[1.7.0\] - 2024-09-20[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#170---2024-09-20 'Direct link to [1.7.0] - 2024-09-20')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-12 'Direct link to Features')

- **Customizing patterns in Studio:** System flows, also known as patterns, are pre-built flows available out of the box, designed to handle conversations that go off track. For example, they help when:
 
 - The assistant asks for information (like an amount of money), but the user responds with something else.
 - The user interrupts the current flow and changes the topic.
 - The user changes their mind about something they said earlier.
 
 You can now fully customize these system flows in Studio in the “System flows” tab.
 
 [Learn more about patterns in Studio](https://www.rasa.com/docs/studio/build/flow-building/system-flows/)
 
 ![image](https://www.rasa.com/docs/assets/images/customizing_patterns_in_studio-5d09507ff34bb3aaf9cd3030a74e3d61.png)
 
- **Custom actions in the Collect step:** Studio now supports collecting slot values via custom actions, in addition to the existing method of using responses. This feature, available from Rasa Pro 3.8, enhances the flexibility of the Collect step.
 
 Instead of solely relying on templated responses, you can now employ a custom action to collect slot values. This is particularly useful when you need to display dynamic options fetched from a database, such as presenting available values as buttons.
 
 [Learn more about different ways to collect information](https://www.rasa.com/docs/studio/build/flow-building/collect/)
 
 ![image](https://www.rasa.com/docs/assets/images/CAinCollect-65d55fa8c03f511e14d49cb801c79115.png)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-16 'Direct link to Improvements')

- You can now export your flow as an image, making it easier to share and visualize your conversations.
 
 ![image](https://www.rasa.com/docs/assets/images/export_flow_as_image-8acd5c7ad54dc72ebb55f6e6f830f65d.png)
 

## \[1.6.1\] - 2024-09-10[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#161---2024-09-10 'Direct link to [1.6.1] - 2024-09-10')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-36 'Direct link to Bugfixes')

- Fixed issue in upload/download assistant with entity slot mapping with roles.
- Prevent slot creation and update if name contains the dot character.
- Support `rasa studio import` to work with flows starting with condition step.

## \[1.6.0\] - 2024-08-30[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#160---2024-08-30 'Direct link to [1.6.0] - 2024-08-30')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-13 'Direct link to Features')

- **Assistant Settings:** The new "Assistant settings" page in Studio is designed to streamline how you manage and configure your assistant. This new page replaces the "Manage assistants" modal and features two key areas:
 
 1. **General settings:** View and manage basic settings such as the assistant's name and mode.
 
 2. **Configuration:** Allows users in `developer` role to directly edit the `config.yml` and `endpoints.yml` files, enabling real-time adjustments without leaving Studio.
 
 
 This update makes it easier to see and adjust your assistant’s settings directly within Studio, enhancing control and flexibility.
 
 [Learn more about Config in Studio](https://rasa.com/docs/studio/user-guide/configure/)
 
 ![image](https://www.rasa.com/docs/assets/images/Configuration-bc2003e9e2b28897e566658706e0c89a.png)
 
- **Scalable Command Generator:** The [flow-retrieval](https://www.rasa.com/docs/reference/config/components/llm-command-generators/#retrieving-relevant-flows) feature, introduced in Rasa Pro 3.8, is now implemented in Studio. It ensures that only the flows relevant to the conversation context are included in the LLM prompt. This optimizes performance by managing the number of flows more efficiently and reduces LLM-related costs.
 
 Enabled by default in the `config.yml` file accessible via the new Assistant settings page, this feature can be adjusted or disabled as required. It also allows users to designate specific flows as "always included" in the prompt, ideal for general flows like "welcome" and "chitchat" to ensure they are consistently retrievable.
 
 ![image](https://www.rasa.com/docs/assets/images/SCG-f26ffd7005f9c40c7fe1a0542f317260.png)
 
- **Slot Mappings:** Slot mappings are global settings that define how slot values should be extracted.
 
 By default, in Rasa, the slot value is extracted from the user's message using an LLM. However, if users prefer to extract slots from a specific custom action, intent, entity, or the last user message instead of relying on LLM, they can now do so in Studio. [Learn more about slot mappings](https://www.rasa.com/docs/studio/build/flow-building/collect/#slot-mappings-advanced)
 
 ![image](https://www.rasa.com/docs/assets/images/slot-mappings-5ebca9d3208b726ed388a92477f1832a.jpg)
 
- **Flow Change Log:** To enhance collaboration and increase transparency in assistant development, we've introduced the flow change log. This feature provides a detailed history of each flow's changes, including details on who made the change and when. It aims to improve project management by allowing users to track modifications and eventually, revert to previous versions if necessary \[coming soon\].
 
 ![image](https://www.rasa.com/docs/assets/images/flow-change-log-f6cd8b47ce13a86f7fa6500c9c982ae3.png)
 

## \[1.5.6\] - 2024-10-31[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#156---2024-10-31 'Direct link to [1.5.6] - 2024-10-31')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-37 'Direct link to Bugfixes')

- Fixed an issue to prevent steps from being wrongly referenced in flow during export/download

## \[1.5.5\] - 2024-09-26[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#155---2024-09-26 'Direct link to [1.5.5] - 2024-09-26')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-38 'Direct link to Bugfixes')

- Increased database transaction timeout and made performance improvements to prevent errors during flow node creation.

## \[1.5.4\] - 2024-07-31[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#154---2024-07-31 'Direct link to [1.5.4] - 2024-07-31')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-39 'Direct link to Bugfixes')

- Fixed issue where in-development feature flags were not being respected

## \[1.5.3\] - 2024-07-30[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#153---2024-07-30 'Direct link to [1.5.3] - 2024-07-30')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-40 'Direct link to Bugfixes')

- Fixed performance issues related to flow validation

## \[1.5.2\] - 2024-07-26[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#152---2024-07-26 'Direct link to [1.5.2] - 2024-07-26')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-41 'Direct link to Bugfixes')

- Fixed validation issues related to linked flows
- Fixed issue where a new flow's `ready for training` button was disabled even after adding a valid node for the first time
- Other minor bug fixes

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-17 'Direct link to Improvements')

- Update `studio-database-migration` container image and `studio-backend` image to support deployments in restrictive environments

## \[1.5.1\] - 2024-07-23[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#151---2024-07-23 'Direct link to [1.5.1] - 2024-07-23')

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-18 'Direct link to Improvements')

- Update `studio-web-client` container image to use `nginx-unprivileged` base image to support deployments in restrictive environments

## \[1.5.0\] - 2024-07-19[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#150---2024-07-19 'Direct link to [1.5.0] - 2024-07-19')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-14 'Direct link to Features')

- **Flow Validation:** Users are provided with real-time error feedback during the flow creation process. This ensures that errors are caught before training starts and allows for explicit error messages to be displayed in the user interface.
 
 **The feature includes:**
 
 Mapping errors to specific fields where they occurred — allowing users to easily return to the flow and correct them at any time.
 
 ![image](https://www.rasa.com/docs/assets/images/Validation-on-node-3e6ebf671646775835146a9a7209ca7f.png)
 
 Error status on the assistant level - after clicking the "Train" button and before actual training begins, Studio performs a quick validation of all the flows marked as “Ready for training”, indicates if there are errors in any particular flows, and shows the list. This helps users navigate through all the errors until they’re all fixed before attempting to train the model again.
 
 ![image](https://www.rasa.com/docs/assets/images/Validation-on-node-3e6ebf671646775835146a9a7209ca7f.png)
 
- **[Call a flow and return](https://www.rasa.com/docs/studio/build/flow-building/linking-flows/):** This step enables control to jump from a parent flow to a child flow and then return to the original parent flow. The child flow is treated as part of the parent flow, which results in the following behaviors out of the box:
 
 - Slots in the child flow can be filled upfront, without needing to reach the collect step of that slot.
 - Slot values in the child flow can be corrected after they have been filled and control has returned to the parent flow.
 - Slots in the child flow are not reset until the parent flow is completed.
 
 ![image](https://www.rasa.com/docs/assets/images/call-flow-and-return-ed5324a133888cbef78924cdaf97252b.png)
 
- **[Data retention policy](https://www.rasa.com/docs/reference/deployment/automatic-conversation-deletion/):** Users can define a custom period for how long Studio should retain conversation data.
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-42 'Direct link to Bugfixes')

- Fixed issue where the import of a CALM assistant fails when `nlu.yml` is missing or when no entities are present in the `nlu.yml` file
- Fixed import failure of a CALM assistant when slots are of `boolean` type
- Conversation view filter are now reset when user changes the assistant

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-19 'Direct link to Improvements')

When logging in, the user is redirected to a specific page based on their role:

![image](https://www.rasa.com/docs/assets/images/default-landing-page-fcae9b33cc64fabb9f72430905b8f5e9.png)

**In CALM-based assistant**:

- Flow Builder, NLU Editor, leadAnnotator, Developer and Business User are redirected to **Flows** page.

**In Classic assistant:**

- Lead Annotator is redirected to **Annotation Dashboard**.
- Annotator is redirected to **Annotation Inbox**.
- Business User, NLU Editor, Flow Builder, Developer are redirected to **NLU**

Conversation Analyst is redirected to **Conversation View,** regardless of assistant mode.

## \[1.4.0\] - 2024-06-14[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#140---2024-06-14 'Direct link to [1.4.0] - 2024-06-14')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-15 'Direct link to Features')

- **Flow Guards:** [Flow Guard](https://www.rasa.com/docs/studio/build/flow-building/trigger-flows/#flow-guards) enables users to define specific conditions for triggering a flow, ensuring that the flow is not activated solely by user request
 
 ![image](https://www.rasa.com/docs/assets/images/FlowGuards-bc1c740803dca35bdfe22011d87d9b24.png)
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-43 'Direct link to Bugfixes')

- Fixed issue related to primitives not being pre-selected in the Manage window
- The condition editor no longer disappears when a new slot is created
- Fixed issue where Slot is not being updated after a different slot is selected in the dropdown
- Other minor bug fixes

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-20 'Direct link to Improvements')

- Updated label of condition nodes. We now provide a clear indication of the logic within the condition, allowing users to quickly understand the conditions at a glance
- Added validation to check that the slot name doesn't contain any special characters
- When creating a collect message for a categorical slot, only values of a categorical slot can be used to create buttons for the related message, so that wrong entries can be prevented
- In the Flows table long flow descriptions are now visible on hover over the description
- Users can share links to filtered views of the Conversation Table and to individual conversations
- Other minor application improvements

## \[1.3.2\] - 2024-06-05[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#132---2024-06-05 'Direct link to [1.3.2] - 2024-06-05')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-44 'Direct link to Bugfixes')

- Fixed issue where slots on different logical nodes got merged when exporting a flow to yml

## \[1.2.1\] - 2024-06-04[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#121---2024-06-04 'Direct link to [1.2.1] - 2024-06-04')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-45 'Direct link to Bugfixes')

- Fixed issue where slots on different logical nodes got merged when exporting a flow to yml

## \[1.3.1\] - 2024-05-30[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#131---2024-05-30 'Direct link to [1.3.1] - 2024-05-30')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-46 'Direct link to Bugfixes')

- Fixed issue where error raised during an unsupported yml import of a CALM assistant had an incorrect error message

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-21 'Direct link to Improvements')

- Added support for entity annotation to CALM assistant import

## \[1.3.0\] - 2024-05-16[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#130---2024-05-16 'Direct link to [1.3.0] - 2024-05-16')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-16 'Direct link to Features')

- **Assistant Import:** CALM Assistant import allows Rasa Pro customers to import existing assistants into Rasa Studio. [Learn more](https://www.rasa.com/docs/reference/api/command-line-interface/#import-of-calm-assistants).

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-47 'Direct link to Bugfixes')

- Fixed issue where adding a new categorical slot value doesn't cause the if condition to not display the previously defined value anymore.
- Initial Value selection is now saved correctly for new categorical slots.
- In Flows, when adding a Logic step, the nodes order is now correct.
- Other minor bug fixes.

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-22 'Direct link to Improvements')

- **Auto-format primitives**: Validation for message and action names is simplified: they don't need prefixes `utter_`, `utter_ask_` , `utter_invalid_`, or `action_` anymore.
 
- For all primitives that don't allow spaces in names, whitespace is automatically replaced with `_` during typing.
 
 ![image](https://www.rasa.com/docs/assets/images/1.3-changelog-autoformatting-primitives-ecfb0637580cb3e2e638e783452f060e.png)
 
- Search option for managing slots is added.
 
- Behaviour of "Ready for training" checkbox is now consistent.
 
- **Support for additional events in Conversation Review event stream**: Users can now see the following additional events in the conversation stream: `FORM`, `ACTIVE_LOOP`, `FOLLOWUP_ACTION`, `RESTARTED`.
 
- **Review multiple conversations in Conversation Review**: Users can select multiple conversations and create a batch for review.
 
 ![image](https://www.rasa.com/docs/assets/images/1.3-changelog-conversation-view-select-aa845c29a4299e6cd60c1d2248a3fbcc.png)
 
 ![image](https://www.rasa.com/docs/assets/images/1.3-changelog-conversation-view-batch-review-8dbd884cf75e9b9e232c8fa54a01b4a1.png)
 
- **Conversation Review RBAC**: Admins can now assign a Conversation Analyst role to users. Only users with this role will be able to access and use the Conversation Review feature.
 
- Change default classifier from `DIETClassifier` to `LogisticRegressionClassifier` for assistants with NLU triggers.
 
- Other minor application improvements.
 

## \[1.2.0\] - 2024-04-18[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#120---2024-04-18 'Direct link to [1.2.0] - 2024-04-18')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-17 'Direct link to Features')

- **Conversation View:** Helps users identify ways to improve their NLU and CALM assistants by reading real user conversations. With [Conversation View](https://www.rasa.com/docs/studio/analyze/conversation-review/), users can browse user conversations, apply filters to help surface conversations in need of analysis and then see a turn-by-turn breakdown of the conversation, along with relevant session and event data.
 
 ![image](https://www.rasa.com/docs/assets/images/cv-table-5beed744eca4e344b931e55b2f6ae730.png)
 
 ![image](https://www.rasa.com/docs/assets/images/cv-turns-e0d8d3cfcbf9afad5394b8d8a7053ca7.png)
 
- **NLU Triggers:** Enable users to create intents in Modern assistants and use them as a method for triggering flows. [Start step](https://www.rasa.com/docs/studio/build/flow-building/trigger-flows/) is used to provide a comprehensive view of what can initiate or hinder the start of a flow: CALM, [NLU Triggers](https://www.rasa.com/docs/studio/build/flow-building/trigger-flows/#nlu-triggers), or [Links/Calls](https://www.rasa.com/docs/studio/build/flow-building/trigger-flows/#links--calls).
 
 ![image](https://www.rasa.com/docs/assets/images/nlu-triggers-de3f2411e877b77967e8f634e6e04c03.png)
 
- **NLU page in Modern mode:** Bring NLU page into Modern assistants for managing intents, setting a precedent for future primitives management. Users can now create, edit, and delete intents in the NLU page.
 
 ![image](https://www.rasa.com/docs/assets/images/nlu-modern-mode-36a8f9b723df95418fac3c322c66489e.png)
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-48 'Direct link to Bugfixes')

- Fixed issue where the users were unable to export `else` node when it is in the end of the flow
 
- Fixed issues related to inconsistent state of `Manage Slots` modal
 
- Fixed issue where navigating back from a flow takes the user to the 1st page in the list of flow
 
- Other minor bug fixes
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-23 'Direct link to Improvements')

- **Added an optional "Initial value" field to slots:** Let users specify an initial value for any slot, just as they can already do in Rasa Pro.
 
 ![image](https://www.rasa.com/docs/assets/images/initial-slot-value-e7d0463da43308920db91f924527550b.png)
 
- Users no longer have to type `utter_` prefix when creating a message node
 
- Users no longer have to type `action_` prefix when a creating a custom action node
 
- Other minor application improvements
 

## \[1.1.2\] - 2024-03-27[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#112---2024-03-27 'Direct link to [1.1.2] - 2024-03-27')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-49 'Direct link to Bugfixes')

- Fixed issue where users were unable to update a slot under certain conditions

## \[1.1.1\] - 2024-03-26[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#111---2024-03-26 'Direct link to [1.1.1] - 2024-03-26')

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-50 'Direct link to Bugfixes')

- Removed the alphabetical sort of categories in the "Manage slots" modal
- Fixed issue where "Open the flow in a new tab" did not open the selected flow
- "Show variations" button in a message node is only displayed if there are message variations present
- Fixed issue which caused the creation of orphan nodes in flow builder
- Else condition nodes now do not allow assigning conditions to them
- Fixed issue where the flow builder was not listing all the flows while creating a link node
- Fixed issue related to invalid training data with wrong reference ID
- Studio now displays the affected message name in the error toast when an incorrect slot name is used in a message node
- Other minor bug fixes

## \[1.1.0\] - 2024-03-18[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#110---2024-03-18 'Direct link to [1.1.0] - 2024-03-18')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-18 'Direct link to Features')

- Support the logic operators from the Pypred library that are currently missing in Studio
 
 - `not`: Negates a condition
 - `=`: Equal to
 - `!=`: Not equal to
 - `matches`: Uses regular expressions to match strings
 - `not matches`: Uses regular expressions to negate strings.
 
 ![image](https://www.rasa.com/docs/assets/images/all-logical-operators-cfc19ef692020ee94e268798d7395bfd.png)
 
- Allow flow builders to assign slot values to slots directly in the flow without using custom actions or collecting user input. By doing so they can dictate the logic independently from user input. Slots can either be precomputed with a value or reset.
 
 ![image](https://www.rasa.com/docs/assets/images/set-slots-88996780be8a5a3439f6a6b7ec7558ce.png)
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-24 'Direct link to Improvements')

- Enable users to select or unselect flows for training directly from the flow list
 
 ![image](https://www.rasa.com/docs/assets/images/set-ready-for-training-7b201e08b436f530a48d7259e406fcb0.png)
 
- Other minor application improvements
 

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-51 'Direct link to Bugfixes')

- Annotator user is now prohibited from viewing options for editing NLU
- Chat history is now not shared between all assistants and user logins
- Fixed issue related to a race condition when changing categorical slots
- Improved error message when training is initiated with an invalid slot value and invalid config yaml
- Other minor bug fixes

## \[1.0.4\] - 2024-02-12[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#104---2024-02-12 'Direct link to [1.0.4] - 2024-02-12')

info

Make sure to use Rasa Studio Helm chart version `0.4.0` & above for deploying this release.

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-19 'Direct link to Features')

- `Try your assistant` page now maintains chat history
 
 Please note that the chat history is maintained in the local storage of the browser. The chat history is not maintained across different browsers
 

![image](https://www.rasa.com/docs/assets/images/tya-chat-history-37e58a4b38d35764a417029e117b4537.png)

- Users can now search flows by flow description

![image](https://www.rasa.com/docs/assets/images/search-by-flow-description-071041ec77baaa561f50571783e705b9.png)

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-25 'Direct link to Improvements')

- Improved security of Studio container images. Studio now uses `non-root` user for running the application
- Simplified deployment time variables in the latest `0.4.0` Helm chart release
- Other minor application improvements

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-52 'Direct link to Bugfixes')

- Fixed issues connected to `rasa studio download` cli failure due to absence of slots and empty flows
- Other minor bug fixes

## \[1.0.3\] - 2024-01-22[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#103---2024-01-22 'Direct link to [1.0.3] - 2024-01-22')

### Features[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#features-20 'Direct link to Features')

- Users can now search for primitives by typing in the search bar

![image](https://www.rasa.com/docs/assets/images/search-primitives-bcf58f5b12eb21f061b164e851fd0fff.png)

- Users can now use `Azure OpenAI API` with Studio to train their assistants. A new `RASA_CONFIG_FILE` environment variable can now be passed to the `backend` service to define the `config.yaml` file used for training the assistant
 
 Please note that the contents of the `config.yaml` file needs to be `base64` encoded and passed to the new environment variable. The Studio `backend` service will decode the variable value and use it for training the assistant. Users can pass the `Azure OpenAI API` key to the existing `OPENAI_API_KEY_SECRET_KEY` environment variable
 
 The `RASA_CONFIG_FILE` values overrides the `Advanced configuration` options defined in the `Create assistant Project` model in the Studio web client
 
 Sample `config.yaml` file:
 
 ```
      recipe: default.v1  language: en  pipeline:      - name: LLMCommandGenerator      llm:          model_name: gpt-3.5-turbo          api_type: azure          api_base: https://studio-testing.openai.azure.com          api_version: "2023-07-01-preview"          engine: gippity-35  policies:  - name: rasa.core.policies.flow_policy.FlowPolicy
    ```
 
 Corresponding `base64` encoded `RASA_CONFIG_FILE` value:
 
 ```
    cmVjaXBlOiBkZWZhdWx0LnYxCmxhbmd1YWdlOiBlbgpwaXBlbGluZToKICAtIG5hbWU6IExMTUNvbW1hbmRHZW5lcmF0b3IKICAgIGxsbToKICAgICAgbW9kZWxfbmFtZTogZ3B0LTMuNS10dXJibwogICAgICBhcGlfdHlwZTogYXp1cmUKICAgICAgYXBpX2Jhc2U6IGh0dHBzOi8vc3R1ZGlvLXRlc3Rpbmcub3BlbmFpLmF6dXJlLmNvbQogICAgICBhcGlfdmVyc2lvbjogMjAyMy0wNy0wMS1wcmV2aWV3CiAgICAgIGVuZ2luZTogZ2lwcGl0eS0zNQoKcG9saWNpZXM6CiAgLSBuYW1lOiByYXNhLmNvcmUucG9saWNpZXMuZmxvd19wb2xpY3kuRmxvd1BvbGljeQ
    ```
 

### Improvements[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#improvements-26 'Direct link to Improvements')

- Improvements related to order of condition steps in flow export
- Better user notifications when there is a connection issue with a deployed model in `try your assistant` page
- Other minor application improvements

### Bugfixes[​](https://www.rasa.com/docs/reference/changelogs/studio-changelog/#bugfixes-53 'Direct link to Bugfixes')

- Fixed issue related to flow training when first step is the `Logic` step
- Users can now delete unused slot
- Other minor bug fixes