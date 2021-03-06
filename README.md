# Orion for pytorch-a2c-ppo-acktr

## What changed?

To get the original repository of [ikostrikov](https://github.com/ikostrikov/pytorch-a2c-ppo-acktr) to work using Orion, we make a couple of changes.

First, we fork the original repo at commit hash: 4d95ec364c7303566c6a52fb0a254640e931609d

To the top of

```bash
main.py
```
we add:

```bash
#!/usr/bin/env python
from orion.client import report_results
```

and then we run

```
chmod +x main.py
```

To make it executable.

Then, we ensure that we evaluate on a separate set of hold out random seeds for the environment (which should be different than the test set and training seed). For MuJoCo environments where the random seed has an effect, we can simply set the random seed before a rollout. In Atari, we would have to create a new validation set of rollouts perhaps with different human starts.

We create a file with functions for evaluation:

```bash
eval.py
```
And then simply add a registration for the evaluation after training our algorithm:

```python
    validation_returns = evaluate_with_seeds(eval_env, actor_critic, args.cuda, eval_env_seeds)

    report_results([dict(
        name='validation_return',
        type='objective',
        value=np.mean(validation_returns))])
```

Now we're ready to go to run orion's hyperparameter optimization!

## How to search for hyperparameters

```bash
orion -v hunt -n ppo_hopper \
  ./main.py --env-name "Hopper-v2" --algo ppo --use-gae --vis-interval 1 \
  --log-interval 1 --num-stack 1 --num-steps 2048 --num-processes 1 \
  --lr~'loguniform(1e-5, 1.0)' --entropy-coef 0 --value-loss-coef 1 \
  --ppo-epoch 10 --num-mini-batch 32 --gamma~'uniform(.95, .9995)' --tau 0.95 \
  --num-frames 1000000 --eval-env-seeds-file ./seeds.json --no-vis \
  --log-dir~trial.hash_name
```

Notice that this will search over the learning rates and gamma values, while setting the log directory name to be the hashed trial name provided in the orion database.
