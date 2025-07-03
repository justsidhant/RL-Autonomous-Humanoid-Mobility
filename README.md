# Reinforcement Learning for Autonomous Humanoid Mobility

---

## 1. Project overview

The goal is to make the 3-D bipedal robot in the `Humanoid-v4` MuJoCo environment stand, walk, and keep balance using **Trust Region Policy Optimisation (TRPO)**.  
Key facts from the environment:

* **State space** – 376-D continuous vector with unbounded real values.  
* **Action space** – 17-D vector in **[-1, 1]** representing joint torques.  
* The agent is penalised for falling and rewarded for forward progress and alternating steps. 

---

## 2. Method

| Component | Details |
|-----------|---------|
| **Policy network** | 3-layer MLP – 376 → 64 → 64 → 17 with `tanh` activations. |
| **Value network**  | Same hidden layout – outputs a single state-value. |
| **Regularisation** | Higher `l2_reg` to stop torque explosions. |
| **Batch size**     | Increased relative to smaller tasks (Walker2d, Hopper) to handle large spaces. |
| **Sampling**       | Single-path trajectory collection. |
| **Optimiser**      | Conjugate Gradient for the Fisher–vector product. |

> We trained for **500 iterations**, logging `num_steps`, `num_episodes`, and `reward_batch` at every iteration.

---

## 3. Results

* **Mid-training**: the humanoid can balance and take an occasional step, but often “slides” on one leg.  
* Episodic reward begins near zero, rises gently until roughly episode 150, then accelerates sharply. By episode 250 the agent surpasses 20,000 reward, continues fluctuating upward, and plateaus between 40,000 and 55,000 by episode 500. The steep section corresponds to the moment the humanoid first discovers stable alternating steps.
* Training a single run to the “stand and first step” milestone took about **10 h** on a 16-core CPU.
* A separate run tracked mean episodic reward against environment steps:

| Algorithm | Peak mean reward | Qualitative trend |
|-----------|-----------------|-------------------|
| **SAC**   | ~4,200 by 400k steps | Smooth, monotonic climb after an initial flat period |
| **TD3**   | < 100 for the same horizon | Remains near zero with minor upticks |
| **A2C**   | < 100 for the same horizon | Flat, similar to TD3 |

   SAC is clearly the strongest off-policy method but still underperforms TRPO on this task.
