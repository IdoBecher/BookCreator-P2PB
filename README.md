# BookCreator-P2PB — From Prompt to Picture-Book  
*An end-to-end, reproducible pipeline for five-scene picture-book generation using SD 1.5 + IP-Adapter with CLIP-based selection.*

> **Goal.** Keep the **same main character** recognizable across scenes, while **layouts stay diverse** and each image matches its **scene caption**.

---

## 🌐 Live demo (GitHub Pages)

- Project hub: **https://idobecher.github.io/BookCreator-P2PB/**  

Storybook examples (HTML):
- `docs/rainbow-kittens-adventure.html`  
- `docs/puppy-playdate.html`  
- `docs/robot-waiter-at-the-table.html`

---

## ✨ What this repo contains

1. **LLM text module** – refines title/character and generates **exactly 5** short, visual scenes (with retries + de-dup).
2. **Reference module** – creates/accepts a **reference portrait** and extracts a **focal crop** (category-agnostic) to guide identity.
3. **Rendering** – Stable Diffusion **v1.5** + **IP-Adapter (SD15)**, conditioning on **both** reference images (focus + full) with **shot-aware** scales.
4. **Selection** – samples a small grid and picks the best per scene using **OpenAI CLIP** cosine (text-first) with **reroll** when scores are low.
5. **Export** – saves `scene_best/scene_1..5.png`, `story.json`, and a **storybook HTML** (+ optional PDF).
6. **Publish** – everything in `docs/` is served on **GitHub Pages**.


---

## 🧭 Repository layout

```text
BookCreator-P2PB/
├─ code/
│  └─ storybook.ipynb      # end-to-end Colab/Local notebook
├─ docs/                           # GitHub Pages site
│  ├─ index.html                    # Simple gallery/landing page
│  ├─ puppy-playdate.html
│  ├─ puppy-playdate.pdf
│  ├─ puppy-playdate.png
│  ├─ rainbow-kittens-adventure.html
│  ├─ rainbow-kittens-adventure.pdf
│  ├─ rainbow-kittens-adventure.png
│  ├─ robot-waiter-at-the-table.html
│  ├─ robot-waiter-at-the-table.pdf
│  └─ robot-waiter-at-the-table.png
└─ README.md
```


---

## 📦 Installation

```bash
# Python 3.10+ recommended, CUDA if available
pip install -U diffusers transformers accelerate torch torchvision   sentencepiece safetensors opencv-python-headless   git+https://github.com/openai/CLIP.git
```

**Model weights**
- **Stable Diffusion v1.5** — `runwayml/stable-diffusion-v1-5`  
- **IP-Adapter (SD15)** — `h94/IP-Adapter` (`ip-adapter_sd15.bin`)  
- **CLIP** — OpenAI CLIP ViT-L/14 (fallback ViT-B/32)  
- **(Optional) OWL-ViT** for zero-shot focal detection

> If a Hugging Face repo is gated (401/403), run `huggingface-cli login` or set `HF_TOKEN`.

---

## ▶️ Quickstart

### A) Colab (recommended)
1. Open **`notebooks/BookCreator_final.ipynb`** (GPU runtime).  
2. Run top-to-bottom.  
3. You’ll get:
   - `reference_full.png`, `reference_focus.png`
   - `scene_best/scene_1..5.png`
   - `story.json`, `storybook.html` (+ optional PDF)

### B) Local CLI (optional)
```bash
python -m code.render_scenes   --topic "robot waiter at the table"   --character "a friendly red service robot"   --outdir outputs/robot_waiter
```

---

## 🧱 How it works (high level)

**Dual-reference IP-Adapter**  
We condition on **two** images:
- `reference_focus.png` (256×256): stabilizes face/details.  
- `reference_full.png` (512×512): carries color/pattern/composition cues.  

**Shot-aware scales (examples):** wide≈0.18 (cap≤0.32), medium/portrait≈0.26 (cap≤0.42).  
Per scene we render a small grid of `(ip_scale, guidance)` variants.

**Generic focal crop (category-agnostic)**  
1) **OWL-ViT** zero-shot proposals from the head noun (e.g., “kitten/head/face/full body”).  
2) If weak → **CLIP sliding windows** over multiple window sizes; pick max cosine vs a compact “close-up face of …” text.  
3) Else soft **center** fallback.  
Works for cats/dogs/robots/toys/etc. (no cat-/human-specific face detector required).

**CLIP-aware prompts**  
Truncate scene text for both SD and CLIP to stay **<77 tokens**; keep key nouns/attributes.

**Selection: text-first + reference tie-break**  
Rank by text→image similarity, keep Top-K, break ties by reference→image similarity.  
→ **Scene fidelity** + **identity consistency** together.

**Reroll when low**  
If best CLIP score < τ (e.g., ~0.27 for ViT-L/14), rerender 2–3 extra candidates with **slightly lower IP-scale** to increase composition diversity.

---


## 🧪 Notes & tips

- **Colab constraints**: prefer fp16 on GPU; don’t move fp16 pipelines to CPU. If you load with `device_map="auto"`, don’t pass `device=` to `pipeline()`.
- **Gated repos**: 401/403 → authenticate (`HF_TOKEN`) or choose public weights.
- **CLIP 77 tokens**: always truncate scene texts for CLIP/SD; keep key nouns/attributes.
- **Duplicate subjects**: strengthen negatives (`duplicate, twins, many, watermark, text`).
- **Large files on GitHub**: for Pages/README previews use compressed **thumbnails** (PNG/WebP, ≤1600px). Heavy PDFs/images → compress or attach via **Releases** (or Git LFS).

---

## 📚 References

- Rombach et al., *High-Resolution Image Synthesis with Latent Diffusion Models* (CVPR’22)  
- Ye et al., *IP-Adapter: Text Compatible Image Prompt Adapter* (arXiv’23)  
- Radford et al., *CLIP: Learning Transferable Visual Models from Natural Language Supervision* (ICML’21)  
- Minderer et al., *OWL-ViT: Simple Open-Vocabulary Object Detection with Vision Transformers* (ECCV’22)  
- Hugging Face: **diffusers**, **transformers**

---


## ✨ Project status

Final notebook + published storybooks.  
Contributions (typo fixes / docs / small robustness patches) are welcome.
