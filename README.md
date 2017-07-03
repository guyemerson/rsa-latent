# rsa-latent
Alternative frameworks for gradable predicates in the Rational Speech Acts model.

This code was written at ESSLLI 2016, after taking the course [Composition in Probabilistic Language Understanding](http://gscontras.github.io/ESSLLI-2016/), led by Greg Scontras and Michael Henry Tessler.

It can be run using [WebPPL](http://webppl.org/).

In the original formulation of RSA, gradable predicates are dealt with using a latent variable for the threshold (e.g. there is a price threshold for the predicate for "expensive"). The literal listener and the speaker condition on a specific value for this latent variable, while the pragmatic listener jointly infers the threshold and the world state:

- P_L0(s|u,v)
- P_S(u|s,v)
- P_L1(s,v|u) 

While this formulation leads to a pragmatically-determined threshold, it also has two downsides:

### 1. Intractable ambiguity

The pragmatic listener defines a joint distribution over world states and latent variables. However, this a distribution over not just the latent variables relevant for understanding the observed utterance, but over all latent variables for all possible utterances.  This is because, in order to infer which utterance would be chosen in any particular state, we have to consider all meanings of all utterances.

In order to calculate the pragmatic listener's posterior distribution over world states, given an observed utterance,
we have to marginalise out the latent variables. However, because these variables are not independent of each other or the world state (in the posterior distribution), exact inference requires summing over all possible combinations of values for these latent variables.

If we scale up the model so that there are many possible utterances, each with its own set of readings, we need to have a separate latent variable for every possible ambiguity. In a real world setting, the number of utterances will be vast, and each utterance will have a large number of readings. Even with only 10 utterances, each with 4 readings, there are 4^10 = 1million combinations. With 1000 utterances, each with 100 readings, we get 100^1000 = 10^2000 combinations, which is never going to be practical, or psychologically plausible.

### 2. Lack of optimality

The RSA framework includes an "optimality" parameter for the speaker. However, we can also view RSA as dealing with a signalling game, and find the optimal strategies. Ideally, as we increase the optimality parameter, we should tend towards an optimal strategy. If not, then we are forced to say that RSA models a certain class of suboptimal strategies.

If there are only two possible utterances, and we want to maximise the pragmatic listener's posterior probability of the correct world state, then the optimal behaviour is simple: we want to split the world states into two sets (one for each utterance), where the prior probability of each set is as close as possible to 1/2. By splitting the states into disjoint sets (which can be seen as pragmatically-determined denotations), we minimise the listener's uncertainty, and hence maximise the probability of the correct inference.

In the original RSA approach to gradable predicates, this optimal behaviour is not achieved, even as optimality tends to infinity. This is because the literal listener and the speaker draw the threshold from a prior that cannot be influenced pragmatically. As the optimality parameter is increased, the speaker tends towards optimal behaviour for that given threshold -- but the combined behaviour when drawing a random threshold will not be optimal.

For example, suppose the speaker has to choose between uttering "expensive" and uttering nothing (which is always true). For a fixed threshold, the optimal utterance is to say "expensive" when the object is above the theshold, and to say nothing when it's below. However, this means that if there is some positive probability of the threshold being below the given price, the speaker will have a positive probability of uttering "expensive"; and if there's some positive probability of the threshold being above the given price, the speaker will have a positive probability of uttering nothing. But for the gradable predicate to be relevant, there must be some probability of the threshold being either side, which therefore gives some probability of choosing either utterance -- but as explained above, this is not optimal!

# Alternative Approaches to Gradable Predicates

Here are two alternative approaches to gradable predicates in the RSA framework, which avoid the problems discussed above.

### 1. Permanently lifted variables

In the original version of RSA, the latent variables are taken as given by the speaker and literal listener, and "lifted" by the pragmatic listener. We can instead take the latent variables to be unobserved at every level, and so they are "permanently lifted" rather than "pragmatically lifted". We have:

- P_L0(s,v|u)
- P_S(u,v|s)
- P_L1(s,v|u)

This deals with the problem of intractable inference, because now the latent variables associated with a particular utterance are independent of the world state given a different utterance. This means that it is trivial to marginalise out these variables - we can simply drop them at the end.

This also allows truly optimality behaviour, because the speaker is never forced to assume a particular threshold. For any given world state, as the optimality parameter tends to infinity, the speaker tends towards always picking a single utterance (except in the rare case when there happen to be two utterances with identical payoffs). 

Note that the pragmatically-determined threshold of use for a gradable predicate will not be the same as the posterior distribution for the latent variable. For example, as the optimality parameter tends to infinity, the use of a gradable predicate tends to a sharp cutoff point -- but on hearing nothing, the posterior distribution over the threshold variable will remain the same as the prior.

This approach is taken in `rsa_gradable_latent.wppl`

### 2. Inherently vague predicates

Alternatively, we can take lexical meaning to be inherently vague. Rather than having an unknown threshold which exactly determines the truth of an utterance, we can view truth as a random variable -- in other words, we can exactly know the meaning of a predicate, but this meaning can be inherently vague. This approach is taken by [Emerson and Copestake (2016)](https://arxiv.org/abs/1606.08003), and discussed from a philosophical viewpoint by [Sutton (2017)](https://link.springer.com/article/10.1007/s10670-017-9910-6). Now, for each utterance, we have a probability of truth, which is used by the speaker and listener:

- P(t_u|s)
- P_L0(s|u)
- P_S(u|s)
- P_L1(s|u)

Since these truth values are defined as conditional probabilities, they are conditionally independent by definition, and so inference is tractable.

Optimal behaviour is possible because the speaker is not forced to take a given truth value. As with the permanently lifted model, as the optimality parameter tends to infinity, the speaker tends towards always picking a single utterance.

Note that while semantic meanings are assumed fixed (the distributions P(t_u|s) do not change), a sharp cutoff point for the use of the predicate can emerge, pragmatically determined based on the prior distribution over states.

This approach is taken in `rsa_gradable_vague.wppl`
