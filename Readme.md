# Dual-Path Inference Fusion (DPIF) for ZSD

This is the official code for **Inference Fusion with Associative Semantics for Unseen Object Detection**, AAAI 2021.



### Environment

```
2 x TITAN V
python==3.7.6
pytorch==1.4.0
```

Except pytorch, you should also install:

```
pip install -r requirements.txt
```



### Dataset

We provide the data example and training & testing details for MSCOCO 2014 dataset with 65 seen classes and 15 unseen classes.

Use soft link to make the path to coco data:

```
cd ROOT_DIR/data
ln -s /path/to/MSCOCO coco
```

Use provided script to obtained the standard 65/15 MSCOCO ZSD train and test annotations:

```
python convert_coco.py
mv *.json ROOT_DIR/data/annotations/
```



### Concept Association

The association between seen classes and unseen classes is precomputed:

```
data/cls2asso_coco_w2v_dist_5.json
```

It is obtained by this script:

```
python cal_concept_association.py
```



### Pre-trained model

Experiment result on MSCOCO with model pre-trained by us:

| Model                | Evaluation on           | mAP@0.5 | Recall@100 |
| -------------------- | ----------------------- | ------- | ---------- |
| Standard Faster-RCNN | Supervised Seen Classes | 32.34   | 57.73      |
| DPIF                 | ZSD Unseen Classes      | 19.82   | 55.73      |
|                      | GZSD Seen Classes       | 29.82   | 56.68      |
|                      | GZSD Unseen Classes     | 19.46   | 38.70      |
|                      | GZSD Harmonic Mean      | 23.55   | 46.00      |




### Training

Firstly, train the standard Faster-RCNN as our detection framework on the training set:

```
python trainori_net.py --dataset coco6515 --net res50 --bs 14 --nw 4 --lr 0.01 --lr_decay_step 4 --cuda --mGPUs
```

This training step needs 10 epochs to converge.



Then, rename the trained weight:

```
cp ROOT_DIR/models/res50/coco6515/faster_rcnn_1_10_XXXX.pth ROOT_DIR/models/res50/coco6515/faster_rcnn_1_10_vanilla.pth
```



Secondly, train the Visual2Semantic Mapper and Association Predictor on the training set:

```
python trainzsd_net.py --dataset coco6515 --net res50 --bs 18 --nw 4 --lr 0.01 --lr_decay_step 4 --s 10 --cuda --mGPUs
```

This training step usually needs only 1 epoch to converge.



### Testing

If you want to use the model trained by us, copy the weight file to the specific path:

```
cp /path/to/pretrained/models ROOT_DIR/models/res50/coco6515/faster_rcnn_10_1_1.pth 
```



Detect and evaluate 10098 testing COCO images for Zero-shot Detection:

```
CUDA_VISIBLE_DEVICES=0 python testzsd_net.py --dataset coco6515 --net res50 --checksession 10 --checkepoch 1 --checkpoint 1 --cuda
```



Detect and evaluate 10098 testing COCO images for Generalized Zero-shot Detection:

```
CUDA_VISIBLE_DEVICES=1 python testzsd_net.py --dataset coco6515 --net res50 --checksession 10 --checkepoch 1 --checkpoint 1 --cuda --gzsd
```



