# ByteTrack
ByteTrack: Multi-Object Tracking BY AssociaTing Every Detection Box

<img src="assets/teasing.png" width="400"/>

## Abstract
Multi-object tracking (MOT) aims at estimating bounding boxes and identities of objects in videos. Most methods obtain identities by associating detection boxes in sequence. To filter out the background, previous methods only keep the boxes whose scores are higher than a threshold. However, the objects in low scores, \eg occluded objects, are simply thrown away and bring non-negligible missing detections and fragmented trajectories. To solve this problem, we present a simple, effective and generic detection-driven association method, called BYTE, tracking BY associaTing Every detection box instead of only the high score ones. We utilize the similarity with tracklets to recover the objects in low score detection boxes and filter out the background, simultaneously. We apply BYTE to 9 different state-of-the-art trackers and achieve consistent improvement on IDF1 score ranging from 1 to 10 points. To put forwards the state-of-the-art performance of MOT, we design a simple and strong tracker, named ByteTrack. For the first time, we achieve 80.3 MOTA, 77.3 IDF1 and 63.1 HOTA on the test set of MOT17 with 30 FPS running speed on a single V100 GPU.

## Tracking performance
### Results on MOT challenge test set
| Dataset    |  MOTA | IDF1 | HOTA | MT | ML | FP | FN | IDs | FPS |
|------------|-------|------|------|-------|-------|------|------|------|------|
|MOT17       | 80.3 | 77.3 | 63.1 | 53.2% | 14.5% | 25491 | 83721 | 2196 | 29.6 |
|MOT20       | 77.8 | 75.2 | 61.3 | 69.2% | 9.5%  | 26249 | 87594 | 1223 | 13.7 |

## Installation

Step1. Install ByteTrack.
```shell
git clone https://github.com/ifzhang/ByteTrack.git
cd ByteTrack
pip3 install -r requirements.txt
python3 setup.py develop
```

