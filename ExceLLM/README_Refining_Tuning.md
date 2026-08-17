# A Tiny LLM in a Spreadsheet, With Its Training Opened Up

A working next-word predictor trained on *Alice's Adventures in Wonderland*, running entirely in Excel formulas: a 10-word context window, attention weights the model learned by itself, and, unusually, the training step itself recomputed live: forward pass, loss, backpropagation, and a gradient-descent weight update. No macros, no add-ins.

## What it is

A real (very small) neural language model. It reads the last 10 words, blends them with learned attention, passes the blend through a 16-dim hidden layer with tanh, and scores all 2,627 words of the book's vocabulary. It was trained by Adam/gradient descent on the 26,673 ten-word windows of the book, and the learned weights are baked into the sheets as plain numbers.

The headline result: the 10 position-attention scores were trainable parameters, and the model discovered recency on its own. Its learned attention weights rise smoothly from 2.7% on the oldest word to 40.2% on the most recent one. Training loss fell from 6.97 to 2.59 over 120 epochs; the model now predicts 47% of the book's next words exactly right.

## The tabs

| Tab | Contents |
| --- | --- |
| README | Quick start, inside the workbook |
| Prompt | The interactive model: 10 yellow word cells, every step visible, top-10 predictions |
| Training Example | One real passage from chapter 1, its forward pass, and the loss = -LN(p(target)) |
| Backpropagation | The actual gradients for that example, live: g = p - y, dOut, dh, dpre, dW, dc, then new = old - lr x gradient |
| Circuits & Activations | Live trace of WHY the model picked its word: neuron contributions to the logit, then what feeds the busiest neuron |
| How It Works | Plain-language explanation of every step, forward and backward |
| Tokenizer | Text to token IDs, with a live demo cell |
| Vocabulary | All 2,627 distinct words plus <unk>, with frequencies |
| Embeddings, Hidden Weights, Output Weights | The learned matrices (84,362 parameters total) |
| Prediction | One MMULT scores the whole vocabulary; softmax with temperature |

## The Gradient Descent Lab

Found a prediction you dislike? The Gradient Descent Lab tab lets you pick any 10-word context and any desired next word, then runs 20 live gradient-descent steps on the output weights until the model agrees. Each step applies the exact rule from the Backpropagation tab, new = old - lr x gradient, to just two rows of the output matrix: your word's and its strongest rival's. Because everything else is frozen, the step-by-step table is mathematically exact, and you can watch p(your word) climb row by row: that curve IS gradient descent. The tab ends with the 32 numbers that changed, a check that the updated weights reproduce the table's final logit, and the overfitting lesson: crank the learning rate to 1.0 and see why hammering one example damages the rest (catastrophic forgetting).

Default example: chapter 8's "a large rose tree stood near the entrance of the" tuned toward "garden", which goes from a shaky 28% to a confident 96%.

## Paste a sentence

On both the Prompt tab and the Lab there is now a wide yellow cell where you can paste a whole sentence. Its last 10 words are split into the context automatically (punctuation stripped), so you never have to distribute words across cells by hand. Clear it to go back to typing individual words. This matters because a full sentence typed into a single word cell is not any word in the vocabulary, so it would silently become one <unk> token.

## Six MMULTs

Three run the model forward: attention mixing (1x10 times 10x16), the hidden layer (1x16 times 16x16), and scoring (2,628x16 times 16x1). Three run it backward on the Backpropagation tab: dh (1x2,628 times 2,628x16), dW as an outer product (16x1 times 1x16), and dc (1x16 times 16x16). Training really is roughly the forward pass in reverse, and the workbook shows both directions.

## Things to try

- Keep the default prompt and watch it correctly continue "...making a daisy chain would" with "be" at 79% confidence.
- On Training Example, the model wrongly guesses "it" where the book says "alice": exactly why its gradients are large. Change the target word to something absurd and watch the loss explode.
- On Backpropagation, find the neuron where dpre = 0 exactly: a saturated tanh passing no gradient, live. Then raise the learning rate to 0.5 and watch the updated weight matrix overshoot: training instability you can cause yourself.
- On Circuits & Activations, change the prompt and watch different neurons take over the prediction.
- Set temperature to 0.3 (confident) or 3.0 (chaotic). Shift attention onto old positions and watch quality collapse.

## Honest differences from a real LLM

Context 10 words instead of hundreds of thousands. Dimensions 16 instead of about 10,000. One layer instead of about 100. Position-based learned attention here; content-based query/key/value attention with many heads per layer there. Whole-word tokens here, subwords there. 84,362 parameters here, up to trillions there. But every step, forward and backward, is the real mechanism at about 1:10,000,000 scale.

Text source: Project Gutenberg eBook #11. The MMULT arrays ship pre-entered; Excel 365 and older Ctrl+Shift+Enter Excel both just recalculate.
