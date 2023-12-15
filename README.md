---
license: apache-2.0
language:
- fr
- it
- de
- es
- en
inference: false
widget:
  - text: "[INST] You are a pirate chatbot who always responds with Arr and pirate speak!\nThere's a llama on my lawn, how can I get rid of him? [/INST]"
    output:
      text: "Arr, shiver me timbers! Ye have a llama on yer lawn, ye say? Well, that be a new one for me! Here's what I'd suggest, arr:\n\n1. Firstly, ensure yer safety. Llamas may look gentle, but they can be protective if they feel threatened.\n2. Try to make the area less appealing to the llama. Remove any food sources or water that might be attracting it.\n3. Contact local animal control or a wildlife rescue organization. They be the experts and can provide humane ways to remove the llama from yer property.\n4. If ye have any experience with animals, you could try to gently herd the llama towards a nearby field or open space. But be careful, arr!\n\nRemember, arr, it be important to treat the llama with respect and care. It be a creature just trying to survive, like the rest of us."
---
# Model Card for Mixtral-8x7B
The Mixtral-8x7B Large Language Model (LLM) is a pretrained generative Sparse Mixture of Experts. The Mixtral-8x7B outperforms Llama 2 70B on most benchmarks we tested.

For full details of this model please read our [release blog post](https://mistral.ai/news/mixtral-of-experts/).

## Warning
This repo contains weights that are compatible with [vLLM](https://github.com/vllm-project/vllm) serving of the model as well as Hugging Face [transformers](https://github.com/huggingface/transformers) library. It is based on the original Mixtral [torrent release](magnet:?xt=urn:btih:5546272da9065eddeb6fcd7ffddeef5b75be79a7&dn=mixtral-8x7b-32kseqlen&tr=udp%3A%2F%http://2Fopentracker.i2p.rocks%3A6969%2Fannounce&tr=http%3A%2F%http://2Ftracker.openbittorrent.com%3A80%2Fannounce), but the file format and parameter names are different. Please note that model cannot (yet) be instantiated with HF.

## Instruction format

This format must be strictly respected, otherwise the model will generate sub-optimal outputs.

The template used to build a prompt for the Instruct model is defined as follows:
```
<s> [INST] Instruction [/INST] Model answer</s> [INST] Follow-up instruction [/INST]
```
Note that `<s>` and `</s>` are special tokens for beginning of string (BOS) and end of string (EOS) while [INST] and [/INST] are regular strings.

As reference, here is the pseudo-code used to tokenize instructions during fine-tuning:
```python
def tokenize(text):
    return tok.encode(text, add_special_tokens=False)

[BOS_ID] + 
tokenize("[INST]") + tokenize(USER_MESSAGE_1) + tokenize("[/INST]") +
tokenize(BOT_MESSAGE_1) + [EOS_ID] +
…
tokenize("[INST]") + tokenize(USER_MESSAGE_N) + tokenize("[/INST]") +
tokenize(BOT_MESSAGE_N) + [EOS_ID]
```

In the pseudo-code above, note that the `tokenize` method should not add a BOS or EOS token automatically, but should add a prefix space. 

## Run the model

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistralai/Mixtral-8x7B-Instruct-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

model = AutoModelForCausalLM.from_pretrained(model_id)

text = "Hello my name is"
inputs = tokenizer(text, return_tensors="pt")

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

By default, transformers will load the model in full precision. Therefore you might be interested to further reduce down the memory requirements to run the model through the optimizations we offer in HF ecosystem:

### In half-precision

Note `float16` precision only works on GPU devices

<details>
<summary> Click to expand </summary>

```diff
+ import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistralai/Mixtral-8x7B-Instruct-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

+ model = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype=torch.float16).to(0)

text = "Hello my name is"
+ inputs = tokenizer(text, return_tensors="pt").to(0)

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```
</details>

### Lower precision using (8-bit & 4-bit) using `bitsandbytes`

<details>
<summary> Click to expand </summary>

```diff
+ import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistralai/Mixtral-8x7B-Instruct-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

+ model = AutoModelForCausalLM.from_pretrained(model_id, load_in_4bit=True)

text = "Hello my name is"
+ inputs = tokenizer(text, return_tensors="pt").to(0)

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```
</details>

### Load the model with Flash Attention 2

<details>
<summary> Click to expand </summary>

```diff
+ import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistralai/Mixtral-8x7B-Instruct-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

+ model = AutoModelForCausalLM.from_pretrained(model_id, use_flash_attention_2=True)

text = "Hello my name is"
+ inputs = tokenizer(text, return_tensors="pt").to(0)

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```
</details>

## Limitations

The Mixtral-8x7B Instruct model is a quick demonstration that the base model can be easily fine-tuned to achieve compelling performance. 
It does not have any moderation mechanisms. We're looking forward to engaging with the community on ways to
make the model finely respect guardrails, allowing for deployment in environments requiring moderated outputs.

# The Mistral AI Team
Albert Jiang, Alexandre Sablayrolles, Arthur Mensch, Blanche Savary, Chris Bamford, Devendra Singh Chaplot, Diego de las Casas, Emma Bou Hanna, Florian Bressand, Gianna Lengyel, Guillaume Bour, Guillaume Lample, Lélio Renard Lavaud, Louis Ternon, Lucile Saulnier, Marie-Anne Lachaux, Pierre Stock, Teven Le Scao, Théophile Gervet, Thibaut Lavril, Thomas Wang, Timothée Lacroix, William El Sayed.