Step2. Install [pycocotools](https://github.com/cocodataset/cocoapi).

```shell
pip3 install cython; pip3 install 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI'
```

Step3. Others
```shell
pip3 install cython_bbox
```

## Data preparation

Download [MOT17](https://motchallenge.net/), [MOT20](https://motchallenge.net/), [CrowdHuman](https://www.crowdhuman.org/), [Cityperson](https://github.com/Zhongdao/Towards-Realtime-MOT/blob/master/DATASET_ZOO.md), [ETHZ](https://github.com/Zhongdao/Towards-Realtime-MOT/blob/master/DATASET_ZOO.md) and put them under <ByteTrack_HOME>/datasets in the following structure:
```
datasets
   |——————mot
   |        └——————train
   |        └——————test
   └——————crowdhuman
   |         └——————Crowdhuman_train
   |         └——————Crowdhuman_val
   |         └——————annotation_train.odgt
   |         └——————annotation_val.odgt
   └——————MOT20
   |        └——————train
   |        └——————test
   └——————Cityscapes
   |        └——————images
   |        └——————labels_with_ids
   └——————ETHZ
            └——————eth01
            └——————...
            └——————eth07
```


## Model zoo

### Ablatioin model

Train on CrowdHuman and MOT17 half train, evaluate on MOT17 half val

| Model    |  MOTA | IDF1 | IDs | FPS |
|------------|-------|------|------|------|
|ByteTrack_ablation | 76.6 | 79.3 | 159 | 29.6 |

### MOT17 Test model

Train on CrowdHuman, MOT17, Cityperson and ETHZ, evaluate on MOT17 train

| Model    |  MOTA | IDF1 | IDs | FPS |
|------------|-------|------|------|------|
|ByteTrack_x | 90.0 | 83.3 | 422 | 29.6 |
|ByteTrack_l | 88.7 | 80.7 | 460 | 43.7 |
|ByteTrack_m | 87.0 | 80.1 | 477 | 54.1 |
|ByteTrack_s | 79.2 | 74.3 | 533 | 64.5 |


### MOT20 test model

Train on CrowdHuman and MOT20, evaluate on MOT20 train

| Model    |  MOTA | IDF1 | IDs | FPS |
|------------|-------|------|------|------|
|ByteTrack_x | 90.0 | 83.3 | 422 | 29.6 |

```shell
cd <ByteTrack_HOME>
mkdir pretrained
cd pretrained
hdfs dfs -get hdfs://haruna/home/byte_arnold_hl_vc/zhangyifu/debug1/models/bytetrack_models.tar.gz
tar -zxvf bytetrack_models.tar.gz
```

## Training

* **Train ablation model (MOT17 half train and CrowdHuman)**

```shell
cd <ByteTrack_HOME>
python3 tools/train.py -f exps/example/mot/yolox_x_ablation.py -d 8 -b 48 --fp16 -o -c pretrained/yolox_x.pth
```

* **Train MOT17 test model (MOT17 train, CrowdHuman, Cityperson and ETHZ)**

```shell
cd <ByteTrack_HOME>
python3 tools/train.py -f exps/example/mot/yolox_x_mix_det.py -d 8 -b 48 --fp16 -o -c pretrained/yolox_x.pth
```

* **Train MOT20 test model (MOT20 train, CrowdHuman)**

```shell
cd <ByteTrack_HOME>
python3 tools/train.py -f exps/example/mot/yolox_x_mix_mot20_ch.py -d 8 -b 48 --fp16 -o -c pretrained/yolox_x.pth
```

## Tracking

* **Evaluation on MOT17 half val**

Run ByteTrack:

```shell
cd <ByteTrack_HOME>
python3 tools/track.py -f exps/example/mot/yolox_x_ablation.py -c pretrained/bytetrack_ablation.pth.tar -b 1 -d 1 --fp16 --fuse
```
You can get 76.6 MOTA using our pretrained model. 

Run other trackers:
```shell
python3 tools/track_sort.py -f exps/example/mot/yolox_x_ablation.py -c pretrained/bytetrack_ablation.pth.tar -b 1 -d 1 --fp16 --fuse
python3 tools/track_deepsort.py -f exps/example/mot/yolox_x_ablation.py -c pretrained/bytetrack_ablation.pth.tar -b 1 -d 1 --fp16 --fuse
python3 tools/track_motdt.py -f exps/example/mot/yolox_x_ablation.py -c pretrained/bytetrack_ablation.pth.tar -b 1 -d 1 --fp16 --fuse
```

* **Test on MOT17**

Run ByteTrack:

```shell
cd <ByteTrack_HOME>
python3 tools/track.py -f exps/example/mot/yolox_x_mix_det.py -c pretrained/bytetrack_mot17_test.pth.tar -b 1 -d 1 --fp16 --fuse
python3 tools/interpolation.py
```
Submit the txt files to [MOTChallenge](https://motchallenge.net/) website and you can get 79+ MOTA (For 80+ MOTA, you need to carefully tune the test image size and high score detection threshold of each sequence). 

* **Test on MOT20**

For MOT20, you need to clip the bounding boxes inside the image. 

Run ByteTrack:

```shell
cd <ByteTrack_HOME>
python3 tools/track.py -f exps/example/mot/yolox_x_mix_mot20_ch.py -c pretrained/bytetrack_mot20_test.pth.tar -b 1 -d 1 --fp16 --fuse
python3 tools/interpolation.py
```
Submit the txt files to [MOTChallenge](https://motchallenge.net/) website and you can get 77+ MOTA (For higher MOTA, you need to carefully tune the test image size and high score detection threshold of each sequence). 


## Demo

<img src="assets/palace_demo.gif" width="600"/>

```shell
cd <ByteTrack_HOME>
python3 tools/demo_track.py video -f exps/example/mot/yolox_x_mix_det.py -c pretrained/bytetrack_mot17_test.pth.tar --fp16 --fuse --save_result
```
