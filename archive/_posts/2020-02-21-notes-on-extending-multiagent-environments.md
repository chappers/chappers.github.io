---
layout: post
category:
tags:
tagline:
---

In this post we'll quickly go through how to add "single" agent policies to QMIX (multiagent) environment specifically to RLlib. Why would we want to do this?

In my mind, algorithms like MAVEN make use of hierarchical policies would need constructs like this.

_Side note_: I dislike using PyMARL as an environment as theres not enough examples to get things working quickly. Whilst RLlib is fairly opinionated - at the very least, its easier to mix and match items together; especially when working with other problems which are not MARL (Multi-agent reinforcement learning).

Of course this is not without downsides; being heavy on RLlib means that doing anything that is "custom" does end up being a lot of code! At this stage its something that I've learnt to live with and the constraints do encourage some form of discipline.

**Broad Idea**

The broad idea is that we make use of QMIX as a "base policy trainer" which we then manually alter policies on top.

However, to use QMIX we need to make use of "agent groups" as part of the Multi-agent environments. These environment allow one to group units logically together (for example, when training for an RTS game, one approach might be to group all the units which are the same unit type together, and have them cooperate).

To do this let's start working backwards (even though this is not how I came to build it!)

```py
config = {
    "env": "my_registered_grouped_env",
    "multiagent": {
        "policies": {
            "pol1": (None, obs_space, act_space, {}),
            "pol2": (MyOtherPolicy, obs_space, act_space, {}),
        },
        "policy_mapping_fn": lambda x: "pol1" if cond else "pol2",
    },
}
```

This layout is the same as an ordinary hierarchical setup in the RLlib documentation. Working backwards, we need to create our groups. Notice that we refer to the groups as part of the `policy_mapping_fn`.

```py
grouping = {"group_1": [0, 1], "group_2": [2]}
obs_space = Tuple(
    [
        Dict({"obs": MultiDiscrete([2, 2, 2, 3]), ENV_STATE: MultiDiscrete([2, 2, 2])}),
        Dict({"obs": MultiDiscrete([2, 2, 2, 3]), ENV_STATE: MultiDiscrete([2, 2, 2])}),
    ]
)
act_space = Tuple([MyEnv.action_space, MyEnv.action_space])
register_env(
    "my_registered_grouped_env",
    lambda config: MyEnv(config).with_agent_groups(
        grouping, obs_space=obs_space, act_space=act_space
    ),
)
```

So then the lambda condition can be replaced with `policy_mapping_fn = lambda x: "pol1" if x == "group_1" else "pol2"`.

Then if we run:

```py
tune.run("QMIX", config=config)
```

Then it would work as expected.

<details>
<summary>A cleaned up example is shown here</summary>

<pre><code>
import argparse
from gym.spaces import Tuple, MultiDiscrete, Dict, Discrete, Box
import numpy as np

import ray
from ray import tune
from ray.tune import register_env, grid_search
from ray.rllib.env.multi_agent_env import MultiAgentEnv
from ray.rllib.policy import Policy
from ray.rllib.agents.qmix.qmix_policy import ENV_STATE
from ray.rllib.agents.ppo.ppo import PPOTFPolicy
from ray.rllib.agents.dqn.dqn_policy import DQNTFPolicy

parser = argparse.ArgumentParser()
parser.add_argument("--stop", type=int, default=50000)
parser.add_argument("--run", type=str, default="PG")
parser.add_argument("--num-cpus", type=int, default=0)


config = {
    "sample_batch_size": 4,
    "train_batch_size": 32,
    "exploration_fraction": 0.4,
    "exploration_final_eps": 0.0,
    "num_workers": 0,
    "mixer": None,  # grid_search([None, "qmix", "vdn"]),
    "env_config": {"separate_state_space": True, "one_hot_state_encoding": True},
}
group = True


class TwoStepGame(MultiAgentEnv):
    action_space = Discrete(2)

    def __init__(self, env_config):
        self.state = None
        self.agent_1 = 0
        self.agent_2 = 1
        # MADDPG emits action logits instead of actual discrete actions

        # env_config.get("one_hot_state_encoding", False)
        self.one_hot_state_encoding = True
        self.with_state = True  # env_config.get("separate_state_space", False)

        # Each agent gets the full state (one-hot encoding of which of the
        # three states are active) as input with the receiving agent's
        # ID (1 or 2) concatenated onto the end.
        self.observation_space = Dict(
            {"obs": MultiDiscrete([2, 2, 2, 3]), ENV_STATE: MultiDiscrete([2, 2, 2])}
        )

    def reset(self):
        self.state = np.array([1, 0, 0])
        return self._obs()

    def step(self, action_dict):
        state_index = np.flatnonzero(self.state)
        if state_index == 0:
            action = action_dict[self.agent_1]
            assert action in [0, 1], action
            if action == 0:
                self.state = np.array([0, 1, 0])
            else:
                self.state = np.array([0, 0, 1])
            global_rew = 0
            done = False
        elif state_index == 1:
            global_rew = 7
            done = True
        else:
            if action_dict[self.agent_1] == 0 and action_dict[self.agent_2] == 0:
                global_rew = 0
            elif action_dict[self.agent_1] == 1 and action_dict[self.agent_2] == 1:
                global_rew = 8
            else:
                global_rew = 1
            done = True

        rewards = {self.agent_1: global_rew / 2.0, self.agent_2: global_rew / 2.0}
        obs = self._obs()
        dones = {"__all__": done}
        infos = {}
        return obs, rewards, dones, infos

    def _obs(self):
        return {
            self.agent_1: {"obs": self.agent_1_obs(), ENV_STATE: self.state},
            self.agent_2: {"obs": self.agent_2_obs(), ENV_STATE: self.state},
        }

    def agent_1_obs(self):
        return np.concatenate([self.state, [1]])

    def agent_2_obs(self):
        return np.concatenate([self.state, [2]])


class RandomPolicy(Policy):
    """Hand-coded policy that returns random actions."""

    def compute_actions(
        self,
        obs_batch,
        state_batches=None,
        prev_action_batch=None,
        prev_reward_batch=None,
        info_batch=None,
        episodes=None,
        **kwargs
    ):
        """Compute actions on a batch of observations."""
        return [self.action_space.sample() for _ in obs_batch], [], {}

    def learn_on_batch(self, samples):
        """No learning."""
        return {"learner_stats": {}}

    def set_epsilon(self, *args, **kwargs):
        return None

    def update_target(self, *args, **kwargs):
        return None


grouping = {"group_1": [0, 1], "group_2": [2]}
obs_space = Tuple(
    [
        Dict({"obs": MultiDiscrete([2, 2, 2, 3]), ENV_STATE: MultiDiscrete([2, 2, 2])}),
        Dict({"obs": MultiDiscrete([2, 2, 2, 3]), ENV_STATE: MultiDiscrete([2, 2, 2])}),
    ]
)
act_space = Tuple([TwoStepGame.action_space, TwoStepGame.action_space])
register_env(
    "grouped_twostep",
    lambda config: TwoStepGame(config).with_agent_groups(
        grouping, obs_space=obs_space, act_space=act_space
    ),
)


def pol_map(x):
    if x == "group_1":
        return "pol1"
    else:
        return "pol2"


config = {
    "env": "grouped_twostep",
    "multiagent": {
        "policies": {
            "pol1": (None, obs_space, act_space, {}),
            "pol2": (
                RandomPolicy,
                obs_space,
                act_space,
                {"fcnet_activation": "softmax"},
            ),
        },
        "policy_mapping_fn": pol_map,
    },
}
tune.run("QMIX", config=config)
</code></pre>
</details>
