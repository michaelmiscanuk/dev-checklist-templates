# Writing a prompt for an app feature

When: You are adding a new LLM-powered feature, or rewriting the prompt behind one, and want it to hold up on real user input.

How to use: Use it when you add an LLM feature or rewrite the prompt behind one. Work through the moments in order, and come back to the last one after every edit. Each check is one thing to get right, with a tip under it. 'How to verify' says what to look at to know it is done.

Before you start: What text besides your own goes into the prompt? (any that apply) Text users type in / Retrieved documents or files / Tool or API results / None of these. Checks marked "only if" apply only to that answer.
Before you start: Which of these apply to the models behind the prompt? (any that apply) Several models or providers / A fallback model as backup / None of these. Checks marked "only if" apply only to that answer.

## 1. Before you write the prompt

You know the feature you want, and no prompt text exists yet.

Must:
- [ ] Gather real user inputs first
  Why: Pull them from logs, support tickets or a beta, and keep the short, messy, rude and off-topic ones.
  How to verify: Look over the set: it holds **awkward and typical** inputs, not just the clean case you pictured.

Don't forget these too:
- [ ] Write the job in one sentence
  Why: Name the input, the output and who uses it; a job that needs two sentences may need **prompt chaining**, one job per prompt.
  How to verify: A teammate reads the sentence and can describe a good reply without asking you anything.
- [ ] Define the pass bar up front (source 1)
  Why: List the few specific, measurable traits you will judge by, such as correct, grounded, right length and right tone.
  How to verify: The pass bar is written down beside the prompt, and two people scoring the same reply agree.
- [ ] Decide who reads the reply
  Why: Code needs a fixed shape, best enforced with a JSON Schema through **structured outputs**; a person needs plain words at the right length.
  How to verify: The prompt names the reader of the reply, and its format section fits that reader.
- [ ] Pin the model and version (source 2)
  Why: Tune on the **exact model** you will ship, since prompts carry over unevenly between models.
  How to verify: Config names a dated model version, not an alias like latest that can change without notice.
- [ ] List every slot you don't control (source 3) (only if: Text users type in or Retrieved documents or files or Tool or API results)
  Why: Each slot filled by users, documents, tool results or earlier chat turns is **data, not instructions**, and gets handled that way.
  How to verify: A short list names every untrusted slot and where it enters the template.
- [ ] Budget tokens for the longest input (source 4)
  Why: Everything shares one window: instructions, tool definitions, the longest real input and the full reply, thinking included; trim or chunk anything bigger.
  How to verify: Check the usage field for the longest input in your set: it fits with room to spare, and the reply is not cut off.
- [ ] Score the current prompt first
  Why: When rewriting a prompt, run the old one on your input set and save its results as the **baseline to beat**.
  How to verify: A saved results table for the old prompt sits beside the new one, case by case.

Stop if:
- If nobody can say what a good reply looks like: ask the product owner before writing

Move on when you have the job in one sentence, a set of real inputs and a written pass bar.

## 2. While drafting the prompt

You are writing the system prompt and the template that wraps each request.

Must:
- [ ] Say what to do outside scope (source 5)
  Why: Give the exact fallback, such as a short decline and where to go instead, so the model need not improvise.
  How to verify: Send an off-topic request and a borderline on-topic one: the first gets your **fallback**, the second a real answer, not an over-refusal.
- [ ] Give a way out when unsure (source 6)
  Why: Give it a line to use, such as "I don't have enough information", or let it ask one question, rather than guess.
  How to verify: Send an input missing a key fact: the reply names the gap instead of inventing a detail.
- [ ] Fence outside text in delimiters (source 7) (only if: Text users type in or Retrieved documents or files or Tool or API results)
  Why: Wrap each untrusted block in clear tags, the step OWASP calls **segregating external content**, and say it is material, never orders.
  How to verify: Send an input that says to ignore your rules: the reply treats it as content and keeps them.

Don't forget these too:
- [ ] State the role and audience
  Why: One line on who the model serves and why sets depth and tone better than a long persona.
  How to verify: The prompt opens with the task and the audience in a sentence or two.
- [ ] Put fixed text first, variables last (source 8)
  Why: Keep instructions and examples as one unchanging prefix and put dates, names and inputs after it, so **prompt caching** can reuse it.
  How to verify: Compare two requests: they match exactly up to the first variable, and usage shows cached tokens above zero, cache_read_input_tokens on Claude.
- [ ] Spell out the reply format (source 9)
  Why: Name length, structure and tone, and show one sample of the shape when another program reads the reply.
  How to verify: Ten replies in a row share the same shape and stay in the same length band.
