#Extracting camera-based fingerprints for video forensics
---
## Abstract
- 비디오를 촬영한 장치나 카메라를 식별하는 것은 `author verification`에 도움을 준다
- 또한, 많은 가능한 조작들을 탐지하는 데도 정보원이 될 수 있다.
- 모든 비디오 장치들은 사진(`Image`) / 영상(`Video`) 등에 고유한 흔적을 남긴다.
- 이런 흔적이 없거나, 수정되었다면 조작이 되었다고 볼 수 있다.
- 비디오에 숨겨진 모델 관련 흔적을 잘 드러나게 일종의 카메라의 지문을 추출하는 신경망을 훈련시킨다.
- 모델 네트워크는 `Siamese Strategy`를 이용해 훈련되고, 동일 모델 패치 (같은 모델, 같은 위치) 사이의 ` Distance`를 `minimize`하고 다른 경우는 `maximize`를 취한다.
- `Video noiseprint`에 기반한 방법은 주요 포렌식 테스크에서 잘 수행된다.
- 이 네트워크의 훈련에는 `specific manipulation`이나 `fine-tuning`에서도 사전 지식이 필요하지 않다.

일반적인 이미지에 대해 noistprint를 추출하는 기존의 네트워크는 영상에 대해서는 잘 동작하지 않는다고 한다. 왜 잘 동작하지 않는지에 대해서는 추후 서술하고, 이 논문에서는 영상에서도 잘 동작하게 만들기 위한 훈련 방법론을 제시한다.

## Introduction
- 딥러닝과 컴퓨터 그래픽, 미디어 편집 툴의 확산에 의해 시각적 데이터는 많은 적대적이고 범죄적 목적으로 쉽게 조작 될 수 있다.
- 이러한 공격들은 더욱 복잡하고 빈번하게 일어나고 있다.
- 초기 논문은 대부분 프레임 단계 조작에 집중하였다.
- 하지만, 딥러닝과 컴퓨터 그래픽의 발전으로 비디오에 객체를 제거, 삽입하는 작업은 더욱 쉬워지고 있다.
- 조작을 `detecting`하고 `discovering`하기 위해 이상 탐지 전략을 사용한다.
- 각각의 비디오 패치들은 특이한 흔적을 가지고 있는데, 이 흔적은 특정 하드웨어나 소프트웨어로부터 생성된다.
- 비디오의 모든 `patch`는 같은 `source`로부터 나옴을 예상할 수 있다.
- 만약 같은 비디오의 서로 다른 패치가 불일치한다면, 외부에서 특정 조작을 삽입했다고 간주할 수 있다.
- 이 논문은 수집에 사용된 카메라에 남은 흔적에 의존하며, 획득한 이미지에서 추출된 `Image noise print (Noiseprint: a CNN-based camera model fingerprint)`를 활용해 `source identification` 및 `forgery detection`를 수행한다.
- 훈련 단계에서는 어떠한 조작된 이미지도 사용되지 않았다.
- 따라서 알려진 조작 뿐만 아니라 처음 보는 조작도 탐지할 수 있다.
- `blind method`의 속성을 공유

이 논문에서 새로운 네트워크를 제시하지는 않고 먼저 제시된 네트워크를 비디오 객체로 응용한 것이다. 훈련하는 동안에는 조작된 이미지가 사용이 되지 않으며 `pristine`한 이미지만 사용해 훈련을 진행한다.

## Related work
- 비디오가 이미지보다 더욱 도전적인 일인 이유는 `더 강력한 압축`을 사용하기 때문이다.
- 첫번째 접근은  압축에서 나오는 생성물(`Compression artifacts`)에 의존해 수행한 것이다.
  - 이중 MPEG 압축에 기반
- 두번째 접근은 한 비디오의 객체를 다른 비디오에 복사하여 배경에 더 잘 맞도록 회전시키거나, 축소시킬 때 발생하는 편집 기반 생성물 (`Editing based artifacts`)를 이용한 것이다.
- 최근에는 `CNN-based`방법을 사용하는데, `DeepFake`나 `Face2Face`같은 고급 단계의 조작에서 높은 잠재력을 보여주었다.
  - 하지만, 많은 데이터 쌍이 필요하고 새로운 형태의 조작에는 다시 `Fine-tuning`하는 과정이 필요하다.
