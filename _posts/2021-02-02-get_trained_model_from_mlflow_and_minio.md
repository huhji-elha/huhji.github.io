---
layout: post
title:  "MLflow와 MinIO에서 학습 완료된 모델 불러오기 : torch, tensorflow, scikit-learn"
date:   2021-02-02 20:11:03 +0530
categories: MLflow MinIO Pytorch Tensorflow scikit-learn
---
🤹‍♀️ 학습된 모델을 새로운 환경에서 불러와 사용해야 하는 경우

_____________________________________


#### 모듈 설계

- **Train module** : K8s에서 학습 ▶️ mlflow를 이용해 학습된 모델을 MinIO에 저장 ▶️ mlflow의 run_uuid를 DB에 저장

- **Inference module** : 학습 완료된 모델 중 추론할 모델 선택 ▶️ 선택한 모델에 해당하는 run_uuid 조회 ▶️ MinIO Storage에서 모델 파일 가져오기 ▶️ 저장된 모델 형식에 따라 불러와 추론 진행

#### 개발 조건 

- MinIO 서버에서 학습된 모델을 불러온다.
- 기존 학습에 대한 정보가 없는 새로운 K8s Pod에서 추론을 실행하기 때문에 학습한 모델 + 모델 코드 정보가 필요한 상황


> Tensorflow — .h5로 우선 저장하되, h5 형식으로 저장이 안되는 코드는 checkpoint 형식으로 저장

> Pytorch — .pth로 저장, 학습한 model 자체를 인스턴스로 저장. torch의 save 모듈이 pickle을 사용하므로 모델 파일(ex. resnet.py)을 경로와 함께 저장하는 것이 필요 (pickle은 파일 구조를 함께 저장한다.)

> ScikitLearn — .pkl로 저장




## Minio에서 파일 가져오기

```python
#!pip install minio
from minio import Minio

class MinioManager:
    def __init__(self, run_uuid):
        self.client = Minio("mlflow_s3_checkpoint", "minio_access_key", "minio_secret_acces_key", secure=False)
				# ex) Minio("123.456.78.9:9000", "myname", "1234!")
        self.run_uuid = run_uuid

    # model.pth / model.h5 download
	  def load_model_weights(self, weights_file_name):
        weights_path = f"path/{self.run_uuid}/artifacts/model/data/{weights_file_name}"
        self.client.fget_object("storage-path", weights_path, './'+ weights_file_name)

    # model.py download
	  def load_model_object(self):
        model_class_path = f"path/{self.run_uuid}/artifacts/architecture/models/"
        os.makedirs("./models/", exist_ok=True)
        for obj in self.client.list_objects("storage-path", model_class_path, './models/'):
            obj_name = obj.object_name
            if '.pyc' not in obj_name:
                self.client.fget_object("storage-path", obj_name, './' + obj_name[obj_name.index("models/"):])

```

## Tensorflow (2.*)

### .h5

```python
# 모델 저장 방식 (train.py)
os.makedirs('data', exist_ok=True)

# model 객체 저장 가능한 경우
try:
  model.save("data/model.h5")
# weights만 저장 가능한 경우
except:
  model.save_weights("data/model")
  mlflow.log_artifact("models/", artifact_path="architecture")
mlflow.log_artifact("data/", artifact_path="model")
```

```python
import keras
import numpy as np

# minio에서 h5 파일 가져오기
minio_manager = MinioManager(run_uuid="run_uuid_sample")
minio_manager.load_model_weights("model.h5")

# bytes --> h5 형식으로 읽어오기 위해 file write
h5_save = open('model.h5', 'wb')
h5_save.write(h5_model)
h5_save.close()

# keras model load
model = keras.models.load_model('model.h5', compile=False)
# model 형식 : <tensorflow.python.keras.engine.functional.Functional at 0x251a71de710>

# input dummy array
dummy = np.zeros((1, 224, 224, 3)) # ImageNet Trained model
model.predict(dummy)
# output array : (1, 1000)
```

## Pytorch (1.*)

### .pth (model saved)

```python
# 모델 저장 방식 (train.py)
os.makedirs('data', exist_ok=True)

# model 객체 저장 가능한 경우
try:
  torch.save({"model": self.best_model}, "data/model.pth")
# weights 저장만 가능한 경우
except:
  torch.save({"weights": self.best_model.state_dict()}, "data/model.pth")

# models 폴더 내 model.py 저장
mlflow.log_artifact("models/", artifact_path="architecture")
mlflow.log_artifact("data/", artifact_path="model")
```

```python
# model load
import torch
minio_manager = MinioManager(run_uuid="run_uuid_sample")
minio_manager.load_model_weights("model.pth")
minio_manager.load_model_object()

# model load
model = torch.load('model.pth', map_location=torch.device('cuda'))

# 학습된 model 자체를 저장했으므로 다음과 같은 Dict 형태로 되어있다.
"""
{'model': ResNet(
   (criterion): BCEWithLogitsLoss()
   (conv1): Conv2d(3, 64, kernel_size=(7, 7), stride=(2, 2), padding=(3, 3), bias=False)
   (bn1): BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
   (relu): ReLU(inplace=True)
   (sigmoid): Sigmoid()
   (maxpool): MaxPool2d(kernel_size=3, stride=2, padding=1, dilation=1, ceil_mode=False)
   (layer1): Sequential(
     (0): BasicBlock(
       (conv1): Conv2d(64, 64, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
       (bn1): BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
       (relu): ReLU(inplace=True)
       (conv2): Conv2d(64, 64, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
       (bn2): BatchNorm2d(64, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
     )
     .....
"""
# input dummy array
from torchvision import transforms
from PIL import Image
image = Image.open(filename).convert("RGB")
preprocess = transforms.Compose([
        transforms.Resize([256, 256]),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
    ])
input_tensor = preprocess(image)
input_batch = input_tensor.unsqueeze(0)

output = model(input_batch)

# if using GPU
output = model(input_batch.to('cuda'))
```

## Scikit-learn

```bash
# pickle 모델 불러오기 (linear regression)
minio_manager = MinioManager(run_uuid="run_uuid_sample")
minio_manager.load_model_weights("model.pkl")
minio_manager.load_model_object()

# model load
import pickle
with open('model.pkl', 'rb') as f:
    model = pickle.load(f)

# load test data
import pandas as pd
data = pd.read_csv("test.csv")

# output
output = model(data)
```