# Group Relative Policy Optimization (GRPO)

Introduced by Shao et al. (2024) in *DeepSeekMath: Pushing the Limits of Mathematical
Reasoning in Open Language Models*.

Presented by Aarohi Srivastava on June 19, 2026.

## If PPO already works, why GRPO?

Over the past few weeks, we've discussed some approaches to LLM post-training:
- **RLHF** uses human preferences to train a reward model.
- **PPO** uses reinforcement learning to optimize a policy with respect to that reward model.
- **DPO** removes the reinforcement learning stage entirely and directly learns from preference pairs.

At this point, it is natural to ask: "If PPO works and DPO avoids reinforcement learning altogether, why do we need another method?"

The answer is that some tasks provide a form of supervision that differs from human preferences. In domains such as mathematics and coding, responses can often be evaluated automatically. Rather than asking humans which response is better, we can simply check whether the answer is correct.

This creates a setting in which reinforcement learning becomes particularly attractive. However, PPO remains expensive because it requires training and maintaining a critic alongside the policy itself.

The goal of GRPO is therefore not to replace PPO, but to simplify one of its central components: advantage estimation.

DeepSeekMath introduces **Group Relative Policy Optimization (GRPO)**, a PPO variant that removes the critic and instead estimates advantage by comparing responses within a group.

---

## PPO in one page

Recall that PPO training typically involves four components:

1. **Policy model**: the LLM being optimized
2. **Reward model**: assigns rewards to generated responses
3. **Critic**: predicts expected reward
4. **Reference model**: prevents the policy from drifting too far from its initial behavior

Given a prompt, the policy generates a response. The reward model assigns a reward, while the critic estimates how much reward was expected. PPO then computes an advantage:

$$
A = R - V
$$

where:

- $R$ is the observed reward
- $V$ is the critic's prediction

Intuitively:

- positive advantage → this response was better than expected
- negative advantage → this response was worse than expected

The policy is updated to make high-advantage responses more likely.

---

**[INSERT FIGURE 4 FROM PAPER HERE]**

*Figure 4 from DeepSeekMath compares the PPO and GRPO training pipelines.*

---

## The problem with PPO

PPO works well, but it comes with a cost.

The critic must be trained alongside the policy. In practice, this means maintaining and updating another large neural network during training.

This introduces several challenges:

- additional memory requirements
- additional computation
- instability due to critic errors
- more complicated training pipelines

DeepSeekMath asks a simple question:

> Can we estimate advantage without training a separate critic?

GRPO answers "yes."

---

## The core idea behind GRPO

Suppose we ask the model the same question multiple times.

**Prompt**

> Solve: $17 \times 24$

The model generates four responses:

| Response | Reward |
|-----------|----------|
| A | 1.0 |
| B | 1.0 |
| C | 0.0 |
| D | 0.0 |

Rather than asking:

> Was response A better than expected?

GRPO asks:

> How did response A compare to the other responses in the group?

The group itself becomes the baseline.

Instead of learning a separate estimate of expected reward, GRPO derives that estimate from the sampled responses.

This eliminates the need for a critic.

---

## Group-relative advantage

The simplest intuition is:

$$
A_i = R_i - \text{group average reward}
$$

Responses above the group average receive positive advantage.

Responses below the group average receive negative advantage.

This captures the main idea behind GRPO:

> reward responses that perform better than their peers.

The actual DeepSeekMath implementation goes one step further and normalizes rewards using the group's mean and standard deviation:

$$
\hat r_i
=
\frac{r_i-\text{mean}(r)}
{\text{std}(r)}
$$

This normalized quantity serves as the advantage signal used during optimization.

Intuitively:

- responses far above the group average receive strong positive updates
- responses far below the group average receive strong negative updates
- responses near the average receive relatively little update

### A worked example

Suppose the model generates four responses with rewards:

| Response | Reward |
|-----------|----------|
| A | 10 |
| B | 8 |
| C | 4 |
| D | 2 |

