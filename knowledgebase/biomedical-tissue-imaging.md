# Biomedical Tissue Imaging / DSU Biomedical Research Lab

> Index: [00_INDEX.md](00_INDEX.md) · Roles: Data Scientist, ML Engineer, AI Research Intern, Computer Vision Engineer, Research Assistant, Biomedical Data Scientist

## Metadata
- Project / Organization: Biomedical Tissue Imaging; DSU Biomedical Research Lab
- Role: Data Science Researcher
- Primary tools: Python, PyTorch, YOLOv8, pandas, NumPy, Matplotlib, OpenCV
- Primary domains: Biomedical AI, Computer Vision, Deep Learning, Image Segmentation, Research
- Modeling context: CNN-based segmentation, Cyto R-CNN, YOLOv8, cytoplasm/nucleus segmentation, COCO annotations

## Experience Scope
Biomedical tissue imaging research improving ML models for cell segmentation in histological images: CNN-based image segmentation, cytoplasm boundary detection, annotation conversion, model evaluation, and research communication. Explored improving Cyto R-CNN / whole-cell imaging approaches and later YOLOv8-based segmentation. Roy contributed to the technical pipeline and authored/published an article analyzing CNN-based segmentation models and AP metrics for cytoplasm boundary precision.

## Research Problem
Accurate segmentation of cell structures in histological images matters for cancer research and biomedical analysis. Traditional and deep learning approaches can struggle with precise cytoplasm boundary detection, whole-cell segmentation, and annotation compatibility. The project aimed to improve segmentation accuracy and evaluate performance across architectures and metrics.

## Responsibilities
- Developing or supporting CNN models for instance segmentation.
- Working with PyTorch-based model workflows.
- Performing hyperparameter tuning.
- Optimizing Cyto R-CNN and whole-cell imaging techniques.
- Exploring alternative deep learning architectures.
- Working with YOLOv8 for cytoplasm segmentation.
- Engineering a data pipeline to convert 4k+ CytoNuke annotations into COCO format.
- Supporting training across 3 model architectures.
- Evaluating segmentation quality using AP metrics.
- Publishing/writing an article explaining CNN-based segmentation models and boundary precision.
- Communicating research findings through presentations or written deliverables.

## Technical Contributions
**Annotation conversion pipeline:** Engineered a pipeline converting thousands of CytoNuke annotations into COCO format (required for many CV model architectures).
**Model training & evaluation:** Supported training YOLOv8 in PyTorch; evaluated segmentation using AP/AP50 — improved cytoplasm segmentation with ~5% AP50 improvement over Cyto-RCNN.
**Research communication:** Published/authored an article analyzing CNN-based segmentation models and AP metrics — explaining complex model evaluation in a research format.

## Challenges
COCO conversion issues; deadline pressure; need to evaluate alternative model paths; balancing research article development with model experimentation; biomedical imaging complexity; measuring cytoplasm boundary precision effectively. Roy proposed a parallel path: while teammates continued YOLOv8 improvements, he supported a Medium/article-style deliverable to ensure a strong final output.

## Goals and Impact
- Converted 4k+ annotations into COCO format.
- Enabled training across 3 model architectures.
- Improved cytoplasm segmentation using YOLOv8.
- Increased AP50 by ~5% over Cyto-RCNN.
- Supported cancer research through better histological image analysis.
- Produced research communication explaining model evaluation and segmentation.

## Skills Demonstrated
**Technical:** Python, PyTorch, YOLOv8, CNNs, computer vision, image segmentation, instance segmentation, COCO annotation format, OpenCV, pandas, NumPy, Matplotlib, AP/AP50 evaluation, hyperparameter tuning, biomedical AI.
**Business / soft / research:** Research communication, technical writing, problem solving under deadline pressure, collaboration, experimental troubleshooting, model evaluation explanation, biomedical domain learning.

## Applicable Role Families
Data Scientist, Machine Learning Engineer, AI Research Intern, Computer Vision Engineer, Data Analyst (research), Research Assistant, Biomedical Data Scientist, Software Engineer (ML systems).

## Interview Story Angles & STAR Story
1. Converting thousands of annotations into COCO format to enable model training.
2. Improving cytoplasm segmentation AP50 by 5%.
3. Troubleshooting model architecture and data format issues under deadline pressure.
4. Explaining AP metrics and segmentation quality in a published article.
5. Applying deep learning to biomedical research.

**STAR — Segmentation Pipeline:** S: Biomedical image segmentation required standardized annotation formats and better cytoplasm detection. T: Prepare data and improve model performance. A: Converted 4k+ annotations to COCO and supported YOLOv8 training. R: Improved AP50 by ~5% over Cyto-RCNN. *Best for: ML, computer vision, research.*
