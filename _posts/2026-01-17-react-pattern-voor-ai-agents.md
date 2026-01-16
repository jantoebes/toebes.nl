---
layout: post
title: "Het ReAct Pattern voor AI Agents"
date: 2026-01-17
---

Het **ReAct pattern** (Reasoning + Acting) is een krachtige benadering voor het bouwen van intelligente AI agents die kunnen redeneren én handelen.

## Wat is ReAct?

ReAct combineert twee fundamentele capaciteiten:

- **Reasoning**: De agent denkt na over het probleem en plant stappen
- **Acting**: De agent voert acties uit met tools en observeert resultaten

Dit cyclische proces zorgt ervoor dat agents adaptief en efficiënt complexe taken kunnen oplossen.

## De ReAct Loop

Het basispatroon ziet er als volgt uit:

```javascript
async function reactLoop(task, maxIterations = 10) {
  let observation = `Task: ${task}`;
  
  for (let i = 0; i < maxIterations; i++) {
    // 1. THOUGHT: Agent redeneert over de situatie
    const thought = await agent.think(observation);
    console.log(`Thought: ${thought}`);
    
    // 2. ACTION: Agent kiest en voert een actie uit
    const action = await agent.chooseAction(thought);
    console.log(`Action: ${action.name} - ${action.input}`);
    
    // 3. OBSERVATION: Bekijk het resultaat van de actie
    observation = await executeAction(action);
    console.log(`Observation: ${observation}`);
    
    // 4. Check of taak voltooid is
    if (await agent.isTaskComplete(observation)) {
      return observation;
    }
  }
  
  throw new Error('Max iterations reached');
}
```

## Praktisch Voorbeeld

Stel je voor dat een agent informatie moet opzoeken en samenvatten:

**Iteration 1:**
- **Thought**: "Ik moet eerst de Wikipedia API gebruiken om informatie op te halen"
- **Action**: `search_wikipedia("Agentic AI")`
- **Observation**: "Found article about autonomous AI systems..."

**Iteration 2:**
- **Thought**: "De informatie is opgehaald, nu moet ik de belangrijkste punten extraheren"
- **Action**: `extract_key_points(article_text)`
- **Observation**: "Key points: 1. Autonomous decision making, 2. Goal-directed behavior..."

**Iteration 3:**
- **Thought**: "Ik heb genoeg informatie, tijd om een samenvatting te maken"
- **Action**: `generate_summary(key_points)`
- **Observation**: "Summary created successfully"

## Implementatie in Python

Hier is een complete implementatie:

```python
from typing import List, Dict, Any
import openai

class ReActAgent:
    def __init__(self, tools: List[callable]):
        self.tools = {tool.__name__: tool for tool in tools}
        self.history = []
    
    async def think(self, observation: str) -> str:
        """Laat de agent redeneren over de huidige situatie"""
        prompt = self._build_prompt(observation)
        
        response = await openai.ChatCompletion.acreate(
            model="gpt-4",
            messages=[
                {"role": "system", "content": "You are a helpful AI agent."},
                {"role": "user", "content": prompt}
            ]
        )
        
        return response.choices[0].message.content
    
    async def choose_action(self, thought: str) -> Dict[str, Any]:
        """Parse de gedachte en kies een actie"""
        # Extract action from thought (simplified)
        for tool_name in self.tools.keys():
            if tool_name in thought.lower():
                return {
                    "name": tool_name,
                    "input": self._extract_input(thought)
                }
        
        return {"name": "final_answer", "input": thought}
    
    def _build_prompt(self, observation: str) -> str:
        """Bouw de prompt met geschiedenis en beschikbare tools"""
        tools_desc = "\n".join([
            f"- {name}: {tool.__doc__}"
            for name, tool in self.tools.items()
        ])
        
        return f"""
Observation: {observation}

Available tools:
{tools_desc}

History:
{self._format_history()}

What should you do next? Think step by step.
"""
    
    def _format_history(self) -> str:
        return "\n".join([
            f"{i+1}. {step}"
            for i, step in enumerate(self.history[-5:])
        ])
    
    def _extract_input(self, thought: str) -> str:
        # Simplified extraction logic
        return thought.split("with input:")[-1].strip()
```

## Tools voor de Agent

De kracht van ReAct komt van de tools die je beschikbaar maakt:

```python
def search_web(query: str) -> str:
    """Search the web for information"""
    # Implementation here
    return f"Search results for: {query}"

def calculate(expression: str) -> str:
    """Perform mathematical calculations"""
    try:
        result = eval(expression)
        return f"Result: {result}"
    except Exception as e:
        return f"Error: {e}"

def save_to_file(content: str, filename: str) -> str:
    """Save content to a file"""
    with open(filename, 'w') as f:
        f.write(content)
    return f"Saved to {filename}"

# Initialiseer agent met tools
agent = ReActAgent(tools=[search_web, calculate, save_to_file])
```

## Voordelen van ReAct

1. **Transparantie**: Je kunt elke stap van de redenering volgen
2. **Flexibiliteit**: Agent past zich aan op basis van observaties
3. **Tool gebruik**: Combineert LLM reasoning met externe tools
4. **Foutafhandeling**: Kan herstellen van mislukte acties

## Uitdagingen

- **Token efficiency**: Elke iteratie kost tokens
- **Loop detection**: Agent kan in herhaling vervallen
- **Tool reliability**: Kwaliteit hangt af van beschikbare tools
- **Prompt engineering**: Goede prompts zijn cruciaal

## Volgende Stappen

In toekomstige posts ga ik dieper in op:

- Memory systemen voor langere contexten
- Multi-agent ReAct patterns
- Optimalisatie van prompt strategieën
- Error recovery en retry logic

Het ReAct pattern is een fundamentele bouwsteen voor moderne AI agents. Door redenering en actie te combineren, kunnen we systemen bouwen die echt autonoom complexe taken kunnen oplossen.

## Referenties

- [ReAct Paper](https://arxiv.org/abs/2210.03629) - Originele publicatie
- [LangChain ReAct](https://python.langchain.com/docs/modules/agents/agent_types/react) - Implementatie
- [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) - Praktisch voorbeeld

Heb je vragen of ideeën over ReAct patterns? Laat het me weten!
