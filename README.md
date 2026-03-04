
Starting point for your own agent development using the [Embabel framework](https://github.com/embabel/embabel-agent).

The Project is based on the 
- [Agents that extract and use preferences from conversations](https://medium.com/embabel/agents-that-extract-and-use-preferences-from-conversations-7b22cca9abb3)

- [Context Engineering Needs Domain Understanding](https://medium.com/@springrod/context-engineering-needs-domain-understanding-b4387e8e4bf8)

Uses Spring Boot 3.5.10 and Embabel 0.3.3.

Add your magic here!

Illustrates:

- Embabel Domain Injected Context Engineering (DICE)
- Dice can be used to manage extracted user preferences and use them to enrich your LLM context.

# Running

Run the shell script to start Embabel under Spring Shell:

```bash
./scripts/shell.sh
```
# DICE (Domain Injected Context Engineering)

DICE supplements context engineering by emphasizing the importance of a domain model to help structure context, and considering LLM outputs as well as inputs.

Despite their seductive ability to work with natural language, LLMs become safer to use the more we add structure to inputs and outputs.

DICE helps LLMs converse in the established language of our business and applications.

The following diagram gives an overview of the Embabel design for the proposition pipeline. The PropositionPipelineController is a Spring MVC Rest controller that exposes an endpoint for extracting propositions from the provided text. The MemoryController has endpoints for listing and searching the stored propositions, i.e., the memory. It also provides endpoints for creating and deleting propositions.

The PropositionPipeline orchestrates the proposition extraction. It uses the extractor to find propositions, and the revisor to determine whether we are dealing with a new proposition or an update to an existing one. Propositions are stored or updated in the repository.

Design for the proposition pipeline

<img width="1173" height="835" alt="image" src="https://github.com/user-attachments/assets/093dfb50-d14e-4985-96fa-d33b13845cd2" />

# Integrate the extraction tool with the conversation
Now that I have a pipeline to extract preferences from text, I want to use this to extract user preferences from the conversation. The extraction begins by listening to an event I discussed a few sections back. After the user responds to a request, the conversation is published as an event.

The ConversationAnalysisRequestEvent contains the conversation and the KnowledgeUser. I wrote the ConversationPropositionExtraction class using the impromptu sample code. This class listens for the event and uses several components to extract entities and propositions from the conversation. The entities to extract are defined in the DataDictionary. With the help of the ChunkHistoryStore, re-analysing the same chunk is prevented. After extraction, the propositions and entities are stored in the PropositionRepository and the EntityRepository.

Internally, the PropositionPipeline discussed in the previous section is used for the extraction of the propositions. The following diagram gives an overview of the discussed components.

<img width="1173" height="656" alt="image" src="https://github.com/user-attachments/assets/c7153de1-9a41-44e9-9e1f-39f96cdb78ae" />

With user preferences extracted as propositions from the conversation, we only need to use the preferences in the context of the agent.

# Inject user preferences into the context of the Agent
Propositions are exposed through a memory component. The Memory needs the PropositionRepository and a MemoryProjector. This projector translates proposals as semantic memory (facts), procedural memory (preferences/rules) or episodic memory (events).The memory is exposed as a reference to the LLM call. 
