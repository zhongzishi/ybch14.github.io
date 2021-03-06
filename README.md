# Hello

Welcome to my blog. This blog contains some of the notes I've taken when reading papers, books or something else. Now the topics are updated to Computer Vision (temporarily including object detection, ImageNet evolution and semantic segmentation) and Natural Language Processing (temporarily including only some prior knowledge, deep learning methods are on the TODO list). The articles are all in Chinese since the papers are all in English. 

Hope this can push me learning all the time :-)

P.S.: Recently I'm reading *Introduction of Algorithm* and working on [this repo](https://github.com/ybch14/Intro-of-Algorithm-Implementation.git), this blog may be updating very slowly for a period of time.

## About me

I'm a starting Ph.D. student at THU-NGNLAB, Dept. of E.E., Tsinghua University, majoring NLP (more specifically, sentiment analysis and knowledge graph). In 2017 I was a research intern at Owlii Inc., studying computer graphics and 3D deep learning.

## Content
- Computer Vision
    - Object detection
        - Region Proposal Based Method
            - [Selective Search](CV_Object_detection/Selective_Search.md)
            - [R-CNN (Region-based CNN)](CV_Object_detection/R-CNN.md)
            - [SPP (Spatial Pyramid Pooling)](CV_Object_detection/SPP.md)
            - [Fast R-CNN (Faster version of R-CNN)](CV_Object_detection/Fast_R-CNN.md)
            - [Faster R-CNN (Faster version of Fast R-CNN)](CV_Object_detection/Faster_R-CNN.md)
            - [FPN (Feature Pyramid Network)](CV_Object_detection/FPN.md)
            - [Mask R-CNN (Add semanic segmentation task on Faster R-CNN)](CV_Object_detection/Mask_R-CNN.md)
        - Without Proposal Method
            - YOLO (You only look once, object detection without region proposals)
            - SSD (Single-shot MultiBox Detector, object detection without region proposals)
    - ImageNet Evolution
        - [AlexNet (The first neural network for ImageNet classification, ILSVRC 12 winner)](CV_ImageNet_evolution/AlexNet.md)
        - [GoogLeNet (ILSVRC 14 winner, Inception v1)](CV_ImageNet_evolution/GoogLeNet.md)
            - [Inception v2 (Inception + BN)](CV_ImageNet_evolution/Inception-v2.md)
            - [Inception v3 (Inception + Factorization)](CV_ImageNet_evolution/Inception-v3.md)
            - [Inception v4 (Optimized Inception v3)](CV_ImageNet_evolution/Inception-v4.md)
            - [Inception-ResNet v1 (Inception + Residual Connection)](CV_ImageNet_evolution/Inception-ResNet-v1.md)
            - [Inception-ResNet v2 (Larger Inception-ResNet v1)](CV_ImageNet_evolution/Inception-ResNet-v2.md)
        - [VGGNet (ILSVRC 14 second winner)](CV_ImageNet_evolution/VGGNet.md)
        - ResNet (ILSVRC 15 winner, COCO 15 winner)
        - ResNeXt (Combined version of ResNet and Inception)
        - DenseNet (CVPR 2017 Best Paper)
    - Semantic Segmentation
        - [FCN (Fully Connected Networks)](CV_Semantic_segmentation/FCN.md)
        - DeepLab (First introducing CRF)
        - FCIS (COCO 2016 winner)
        - R-FCN (Region-based FCN)
    - 3D Deep Learning
        - PointNet & PointNet++ (Point cloud deep learning architecture)
        - PointCNN (Another point cloud deep learning method)

- Natural Language Processing
    - Machine Learning basis
        - [LDA (Latent Dirichlet Allocation)](NLP_Machine_learning_basis/LDA.md)
        - CRF (Conditional Random Field)
    - Named Entity Recognition
        - [Semi-supervised sequence tagging with bidirectional language models](NLP_Named_Entity_Recognition/Semi_supervised_sequence_tagging_with_bidirectional_language_models.md)
    - Relation Extraction
        - Fully Supervised Method
            - [TreeLSTM](NLP_FSRE/TreeLSTM.md)
        - Distant Supervised Method
    - Sentiment Analysis
    - Knowledge Graph Embedding
- Recommender System 
    - [Deep Learning Methods Survey](RS_survey/survey.md)

## TODO

- DESIGN SIDE BAR NAV MENU
- Complete the above list

## Note

- This page originally come from jekyll Cayman theme. All the other parts and color adjustments are written by myself with CSS and JS. Writing a static web page is tried but interesting :-) The network architecture is supported by ```mermaid``` ([GitHub](https://github.com/knsv/mermaid)). Great thanks to the author!
- The side bar is under construction.
