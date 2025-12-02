<img src="https://github.com/datalabdesign/moodie/blob/main/logo_moodie_all.png" alt="Logo MOODIE ALL" width="600"/>

# MOODIE · Modular Observational & Operational Design Image Explorer (Light)
> **Open-source (beta) tool for collecting, exploring, and visually analyzing large image archives — without needing to code.**

MOODIE was created as a **pedagogical laboratory** that introduces Design students and professionals to computational vision methods, visual data mining, and trend mapping practices.  
The complete workflow is organized into **two independent yet complementary notebooks**:

| Sequence | Notebook (Google Colab) | What it does | Open now |
|----------|--------------------------|--------------|-----------|
| **1.** | **MOODIE Grabber**<br><sub>Collection & preprocessing</sub> | Performs batch _download_ from a CSV of links, renames files, removes duplicates, and generates the first structured dataset. | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datalabdesign/moodie/blob/main/modulo_grabber/01_MOODIE_GRABBER_v2_BETA.ipynb) |
| **2.** | **MOODIE Image Trends**<br><sub>Analysis, color, recommendations</sub> | Takes an **image folder** (from the Grabber or your own), extracts features, removes duplicates, generates color maps, image-walls, and a Top N visual recommendation system. | [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datalabdesign/moodie/blob/main/modulo_trend/MOODIE_IMAGE_TREND_LIGHT_V02.ipynb) |

> **Workflow Overview** • *(a) Collect ⇢ (b) Explore ⇢ (c) Recommend)*  
> 1️⃣ **Grabber** downloads/organizes ► 2️⃣ **Trends** analyzes, generates palettes, dashboards, and recommendations.

---

## ✦ Key Features

### ① MOODIE Grabber
- Reads CSVs with image links (configurable column).  
- **Sampling** (down-/up-/random) before download.  
- Optional naming (`original`, `hex`, `pHash`).  
- **Duplicate removal** using pHash or CSV keys.  
- Metadata extraction (EXIF, dimensions, domain, file size).  
- Outputs: `imagens/` folder, structured CSV dataset, reports, and a ready-to-use structure for the Trends module.

### ② MOODIE Image Trends
- Extra **sampling** (useful for rapid prototyping).  
- **Feature extraction** with VGG16 · ResNet50 · MobileNetV2 · InceptionV3 (TensorFlow).  
- **Three-layer deduplication** (MD5, pHash, embedding).  
- **Perceptual color maps** + thematic and accessible palettes (color-blindness simulation).  
- **Image-walls** sorted by similarity, color, ranking, or representativeness.  
- **Recommendation system**:  
  - Cosine • Euclidean • Correlation metrics.  
  - Adjustable weights for embeddings **and** extra columns (mean, median, Jaccard).  
  - Works with internal images or **external uploads** as reference.  
- Automatic dashboards (reference + wall + palettes) ready for presentations.  
- **ZIP export** with CSVs, dashboards, and generated images (one click).

---

## ✦ Quick Requirements

Both notebooks run **100% in Google Colab**.  
Dependencies (`tensorflow`, `scikit-learn`, `distinctipy`, `umap-learn`, etc.) are automatically installed on first execution.

- Python ≥ 3.9 (Colab version).  
- Optional GPU in Colab to speed up embedding extraction.

---

## ✦ Step-by-Step Workflow

1. **Prepare your CSV** (links) **or** a folder/ZIP of images.  
2. **Open the Grabber notebook** → configure parameters → run cells in sequence.  
   - Output: `imagens/` folder + `datasets/` + `reports/`.  
3. **Assemble (or revise) the image folder** to be analyzed.  
4. **Open the Trends notebook** → load the `imagens/` folder (and CSV if available).  
5. Follow the Trends sections:  
   1. _Import → Features → Duplicates → Colors → Image-walls → Recommendations_.  
6. Download the final ZIP with dashboards, palettes, and CSVs for use in reports, presentations, or moodboards.

*(You may skip the Grabber if you already have local images.)*

---

## About the Author

[Elias Bitencourt](https://eliasbitencourt.com) is an Assistant Professor in the Design Program at the State University of Bahia (UNEB). He holds a PhD in Communication from FACOM/UFBA and an MA in Culture and Society from IHAC/UFBA. He was a visiting researcher at the Milieux Institute (Concordia University, Canada) in 2019. He currently leads **Datalab/Design (CNPq)** at UNEB, a research center focused on data visualization and digital methods. His work investigates data visualization, platform studies, digital imaginaries, and algorithmic mediation in social relations. He is a collaborating researcher at the **Inova Media Lab** (Universidade Nova de Lisboa) and at the international **Public Data Lab** network.

---

## Status

The project is in **beta phase** and under continuous development. Contributions, suggestions, and collaborations are welcome.

---

## License

This repository is under a **Restricted Use License with Attribution** (academic and non-commercial).  
Content may be used for academic and non-commercial purposes with proper attribution to the author.  
Modifications or redistribution require permission. See the [`LICENSE.txt`](LICENSE.txt) file for details.

## How to cite MOODIE (APA format):

Bitencourt, E. (2025). *MOODIE Image Trends: Modular Observational & Operational Design Image Explorer Light* (Beta version) [GitHub repository]. Datalab/Design – Universidade do Estado da Bahia. https://github.com/datalabdesign/moodie

<p align="left">
  <sub>Website: <a href="https://datalab.org">datalab.org</a> · Instagram: <a href="https://www.instagram.com/datalabdesign">@datalabdesign</a></sub>
</p>