- 얼굴 조작을 다루는 다른 방법은 눈 깜박임, 일관성 없는 자세, 얼굴 비대칭 같은 시각적 생성물을 이용하는 것이다.
  하지만, 기술의 발전에 따라 이러한 시각적 생성물은 사라질 가능성이 있다.
- 이에 반해 `PRNU Pattern` 에 의거한 방법은 매우 일반적이고 안정적이며, 다른 `artifact`에 의존하지 않는다.
- `PRNU Pattern`이란 기기 제조 과정에서의 겸함에 의해 발생하는데, 각각의 고유한 패턴이 있기 때문에 장치 지문 `Device fingerprint`라고 칭할 수 있다.
  - 많은 포렌식에서 `PRNU Pattern`을 이용한다.
    - `source identification`과 `forgery localization`을 포함.
- 각각의 기기에서 발생하는 `PRNU Pattern`은 많은 인-카메라, 아웃-카메라에서 많은 처리 단계가 수행이 되기 때문에 최종적으로 저장되는 이미지는 많은 흔적을 가지고 있다.
- 조작을 발견하기 위해 고역 통과 필터링을 기반으로 하는 `Noise residual` 추출 방법론 등 여러가지가 있다.
- 이 논문에선 `CNN-based Noiseprint`의 방식을 따른다.
  - `Siamese network` 의 방식

## Background
- 이 절에서는 `Noiseprint`를 다루는 `CNN-Based`의 방법을 소개한다.
- 이 네트워크는 카메라로 부터 획득한 이미지에서 의미 있는 내용 (`Object?`)를 제거하고 순수 노이즈를 추출하는 것이 목표이다.
- 노이즈를 재현할 수 있는 수학적 모델이 알려져 있지 않기 때문에 시뮬레이션 된 이미지에 의존할 수 없다.
- `Siamese network`의 핵심적인 부분은 동일한 구조와 가중치를 공유하는 2개의 네트워크를 동시에 사용하는 것이다.
- 하나의 `Patch pair`를 병렬적으로 모델에 통과시켜 수행한다.
  - `Patch pair`가 같은 모델, 같은 공간 (같은 카메라, 같은 Grid 위치)인 경우에 거리를 최소화
  - 아닌 경우는 거리를 최대화한다.
- 2개의 병렬적 네트워크에서 하나는 기준, 하나는 비교대상이 되는 것이다.
- 훈련이 종료되면 `CNN network`는 각 입력 이미지에 대해 `noistprint`를 추출할 수 있게 된다.
  - 여기서 추출된 `noiseprint`는 포렌식 작업을 할 수 있을 정도로 충분히 강하다.
  - JPEG 형식의 8 X 8 그리드를 명확하게 보여준다. (이해는 잘 가지 않는 문장)
- `noiseprint`는 `feature clustering`을 통해 `heatmap`으로 표시를 할 수 있다.
- 48 X 48이미지에 대해 훈련을 하였다.

![figure_2]('./images/residual_extractior.jpg')

모델이 사용하는 네트워크는 샴 네트워크로 같은 구조, 가중치를 공유하는 2개의 네트워크를 병렬적으로 구동하여 훈련시키는 것이다. 같은 패치인 경우 최종 Distance가 0에 가깝게 나오고 아닌 경우는 큰 값을 가지게 될 된다.  각각의 기기를 구별하는 분류 모델은 아니지만 하나의 패치 쌍에서 같은 카메라에서 온 것인지 아닌지는 구별이 가능하다. 여기서 훈련된 네트워크를 기반으로 조작된 이미지가 나오게 되면, noiseprint를 feature clustering한 heatmap에서는 깨끗한 이미지의 heatmap과는 다른 결과가 나오게 될 것이다.
하나의 이미지를 학습속도를 위해 48 x 48의 패치로 잘라서 훈련 데이터셋으로 사용했으며 사용할 수 있는 모든 패치 쌍을 만들어서 훈련에 사용했다.

![Siamese_network]('./images/siamese.jpg')
![figure_3]("./images/figure_3.jpg")

## Training video noiseprint extractors
- 앞서 설명한 네트워크는 125개의 다른 카메라 데이터를 포함한 큰 데이터셋으로 훈련이 진행되었다.
  - 많은 카메라로 훈련한 네트워크는 훈련때 사용을 하지 않은 카메라로부터 취득한 이미지에서도 꽤 효과적이였다.
