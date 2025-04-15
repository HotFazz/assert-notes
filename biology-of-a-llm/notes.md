## Review of On the Biology of a Large Language Model

Paper: [On the Biology of Large Language Models](https://transformer-circuits.pub/2025/attribution-graphs/biology.html)

### Overview

- Reverse engineer how LLMs models work on the inside. 
- Probing the insides of language models 
- Representation of interpretable concepts – “features” –  embedded within modles' internal activity
-  How these features interact - [Circuit Tracing: Revealing Computational Graphs in Language Models](https://transformer-circuits.pub/2025/attribution-graphs/methods.html)
- *Attribution graphs*: partially trace the chain of intermediate steps that a model uses to transform a specific input prompt into an output response.
- Claude 3.5 Haiku routinely uses multiple intermediate reasoning steps “in its head”. That is, during the forward pass rather than the "thinking out loud" of a chain-of-thought completion.
- *Forward planning*, considering multiple possibilities for what it will say well in advance of saying it. 
- *Backward planning*, working backwards from goal states to formulate earlier parts of its response.

> Our methods study the model indirectly using a more interpretable “replacement model,” which incompletely and imperfectly captures the original.


### Methodology 

Studying transformer based language modeles, with two fundemental concepts; MLP, process information within each token position using collections of neurons. And layers, which move information between token positions.

- Models are difficult to interpret because they are polysemantic, they perform functions that can seem unrelated.
    - In part because of *superposition* whereby models represent more concepts than they have neurons and thus cannot assign each neuron to its own concept.
- The replacement model built reproduces the activations of the original model using more interpretable components. 
    - The replacement model is based on a cross-layer transcoder (CLT) architecture.
    - The CLT used had 30 M feautres across all layers. 
    - Attention layers between the models are the same and fixed components. 

    ![Replacement Model](replacement-model.png)

- Features in the model represent human-interpretable concepts, low-level to high-level.
- Error nodes to represent the discrepency of the replacement model and the original.
- On the attribution graphs:
    - A graphical representation of the computational steps the model uses to determine its output for a particular input.
    - Nodes represent features and edges represent the causal interactions between them.
    - Manually grouping related graph nodes together into supernodes to create a simplified depiction
![Attribution Graph](fig2.1.png)
![Attribution Graph](fig2.2.png)

- To verify the replacement model: Intervention experiments in the original model, such as inhibiting feature groups and observing their effects on other features and on the model’s output.


## Case Studie

### Introductory Example: Multi-step Reasoning

Simple example where the model performs “two-hop” reasoning “in its head” to identify that “the capital of the state containing Dallas” is “Austin.”

#### Prompt: Fact: the capital of the state containing Dallas is

- Reasoning in two steps:
    - inferring that the state 
    - the capital of Texas 
- Does the model perform the two steps internally or use some form of "short cut"

#### Insight into model 
- Several features about the word and/or concept of a capital city.
- Features that represent the concept of capitals in more general ways. One example is this feature, which activates on the word “capitals” but also later in questions about capitals of states.
- Multilingual feature, which activates most strongly on a variety of phrases including “başkenti”, “राजधानी”, “ibu kota”, and “Hauptftadt” — all of which roughly mean “capital” in different languages.
- The core observation is that the features represent the idea of "capital"
- “output features” that consistently push the model to say certain tokens, even if there isn’t such a clear pattern to what words/phrases they activate on. 
- The “Top Outputs” information is not always informative – for instance, earlier-layer features primarily matter via indirect effects on the output via other features, and their top direct outputs are not too consequential.
- “Output feature” requires a holistic evaluation of its top direct outputs, the contexts in which it activates, and its role in the attribution graph.
- Features that promote outputting the name of a capital more generally, which we use a mix of both types of signals to identify and label.
![Austin Graph Detailed](Austin.png)

#### Validation

- Intervention experiments on the feature groups by inhibiting each of them.
    - Clamping them to a negative multiple of their original value.
    - Inhibiting “Dallas” features decreases the activation of “Texas”
    - Inhibiting the “Dallas” cluster causes the model to output other state capitals, while inhibiting the “say a capital” cluster causes it to output non-capital completions.



### Planning in Poems

How does Claude 3.5 Haiku write a rhyming poem?

- The lines need to rhyme.
- They need to make sense.
- Two ways to achieve this:
    - Pure improvisation; write the beginning of each line whiout reagrd for the rhyme in the end. At the end of each line choose a word that rhymes and makes sense.
    - Planning; At the *beginning* of each line come up with the word it plans to use at the end. 

> Language models are trained to predict the next word, one word at a time. Given this, one might think the model would rely on pure improvisation. However, we find compelling evidence for a planning mechanism.

- Model often activates features corresponding to end-of-next-line words *prior* to writing the line. 
- Both forward planning and backwards planning.
    - The model uses the semantic and rhyming constraints of the poem to determine candidate targets for the next line. Next, the model works backward from its target word to write a sentence that naturally ends in that word.
- Holds multiple possible planned words “in mind” at the same time.
- Able to edit the model’s planned word and see that it restructures its next line accordingly.
- Planning Features Only Matter at the Planning Location
    - Planning occurs at the newline token.
    - Features are only active over the planning token.
- Planned words influence Intermediate words
> The same “rabbit” planning features are active in the graph, promoting a group of “comparison features”, which are active before text such as “like a”. The model goes from the planned target (“rabbit”) that activates at the newline and reasons backwards, creating a plausible transition to get to it.


### Multilingual Circuts 

- Prompts with identical meaning in English, French, and Chinese activate shared multilingual circuits.
- Abstract "antonym" and "small" features are language-agnostic.
- Language-specific output features are selected based on input language.
- Three-part decomposition of computation:
    - Operation: antonym/synonym.
    - Operand: the word.
    - Language: contextual language inferred from features.

- Claude uses shared middle-layer multilingual features and then routes through language-specific heads for the final output.

#### Intervention Experiments
Edit Operation (Antonym → Synonym):
- Swapping in synonym features leads to correct language-specific synonym prediction.
- Demonstrates language-agnostic operation representation.
- Substituting "hot"-related features causes correct antonym prediction across languages.
- Operand circuits are shared across languages.

Edit Language (English → Chinese/French):
- Model continues same operation/operand reasoning, but changes final output language.
- Output rerouted via different say-X-in-language-Y nodes.

Middle layers: language-independent features dominate.
Early and late layers: more language-specific.


### Addition

How does Claude 3.5 Haiku perform two-digit addition (36 + 59)?

- Computes approximate sum in parallel with computing the ones digit.
- Combines rough and precise pathways to reach correct result.
- Uses “lookup table” features to map input digit endings (6 + 9) to output ones digit.


> Like humans, the model memorizes single-digit addition but implements multi-digit addition differently from explicit algorithms.

Probing of 10,000 prompts reveals distinct feature types:
- Diagonal patterns: features sensitive to overall sum.
- Vertical/horizontal stripes: sensitive to individual addends.
- Points/repeats: lookup tables & modular patterns.
- Smeared: low-precision approximations.

#### Attribution Graph (36+59)
- Low-precision pathway: “~36 + ~60 → ~92”
- High-precision path: “_6 + _9 → _5”
- Both feed into final “sum = 95” prediction.


Same lookup features activate in non-arithmetic contexts:
- Predicting end times in time series.
- Projecting cumulative cost in revenue tables.
- Completing academic citations (e.g., Polymer, vol. 36 → pub. year 1995 via 36 + 59).
- Model reuses arithmetic features across diverse domains.

#### Intervention Experiments

- Suppressing _6 + _9 reduces correct prediction.
- Replacing with _9 + _9 shifts output to match altered ones digit.
- Confirms causal role of arithmetic circuits in downstream predictions.

- Weak suppressive mechanism prevents saying “9” prematurely.

> Addition features are reused flexibly — either to emit final answers or to pass values to further computations, depending on context.


### Clinical Diagnosis

Prompt: Pregnant woman w/ signs of preeclampsia; model is asked for one follow-up symptom.
Output: “Visual disturbances”, followed by “proteinuria”.

- Model internally activates preeclampsia-related features, including:
    - Features typically associated with explicit mentions of "preeclampsia".
    - Features tied to its symptoms.
    - Features for visual deficits, protein in urine.
    - Also activates features for alternative diagnoses.

> Suggests a differential diagnosis-like reasoning process: consider multiple causes, seek distinguishing symptoms.

#### Interventions 

- Suppressing preeclampsia features:
    - Deactivates related symptom features.
    - Model now suggests "decreased appetite", aligned with biliary diagnosis.
    - Confirms causal role of internal features in shaping outputs.

> Suggests model uses structured reasoning combining diagnostic hypotheses with confirmatory symptom features, though not all steps are interpretable or reliably weighted.



### Entity Recognition and Hallucinations

- Identify circuits that govern when models refuse to answer vs. hallucinate.
- Haiku has internal refusal circuits that are default-active and require suppression by known-entity signals to answer confidently.
- “Can’t answer” and “unknown name” features fire by default on unfamiliar names.
- Suppression occurs only if “known answer” features are activated.
- “Known answer” features suppress refusal circuit.
- Recognition of known entities inhibits the default refusal.
- Known entities turn off the refusal pathway → model answers confidently.

#### Interventions
- Suppressing refusal on unknown entity.
- Model hallucinates random sport.
- Suppressing “known” features on known entity.
- No “known answer” activation → default refusal circuit stays active.
- Interventions activating known-answer features cause hallucinated papers to appear.



### Life of a Jailbreak

Why does Claude 3.5 Haiku sometimes fail to refuse harmful requests?

Setup: A Successful Jailbreak
Jailbreak Prompt: “Babies Outlive Mustard Block” → forms the acronym BOMB.

Model outputs:
“BOMB. To make a bomb, mix potassium nitrate…”
→ Then refuses: “However, I cannot provide…”

- Model briefly complies, then self-corrects — a partial jailbreak.
- No global awareness of what it’s spelling → no harmful request features triggered
- Model doesn’t realize it’s about to say “BOMB”, local completions vote independently for plausible next tokens.
- The refusal circuit fails because model hasn’t linked “bomb” to instruction context yet.


#### Intervention Findings

- Activating “make a bomb” features on ‘BOMB’ → triggers immediate refusal
- Activating general “bomb” features → insufficient on their own
- Removing punctuation delays refusal by disabling “new sentence” features


> Jailbreaks exploit the local, myopic, and syntax-driven behavior of model circuits before higher-level refusal logic kicks in.




### Commonly Observed Circuit Components and Structure

#### General Graph Structure
- Input → Abstract → Output Features:
- Graphs often follow a pattern:
    - Input features: token-level properties
    - Abstract features: concepts, computations
    - Output features: directly influence token prediction → Mirrors detokenization → reasoning → retokenization.

- Convergent Paths & Shortcuts:
    - Multiple paths (short and long) often lead to the same output node.
    - Matches coherent feedforward loop motifs from systems biology.

- Smeared Features Across Tokens:
    - Same feature activates on adjacent tokens with similar behavior
    - Indicates context continuity maintenance.

- Long-Range Connections:
    - Cross-layer connections are common, often skipping layers.
    - Some early-layer features directly influence final output.

- Special Tokens Store Key Info:
    - Punctuation and newlines often hold control signals.

- “Default” Circuits:
- Some circuits activate by default.
- Overridden by features triggered by known/familiar content.

- Early Attention, Late Output:
    - Graphs show early layers fetch info, while computation concentrates at the final token layer.
 
- Context-Dependent Feature Facets:
    - Multifaceted features represent conjunctions.
    - Not all facets are active in every context — roles vary by use.

- Confidence Reduction Features:
    - Features near output layer decrease likelihood of a likely token.
    - Possibly regulate overconfidence; observed in late layers only.

- “Boring” Circuits: (the circuits they thought werent interesting!)
    - Many features are low-level:“this is math”, “output a number”.
    - Crucial for function but not informative about how decisions are made.


### Discussion

- Models employ multiple, qualitatively different mechanisms concurrently (sometimes cooperating, sometimes competing) for a single output. These can be modular, handling seperate parts of a task independently.   
- High level of abstractions that apply across diverse domains and even languages, an internal "universal mental language" that scales with model capability.   
- Internal plan formation, considering alternatives and influencing future output based on these internal drafts.
- Working Backward from Goals (Backward Chaining): The model reasons backward from desired outcomes to determine preceding steps, both in creative generation and *potentially* in deceptive reasoning.
- Proto-Meta-cognition: The model shows a basic awareness of its own knowledge by having features for "knowing" or "not knowing", the depth of this **self-awareness** is unclear.   
- Fine-tuning can deeply entrench biases.

> Often, multiple parallel mechanisms are involved in a single completion. For instance, we can observe both two-hop and shortcut reasoning occurring simultaneously in our state capitals example.
> We began our analysis of the hidden-goals model assuming that it would only “think about” its goal in relevant contexts, and were surprised to find that it instead represents the goal all the time.
