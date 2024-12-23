# 이윤영 - SegDiff: Image Segmentation with Diffusion Probabilistic Models

# 1. Introduction

- 대다수의 diffusion 모델은 절대적인 정답이 없는 도메인에 적용됨.
- 그래서 본 연구에서는 segmentation problem에 도전
- 기존 segmentation의 최신 방법들과 달리, end-to-end 학습
- 입력 이미지 $I$와 binary segmentation map의 현재 예측치인 $x_t$는 서로 다른 인코더를 통해 처리되며, 이 다채널 텐서의 합은 다음 예측치 $x_{t-1}$을 제공하기 위해 U-Net을 통과하는 디노이징 네트워크를 사용함
- diffusion 과정이 본질적으로 확률적이기 때문에 여러 번의 결과를 단순히 평균하여 결합
    
    → 전체적인 정확도 향상
    

**주요 기여 사항**

- 최초로 확산 모델을 이미지 분할 문제에 적용함
- 입력 이미지에 모델을 조건화하는 새로운 방법을 제안함
- 확산 모델의 성능과 보정을 개선하기 위해 multiple generation 개념을 도입함
- 여러 벤치마크에서 sota 성능 달성 (특히 작은 데이터셋에서 큰 성능 차이 보임)

# 2. Related Work

## Image Segmentation

- 각 픽셀이 특정 클래스에 속하는지 여부를 나타내는 레이블을 할당하는 문제
- U-Net, transformer, 하이퍼네트워크 등을 결합한 모델들이 존재함

## Diffusion Probabilistic Models, DPM

- 간단한 가우시안 분포를 복잡한 분포로 변환할 수 있는 마르코프 체인 기반 생성 모델

## Conditional Diffusion Probabilistic Models

- 본 연구에서는 이미지 segmentation 문제를 조건부 생성으로서 해결하기 위해 확산 모델을 사용함
- class-conditioned generation 방법이 포함됨
    - 클래스 임베딩을 타임 스탬프 임베딩에 추가하여 얻음
    - 추가 학습 없이 주어진 참조 이미지를 기반으로 이미지를 생성함
- image-to-image / text-to-image 등의 다양한 기법에서 사용됨

# 3. Background

- 기본적인 DDPM의 수학적인 논리를 설명함
- DDPM
    
<img width="400" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-07_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7 43 22" src="https://github.com/user-attachments/assets/e4bd5fa8-276e-4163-a7b6-cf09ccc38e8b">

    
<img width="400" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-07_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7 44 05" src="https://github.com/user-attachments/assets/67408e0f-3dd6-4f7f-bc4b-382dc9025465">

    

# 4. Method

<img width="945" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-07_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 11 27" src="https://github.com/user-attachments/assets/b5c09b0c-da6c-41ca-8c7d-68453b38f701">


- $x_t$ : 현재 예측값
- $I$ : 입력 이미지
- $\epsilon_\theta$ : 노이즈 제거 함수

<img width="309" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-08_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7 18 26" src="https://github.com/user-attachments/assets/040bffca-3527-461b-95f5-37595155f171">


- $E$ : U-Net의 인코더
- $F$ : $x_t$의 segmentation map을 인코딩
- $G$ : 입력 이미지 인코딩
- $D$ : U-Net의 디코더
- residual connection을 통해서 $F(x_t)+G(I)$로 더해줘서 인코더$E$에 전달함
- 현재 단계 인덱스 $t$는 두개의 다른 네트워크 $D, E$에 전달됨.

<img width="473" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6 12 28" src="https://github.com/user-attachments/assets/e8264dea-6ba0-4ede-85e4-5784f46627d8">


## 4.1. Imploying Multiple Generation

<img width="603" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6 33 42" src="https://github.com/user-attachments/assets/7dbde935-3ca0-49e0-be47-3b9c4ada09c4">


- 표준 분포에서 랜덤하게 샘플링된 $z$ 때문에 추가적인 노이즈 $\sigma_\theta(x_t, t)\cdot z$ 는 inference마다 달라질 수 있음
- 이러한 이유 때문에 동일한 input에서 여러번의 inference를 돌리면 결과에 큰 변동성이 발생하게 됨
- 본 연구에서는 이 문제를 inference를 여러번 돌려서 결과를 단순 평균화
    - segmentation 결과를 안정화하고 성능을 향상함