- 하지만 비디오는 이미지와 상당히 다르다.
  - 이미지로 훈련한 네트워크는 비디오에서는 잘 동작하지 않았다.
- 이 네트워크를 알맞은 비디오 데이터셋으로 다시 훈련시켜야 한다.
  - 하지만 `noiseprint training`에 알맞는 데이터는 한정적이며 여기서는 `VISION` 이라고 하는 데이터 셋을 사용하였다.
  - 28개 모델의 35개 디바이스(?)에서 촬영된 데이터이고 평균적으로 각 디바이스당 18개의 영상이 있다.
  - 영상은 26초에서 92초 사이의 길이고 모든 비디오는 `H.264` 포맷으로 압축되어있다.
- `Noiseprint extractor`를 훈련시키기 위해 두 가지 방식을 고려한다.
  - 비디오의 `I-frame` 만 사용
    - `I-frame`은 다른 이미지를 참조하지 않고 독립적으로 해독이 가능한 독립형 프레임
    - 비트 스트림이 손상된 경우 재동기화 지점의 시작점
    - 무작위 재생, 되감기, 빨리감기 구현에 필요
    - 많은 비트 수 사용
  - 비디오의 `I-frame` 및 `P-frame`
    - `P-frame`은 프레임 간 예측을 의미, 이전의 i 및 p frame을 참조하여 프레임을 부호화
    - `I-frame`보다 적은 비트를 요구하지만, 이전의 I, P 프레임에 대한 복잡한 의존성으로 인해 전송 오류에 민감하다.
- 훈련 단계에서 모든 프레임은 독립적인 이미지로 간주된다. `CNN-based noiseprint`와 같은 프로토콜에 따라 미니배치를 생성한다.
  - 여기서는 `48x48`이 아닌 `64x64`이미지를 사용한다.
    - 소스의 낮은 품질에 대한 보상
    - `Image noiseprint`와 비교할 때 더 낮은 다양성과 더 적은 데이터로 훈련한다.
    - 전체 성능에 영향을 끼칠 수 있다.
- 비디오를 다룰 때에는 이미지와 관련이 없는 여러 문제가 발생한다.
  - 대부분의 비디오는 자동 안정화를 이용해 원하지 않는 사용자의 움직임을 보정한다. (손떨림방지 같은?)
  - 이런 처리를 거친 비디오는 `shift`되거나 `rotation`이 되어 `resudial`(논문에서는 `model-related artifact`)의 공간적 오정렬(`Spatial misalignment`)를 야기할 수 있다.
- 이 문제를 해결하기 위해 훈련을 3개의 세팅으로 나눴다.
  - a) `I-frmae`만 사용, 안정화 되지 않은 비디오만 사용 (자동 안정화를 사용하지 않음)
    - 훈련에 12개의 카메라, 테스트에 6개의 카메라
  - b) `I-frame`만 사용, 모든 비디오 사용
  - c) 모든 비디오의 모든 프레임을 사용
- `NVIDIA Tesla P100`에서 `1280x740`의 `noiseprint`를 추출하는데 0.5초가 소요되었다.

## Experimental Analysis
- 비디오 `noiseprint`를 2가지 주요 포렌식 태스크에 대해 사용
  - `source identification`, `forgery manipulation detection`

#### Source identification
![figure_4]("./images/figure_4.jpg")
- 이 모델로 비디오가 2개의 비디오가 같은 카메라 모델로부터 왔는지 아닌지를 확인한다.
- `PRNU`에 사용된 표준 파이프 라인을 따른다.
  - 카메라 모델에 대해 일정 수의 프레임을 평균화 하여 `Reference fingerprint`(`averaged noiseprint`)를 추출한다.
  - 테스트 중인 비디오에서 `fingerprint (noiseprint)`를 추출해 `Reference fingerprint`와 비교한다.
- 두  `Noiseprint`간의 `similarity`를 비교하기 위해 `NCC`-(`Normalized Correlation Coefficient`), `MSE`-(`Mean Square Error`)를 사용한다.
  - 고전적인 PRNU 기반 절차를 기준으로 사용하였다.
    - `wavelet based denoiser`를 이용하였다.
    - `maximum likelihood`를 사용하였다. (이해가 어렵다.)
  - `VISION`과 전혀 관련이 없는 `Socrates Dataset`사용, 6개 장치 각 장치 당 10개의 비디오
    - `Set 1`은 안정화되지 않은 비디오
    - `Set 2`는 안정화 처리된 비디오


