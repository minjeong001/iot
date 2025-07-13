
# 반려동물 인식 AI 프로젝트 보고서 실습과제 2

## 1. 서론

이 프로젝트는 인공지능(AI) 기반의 반려동물 얼굴 및 행동 인식 모델 개발을 목표로 한다.  
컴퓨터 비전 기술을 활용한 AI는 반려동물 산업에서 건강 관리, 행동 분석 등에 중요한 역할을 할 수 있다.  
본 프로젝트에서는 이미지 수집 및 Bounding Box 라벨링, YOLOv8 기반의 전이 학습을 통해 효율적인 모델을 구축하였고,  
이는 향후 반려동물 관련 AI 서비스 및 연구에 기여할 수 있는 기반이 될 것이다.

---

## 2. 실습 과정 및 코드

### (1) 데이터셋 폴더 구조 변경
#### 아래 그림처럼 your_dataset 폴더 안의 구조를 정확히 맞추어준다.
```
your_dataset/
├── images/         # 모든 이미지 파일이 들어갈 폴더
│   ├── train/      # 학습 이미지 (dog__1_.jpg ~ dog__81_.jpg)
│   └── val/        # 검증 이미지 (dog__82_.jpg ~ dog__101_.jpg)
├── labels/         # 모든 라벨 파일이 들어갈 폴더  
│   ├── train/      # 학습 라벨 (dog__1_.txt ~ dog__81_.txt)
│   └── val/        # 검증 라벨 (dog__82_.txt ~ dog__101_.txt)
└── your_dataset.yaml

# 파일 이동: 기존 all_data 폴더에서 dog__N_.jpg 파일들을 images/train/ 또는 images/val/로, dog__N_.txt 파일들을 labels/train/ 또는 labels/val/로 정확히 이동시킵니다.
```

### (2) your_dataset.yaml 작성
```yaml
train: images/train/
val: images/val/
nc: 1
names: ['dog']
# 저장 시 주의 : 반드시 UTF-8 인코딩으로 저장
```
---

### (3) 구글 드라이브 업로드

- 수정된 your_dataset 폴더를 다시 dog.zip으로 압축
- `dog.zip` 파일을 구글 드라이브에 업로드 (기존파일 존재시 덮어쓰기)
- 코랩에서 아래 순서대로 실행

```python
!pip install ultralytics  #1단계: Ultralytics 라이브러리 설치
from google.colab import drive #2단계: 구글 드라이브 마운트
drive.mount('/content/drive')
!cp "/content/drive/MyDrive/me/dog.zip" /content/ #3단계: dog.zip 파일 코랩 환경으로 복사
!unzip -q /content/dog.zip -d /content/ #4단계: dog.zip 압축 해제
```

---

### (4) YOLOv8 학습 실행

```python
from ultralytics import YOLO
model = YOLO('yolov8n.pt') #yolov8n모델 로드

results = model.train(
    data='/content/your_dataset/your_dataset.yaml', #수정된 파일 경로
    epochs=50, #학습 반복 횟수
    imgsz=640, #이미지 크기
    batch=16,  #배치 크기
    name='dog_detection_model_folder_structure' #학습 결과 저장
)
```

---

### (5) 모델 저장
#### 구글 드라이브로 저장
```python
results_path = '/content/runs/detect/dog_detection_model_folder_structure/'
destination_path = '/content/drive/MyDrive/YOLO_Models/'
!mkdir -p {destination_path}
!cp -r {results_path} {destination_path}
print(f"학습 결과가 '{destination_path}'에 성공적으로 복사되었습니다.")
```

---

### (6) 모델 예측

```python
model = YOLO('/content/drive/MyDrive/YOLO_Models/dog_detection_model_folder_structure/weights/best.pt') 
test_image_path = '/content/drive/MyDrive/test_dog.jpg' #테스트 할 이미지 경로 입력
results = model.predict(source=test_image_path, save=True, conf=0.5)

for r in results:
    print(f"예측 결과 저장 경로: {r.save_dir}")
print("\n 예측이 완료되었습니다. 결과 이미지는 코랩 파일 탐색기의 'runs/detect/predict' 폴더에서 확인하거나,
저장 경로를 통해 접근할 수 있습니다.")

```
---

## 3. 실습 결과

- 강아지 인식 성공 시, bounding box가 그려진 이미지가 저장됨
- `runs/detect/predict/` 경로에 결과 이미지 확인 가능
- ![강아지 인식 결과](yolo_result.png)
