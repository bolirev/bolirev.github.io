---
title: "LLM, Tools and the Hanoi towers: When tools improve puzzle resolutions … marginally"
categories:
  - Blog
tags:
  - AI
  - LLM
  - tools
---

The read of the [illusion of thinking](https://machinelearning.apple.com/research/illusion-of-thinking) (from Apple’s research) found reasonance in my own fun in problem solving. Once a certain problem complexity has been reached, I give up when not equipped with the right tools. So it lead me to this question:

- Does LLM solves puzzle better when provided with the right tools?

- Does thinking AI are able to pick up the right tools to solve it?

In this article, I wanted to explore the first question based on reading about [Model Context Protocol (MCP)](https://cobusgreyling.medium.com/using-langchain-with-model-context-protocol-mcp-e89b87ee3c4c), the [hanoi algorithm](https://medium.com/@davidlfliang/intro-python-algorithms-tower-of-hanoi-a1ea8e790f59), and the Illusion of thinking article.

After seting up an MCP server for puzzle resolution, generated the prompts based on the Illusion of thinking with minor variations to include tool use, I obtained the following success rate:

![Figure: Success rates for solving the Tower of Hanoi puzzle across different configurations and models. The plot shows three subplots representing different experimental conditions: (1) No MCP, No Pseudocode; (2) No MCP, With Pseudocode; and (3) With MCP, No Pseudocode. Each subplot displays success rates as a function of the number of disks (3–10) for two AI models: o4-mini (blue) and gpt-4.1-mini (red). The trends suggest that both models struggle more with larger disk counts, particularly in configurations without MCP assistance. Interestingly tool usage benefits most the non thinking models. However all models failed for 8+ disks.](/assets/images/posts/llm-tools-and-the-hanoi-towers-when-tools-improve-puzzle-resolutions-marginally/01-b0ff6627fbbd.png)
*Figure: Success rates for solving the Tower of Hanoi puzzle across different configurations and models. The plot shows three subplots representing different experimental conditions: (1) No MCP, No Pseudocode; (2) No MCP, With Pseudocode; and (3) With MCP, No Pseudocode. Each subplot displays success rates as a function of the number of disks (3–10) for two AI models: o4-mini (blue) and gpt-4.1-mini (red). The trends suggest that both models struggle more with larger disk counts, particularly in configurations without MCP assistance. Interestingly tool usage benefits most the non thinking models. However all models failed for 8+ disks.*

The more help the models receive the better it performed. But I did not expected the performance to drop to zero for 8+ disks when the tool was provided. After all the tool is providing the solution to the problem.

I was wondering whether the output of the tool could not be comprehended by the model. The python function implemented the solution returned a nx3 lists describing a list of move as disk_id, from_peg, to_peg. But the tool returned the flatten list…

    ["1", "0", "1", "2", "0", "2", "1", "1", "2"]

The logical next step was to transform the output to return a list of dictionary.

    ["{
      \\"move_nb\\": 0
      \\"disk\\": 1
      \\"from_peg\\": 0
      \\"to_peg\\": 1
    }", "
      \\"move_nb\\": 1
      \\"disk\\": 2
      \\"from_peg\\": 0
      \\"to_peg\\": 2
    }", "
      \\"move_nb\\": 2
      \\"disk\\": 1
      \\"from_peg\\": 1
      \\"to_peg\\": 2
    }"]'

![Figure: Success rates for solving the Tower of Hanoi puzzle with verbose tool use (i.e. v2). The two AI models previously used as color-coded as in previous figure: o4-mini (blue) and gpt-4.1-mini (red). The trends suggest that both models struggle more with larger disk counts. Interestingly the verbose tool improve the performance for n=6, but not the one above.](/assets/images/posts/llm-tools-and-the-hanoi-towers-when-tools-improve-puzzle-resolutions-marginally/02-a3af5831140f.png)
*Figure: Success rates for solving the Tower of Hanoi puzzle with verbose tool use (i.e. v2). The two AI models previously used as color-coded as in previous figure: o4-mini (blue) and gpt-4.1-mini (red). The trends suggest that both models struggle more with larger disk counts. Interestingly the verbose tool improve the performance for n=6, but not the one above.*

#### Conclusion and open questions

Our experimental analysis of AI models solving the Tower of Hanoi puzzle revealed:

**Tool Usage Impact:**

1.  Tool usage improved more the performance of the non-thinking model that its thinking alternative
2. The verbose tool implementation (v2) showed improved performance for 6-disk puzzles but did not extend to larger disk counts

**Complexity Challenge:**

Success rates dropped dramatically reaching 0% for 8+ disks, even when tool were used to solve the problem

#### Following thoughts

1. **Performance Degradation**: Why do both models exhibit such a dramatic drop in performance between 6 and 8 disks? This suggests a fundamental limitation in the models’ reasoning capabilities for this type of recursive problem.

2. **Tool Effectiveness**: While verbose tool use improved 6-disk performance, why does this improvement not scale to larger disk counts? Perhaps a more interactive approach with validation procedures would help to solve the problem accurately -\> aka more tools.

#### Implementation:

[GitHub - bolirev/ai-hanoi-mcp](https://github.com/bolirev/ai-hanoi-mcp)
