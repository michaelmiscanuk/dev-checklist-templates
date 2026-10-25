## Writing a prompt for an app feature

When: You are adding a new LLM-powered feature, or rewriting the prompt behind one, and want it to hold up on real user input.

Before you start: What text besides your own goes into the prompt? (any that apply) Text users type in / Retrieved documents or files / Tool or API results / None of these. Checks marked "only if" apply only to that answer.
Before you start: Which of these apply to the models behind the prompt? (any that apply) Several models or providers / A fallback model as backup / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you write the prompt

You know the feature you want, and no prompt text exists yet.

Must:
- [ ] Gather real user inputs first

Stop if:
- If nobody can say what a good reply looks like: ask the product owner before writing

Move on when you have the job in one sentence, a set of real inputs and a written pass bar.

### 2. While drafting the prompt

You are writing the system prompt and the template that wraps each request.

Must:
- [ ] Say what to do outside scope
- [ ] Give a way out when unsure
- [ ] Fence outside text in delimiters (only if: Text users type in or Retrieved documents or files or Tool or API results)

Stop if:
- If the prompt must hold personal or secret data: ask your security or privacy lead first
- If every new complaint needs another rule: stop and ask whether the job is too broad

Move on when A full prompt runs end to end on a handful of real inputs.

### 3. When you add examples

Rules alone miss some cases, and you want to show the model what good looks like.

Must:
- [ ] Vary examples on every surface

Stop if:
- If replies copy names or facts from your examples: swap in varied examples before going on

Move on when examples cover the main input kinds and replies no longer copy their surface.

### 4. Before the prompt ships

It works on your favourite cases and is about to meet real users.

Must:
- [ ] Run the real input set
- [ ] Version the prompt like code
- [ ] Test on every target model (only if: Several models or providers or A fallback model as backup)

Stop if:
- If testers can make it drop its rules: pause launch and hand it to your security reviewer
- If replies pass on your set but fail on real traffic: hold the launch and widen the set

Move on when the prompt meets the pass bar on real and awkward inputs and every call logs its version.

### 5. Every time the prompt changes (every time)

A bug report, a new case or a model upgrade means an edit to the prompt.

Must:
- [ ] Rerun every old case

Stop if:
- If fixing one case breaks two others: revert the edit and rethink the rule

Move on when the edited prompt passes the old cases and the new one, and the change is committed.

---
Must checks only. Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/writing-a-prompt-for-an-app-feature.md). Interactive version: [Writing a prompt for an app feature on CheatGrid](https://www.cheatgrid.com/checklists/ai-engineer/writing-a-prompt-for-an-app-feature?utm_source=github&utm_medium=listing&utm_campaign=writing-a-prompt-for-an-app-feature).
