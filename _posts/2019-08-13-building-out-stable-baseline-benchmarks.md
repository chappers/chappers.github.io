---
layout: post
category : 
tags : 
tagline: 
---

In this post we'll go through how one could build a Keras model using `stable-baselines` library as well as the conditions to create a default gym environment. The simple question is if we want to use the Keras APIs to build the basis for a Policy using PPO; we should be able to do this in a fairly straightforward manner. 

Suppose you wanted to create a simple MLP model:

```py
import tensorflow as tf

# ...think of this as Input...
# flat = tf.keras.Input(shape=(784,))
flat = tf.keras.layers.Flatten()(self.processed_obs)

x = tf.keras.layers.Dense(64, activation="tanh", name="pi_fc_0")(flat)
pi_latent = tf.keras.layers.Dense(64, activation="tanh", name="pi_fc_1")(x)

x1 = tf.keras.layers.Dense(64, activation="tanh", name="vf_fc_0")(flat)
vf_latent = tf.keras.layers.Dense(64, activation="tanh", name="vf_fc_1")(x1)

value_fn = tf.keras.layers.Dense(1, name="vf")(vf_latent)
```

This allows one to define the latent value function and the latent policy for modelling. This is an important part of PPO versus other methods as intuitively it also allows for parameter sharing.

This can be added to the `Policy` object which furthermore requires us to define `probability distribution`, `policy` and `q values`.

```py
from stable_baselines.common.distributions import ProbabilityDistribution

pdtype = ProbabilityDistribution()
proba_distribution, policy, q_value = pdtype.proba_distribution_from_latent(
    pi_latent, vf_latent, init_scale=0.01
)
```

Using this in conjunction with some boilerplate code allows us to construct a policy which can be used for RL purposes. 

Full Example using Cart Pole is shown below.


```py
import tensorflow as tf

from stable_baselines import PPO2
from stable_baselines.common.policies import ActorCriticPolicy


class KerasPolicy(ActorCriticPolicy):
    def __init__(
        self, sess, ob_space, ac_space, n_env, n_steps, n_batch, reuse=False, **kwargs
    ):
        super(KerasPolicy, self).__init__(
            sess, ob_space, ac_space, n_env, n_steps, n_batch, reuse=reuse, scale=False
        )

        with tf.variable_scope("model", reuse=reuse):
            flat = tf.keras.layers.Flatten()(self.processed_obs)

            x = tf.keras.layers.Dense(64, activation="tanh", name="pi_fc_0")(flat)
            pi_latent = tf.keras.layers.Dense(64, activation="tanh", name="pi_fc_1")(x)

            x1 = tf.keras.layers.Dense(64, activation="tanh", name="vf_fc_0")(flat)
            vf_latent = tf.keras.layers.Dense(64, activation="tanh", name="vf_fc_1")(x1)

            value_fn = tf.keras.layers.Dense(1, name="vf")(vf_latent)

            self._proba_distribution, self._policy, self.q_value = self.pdtype.proba_distribution_from_latent(
                pi_latent, vf_latent, init_scale=0.01
            )

        self._value_fn = value_fn
        # self.initial_state = None
        self._setup_init()

    def step(self, obs, state=None, mask=None, deterministic=False):
        if deterministic:
            action, value, neglogp = self.sess.run(
                [self.deterministic_action, self.value_flat, self.neglogp],
                {self.obs_ph: obs},
            )
        else:
            action, value, neglogp = self.sess.run(
                [self.action, self.value_flat, self.neglogp], {self.obs_ph: obs}
            )
        return action, value, self.initial_state, neglogp

    def proba_step(self, obs, state=None, mask=None):
        return self.sess.run(self.policy_proba, {self.obs_ph: obs})

    def value(self, obs, state=None, mask=None):
        return self.sess.run(self.value_flat, {self.obs_ph: obs})


model = PPO2(KerasPolicy, "CartPole-v1", verbose=1)
model.learn(25000)

env = model.get_env()
obs = env.reset()

reward_sum = 0.0
for _ in range(1000):
    action, _ = model.predict(obs)
    obs, reward, done, _ = env.step(action)
    reward_sum += reward
    env.render()
    if done:
        print("Reward: ", reward_sum)
        reward_sum = 0.0
        obs = env.reset()

env.close()
```

