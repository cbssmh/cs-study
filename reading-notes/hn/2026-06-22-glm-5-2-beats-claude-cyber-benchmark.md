# 1. Title



We Have Mythos at Home: GLM-5.2 Beats Claude in Cyber Benchmarks



---



# 2. Source



- **Author / Organization:** Katie Paxton-Fear, Seth Jaksik, Brenden Noblitt, Erik Buchanan / Semgrep

- **Link:** https://semgrep.dev/

- **Date:** 2026-06-22



---



# 3. One-line Summary



GLM-5.2 demonstrated that a low-cost open-weight model can outperform Claude Code on IDOR vulnerability detection under identical prompting, while Semgrep argues that the surrounding harness contributes even more to performance than the model itself.



---



# 4. Key Points



- Semgrep benchmarked multiple LLMs on **IDOR (Insecure Direct Object Reference)** detection.

- GLM-5.2 achieved **39% F1**, outperforming Claude Code in the prompt-only setup.

- Semgrep's proprietary Multimodal harness reached **61% F1** with GPT-5.5 and **53%** with Opus 4.8.

- The benchmark intentionally isolated **model capability vs. harness capability**.

- GLM-5.2 was evaluated without endpoint discovery or navigation assistance.

- GLM-5.2 offers significantly lower inference cost (~$0.17 per vulnerability found).

- Open-weight models remain behind specialized security pipelines but have narrowed the gap substantially.

- The largest performance improvement came from workflow orchestration rather than switching models.

- Authors caution that conclusions are limited to one vulnerability class (IDOR) and one benchmark.



---



# 5. Deep Dive (Structured Understanding)



## Problem



LLM-based vulnerability detection quality depends on both:



- the underlying language model

- the surrounding agent/harness



Most discussions compare models while ignoring how much infrastructure guides them.



---



## Approach



Semgrep held constant:



- identical IDOR dataset

- identical evaluation metric (F1)

- identical prompt



and varied:



- LLM

- execution harness



Three categories were compared:



1. Semgrep Multimodal (custom security harness)

2. Claude Code SDK

3. Prompt-only Pydantic AI harness for open-weight models



---



## Key Insight



The benchmark suggests **workflow engineering matters more than raw model quality**.



A well-designed harness:



- discovers endpoints

- filters relevant context

- guides reasoning



This produced much larger gains than simply upgrading the base model.



At the same time, GLM-5.2 demonstrated that modern open-weight models have become competitive enough to outperform frontier coding agents in some specialized tasks.



---



## Result / Impact



Performance ranking:



| Configuration | F1 |

|---------------|----|

| Semgrep Multimodal + GPT-5.5 | 61% |

| Semgrep Multimodal + Opus 4.8 | 53% |

| GLM-5.2 | 39% |

| Claude Code | 37% |

| MiniMax M3 | 23% |

| Kimi K2.7 | 22% |



The benchmark shifts attention from **"Which model?"** toward **"Which workflow?"**



---



# 6. Why It Matters



This reflects a broader industry transition:



- Open-weight models are rapidly approaching closed frontier models on practical engineering tasks.

- Competitive advantage is increasingly moving from foundation models to **agent systems, orchestration, retrieval, and tooling**.

- Organizations may obtain comparable security capability while reducing inference cost and maintaining on-premise deployment.



---



# 7. Critical Analysis



- The benchmark evaluates only **IDOR detection**, which may not generalize to broader security research.

- Semgrep compares its proprietary harness against relatively simple harnesses, making it difficult to isolate pure model capability.

- GLM-5.2 was not evaluated inside Semgrep's own harness, leaving unanswered how much further it could improve.

- Cost comparisons emphasize GLM but provide limited economic comparison across all models.

- The benchmark measures vulnerability detection rather than exploit generation, code repair, or end-to-end offensive capability.



---



# 8. Connections



### 1. Agent Engineering



Supports the growing consensus that agent architecture and orchestration increasingly dominate raw model improvements.



### 2. Open-weight LLM Trend



Joins recent advances from DeepSeek, Qwen, MiniMax, and Kimi demonstrating that open-weight models are closing the capability gap.



### 3. AI Security Automation



Aligns with the movement toward autonomous AppSec pipelines combining:



- static analysis

- LLM reasoning

- repository navigation

- workflow automation



rather than relying on standalone chat models.



---



# 9. Keywords



- GLM-5.2

- Open-weight LLM

- Semgrep

- IDOR

- Application Security

- Vulnerability Detection

- AI Harness

- Agent Engineering

- Static Analysis

- Cybersecurity



---



# 10. TL;DR



- GLM-5.2 outperformed Claude Code on Semgrep's IDOR benchmark while costing far less.

- The benchmark argues that harness design contributes more to performance than model selection.

- Open-weight models are becoming practical alternatives for production security tooling, especially when combined with strong orchestration.
