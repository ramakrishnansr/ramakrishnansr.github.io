# Lab: The Censor!

This lab can vary in difficulty depending on your performance in previous assignments. You have two options: write the code yourself or utilize the tools from your previous homeworks. Our mission is to act as a censor (you get to be the "bad guy" in this scenario), ensuring that certain websites are blocked. 
Our motto: "You shall not pass!" Let's dive into the task.

We will be focusing on censoring the following **organizations** (Yes, we are not talking about websites, but organizations):
- Google
- Facebook
- Ram (Yes, the professor; for this lab, assume he has only one webpage, which is this one)

## Evaluation Criteria

Your grade will be based on how you answer the following questions and implement your solutions:

- When censoring Google, do you only block www.google.com?
- Implement at least 2 different ways you, as a censor, will detect someone connecting to these organizations
- What other encryption methods for achieving private communications have you learned? Do you think they'll bypass your censorship mechanisms? Block them! (At least one)
- Are there any exceptions you should consider? (Look closely at the google.com case). Let's say you are a small country, of course you want to censor, but it's one thing to censor and another thing to cause issues for finance and business in your country. For this task, the most important thing is that you support your decision.
- **Bonus Points (2)**: Are there other methods to perform censorship? Use your creativity.

To clarify, by the end of this lab, you should be able to detect when a browser attempts to access these sites. While it's not necessary to actually block the communication, detection is crucial.

**Bonus Points (8)**: Implement a mechanism to actually block the communication from a client! This might require setting up a more complex environment, such as a VM acting as a proxy/router. Just an idea!