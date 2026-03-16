Basic TranSIESTA calculation
=========================
## Contents
1. TranSIESTA 개요

## TranSIESTA 개요

SIESTA 내 구현된 TranSIESTA는 density functional theory (DFT)과 non-equilbrium Green function (NEGF) 방법론을 결합한 DFT-NEGF 방법론 기반으로 양자수송 특성 계산할 수 있는 프로그램이다.[1](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.65.165401) DFT-NEGF의 알고리즘을 먼저 살펴보자.

<center><img src="../../transiesta/img/overview-01.png" width="60%" height="60%"></center>

TranSIESTA는 위의 DFT-NEGF 알고리즘에 따라서 구현된 코드로 이를 통해 양자수송 특성을 계산하는 과정을 다음과 같다.

<center><img src="../../transiesta/img/overview-02.png" width="80%" height="80%"></center>

가장 먼저 **junction modeling** 과정은 양자수송 특성 계산의 대상인 junction을 모델링하는 과정이다. 일반적으로 주기적 경계 조건을 가지는 DFT 계산의 모델과 다르게 junction 모델을 몇가지 특징을 가진다. 이에 대해서는 바로 다음장에서 알아본다.  
그 다음로 **electrode calculation** 과정은 DFT-NEGF 계산에서 전극에 해당하는 self-energy를 계산하기 위한 헤밀토니안을 얻는 과정이다. 이때 과정에서는 주기적 경계조건을 가진 벌크 구조의 전극에 대해 평형 상태의 DFT 전자구조 계산 필요한다. 이는 SIESTA 프로그램을 통해 진행한다.  
그 다음으로는 **scattering region calculation**이다. 이 과정에서는 이전 **electrode calculation**에서 얻은 전극의 헤밀토니안 정보를 가지고 self-energy를 계산하고 이를 통해서 NEGF 계산을 수행하는 단계이다. 비평형 상태의 밀도행렬이 수렴할 때까지 반복적인 계산을 수행하여 비평형 상태의 전자구조 특성을 얻을 수 있다. 이에 대한 계산은 TranSIESTA 프로그램으로 진행된다.  
마지막 과정인 **post-processing** 과정에서는 이전 단계에서 얻은 수렴된 Green's function 등의 정보를 기반으로 전류 특성과 같은 양자수송 특성을 얻는 과정이다. 이에 대한 계산은 Tbtrans 유틸리티 코드로 수행할 수 있다.


### 참고문헌
[[1]](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.65.165401): Mads Brandbyge, José-Luis Mozos, Pablo Ordejón, Jeremy Taylor, and Kurt Stokbro, Phys. Rev. B 65, 165401 (2002)