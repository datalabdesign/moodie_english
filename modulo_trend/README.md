<img src="logo_moodie_trend.png" alt="Logo MOODIE TREND" width="350"/>

# MOODIE Image Trends · Trend Analysis Module
> **A visual learning pathway for those who do not program yet.**

MOODIE Image Trends is a notebook that transforms **a collection of images** into a set of **dashboards, palettes, and recommendations** ready for trend analysis.

It was designed for courses in **Visual Research**, **Fashion Design**, **UX Research**, **Digital Methods** and **Digital Humanities**, where students benefit from advanced AI techniques integrated into design processes **without writing code**. 

Its core motivation is not to offer a plug and play solution for visual analysis, but rather to introduce, in a critical and methodologically oriented way, the use of computer vision models as analytical resources for working with visualities and large digital image collections. While MOODIE can indeed be used as a ready-to-run tool, it was originally conceived as a plug and play interface for critically examining and comparing the methodological implications of different computer vision architectures when applied to visual research. 

[![Open in Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datalabdesign/moodie/blob/main/modulo_trend/MOODIE_IMAGE_TREND_LIGHT_V02.ipynb)

---

## ✦ Key Features

- **Intelligent sampling**  
  - Downsample, Upsample, or Random sampling, even when no CSV is present.
- **Visual feature extraction**  
  - Pretrained models: **VGG16, ResNet50, MobileNetV2, InceptionV3**.
- **Duplicate removal**  
  - Combines **MD5 hash, pHash**, and **embedding-based** analysis.
- **Perceptual color maps & palettes**  
  - LAB clustering → thematic palettes: *Colorful, Bright, Soft, Deep, Dark* + **Accessible** (protan/deutan/tritan color-blindness simulation).
- **Image-walls**  
  - Mosaics ordered by similarity, dominant color, ranking, or representativeness.
- **Recommendation system**  
  - Top-N search using **Cosine, Euclidean, or Correlation**, with adjustable weights for embeddings and extra columns (mean, median, Jaccard).  
  - Accepts **internal images** or **external uploads** as reference.
- **Automatic dashboards**  
  - Visual summary (reference + wall + color maps) ready for presentations.
- **ZIP export**  
  - All dashboards, CSVs, and recommended images in one click.

---

## ✦ Quick Requirements

This module runs directly in **Google Colab** — no local installation required.  
Extra libraries are automatically installed ( `tensorflow`, `distinctipy`, `umap-learn`, etc. ).

- Python ≥ 3.9 (Colab default)  
- `pandas`, `numpy`, `Pillow`, `ipywidgets`, `scipy`, `scikit-learn`  
- Optional GPU in Colab to speed up feature extraction

## What you need

* A folder or ZIP file containing the images you want to examine (.jpg/.png).  
* A CSV file containing metadata or additional information about your images (optional).  
* A Google account → open the notebook in **Google Colab** and run cell by cell.

<sub>You can use the data produced in the MOODIE Grabber module.</sub>

---

## 1. Overview of the Workflow

| Step | Input | Process | Output | Potential Uses |
|------|--------|----------|---------|----------------|
| 1. Import | • **`imagens/`** folder<br>• (Optional) **CSV** with metadata | Notebook loads images; if CSV is provided, it matches rows to image filenames. | `global_df` (DataFrame) with image names and optional CSV columns | Organize and catalog visual collections; associate text/metadata with images. |
| 2. Feature Extraction | `global_df` + model selection | Each image is encoded into a numeric vector (embedding). | New column `…_features` added to `global_df` | Transform images into comparable numeric data; automatically identify visual similarity. |
| 3. Duplicate Removal | `global_df` with embeddings | Greedy algorithm detects exact duplicates, near-duplicates (pHash), and embedding similarity; moves copies to a subfolder and saves CSVs. | • `dataset_no_duplicates.csv`<br>• `dataset_duplicates.csv`<br>• 3 image-walls (full / no dups / only dups) | Clean visual datasets, reduce redundancy, focus on unique content, detect subtle variations. |
| 4. Perceptual Color Maps & Palettes | `global_df` (no duplicates) | Clusters dominant & representative colors ➀ → perceptual map → thematic and accessible palettes (WCAG + color-blindness simulation). | PNGs of maps and palettes + CSV with color cells | Visualize color distribution, identify dominant palettes, generate accessible palettes. |
| 5. Imagewalls | Filtered DataFrame (by ranking, color, representativeness ②, etc.) | Builds ordered mosaic walls; optionally adds a summary palette. | JPEG wall + **wall_info** & **images_info** CSVs | Produce visual summaries, synthesize a corpus, generate design moodboards. |
| 6. Recommendation | Embeddings + extra columns + user parameters | Computes similarity; combines weights; returns Top-N, generates wall + color dashboards + synthesis dashboard; exports ZIP. | • `imagewall_topN.png`<br>• color dashboards<br>• `dashboard_resumo.png`<br>• CSVs with metadata | Map visual patterns, explore trends, build custom recommendations, create exploratory visual boards. |

➀ **Method:** weighted k-means on color frequency → LAB clustering → each cell displayed in a 20×20 grid.

② **Method:** representativeness selects images closest to the centroid of the group or of the corpus.

---

## 2. Detailed Modules

### 2.1 Sampling (Sample Dataset)
> **Goal**  
> Create a representative (or balanced) subset of the corpus via downsampling, upsampling, or random sampling — enabling fast prototyping and control of dataset size.

*Optional use* — allows selecting sample subsets through randomness.  
If your dataset includes a CSV, the interface offers multiple sampling modes.  
If you only provide images, the interface automatically allows specifying the number of images desired in the final subset.

| Input | Method | When to use |
|-------|---------|--------------|
| CSV | Downsample | Reduce dataset size while preserving category proportions; ideal for large datasets. |
| CSV | Upsample | Expand minority categories by duplicating samples; ideal for balancing datasets. |
| CSV/IMAGE | Random Sample | Select a random subset regardless of categories; ideal for quick inspection or when categories do not matter. |

---

### 2.2 Visual Feature Extraction
> **Goal**  
> Convert each image into a numeric vector (*embedding*) using pretrained computer-vision models.

Moodie uses different AI models to "read" images and convert them into numerical representations (*features*). These vectors enable visual analysis such as clustering, semantic comparison, and thematic dashboards.

Models are organized by architecture, and output may include **ImageNet-1k labels** (for CNN and ViT) or **free-text captions** (for encoder–decoder multimodal models like ViT-GPT2 and BLIP).

#### CNN Models (ImageNet-1k)

| Model | Output Type | Recommended For | Reference |
|-------|--------------|------------------|-----------|
| MobileNetV2 | Features + Top-3 ImageNet labels | Fast analysis; large datasets; texture/color | <https://arxiv.org/abs/1801.04381> |
| VGG16 | Features + Top-3 ImageNet labels | Texture and edge-focused grouping | <https://arxiv.org/abs/1409.1556> |
| ResNet50 | Features + Top-3 ImageNet labels | Balanced shape/texture analysis | <https://arxiv.org/abs/1512.03385> |
| InceptionV3 | Features + Top-3 ImageNet labels | Multi-scale object variation | <https://arxiv.org/abs/1512.00567> |
| EfficientNet-B0 | Features + Top-3 ImageNet labels | Fine details in natural scenes | <https://arxiv.org/abs/1905.11946> |

#### Vision Transformers (ViT) – ImageNet-1k

| Model | Output Type | Recommended For | Reference |
|-------|--------------|------------------|-----------|
| ViT16_1k | Features + Top-3 ImageNet labels | Composition and layout structures | <https://arxiv.org/abs/2010.11929> |

#### ViT + Captioning (Multimodal Encoder–Decoder)

| Model | Output Type | Recommended For | Reference |
|-------|--------------|------------------|-----------|
| ViT-GPT2 | Features + Automatic caption (EN) | Action/context description; thematic exploration | ViT + GPT-2 |
| BLIP (base) | Features + Automatic caption (EN) | More semantic descriptions; thematic dashboards | <https://arxiv.org/abs/2201.12086> |

---

### 2.3 Duplicate Removal
> **Goal**  
> Detect and separate exact and near-duplicate images (hash, pHash, embedding) to keep only unique items.

The process uses three layers:

1. **MD5 hash** — exact file copies  
2. **pHash** — visually similar images with small edits  
3. **Embedding similarity** — deep visual resemblance

Duplicates are moved into `imagens/duplicatas/`, leaving a clean dataset and producing:

- Clean dataset  
- Duplicates dataset  
- Three image-walls (full / unique / duplicates)

---

### 2.4 Color Analysis and Palette Generation
> **Goal**  
> Map the chromatic distribution of the corpus, cluster colors into perceptual “zones,” and generate thematic and accessible palettes.

This step produces:

- Perceptual color map  
- Thematic palettes: Colorful, Bright, Soft, Deep, Dark  
- Accessible palette (protan/deutan/tritan-safe)  
- CSVs with color cells

Moodie uses:

- LAB clustering  
- Hue segmentation  
- k-means weighted by frequency  
- WCAG contrast checks for accessible palettes

Renaming options:

| Rename Option | Description | Useful For |
|---------------|-------------|------------|
| No rename | Keep original filenames | When naming structure already exists |
| HEX | Use dominant color hex code | Organizing by color |
| pHash | Use perceptual hash | Grouping visually similar images |

EXIF metadata (if present) is extracted: timestamp, camera model, lens info, GPS, etc.

---

### 2.5 Corpus Visualization
> **Goal**  
> Build ordered image-walls that allow fast visual inspection, revealing patterns, duplicates, and aesthetic coherence.

Imagewalls can be:

- Global  
- Grouped by column  
- Filtered by representativeness, ranking, or dominant color  
- Square-cropped (optional)  
- With or without color palettes

Outputs are saved to `imagewall/`.

---

### 2.6 MOODIE Trends — Recommendation System
> **Goal**  
> Find similar images, create curatorial sets, and generate color dashboards using visual similarity (embeddings) + optional metadata.

Users configure:

- Image column  
- Embedding column  
- Similarity metric (Cosine, Euclidean, Correlation)  
- Embedding weight  
- Extra metadata columns (with aggregators: none, mean, median, Jaccard)

Top-N results produce:

- `imagewall_topN.png`  
- Color dashboards  
- Summary dashboard  
- CSVs  
- ZIP export

External queries are supported: upload an image → embed → compare with corpus → return Top-N.

---

## About the Author

**Elias Bitencourt** is an Assistant Professor at the Design Program of the State University of Bahia (UNEB). He holds a PhD in Communication from FACOM/UFBA and an MA in Culture and Society from IHAC/UFBA. He was a visiting researcher at the Milieux Institute (Concordia University, Canada) in 2019. He currently leads **Datalab/Design (CNPq)**, a research center focused on data visualization and digital methods. His research investigates data visualization, platform studies, digital imaginaries, and algorithmic mediation in social relations. He is a collaborating researcher at the **Inova Media Lab** (Universidade Nova de Lisboa) and at the international **Public Data Lab**.  
[More at](https://eliasbitencourt.com)

---

## Status

This project is currently in **beta** and under continuous development. Contributions, suggestions, and collaborations are welcome.

---

## License

This repository is under a **Restricted Use License with Attribution**.  
Content may be used for academic and non-commercial purposes with proper attribution to the author.  
Modifications or redistribution require permission.  
See the file [`LICENSE.txt`](./LICENSE.txt) for more details.

## How to cite MOODIE (APA format):

Bitencourt, E. (2025). *MOODIE Image Trends: Modular Observational & Operational Design Image Explorer* (Beta version) [GitHub repository]. Datalab/Design – Universidade do Estado da Bahia. https://github.com/datalabdesign/moodie