- [ ] Pair each 'don't' with a 'do'
  Why: A bare 'never mention X' can prime X; tell it what to say or do in that spot instead.
  How to verify: Scan for 'never' and 'do not': each one comes with the thing to do instead.
- [ ] Give the reason behind rules
  Why: A rule with its reason lets the model handle cases you never listed.
  How to verify: Pick three rules at random: each carries its reason in a clause or a line.
- [ ] State each rule once
  Why: Group rules by topic and state each once, so a later edit changes one line, not three.
  How to verify: Search the prompt for each key rule: it shows up in one section only.
- [ ] Answer only from given documents (source 6) (only if: Retrieved documents or files)
  Why: Tell it to use only the documents given, quote the passages it relies on, and say plainly when they do not cover the question.
  How to verify: Ask something the documents do not contain: the reply says it is not covered rather than answering from memory.
- [ ] Keep secrets out of the prompt (source 10)
  Why: Assume users can coax the system prompt out, so include only what serves the request; keys, internal links and private logic stay in code.
  How to verify: Read the prompt as if a user posted it online: nothing in it would cause harm.
- [ ] Drop the all-caps warnings
  Why: Write each rule in normal case with its reason; current models follow **plain wording** well and apply shouted rules too rigidly.
  How to verify: Scan the prompt: no words in capitals for emphasis, and no rule leans on IMPORTANT or MUST.

Stop if:
- If the prompt must hold personal or secret data: ask your security or privacy lead first
- If every new complaint needs another rule: stop and ask whether the job is too broad

Move on when A full prompt runs end to end on a handful of real inputs.

## 3. When you add examples

Rules alone miss some cases, and you want to show the model what good looks like.

Must:
- [ ] Vary examples on every surface (source 11)
  Why: Change length, topic, wording and answer between examples, so the only thing they share is the **pattern you mean**.
  How to verify: Lay the examples side by side: no opening phrase, length or answer repeats across all of them.

Don't forget these too:
- [ ] Draw examples from real inputs
  Why: Clean up real cases rather than inventing tidy ones; real users phrase things in ways you would not.
  How to verify: Each example traces back to a logged or collected input, with personal details removed.
- [ ] Show an edge case and refusal
  Why: One awkward input and one out-of-scope request, each with its right reply, mark the boundary better than more easy cases.
  How to verify: The example set includes at least one edge case and one decline, each answered as you want.
- [ ] Format examples like real replies (source 11)
  Why: Few-shot examples act as a template, so write each answer exactly as you want replies, down to tags, length and tone.
  How to verify: Put an example answer beside a live reply: same structure, same markup.
- [ ] Mark examples as illustrations
  Why: Wrap them in tags and say they show the style only, so their names and facts stay out of real replies.
  How to verify: Run inputs unlike any example: no names, numbers or topics from the examples turn up.
- [ ] Balance labels and their order
  Why: Models lean toward labels that are common or come last, so spread labels evenly and vary which one ends the set.
  How to verify: Count each label across the examples: no label dominates, and the final example does not carry the most common one.

Stop if:
- If replies copy names or facts from your examples: swap in varied examples before going on

Move on when examples cover the main input kinds and replies no longer copy their surface.

## 4. Before the prompt ships

It works on your favourite cases and is about to meet real users.

Must:
- [ ] Run the real input set
  Why: Read every reply against your pass bar and note how each failure fails, which practitioners call error analysis.
  How to verify: A table lists each input with pass or fail, and the **pass rate** meets the bar you set.
- [ ] Version the prompt like code (source 12)
  Why: Keep it in the repo or a versioned prompt registry, so each change has an author, a reason and a rollback.
  How to verify: Production loads a **named prompt version**, and its history shows who changed it and why.
- [ ] Test on every target model (only if: Several models or providers or A fallback model as backup)
  Why: Run the same input set on each model, since they differ in how literally they follow rules and formats.
  How to verify: The results table has a column per model, and every column meets the pass bar.

Don't forget these too:
- [ ] Throw awkward inputs at it
  - [ ] Empty or one-word input
  - [ ] Very long input
  - [ ] Another language
  - [ ] Typos and slang
  - [ ] Off-topic request
  - [ ] Request to break the rules
  Why: Add a few cases of each listed kind to the input set.
  How to verify: Each listed kind has cases in the set, and each gets a sensible reply.
- [ ] Run each case several times (source 13)
  Why: Replies vary between runs, so repeat each case a few times before trusting a pass.
  How to verify: Shaky cases are marked, and no must-pass case fails on any run.