## 4.2. Training

- 확산 단계의 총 수 : $T$ → 사용자의 input
- 각 반복에서 랜덤 샘플 $(I_i , M_i)$(이미지와 관련된 binary segmentation map 정답)을 가져옴
- 반복 번호 $1\leq t \leq T$는 uniform한 분포에서 샘플링, $\epsilon$은 표준 분포에서 샘플링됨
- $x_t$ 샘플링하고, $F(x_t)+G(I_i)$계산하고, 네트워크 $E, D$ 적용하여 $\epsilon_\theta (x_t, I_i, t)$를 얻음

<img width="363" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6 45 46" src="https://github.com/user-attachments/assets/f1ae621a-fd76-41e0-b91c-c0c94b8f3a41">


- 최종 손실은 위와 같음
- training할 때, 입력이미지 $I_i$의 정답 segmentation 알려져 있으며, 손실은 $x_0 = M_i$로 설정하여 계산

## 4.3. Architecture

- 입력 이미지 인코더 $G$는 batch normalize없이 RRDB(Residual in Residual Dense Blocks)로 구성됨
- $G$ : 2D conv input layer → RRDB → 2D conv layer → Leaky ReLU → 2D conv output layer
- $F$ : 단일 채널 입력 + $C$채널 출력 가지는 2D conv layer
- $D,\  E$는 U-Net 기반 - 각 레벨은 residual block으로 되어있음
    - residual block은 두개의 conv block으로 구성됨
    - conv 블록은 group norm, SiLU, 2D conv layer로 구성됨
    - residual 블록은 time 임베딩을 linear layer, SiLU, 또 다른 linear layer로 구성됨
        - 그 결과는 첫번째 2D conv block 출력에 더해짐
    - residual block에는 residual connection이 포함되어있음
- 16*16, 8*8 resolution일 때는 attention layer가 있음
- $E$에는 동일한 depth의 residual block 뒤에 downsample block이 있으며, 이는 stride가 2인 2D conv layer.
- $D$에는 동일한 depth의residual block 뒤에 upsample block이 있으며, 이는 공간 크기를 두배로 늘려주는 nearest iterpolation, 2D conv layer로 이루어짐

# 5. Experiments

### DATASET / Evaluation

- 총 세가지의 데이터셋을 사용함
1. Cityscapes dataset
    - 8개 객체 카테고리가 있음
    - 목표 : 각 객체 주위에 바운딩 박스를 포함한 잘린 패치가 주어졌을때, 객체의 픽셀단위 마스크를 복구하는것
    - mIoU를 통해 평가됨
        
<img width="443" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 18 47" src="https://github.com/user-attachments/assets/c791b67a-de91-4ffe-b2b7-fd01a23ec7cc">

        
2. Vauhingen dataset
    - 독일 지역의 항공이미지로 구성되어있음
    - 목표 : 각 이미지에서 중심 건물을 segmentation 하는 것
    - mIoU, F1-score, WCov, BoundF등 여러 지표를 통해 평가됨
    - 예측이 정답과 일정한 거리 threshold 내에 있으면 올바른 것으로 간주
3. MoNuSeg dataset
    - 7개 장기에서 개별 핵 주석이 포함된 현미경 이미지
    - mIoU, F1-score 통해 평가됨

### Training Details

- 각 데이터셋에 맞는 학습 방법의 디테일들…
- Segformer Stdc를 제외한 모든 baseline 방법들은 pretrain된 weight에 의존함 → 우리는 랜덤하게 초기화 했다~

### Results

<img width="915" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 24 23" src="https://github.com/user-attachments/assets/501bfc2b-7aef-4f35-8288-0d2fc18029cb">


<img width="435" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 24 37" src="https://github.com/user-attachments/assets/a5bcc711-191c-4c02-b72a-eb7f99a1f08e">


