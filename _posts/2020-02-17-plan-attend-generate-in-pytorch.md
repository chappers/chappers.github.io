---
layout: post
category:
tags:
tagline:
---

There have been several different blog posts which talk about Encoder Decoder mechanisms and annotate their implementation. In this blog post we will do like-wise and annotate the ["Plan, Attend, Generate: Planning for Sequence-to-Sequence Models"](https://papers.nips.cc/paper/7131-plan-attend-generate-planning-for-sequence-to-sequence-models) which was part of NIPS 2017.

Although this isn't a particular famous or popular paper (with 5 citations); it is a generalisation of a RL approach called STRAW from ["Strategic Attentive Writer for Learning Macro-Actions"](https://arxiv.org/abs/1606.04695) which was part of NIPS 2016.

This post is written and based heavily on the implementation and post ["The Annotated Encoder-Decoder with Attention"](https://bastings.github.io/annotated_encoder_decoder/) which in turn was the basis for "Plan, Attend, Generate" (PAG).

Although this isn't the official implementation - we will try our best to adhere to the implementation and notes within the original paper and talk through the assumptions which I make when implementing the model. There is an [official implementation which I will reference](https://github.com/Dutil/PAG/blob/3e9f9beac6072cdc1aabaa5f015574402b12929c/planning.py#L685) - however I have found it difficult to align all parts of the code together

_Note: we won't have a useful implementation here; it might be an item that I consider in the future_

# Model Architecture

The model architecture is shown below; as presented in the paper. The overarching goal is that the model learns to plan and execute alignments. It differs from standard sequence-to-sequence model with attention in that it makes a plan for future alignment and whether to follow the plan using a separate commitment vector.

![pag image](/img/pag/pag.png)

## Encoder

The encoder used is the _same_ encoder as per the attention-based neural machine translation paper (Attention-NMT). It is a bidirection RNN, which we will use a Bi-GRU in this scenario.

For each input position $i$, an annotation vector $h_i$ is created by generating both the forward and backward encoder states; this will contain the full context for position $i$ as it will have both information on preceding and proceeding token information. Typically, this information would have an embedding vector formed as well so that the model can exploit tokens which are similar.

## Decoder

On first glance, the decoder is very much the same as Attention-NMT approach; whereby the decoder is formed by taking in some additional information called _context_

$$s_t = f_{\text{decode}}(s_{t-1}, y_t, \psi_t)$$

Where $y_t$ is the previously generated token, and $\psi_t$ is the context obtained by weighted sum of encoder annotations $\psi_t = \sum_{i} \alpha_{ti} h_i$

Where does the difference lie?

In the attention mechanism! Rather than simply creating a mechanism with attention, we also add a _planning_ mechanism, which is _generated_ at each time step of the decoder - leading to the name "Planning, Attend, Generate".

**Attend**

The "attend" (or alignment) mechanism is formed through generating a candidate alignment plan $A_t$; based on the number of steps we "look ahead". For the $i$th look ahead step, it is generated from

$$A_t[i] = f_{\text{align}}(\textbf{s}_{t-1}, \textbf{h}_j, \beta_t^i, \textbf{y}_t)$$

where the alignment vector for the first time step (i.e. $A_t[0]$) is the **attention mechanism** of interest. In this scenario $f_{\text{align}}$ is presented as a MLP, with $\beta_t^i$ to be the summary of the alignment matrix at $i$th planning step for time $t-1$.

**Planning**

If we simply generate a brand new alignment vector without any context with what was considered before - then we've done no better than simply using attention. So we must have a way to plan. To plan, it use MLP with gumbel-softmax trick. Note that this is recomputed at every time-step to redetermine whether or not we need to re-plan or not!

$$c_t = f_{\text{plan}}(s_{t-1})$$

Whereby the output is a one-hot encoded vector which indicates when the next "planning" step should occur.

**Generate**

Finally, how is all of this generated? As per the planning stage - if the next step, i.e. $g_t = c_t[0]$, is the commitment switch. When $g_t = 0$ this indicates that we proceed as planned, and everything (decoder and plan) is moved forward using the _shift_ operator. When $g_t = 1$, we update our information and interpolate the previous alignment plan to create a new alignment plan. These are combined in an additive manner (basically a weighted sum, so we add sigmoid activation) $\mathbf{u}_{ti}$ which is also learned.

$$\mathbf{u}_{ti} = f_{\text{update}}(\mathbf{h}_i, \mathbf{s}_{t-1})$$

$$\bar{A}_t[:, i] = (1-\mathbf{u}_{ti}) \cdot A_{t-1}[:, i] + \mathbf{u}_{ti} \cdot A_{t}[:, i]$$

Pseudo-code

```
for j in input_token:
  for t in output_token:
    if g = 1:
      compute commitment plan: c
      update alignment plan through interpolation: A = (1-u) A[old] + u A[new]
    if g = 0:
      shift commitment plan: c
      shift alignment plan: A
    compute alignment alpha = A[0]
```

# Coding!

**Prelim**

We shall use PyTorch (at the time of writing it is version 1.3.X)

```py
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
import math, copy, time
from torch.nn.utils.rnn import pack_padded_sequence, pad_packed_sequence
```

## Model Class

The EncoderDecoder class is almost identical to Attention-NMT. The differences are around how to retain semblance of state between subsequent decoder calls

```py
class EncoderDecoder(nn.Module):
    """
    A (mostly) standard Encoder-Decoder architecture. Base for this and many
    other models.
    """

    def __init__(self, encoder, decoder, src_embed, trg_embed, generator):
        super(EncoderDecoder, self).__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.src_embed = src_embed
        self.trg_embed = trg_embed
        self.generator = generator

    def forward(
        self,
        src,
        trg,
        src_mask,
        trg_mask,
        src_lengths,
        trg_lengths,
        previous_hidden=None,
        action_plan=None,
        previous_out=None,
        commit_plan=None,
    ):
        """Take in and process masked src and target sequences."""
        encoder_hidden, encoder_final = self.encode(src, src_mask, src_lengths)

        decoder_states, hidden, pre_output_vectors, action_plan, commit_plan = self.decode(
            encoder_hidden,
            encoder_final,
            src_mask,
            trg,
            trg_mask,
            hidden=previous_hidden,
            action_plan=action_plan,
            previous_out=previous_out,
            commit_plan=commit_plan,
        )

        # if hidden is None:
        #     hidden = self.init_hidden(encoder_final)

        # commitment_plan = self.planner(hidden)
        return decoder_states, hidden, pre_output_vectors, action_plan, commit_plan

    def encode(self, src, src_mask, src_lengths):
        src = self.src_embed(src)
        # print("embed src", src.shape)
        return self.encoder(src, src_mask, src_lengths)

    def decode(
        self,
        encoder_hidden,
        encoder_final,
        src_mask,
        trg,
        trg_mask,
        hidden=None,
        action_plan=None,
        previous_out=None,
        commit_plan=None,
    ):
        # print("embed trg pre", trg.shape)
        trg = self.trg_embed(trg)
        # print("embed trg", trg.shape)
        return self.decoder(
            trg,
            encoder_hidden,
            encoder_final,
            src_mask,
            trg_mask,
            hidden=hidden,
            action_plan=action_plan,
            previous_out=previous_out,
            commit_plan=commit_plan,
        )
```

We also make use of our embeddings and generators which are simply one layer items. Of course for embeddings you could use the inbuilt embedding as well if the sequences are word tokens.

```py
class ContinuousEmbedding(nn.Module):
    def __init__(self, emb_size):
        super(ContinuousEmbedding, self).__init__()
        self.embedding = nn.Linear(1, emb_size, bias=False)

    def forward(self, x):
        x = x.unsqueeze(-1)
        x = self.embedding(x)
        return x


class Generator(nn.Module):
    """Define standard linear + softmax generation step."""

    def __init__(self, hidden_size, output_size):
        super(Generator, self).__init__()
        self.proj = nn.Linear(hidden_size, output_size, bias=False)

    def forward(self, x):
        x = self.proj(x)
        return x
```

## Encoder

The encoder is exactly the same as Attention-NMT. As PyTorch enables much of this out of the box, there's very little to worry about. We make use of the utility functions as part of pytorch to handle padding for varying sentence lengths as well.

```py
class Encoder(nn.Module):
    """Encodes a sequence of word embeddings
    """

    def __init__(self, input_size, hidden_size, num_layers=1, dropout=0.0):
        super(Encoder, self).__init__()
        self.num_layers = num_layers
        self.rnn = nn.GRU(
            input_size,
            hidden_size,
            num_layers,
            batch_first=True,
            bidirectional=True,
            dropout=dropout,
        )

    def forward(self, x, mask, lengths):
        """
        Applies a bidirectional GRU to sequence of embeddings x.
        The input mini-batch x needs to be sorted by length.
        x should have dimensions [batch, time, dim].

        what does the util functions do? Ensure that all inputs have
        the same length!
        """
        packed = pack_padded_sequence(x, lengths, batch_first=True)
        output, final = self.rnn(packed)
        output, _ = pad_packed_sequence(output, batch_first=True)

        # we need to manually concatenate the final states for both directions
        fwd_final = final[0 : final.size(0) : 2]
        bwd_final = final[1 : final.size(0) : 2]
        final = torch.cat([fwd_final, bwd_final], dim=2)  # [num_layers, batch, 2*dim]

        return output, final
```

## Decoder

The decoder is a conditional GRU. Again we don't change things too much from the Attention-NMT approach, as the difference lies within the PAG mechanism. The largest difference is the requirement to maintain some form of state.

```py

class Decoder(nn.Module):
    """A conditional RNN decoder with attention."""

    def __init__(
        self, emb_size, hidden_size, attention, num_layers=1, dropout=0.5, bridge=True
    ):
        super(Decoder, self).__init__()

        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.attention = attention
        self.dropout = dropout

        self.rnn = nn.GRU(
            emb_size + 2 * hidden_size,
            hidden_size,
            num_layers,
            batch_first=True,
            dropout=dropout,
        )

        # to initialize from the final encoder state
        self.bridge = (
            nn.Linear(2 * hidden_size, hidden_size, bias=True) if bridge else None
        )

        self.dropout_layer = nn.Dropout(p=dropout)
        self.pre_output_layer = nn.Linear(
            hidden_size + 2 * hidden_size + emb_size, hidden_size, bias=False
        )

    def forward_step(
        self,
        prev_embed,
        encoder_hidden,
        src_mask,
        proj_key,
        hidden,
        beta=None,
        previous_out=None,
        prev_action_plan=None,
        prev_commit_plan=None,
    ):
        """Perform a single decoder step (1 word)
        prev_embed, encoder_hidden, src_mask, proj_key, hidden
        """

        # compute context vector using attention mechanism
        query = hidden[-1].unsqueeze(1)  # [#layers, B, D] -> [B, 1, D]

        context, attn_probs, action_plan, commit_plan = self.attention(
            query=query,
            proj_key=proj_key,
            value=encoder_hidden,
            mask=src_mask,
            beta=None,
            previous_out=previous_out,
            prev_action_plan=prev_action_plan,
            prev_commit_plan=prev_commit_plan,
        )

        # update rnn hidden state
        rnn_input = torch.cat([prev_embed, context], dim=2)
        output, hidden = self.rnn(rnn_input, hidden)

        pre_output = torch.cat([prev_embed, output, context], dim=2)
        pre_output = self.dropout_layer(pre_output)
        pre_output = self.pre_output_layer(pre_output)

        return output, hidden, pre_output, action_plan, commit_plan

    def forward(
        self,
        trg_embed,
        encoder_hidden,
        encoder_final,
        src_mask,
        trg_mask,
        hidden=None,
        max_len=None,
        action_plan=None,
        previous_out=None,
        commit_plan=None,
    ):
        """Unroll the decoder one step at a time.

        trg_embed       = encoder_hidden,
        encoder_hidden  = encoder_final,
        encoder_final   = src_mask[[0], :, :],
        src_mask        = prev_y,
        trg_mask        = prev_mask,
        hidden          = hidden
        """

        # the maximum number of steps to unroll the RNN
        if max_len is None:
            max_len = trg_mask.size(-1)

        # initialize decoder hidden state
        if hidden is None:
            hidden = self.init_hidden(encoder_final)

        # pre-compute projected encoder hidden states
        # (the "keys" for the attention mechanism)
        # this is only done for efficiency

        # TODO fix this hack...should be created under self.attention, as it is shared?
        # this is shared with the generate layer as
        proj_key = self.attention.attend.key_layer(encoder_hidden)
        if action_plan is not None:
            prev_action_plan = action_plan.clone().detach()
        else:
            prev_action_plan = None

        if commit_plan is not None:
            prev_commit_plan = commit_plan.clone().detach()
        else:
            prev_commit_plan = None

        if self.attention.attend.summary_layer is not None and action_plan is not None:
            beta_summary = self.attention.attend.summary_layer(action_plan)
        else:
            beta_summary = None

        if (
            self.attention.attend.prev_summary_layer is not None
            and previous_out is not None
        ):
            prev_summary = self.attention.attend.prev_summary_layer(previous_out)
        else:
            prev_summary = None

        # here we store all intermediate hidden states and pre-output vectors
        decoder_states = []
        pre_output_vectors = []

        # unroll the decoder RNN for max_len steps
        for i in range(max_len):
            prev_embed = trg_embed[:, i].unsqueeze(1)
            output, hidden, pre_output, action_plan, commit_plan = self.forward_step(
                prev_embed=prev_embed,
                encoder_hidden=encoder_hidden,
                src_mask=src_mask,
                proj_key=proj_key,
                hidden=hidden,
                beta=beta_summary,
                previous_out=prev_summary,
                prev_action_plan=prev_action_plan,
                prev_commit_plan=prev_commit_plan,
            )
            decoder_states.append(output)
            pre_output_vectors.append(pre_output)

        decoder_states = torch.cat(decoder_states, dim=1)
        pre_output_vectors = torch.cat(pre_output_vectors, dim=1)

        return (
            decoder_states,
            hidden,
            pre_output_vectors,
            action_plan,
            commit_plan,
        )  # [B, N, D]

    def init_hidden(self, encoder_final):
        """Returns the initial decoder state,
        conditioned on the final encoder state."""

        if encoder_final is None:
            return None  # start with zeros

        return torch.tanh(self.bridge(encoder_final))
```

## Plan, Attend, Generate

To implement the PAG mechanism, we take some liberties in how it was done in the original paper. The original paper does not specify in great detail how each component was contructed; though we note the similarities between Attention-NMT and PAG approaches.

Starting with the _update_ layer - this one is one of the easier ones as it needs only combine information on $\mathbf{h}$ and $\mathbf{s}$. In order to effectively combine them together, embedding both inputs is created so that they can be safely combined through addition. The embeddings both us `tanh` as to safely allow them to shift together than `sigmoid` is applied to generate the weighted sum.

```py
class PAGGenerate(nn.Module):
    """
    Implements the update layer as part of PAG
    this is part of the "generate" step
    """

    def __init__(self, hidden_size, plan_length=1, query_size=None, score_size=None):
        super(PAGGenerate, self).__init__()
        query_size = hidden_size if query_size is None else query_size
        self.score_size = hidden_size if score_size is None else score_size
        self.query_layer = nn.Linear(query_size, hidden_size, bias=False)
        self.energy_layer = nn.Linear(hidden_size, plan_length, bias=False)

        # build embedding of the value (hidden)
        self.value_embedding = nn.Linear(hidden_size * 2, plan_length, bias=False)

    def forward(
        self,
        query=None,
        proj_key=None,
        value=None,
        mask=None,
        beta=None,
        previous_out=None,
    ):
        # TODO remove beta and previous_out later.

        assert mask is not None, "mask is required"

        # We first project the query (the decoder state).
        # The projected keys (the encoder states) were already pre-computated.
        decoder_state = self.query_layer(query)

        # Calculate scores.
        scores = self.energy_layer(torch.tanh(decoder_state + proj_key))
        scores = scores.permute(0, 2, 1)
        value_embed = self.value_embedding(torch.tanh(value))
        value_embed = value_embed.permute(0, 2, 1)

        update = value_embed + scores
        return torch.sigmoid(update)
```

In a similar way the commitment plan can be generated in a rather straightforward fashion

```py
class PAGPlan(nn.Module):
    """
    Implements the commitment plan layer as part of PAG
    This is part of the "plan" step
    """

    def __init__(self, hidden_size, plan_length=1, query_size=None, score_size=None):
        super(PAGPlan, self).__init__()
        query_size = hidden_size if query_size is None else query_size
        self.score_size = hidden_size if score_size is None else score_size
        self.commit_layer = nn.Linear(query_size, plan_length, bias=False)

    def forward(
        self,
        query=None,
        proj_key=None,
        value=None,
        mask=None,
        beta=None,
        previous_out=None,
    ):
        # TODO remove beta and previous_out later.

        # We first project the query (the decoder state).
        # The projected keys (the encoder states) were already pre-computated.
        decoder_state = self.commit_layer(query)

        log_proba = F.log_softmax(decoder_state, dim=-1)
        commit_vector = F.gumbel_softmax(log_proba, 0.01)
        return commit_vector
```

The attend mechanism borrows heavily from the attention mechanism in Attend-NMT; the only difference is that if $\beta$ and $y$ tokens are generated then the embedding is like-wise added before the final alignment plan is generated.

```py
class PAGAttend(nn.Module):
    """Implements Plan Attend Generate (MLP) attention as per here:
    This performs "attend" step

    Francis Dutil, Caglar Gulcehre, Adam Trischler, Yoshua Bengio, Plan, Attend, Generate:
    Planning for Sequence-to-Sequence Models (NIPS 2017)

    If no history (previous action plan) or previous tokens are generated, it will
    degerate to Bahdanau Attention:
    "Neural Machine Translation by Jointly Learning to Align and Translate"
    """

    def __init__(
        self,
        hidden_size,
        plan_length=1,
        key_size=None,
        query_size=None,
        summary_layer=None,
        pred_embedding=None,
    ):
        super(PAGAttend, self).__init__()
        # We assume a bi-directional encoder so key_size is 2*hidden_size
        key_size = 2 * hidden_size if key_size is None else key_size
        query_size = hidden_size if query_size is None else query_size

        self.key_layer = nn.Linear(key_size, hidden_size, bias=False)

        self.query_layer = nn.Linear(query_size, hidden_size, bias=False)
        # used for alignment plan.
        self.energy_layer = nn.Linear(hidden_size, plan_length, bias=False)

        # generates the betas
        self.summary_layer = summary_layer
        self.prev_summary_layer = pred_embedding
        # this is used instead of the energy layer?
        # self.alignment_plan = nn.Linear(hidden_size, plan_length, bias=False)

        # to store attention scores
        self.alphas = None

    def forward(
        self,
        query=None,
        proj_key=None,
        value=None,
        mask=None,
        beta=None,
        previous_out=None,
    ):
        # mask is prev_y
        assert mask is not None, "mask is required"
        # https://github.com/Dutil/PAG/blob/3e9f9beac6072cdc1aabaa5f015574402b12929c/planning.py#L685

        # We first project the query (the decoder state).
        # The projected keys (the encoder states) were already pre-computated.
        decoder_state = self.query_layer(query)

        # if self.action_plan is None:
        #     print("action_plan None")
        #     beta = 0
        # else:
        #     print("action_plan", self.action_plan.shape)
        #     beta = self.summary_layer(self.action_plan.permute(0, 2, 1))
        #     print("beta", beta.shape)
        if beta is None:
            beta = 0
        if previous_out is None:
            previous_out = 0

        # print("decoder", decoder_state.shape)
        # beta = torch.tanh(self.summary_layer(self.action_plan))
        # decoder_token_proj = torch.tanh(self.proj_pred(mask))
        # decoder_state + beta + proj_key underneath

        # Calculate scores.
        scores = self.energy_layer(
            torch.tanh(decoder_state + proj_key + beta + previous_out)
        )
        scores = scores.permute(0, 2, 1)

        # Mask out invalid positions.
        # The mask marks valid positions so we invert it using `mask & 0`.
        # not true here...
        # scores.data.masked_fill_(mask == 0, -float("inf"))

        # Turn scores to probabilities.
        # only take first column for calculating
        # alpha_t = softmax(A_t[0])
        alphas = F.softmax(scores[:, [0], :], dim=-1)
        self.action_plan = scores
        self.alphas = alphas

        # The context vector is the weighted sum of the values.

        # context shape: [B, 1, 2D], alphas shape: [B, 1, M]
        context = torch.bmm(alphas, value)
        return context, alphas, scores.data
```

**Putting it all together**

To put it all together, we have to make use of the shift operation

```py
def shift(tensor, dim=1):
    """
    Shifts tensor along this dimension by one spot and fills in shifted
    with zeros; quickest way is to pad the target dim and then slice
    """
    pad = []
    for idx in range(len(tensor.shape)):
        if idx != dim:
            pad.extend([0, 0])
        else:
            pad.extend([1, 0])
    pad = pad[::-1]
    slice_ls = [slice(None) for _ in range(len(tensor.shape))]
    slice_ls[dim] = slice(1, None)
    tensor_pad = F.pad(tensor, pad=pad)
    return tensor_pad[slice_ls]
```

Rather than having some fancy for-loop, I implemented the switch through doing an additive sum across a binary representation of $g$ (that is I create a binary matrix which I then compute `g*X[recompute] + (1-g)X[shift])`. I find that this works "well enough" for my purposes.

```py
class PAGAttention(nn.Module):
    """
    Performs full attention as documented (minus a couple of shortcuts for now...)
    """

    @staticmethod
    def shift(tensor, dim=1):
        """
        Shifts tensor along this dimension by one spot and fills in shifted
        with zeros; quickest way is to pad the target dim and then slice
        """
        pad = []
        for idx in range(len(tensor.shape)):
            if idx != dim:
                pad.extend([0, 0])
            else:
                pad.extend([1, 0])
        pad = pad[::-1]
        slice_ls = [slice(None) for _ in range(len(tensor.shape))]
        slice_ls[dim] = slice(1, None)
        tensor_pad = F.pad(tensor, pad=pad)
        return tensor_pad[slice_ls]

    @staticmethod
    def create_binary_repr(tensor, shape):
        for _ in range(len(shape) - 1):
            tensor = tensor.unsqueeze(1)
        shape = list(shape)
        shape[0] = -1
        return tensor.expand(*shape)

    def __init__(self, plan, attend, generate):
        super(PAGAttention, self).__init__()
        self.plan = plan
        self.attend = attend
        self.generate = generate

    def forward(
        self,
        query=None,
        proj_key=None,
        value=None,
        mask=None,
        beta=None,
        previous_out=None,
        prev_action_plan=None,
        prev_commit_plan=None,
    ):
        """
        The central tenent in this approach is to reduce the amount of data
        which flows into the downstream layers (assuming that the evaluation)
        is expensive.

        We will operate using the commitment plan here to skip - for implementation
        reasons we won't actually "skip" but instead zero out parts of the solution
        This should operate correctly as we don't add "biases" to our data.
        """

        if prev_commit_plan is not None:
            next_action_vector = prev_commit_plan[:, 0, 0].round()
        else:
            next_action_vector = None

        if prev_action_plan is not None:
            shift_action_plan = self.shift(prev_action_plan, 1)
            # print("-----------")
            # print(prev_action_plan.shape)
            # print(self.shift(prev_action_plan, 1).shape)
            # print(prev_action_plan[0, :, 0])
            # print(self.shift(prev_action_plan, 1)[0, :, 0])
            # print("-----------")

        if prev_commit_plan is not None:
            shift_commit_plan = self.shift(prev_commit_plan, 2)
            # print(prev_commit_plan.shape)
            # print(self.shift(prev_commit_plan, 2).shape)

        # TODO check the previous commit weights
        # and move everything along - no need to eval if not needed here...

        # this already gracefully handles scenario when beta is None or 0
        context, attn_probs, action_plan = self.attend(
            query=query,
            proj_key=proj_key,
            value=value,
            mask=mask,
            beta=beta,
            previous_out=previous_out,
        )

        # repeat alignment uses context - set it to be None for now.
        commit_weights = self.plan(
            query=query,
            proj_key=proj_key,
            value=value,
            mask=mask,
            beta=beta,
            previous_out=previous_out,
        )

        # if in repeat - then weights don't update! Simple!
        update_weights = self.generate(
            query=query,
            proj_key=proj_key,
            value=value,
            mask=mask,
            beta=beta,
            previous_out=previous_out,
        )
        if (
            update_weights is not None
            and action_plan is not None
            and prev_action_plan is not None
        ):
            # update_weights = update_weights.expand(-1, -1, action_plan.size(2))
            action_plan = (update_weights * action_plan) + (
                update_weights * prev_action_plan
            )

        # finally double check whether it really should be updated or not via
        if next_action_vector is not None:
            action_plan_binary = self.create_binary_repr(
                next_action_vector, prev_action_plan.shape
            )
            commit_binary = self.create_binary_repr(
                next_action_vector, commit_weights.shape
            )
            action_plan = (action_plan_binary * action_plan) + (
                (1 - action_plan_binary) * shift_action_plan
            )
            commit_weights = (commit_binary * commit_weights) + (
                (1 - commit_binary) * shift_commit_plan
            )

        return context, attn_probs, action_plan, commit_weights
```

# Putting it all together

To put it altogether, it looks something like this:

```py
# set up network...
attention = PAGAttention(
    plan=PAGPlan(hidden_size, seq),
    attend=PAGAttend(
        hidden_size,
        seq,
        summary_layer=lambda x: torch.tanh(
            nn.Linear(score_size[1], score_size[1], bias=False)(x)
        ),
        pred_embedding=lambda x: torch.tanh(
            nn.Linear(trg_y.size(1), hidden_size, bias=False)(x)
        ),
    ),
    generate=PAGGenerate(hidden_size, seq, score_size=score_size[1]),
)

net = EncoderDecoder(
    Encoder(emb_size, hidden_size, num_layers=num_layers, dropout=dropout),
    Decoder(emb_size, hidden_size, attention, num_layers=num_layers, dropout=dropout),
    ContinuousEmbedding(emb_size),
    ContinuousEmbedding(emb_size),
    Generator(hidden_size, pred_shape),
)
```

At a later stage I'll try running this over a proper dataset to see the performance!

_Update 21 Feb 2020_

We can create a lighter weight version based on [Tensorflow port and tutorial](https://www.tensorflow.org/tutorials/text/nmt_with_attention).

```py
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
import math, copy, time
from torch.nn.utils.rnn import pack_padded_sequence, pad_packed_sequence


class Encoder(nn.Module):
    """Encodes a sequence of word embeddings
    """

    def __init__(self, input_size, hidden_size, embed_size=None, num_layers=1):
        super(Encoder, self).__init__()
        embed_size = hidden_size if embed_size is None else embed_size
        self.num_layers = num_layers
        self.embed = nn.Linear(input_size, embed_size)
        self.rnn = nn.GRU(
            embed_size, hidden_size, num_layers, batch_first=True, bidirectional=True
        )

    def forward(self, x, hidden=None):
        """
        Applies a bidirectional GRU to sequence of embeddings x.
        The input mini-batch x needs to be sorted by length.
        x should have dimensions [batch, time, dim].

        what does the util functions do? Ensure that all inputs have
        the same length!
        """
        x = self.embed(x)
        output, final = self.rnn(x, hidden)

        # final = final.permute(1, 0, 2)

        # we need to manually concatenate the final states for both directions
        fwd_final = final[0 : final.size(0) : 2]
        bwd_final = final[1 : final.size(0) : 2]
        final = torch.cat([fwd_final, bwd_final], dim=2)  # [num_layers, batch, 2*dim]
        final = final.permute(1, 0, 2)

        return output, final


class PlanAttention(nn.Module):
    """Implements "Plan Attend Generate" attention"""

    @staticmethod
    def shift(tensor, dim=1):
        """
        Shifts tensor along this dimension by one spot and fills in shifted
        with zeros; quickest way is to pad the target dim and then slice
        """
        pad = []
        for idx in range(len(tensor.shape)):
            if idx != dim:
                pad.extend([0, 0])
            else:
                pad.extend([1, 0])
        pad = pad[::-1]
        slice_ls = [slice(None) for _ in range(len(tensor.shape))]
        slice_ls[dim] = slice(1, None)
        tensor_pad = F.pad(tensor, pad=pad)
        return tensor_pad[slice_ls]

    @staticmethod
    def create_binary_repr(tensor, shape):
        for _ in range(len(shape) - 1):
            tensor = tensor.unsqueeze(1)
        shape = list(shape)
        shape[0] = -1
        return tensor.expand(*shape)

    def __init__(self, hidden_size, units, plan_length, output_size):
        super(PlanAttention, self).__init__()

        # We assume a bi-directional encoder so key_size is 2*hidden_size
        key_size = 2 * hidden_size
        self.plan_length = plan_length
        self.units = units

        self.w1 = nn.Linear(key_size, units * plan_length, bias=False)
        self.w2 = nn.Linear(key_size, units * plan_length, bias=False)
        self.w3 = nn.Linear(plan_length, 1)
        self.energy_layer = nn.Linear(units, 1, bias=False)
        self.action_plan_layer = nn.Linear(units, plan_length, bias=False)
        self.prev_decoder_layer = nn.Linear(output_size, units, bias=False)

        # commitment
        self.commit_layer = nn.Linear(key_size, plan_length, bias=False)

        # update weights
        self.update_w1 = nn.Linear(key_size, units * plan_length, bias=False)
        self.update_w2 = nn.Linear(key_size, units * plan_length, bias=False)
        self.update_context = nn.Linear(key_size, units * plan_length, bias=False)
        self.update_layer = nn.Linear(units, 1, bias=False)

    def forward(self, query=None, values=None, commit_plan=None, action_plan=None):
        # preliminary planning actions...
        if commit_plan is not None:
            next_action_vector = commit_plan[:, 0].round()
            shift_commit_plan = self.shift(commit_plan, 1)
        else:
            next_action_vector = None

        if action_plan is not None:
            shift_action_plan = self.shift(action_plan, 1)

        # hidden shape == (batch_size, hidden size)

        # score shape == (batch_size, max_length, 1)
        # we get 1 at the last axis because we are applying score to self.V
        # the shape of the tensor before applying self.V is (batch_size, max_length, units)
        if action_plan is not None:
            beta = torch.tanh(self.w3(action_plan.permute(0, 2, 1)))
            beta = beta.unsqueeze(3)
        else:
            beta = 0

        # e_ij = a(s_i-1, h_j)
        w1 = torch.tanh(self.w1(query))
        w1 = w1.view(w1.shape[0], w1.shape[1], self.plan_length, self.units)
        w2 = torch.tanh(self.w2(values))
        w2 = w2.view(w2.shape[0], w2.shape[1], self.plan_length, self.units)

        scores = self.energy_layer(w1 + w2 + beta)
        scores = scores.squeeze(3).permute(0, 2, 1)

        # Turn scores to probabilities.
        # attention_weights shape == (batch_size, max_length, 1)
        attention_weights = F.softmax(scores[:, [0], :], dim=-1)
        self.alphas = attention_weights

        # The context vector is the weighted sum of the values.
        context = torch.bmm(attention_weights, values)

        # compute commitment
        commit_proba = F.log_softmax(self.commit_layer(query).squeeze(1), dim=-1)
        commit_plan = F.gumbel_softmax(commit_proba, 0.01)
        print("commit_plan", commit_plan.shape)

        # compute update gate
        update_w1 = torch.tanh(self.update_w1(query))
        update_w1 = update_w1.view(
            update_w1.shape[0], update_w1.shape[1], self.plan_length, self.units
        )
        update_w2 = torch.tanh(self.update_w2(values))
        update_w2 = update_w2.view(
            update_w2.shape[0], update_w2.shape[1], self.plan_length, self.units
        )
        update_context = torch.tanh(self.update_context(context))
        update_context = update_context.view(
            update_context.shape[0],
            update_context.shape[1],
            self.plan_length,
            self.units,
        )

        update_vector = self.update_layer(update_w1 + update_w2 + update_context)
        update_vector = update_vector.squeeze(3)
        update_vector = update_vector.permute(0, 2, 1)
        # context shape: [B, 1, 2 * hidden_size], alphas shape: [B, 1, max_length]

        # set action plan
        if action_plan is None:
            action_plan = scores
        else:
            action_plan = ((1 - update_vector) * shift_action_plan) + (
                update_vector * scores
            )

        # process all updates accordingly
        if next_action_vector is not None:
            # update all things selectively
            action_plan_binary = self.create_binary_repr(
                next_action_vector, shift_action_plan.shape
            )
            commit_binary = self.create_binary_repr(
                next_action_vector, shift_commit_plan.shape
            )

            action_plan = (action_plan_binary * action_plan) + (
                (1 - action_plan_binary) * shift_action_plan
            )
            commit_plan = (commit_binary * commit_plan) + (
                (1 - commit_binary) * shift_commit_plan
            )

        return context, attention_weights, action_plan, commit_plan


class Decoder(nn.Module):
    def __init__(
        self,
        input_size,
        output_size,
        hidden_size,
        embed_size=None,
        num_layers=1,
        attention_config={},
    ):
        super(Decoder, self).__init__()
        embed_size = hidden_size if embed_size is None else embed_size
        self.num_layers = num_layers
        self.hidden_size = hidden_size
        self.embed = nn.Linear(output_size, embed_size)
        self.rnn = nn.GRU(
            embed_size + hidden_size * 2,
            hidden_size,
            num_layers,
            batch_first=True,
            bidirectional=True,
        )
        self.attention = PlanAttention(**attention_config)
        self.projection = nn.Linear(hidden_size * 2, output_size)

    def forward(self, x, hidden, enc_output, commit_plan=None, action_plan=None):
        x = self.embed(x)

        # enc_output shape == (batch_size, max_length, hidden_size)
        context_vector, attention_weights, action_plan, commit_plan = self.attention(
            hidden, enc_output, commit_plan, action_plan
        )

        x = torch.cat([x, context_vector], dim=-1)

        output, state = self.rnn(x)
        x = self.projection(output)
        return x, state, attention_weights, action_plan, commit_plan


enc = Encoder(10, 12, 13)
x = torch.from_numpy(np.random.normal(size=(6, 3, 10))).type(torch.FloatTensor)

# always the last decoder output (i.e. y_t)
d_x = torch.from_numpy(np.random.normal(size=(6, 1, 9))).type(torch.FloatTensor)

sample_output, sample_hidden = enc(x)
print("input", x.shape)
print("sample_output", sample_output.shape, "sample_hidden", sample_hidden.shape)

attention_layer = PlanAttention(12, 9, 5, 9)
attention_result, attention_weights, action_plan, commit_plan = attention_layer(
    sample_hidden, sample_output
)
print(
    "attention_result",
    attention_result.shape,
    "attention_weights",
    attention_weights.shape,
)

attention_result, attention_weights, action_plan, commit_plan = attention_layer(
    sample_hidden, sample_output, None, action_plan
)
print(
    "attention_result",
    attention_result.shape,
    "attention_weights",
    attention_weights.shape,
    "action_plan",
    action_plan.shape,
)


decoder = Decoder(
    10,
    9,
    12,
    14,
    attention_config=dict(hidden_size=12, units=9, plan_length=5, output_size=9),
)
(
    sample_decoder_output,
    decoder_state,
    attention_weights,
    action_plan,
    commit_plan,
) = decoder(d_x, sample_hidden, sample_output)
print("sample_decoder_output", sample_decoder_output.shape)
print(
    "decoder_state", decoder_state.shape, "attention_weights", attention_weights.shape
)

(
    sample_decoder_output,
    decoder_state,
    attention_weights,
    action_plan,
    commit_plan,
) = decoder(d_x, sample_hidden, sample_output, None, action_plan)
print("sample_decoder_output", sample_decoder_output.shape)
print(
    "decoder_state", decoder_state.shape, "attention_weights", attention_weights.shape
)
```