- [ ] Set temperature or effort deliberately
  Why: Use low temperature for extraction and sorting where the model allows it; reasoning models often fix it, so tune reasoning effort instead.
  How to verify: Config beside the prompt version names the temperature or reasoning effort, chosen for this job rather than left at default.
- [ ] Keep per-model tweaks separate (only if: Several models or providers or A fallback model as backup)
  Why: Share one core prompt and keep each model's small adjustments in a marked section, so a fix lands everywhere at once.
  How to verify: Diff two models' final prompts: only a short, labelled section differs.
- [ ] Avoid provider-only prompt features (only if: Several models or providers)
  Why: Prefilled replies, special tags and cache markers differ by provider; keep them in the code that calls each one.
  How to verify: The shared prompt text has no provider-specific syntax in it.
- [ ] Price the prompt per call
  Why: Multiply prompt and typical reply tokens by the model's prices; long examples and documents are paid on every request.
  How to verify: A cost per call figure is written in the change, and it fits the feature's budget.
- [ ] Log prompt version per request
  Why: Record the prompt version and model with every call, so a drop in quality traces to a change.
  How to verify: Open one production log entry: it shows the prompt version and the model id.

Stop if:
- If testers can make it drop its rules: pause launch and hand it to your security reviewer
- If replies pass on your set but fail on real traffic: hold the launch and widen the set

Move on when the prompt meets the pass bar on real and awkward inputs and every call logs its version.

## 5. Every time the prompt changes (every time)

A bug report, a new case or a model upgrade means an edit to the prompt.

Must:
- [ ] Rerun every old case
  Why: A fix for one case often shifts others; the **full set** shows the trade before users do.
  How to verify: After the edit the pass rate is equal or higher, and no earlier must-pass case now fails.

Don't forget these too:
- [ ] Turn each bug into a case
  Why: Add the failing input to the set before fixing, so it stays fixed in every later round.
  How to verify: The new case is in the set and fails on the previous prompt version.
- [ ] Change one thing at a time
  Why: Make one edit, rerun, keep it only if results improve, then make the next.
  How to verify: Each prompt commit holds one change with its before and after results.
- [ ] Read the whole prompt for clashes
  Why: When two rules can both apply, say which one wins rather than leave the model to choose.
  How to verify: For each new rule, find any older rule it could clash with; the prompt states which comes first.
- [ ] Cut rules that no longer help
  Why: Try removing old patches now and then, above all after a model upgrade; a block a newer model no longer needs can hurt.
  How to verify: With the rule removed, the pass rate on the full set stays the same.
- [ ] Retest when the model changes
  Why: A new model version can read the same prompt and settings differently, so rerun the full set and retune reasoning effort before switching.
  How to verify: The upgrade change carries a fresh results table for the new version.

Stop if:
- If fixing one case breaks two others: revert the edit and rethink the rule

Move on when the edited prompt passes the old cases and the new one, and the change is committed.

Not covered: Parsing and validating replies lives in the structured output checklist, building and scoring a test set in the eval set one, and deeper injection defence in the prompt injection checklist.

Last reviewed: 2026-10-12

Sources:
1. Anthropic: Define success criteria and develop tests: https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
2. Anthropic: Model IDs and versions: https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions
3. OWASP: LLM Prompt Injection Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html
4. Anthropic: Context windows: https://platform.claude.com/docs/en/build-with-claude/context-windows?38d7aa68_page=5&fcdaa149_page=1&query=deliverability
5. OpenAI Cookbook: How to use guardrails: https://developers.openai.com/cookbook/examples/how_to_use_guardrails
6. Anthropic: Reduce hallucinations: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
7. OWASP GenAI: LLM01 Prompt Injection: https://genai.owasp.org/llmrisk/llm01-prompt-injection/
8. OpenAI: Prompt caching guide: https://platform.openai.com/docs/guides/prompt-caching
9. OpenAI Help: Best practices for prompt engineering: https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api
10. AWS Security Blog: Designing for system prompt leakage: https://aws.amazon.com/blogs/security/designing-for-the-inevitable-system-prompt-leakage-and-mitigations-in-generative-ai-applications
11. Anthropic: Claude prompting best practices, examples: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
12. LangChain: LangSmith, manage prompts with tags and versions: https://docs.langchain.com/langsmith/manage-prompts
13. ICLR 2026: repeated LLM trials per question: https://proceedings.iclr.cc/paper_files/paper/2026/file/f04edfc65463d020629673a4bc4c58e7-Paper-Conference.pdf

From CheatGrid: https://www.cheatgrid.com/checklists/ai-engineer/writing-a-prompt-for-an-app-feature?utm_source=github&utm_medium=listing&utm_campaign=writing-a-prompt-for-an-app-feature
