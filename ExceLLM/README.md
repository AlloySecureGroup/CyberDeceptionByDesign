# A Tiny LLM in a Spreadsheet

A complete, working next-word predictor trained on *Alice's Adventures in Wonderland*, running entirely in Excel formulas. No macros, no add-ins, no internet. Open `tiny_llm.xlsx`, go to the **Prompt** tab, and type.

## What it is

This workbook contains a real (very small) neural language model. It was trained with gradient descent on all 26,682 consecutive word pairs in the book, and the learned weights are baked into the sheets as plain numbers. The spreadsheet then performs, live, the same pipeline a large language model uses:

1. **Tokenize**: your words are looked up in the Vocabulary tab with `MATCH`. The row number is the token ID. Unknown words become token 1, `<unk>`, and are ignored.
2. **Embed**: each token ID looks up its learned 16-number vector with `INDEX`.
3. **Attend**: editable attention scores pass through a softmax, and `MMULT` #1 blends the three word vectors into one context vector (1x3 times 3x16).
4. **Hidden layer**: `MMULT` #2 multiplies the context by a learned 16x16 matrix, then `TANH` squashes each value. The nonlinearity is what makes it a neural network.
5. **Score every word**: `MMULT` #3, on the Prediction tab, multiplies the 2,628x16 output matrix by the hidden vector, producing 2,628 logits in a single array formula.
6. **Softmax and predict**: `EXP((logit - max) / temperature)` normalized to probabilities, ranked with `LARGE` and `MATCH`. The top word is the prediction; the top 10 are shown with probability bars.

## The tabs

| Tab | Contents |
| --- | --- |
| README | Quick start, inside the workbook |
| Prompt | The interactive model. Yellow cells are inputs; every step is visible |
| How It Works | Plain-language explanation of each step and of the training process |
| Tokenizer | Walk-through of text to token IDs, with a live demo cell |
| Vocabulary | All 2,627 distinct words of the book plus `<unk>`, with frequencies |
| Embeddings | The learned input embedding matrix (2,628 x 16) |
| Hidden Weights | The learned 16 x 16 hidden layer matrix |
| Output Weights | The learned output projection matrix (2,628 x 16) |
| Prediction | Where one MMULT scores the whole vocabulary, then softmax |

## Things to try

- The default prompt, "off with her", lets the Queen of Hearts finish her own sentence.
- "said the march" puts about 85% of the probability on "hare". Also try "the white rabbit", "a golden key", "the mock turtle".
- Set temperature to 0.3 for a confident model, 3.0 for a chaotic one.
- Move the attention weight from the last word to the first and watch predictions degrade. The model was trained to read the most recent word, just as real attention heads learn where to look.
- Type a word the book never uses ("computer") and watch it become `<unk>`.

## How it was trained

Architecture: input embeddings (2,628 x 16), a 16x16 hidden layer with tanh, and an output projection (2,628 x 16), for 84,352 parameters total. Trained for 120 epochs with the Adam optimizer, minimizing cross-entropy on next-word prediction. Loss fell from 7.49 (random guessing over the vocabulary) to 3.62. Text source: Project Gutenberg eBook #11.

## Honest differences from a real LLM

Context window of 3 words instead of hundreds of thousands. Embedding size 16 instead of about 10,000. One layer instead of about a hundred. Hand-set attention scores instead of learned query/key/value attention with many heads. Whole-word tokens instead of subword pieces. 84 thousand parameters instead of up to trillions. But the pipeline you can trace on the Prompt tab is, step for step, the real thing.

## Notes

The MMULT array formulas ship pre-entered. In Excel 365 they simply recalculate; in older Excel they are legacy Ctrl+Shift+Enter arrays and also just work. Everything recomputes instantly when you edit a yellow cell.
