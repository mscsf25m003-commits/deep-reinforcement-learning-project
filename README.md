# 🛡️ Bodyguard vs Intruder: A Deep Reinforcement Learning Journey

*"Why is it so hard to teach an AI to play tag?"*

I spent weeks trying to teach a blue dot (bodyguard) to catch a red dot (intruder) before it reaches the red zone. Here's everything I tried, what worked, what didn't, and what I learned.

![Bodyguard chasing intruder](gifs/zone_defender.gif)

---

## 🎯 What I Built

A custom 2D game where:
- **Blue dot (Bodyguard)** wants to catch the red dot
- **Red dot (Intruder)** wants to reach the red zone (top-left corner)
- **Both move smoothly** (continuous actions, like a joystick)
- **Both learn** using AI algorithms (PPO, A2C, SAC)

The environment is 100% custom code using Gymnasium. I wrote everything: physics, rewards, zones, movement rules.

---

## 📊 All My Experiments (The Short Version)

| # | Bodyguard | Intruder | Setup | Steps | Win Rate | Verdict |
|---|-----------|----------|-------|-------|----------|---------|
| 1 | PPO | PPO | 1v1 | 500k | 0% | ❌ Failed |
| 2 | PPO (shared) | PPO | **2v1** | 1.4M | **100%** | ⚠️ Too easy |
| 3 | PPO | PPO | 1v1 | 500k | 0% | ❌ Failed |
| 4 | **A2C** | PPO | 1v1 | 500k | 0% *(7 wins during training)* | 🔥 **Promising!** |
| 5 | PPO | **Frozen** | 1v1 | 200k | 0% | ❌ Failed |
| 6 | **SAC** | PPO | 1v1 | 500k | 0% | ❌ Failed |
| 7 | PPO | PPO | 1v1 | **2.15M** | 0% *(100% vs scripted)* | ❌ Intruder dominates |

---

## 📈 Summary (2-3 Lines)

> **The bodyguard learned to catch a predictable intruder (100% win rate) and showed real promise during training (7 wins with A2C+PPO). But against a fully adaptive intruder in 1v1 self-play, it consistently failed (0% win rate). The intruder learns too fast and dominates before the bodyguard can adapt. With more training time and better hardware, I believe success is possible.**

---

## 🔬 Experiment 1: Both Use PPO

| Detail | Value |
|--------|-------|
| Bodyguard algorithm | PPO |
| Intruder algorithm | PPO |
| Total training steps | ~500,000 |
| **Result** | **0% win rate** |

**💡 My Insight:** Both agents used the same algorithm but the bodyguard never learned to chase. The intruder just ran straight to the zone and the bodyguard stood still. I think PPO's conservative updates make it hesitant to commit to aggressive chasing. Same algorithm doesn't mean same outcome — the task asymmetry matters more than the algorithm.

---

## 🧪 Experiment 2: Two Bodyguards vs One Intruder

| Detail | Value |
|--------|-------|
| Bodyguards | 2 (sharing same policy) |
| Intruder | 1 |
| Total training steps | ~1,400,000 |
| **Result** | **100% win rate** |

**💡 My Insight:** Two bodyguards worked together and caught the intruder every single time. But here's the problem — they just ran in straight lines toward where the intruder started. They caught it in 5-6 steps, no strategy, no adaptation. This felt like cheating. Two agents chasing one is mathematically unfair. The intruder had no chance. This experiment taught me that making the problem easier doesn't create intelligent behavior — it creates lazy shortcuts.

---

## 🧪 Experiment 3: Single Bodyguard

| Detail | Value |
|--------|-------|
| Bodyguards | 1 |
| Intruder | 1 |
| Total training steps | ~500,000 |
| **Result** | **0% win rate** |

**💡 My Insight:** Removing one bodyguard changed everything. Now the intruder had a fighting chance. The single bodyguard still tried the straight-line strategy but it wasn't fast enough. This showed me that 1v1 is the real challenge. Two bodyguards hid the underlying problem — the bodyguard wasn't learning smart chasing, it was just outnumbering the intruder.

---

## 🔥 Experiment 4: Actor-Critic (A2C) for Bodyguard, PPO for Intruder

| Detail | Value |
|--------|-------|
| Bodyguard algorithm | A2C (Actor-Critic) |
| Intruder algorithm | PPO |
| Total training steps | ~500,000 |
| **Result** | **0% in final test — but 7 wins during training!** |

