<img src="MOODIE_GRABBER.png" alt="Logo MOODIE" width="400"/>

# MOODIE Grabber (Image Collection Module)

**MOODIE Grabber** is the image collection module of the [MOODIE](https://github.com/datalabdesign/moodie) tool, developed to enable the download and preprocessing of images from a CSV file containing links. It is currently in **beta**, serving as the first functional module of the MOODIE tool.

This module was designed as a practical introduction to using images in research pipelines for design, providing everything from collection to the first structured metadata that feeds later stages of visual analysis and creative exploration.

[![Open in Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/datalabdesign/moodie/blob/main/modulo_grabber/moodie_grabber_clean.ipynb)

---

## ✦ Updates (08 / 05 / 2025)

* **Full JSON compatibility**  
  The module automatically detects JSON list structures, flattens nested keys, and converts the content into CSV before analysis — ideal for APIs that return large collections of items.

* **Parallel image download (+ retry/back-off)**  
  Images are now downloaded with up to 10 simultaneous threads, using random *User-Agent* headers and a retry policy (429/5xx codes), speeding up the process and reducing server-side blocking.

---

## ✦ Features

MOODIE Grabber allows:

- **Reading CSV or JSON** files containing image links (configurable field);
- **Statistical sampling** of the full dataset:
  - Proportional sampling, random sampling, class-based sampling, or size-based sampling (upsample/downsample);
- **Duplicate removal** based on selected columns;
- **Automatic parallel image download**, with:
  - Built-in retry/back-off policy;
  - Naming strategies: original name, hexadecimal hash, or perceptual hash;
- **Automatic metadata extraction**:
  - File size, dimensions, type, and image format;
  - EXIF information (when available);
  - Link source domain (useful for provenance analysis);
- Generation of a **structured dataset** with metadata from downloaded images;
- Integration with the other modules of the MOODIE tool (Metadata and Trends).

---

## ✦ Requirements

This module is compatible with Google Colab and does not require local installation. All libraries used are available by default in the Colab environment:

- `pandas`
- `numpy`
- `PIL` (from the `Pillow` package)
- `imagehash`
- `tqdm`
- `requests`
- `ipywidgets`
- `matplotlib`
- `IPython.display`

---

## ✦ How to Use

Running MOODIE Grabber in Google Colab is done **step by step by executing the notebook cells in order** and interacting with the interface. Below is a practical summary of each stage:

### 0.0 Before starting, you will need a CSV
Prepare a `.csv` file with at least one column containing image links, as in the example below:

| col a  | link_imagem                  |
|--------|------------------------------|
| ...    | https://example.com/img1.jpg |
| ...    | https://example.com/img2.jpg |

### 01. Installing dependencies
Click the **“Install Dependencies”** button and wait for the required packages to be installed.

### 02. Connecting to Google Drive (optional but recommended)
MOODIE works best when connected to your Google Drive. This helps avoid slowdowns with large files and ensures your data remains saved across sessions.

If you prefer, you can also work only with local uploads without using Drive.

### 03. Creating working directories
You will be able to configure:

- Whether to save data to your Google Drive or only to Colab’s temporary environment;
- Whether you already have an existing folder or want to create a new project directory;
- The project name, which will be used for naming files, e.g., `dataset_project_name.csv`;
- Notes about the project, stored for future reference.

Set the execution parameters at the beginning of the notebook:

- Path to the CSV file;
- Name of the column containing the links;
- Folder path where images will be saved.

The folder structure is designed for integration with the other MOODIE modules, though this module only uses some of them:

```
/your directory
├── datasets ---> Where CSV datasets will be saved
├── reports ---> Where processing reports and documentation will be saved
├── imagens ---> Where downloaded images will be stored
├── imagewall ---> Where imagewall visualizations will be stored (trends module)
├── paleta ---> Where extracted color palettes will be saved (metadata and trends modules)
```



### 04. Uploading and analyzing the CSV (mandatory)
You will upload a CSV containing your image links. MOODIE will:

- Load the file and store the data in a global DataFrame;
- Display columns and data types;
- Identify missing values;
- Analyze columns with unique values;
- Suggest possible keys for duplicate detection.

### 05. Removing duplicates (optional)
Based on the previous analysis, you may choose to remove duplicates using specific columns. Keep in mind:

- Only remove duplicates if you are certain they represent the same item;
- Datasets with memes or viral images may contain intentional repetitions;
- Always keep a copy of your original CSV.

### 06. Dataset sampling (optional)
You may apply sampling to:

- Reduce dataset size;
- Balance categories;
- Test subsets before processing the full dataset.

### 07. Image naming strategies
Before downloading, choose how you want your images to be named:

- **Original Name**: keeps the name from the URL (may cause conflicts if names repeat);
- **HEX**: generates unique names with hexadecimal identifiers;
- **pHash**: uses the perceptual hash of the image, useful for identifying visually similar content.

### After configuring
**Continue running each cell in sequence**, always reading the instructions and responding to the displayed widgets. MOODIE will take care of the rest automatically.

At the end of execution, a `.csv` dataset with metadata from the downloaded images will be generated, and you will be able to access everything within the subdirectories created as shown above.

---

## About the Author

**Elias Bitencourt** is an Associate Professor in the Design Program at the State University of Bahia (UNEB). He holds a PhD in Communication from FACOM/UFBA and an MA in Culture and Society from IHAC/UFBA. He was a visiting researcher at the Milieux Institute (Concordia University, Canada) in 2019. He currently leads **Datalab/Design (CNPq)** at UNEB, a research center focused on data visualization and digital methods. His research investigates data visualization, platform studies, digital imaginaries, and algorithmic mediation in social relations. He is a collaborating researcher at the **Inova Media Lab** (Universidade Nova de Lisboa) and the international **Public Data Lab** network. [More at](https://eliasbitencourt.com)

---

## Status

The project is currently in **beta phase** and under continuous development. Contributions, suggestions, and collaborations are welcome.

---

## License

This repository is under a **Restricted Use License with Attribution**.  
Content may be used for academic and non-commercial purposes with proper attribution to the author.  
Modifications or redistribution require permission.  
See the [`LICENSE.txt`](./LICENSE.txt) file for more details.

## How to cite MOODIE (APA format):

Bitencourt, E. (2025). *MOODIE Image Trends: Modular Observational & Operational Design Image Explorer* (Beta version) [GitHub repository]. Datalab/Design – Universidade do Estado da Bahia. https://github.com/datalabdesign/moodie

