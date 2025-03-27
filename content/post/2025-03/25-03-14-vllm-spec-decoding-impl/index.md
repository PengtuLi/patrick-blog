---
title: "Vllm Spec Decoding Impl"
author : "tutu"
description:
date: '2025-03-14'
lastmod: '2025-03-14'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['deep learning']
---

本文主要是分析vllm推测解码实现的部分

## vllm 推测解码伪代码分析

```python

"""Sampling parameters for text generation.

Overall, we follow the sampling parameters from the OpenAI text completion
API (https://platform.openai.com/docs/api-reference/completions/create).
In addition, we support beam search, which is not supported by OpenAI.

Args:
    n: Number of output sequences to return for the given prompt.
    best_of: Number of output sequences that are generated from the prompt.
        From these `best_of` sequences, the top `n` sequences are returned.
        `best_of` must be greater than or equal to `n`. By default,
        `best_of` is set to `n`.
    presence_penalty: Float that penalizes new tokens based on whether they
        appear in the generated text so far. Values > 0 encourage the model
        to use new tokens, while values < 0 encourage the model to repeat
        tokens.
    frequency_penalty: Float that penalizes new tokens based on their
        frequency in the generated text so far. Values > 0 encourage the
        model to use new tokens, while values < 0 encourage the model to
        repeat tokens.
    repetition_penalty: Float that penalizes new tokens based on whether
        they appear in the prompt and the generated text so far. Values > 1
        encourage the model to use new tokens, while values < 1 encourage
        the model to repeat tokens.
    temperature: Float that controls the randomness of the sampling. Lower
        values make the model more deterministic, while higher values make
        the model more random. Zero means greedy sampling.
    top_p: Float that controls the cumulative probability of the top tokens
        to consider. Must be in (0, 1]. Set to 1 to consider all tokens.
    top_k: Integer that controls the number of top tokens to consider. Set
        to -1 to consider all tokens.
    min_p: Float that represents the minimum probability for a token to be
        considered, relative to the probability of the most likely token.
        Must be in [0, 1]. Set to 0 to disable this.
    seed: Random seed to use for the generation.
    stop: List of strings that stop the generation when they are generated.
        The returned output will not contain the stop strings.
    stop_token_ids: List of tokens that stop the generation when they are
        generated. The returned output will contain the stop tokens unless
        the stop tokens are special tokens.
    bad_words: List of words that are not allowed to be generated.
        More precisely, only the last token of a corresponding
        token sequence is not allowed when the next generated token
        can complete the sequence.
    include_stop_str_in_output: Whether to include the stop strings in
        output text. Defaults to False.
    ignore_eos: Whether to ignore the EOS token and continue generating
        tokens after the EOS token is generated.
    max_tokens: Maximum number of tokens to generate per output sequence.
    min_tokens: Minimum number of tokens to generate per output sequence
        before EOS or stop_token_ids can be generated
    logprobs: Number of log probabilities to return per output token.
        When set to None, no probability is returned. If set to a non-None
        value, the result includes the log probabilities of the specified
        number of most likely tokens, as well as the chosen tokens.
        Note that the implementation follows the OpenAI API: The API will
        always return the log probability of the sampled token, so there
        may be up to `logprobs+1` elements in the response.
    prompt_logprobs: Number of log probabilities to return per prompt token.
    detokenize: Whether to detokenize the output. Defaults to True.
    skip_special_tokens: Whether to skip special tokens in the output.
    spaces_between_special_tokens: Whether to add spaces between special
        tokens in the output.  Defaults to True.
    logits_processors: List of functions that modify logits based on
        previously generated tokens, and optionally prompt tokens as
        a first argument.
    truncate_prompt_tokens: If set to an integer k, will use only the last k
        tokens from the prompt (i.e., left truncation). Defaults to None
        (i.e., no truncation).
    guided_decoding: If provided, the engine will construct a guided
        decoding logits processor from these parameters. Defaults to None.
    logit_bias: If provided, the engine will construct a logits processor
        that applies these logit biases. Defaults to None.
    allowed_token_ids: If provided, the engine will construct a logits
        processor which only retains scores for the given token ids.
        Defaults to None.
"""

prefill stage

  get prompt_token_ids=array('l', [2, 29631, 10, 527, 9, 410, 1211, 32280, 14604, 4]), output_token_ids=(280,), cumulative_logprob=0.0, get_num_computed_tokens=10

WHILE not meet eos in decode stage 

  call spculatice_decode_step_using_muti-step-worker
  
    assert for one loop for better show up
    
    call draft proposal with given num lookahead using top-1 proposer
  
      excute model
  
      sample_params is SamplingParams(n=1, presence_penalty=0.0, frequency_penalty=0.0, repetition_penalty=1.0, temperature=0.8, top_p=0.95, top_k=-1, min_p=0.0, seed=None, stop=[], stop_token_ids=[], bad_words=[], include_stop_str_in_output=False, ignore_eos=False, max_tokens=1024, min_tokens=0, logprobs=None, prompt_logprobs=None, skip_special_tokens=True, spaces_between_special_tokens=True, truncate_prompt_tokens=None, guided_decoding=None)
  
      return proposal_tokens = tensor([ 16,   5, 129,  86, 939]) and proposal_probs
    
    call score_proposal with SequenceData and proposal_tokens using mqa_score
  
      imput is prompt_token_ids=[2, 29631, 10, 527, 9, 410, 1211, 32280, 14604, 4]
      input is new_output_token_ids=[280, 16, 5, 129, 86, 939]
  
      excute model
  
      sample_params is SamplingParams(n=1, presence_penalty=0.0, frequency_penalty=0.0, repetition_penalty=1.0, temperature=0.8, top_p=0.95, top_k=-1, min_p=0.0, seed=None, stop=[], stop_token_ids=[],d bad_words=[], include_stop_str_in_output=False, ignore_eos=False, max_tokens=1024, min_tokens=0, logprobs=None, prompt_logprobs=None, skip_special_tokens=True, spaces_between_special_tokens=True, truncate_prompt_tokens=None, guided_decoding=None)
  
      get target_sampler_output=(outputs=[], sampled_token_probs=torch.Size([6, 50272]), sampled_token_ids=torch.Size([6, 1]), spec_decode_worker_metrics=None)
      get target_sampler_output.sampled_token_ids=tensor([[ 40,  10, 129, 169,  38, 655]])
      note 655 is bonus token
      
      return sampled_token_ids
  
    call verification with proposal_scores and proposal_tokens
      
      input is proposal_scores = tensor([[ 40,  10, 129, 169,  38, 655]], device='cuda:0')
      input is proposal_tokens = tensor([ 16, 5, 129, 86, 939]
  
      call modified rejection sampling on each sequence.
        
	    call get_accepted
	    """Create bool matrix over the proposed draft tokens. If
            True, then a token can be accepted, else it should be
            rejected.

            Given :math:`q(\hat{x}_{n+1}|x_1, \dots, x_n)`, the probability of
            :math:`\hat{x}_{n+1}` given context :math:`x_1, \dots, x_n` according
            to the target model, and :math:`p(\hat{x}_{n+1}|x_1, \dots, x_n)`, the
            same conditional probability according to the draft model, the token
            is accepted with probability:

            .. math::
                \min\left(1, \frac{q(\hat{x}_{n+1}|x_1, \dots, x_n)}
                               {p(\hat{x}_{n+1}|x_1, \dots, x_n)}\right)

            This implementation does not apply causality. When using the output,
            if a token is rejected, subsequent tokens should not be used.

            Returns a bool tensor of shape [batch_size, k] specifying which tokens
            are accepted.
            """

	    call_get_recoverd
	    """Create a probability distribution for each proposed token which can
            be sampled if the proposed token is rejected.

            When this routine is applied sequentially, the true distribution of the
            target model is recovered (within hardware numerics).

            The probability distribution used in this rejection case is constructed
            as follows. Given :math:`q(x|x_1, \dots, x_n)`, the probability of
            :math:`x` given context :math:`x_1, \dots, x_n` according to the target
            model and :math:`p(x|x_1, \dots, x_n)`, the same conditional probability
            according to the draft model:

            .. math::
                x_{n+1} \sim (q(x|x_1, \dots, x_n) - p(x|x_1, \dots, x_n))_+

            where :math:`(f(x))_+` is defined as:

            .. math::
                (f(x))_+ = \frac{\max(0, f(x))}{\sum_x \max(0, f(x))}

            See https://github.com/vllm-project/vllm/pull/2336 for a visualization
            of the draft, target, and recovered probability distributions.

            Returns a tensor of shape [batch_size, k, vocab_size].

            Note: This batches operations on GPU and thus constructs the recovered
            distribution for all tokens, even if they are accepted. This causes
            division-by-zero errors, so we use self._smallest_positive_value to
            avoid that. This introduces some drift to the distribution.
            """

	    then call multinomial-sample and get recover token

      get accepted = tensor([[ True,  True,  True, False,  True]],device='cuda:0'),
      get recovered_token_ids = tensor([[ 169,  114,  144, 2949,   47]],device='cuda:0')(Token ids sampled from a recovered distribution, to be used when a token is rejected.)
  
      return accepted_token_ids = tensor([[  16,    5,  129, 2949,   -1,   -1]],device='cuda:0')

ENDWHILE

```