The average reward is:

$$
\mu = 6
$$

Relative to the group:

| Response | Relative Reward |
|-----------|----------|
| A | +4 |
| B | +2 |
| C | -2 |
| D | -4 |

GRPO therefore increases the probability of A and B while decreasing the probability of C and D.

The key idea is that no critic was needed to determine which responses were above or below expectation. The group itself provides the baseline.

---

## What actually happens during training?

The GRPO training loop looks roughly like:

1. Sample a prompt
2. Generate multiple responses
3. Score each response
4. Compute group-relative advantages
5. Update the policy
6. Repeat

Unlike DPO, the model is learning from its own generated trajectories.

Unlike PPO, no critic is trained.

---

## Why is GRPO particularly useful for reasoning?

GRPO works best when rewards can be computed automatically.

### Mathematics

Correct answer:

$$
r=1
$$

Incorrect answer:

$$
r=0
$$

### Coding

Reward may be based on:

- number of unit tests passed
- execution success
- correctness metrics

### Other verifiable domains

Any task where outputs can be automatically checked can potentially provide rewards for GRPO.

This is one reason GRPO became closely associated with reasoning models. Mathematics provides an unusually clean reward signal.

The model does not need human annotators to determine whether $17 \times 24 = 408$.

---

## Wait...what happened to the reward model?

One point of confusion is that GRPO removes the critic, not necessarily the reward function.

Different settings are possible.

### Human preference rewards

A reward model can still be used.

In this case, GRPO resembles PPO but replaces the critic with a group-based baseline.

### Verifiable rewards

The reward can be computed directly.

Examples include:

- exact-match accuracy
- unit-test pass rates
- symbolic verification

This was the setting emphasized by DeepSeekMath.

---

## How does GRPO compare to PPO and DPO?

| Method | Reward Model | Critic | Reinforcement Learning | Online Generation |
|----------|----------|----------|----------|----------|
| PPO-RLHF | ✓ | ✓ | ✓ | ✓ |
| DPO | ✗ | ✗ | ✗ | ✗ |
| GRPO | Optional | ✗ | ✓ | ✓ |

A useful way to think about these methods is:

- **PPO:** keep RL, keep critic
- **DPO:** remove RL entirely
- **GRPO:** keep RL, remove critic

---

## Results from DeepSeekMath

The DeepSeekMath paper applies GRPO to mathematical reasoning tasks and reports consistent improvements over instruction-tuned baselines.

In particular, reinforcement learning improves performance on benchmarks such as GSM8K and MATH after instruction tuning has already been performed.

---

**[INSERT TABLE 5 FROM PAPER HERE]**

*Table 5 shows performance gains obtained through reinforcement learning after instruction tuning.*

---

## Strengths

- Removes the critic
- Reduces memory and training costs
- Retains the benefits of reinforcement learning
- Works naturally with verifiable rewards
- Particularly effective for reasoning and coding tasks

---

## Limitations

GRPO still requires:

- online generation
- reward computation
- reinforcement learning infrastructure

It is therefore more expensive than DPO.

Additionally, GRPO assumes that the policy is already capable of generating at least some useful responses. If every response in a group is equally poor, the relative comparison provides little learning signal.

For this reason, GRPO is typically applied after instruction tuning rather than directly on a pretrained base model.

---

## Takeaways

GRPO can be viewed as a simplified form of PPO.

The key insight is that a separate critic is not strictly necessary. Instead, multiple responses to the same prompt can be used to construct a baseline and estimate relative advantage.

Conceptually:

$$
\text{PPO: } A = R - V
$$

$$
\text{GRPO: } A_i \approx R_i - \text{group baseline}
$$

By replacing a learned critic with group-relative comparisons, GRPO retains the benefits of reinforcement learning while reducing training complexity. This makes it especially attractive for domains such as mathematics and coding, where rewards can be computed automatically and large amounts of training data can be generated on the fly.
