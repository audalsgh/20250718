# 20일차

## YOLOv11, YOLOv12 정리
[Ultralytics YOLOv12 공식 문서](https://docs.ultralytics.com/ko/models/yolo12/)<br>
YOLO 아키텍처 구조 : (백본 -> NECK -> HEAD) 순서<br>
특징 추출 (Feature Extraction) -> 다중 스케일 특징 통합 (Feature Fusion) -> 예측 (Detection / Classification / Segmentation 등) 순서<br>
<br>
YOLO의 Neck은 다양한 스케일의 특징 맵(feature maps)을 융합하여 작은/중간/큰 객체를 모두 감지할 수 있게 하는 YOLO의 구조중 하나<br>
YOLO의 Head는 객체의 위치, 클래스, 크기 등을 최종 예측하는 단계<br>

**YOLOv11이 YOLOv8을 개선한 차세대 모델이였다면, YOLOv12는 구조를 처음부터 새로 설계하여 기존의 YOLO계열과는 다르다.**

YOLOv8:<br> 
[ C2f → PAN → ConvHead ]<br>
YOLOv11:<br>
[ RepNCSPELAN4C → CBLinearNeck → DConvHead ]<br>
YOLOv12:<br>
[ RepVGGBlock기반 SwiftNet → SwiftNeck → DRepHead ]

YOLOv12
- 가장 최신 모델로, 정확도와 속도가 제일 우수하다.
- 백본 구조 : RepNCSPELAN4C -> RepVGGBlock기반 SwiftNet (효율적인 2D residual 구조 + MobileNet-style 경량화)
- Neck 구조	: CBLinearNeck ->	SwiftNeck (PAN이 아닌 1x1 + 3x3 블록 기반, 속도 최적화)
- Head 구조	DConvHead	-> Depthwise + RepConv의 하이브리드인 DRepHead (더 깊은 표현력)

**모델 아키텍처 용어 정리**
1. SwiftNet 백본<br>
같은 정확도 기준 모델 사이즈는 YOLOv11의 (3~70M)보다도 작아짐<br>
더 빠르고, 적은 파라미터로 높은 표현력 확보<br>
3. SwiftNeck neck<br>
layer수가 적어서 단순하고 얕은 구조
PANet처럼 복잡한 비대칭 경로는 제거됨
연산량(FLOPs) 감소
실시간 Edge 디바이스에서 10~15% 속도 향상
메모리 사용량 감소
5. DRepHead Head

<img width="818" height="620" alt="image" src="https://github.com/user-attachments/assets/2f2d2699-24f3-4846-8737-5e360f2d3357" />

## Roboflow에 가입하고, YOLOv11 코드를 얻도록 전이학습까지 해보자
https://www.youtube.com/watch?v=N8ZUm-26zyk<br>
https://www.youtube.com/watch?v=0lyPAGcrDLU

두 링크를 참고하거나, 교수님의 블로그를 참고하여 최소 500장 이상 라벨링하기
