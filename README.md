Welcome to the official repository for **Siren**, a project aimed at understanding and mitigating harmful behaviors in large language models (LLMs). 
This repository contains the resources for reproducing the experiments described in our work.

## Repository Structure

### `train`
The `train` directory contains:
- Randomly selected subset of data samples.
- Configuration `.yaml` files for SFT (Supervised Fine-Tuning) and DPO (Direct Preference Optimization) training, prepared using the [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory).

### `data`
The `data` directory provides the benchmark dataset [**HarmfulBehaviors**](https://github.com/llm-attacks/llm-attacks), a standardized resource for evaluating LLM jailbreak scenarios.


### `siren`
The `siren` directory contains the code for reproducing our experiments.


---

## Installation and Usage
To use Siren, follow these steps:
- You might need to **download LLaMA-3-8B into a local directory first** according to the instructions of [meta-llama/Meta-Llama-3-8B-Instruct](https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct). 

```bash
git clone https://github.com/YiyiyiZhao/siren.git
cd siren
pip install -r requirements.txt
```

### Pre-Processing
Adaptors with names like `sft_llama3_decop_1` indicate the first stage of post-training (only SFT).
Therefore, you need to **first merge the SFT adaptors with the public LLMs (e.g., LLaMA-3)**.
The DPO post-training stage is applied to the merged models.

```bash
cd siren
```
- Make sure to modify lines 42 to 52 in the `merge_sft_base.py` script, replacing the placeholder paths with **the actual paths where you want to save the merged model (SFT with Base)**.
- Make sure to modify line 27 in the `merge_sft_base.py` script, replacing the placeholder paths with **the actual path where [meta-llama/Meta-Llama-3-8B-Instruct](https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct) downloaded**.

```bash
python merge_sft_dpo.py
```
### Step 1: Configure Model Paths
When you have trained SFT and DPO adapters, update the specific paths in the file `utils/generate_Queries_sft_dpo.py`, lines 11 to 30:

```python
self.base_model_dict = {
    "llama3": {"decop_1": "Your-Merged-Model-Path/sft_llama3/decop_1",
               "decop_2": "Your-Merged-Model-Path/sft_llama3/decop_2",
               "combined": "Your-Merged-Model-Path/sft_llama3/combined"},
    "mistral": {"decop_1": "Your-Merged-Model-Path/sft_mistral/decop_1",
                "decop_2": "Your-Merged-Model-Path/sft_mistral/decop_2",
                "combined": "Your-Merged-Model-Path/sft_mistral/combined"},
    "qwen": {"decop_1": "Your-Merged-Model-Path/sft_qwen/decop_1",
             "decop_2": "Your-Merged-Model-Path/sft_qwen/decop_2",
             "combined": "Your-Merged-Model-Path/sft_qwen/combined"},
}

self.adaptor_path_dict = {
    "llama3": {
        "decop_1": "Yiyiyi/sft_dpo_llama3_decop_1",
        "decop_2": "Yiyiyi/sft_dpo_llama3_decop_1",
        "combined": "Yiyiyi/sft_dpo_llama3_decop_1"
    },
    "mistral": {
        "decop_1": "Yiyiyi/sft_dpo_llama3_decop_1",
        "decop_2": "Yiyiyi/sft_dpo_llama3_decop_1",
        "combined": "Yiyiyi/sft_dpo_llama3_decop_1"
    },
    "qwen": {
        "decop_1": "Yiyiyi/sft_dpo_llama3_decop_1",
        "decop_2": "Yiyiyi/sft_dpo_llama3_decop_1",
        "combined": "Yiyiyi/sft_dpo_llama3_decop_1"
    }
}
```

***You can manually download the adapters [here](https://huggingface.co/Yiyiyi/models).  
If you keep the `self.adaptor_path_dict` the same as shown, it will download the adapters automatically.***

![Adapters](model_adapters.png)

### Step 2: Configure API Keys
Set the API keys for querying external LLMs in `utils/remote_llm_response.py`, lines 10 to 22:

```python
self.gpt_api_keys = os.environ.get("OPENAI_API_KEYS")
self.gpt_api_key_list = self.gpt_api_keys.split(',')
self.gpt_base_url = os.environ.get("OPENAI_BASE_URL")

self.claude_api_keys = os.environ.get("ANTHROPIC_API_KEYS")
self.claude_api_key_list = self.claude_api_keys.split(',')
self.claude_base_url = os.environ.get("ANTHROPIC_BASE_URL")

self.gemini_api_keys = os.environ.get("GEMINI_API_KEYS")
self.gemini_api_key_list = self.gemini_api_keys.split(',')
self.gemini_base_url = os.environ.get("GEMINI_BASE_URL")
```

During our experiments, each remote LLM has more than one API key. You can modify this part to fit your settings in the `utils/remote_llm_response.py` file. 

---

## Commands

After setting up the paths and API keys, you can initiate attacks with the following commands. Refer to `commands.sh` or `commands_remote.sh` for more examples.

### Examples:

```bash
CUDA_VISIBLE_DEVICES=0,1 python main_sft_dpo.py --attacker_model llama3 --adaptor decop_1 --victim_model mistral > sft_dpo_llama3_1_mistral.log 2>&1 &
CUDA_VISIBLE_DEVICES=1,2 python main_sft_dpo.py --attacker_model mistral --adaptor combined --victim_model llama3 > sft_dpo_mistral_com_llama3.log 2>&1 &
```

or

```bash
CUDA_VISIBLE_DEVICES=0 python main_sft_dpo.py --attacker_model qwen --adaptor decop_2 --victim_model claude-3-5-haiku-20241022 > sft_dpo_qwen_2_claude.log 2>&1 &
CUDA_VISIBLE_DEVICES=4 python main_sft_dpo.py --attacker_model llama3 --adaptor decop_1 --victim_model gpt-4o-2024-08-06 > sft_dpo_llama3_1_gpt.log 2>&1 &
CUDA_VISIBLE_DEVICES=1 python main_sft_dpo.py --attacker_model mistral --adaptor decop_2 --victim_model gemini-1.5-pro-latest > sft_dpo_mistral_2_ge.log 2>&1 &
```
After you have the interaction records, you can calculate ASR with
```bash
python cal_asr.py
```

### Cite: 
```
@inproceedings{DBLP:conf/acsac/ZhaoZ25,
  author       = {Yi Zhao and
                  Youzhi Zhang},
  title        = {Siren: {A} Learning-Based Multi-Turn Attack Framework for Simulating
                  Real-World Human Jailbreak Behaviors},
  booktitle    = {{IEEE} Annual Computer Security Applications Conference, {ACSAC} 2025,
                  Honolulu, HI, USA, December 8-12, 2025},
  pages        = {1209--1220},
  publisher    = {{IEEE}},
  year         = {2025},
  url          = {https://doi.org/10.1109/ACSAC67867.2025.00095},
  doi          = {10.1109/ACSAC67867.2025.00095},
  timestamp    = {Thu, 12 Mar 2026 16:35:22 +0100},
  biburl       = {https://dblp.org/rec/conf/acsac/ZhaoZ25.bib},
  bibsource    = {dblp computer science bibliography, https://dblp.org}
}
```
