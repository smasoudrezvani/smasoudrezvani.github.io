
# Enhancing Graph Neural Network-based Fraud Detectors against Camouflaged Fraudsters

This repository contains the implementation of a fraud detection system based on Graph Neural Networks (GNNs), inspired by the paper "Enhancing Graph Neural Network-based Fraud Detectors against Camouflaged Fraudsters" by Yingtong Dou et al. The system is designed to detect fraudsters who camouflage their fraudulent behaviors to avoid detection. It integrates reinforcement learning and similarity measures to improve the GNN's performance.


# Project Overview
Fraud detection is a critical application of machine learning, especially for systems with complex interactions between users. This project tackles the challenge of fraudster camouflage, where fraudulent behaviors are disguised to bypass conventional detection methods.

The model used in this project, called CARE-GNN (CAmouflage-REsistant Graph Neural Network), implements three key components:

- 1. Label-Aware Similarity Measure: This helps identify the most informative neighbors of each node based on domain-specific labels.
- 2. Reinforcement Learning-based Neighbor Selection: Adaptive RL techniques are used to optimize neighbor selection thresholds, filtering out irrelevant or camouflaged nodes.
- 3. Relation-Aware Aggregation: The model aggregates information across different relations between nodes in a graph, ensuring more accurate fraud detection.


# Key Features
- Adaptive neighbor selection based on similarity and reinforcement learning.
- Multi-relation graph aggregation for better performance in complex fraud networks.
- Benchmarked against real-world datasets, achieving state-of-the-art results.

# Installation
To run the code in this repository, follow these steps:

1. Clone the repository:
''' bash

git clone https://github.com/vida-lashani/GNN_Fraud_Detection.git'''
2. Install the necessary Python libraries:

''' pip install -r requirements.txt'''

# Usage
You can run the code through the Jupyter notebook provided (similarity_measure.ipynb). This notebook implements the CARE-GNN model and runs experiments on a dataset (e.g., Yelp, Amazon) to detect fraudsters.

To start the Jupyter notebook, run:

'''jupyter notebook similarity_measure.ipynb'''

Ensure that you have your dataset in the correct format as described in the paper. You can customize the dataset loading part of the notebook to match your file structure.

# Datasets
The model is tested on two real-world datasets:

- Yelp: Hotel and restaurant reviews, where fraudsters manipulate reviews to deceive users.
- Amazon: Product reviews, where fraudulent users provide misleading feedback.
Refer to the original paper for detailed dataset descriptions and preprocessing methods.

# Results
The experiments demonstrate that CARE-GNN outperforms existing GNN models in detecting camouflaged fraudsters. The use of adaptive thresholds and relation-based aggregation significantly improves the model’s accuracy.


