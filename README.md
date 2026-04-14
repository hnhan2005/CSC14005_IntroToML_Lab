# Lab 01: Vietnam Travel Planner
**Models**:
- Traditional: Random Forest
- Hybrid: Content-based Filtering + XGBoost
- Deep learning: TextCNN

---
## Group information

**Course:** CSC14005 - Introduction to Machine Learninng
**Group 3 - Class 23KHDL1**

1. **23127102 - Lê Quang Phúc**
   - Contributions: Problem definitionm, data collection, modeling for Traditional paradigm, README.md

2. **23127212 - Nguyễn Quang Đăng Khoa**
   - Contributions: Problem definition, data collection, modeling for Deep Learning paradigm, application

3. **23127241 - Đoàn Thành Phát**
   - Contributions: Problem definition, data exploration, modeling for Hybrid paradigm, application

4. **23127332 - Trần Tiến Cường**
   - Contributions: Problem definition, data exploration, modeling for Hybrid paradigm, report

5. **23127442 - Trầm Hữu Nhân**
   - Contributions: Problem definition, data collection, modeling for Deep Learning paradigm, report

---

## Overview

The problem is to build a **recommendation system** that suggests lists of places, applying **multi-class text classification** based on **machine learning** models, and then performing post-processing and inference to generate a suggested list. This helps to create a suggested travel list based on the user's input description.

**Output:** Recommendation list: 
- Top 3 Attraction
- Top 2 Dining  
- Top 1 Hotel

---

## Machine learning problem
The problem is stated in terms of **multi-class classification**:

**Input:** User comments/descriptions + some characteristic information (ratings, price, etc.) **Output:** Prediction of the most suitable location (class) + prediction probability

---

## Data source and description

### Source
Data was collected from popular travel platforms:
- **Google Maps** - Locations, reviews, comments
- **Booking.com** - Hotels, prices, reviews
- **Foody.vn** - Restaurants, ratings, comments
Focus on **Ho Chi Minh City** with the following categories:
- **Hotel** - Accommodation
- **Dining** - Restaurants and eateries
- **Attraction** - Tourist attractions

### Features description
| Columns | Data type | Description |
|-----|------|-------|
| name | str | location name |
| address | str | location address |
| star | float | rating score (0-5) |
| nums_comments | int | total of comments |
| price | int | ticker price |
| category | str | category |
| hours | str | operation hours |
| comments | list str | list of comments |
| comment_scores | list float | list of comment scores |
| url | str | path |


### Data info
- Approximately 6,000+ orginal locations
- Approximately 130,000+ reviews after exploding
- 3 main categories: Attraction, Hotel, Dining

---

## Pipeline
- Preprocessing and features engineering
- Data splitting
- Model training
- Model evaluation and final retraining
- Post processing and system inference

## File Structure Explanation

The project is organized into multiple Jupyter notebooks, each serving a specific purpose:

```
CSC14005_INTROTOML_LAB/
│
├── checkpoints/
│   ├── *. pkl                           # Model checkpoints
│
├── data/
│   ├── *. csv                           # Data files
│
├── notebooks/
│   ├── collection_data.ipynb            # Dataset collection and evaluation
│   ├── exploration_dât.ipynb            # EDA and preprocessing
│   ├── deep_learning.ipynb              # Deep learning paradigm - TextCNN
│   ├── hybrid.ipynb                     # Hybrid paradigm - CBF + XGBoost
│   └── traditional.ipynb                # Traditional paradigm - Random Forest
│
├── .gitignore                           # Git ignore rules
├── requirements.txt                     # Python dependencies
├── README.md                            # Project documentation
└── link.txt                             # Relevant links 

```

## How to Run Instructions

### Prerequisites
- Python 3.11.9
- RAM: 8GB or higher
- Disk space: ~6GB for dependencies and data

### Installation steps

1. **Clone the repository:**

```bash
git clone <https://github.com/hnhan2005/CSC14005_IntroToML_Lab.git>
cd CSC14005_INTROTOML_LAB
```

2. **Create virtual environment (recommended):**

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/Mac
python3 -m venv venv
source venv/bin/activate
```

3. **Install dependencies:**

```bash
pip install -r requirements.txt
```

4. **Download pretrained embedding:**
- FastTest pretrained embedding: [cc.vi.300.bin](https://fasttext.cc/docs/en/crawl-vectors.html)

## Relevant links
- Application repo: https://github.com/tphat2205/HCM-Trip-Planner
- Application link: https://hcm-trip-planner.vercel.app/
- Machine learning checkpoints: https://drive.google.com/drive/folders/1l0caQ4srtrgOgc4sOkse8I7Zb2pMDFny?usp=sharing

## References
- Original TextCNN: [Yoon Kim, 2014](https://arxiv.org/abs/1408.5882)
- FastTest pretrained embedding: [cc.vi.300.bin](https://fasttext.cc/docs/en/crawl-vectors.html)

