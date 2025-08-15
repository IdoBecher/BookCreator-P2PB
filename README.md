# BookCreator-P2PB — From Prompt to Picture-Book  
*A Colab-friendly pipeline that turns a short prompt (**topic, character**) into a 5-scene picture-book using SD 1.5 + IP-Adapter with CLIP-based selection.*

> **Goal.** Keep the **same main character** recognizable across scenes, while **layouts stay diverse** and each image matches its **scene caption**.

---

## 🌐 Live demo (GitHub Pages)

- Project hub: **https://idobecher.github.io/BookCreator-P2PB/**
- Storybooks (examples):  
  - `/docs/storybook_cat.html`  
  - `/docs/storybook_dog.html`  
  - `/docs/storybook_robot.html`

> Your Pages site serves everything under `docs/`. Put `index.html` there as the landing page.

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
BookCreator-P2PB/
├─ notebooks/
│  └─ BookCreator_final.ipynb      # end-to-end Colab/Local notebook
├─ code/                           # (optional) split helpers
│  ├─ generate_scenes.py           # LLM prompting, parsing, de-dup
│  ├─ make_reference.py            # auto reference + focal crop (OWL-ViT → CLIP → center)
│  ├─ render_scenes.py             # SD1.5 + IP-Adapter, variant grid, CLIP ranking, export
│  └─ utils_clip.py                # CLIP backends, text truncation (<77 tokens)
├─ docs/                           # GitHub Pages site
│  ├─ index.html                   # gallery/links
│  ├─ stories/                     # exported storybooks (HTML/PDF/images)
│  └─ assets/                      # thumbnails used by index.html
├─ outputs/                        # local artifacts (reference_*.png, scene_best/, story.json, …)
├─ LICENSE
└─ README.md
