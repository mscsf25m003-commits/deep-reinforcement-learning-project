# 🛡️ Bodyguard vs Intruder: A Deep Reinforcement Learning Project

I spent weeks trying to teach a blue dot (bodyguard) to catch a red dot (intruder) before it reaches the red zone. Here's everything I tried, what worked, what didn't, and what I learned.

![Bodyguard chasing intruder](gifs/zone_defender.gif)

---

## 🎯 What I Built

A custom 2D game where:
- **Blue dot (Bodyguard)** wants to catch the red dot
- **Red dot (Intruder)** wants to reach the red zone 
- **Both move smoothly** (continuous actions)
- **Both learn** using AI algorithms (PPO, A2C, SAC)

The environment is custom code using Gymnasium.
---

## All My Experiments 
**One important note:** Throughout all my experiments, I only changed the **bodyguard's algorithm**. The **intruder always used PPO**

| # | Bodyguard | Intruder | Setup | Steps | Win Rate | Verdict |
|---|-----------|----------|-------|-------|----------|---------|
| 1 | PPO | PPO | 1v1 | 500k | 0% |  Failed |
| 2 | PPO (shared) | PPO | **2v1** | 1.4M | **100%** | Too easy |
| 3 | **A2C** | PPO | 1v1 | 500k | 0% *(7 wins during training)* |  **Promising!** |
| 4 | PPO | **Frozen** | 1v1 | 200k | 0% | Failed |
| 5 | **SAC** | PPO | 1v1 | 500k | 0% |  Failed |
| 6 | PPO | PPO | 1v1 | **2.15M** | 0% *(100% vs scripted)* |  Intruder dominates |

---

##  Summary 

> **I built a custom 2D environment where a blue bodyguard tries to catch a red intruder before it reaches the top-left red zone. After 7 experiments and 2.15 million training steps, the bodyguard learned to catch a predictable intruder (100% win rate) but failed to beat an adaptive, learning intruder (0% win rate). The key insight: the intruder learns too fast and dominates self-play.**

---

##  Experiment 1: Both Use PPO

| Detail | Value |
|--------|-------|
| Bodyguard algorithm | PPO |
| Intruder algorithm | PPO |
| Total training steps | ~500,000 |
| **Result** | **0% win rate** |

**💡 My Insight:** Both agents used the same algorithm but the bodyguard never learned to chase. The intruder just ran straight to the zone and the bodyguard stood still.Same algorithm doesn't mean same outcome — the task asymmetry matters more than the algorithm.

---

##  Experiment 2: Two Bodyguards vs One Intruder

| Detail | Value |
|--------|-------|
| Bodyguards | 2 (sharing same policy) |
| Intruder | 1 |
| Total training steps | ~1,400,000 |
| **Result** | **100% win rate** |

**💡 My Insight:** Two bodyguards worked together and caught the intruder every single time. But here's the problem — they just ran in straight lines toward where the intruder started. They caught it in 5-6 steps, no strategy, no adaptation. 

---

##  Experiment 3: Single Bodyguard

| Detail | Value |
|--------|-------|
| Bodyguards | 1 |
| Intruder | 1 |
| Total training steps | ~500,000 |
| **Result** | **100% win rate** |

**My Insight:** Removing one bodyguard nothing changed because the policy is same that the bodyguard agent learn already .same problem, just ran in straight line toward where the intruder started and caught it in 5-6 steps, no strategy, no adaptation

---

##  Experiment 4: Actor-Critic (A2C) for Bodyguard, PPO for Intruder

| Detail | Value |
|--------|-------|
| Bodyguard algorithm | A2C (Actor-Critic) |
| Intruder algorithm | PPO |
| Total training steps | ~500,000 |
| **Result** | **0% in final test — but 7 wins during training!** |

**My Insight:**  During training, I actually saw the bodyguard win 7 times. But when I saved the model and tested it, it lost every single time. Why? I think the bodyguard overfit to specific training episodes. Or maybe I interrupted training too early. The key takeaway: **the bodyguard CAN learn!** The 7 wins prove that. With more training steps (maybe 1-2 million more), I believe it would have worked. This experiment gave me hope.