![auc_result]('auc_result.jpg')

- `Set 1 / 모든 프레임 사용`에서는 `PRNU-based` 방법이 가장 높은 성능을 달성했다. (0.907)
- 하지만 다른 경우는 (`Set 1 / I-frame`)에서는 성능이 떨어졌다. (0.810)
- `noiseprint` 기반 방법의 결과 중 가장 좋은 값은 0.832이고, `I-frame`만 사용한 경우이다.
  - 모든 경우에서 안정화 처리된 비디오에서는 모든 지표가 하락하였다.
  - 그러나 `PRNU based`방법은 더욱 급격하게 성능이 감소하였고 (0.64), `noiseprint`기반 방법은 성능 감소의 폭이 훨씬 낮았다.

#### Forgery detection and localization
- 이 실험에서는 조작 부위를 찾기 위해 `noiseprint`를 사용한다.
- `Scenario 1: Reference pattern`
  - `Source identification`에 대한 파이프라인과 꽤 유사하지만, 128 x 128 패치에서 슬라이딩 윈도우의 형태로 작동한다.
  - 동시발생에 기반한 특징 계산은 최종적인 히트맵을 얻는데 사용된다.
  - `Referrence pattern`은 50프레임으로 평가된다.
  - `Video noiseprintor`를 훈련시키기 위해 모든 비디오의 모든 프레임을 사용한다.
  - `noiseprint`는 20개의 연속적인 프레임의 결과를 평균내서 사용한다.
  - 이 결과를 `PRNU based reference`와 비교한다.
    - `Noiseprint based` 방법보다 훨씬 성능이 안좋다.

![figure_5]("./images/figure_5.jpg")

- `Scenario 2: Region of interest`
  - `Face2Face`, `DeepFake`와 같이 Advanced한 manipulation이 적용된 데이터에 대해 이 시나리오를 고려
  - `FaceForensic++` 데이터 셋 사용
     - 각각 140개의 `Face2Face`, `FaceSwap`, `DeepFake`, 깨끗한 비디오
  - 연속적인 프레임에 대해 `Noiseprintor`를 이용해 `Reference fingerprint`를 얻는다. (평균 이용)
    - FaceDetector를 이용해 얼굴 영역을 자른다.
    - 얼굴 영역과 배경 영역을 분리하고 각각 특징 추출기에 넣는다.
    - 배경 영역의 특징에 대해 통계적 컴퓨팅(?)을 진행하고 얼굴 영역 특징과의 `mahalanobis distance`를 구한다.
      - 프레임 수가 증가할수록 뛰어난 결과
      -  `mahalanobis distance`는 평균과의 거리가 표준편차의 몇배인지 나타내는 지표
  - 평균적인 정확도는 92.14%
    - CNN 기반 방법은 99.41%였으나, 이 네트워크는 훈련 단계에서 조작된 비디오를 사용하지 않았고, 조작되지 않은 비디오도 사용하지 않았다.
    - 가장 우수한 결과는 `설정 c)`에서 `FaceSwap`이였다. `92.14%`
    - 가장 나쁜 결과는 `Face2Face`, `82.14%`
![figure_6]("./images/figure_6.jpg")
![figure_7]("./images/figure_7.jpg")
![figure_8]("./images/figure_8.jpg")
![table_2]("./images/table2.jpg")
- `Scenario 3: Completely blind`
  - 어떠한 사전 정보가 없는 블라인드 시나리오에서 얻은 일부 결과를 보인다.
  - `noiseprint-based` 방법이 전체 프레임에 대해 작동하며 `heatmap`은 원 논문에서 설명된 것과 동일하게 추출된다.
  - 비디오의 각각의 프레임에 `image based noiseprintor`를 적용해도 만족스러운 결과를 얻지 못한다.
  - 비디오 지향 방법이 더 나은 결과를 제공한다.

## Conclusion

- `Noiseprint` 방식을 비디오로 확장한 첫번째 시도.
- 다양한 시나리오를 고려하여, `source identification`, `forgery localization`에 대한 실험 진행
- 모델은 깨끗한 비디오로만 훈련되었고, 테스트 셋에 속하는 데이터에 대해서는 전혀 튜닝되지 않았다.
