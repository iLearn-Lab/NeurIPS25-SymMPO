# SymMPO: Mitigating Hallucination Through Theory-Consistent Symmetric Multimodal Preference Optimization

<div align="center">

<a target="_blank" href="https://github.com/Liuwq-bit">Wenqi&#160;Liu</a><sup>1</sup>,
<a target="_blank" href="https://scholar.google.com/citations?user=29gP4okAAAAJ">Xuemeng&#160;Song</a><sup>2</sup>,
<a target="_blank" href="https://scholar.google.com/citations?user=tvPOeFQAAAAJ">Jiaxi&#160;Li</a><sup>3</sup>,
<a target="_blank" href="https://scholar.google.com/citations?user=im-bS2YAAAAJ">Yinwei&#160;Wei</a><sup>1</sup>,
<a target="_blank" href="https://scholar.google.com/citations?user=VWunnXEAAAAJ">Zheng&#160;Na</a><sup>4</sup>,
<a target="_blank" href="https://scholar.google.com/citations?user=aZZfn90AAAAJ">Jianhua&#160;Yin</a><sup>1</sup>,
<a target="_blank" href="https://scholar.google.com/citations?user=yywVMhUAAAAJ">Liqiang&#160;Nie</a><sup>5</sup>
<br>
<sup>1</sup>Shandong University&#160;&#160;&#160;
<sup>2</sup>Southern University of Science and Technology&#160;&#160;&#160;
<sup>3</sup>University of Georgia&#160;&#160;&#160;
<br>
<sup>4</sup>National University of Singapore&#160;&#160;&#160;
<sup>5</sup>Harbin Institute of Technology, Shenzhen&#160;&#160;&#160;
<br />

<a href='https://arxiv.org/abs/2506.11712'><img src='https://img.shields.io/badge/Paper-Arxiv-red'></a>
<a href='https://huggingface.co/Tanh01/SymMPO_similar'><img src='https://img.shields.io/badge/Model-7B/13B-yellow'></a>
<a href='https://huggingface.co/datasets/Tanh01/SymMPO_Dataset'><img src='https://img.shields.io/badge/Dataset-HF-blue'></a>

</div>

<p align="center">
  <img src="asset/symmpo.png" width="95%" />
</p>

---

## Updates

