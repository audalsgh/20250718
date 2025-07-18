# 20일차

## YOLOv11, YOLOv12 정리
[Ultralytics YOLOv12 공식 문서](https://docs.ultralytics.com/ko/models/yolo12/)<br>
YOLO 아키텍처 구조 : (백본 -> NECK -> HEAD) 순서<br>
특징 추출 (Feature Extraction) -> 다중 스케일 특징 통합 (Feature Fusion) -> 예측 (Detection / Classification / Segmentation 등) 순서<br>

<br>
YOLO의 Backbone은 이미지에서 기본 특징(edges, textures 등)을 추출하는 앞단.<br>
YOLO의 Neck은 다양한 스케일의 특징 맵(feature maps)을 융합하여 작은/중간/큰 객체를 모두 감지할 수 있게 하는 중간단.<br>
YOLO의 Head는 객체의 위치, 클래스, 크기 등을 최종 예측하는 마지막 단.<br>
<br>

**YOLOv11이 YOLOv8을 개선한 차세대 모델이였다면, YOLOv12는 구조를 처음부터 새로 설계하여 기존의 YOLO계열과는 다르다.**

YOLOv8:<br> 
  [ C2f → PAN → ConvHead ]<br>
YOLOv11:<br>
  [ RepNCSPELAN4C → CBLinearNeck → DConvHead ]<br>
YOLOv12:<br>
  [ RepVGGBlock기반 SwiftNet → SwiftNeck → DRepHead ]

**YOLOv12**
- 가장 최신 모델로, 정확도와 속도가 제일 우수하다.
- 백본 구조 : RepNCSPELAN4C -> RepVGGBlock기반 SwiftNet (효율적인 2D residual 구조 + MobileNet-style 경량화)
- Neck 구조	: CBLinearNeck ->	SwiftNeck (PAN이 아닌 1x1 + 3x3 블록 기반, 속도 최적화)
- Head 구조	DConvHead	-> Depthwise + RepConv의 하이브리드인 DRepHead (더 깊은 표현력)

**모델 아키텍처 용어 정리**
1. SwiftNet 백본<br>
같은 정확도 기준 모델 사이즈는 YOLOv11의 (3~70M)보다도 작아짐<br>
더 빠르고, 적은 파라미터로 높은 표현력 확보<br>
3. SwiftNeck neck<br>
layer수가 적어서 단순하고 얕은 구조<br>
PANet처럼 복잡한 비대칭 경로는 제거됨<br>
연산량(FLOPs) 감소<br>
실시간 Edge 디바이스에서 10~15% 속도 향상<br>
메모리 사용량 감소<br>
3. DRepHead Head<br>
파라미터 수와 메모리 사용을 모두 감소시킨 경령화 Head<br>
Depthwise Separable Conv = MobileNet 스타일의 연산량 최소화<br>
RepConv (Reparameterized Conv) =	학습 시 더 깊은 구조, 추론 시 병합하여 빠르게<br>
(Conv + BN + ReLU) → RepConv로 통합 = fused 모델 구성으로 추론 최적화<br>
다중 anchor-free head = 단일뿐만 아니라 multi-head 설계 가능<br>

### YOLOv12 결론
SwiftNet : 학습 시엔 여러 Conv + BN 레이어를 쌓아 깊지만, 추론 시엔 단일 Conv로 병합되도록 설계됨<br>
> Rep 구조 → 추론 시 단일 Conv로 단순화가 핵심 기술
 
SwiftNeck : 기존 PAN/FPN 구조보다 더 빠르고 단순하게 특징 융합.(얕다)<br>
> PAN 구조 제거 → 더 빠른 융합이 핵심 기술

DRepHead : 경량화된 head로 학습은 여러 Conv 레이러로 깊지만, 추론때엔 단일 Conv로 병합되었기에 빠름.(추론시 병합)<br>
> Depthwise Separable Convolution (DWConv) : 일반 Conv 대비 연산량(FLOPs)을 대폭 줄이는 핵심 기술, (Channel 간 독립적 연산을 수행하여 경량화 실현)
 
-> YOLOv12 전체는 이 두 구조로 인해 모바일 환경에서도 높은 정확도와 속도를 동시에 달성. 기존보다 연산량 20~30% 감소

<img width="818" height="620" alt="image" src="https://github.com/user-attachments/assets/2f2d2699-24f3-4846-8737-5e360f2d3357" />

| 모델 종류         | 설명                                    |
| ------------- | ------------------------------------- |
| `yolov12n.pt` | **Nano**: 가장 작고 빠름. 모바일 및 임베디드 환경에 최적 |
| `yolov12s.pt` | **Small**: 가볍고 정확도도 준수. 실시간 웹캠/드론에 적합 |
| `yolov12m.pt` | **Medium**: 속도와 정확도의 균형. 대부분 상황에서 추천  |
| `yolov12l.pt` | **Large**: 높은 정확도. 고해상도 객체 인식에 적합     |
| `yolov12x.pt` | **X-Large**: 최고 성능. 정확도 우선 환경에 적합     |

단점<br>
❌ YOLOv8 / YOLOv11과도 구조적으로 호환되지 않음<br>
→ SwiftNet, SwiftNeck, DRepHead 등 새로운 구조 기반이기 때문에 기존 .yaml이나 커스텀 코드 재활용이 어려움<br>
❌ 아직 생태계가 안정적이지 않음<br>
→ 학습 로그, 튜토리얼, 전환 툴 등의 자료가 YOLOv8/11보다 부족함<br>

## Roboflow에 가입하고, YOLOv11 코드를 얻도록 전이학습까지 해보자
https://www.youtube.com/watch?v=N8ZUm-26zyk<br>
https://www.youtube.com/watch?v=0lyPAGcrDLU

두 링크를 참고하거나, 교수님의 블로그를 참고하여 최소 500장 이상 라벨링하기
