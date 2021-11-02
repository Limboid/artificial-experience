# The Artificial Experience

The `artificial-experience` is a very general environment and also a library for training and evaluating models, optimziers, and training paradigms and pipelines across dozens of domains, tasks, and new and existing open-source dataset loaders, environments, and hubs simultaneously, lifelong, and in-context. To unify such diverge machine learning approaches, the `artificial-experience` enforces a few key concepts:

1. **Inputs and outputs are labeled by modality**. Each interaction step input or output is a Python `dict` object with `Modality` object keys and `Tensor` or `None` values. A `Modality` defines a combination of `structure` (set, grid, or graph), `representation` (binary, categorical, integer, real), and `context` ("natural", "computer", or other natural language tag).

2. **Datasets are wrapped into `DatasetEnv`s** 1 minibatch = 1 environment step. `DatasetEnv` tries to automatically guess modalities. Many supervised and self-supervised learning problems can be structured using this environment and a prediction-based learning objective.

3. **Environments are wrapped into `SynEnv`s** which present a single  interaction sequence spanning multiple environments. Inter-environment transitions be be triggered on `done` (defualt) or by a custom `should_transition` function. When transitioning, the `SynEnv` first calls a `transition_fn` function with the new environment observation and action spaces which can be used to add new encoders and decoders to the policy. It then runs a few interaction steps where the agent observes a natural language instruction (e.g. `predict the class of images on imagenet`) on the input key `'text:instruction'` associated with the new environment. Finally, interaction begins in the new environment.

4. **Environments compose long-term lifelong learning pipelines**. Each environment is a gym-compatible object and can be wrapped into pipelines and networks.
    - `Interleave` is a high level version of `SynEnvironment` that interleaves interactions from a list of environments with an arbitrary interleave pattern. For example, the interleave patten `[EnvA, EnvB, EnvC, EnvB, EnvC]` takes the first interaction from `EnvA`, the second from `EnvB`, the third from `EnvC`, the fourth from `EnvB`, and the fifth from `EnvC`. The environment is done when either the first or all sub-environments are done. 
    - `Multitasking` makes an agent interact in multiple environments simultaneously. The environment is done when either the first or all sub-environments are done.
    - `Teacher` occasionally reverts the wrapped environment's state to a previous state where performance maximally increased. It can buffer with a rolling history, top k, or arbitrary `should_store_state` function. This wrapper is useful for implementing Go-Explore-type algorithms.
    - `Augment` is a base class for wrapper environments that augment specific inputs and outputs.
    - `Dropout` is an `Augment` environment wrapper that occasionally replaces an input or output value with 0 or another specified value.
    - `Noisy` is an `Augment` environment wrapper that adds noise to an input or output value.
    - `Repeat` is an `Augment` environment wrapper that occasionally repeats an input or output value for multiple interaction steps.
    - `DropKeyValue` is an `Augment` environment wrapper that occasionally drops an input or output value *and key* from the dictionary.
    - `ObserveReward` directly includes wrapped environment's reward in the observation space.
    - `Advantage` directly includes the wrapped environment's Nth-order reward advantage in the observation space.
    - `ObserveActions` feeds the agent's actions into the next interaction step observation. 
    - `PredictInputs` expects and rewards agents for predicting the next input values.
    - `StaticTimescaled` allows tuning the amount of 'ponder' steps that a model gets between external environment interactions. Inputs can be dropped or repeated. Outputs can be averaged, max pooled, min pooled, randomly selected, or last index selected along the time dimension.
    - `DynamicTimescaled` is like `StaticTimescaled` but it gives the agent the ability to observe and modify its external environment interaction timescale.
    - `PenalizeCompute` decreases reward proportional to the amount of compute used since the last interaction. This penalizes the agent (such as in a `DynamicTimescaled`) for using too much compute.

**Suggestions for agents**
- Maintain reccurent states across transitions to learn meta-learning.
- Give the recurrent state strong expressive potential over the activation landscape.
- Only make architecture changes (new encoders and decoders) when absolutely necessary
- Occasionally train on input prediction error supervisedly in between training epochs.
- Try to put the 1st-order optimizer inside your model.

## Getting Started

**IGNORE THIS SECTION. THIS IS A WORK IN PROGRESS**

First, install the `artificial-experience`
```base
$ pip install artificial-experience
```

You can optionally install extras:
```
$ pip install artificial-experience[baselines]  # with baselines
```

## Examples

TODO

```python



```


```python
env = AEEnv(envs=[
    DatasetEnv(tfds.load('coco')), # multimodal information
    DatasetEnv(hub.load('hub://activeloop/mnist-train'))  # cloud-native data
    DatasetEnv(tfds.load('anli'), epochs=4, batch_size=1024), # quick customization
    gym.make('CartPole-v0'), # continous observation, discrete control
    gym.make('Pong-v0'), # rgb image, discrete actions
    gym.make('HalfCheetah-v2'), # continuous observation, continuous control
    gym_starcraft.envs.starcraft_base_env(), # starcraft env
    pettingzoo.atari.mario_bros_v2.env() # multiagent atari env
])
```

`AEEnv` also makes it easy to train on prespecified problem domains with datasets and environments minimally specified by some overlapping hierarchial tag-based system. Not all environments have the `.tag` attribute, so those will be ignored. However, the inbuilt list of envionrments should all support this schema. These filters can be changed at any moment between `AEEnv` steps. See Appendix A for a list of what I want to support.
```python
env = AEEnv(
    include=[
        domains.text_commonsense, 
        'domains.image', 
        'domains.multiagent'],
    exclude=[
        lambda x: False if isinstance(x, Env) and x.version<2 else True,
        domains.multiagent.atari],
) # train on text-commonsense (specific), image datasets (broad), and multiagent RL environments (broad) but don't train on the multiagent/atari environment or multiagent environments that don't have a environment specified reward.

env = AEEnv() # train on all inbuilt datasets and environments
```

## Features

TODO. List every public function, class, and method.


## Datasets and environments

The categories overlap. For instance, image captioning might be in the `image` category, but also in the `text` category. The high-level hierarchy might be:
- images
- text
- video
- audio
TODO

### NLP
from Google's [FLAN blog post](https://ai.googleblog.com/2021/10/introducing-flan-more-generalizable.html): 
- Natural language inference: ANLI, RTE, CB, SNLI, MNLI, QNLI, WNLI, QNLI, 
- Commonsense: CoPA, HeliaSwag, PiQA, StoryCloze
- Sentiment: IMDB, Sent140, SST-2, Yelp
- Paraphrase: MRPC, QQP, PAWS, STS-B
- Closed book QA: ARC (easy/chal), NQ, TQA
- Struct to Text: CommonGen, DART, E2ENLG, WEBNLG
- Reading Comp: 
- Reading Comp w/o commonsensne:
- Conference:
- Misc.:
- Summarization:
- Translation:

### Images

### Video

### 