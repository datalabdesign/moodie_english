<img src="logo_moodie_trend.png" alt="Logo MOODIE TREND" width="350"/>

# MOODIE Image Trends · Trend Analysis Module
> **A visual learning pathway for those who do not program yet.**

MOODIE Image Trends is a notebook that transforms **a collection of images** into a set of **dashboards, palettes, and recommendations** ready for trend analysis.

It was designed for courses in **Visual Research**, **Fashion Design**, **UX Research**, **Digital Methods** and **Digital Humanities**, where students benefit from advanced AI techniques integrated into design processes **without writing code**. 

Its core motivation is not to offer a plug and play solution for visual analysis, but rather to introduce, in a critical and methodologically oriented way, the use of computer vision models as analytical resources for working with visualities and large digital image collections. While MOODIE can indeed be used as a ready-to-run tool, it was originally conceived as a plug and play interface for critically examining and comparing the methodological implications of different computer vision architectures when applied to visual research. 

[![Open in Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datalabdesign/moodie/blob/main/modulo_trend/MOODIE_IMAGE_TREND_LIGHT_V02.ipynb)

---

## ✦ Key Features

- **Smart Sampling**  
  - *Downsample, Upsample* or random sampling, even when no CSV is provided.
- **Visual Feature Extraction**  
  - Pretrained models **VGG16, ResNet50, MobileNetV2, InceptionV3**.
- **Duplicate Removal**  
  - Combines **MD5 hash, pHash**, and **embedding** analysis.
- **Perceptual Maps & Palettes**  
  - LAB color grouping → *Colorful, Bright, Soft, Deep, Dark* palettes and **Accessible** palettes (simulation of protan/deutan/tritan color-blindness).
- **Image-walls**  
  - Mosaics sorted by similarity, dominant color, ranking, or representativeness.
- **Recommendation System**  
  - Top-N search using **Cosine, Euclidean or Correlation**, with adjustable weights for embeddings and extra columns (mean, median, Jaccard).  
  - Supports **internal images** or **external uploads** as reference.
- **Automatic Dashboards**  
  - Visual summary (reference + wall + color maps) ready for presentation.
- **ZIP Export**  
  - All panels, CSVs, and recommended images in one click.

---

## ✦ Quick Requirements

This module runs directly in **Google Colab** — no local installation required.  
Extra libraries are installed automatically by the notebook (`tensorflow`, `distinctipy`, `umap-learn`, etc.).

- Python ≥ 3.9 (Colab default)  
- `pandas`, `numpy`, `Pillow`, `ipywidgets`, `scipy`, `scikit-learn`  
- Optional GPU in Colab to speed up feature extraction

## What You Need

* A folder or zip file with the images you want to examine (.jpg/.png).  
* A CSV file containing metadata and additional complementary information about your images (optional)  
* Google account → open the notebook in **Google Colab** and run cell by cell.

<sub>_you may use the data obtained in the Moodie grabber module_</sub>

---

## 1. Overview of the Workflow

| Step | Input | What Happens | Output | Potential Uses (What this enables) |
|-------|-------------|---------------|-----------|---------------------------------------|
| 1. Import | • Folder **`images/`**<br>• (Optional) **CSV** file with metadata | Notebook loads images; if CSV exists, it links each row to the correct file by filename. | `global_df` (DataFrame) with at least the image filename column plus any extra CSV columns | Organize and catalog visual collections; associate textual metadata to images for future analysis. |
| 2. Feature Extraction | `global_df` + chosen model | Each image is encoded into a numerical vector (embedding). | New column `…_features` appended to `global_df` | Transform images into comparable numeric data, identify visual similarity automatically. |
| 3. Duplicate Removal | `global_df` with embeddings | Greedy algorithm identifies: 1) exact files, 2) near-identical images (pHash), 3) embeddings that are too close; moves copies to subfolder and generates CSVs. | • **dataset_no_duplicates.csv**<br>• **dataset_duplicates.csv**<br>• 3 image-walls (full / without duplicates / only duplicates) | Clean visual datasets, optimize analyses by focusing on unique content, identify and analyze subtle variations among nearly identical images. |
| 4. Perceptual Maps & Palettes | `global_df` (no duplicates) | Groups dominant & representative colors ➀, creates perceptual map, extracts thematic palettes and accessible versions (WCAG + color-blindness simulation). | PNGs of maps, thematic and accessible palettes + CSV with color cells | Visualize color distribution, identify predominant and thematic color palettes, generate accessible palettes for inclusive design. |
| 5. Imagewalls | Filtered DataFrame (by ranking, color, representativity ②, etc.) | Builds wall of thumbnails sorted accordingly; optionally generates summary palette. | JPEG of wall + CSVs **wall_info** & **images_info**, plus reports and CSVs with metadata linked to images in the wall | Create general visualizations or summary views of the corpus by specific criteria (visual similarity, colors, metadata like price or engagement), generate visual references for design projects. |
| 6. Recommendation | Embeddings + extra columns + user parameters | Computes similarity (cosine/euclidean/correlation); combines weights; outputs Top-N; generates wall + color dashboards + summary; packages everything in ZIP. | • imagewall_topN.png<br>• dashboards_dom / rep<br>• dashboard_resumo.png<br>• CSVs with descriptive recommendation details | Explore trends and map visual patterns by combining customizable recommendation algorithms with user-defined parameters and advanced CV techniques. |