- [04/2026] Formatted README.
- [09/2025] We release the training code, [model weights](https://huggingface.co/Tanh01/SymMPO_similar), and [dataset](https://huggingface.co/datasets/Tanh01/SymMPO_Dataset).
- [09/2025] SymMPO was accepted by NeurIPS 2025! 🎉🎉🎉
- [06/2025] We release the [arXiv paper](https://arxiv.org/abs/2506.11712).

---

## Introduction

We present **SymMPO**, a framework for mitigating hallucination in multimodal large language models (MLLMs). Our method introduces a theory-consistent symmetric multimodal preference optimization approach that addresses the hallucination problem from a principled perspective. This repository provides the official implementation, pretrained checkpoints, and evaluation scripts built on top of [LLaVA](https://github.com/haotian-liu/LLaVA).

---


## Project Structure

```text
.
├── asset/                  # Figures
├── eval/                   # Evaluation scripts (HallusionBench, Object-HalBench, MMHal, AMBER, MMSTAR)
├── llava/                  # LLaVA model code (architecture, encoders, projectors)
├── muffin/                 # Training & data utilities
├── script/
│   ├── train/              # Training scripts (full / LoRA)
│   └── eval/               # Evaluation shell scripts
├── run.sh                  # Quick-start training entry
├── requirements.txt
└── README.md
```

---

## Installation

Our codebase requires CUDA version 11.8.

```bash
conda create -n symmpo python=3.10 -y
conda activate symmpo
pip install -r requirements.txt
```

---

## Checkpoints / Models

- **SymMPO Model**: [Tanh01/SymMPO_similar](https://huggingface.co/Tanh01/SymMPO_similar)

Additionally, download the following pretrained models:

| Model | Link |
|---|---|
| LLaVA-v1.5-7B | [liuhaotian/llava-v1.5-7b](https://huggingface.co/liuhaotian/llava-v1.5-7b) |
| CLIP ViT-L/14@336 | [openai/clip-vit-large-patch14-336](https://huggingface.co/openai/clip-vit-large-patch14-336) |

After downloading, update the model paths:

1. Set the LLaVA model path in the 3rd line of `run.sh`.
2. Set the CLIP model path in:
   - The 4th line of `run.sh`
   - The 6th line of `llava/model/multimodal_encoder/builder.py`
   - The 14th line of `llava/model/multimodal_encoder/clip_encoder.py`

---

## Dataset

- **Training Dataset**: [SymMPO_Dataset](https://huggingface.co/datasets/Tanh01/SymMPO_Dataset)

Download and place the data according to the path specified in `run.sh`.

---

## Usage

### Training

```bash
bash run.sh
```

The default configuration in `run.sh`:

```bash
bash script/train/llava15_train_main.sh \
    SymMPO_test \
    "[Path of your LLaVA model]" \
    "[Path of your vision tower model]" \
    demo_data/similar \
    0,1,2,3 \
    5e-6 \
    0.5
```

### Evaluation

During evaluation, HallusionBench / Object-HalBench / MMHal-Bench require assessment using DeepSeek-V3 / GPT-3.5 / GPT-4.

#### HallusionBench

1. Download [Questions and Annotations](https://github.com/tianyi-lab/HallusionBench/blob/main/HallusionBench.json) and [Figures](https://drive.google.com/file/d/1eeO1i0G9BSZTE1yd5XeFwmrbe1hwyf_0/view?usp=sharing).
2. Run evaluation:

```bash
bash script/eval/eval_hallusion.sh [ckpt_path] [base_path or "No"] [YOUR_DEEPSEEK_API_KEY] [GPU_ID]
```

> We default to DeepSeek-V3. Replace `{YOUR_DEEPSEEK_API_KEY}` with a valid key, or modify [line 48](https://github.com/Liuwq-bit/SymMPO/blob/master/eval/hallusion_evaluation.py#L48) in `eval/hallusion_evaluation.py`.

#### Object-HalBench

1. Download data from [COCO](http://images.cocodataset.org/annotations/annotations_trainval2014.zip).
2. Install supplement models:

```python
import nltk
nltk.download('wordnet')
nltk.download('punkt')
```

```bash
python -m spacy download en_core_web_trf
```

3. Run evaluation:

```bash
bash script/eval/eval_objhal.sh [ckpt_path] [base_path or "No"] [YOUR_OPENAI_API_KEY] [GPU_ID]
```

> We default to **gpt-3.5-turbo-0125**. Replace `{YOUR_OPENAI_API_KEY}` with a valid key, or modify [line 51](https://github.com/Liuwq-bit/SymMPO/blob/master/eval/gpt4_grpc.py#L51) in `eval/gpt4_grpc.py`.

#### MMHal-Bench

1. Download data from [MMHal-Bench](https://drive.google.com/file/d/1mQyAbeGgRyiVV6qjVkUI1uY_g9E-bDTH/view?usp=sharing).
2. Run evaluation:

```bash
bash script/eval/eval_mmhal.sh [ckpt_path] [base_path or "No"] [YOUR_OPENAI_API_KEY] [GPU_ID]
```

> We default to **gpt-4-1106-preview**. Replace `{YOUR_OPENAI_API_KEY}` with a valid key, or modify [line 51](https://github.com/Liuwq-bit/SymMPO/blob/master/eval/gpt4_grpc.py#L51) in `eval/gpt4_grpc.py`.

#### AMBER

1. Download AMBER [data](https://github.com/junyangwang0410/AMBER/tree/master) and [images](https://drive.google.com/file/d/1MaCHgtupcZUjf007anNl4_MV0o4DjXvl/view?usp=sharing).
2. Install supplement model:

```bash
python -m spacy download en_core_web_lg
```

3. Run evaluation:

```bash
bash script/eval/eval_amber.sh [ckpt_path] [base_path or "No"] [GPU_ID] [data_dir]
```

#### MMSTAR

1. Download data from [MMSTAR](https://huggingface.co/datasets/Lin-Chen/MMStar).
2. Run evaluation:

```bash
bash script/eval/eval_mmstar.sh [ckpt_path] [base_path or "No"] [GPU_ID] [data_dir]
```

---

## Citation

If you find our work helpful, please consider citing:

```bibtex
@inproceedings{
  liu2025mitigating,
  title={Mitigating Hallucination Through Theory-Consistent Symmetric Multimodal Preference Optimization},
  author={Wenqi Liu and Xuemeng Song and Jiaxi Li and Yinwei Wei and Na Zheng and Jianhua Yin and Liqiang Nie},
  booktitle={The Thirty-ninth Annual Conference on Neural Information Processing Systems},
  year={2025},
  url={https://openreview.net/forum?id=tIW29IpCwG}
}
```

---

## Acknowledgement

- [TPO](https://github.com/topic-overwrite/topic-level-overwrite) and [RLAIF-V](https://github.com/RLHF-V/RLAIF-V): This work extends the implementations provided by these projects.
- [LLaVA](https://github.com/haotian-liu/LLaVA): The training process was carried out on the LLaVA model.
