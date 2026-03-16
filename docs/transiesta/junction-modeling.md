Junction modeling
=========================
## Contents
1. Junction 모델의 특징
2. Scattering/electrode 영역의 구분
3. Electrode principal cell

## Jucntion 모델의 특징

<center><img src="../img/junction-modeling-01.png" width="40%" height="40%"></center>

Density function theory (DFT)과 Non-equilibrium Green's function (NEGF) 방법론을 결합한 DFT-NEGF 방법론을 통한 양자수송 특성 계산에서는 전통적인 DFT 계산과 근본적으로 모사하고자 시스템의 "경계조건"에 차이가 있다. 위 그림과 같이 전통적인 DFT 계산에서 다루는 분자/고체 시스템은 고립되거나(closed) 주기적인(periodic) 경계 조건을 가지만, DFT-NEGF이 모사하고자 하는 junction 시스템은 양단이 무한한 전극으로 이어진 **열린 경계조건(open boundary condition)**을 가진다. 이러한 경계조건의 차이 때문에 DFT-NEGF이 다루는 junction model은 일반적인 DFT 계산에서의 계산 모델과 구별되는 특징을 가진다.

<center><img src="../img/junction-modeling-02.png" width="80%" height="80%"></center>

Junction 모델은 일반적으로 두 전극과 채널으로 구성되어 있기 때문에 세 가지 영역으로 나눌 수 있다. **Left electrode**, **right electrode** 구역과 **scattering** 영역이다. **Left electrode**와 **right electrode** 영역은 각각 한쪽 방향으로 무한히 반복되는 전극을 대표하며 DFT-NEGF 계산 시에는 self-energy로 치환되는 곳이다(위 그림 참조). **Scattering** 구역은 junction 모델에서 채널 물질이 위치하며 주기성을 가지지 않는 곳이며 전압이 인가된 비평형 상태에서는 전압이 강하되는 위치이기도 하다.

## Scattering/electrode 영역의 구분

<center><img src="../img/junction-modeling-03.png" width="30%" height="30%"></center>

위에서 기술했듯이 각 구역은 명확한 특징을 가지기 때문에 보다 정확한 DFT-NEGF 계산을 위해서는 엄밀한 **scattering/electrode** 영역의 구분이 매우 중요하다. 가장 근본적으로는 DFT-NEGF 방법론은 "Landauer formulation"을 통해 양자수송 특성을 기술하기 때문에 양쪽 **electrode** 영역 사이의 상호작용이 없게 **scattering** 영역을 설정해야한다.

<center><img src="../img/junction-modeling-04.png" width="80%" height="80%"></center>

영역을 구분할 때 주의할 다른 사항으로는 **electrode** 영역이 무한한 주기적인 조건을 가지는 전극 구조를 대표하여 self-energy로 치환된 영역임을 고려해야한다는 점이다. 위 사진과 같은 Si-SiO2-Si junction 모델에서 접합부에 위치한 Si 전극의 첫번째 줄의 경우 계면의 효과로 인해서 변형이 일어난 것을 확인할 수 있다. 이러한 접합부에서 일어나는 변형은 전극에 주기적으로 나타나는 특성이 아니기 때문에 이 영역을 **electrode** 영역에 포함시켜서는 안될 것이다. 이와 같이 전극 물질이지만 scattering 영역에 속하는 것을 buffer layer라고 한다.

<center><img src="../img/junction-modeling-05.png" width="50%" height="50%"></center>

마지막으로 전자구조의 특성으로도 **scattering**과 **electrode** 영역의 구분지을 수 있다. 양쪽 전극에 서로 다른 전압이 인가된 **비평형 상태**를 모사하는 상황에서 양쪽의 **electrode** 영역은 서로 다른 화학적 포텐셜(chemical potential)을 대표하기 때문에 일정한 포텐셜을 가지고 있으며 전압의 강하는 **scattering** 영역에서 일어난다. 이를 고려하여 전압 강하를 일으키는 screening이 항상 **scattering** 영역에 일어나도록 구역을 설정해야한다.

## Electrode principal cell

<center><img src="../img/junction-modeling-06.png" width="80%" height="80%"></center>

마지막으로 junction을 모델링할 시에 고려해야할 점은 self-energy로 치환되는 **electrode** 영역이 완벽한 principal cell 조건을 만족해야한다는 점이다. Self-energy 계산 시의 벌크 전극의 **electrode** 영역의 물질과 동일한 구조를 가지며 전자 수송 방향으로 무한한 주기성을 가진다. 이러한 전극의 self-energy를 계산 시에 가장 중요한 것은 전극의 단위 격자가 전자의 수송 방향으로 반드시 가장 가까운 인접 단위 격자와만 상호작용해야 한다는 점이다 (위 그림 참조). 이는 행렬로 표현시에 tridiagonal matrix로 나타낼 수 있다 이와 같은 조건을 만족하는 단위 격자를 principal cell이라 한다. 따라서 junction을 모델링할 시에 **electrode** 영역을 너무 짧게 설정하지 않아야하는 것을 명심해야한다.