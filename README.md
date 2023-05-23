# Researcher Resolver

## Introduction

Researcher Resolver is a software project that utilizes prompt engineering techniques to improve the response quality of large language models. It aims to bring out a deeper level of understanding and reasoning from the models, going beyond simple responses. The approach implemented here is primarily influenced by the insights from the video *[GPT 4 is Smarter than You Think: Introducing SmartGPT](https://www.youtube.com/watch?v=wVzuvf9D9BU)* which utilizes concepts and prompts from the following research papers:

- [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2202.11585)
- [Reflexion: an autonomous agent with dynamic memory and self-reflection](https://www.semanticscholar.org/paper/Reflexion%3A-an-autonomous-agent-with-dynamic-memory-Raghu-Bernstein/6b56f8de8b9e2a8c5ba816e28e8758e0c66f9775)
- [DERA: Enhancing Large Language Model Completions with Dialog-Enabled Resolving Agents](https://www.semanticscholar.org/paper/DERA%3A-Enhancing-Large-Language-Model-Completions-Bosselut-Clark/cfda18a96d7f0b14a2bb93b8b2be4d2732f249fe)

## Prompts

The core of the solution is a series of carefully engineered prompts that drive the language model's responses. These prompts are designed to engage the language model in more in-depth reasoning and exploration of the question at hand, enhancing the quality and thoroughness of the output.

In this example, the question is asked three times to get a diversity of responses for the researcher to then evaluate.   

```
question_and_prompt = f"{question} Let's work this out in a step-by-step way to ensure we have the right answer."

```

To mitigate cost issues while still leveraging the strategies outlined here, you can manually add prompts chatgpt. For example, you can append `"let's think step by step"` to the end of your question.

## Researcher step

- The researcher prompts asks the language model to find flaws in each of the three original responses and the original responses are included in the request.

```python
def researcher(response_1, response_2, response_3):
    researcher_prompt = f"""
    You are a researcher tasked with investigating the 3 response options provided.  
    List the flaws and faulty logic of each answer option. Let's work this out in a 
    step by step way to be sure we have all the errors.
    """
    researcher_plus_responses = researcher_prompt + "response 1: " + response_1 + "response 2: " + response_2 + "response 3: " + response_3
    result = send_message(researcher_plus_responses)
    return result
```

## Resolver step

- The resolver prompt is asked to choose the best answer and improve upon it.

```python
def resolver(researcher_response, response_1, response_2, response_3):
    resolver_prompt = f"""
    You are a resolver tasked with 1) finding which of the 3 answer options the 
    researcher thought was best. 2) improving that answer, and 3) Printing the improved
    answer in full.  Let's work this out in a step by step way to be sure
    we have the right answer.
    """
    resolver_prompt_plus_responses = resolver_prompt + "researcher criticisms: " + researcher_response + "\n orginal responses: \n" + "response 1: " + response_1 + "response 2: " + response_2 + "response 3: " + response_3
    result = send_message(resolver_prompt_plus_responses)
    return result
```

## Getting Started

### Prerequisites

You'll need the following to run Researcher Resolver:

- Python 3.6 or above (Tested on 3.11)
- Jupyter Lab or Jupyter Notebook

### Installation

Install the OpenAI Python package:

- `pip install openai`

You'll also need to create a `.env` file in your project root with your OpenAI API key in the format `OPENAI_API_KEY=`

## Usage

Run the provided Jupyter notebook. It will prompt you for a question via standard input if you run all of the cells.

## Limitations and Current Problems

This approach has a few limitations:

- **Cost:** As this solution uses more tokens than a single request, the cost can be around 10x higher.
    - (3x) Initial question is asked 3 times
    - (3x) Researcher step is given all of the question responses to critically evaluate
    - (3x) Resolver step is given all of the question responses and the researcher criticism
- When dealing with long queries and/or long responses, asking the question a single time is preferable to this approach, which would exceed context limits earlier.
- **Time:** Our approach requires more time to execute due to additional API calls.
- **Compatibility:** The prompt engineering techniques we use are not as effective with smaller or non-instruction-tuned models.

## When is this useful ?

- The general approach is useful to understand.  Just adding the simple statement “Lets think step by step”, improves the quality of results when interacting with language models GPT 3.5, GPT 4, and likely most large RLHF tuned models.  Understanding the prompts and flow used here can help when interacting with language models through a chat interface.
- When you don’t want to babysit the response in ChatGPT, use this code.
    - Using GPT 4 in chatgpt can be slow.  If the question is likely to need follow up to coax a better answer, then it will be time saving for the end user to use this approach and come back to just view the results later.  Even if you don’t ultimately choose to use the final output of the resolver step, having the multiple responses and the critiques of those responses can help guide you to a better next step.

## Future Directions

- Researcher Resolver
    - Asking each question from a different viewpoint to generate diverse answers.
        - scientist
        - researcher
        - journalist
        - ect.
    - Further experimenting with temperature settings and number of generations.
    - Evaluating the approach using standard metrics such as MMLU.
    - Add a web application or command line application
        - the jupyter notebook use is intentional to make it easy for other people run the code and experiment.  If this was to be used an an application, the interface of the jupyter notebook is not ideal.
- Other exploration
    - Implementing other prompt engineering techniques from recent research
        - Tree of Thought
        - Reasoning and Acting (REACT)
        - AutoGPT like subtasking
            - not for the purpose of having an agent act automatically, but for the idea of breaking down task into subtasks.  For large tasks, this seems like it can help a lot as it helps address limitations of the context window