**My thought on the asymmetry:** The bodyguard has to chase a continuously moving, intelligent target. The intruder just has to reach a fixed zone. That's fundamentally harder for the bodyguard. The intruder's job is some how simpler than bodyguard — just run to the corner. The bodyguard has to predict, intercept, and adapt.

---

## Experiment 5: Frozen Intruder

| Detail | Value |
|--------|-------|
| Approach | Froze intruder (it stopped learning) |
| Intruder behavior | Fixed scripted movement |
| Total training steps | ~200,000 |
| **Result** | **0% win rate** |

**My Insight:** I thought freezing the intruder would help the bodyguard learn basic chasing before facing a smart opponent. But the opposite happened. Without a challenging opponent, the bodyguard had no pressure to improve. It learned lazy habits. This taught me that **adversarial pressure is necessary for real learning** — but too much pressure too early crashes everything. It's a delicate balance.

---

## Experiment 6: SAC for Bodyguard, PPO for Intruder

| Detail | Value |
|--------|-------|
| Bodyguard algorithm | SAC (Soft Actor-Critic) |
| Intruder algorithm | PPO |
| Why SAC? | Replay buffer, faster updates, designed for continuous actions |
| Total training steps | ~500,000 |
| **Result** | **0% win rate** |

**My Insight:** I chose SAC because I thought its replay buffer and frequent updates would help the bodyguard learn faster. But SAC is designed for maximum exploration — it wants to try different actions, not commit to a straight chase.The bodyguard explores random directions.

---

##  Experiment 7: Full Pipeline — 2.15 Million Steps

| Phase | What Happened | Steps |
|-------|--------------|-------|
| **Phase 1** | Bodyguard vs scripted intruder (straight line to zone) | 200,000 |
| **Phase 2** | Intruder learns to beat Phase 1 bodyguard | 150,000 |
| **Phase 3** | Self-play: 6 alternating training rounds | 1,800,000 |
| **Total** | | **2,150,000 steps** |

**Results by Phase:**

| Stage | Bodyguard Win Rate |
|-------|-------------------|
| After Phase 1 (vs scripted) | **100%** ✅ |
| After Phase 2 | 0% (intruder dominates) |
| Self-play Round 1 | 0% |
| Self-play Round 2 | 0% |
| Self-play Round 3 | 0% |
| Self-play Round 4 | 0% |
| Self-play Round 5 | 0% |
| Self-play Round 6 | 0% |
| **Final** | **0%** |

**My Insight:** The bodyguard learned to catch a predictable intruder perfectly (100%). That proves the environment works and the bodyguard CAN intercept. But as soon as the intruder started learning, everything fell apart. The intruder figured out how to evade the bodyguard's simple strategy, and the bodyguard never adapted. It's like the intruder learned faster and the bodyguard got stuck in a local optimum.

**My conclusion from this experiment:** The problem isn't that the bodyguard can't learn — it's that the intruder learns too fast. In self-play, the intruder dominates early and the bodyguard never recovers. Future work needs to slow down the intruder's learning or give the bodyguard a head start.

---



### What I Learned

| Lesson | Why It Matters |
|--------|----------------|
| **Asymmetric goals are hard** | Intruder's job is simpler (reach a fixed zone). Bodyguard's job is harder (catch a moving, adapting target). This imbalance makes learning much harder for the bodyguard. |
| **Two agents is cheating** | Two bodyguards got 100% win rate but learned nothing smart. Easy problems create lazy solutions. |
| **The bodyguard CAN learn** | 100% win rate vs scripted intruder + 7 wins during A2C training proves the capability exists. |
| **Same algorithm ≠ same outcome** | Both using PPO gave 0% win rate. Task asymmetry matters more than algorithm choice. |
| **Intruder learns too fast** | In self-play, the intruder dominates early and the bodyguard never recovers. Need to slow down intruder or give bodyguard head start. |
| **Training time is the real bottleneck** | I had limited time and hardware. with better hardware that allow 10+ million steps, which I believe would work. |


