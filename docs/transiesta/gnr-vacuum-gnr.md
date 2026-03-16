GNRs-vacuum-GNRs junction model
=========================
## Contents
1. Junction 모델링
2. 수렴성 테스트
3. 비평형 상태 계산

> Input: [Link](https://1drv.ms/u/s!ArHICXVqPANVuyL_-aoSshJh_vwO?e=guzzZ7)

그동안은 matrix Green's function을 기반으로 평형상태의 양자수송 특성을 계산하는 법에 대해서 다루었었다. 그러나 전압이 인가된 비평형 상황에서의 계산에서는 non-equilibrium Green's function (NEGF)의 도입이 필요하다. 이번 장에서는 graphene nanoribbons (GNRs)-vacuum-GNRs로 이루어진 비균질 모델에 대해서 junction을 정의하고 보다 정확한 계산을 위해서 수렴성 테스트를 진행하는 법을 소개하고 마지막으로 비평형 상태의 양자수송 특성을 계산하는 방법에 대해서 다룬다.

## Junction 모델링

<center><img src="../img/gnr-vacuum-gnr-01.png" width="80%" height="80%"></center>

우리가 다룰 GNRs-vacuum-GNRs 구조는 양쪽으로 반-무한히 이어진 GRNs 전극 영역과 그 사이에 vacuum 영역으로 구분할 수 있다. 계산을 엄밀히 진행하기 위해서는 이전 **Junction modeling** 강의에서 언급했듯이 scattering 영역과 electrode 영역을 구분해야한다. 이를 위해서 다음과 같은 순서로 electrode/scattering 영역을 결정하여 junction 모델을 만들어보자.  

**Step 1:** Electrode의 principal cell 결정  
**Step 2:** Buffer layer 테스트를 통한 scattering/electrode 영역 결정  


### Step 1: Electrode의 principal cell 결정

<center><img src="../img/gnr-vacuum-gnr-02.png" width="80%" height="80%"></center>

DFT-NEGF 방법에서 필요한 전극의 self-energy를 구하기 위해서는 위와 같이 수송 방향으로 주기성을 가지는 전극의 벌크 구조에 대한 계산이 필요하다. 위 예시에서는 1, 2와 3 개의 단위격자로 구성된 전극 구조가 나열되어있다. 여기서 중요한 것은 우리가 설정한 전극은 완벽한 principal cell의 조건을 만족해야한다는 점이다. 이를 확인하기 위해서 각 구조에 대해서 electrode 계산을 진행해본다.

```bash
cd 1.modeling/1.principal_cell_test/1-unit
sbatch slm_siesta_run
cd ../2-unit
sbatch slm_siesta_run
cd ../3-unit
sbatch slm_siesta_run
```

주의할 점은 전극에 대한 계산은 DFT 계산이므로 `RUN.fdf` 파일에 `SolutionMethod`가 `diagon`인 것을 꼭 확인하자. 또한 TranSIESTA 계산을 위한 `.TSHS`을 얻기 위해 SIESTA의 옵션을 키거나 TranSIESTA 실행 파일을 통해 계산을 진행한다.

SIESTA 프로그램은 반경이 정해진 수치적 원자 오비탈(numerical atomic orbital)을 기저함수로 하는 DFT 프로그램이다.  SIESTA는 주기적인 고체 시스템을 다룰 때 이미지 단위격자 간의 상호작용을 고려하기 위해 주기성의 방향으로 단위 격자가 영향을 미치는 최대한의 이미지 격자까지 supercell을 늘려준다. 이에 대한 정보가 `superc: Internal auxiliary supercell:` 키워드로 나타나있다. 계산이 완료되면 다음과 같은 명령어를 통해 각 계산에서 생성된 supercell 정보를 확인해본다.

```bash
cd ..
grep 'superc: Internal auxiliary supercell:' */stdout.txt

1-unit/stdout.txt:superc: Internal auxiliary supercell:     1 x     1 x     7  =       7
1-unit/stdout.txt:superc: Internal auxiliary supercell:     1 x     1 x     7  =       7
2-unit/stdout.txt:superc: Internal auxiliary supercell:     1 x     1 x     5  =       5
2-unit/stdout.txt:superc: Internal auxiliary supercell:     1 x     1 x     5  =       5
3-unit/stdout.txt:superc: Internal auxiliary supercell:     1 x     1 x     3  =       3
3-unit/stdout.txt:superc: Internal auxiliary supercell:     1 x     1 x     3  =       3
```

위 결과를 확인해보면 1개의 단위격자로 계산하는 경우 `1 x 1 x 7`의 이미지 격자가 생성되는 것을 확인할 수 있다. 이는 단위격자가 수송 방향의 양옆으로 최소 3개의 단위격자만큼의 거리로 상호작용하고 있다는 뜻이다. 따라서 이는 self-energy를 계산하는 principal cell의 조건을 만족하지 않는다. 그러나 전극의 크기를 늘려서 3개의 단위격자를 사용할 때는 `1 x 1 x 3`의 이미지 격자가 생성되어 인접한 단위격자와만 상호작용 하고 있다는 것을 알 수 있다. 따라서 electrode 영역을 설정할 때 최소 3개의 단위격자를 이용한 전극 구조를 사용하는 것이 좋다.

### Step 2: Buffer layer 테스트를 통한 scattering/electrode 영역 결정

<center><img src="../img/gnr-vacuum-gnr-03.png" width="80%" height="80%"></center>

그 다음으로는 scattering 영역과 electrode 영역을 구분하기 위해 어느 정도의 buffer layer가 필요한지 테스트해본다. Electrode 영역의 길이는 이전 과정에서 구한 3개의 단위격자 크기로 고정하고 아래와 같이 다른 scattering 영역의 길이를 가진 여러 모델에 대해 평형상태의 transmission 값의 차이를 살펴볼 것이다. 

- 5개 단위격자 (한쪽 방향의 총 GNRs 격자 개수)

<center><img src="../img/gnr-vacuum-gnr-04.png" width="60%" height="60%"></center>

- 6개 단위격자

<center><img src="../img/gnr-vacuum-gnr-05.png" width="60%" height="60%"></center>

- 7개 단위격자

<center><img src="../img/gnr-vacuum-gnr-06.png" width="60%" height="60%"></center>


각 구조에 해당하는 디렉토리에 들어가서 이전 전극 계산에서 구한 `electrode.TSHS` 파일을 `input` 디렉토리에 복사한 후에 TranSIESTA 계산을 진행해준다. 

**TranSIESTA calculation (scattering region calculation)**
```bash
cd ../2.buffer_layer_test/5-unit
cp ../../1.principal_cell_test/3-unit/OUT/electrode.TSHS input/.
sbatch slm_siesta_run

cd ../6-unit
sbatch slm_siesta_run
cp ../../1.principal_cell_test/3-unit/OUT/electrode.TSHS input/.

cd ../7-unit
sbatch slm_siesta_run
cp ../../1.principal_cell_test/3-unit/OUT/electrode.TSHS input/.
```

계산이 완료되면 TBtrans 계산을 진행하여 transmission 결과를 얻는다.

**TBtrans calculation (scattering region calculation)**

```bash
cd ../2.buffer_layer_test/5-unit
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt

cd ../6-unit
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt

cd ../7-unit
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt
```

### Results: transmission

다음 명령어를 통해서 transmission을 그려보자.

```bash
cd ..
python show_trans.py  5-unit/OUT_tbt/scattering.TBT.AVTRANS_Left-Right 6-unit/OUT_tbt/scattering.TBT.AVTRANS_Left-Right 7-unit/OUT_tbt/scattering.TBT.AVTRANS_Left-Right
```

<center><img src="../img/gnr-vacuum-gnr-07.png" width="70%" height="70%"></center>

위 결과를 통해 모든 구조의 transmission이 일치하는 것을 확인할 수 있다. 따라서 "5개 단위격자" 길이의 junction 모델을 이용하는 것이 계산량 측면에서 효율적이다.


## 수렴성 테스트

위 과정을 통해서 junction 모델을 확정했다면 이제부터 본격적인 DFT-NEGF 계산을 진행할 준비가 되었다. 일반적인 DFT 계산과 마찬가지로 DFT-NEGF 계산도 계산의 수렴성을 테스트하는 것이 중요하다.

1. Electrode 계산에 대한 k-point 수렴성 테스트
2. Scattering region 계산에 대한 k-point 수렴성 테스트
3. Post-processing 단계에서의 k-point 수렴성 테스트

우선 첫번째 단계는 SIESTA을 통한 electrode 계산에서 k-point sampling의 수렴성을 테스트하는 과정이다. 이는 일반적인 DFT 계산과 동일하다. 전극구조는 벌크 구조이기 때문에 주기성에 따라서 아래와 같이 (A1), (A2), (A3)에 대해 모두 K-point의 수렴을 확인해야한다. 만약 전자의 수송 방향이 (A3)이라면 self-energy를 계산하기 위해서 이 방향의 k-point는 특히 더 많이 줄 필요가 있다.

- Electrode calculation
```
%block kgrid_Monkhorst_Pack
(A1) 0    0    0.0
0    (A2) 0    0.0
0    0    (A3) 0.0
%endblock kgrid_Monkhorst_Pack
```

그 다음으로 TranSIESTA를 통한 scattering region의 계산에 대해서도 k-point sampling의 수렴성을 테스트해야한다. Scattering region 계산의 k-point는 수송 방향의 k-point (A3)를 제외하고 반드시 electrode 계산의 k-point (A1, A2)와 동일하게 설정해야한다. 즉, 이 단계에서는 electrode 계산에서 수렴했던 k-point가 scattering region 계산에서도 수렴하는지 확인하는 과정이다. 이를 확인하는 방법은 k-point에 따라 transmission의 수렴성을 확인하거나 비형평 상태에서는 electrostatic potential 등을 확인해보자.

- Scattering region calculation
```
%block kgrid_Monkhorst_Pack
(A1) 0    0    0.0
0    (A2) 0    0.0
0    0    1    0.0
%endblock kgrid_Monkhorst_Pack
```

마지막으로는 TBtrans를 통한 post-processing 단계에서의 k-point 수렴성 테스트이다. 이과정에서는 이전의 과정에서 구한 k-point와 다른 값을 줄 수 있다. 일반적으로 post-processing에서 사용되는 k-point는 scattering/electrode 계산에서 이용된 k-point보다 많은 값을 이용한다. 이를 통해 transmission과 같은 전자구조에 대해 TBtrans의 k-point을 많이주어 더 높은 resolution의 결과를 얻을 수 있다.

- Post-processing calculation
```
%block kgrid_Monkhorst_Pack
(B1) 0    0    0.0
0    (B2) 0    0.0
0    0    1    0.0
%endblock kgrid_Monkhorst_Pack
```

우선 평형상태의 계산을 통해서 위 과정을 따라해보자. 참고로 GNRs-vacuum-GNRs 모델의 경우에는 1차원 모델이기 때문에 electrode 영역 계산 시에 사용되는 전자의 수송 방향(z축)으로 주어지는 k-point만 고려하면 된다. 따라서 z축 방향으로 여러 k-point을 바꾸어보며 electrode 계산을 진행해보자.

```bash
cd ../../2.convergence_test/1.electrode/3-unit_k5
sbatch slm_siesta_run

cd ../3-unit_k10
sbatch slm_siesta_run

cd ../3-unit_k15
sbatch slm_siesta_run

cd ../3-unit_k20
sbatch slm_siesta_run

cd ../3-unit_k30
sbatch slm_siesta_run
```

계산이 완료되면 다음과 같은 명령어로 각 계산에 대한 총 에너지를 확인할 수 있다.

```bash
grep -r 'Total =' */stdout.txt
3-unit_k5/stdout.txt:siesta:         Total =   -3989.050533
3-unit_k10/stdout.txt:siesta:         Total =   -3989.056338
3-unit_k15/stdout.txt:siesta:         Total =   -3989.055853
3-unit_k20/stdout.txt:siesta:         Total =   -3989.055784
3-unit_k30/stdout.txt:siesta:         Total =   -3989.055976
```

위와 같이 `k=10` 이후로부터 총 에너지가 0.001 eV 이하 수준으로 수렴한 것을 확인할 수 있다. 이러한 k-point들에 대해서 scattering 계산에서 수렴성을 확인해보자.

```bash
cd ../../2.scattering/5-unit_k5
cp ../../1.electrode/3-unit_k5/OUT/electrode.TSHS input/.
sbatch slm_siesta_run

cd ../5-unit_k10
cp ../../1.electrode/3-unit_k10/OUT/electrode.TSHS input/.
sbatch slm_siesta_run

cd ../../2.scattering/5-unit_k15
cp ../../1.electrode/3-unit_k15/OUT/electrode.TSHS input/.
sbatch slm_siesta_run

cd ../../2.scattering/5-unit_k20
cp ../../1.electrode/3-unit_k20/OUT/electrode.TSHS input/.
sbatch slm_siesta_run

cd ../../2.scattering/5-unit_k30
cp ../../1.electrode/3-unit_k30/OUT/electrode.TSHS input/.
sbatch slm_siesta_run
```

계산이 완료되면 TBtrans 계산을 통해 양자수송 특성 결과들을 얻는다.

```bash
cd ../5-unit_k5
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt

cd ../5-unit_k10
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt

cd ../5-unit_k15
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt

cd ../5-unit_k20
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt

cd ../5-unit_k30
cp OUT/scattering.TSHS input/.
sbatch slm_siesta_run_tbt
```

다음 명령어를 통해서 각 계산에 대한 tranmission 값들을 비교해보자.

```bash
cd ..
python show_trans.py  5-unit/OUT_tbt/scattering.TBT.AVTRANS_Left-Right 6-unit/OUT_tbt/scattering.TBT.AVTRANS_Left-Right 7-unit/OUT_tbt/scattering.TBT.AVTRANS_Left-Right
```

<center><img src="../img/gnr-vacuum-gnr-08.png" width="70%" height="70%"></center>

위 결과를 확인해보면 `k=15`로 계산한 전극의 self-energy를 사용한 때부터 transmission이 일치하는 것을 확인할 수 있다. 이를 통해서 scattering/electrode 계산에서의 수렴된 k-point의 값을 얻을 수 있다.


## 비평형 상태 계산

지금까지 junction을 모델링하는 과정이나 수렴성 테스트하는 계산들은 모두 평형 상태 계산을 기반으로 이루어졌다. 그러나 엄밀하게는 비평형 상태의 계산에서는 평형 상태의 계산에서 얻은 junction 구조나 수렴된 계산 파라미터가 적절한지 장담할 수 없다. 그 중에 가장 문제가 되는 것이 바로 junction 구조를 결정하는 것이다.

다음 과정을 통해서 평형상태에서 얻은 junction 구조와 계산 파라미터에 대해 0.5V의 전압이 인가된 계산을 진행해보자. 이에 대해서는 이미 수렴성 테스트를 진행하며 평형 상태에 대한 계산은 마무리했으니 이를 이용할 것이다.


```bash
cd ../3.non_equilibrium/0.5V
cp ../../2.scattering/5-unit_k15/input/electrode.TSHS input/.
cp ../../2.scattering/5-unit_k15/OUT/scattering.TSDE input/.
```

이때 평형상태의 scattering 계산과 마찬가지로 전극의 `electrode.TSHS` 파일을 `input` 디렉토리에 넣고, 추가적으로 평형상태의 계산 결과인 `scattering.TSDE` 파일을 `input` 디렉토리에 넣어주었다. 일반적으로 평형상태에 대한 `.TSDE` 파일은 비평형 상태에 대한 NEGF 계산에서 좋은 초기 조건이 되어주기 때문에 계산을 단축하는데 도움을 준다.

이제 전압을 인가하기 위해 `TS.fdf` 파일을 수정해주면 된다.

- TS.fdf
```
TS.Voltage    0.500 eV
```
이후로 TranSIESTA 계산을 진행준다.

```bash
sbatch slm_siesta_run
```

비평형상태에 대한 TranSIESTA 계산이 진행되고 나면 DFT에서 얻을 수 있는 전자기 포텐셜이나 전자밀도와 같은 기본적인 전자구조 특성들을 얻을 수 있다. 그중에서 전압이 강하되는 특성을 보기 위해 평형 상태와 비평형 상태의 전자기 포텐셜의 차이를 확인해보자.

```bash
cd ..
python show_vh.py 0.5V/OUT/ ../../2.scattering/5-unit_k15/OUT
```
<center><img src="../img/gnr-vacuum-gnr-09.png" width="50%" height="50%"></center>

위 계산 결과를 살펴보면 electrode 영역을 포함하여 모든 junction 영역에서 전압의 강하가 일어나고 있음을 확인할 수 있다. 이전 **junction modeling** 강의에서 언급했듯이 올바른 junction model에서는 screening으로 인한 전압 강하는 항상 scattering 영역에서 일어나야한다. 이는 고려하고 있는 junction 모델이 전압이 인가 시에 충분히 screening할 만큼 길지 않다는 뜻이다.

위 결과를 통해 우리는 junction을 모델링할 때 평형 상태의 transmission의 결과만으로 적절한 buffer layer의 길이를 판별할 수 없다는 사실을 알 수 있다. 따라서 전압이 인가된 상황에서는 위 과정들, **junction modeling**, **수렴성 테스트**이 항상 비평형 상태의 계산을 동반하여 진행해야 한다는 점을 몀심하자. 제대로된 계산 결과를 얻기 위해 더 긴 buffer layer를 가지는 모델에 대해서 계산을 진행해야한다.

**Exercise:** Junction 모델에 포텐셜이 완벽히 걸리는 순간까지 buffer layer를 늘리며 DFT-NEGF 계산을 진행해보자.