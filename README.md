# Object_Detection_YOLO
Object_Detection_YOLO

# You Only Look Once : Unifed, Real-Time Object Detection

R-CNN, Fast R-CNN, Faster R-CNN이 등장하고 Object Detection이 활발하게 진행되고 있었습니다.

하지만, R-CNN에 진화 버전들은 그 시절에는 빠른 편에 속하지만 FPS상 0.5~7 정도밖에 되지 않았습니다.

이렇기 때문에 실제로 적용하기에는 쉽지 않았는데, YOLO는 이러한 속도 문제를 해결하기 위해 제안되었습니다.

## R-CNN

R-CNN은 Object가 있을 만한 RoI(Region of Interest)를 2000개 정도 뽑아냅니다.

그리고 RoI 후보군을 CNN 네트워크로 분류하여 Bounding Box를 찾습니다.

이러한 과정 속에서 RoI 후보군을 찾을 때 Seletive Search를 사용하였고 이 과정이 매우 느린 편에 속하였습니다.

그래서 EdgeBox, Grid 방식 등이 제안되었습니다.

## Grid 방식

YOLO는 더욱 효과적으로 RoI 후보군을 찾기 위해서 Grid 방식을 채택했습니다.

먼저, 그림을 SxS 크기로 나눕니다. 논문에서는 7X7 크기로 분할하였습니다.

Grid가 나뉘었으면 각 Grid Cell 마다 Object를 감지하고 Bounding Box와 해당 Bounding Box의 신뢰도 점수(Confidence Socre)를 예측합니다.

신뢰도 점수는 딥러닝 모델이 예측한 Bounding Box에 Object가 포함되어있다는 확신을 의미합니다.

신뢰도 점수는 다음과 같이 정의합니다.

![eq1](https://latex.codecogs.com/gif.latex?%5Ctextup%7BPr%7D%28Object%29%20*%20%5Ctextup%7BIoU%7D%5E%7Btruth%7D_%7Bpred%7D)

해당 Cell에서 Object가 없다고 판단되면 신뢰도 점수는 0이 됩니다.

각 Cell에서는 분류하는 클래스 확률을 구하는데 이는 Pr(Classi|Object)를 예측합니다. Bounding Box와는 무관하게 셀당 클래스 확률들을 예측합니다.

그리고 신뢰도 점수와 클래스 확률을 곱하여서 각 상자에 대한 클래스 별 신뢰도 점수를 계산합니다. 이러한 점수는 해당 클래스가 상자에 나타날 확률과 예측 상자가 Object에 얼마나 잘 맞는지 모두를 보여줍니다.

## YOLO의 네트워크

![img1](https://github.com/kjo26619/Object_Detection_YOLO/blob/main/YOLO.PNG)
( 출처 : https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Redmon_You_Only_Look_CVPR_2016_paper.pdf )

YOLO는 예측한 Bounding Box의 x,y,w,h와 class별 확률 그리고 Object가 있을 확률을 가지고 네트워크에 학습을 합니다.

네트워크는 GoogleNet에서 착안했으며 24개의 Convolutional Layer가 있고 2개의 Fully Connected layer로 구성되어 있습니다.

YOLO 네트워크의 최종 Output은 SxSx30 으로 나오게 됩니다.

30은 No. Bbox * 5 + No. Class 의 식에서 20개의 클래스와 2개의 Bounding Box를 선택했을 때 나오게 됩니다.

## YOLO의 Loss Function

YOLO의 출력에서는 Bounding Box의 width 및 height를 이미지 width와 height로 정규화하여 0과 1이 되도록 합니다.

그리고 x 및 y 좌표를 Grid 셀 위치를 오프셋으로 0과 1 사이에 바운드되도록 합니다.

그리고 Sum-Squared Error를 최적화합니다.

Sum-Squared Error가 최적화하기 쉽지만, 평균 Precision을 최대화하지 않을 수 있습니다. 이 때문에 가중치를 사용합니다.

그리고, 이미지의 많은 Grid Cell에 Object가 포함되지 않을 수 있습니다.

이럴 경우 신뢰도 점수가 0이 되며, Object를 포함한 셀의 Gradient가 강해지는 경향을 갖습니다.

이들을 해결하기 위해 Bounding Box 좌표 예측의 손실을 늘리고 개체를 포함하지 않는 Box에 대한 신뢰 예측의 손실을 줄입니다.

이를 각각 ![lambda1](https://latex.codecogs.com/gif.latex?%5Clambda_%7Bcoord%7D)과 ![lambda2](https://latex.codecogs.com/gif.latex?%5Clambda_%7Bnoobj%7D) 로 나타내며 각각 5와 .5로 설정했습니다.

![img2](https://github.com/kjo26619/Object_Detection_YOLO/blob/main/YOLO_Loss.PNG)

YOLO에서 트레이닝을 위해 사용하는 Loss Function은 다음과 같습니다.

길지만 큰 문제 없이 각 x, y, w, h, C에 대한 Sum Squared Loss입니다. 

여기서, ![obj1](https://latex.codecogs.com/gif.latex?1%5E%7Bobj%7D_i)는 Cell i에서 Object가 나타나는지 여부입니다.

![obj2](https://latex.codecogs.com/gif.latex?1%5E%7Bobj%7D_%7Bij%7D)는 Cell i에서 j번째 Bounding Box에 대한 책임이라고 나와있는데,

이는 Cell i의 여러 개의 Bounding Box 중 Ground Truth Bounding Box와 가장 높은 IoU를 갖는 Bounding Box가 1을 갖게 되는 것입니다.

즉, 하나의 Bounding Box만 적용이 되는 것입니다.

![obj3](https://latex.codecogs.com/gif.latex?1%5E%7Bnoobj%7D_%7Bij%7D) 는 Object를 찾지 못했을 경우의 책임입니다.

# YOLO의 성능

YOLO는 Titan X GPU를 사용하여 Batch 없이 진행 했을 경우 45FPS가 나왔고, Fast Version은 150 FPS가 나왔다고 합니다.

YOLO의 등장으로 스트리밍 비디오를 처리할 수 있게 되었다고 봐도 무방할 정도입니다.

Fast YOLO는 적은 수의 Convolution Layers를 사용하여 24개 대신 9개만 사용합니다. 그 외에 파라미터는 모두 동일합니다.