**💡 My Insight:** THIS WAS EXCITING! During training, I actually saw the bodyguard win 7 times. The red text popped up saying "INTERCEPT" and I thought I finally cracked it. But when I saved the model and tested it, it lost every single time. Why? I think the bodyguard overfit to specific training episodes. Or maybe I interrupted training too early. The key takeaway: **the bodyguard CAN learn!** The 7 wins prove that. With more training steps (maybe 1-2 million more), I believe it would have worked. This experiment gave me hope.

**My thought on the asymmetry:** The bodyguard has to chase a continuously moving, intelligent target. The intruder just has to reach a fixed zone. That's fundamentally harder for the bodyguard. The intruder's job is simpler — just run to the corner. The bodyguard has to predict, intercept, and adapt.

---

## 🧊 Experiment 5: Frozen Intruder

| Detail | Value |
|--------|-------|
| Approach | Froze intruder (it stopped learning) |
| Intruder behavior | Fixed scripted movement |
| Total training steps | ~200,000 |
| **Result** | **0% win rate** |

**💡 My Insight:** I thought freezing the intruder would help the bodyguard learn basic chasing before facing a smart opponent. But the opposite happened. Without a challenging opponent, the bodyguard had no pressure to improve. It learned lazy habits. This taught me that **adversarial pressure is necessary for real learning** — but too much pressure too early crashes everything. It's a delicate balance.

---

## 🎭 Experiment 6: SAC for Bodyguard, PPO for Intruder

| Detail | Value |
|--------|-------|
| Bodyguard algorithm | SAC (Soft Actor-Critic) |
| Intruder algorithm | PPO |
| Why SAC? | Replay buffer, faster updates, designed for continuous actions |
| Total training steps | ~500,000 |
| **Result** | **0% win rate** |

**💡 My Insight:** I chose SAC because I thought its replay buffer and frequent updates would help the bodyguard learn faster. But SAC is designed for maximum exploration — it wants to try different actions, not commit to a straight chase. In a pursuit task, hesitation is death. The intruder just runs straight while the bodyguard explores random directions. Wrong tool for the job. PPO is actually better for aggressive, committed chasing despite being "older".

---

## ⚔️ Experiment 7: Full Pipeline — 2.15 Million Steps

| Phase | What Happened | Steps |
|-------|--------------|-------|
| **Phase 1** | Bodyguard vs scripted intruder (straight line to zone) | 200,000 |
| **Phase 2** | Intruder learns to beat Phase 1 bodyguard | 150,000 |
| **Phase 3** | Self-play: 6 alternating training rounds | 1,800,000 |
| **Total** | | **2,150,000 steps (~2-3 hours training)** |

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

**💡 My Insight:** This was my most thorough experiment. The bodyguard learned to catch a predictable intruder perfectly (100%). That proves the environment works and the bodyguard CAN intercept. But as soon as the intruder started learning, everything fell apart. The intruder figured out how to evade the bodyguard's simple strategy, and the bodyguard never adapted. It's like the intruder learned faster and the bodyguard got stuck in a local optimum.

**My conclusion from this experiment:** The problem isn't that the bodyguard can't learn — it's that the intruder learns too fast. In self-play, the intruder dominates early and the bodyguard never recovers. Future work needs to slow down the intruder's learning or give the bodyguard a head start.

---

## 🧠 My Overall Insights

### What I Learned

| Lesson | Why It Matters |
|--------|----------------|
| **Asymmetric goals are hard** | Intruder's job is simpler (reach a fixed zone). Bodyguard's job is harder (catch a moving, adapting target). This imbalance makes learning much harder for the bodyguard. |
| **Two agents is cheating** | Two bodyguards got 100% win rate but learned nothing smart. Easy problems create lazy solutions. |
| **The bodyguard CAN learn** | 100% win rate vs scripted intruder + 7 wins during A2C training proves the capability exists. |
| **Same algorithm ≠ same outcome** | Both using PPO gave 0% win rate. Task asymmetry matters more than algorithm choice. |
| **Intruder learns too fast** | In self-play, the intruder dominates early and the bodyguard never recovers. Need to slow down intruder or give bodyguard head start. |
| **SAC is too soft** | Maximum entropy exploration is great for robotics but terrible for aggressive pursuit. PPO's clipped updates work better for chasing tasks. |
| **Training time is the real bottleneck** | With 2.15 million steps and 2-3 hours on free Colab, I hit my limit. Paid Colab or local GPU would allow 10+ million steps, which I believe would work. |

### What I'd Do Differently

1. Start with even slower intruder (speed 0.02 instead of 0.04)
2. Keep scripted intruder for first 500k steps (not 200k)
3. Gradually increase intruder learning (don't turn it on at full strength)
4. Train for 5-10 million steps (with better hardware)
5. Use MADDPG instead of PPO (designed specifically for multi-agent)

---

## 📁 Repository Structure