➀ **Method**: k-means weighted by color frequency → LAB clustering → each cell displayed in 20×20 grid.  
② **Method**: representativity selects the image closest to the centroid average of the group (if grouping is active) or the corpus as a whole.

---

## 2. Detailed Modules

---

### 2.1 Sampling (Sample Dataset)

> **Goal**  
> Create a representative (or balanced) subset of the corpus – via downsizing, upsizing or random sampling – enabling rapid prototyping and control over how many images will be processed.

*Optional step* — allows construction of random subsets.  
If you have a dataset with CSV, the interface will show several sampling options.  
If you only have a folder of images, the interface automatically detects that and allows only the number of images you want in your final sample.

| Input | Method | Recommended When |
|---|---|---|
| CSV | Downsample | Reduce dataset size while maintaining category proportions. Useful when dataset is large and you want a smaller exploratory sample without losing representativity. |
| CSV | Upsample | Duplicate minority classes to increase their sample size. Useful for balancing datasets with highly unequal class distributions. |
| CSV/IMAGES | Random Sample | Select an arbitrary subset of samples without respecting categories. Useful for a quick initial overview or when categories are irrelevant. With only images, this is the only available option. |

---

### 2.2 Visual Feature Extraction

> **Goal**  
> Convert each image into a numerical vector (embedding) using pretrained computer vision models, providing the basis for all visual similarity comparisons.

Moodie uses different computer vision models to “read” your images and convert them into numeric representations.  
These vectors support visual analysis such as clustering, semantic similarity, and construction of thematic dashboards.

Moodie also generates **ImageNet labels** (CNN + ViT architectures) or **free-text captions in English** for encoder-decoder models (ViT + GPT2 / BLIP).

The models are organized by architecture:

---

#### CNN Models (ImageNet-1k)

| Model              | Output Type                     | Recommended Use                                                                 | Reference                                                    |
| -------------------| -------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------|
| **MobileNetV2**     | Features + Top-3 ImageNet labels | Fast analysis of color/texture; ideal for large collections and rapid tests     | Sandler et al., 2018 |
| **VGG16**           | Features + Top-3 ImageNet labels | Strong on texture and edges; good for visual grouping by local patterns         | Simonyan & Zisserman, 2014 |
| **ResNet50**        | Features + Top-3 ImageNet labels | Balanced between global form and texture; good for category differentiation     | He et al., 2015 |
| **InceptionV3**     | Features + Top-3 ImageNet labels | Sensitive to scale variation; useful for images with multi-scale objects        | Szegedy et al., 2015 |
| **EfficientNet-B0** | Features + Top-3 ImageNet labels | Stable for natural scenes; captures fine-grained details and context            | Tan & Le, 2019 |

---

#### Vision Transformers (ViT) – ImageNet-1k

| Model        | Output Type                    | Recommended Use                                                                 | Reference |
|--------------|--------------------------------|----------------------------------------------------------------------------------|-----------|
| **ViT16_1k** | Features + Top-3 ImageNet labels | Captures composition and co-occurrence patterns; useful for complex layouts     | Dosovitskiy et al., 2020 |

---

#### ViT + Captioning (Encoder–Decoder Multimodal)

| Model         | Output Type                       | Recommended Use                                                                | Reference |
|---------------|------------------------------------|---------------------------------------------------------------------------------|-----------|
| **ViT-GPT2**  | Features + Automatic caption (EN)  | Generates short captions; useful for quick triage and contextual description   | ViT + GPT-2 |
| **BLIP (base)** | Features + Automatic caption (EN) | Generates more semantic descriptions; useful for thematic exploration          | Li et al., 2022 |

---

> Important: CNN/ViT ImageNet models return labels limited to ImageNet-1k (~1000 classes).  
> Captioning models produce free-text captions, ideal for semantic analyses.

**How the process works**

1. You select one or more models.  
2. Each image is processed according to the chosen model(s).  
3. Moodie generates columns `*_features`, `*_labels`, `*_scores`.  
4. CSVs are saved and used in subsequent modules.

---

### 2.3 Visual Duplicate Removal

> **Goal**  
> Detect and separate exact or near-identical copies (hash, pHash, embedding) to maintain only unique images and avoid bias in the following analyses.

After extracting visual features, Moodie can identify and remove duplicates.

![](exemplo_duplicatas.png)

**How Moodie identifies duplicates**

1. **Exact Copies (MD5):** Byte-identical files.  
2. **Near-identical (pHash):** Visually similar even with small edits or compression.  
3. **Deep Visual Similarity (Embedding):** Measures proximity of embeddings.

**During removal**

* Anchor image chosen (first one).  
* Duplicates moved to folder `duplicatas`.  
* Progress bars displayed.  
* ZIP generated with:  
  - clean dataset  
  - duplicate dataset  
  - 3 imagewalls  

