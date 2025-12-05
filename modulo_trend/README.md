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
| **MobileNetV2**     | Features + Top-3 ImageNet labels | Fast analysis of color/texture; ideal for large collections and rapid tests     | [Sandler et al., 2018](https://arxiv.org/abs/1801.04381)  |
| **VGG16**           | Features + Top-3 ImageNet labels | Strong on texture and edges; good for visual grouping by local patterns         | [Simonyan & Zisserman, 2014](https://arxiv.org/abs/1409.1556) |
| **ResNet50**        | Features + Top-3 ImageNet labels | Balanced between global form and texture; good for category differentiation     | [He et al., 2015](https://arxiv.org/abs/1512.03385) |
| **InceptionV3**     | Features + Top-3 ImageNet labels | Sensitive to scale variation; useful for images with multi-scale objects        | [Szegedy et al., 2015](https://arxiv.org/abs/1512.00567)  |
| **EfficientNet-B0** | Features + Top-3 ImageNet labels | Stable for natural scenes; captures fine-grained details and context            | [Tan & Le, 2019](https://arxiv.org/abs/1905.11946) |

---

#### Vision Transformers (ViT) – ImageNet-1k

| Model        | Output Type                    | Recommended Use                                                                 | Reference |
|--------------|--------------------------------|----------------------------------------------------------------------------------|-----------|
| **ViT16_1k** | Features + Top-3 ImageNet labels | Captures composition and co-occurrence patterns; useful for complex layouts     | [Dosovitskiy et al., 2020](https://arxiv.org/abs/2010.11929)  |

---

#### ViT + Captioning (Encoder–Decoder Multimodal)

| Model         | Output Type                       | Recommended Use                                                                | Reference |
|---------------|------------------------------------|---------------------------------------------------------------------------------|-----------|
| **ViT-GPT2**  | Features + Automatic caption (EN)  | Generates short captions; useful for quick triage and contextual description   | [ViT](https://arxiv.org/abs/2010.11929) + [GPT-2](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) |
| **BLIP (base)** | Features + Automatic caption (EN) | Generates more semantic descriptions; useful for thematic exploration          | [Li et al., 2022](https://arxiv.org/abs/2201.12086) |

---

> Important: CNN/ViT ImageNet models return labels limited to ImageNet-1k (~1000 classes).  
> Captioning models produce free-text captions, ideal for semantic analyses.

**How the process works:**

1. **Model selection:** You may choose one or more models, mixing architectures if desired.  
2. **Feature extraction:** Each image is processed according to the selected model.  
3. **Vector and label generation:** Up to three columns are created per model: `*_features`, `*_labels`, `*_scores`.  
4. **Storage and visualization:** Results are saved as CSV files and used by the subsequent Moodie modules.

> Tip: for more precise descriptions of action or context, use models such as BLIP or ViT-GPT2.  
> For faster performance with large archives, start with MobileNetV2 or EfficientNet-B0.

---

### 2.3 Visual Duplicate Removal
> **Goal**  
> Detect and separate exact or near-identical copies (hash, pHash, and embeddings) to keep only unique images and avoid bias in subsequent analyses.

After extracting the visual features of your images, Moodie can automatically identify and remove images that are duplicated or highly similar within your collection. This is useful for cleaning your dataset, avoiding redundancy in later analyses, and ensuring that detected visual trends are representative of unique images.

![](exemplo_duplicatas.png)

**How does Moodie identify duplicates?**

Moodie uses three methods to detect duplicates, checking different levels of similarity:

1. **Exact Copies (MD5 Hash):** Looks for files that are identical byte-by-byte. This is equivalent to checking whether two files are the exact same copy.  
2. **Near-Identical Copies (pHash):** Identifies images that are visually very similar even when small differences exist (such as compression artifacts or minor edits). Think of it as finding photos that are almost identical but with slight variations.  
3. **Deep Visual Similarity (Embedding):** Uses the visual “codes” (embeddings) extracted in the previous stage to find images that are deeply similar at a semantic level, even when they are not exact or near-exact copies. If you used more than one model, it is recommended to remove duplicates using the same model you intend to use for subsequent analyses.

**What happens during the removal process?**

* **Anchor Selection:** For each group of duplicate images, Moodie selects the first image encountered as the “anchor” (the one that will be kept).  
* **Moving Duplicates:** Identified copies are automatically moved to a folder named `duplicatas` inside your `imagens/` directory. This keeps your main dataset clean while still allowing you to review the duplicates if needed.  
* **Process Monitoring:** While Moodie works, progress bars display the status of the analysis.  
* **Final Output:** Once the process is complete, a **Download ZIP** button appears. Clicking it provides a compressed `.zip` file containing:  
    * A “clean” dataset (without the removed duplicates).  
    * A log of all duplicates that were found.  
    * Image walls showing:  
        * All images from the initial dataset.  
        * Only the unique images (after duplicate removal).  
        * The collection of images identified as duplicates.

**Why remove duplicates?**

* **More accurate analyses:** Ensures that visual trends and subsequent analytical results are not distorted by multiple copies of the same image.  
* **Space efficiency:** Helps reduce the size of your image collection, making it easier to manage and process.  
* **Better organization:** Keeps your primary dataset more organized and focused on unique items.

By using Moodie’s duplicate-removal system, you ensure a cleaner and more reliable visual dataset for your explorations and trend analyses!

> **Note:** If you are studying memes, virality, or repetition-based patterns, removing duplicates may *not* be recommended. Moodie provides the tools, but **you** are the one who should decide what makes sense for your research, ok?
---

### 2.4 Color Analysis and Palette Generation
> **Goal**  
> Map the chromatic distribution of the corpus, group colors into perceptual “zones,” and generate thematic and accessible palettes that summarize the visual identity of the dataset.

This stage of Moodie dives into the colors of your image collection to create a visual map of the dominant chromatic distribution and suggest thematic palettes relevant to your corpus.

**Perceptual Color Map:**

Imagine a map where each point represents a color found in your image collection. Visually similar colors appear close to one another, forming “regions” of predominant hues. Moodie generates this perceptual map by combining the colors of all images and applying an algorithm that spatially clusters similar colors. The intensity of color in certain areas indicates how concentrated those tones are within the corpus.

**Visual Example:**

![](color_map.png)

*This map shows how colors are distributed across your image collection. Areas with vibrant tones indicate a higher occurrence of those hues.*

**How does Moodie find the “color zones”?**

Moodie uses concepts from **color theory** to identify meaningful regions within the perceptual map. It relies on the idea of the **color wheel**, where hues are arranged in a cycle based on their relationships.

1. **Hue Division:** Moodie divides the hue spectrum into 24 hue bands (each spanning 15 degrees on the color wheel), assigning a name to each band (e.g., Red, Orange, Yellow, Green, Blue, Violet).  
2. **Proximity Grouping:** In the perceptual map, colors with similar hues and close spatial positions tend to cluster. The algorithm identifies these clusters as predominant “color zones” in your corpus.  
3. **Statistical Analysis:** For each zone, Moodie calculates the average tone to determine the representative color of that region.

**Thematic Palettes:**

Beyond the perceptual map, Moodie extracts thematic color palettes, suggesting sets of 5 colors that may be relevant or representative of your corpus from different analytical perspectives.

**Logic and Metrics Behind Thematic Palettes:**

The selection of colors for each thematic palette relies on a combination of the color distribution within the perceptual map and metrics that evaluate the characteristics of colors inside each “zone.” The general logic is to identify distinct and representative colors from different areas of the map, prioritizing those that best fit the intended thematic focus.

| Palette Theme | Core Logic | Metrics Used |
|---|---|---|
| **Colorful** | Prioritizes highly saturated colors with mid-range lightness, aiming for variety and vibrancy. | High saturation, lightness around 0.5, color distance between samples. |
| **Bright** | Selects light and saturated colors, evoking energy and emphasis. | High lightness, high saturation. |
| **Soft** | Chooses light and low-saturation colors (pastel tones), conveying delicacy and calmness. | High lightness, low saturation. |
| **Deep** | Opts for dark and saturated colors, suggesting intensity and depth. | Low lightness, high saturation. |
| **Dark** | Focuses on predominantly dark colors, with some saturation to avoid purely achromatic tones. | Low lightness, moderate to high saturation, weight of dark-color frequency. |
| **Accessible** | Selects colors distinguishable for individuals with different types of color blindness (protanopia, deuteranopia, tritanopia). | Color distance (Delta E) simulated for various types of color blindness (minimum safe contrast). |

For the **Accessible** palette, Moodie attempts to select one representative color from each of the five identified color zones, ensuring that the chosen colors provide sufficient contrast to be distinguishable for users with color vision deficiencies. If it is not possible to retrieve five distinct and safe colors, the palette may contain fewer colors, and an alert is issued to indicate this.


**Renaming Images:**

Moodie offers the option to rename your image files based on different criteria extracted during the analysis:

| Renaming Criterion | Description | Useful for… |
|---|---|---|
| **Do not rename** | Keeps the original filenames. | When you already have a naming convention or do not want to modify file names. |
| **Rename by HEX** | Renames files using the hexadecimal code of the dominant color detected in each image. | Organizing images visually according to their primary color. |
| **Rename by pHash** | Renames files using the image’s pHash (perceptual hash). The pHash generates a visual “fingerprint,” useful for identifying near-duplicate images. | Grouping visually similar images (even when small edits or variations exist). |

**EXIF Metadata:**

EXIF (Exchangeable Image File Format) refers to metadata embedded in image files by most digital cameras and smartphones. When present, Moodie can extract a range of useful information, including:

* **Capture Date and Time:** When the photo was taken.  
* **Camera/Phone Model:** The device used to capture the image.  
* **Camera Settings:** ISO, aperture, shutter speed, focal length.  
* **Lens Information:** Lens model used in the capture.  
* **GPS Data:** Geographic coordinates (latitude and longitude) of where the photo was taken.

These metadata elements can be important for contextual analysis and for understanding the provenance of your images. Moodie checks for their presence and makes them available for further exploration in your dataset.

---

### 2.5 Corpus Visualization
> **Goal**  
> Build ordered image-walls (general or grouped) that support rapid visual inspection, revealing patterns, duplications, and the aesthetic coherence of the dataset.

This stage of Moodie enables the creation of visual representations of your image collection, supporting exploration and pattern identification. You can generate image walls that arrange your images in a grid, with options to filter, group, and display relevant information.

![](exemplo_image_wall.jpg)

**What can you do?**

* **General Visualization:** Create an overview of the entire corpus in a single image grid.  
* **Visual Grouping:** Organize images into separate walls based on categories or specific characteristics (for example, dominant color or visual similarity if you have executed the clustering stage).  
* **Filtering:** Display only a subset of images based on criteria such as representativeness (the most typical images of a group), ranking (if you have classification scores associated with the images), or dominant color.  
* **Color Palette Display:** Automatically include the dominant color palette of each group directly within the visual wall.  
* **Duplicate Removal (Optional):** Before generating visualizations, you may remove duplicate or highly similar images to obtain a cleaner representation of your unique corpus.

**How does it work?**

1. **Image Column Selection:** First, you indicate which column in your dataset contains the filenames of the images.  
2. **Grouping Options (Optional):** If desired, you may choose a column to group your images. Moodie will create a separate image wall for each unique group in that column. For example, if you have a column named “Style,” you can generate one wall for each distinct style.  
3. **Filtering Options (Optional):**  
    * **None:** Displays all images (after duplicate removal, if selected).  
    * **Representativeness:** For each group (or for the entire corpus, if no grouping is used), Moodie identifies the most visually representative images based on previously extracted features. You may specify how many “Top N” images to display per group.  
    * **Ranking:** If you have a ranking column (for example, popularity scores), you can use this option to display images with the highest or lowest rankings (Top N). You must select the ranking column and the order (ascending or descending).  
    * **Dominant Color:** Allows you to display images with the lightest or darkest dominant colors (Top N), using the dominant-color column extracted earlier.  
4. **Thumbnail Size:** You can define the desired width and height of the thumbnails displayed on the image wall. Moodie automatically adjusts the layout to accommodate the number of images.  
5. **Cropping (Optional):** The “Crop (Square)” option instructs Moodie to crop images into a square format before resizing them into thumbnails, ensuring a visually uniform grid.  
6. **Color Palettes (Optional):** You may choose to include a strip showing the dominant color and a small palette of representative colors beneath each image wall (for the overall visualization and/or for each group).  
7. **Duplicate Removal (Optional):** When selected, Moodie checks for and removes images that are exact copies, near-identical visuals (highly similar images), or very close in terms of extracted visual features (embeddings). **If you have already executed the duplicate-removal step earlier, you do not need to enable this option again.**

**How does Moodie handle duplicate removal?**

If you choose to remove duplicates, Moodie applies a sequence of checks:

1. **Exact Duplicates (MD5 Hash):** It compares a unique “code” generated from the content of each file. If two codes match, the files are considered exact copies.  
2. **Near-Identical Duplicates (pHash):** It uses a perceptual hash (pHash) algorithm that creates a visual “fingerprint” of the image. Images whose pHashes differ by up to 4 bits are treated as near-identical copies.  
3. **Feature-Based Duplicates (Optional):** If you have already extracted visual features, Moodie can compare these “visual codes.” Images with very close codes (very high cosine similarity) are considered visually similar and treated as duplicates.

Moodie keeps the first image found in each duplicate group and removes the others before generating the visual walls. A summary of the number of duplicates removed by each method is displayed.

**Output:**

At the end of the process, Moodie generates one or more `.jpg` files containing the visual walls of your corpus (overall and/or grouped), saved in a folder named `imagewall` inside your `project_dir`. If you chose to include color palettes, they will appear beneath each wall. You can view these images directly in your analysis environment.

Corpus visualization is a supportive feature for obtaining quick insights into the visual content of your image collection, identifying patterns, and communicating your findings in a visually rich way.

---

### 2.6 · Moodie Trends — Recommendation System

> **Goal**  
> Find similar images, build curations, and generate color panels using combinations of **visual similarity (embeddings)** and **optional metadata**. Beyond these goals, Moodie Trends can also be used as a practical tool for examining patterns, mapping trends within a dataset, or conducting visual searches across an image library.

This module was designed with two primary aims: to teach, in a practical and intuitive way, the rationale of a recommendation algorithm, and to investigate biases and discrepancies that emerge when different computer vision architectures are inserted into the same recommendation system. The idea is to [*borrow the algorithmic logic*](https://eliasbitencourt.com/research) of hybrid recommendation systems and reapply it as a methodological strategy. In other words, to use recommendation algorithms **as tools for studying the very curations produced by recommendation systems themselves**.

---


![](dashboard_exemplo.png)

#### 1. Initial Configuration

| Step                        | What it is                                                        | What you do in the interface |
|----------------------------|-------------------------------------------------------------------|------------------------------|
| **Image Column**           | Filename of the image in the DataFrame.                           | Select the correct column from the menu. |
| **Embedding Column**       | Visual feature vectors (generated in the previous module).         | Select the embedding column. |
| **Similarity Metric**      | Formula that measures the “distance” between vectors (Cosine, Euclidean, Correlation). | Choose from the menu. |
| **Embedding Weight**       | How much visual similarity should weigh relative to optional metadata. | Adjust the slider (0–10). |

---

#### 2. Similarity Metrics (Embeddings)

| Metric        | How it computes similarity                                                                                   | Use when…                                                                                                   |
|---------------|--------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| **Cosine**    | Measures the angle between vectors, ignoring magnitude.                                                      | You want to find images with similar visual patterns (composition, elements) regardless of brightness or contrast. |
| **Euclidean** | Computes geometric distance between vectors. Smaller distance indicates higher similarity.                   | You want to find images that are globally close in terms of color and texture.                               |
| **Correlation** | Measures linear relationships between vectors, focusing on similar variations in visual characteristics.   | You want to find images with similar lighting changes or color variations, ignoring absolute feature levels. |

---

#### 3. Extra Columns (optional)

After selecting the columns and clicking **“Proceed to Adjustments”**, each column appears with three controls:

1. **Aggregator**  
2. **Separator**  
3. **Weight**

These three settings determine **how** the column influences the final similarity calculation.

##### 3.1 Available Aggregators

| Column Type        | Aggregator | How it works | When to use |
|--------------------|-----------|--------------|-------------|
| **Numeric**        | `none`    | Uses the first value in the cell. | When there is only one numeric value (e.g., age, single price). |
|                    | `mean`    | Computes the average of the values split by the separator. | When there are multiple numbers and you want the overall tendency. |
|                    | `median`  | Computes the median of the values split by the separator. | When you want to reduce the impact of very high or very low values. |
| **Text / Set**     | `none`    | Literal string comparison. | When the cell has a single label (“male”, “Y2K”). |
|                    | `jaccard` | Computes the Jaccard similarity between **sets of terms**. The value ranges from 0 (no overlap) to 1 (all terms overlap). | When the cell contains **multiple terms** (tags, styles, colors) separated by commas or another delimiter. |

##### 3.2 Separator

*Character that splits the cell into pieces before the aggregator is applied.*

| Example cell            | Correct separator | Effect |
|-------------------------|-------------------|--------|
| `10,20,30`              | `,`               | Three numbers → mean or median. |
| `azul;vermelho;verde`   | `;`               | Three terms → Jaccard similarity between sets. |
| `Branco`                | *(empty)*         | Cell treated as a single value. |


##### 3.3 Weight

A 0–10 scale indicating **how strongly** the extra column influences the final similarity score relative to the embedding weight.

---

#### 4. Similarity Search

| Step                     | What it defines                               | In practice |
|--------------------------|------------------------------------------------|-------------|
| **Reference Mode**       | Source of the example image.                  | *Internal image* or *External upload*. |
| **Reference Image**      | File used as the “query.”                     | Type the filename (internal) or upload a file. |
| **Top N**                | Number of images to return.                    | Adjust the slider. |
| **Ordering Criterion**   | How the Top N results are ranked (Relevance, Contrast, Concordance). | Choose from the menu. |

##### How Moodie handles **external images**

1. The image is uploaded and receives **the same embedding model** selected for the recommendation.  
2. Moodie computes the similarity between this external vector and **all** internal images.  
3. The most similar internal image becomes the “bridge” (internal reference).  
4. The rest of the algorithm proceeds exactly as if the reference were internal.

This allows users to query with **any** external image (for example, from Pinterest) and receive recommendations from within their own corpus.


---

#### 5. Ordering Criteria

| Criterion        | Scoring logic                                 | Typical result |
|------------------|-----------------------------------------------|----------------|
| **Relevance**    | Score = similarity (1 → 0).                    | “Most similar” first. |
| **Contrast**     | Score = 1 − similarity.                        | “Most different” first (diversity-oriented search). |
| **Concordance**  | Score = 1 − 2·|sim − 0.5|.                     | Images that sit in a “middle zone” between similar and dissimilar. |

---

#### 6. Outputs Generated

- **Complete and summary CSVs** with scores and metadata.  
- **Image-wall** of the Top N results (rank order).  
- **Color dashboards** (dominant and representative) + **accessible palettes**.  
- **ZIP** containing recommended images, reports, and dashboards for quick download.

These resources allow students and researchers to rapidly assess **similarity, diversity, and chromatic composition** within their recommendations.

---


## About the Author

**Elias Bitencourt** is an Associate Professor at the Design Program of the State University of Bahia (UNEB). He holds a PhD in Communication from FACOM/UFBA and an MA in Culture and Society from IHAC/UFBA. He was a visiting researcher at the Milieux Institute (Concordia University, Canada) in 2019. He currently leads **Datalab/Design (CNPq)**, a research center focused on data visualization and digital methods. His research investigates data visualization, platform studies, digital imaginaries, and algorithmic mediation in social relations. He is a collaborating researcher at the **Inova Media Lab** (Universidade Nova de Lisboa) and at the international **Public Data Lab**.  
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

Bitencourt, E. (2025). *MOODIE Image Trends: Modular Observational & Operational Design Image Explorer* (Beta version) [GitHub repository]. Datalab/Design – Universidade do Estado da Bahia. https://github.com/datalabdesign/moodie_english
