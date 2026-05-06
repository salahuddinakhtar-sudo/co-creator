# Recipe Building Blocks and Configurations

### Some Suggestions
Based on My Experience with the CO-Creator
*   **Try to complete the work related to SmartAI block in one-shot** - On publishing a recipe all your chat with SmartAI block will get reset.
*   **Use JSON Object with all the connection and parameter to get a complete flow in oneshot** - Use Other LLM for getting json object and then paste that into Co-creator

## VOICE-SUPPORTED BLOCKS (for your VOICE recipe)

### MessageBlock
Sends a text message and immediately moves to the next block without waiting for user input.
*   **Required:** `text` (message content, max length depends on channel)
*   **Optional:** `next_block` (where to go after sending)
*   **Use when:** You need to send information, instructions, or prompts without expecting a response.

### AskBlock
Voice-only single-turn Q&A block that classifies user responses into intents and optionally extracts variables.
*   **Required:** `question` (the spoken question, max 500 chars), `intents[]` (at least one intent), `failover_block` (where to go if no intent matches)
*   **Intent structure:** `intent_name` (unique ID), `next_block` (jump target), `variables_to_capture[]` (optional extracted values)
*   **Optional:** `condition` (rule the response must satisfy before proceeding), `examples[]` (up to 5 samples to guide the AI), `max_attempts` (1-50, default 5), `enable_conversation_history` (include prior context), `output_variable` (store detected intent), `enable_nudge` (reminders if user doesn't respond), `answerflow_configuration` (integrate knowledge base), `allow_barge_in` (let user interrupt), `speech_normalisation` (normalize currency, dates, etc. in voice output)
*   **Use when:** You need voice-based intent detection in a single turn.

### SmartAIBlock
Intent classification with LLM-powered multi-turn capability. Supports detailed prompting, intent extraction, and knowledge base integration.
*   **Required:** `llm_prompt` (main prompt, max 5000 words), `intents[]`, `failover_block`
*   **Optional:** `persona` (bot instructions, max 2000 words), `common_instructions` (shared across intents, max 2000 words), `specific_instruction_prompt` (additional context, max 2000 words), `max_attempts` (1-50, default 5), `output_variable` (store intent), `mandatory_intent_identification` (force intent match or allow pass-through), `expressiveness` (0-1 scale for response creativity), `enable_conversation_history`, `enable_nudge`, `answerflow_configuration` (document tags for retrieval, enabled by default if tags provided), `allow_barge_in`, `speech_normalisation`
*   **Use when:** Complex intent detection, variable extraction, or knowledge-base-augmented responses are needed.

### ConditionBlock
Switch-case branching based on variable values.
*   **Required:** `cases[]` (at least 1, max 10), `default_next_block`
*   **Case structure:** `conditions[]` (at least 1), `logical_operator` (AND/OR), `next_block`
*   **Condition structure:** `lhs` (left operand, value or variable), `rhs` (right operand, omit for not operator), `operator` (=, !=, >, >=, <, <=, contains, not_contains, in, not_in, not)
*   **Operators by type:** Numeric (=, !=, >, >=, <, <=), String (=, !=, contains, not_contains), Collection (in, not_in), Unary (not)
*   **Use when:** Route based on captured variable values or system state.

### CodeBlock
Execute TypeScript logic, read/write recipe variables, call plugins.
*   **Required:** `code` (function body, max 8000 chars), `selected_plugins[]`, `next_block`
*   **Optional:** `fail_block` (error handler), `variables` (array of variables the code reads), `result_variable_type` (declare return type: TEXT, INT, DATE, PERCENTAGE, JSON, LIST)
*   **Code structure:** `function main(context, variables, plugins) { /* your code */ }`
*   **Available inside:** `variables.variable_name` (recipe var), `variables.system.date_now` (system var), `variables.BlockName.response.*` (dynamic var), `plugins.SetRecipeVariables(name, value)` (write var)
*   **Use when:** Custom logic, transformations, or variable assignments are needed.

### APIBlock
Make HTTP requests to external APIs.
*   **Required:** `http_method` (GET, POST, PUT, DELETE, PATCH), `next_block` (success), `fail_block` (error)
*   **URL:** Either `url` (full URL, max 1024 chars) or set `use_base_url: true` (use recipe-level webhookBaseURL)
*   **Optional:** `auth` (auth config name), `headers` (key-value, max 30), `query_params` (key-value, max 20), `json_body` / `form_data_body` / `form_url_encoded_body` (one of these, max 4056 chars), `http_timeout` (1-40 sec, default 15), `no_of_retries` (0-10, default 1)
*   **Response variables:** `{{BlockName.response.status_code}}`, `{{BlockName.response.body.field}}`, `{{BlockName.response.headers.Header-Name}}`
*   **Use when:** Fetching data, pushing updates, or integrating external systems.

### TransferBlock
Hand off to human agent.
*   **Required:** `reason` (transfer message, max length depends on channel)
*   **Optional:** `when_to_transfer` (AVAILABLE, ONLINE, ACCEPTING_CHATS), `transfer_to_specific_department` (department name)
*   **Fallback actions:** `action_when_no_agent_available`, `action_when_no_agent_online`, `action_when_out_of_business_hours` (each with type: WAIT_FOR_AGENT or MOVE_TO_NEXT_BLOCK, and optional `next_block`)
*   **Use when:** User needs human assistance or the bot cannot handle the request.

### CloseBlock
End the conversation.
*   **Required:** `message` (closing message)
*   **Use when:** The flow is complete and should terminate.

---

## CHAT-ONLY BLOCKS (NOT available on your voice recipe)

### QuestionBlock
Ask a question and wait for user response; store answer in a variable.
*   **Required:** `question_text`, `invalid_user_input_message`
*   **Optional:** `response_variable` (store answer), `next_block` (default if no button matched), `buttons[]` (optional click targets), `enable_nudge`, `enable_agent_transfer`, `hide_input_text_area` (buttons only, no free text), `make_question_mandatory` (require answer), `reask_message` (if mandatory), `refuse_message` (if not mandatory), `enable_faq`, `use_recipe_level_faq`, `faq_categories[]`
*   **Button structure:** `label`, `action` (link/next_block), `url` (if link), `next_block` (if next_block), `variable_to_set`, `trigger_words` (alternative phrases)

### MediaBlock
Display image, video, audio, or file without waiting for input.
*   **Required:** `media_type` (image, video, audio, file), `media_url`
*   **Optional:** `next_block`

### SliderBlock
Horizontal scrolling cards with images, text, and buttons.
*   **Required:** `cards[]` (at least 1), `invalid_user_input_message`
*   **Card structure:** `image` (optional), `card_title`, `card_description`, `buttons[]` (at least 1 per card)
*   **Button structure:** `label`, `action`, `url` (if link), `next_block` (if next_block), `variable_to_set`

### ButtonBlock
Header text with clickable buttons (no free-text input).
*   **Required:** `header_text`, `buttons[]` (at least 1), `invalid_user_input_message`
*   **Optional:** `next_block` (if set, no waiting), `enable_nudge`, `enable_agent_transfer`, `hide_input_text_area`, `enable_faq`, `use_recipe_level_faq`, `faq_categories[]`

### QuickLinksBlock
WhatsApp-only button with URL and optional media.
*   **Required:** `button_title` (max 20 chars), `button_url`, `body_text` (max 1024 chars)
*   **Optional:** `header_type` (text/media), `header_text` (max 60 chars, if type=text), `media_type` (image/video/file, if type=media), `media_url` (if type=media), `footer` (max 60 chars), `next_block`

### ListBlock
WhatsApp-only interactive list with sections and rows.
*   **Required:** `list_title` (max 24 chars), `list_description` (max 72 chars), `list_button_name` (max 20 chars), `list_title_for_sections` (max 72 chars), `sections[]` (at least 1)
*   **Section structure:** `section_title` (max 24 chars), `rows[]` (at least 1)
*   **Row structure:** `row_title` (max 24 chars), `row_description` (max 72 chars, optional), `variable_to_set`, `next_block`
*   **Optional:** `enable_agent_transfer`, `hide_input_text_area`, `invalid_user_input_message`, `enable_nudge`, `enable_faq`, `use_recipe_level_faq`, `faq_categories[]`

### FormBlock
WhatsApp-only form submission.
*   **Required:** `cta_button_text`, `success_next_block`, `failure_next_block`
*   **Optional:** `question`, `enable_nudge`, `enable_faq`, `use_recipe_level_faq`, `faq_categories[]`
*   **Note:** Form selection and media attachment are UI-only (cannot be set via API).

### WebhookBlock
Receive data via webhook callback (asynchronous integration).
*   **Required:** `url` (webhook URL, max 1024 chars), `failure_message`, `failure_next_block`
*   **Optional:** `type` (synchronous/asynchronous), `headers`, `fallback_next_block`, `response_variables[]` (optional variable extraction)

---

## RECIPE-LEVEL SETTINGS (available for all recipes)

*   **`first_block` (required):** Starting block name
*   **`webhookBaseURL`:** Base URL for relative API/webhook calls
*   **`eventsURL`:** Webhook for room lifecycle events
*   **`enableFillerSoundForVoice` (voice):** Add filler sounds during processing
*   **`faq.primaryConfidence` (chat):** FAQ match threshold (0-100)
*   **`faq.secondaryConfidence` (chat):** FAQ fallback threshold (0-100)
*   **`voicemailSettings` (voice):** `enabled`, `actionType` (LEAVEMESSAGE/DISCONNECT), `leaveMessage`
*   **`promptSettings[]`:** Recipe-wide LLM instructions for SmartAI/AskBlock. Types include Persona, Common Instructions, Specific Instructions, Response Synthesis, Goal, Condition, and Examples (Each max 2000 words)

---

## RECIPE MENU (optional persistent UI)

*   **`title`:** Menu header
*   **`menu_items[]` (max 15):** Requires `label` AND one of the following: `url` (external link), `next_block` (jump target), or `variable_to_set` (assign on click)

---

## FAQ CONFIGURATION (chat recipes only)

*   **Recipe-level:** `update_recipe_faqs` with `faq_categories[]` (categories and jump-block mappings)
*   **Block-level:** `enable_faq`, `use_recipe_level_faq`, `faq_categories[]` (Available on QuestionBlock, ButtonBlock, ListBlock, FormBlock)

---

## VARIABLE TYPES

*   **Recipe variables:** `int`, `positive_int`, `text`, `phone`, `email`, `name`, `percentage`, `location`, `pincode`, `json`, `list`
*   **System variables (read-only):** `system.date_now_utc`, `system.date_now`, `system.time_now`, `user.phone`, `user.ip`, `user.country`, `user.first.response`, `room.channel`, `room.call_type`, `room.call_direction`, `agent.phone`, `agent.current_block`, `agent.previous_block`, `agent.last_message`
*   **Dynamic variables (block-scoped):** `{{BlockName.response.result}}`, `{{BlockName.response.body.field}}`, `{{BlockName.variables.variable_name}}`

# Capabilities and Limitations Guide

## BLOCK CREATION & EDITING

*   **Create new blocks:** MessageBlock, SmartAIBlock, AskBlock, ConditionBlock, CodeBlock, APIBlock, WebhookBlock, TransferBlock, CloseBlock.
*   **Update existing block fields:** Modify text, prompts, intents, conditions, routes, variable assignments, etc.
*   **Delete blocks:** Remove blocks from the recipe.
*   **Set up navigation:** Establish block-to-block navigation and branching logic.
*   **Configure intent classification:** Use SmartAIBlock or AskBlock.
*   **Build conditional logic:** Implement switch-case style routing with ConditionBlock.
*   **Integrate external APIs:** Use APIBlock for GET, POST, PUT, DELETE, and PATCH requests.
*   **Execute custom logic:** Run TypeScript logic with CodeBlock.
*   **Route to human agents:** Hand off conversations using TransferBlock.
*   **End conversations:** Terminate flows with CloseBlock.

---

## VARIABLE & DATA HANDLING

*   **Capture user responses:** Store data into recipe variables (name, email, phone, text, etc.).
*   **Read system variables:** Utilize system data (current time, date, user phone, room channel, agent info, etc.).
*   **Reference dynamic variables:** Use block outputs (API responses, code results, SmartAI extractions).
*   **Embed variables:** Inject variables into messages, prompts, and API calls using the `{{variable_name}}` syntax.
*   **Set conditionally:** Assign variable values based on user choices or logic.
*   **Extract structured data:** Capture variables from user responses via intent matching.

---

## INTENT & KNOWLEDGE INTEGRATION

*   **Configure intents:** Set up SmartAIBlock and AskBlock with multiple intents for classification.
*   **Extract variables:** Pull specific data from user responses for each intent.
*   **Integrate Answerflow:** Connect knowledge bases for AI-powered answers.
*   **Mandatory intent routing:** Force intent matching or allow pass-through routing.
*   **Control AI expressiveness:** Adjust response creativity on a 0-1 scale.
*   **Add examples & rules:** Provide samples and conditioning rules to guide AI behavior.
*   **Enable conversation history:** Maintain multi-turn context.

---

## ROUTING & FLOW CONTROL

*   **Define starting point:** Set `first_block` to define where the recipe starts.
*   **Route via conditions:** Direct users through different paths based on variable values and operators.
*   **Support logic gates:** Utilize AND/OR logic across multiple conditions.
*   **Jump to blocks:** Navigate from buttons, intent matches, API results, or conditions.
*   **Configure fallbacks:** Set up fallback routes for unmatched conditions or API failures.
*   **Nudge users:** Send reminders if they don't respond within a specified timeout.
*   **Agent transfer fallbacks:** Set options for when agents are unavailable (wait, move to block, outside business hours).

---

## SETTINGS & CONFIGURATION

*   **Set base URLs:** Define the recipe-level `webhookBaseURL` for relative API/webhook calls.
*   **Configure events:** Set `eventsURL` to receive room lifecycle events.
*   **Manage voicemail:** Enable/disable voicemail, set messages, and define the action type.
*   **Voice filler sounds:** Configure filler sounds for processing pauses during voice calls.
*   **Global AI prompts:** Add recipe-level prompt settings for SmartAI behavior (Persona, Common Instructions, Specific Instructions, etc.).
*   **Toggle features:** Enable or disable specific features on a per-block basis.

---

## API & WEBHOOK INTEGRATION

*   **Configure API requests:** Set up APIBlock with authentication, headers, query parameters, and request/response bodies.
*   **Set timeouts & retries:** Define timeout limits and retry strategies for API calls.
*   **URL options:** Use relative URLs (if `webhookBaseURL` is set) or absolute URLs.
*   **Reference responses:** Use API response fields in downstream blocks (e.g., `{{BlockName.response.body.field}}`).
*   **Asynchronous integration:** Set up WebhookBlock for asynchronous callbacks.
*   **Success/Failure routing:** Route users to different blocks depending on API success or failure.

---

## CODE EXECUTION

*   **Write custom scripts:** Use TypeScript to transform data, perform calculations, or call plugins.
*   **Access variables:** Read recipe variables and dynamic variables inside the code.
*   **Write variables:** Set recipe variables via the `SetRecipeVariables` plugin.
*   **Return values:** Output computed values (text, numbers, JSON, dates, lists) for downstream use.
*   **Error handling:** Route failures to a designated `fail_block`.

---

## DOCUMENTATION & HELP

*   **Answer questions:** Provide info about block types, parameters, and capabilities.
*   **Explain variables:** Detail how variables work and when to use each type.
*   **Clarify logic:** Explain routing, branching, and flow mechanisms.
*   **Suggest best practices:** Offer advice on optimal recipe design.
*   **Detail limitations:** Provide clear information on constraints and boundaries.

---

## WHAT I CANNOT DO

*   **Create or modify variables:** You must create them in recipe settings first.
*   **Create prerequisites:** Auth configs, code plugins, FAQ categories, Answerflow tags, or departments must be set up in the UI.
*   **Upload media assets:** You must provide valid, hosted URLs.
*   **Add deprecated blocks:** FAQBlock, CatalogBlock, OrdersBlock must be created manually.
*   **Add chat-only blocks to voice:** QuestionBlock, MediaBlock, SliderBlock, ButtonBlock, QuickLinksBlock, ListBlock, and FormBlock cannot be added to a voice recipe.
*   **Rename existing blocks:** Block names are permanently fixed at creation.
*   **Change a block's type:** You must delete the block and recreate it.
*   **Add new languages:** You must add languages in the recipe settings first.
*   **Change the channel:** The recipe's channel (VOICE vs. CHAT) is fixed at creation.
*   **Create forward references:** Cannot link to blocks that do not exist yet.
*   **Bypass prerequisites:** Cannot work around missing variables, auth, plugins, tags, etc.