**Why remove duplicates**

* More accurate analyses  
* Saves space  
* Better organization  

---

### 2.4 Color Analysis and Palette Generation

> **Goal**  
> Map color distribution, group colors into perceptual zones, and generate thematic/accessible palettes summarizing the corpus's visual identity.

Moodie analyzes the chromatic content of your image dataset.

**Perceptual Color Map**

Every point represents a color in the dataset.  
Proximity = perceptual similarity.

![](color_map.png)

**How Moodie finds “color zones”**

1. Division by hue (24 bands, 15° each).  
2. Spatial clustering on perceptual map.  
3. Statistical averaging for each zone.

**Thematic Palettes**

| Theme | Logic | Metrics |
|---|---|---|
| Colorful | High saturation, mid luminance | Saturation, luminance, color distance |
| Bright | Light and saturated | High luminance, high saturation |
| Soft | Light and low saturation | High luminance, low saturation |
| Deep | Dark and saturated | Low luminance, high saturation |
| Dark | Predominantly dark colors | Luminance, saturation, frequency |
| Accessible | Distinguishable under color-blindness simulation | Delta-E under protan/deutan/tritan simulation |

**Renaming Images**

| Renaming Criterion | Description | Useful for |
|---|---|---|
| Do not rename | Keeps original filenames | Existing workflows |
| Rename by HEX | Use dominant color HEX | Organizing images by color |
| Rename by pHash | Use perceptual hash | Grouping visually similar images |

**EXIF Metadata**

If present, Moodie extracts:

* timestamp  
* camera model  
* ISO, aperture, shutter  
* lens model  
* GPS coordinates  

---

### 2.5 Corpus Visualization

> **Goal**  
> Build ordered imagewalls (global or grouped) to enable rapid inspection and reveal patterns, duplicates, and aesthetic coherence.

![](exemplo_image_wall.jpg)

**Capabilities**

* Global visualization  
* Group-based visualization  
* Filtering (representativity, ranking, dominant color)  
* Palette display  
* Optional duplicate removal  

**Workflow**

1. Choose image column  
2. (Optional) choose grouping column  
3. Select filtering method:  
   * None  
   * Representativity  
   * Ranking  
   * Dominant color  
4. Thumbnail size  
5. (Optional) square crop  
6. (Optional) palette display  
7. (Optional) remove duplicates  

Outputs saved in `imagewall/`.

---

### 2.6 · Moodie Trends — Recommendation System

> **Goal**  
> Find similar images, curate sets, and generate color dashboards using combinations of **visual similarity (embeddings)** and **optional metadata**.

![](dashboard_exemplo.png)

---

#### 1. Initial Configuration

| Step | Description | What You Do |
|---|---|---|
| Image Column | Filename in the DataFrame | Select correct column |
| Embedding Column | Vector of visual features | Select correct column |
| Similarity Metric | Cosine, Euclidean, Correlation | Choose metric |
| Embedding Weight | Weight relative to metadata | Adjust slider (0–10) |

---

#### 2. Similarity Metrics

| Metric | How it works | Use when… |
|---|---|---|
| Cosine | Compares angle between vectors | Structure/composition similarity |
| Euclidean | Geometric distance | Color/texture proximity |
| Correlation | Linear co-variation | Similar lighting/color patterns |

---

#### 3. Extra Columns (optional)

Each column gets:

1. Aggregator  
2. Separator  
3. Weight  

##### 3.1 Aggregators

| Column | Aggregator | How it works | Use when |
|---|---|---|---|
| Numeric | none | Uses first value | Single value |
|  | mean | Mean of split values | Multiple numbers |
|  | median | Median | Outlier-resistant |
| Text/Set | none | Literal string | Single label |
|  | jaccard | Set similarity | Multi-term fields |

##### 3.2 Separator

Examples:

| Cell | Separator | Effect |
|---|---|---|
| 10,20,30 | , | Three numbers |
| blue;red;green | ; | Three terms |
| White | none | Single value |

##### 3.3 Weight

0–10 scale.

---

#### 4. Similarity Search

| Step | What it defines | Meaning |
|---|---|---|
| Reference Mode | Source of reference | Internal or upload |
| Reference Image | Query | Filename or upload |
| Top N | Output size | Slider |
| Sort | Ranking | Relevance / Contrast / Concordance |

**External image workflow**

1. Image is embedded  
2. Compared to internal  
3. Closest internal becomes anchor  
4. Entire pipeline proceeds normally  

---

#### 5. Sorting Criteria

| Criterion | Logic | Meaning |
|---|---|---|
| Relevance | High similarity ranks first | Most similar |
| Contrast | 1 − similarity | Most different |
| Concordance | 1 − 2·|sim − 0.5| | Mid-range similarity |

---

#### 6. Outputs

- Full and summary **CSV**  
- **Imagewall** of Top-N  
- **Color dashboards** (dominant & representative)  
- **Accessible palettes**  
- **ZIP** with everything  

These resources allow quick assessment of **similarity, diversity and chromatic composition**.

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