![604abe60-0dc4-4eec-a9af-a95db7c8439d](https://github.com/user-attachments/assets/d042c10a-6950-4b2b-9e2a-82bc2a3370c8)


<img width="458" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 25 43" src="https://github.com/user-attachments/assets/50237547-6dec-4c8c-b514-b8ee57d85894">


- Cityscapes dataset은 두가지 설정에서 평가됨
    - Tight : 객체 마스크 주위의 타이트하게 잘린 패치에서 샘플(이미지와 관련된 segmentation map)을 추출
    - Expansion : 객체 마스크 주위에서 타이트한 크기보다 15% 더 큰 크기로 패치를 추출함 → 객체의 위치에 대한 정보가 적어 더 어려움
- Cityscapes dataset에서 학습 데이터가 적을수록 Segdiff와 성능차이가 뚜렷하다는것이 보임(Fig 3.)

<img width="905" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 34 28" src="https://github.com/user-attachments/assets/6a9f9af9-7c70-41de-a830-899105a24a45">


- diffusion step이 증가함에 따라 모든 데이터셋에서 mIoU가 증가함
- 60 step 이후로는 성능이 특정 상태에 수렴, 성능 향상 거의 없음

<img width="913" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 39 37" src="https://github.com/user-attachments/assets/cc531adb-2850-4654-bc84-92814ca65795">


- generated instance가 증가할수록 mIoU 점수가 향상되는 경향이 있음, 반면에 속도 또한 느려짐

<img width="896" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 44 27" src="https://github.com/user-attachments/assets/9b609b83-e768-4e1b-8ddb-6f919cf59cbf">


- generated instance를 통한 성능 향상의 또다른 측면에는 calibration(모델이 예측한 확률이 실제 결과와 얼마나 일치하냐)
- generated instance가 증가할수록 calibration 점수가 향상됨

### Ablation Study

- 총 네가지의 Ablation 실험해봄
    1. channel dimension에 $[F(x_t), G(I)]$연결해봄
    2. RRDB 대신 FC-HarDNet-70 V2 사용
    3. $x_t$에 $I$를 채널별로 연결해서 인코더를 사용하지 않음
    
     4~6. $F(x_t)$를 U-Net 모듈을 통해 전달하고, 첫번째, 세번쨰, 다섯번째 downsample block 후에 $G(I)$에 추가함
    

<img width="902" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 52 43" src="https://github.com/user-attachments/assets/48a15e5a-4aad-404a-a239-94052bf089ff">


<img width="464" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8 53 08" src="https://github.com/user-attachments/assets/816637e9-f5ce-47a9-a15b-afff62845845">


### Parameter Sensitivity

- 안정성을 테스트하기 위해 성능에 가장 큰 영향을 미칠 수 있는 두가지 하이퍼파라미터(diffusion step, RRDB block 개수)로 실험함

<img width="874" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-11_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9 02 49" src="https://github.com/user-attachments/assets/d0d8a8b5-cca2-409b-a6d1-88b69c432102">


- **RRDB block 개수의 효과**(diffusion step은 100으로 고정)
    - 성능 차이가 크게 없었음

<img width="941" alt="%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-11_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9 03 27" src="https://github.com/user-attachments/assets/7c28d33a-8a87-4b45-a21b-d001238f9ffd">


- **diffusion step T의 효과**(RRDB block 개수는 3으로 고정)
    - diffusion step에 따른 time은 선형적으로 증가함

# 6. Conclusions

- 입력 이미지를 조건화하기 위해 또다른 인코딩 경로를 생성하며, 이는 기존 이미지 segmentation 방법과 유사함
- 다양한 벤치마크에서 SOTA 달성함

**<개인적인 견해>**

- 입력 이미지와 segmentation map을 조건화해서 학습하는 방법은 꽤 좋은 아이디어였던 것 같음. 다만 좀 많이 아쉬웠던건, 원래 segmentation task에서 그런것인지는 모르겠는데 각 dataset별로 성능 차이가 너무 다양하게 나는 것이 아쉬웠음. 특히 ablation에서도 어떠한 파라미터별 직관적인 관계성을 찾아보기 힘들었던 것도 모델의 안정성에 의문을 제기하게 만드는 것 같음
