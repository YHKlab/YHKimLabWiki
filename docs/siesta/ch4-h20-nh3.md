분자 계산 실습 
============

## Contents

1. 주의사항
2. 분자 계산 개요 및 실습 목표
3. 공통 준비: 입력 파일 구성과 실행 방법
4. 분자 계산 실습 (목표별 과제)

---

## 1. 주의사항

이 문서는 "처음 리눅스 터미널과 SIESTA를 함께 써 보는 사람"을 기준으로 설명한다.  
명령어를 실행하기 전에 현재 위치를 한 번 확인하는 습관을 들이면 실수를 많이 줄일 수 있다.

```bash
$ pwd
$ ls
```

아래 예시는 대부분 `input/` 폴더가 있는 계산 디렉토리에서 실행한다고 가정한다.  
즉, `ls`를 했을 때 `input/`과 실행 스크립트가 보이는 위치에서 명령을 입력하면 된다.

### 1.1 `A. SIESTA Edison에서 설치 방식`으로 빌드한 경우

튜토리얼 압축 파일을 풀면 `bin/` 디렉토리에 실행 보조 스크립트가 들어 있다.

```bash
$ tar -zxvf Tutorial.tar.gz
$ cd Tutorial
$ ls
$ cd bin
$ ls
```

튜토리얼 버전에 따라 실행 스크립트 이름이 `run_siesta.py` 또는 `siesta.py`일 수 있다.  
아래 문서에서는 편의상 `run_siesta.py`로 적지만, 실제 파일명이 다르면 그 이름을 그대로 사용하면 된다.

이 스크립트는 SIESTA 실행 명령을 간단히 만들어 주고, `input/`과 `output/`을 분리해 계산 결과를 정리하기 쉽게 해 준다.

계산 디렉토리로 이동한 뒤 실행한다.

```bash
$ ls
$ python ../bin/run_siesta.py
```

만약 `bin/`이 한 단계 위가 아니라 두 단계 위에 있다면 다음처럼 경로를 바꾼다.

```bash
$ python ../../bin/run_siesta.py
```

실행하면 사용할 코어(MPI rank) 개수를 묻고, 입력한 개수로 계산이 시작된다.

### 1.2 `B. SIESTA 클러스터에서 설치` 방식으로 빌드한 경우

클러스터 튜토리얼에는 보통 `slm_siesta_run` 같은 제출 스크립트가 함께 들어 있다.

```bash
$ cd Tutorial/bin
$ ls
$ gedit slm_siesta_run
```
해당 스크립트에서 실제 siesta가 위치해 있는 위치 혹은 해당 bin에 있는 siesta 경로로 수정해준다.
```bash
...
RUNBIN="(siesta가 실제 있는 위치)"
...
```
계산 방법 예시:  
계산 하고자 하는 디렉토리에 실행 스크립트를 복사한 뒤 job을 제출한다.

```bash
$ cp ../../bin/slm_siesta_run .
$ ls
$ qsub slm_siesta_run
```

이 문서의 예시는 대부분 `qsub slm_siesta_run` 기준으로 적혀 있다.  
Edison 환경에서 진행한다면 같은 자리에서 `python ../bin/run_siesta.py`로 바꿔 실행하면 된다.

## 2. 분자 계산 개요 및 실습 목표

이 장에서는 SIESTA를 이용한 **비주기계(분자) 계산의 기본 workflow**를 연습한다.  
중요한 것은 정답 파일을 그대로 따라 치는 것이 아니라, "초기 구조를 준비하고 계산한 뒤 결과를 확인하는 흐름"을 익히는 것이다.

### 실습 목표

- **$CH_4$**
  구조 최적화를 수행하고, **basis 크기에 따라 에너지와 구조가 어떻게 달라지는지** 확인한다.
- **$H_2O$**
  구조 최적화를 수행하고, **전자 밀도(charge density)**를 저장해 결합 전자 분포를 살펴본다.
- **$NH_3$ (과제)**
  앞의 두 예제를 바탕으로, **직접 구조를 수정하고 계산 항목을 선택해 보는 연습**을 한다.

### 분자 계산의 기본 workflow

1. 분자 구조 준비 (`STRUCT.fdf`)
2. 계산 옵션 설정 (`RUN.fdf`, `BASIS.fdf`, `KPT.fdf`)
3. 구조 최적화 실행 (`CG step`)
4. 최적화 결과 구조 확인 (`xyz`, `ANI`, `STRUCT_OUT` 등)
5. 후처리 계산 수행 (basis test, charge density 저장 등)
6. 결과 비교 및 해석

> 이 장의 목표는 "정답 구조를 외우는 것"이 아니라,
> **초기 구조 → 최적화 → 결과 검증 → 추가 분석**의 흐름을 익히는 것이다.

## 3. 공통 준비: 입력 파일 구성과 실행 방법

분자 계산($CH_4$ / $H_2O$ / $NH_3$)은 모두 비슷한 폴더 구조와 입력 파일 구성을 사용한다.  
먼저 어떤 파일이 무엇을 하는지, 어디에서 실행해야 하는지부터 정리해 두면 이후 실습이 훨씬 수월하다.

### 3.1 입력 파일 구성 (공통)

SIESTA 계산에는 일반적으로 다음 파일들이 필요하다.

- `RUN.fdf`: 계산 종류(SCF, 구조 최적화 등), 수렴 조건, 출력 옵션
- `STRUCT.fdf`: 원자 좌표와 셀(cell) 정보
- `KPT.fdf`: k-point 설정. 분자 계산에서는 보통 `1x1x1`을 사용
- `BASIS.fdf`: basis 관련 설정 (`PAO.BasisSize` 등)
- `*.psf`: 원자별 pseudopotential 파일 (예: `C.psf`, `H.psf`, `O.psf`, `N.psf`)

> 분자 계산이라도 `STRUCT.fdf` 안에는 충분히 큰 셀(box)을 넣어 주는 것이 중요하다.
> 셀이 너무 작으면 주기경계조건 때문에 옆 셀의 분자와 원하지 않는 상호작용이 생길 수 있다.

### 3.2 구조 파일 준비 (`STRUCT.fdf`)

분자 계산의 첫 단계는 **초기 구조를 준비하는 것**이다.  
튜토리얼 폴더에 예제 구조가 들어 있더라도, 구조 파일을 직접 만들어 보는 과정을 한 번 익혀 두면 이후 다른 분자나 물질을 다룰 때 도움이 된다.

원하는 분자 구조는 Materials Project 같은 외부 데이터베이스에서 받아 `fdf` 형식으로 변환해 사용할 수 있다.

![materials-project](/siesta/img/ch4-h20-nh3-03.png){: style="display:block; height:300px; margin-left:auto; margin-right:auto;" }

#### 3.2.1 `POSCAR` 파일을 받은 경우

`POSCAR`가 이미 있다면 바로 `STRUCT.fdf`로 바꿀 수 있다.

```bash
$ ls
$ poscar2fdf.py CH4.poscar
```

스크립트가 `bin/` 폴더 안에 있고 실행 권한이 없는 경우에는 `python`으로 실행해도 된다.

```bash
$ python ../bin/poscar2fdf.py CH4.poscar
```

실행이 끝나면 현재 디렉토리에 `STRUCT.fdf` 또는 이에 대응하는 변환 파일이 생성된다.  
생성된 파일 이름은 `ls`로 바로 확인해 두는 것이 좋다.

#### 3.2.2 `CIF` 파일을 받은 경우

`CIF` 파일만 있을 때는 먼저 `POSCAR`로 바꾼 뒤 다시 `STRUCT.fdf`로 변환한다.

```bash
$ vaspkit
1105
struct.cif
$ poscar2fdf.py POSCAR
```

여기서 `1105`는 `vaspkit`에서 구조 파일을 변환할 때 사용하는 메뉴 번호다.  
정상적으로 끝나면 `POSCAR`가 생성되고, 이어서 `STRUCT.fdf`까지 만들 수 있다.

이 과정을 통해 $CH_4$, $H_2O$, $NH_3$의 초기 구조 파일을 준비할 수 있다.

#### 3.2.3 예시 `STRUCT.fdf`

아래는 분자 계산에서 사용하는 기본적인 `STRUCT.fdf` 예시이다.

```bash
NumberOfAtoms    5
NumberOfSpecies  2

%block ChemicalSpeciesLabel
 1 6 C
 2 1 H
%endblock ChemicalSpeciesLabel

LatticeConstant 10.000000000 Ang

%block LatticeVectors
    1.000000000     0.000000000     0.000000000
    0.000000000     1.000000000     0.000000000
    0.000000000     0.000000000     1.000000000
%endblock LatticeVectors

AtomicCoordinatesFormat ScaledCartesian

%block AtomicCoordinatesAndAtomicSpecies
 0.4991728  0.4991728  0.4992459  1  1
 0.6028726  0.4747901  0.4680767  2  2
 0.4747901  0.6028726  0.4680767  2  3
 0.4908115  0.4908115  0.6096102  2  4
 0.4285072  0.4285072  0.4511578  2  5
%endblock AtomicCoordinatesAndAtomicSpecies
```

분자 계산에서는 보통 셀을 크게 잡고 분자를 셀 중앙 근처에 두는 방식으로 시작한다.

### 3.3 `*.psf` (pseudopotential) 준비

SIESTA는 원자별 **슈도포텐셜 파일(`*.psf`)**이 필요하다.  
튜토리얼 폴더에 필요한 `*.psf`, `BASIS.fdf`, `KPT.fdf`, `slm_siesta_run`이 정리되어 있다면 우선 그것을 사용하면 된다.

직접 준비해야 한다면 `https://www.pseudo-dojo.org/` 같은 사이트를 참고할 수 있다.  
이때는 사용하려는 교환-상관 함수(`LDA`, `GGA` 등)에 맞는 pseudopotential을 선택해야 한다.

### 3.4 계산용 디렉토리 구성

아래와 같은 구조로 폴더를 정리해 두면 계산과 결과 확인이 편하다.

```bash
CH4/
 ├─ slm_siesta_run      (또는 run_siesta.py를 참조해 실행)
 └─ input/
     ├─ RUN.fdf
     ├─ STRUCT.fdf
     ├─ BASIS.fdf
     ├─ KPT.fdf
     ├─ C.psf
     └─ H.psf
```

$H_2O$, $NH_3$도 같은 구조를 사용하고, 필요한 원소의 `psf`만 바꾸면 된다.

### 3.5 실행 방법 (환경에 따라 선택)

먼저 계산 디렉토리에서 `input/`이 보이는지 확인한다.

```bash
$ pwd
$ ls
```

그 다음 환경에 맞는 방식으로 계산을 시작한다.

```bash
$ qsub slm_siesta_run
```

```bash
$ python ../bin/run_siesta.py
```

둘 중 무엇을 써야 하는지는 문서 맨 앞의 `1. 주의사항` 기준으로 결정하면 된다.

### 3.6 구조 확인

계산 전후 구조를 확인하는 방법은 여러 가지가 있다.  
처음에는 "어떤 파일을 어떤 프로그램으로 여는지"만 익혀도 충분하다.

#### 3.6.1 VESTA에서 확인 (Edison에서 권장)

`https://jp-minerals.org/vesta/en/download.html`에서 로컬 환경에 맞는 `VESTA`를 설치한다.  
Edison에서는 Jupyter 환경에서 x11 forwarding이 불편할 수 있으므로, 구조 파일을 로컬로 가져와 `VESTA`로 확인하는 편이 더 쉽다.

VESTA는 `*.xyz`, `*.cif`, `POSCAR` 형식의 파일을 열 수 있다.  
만약 파일 이름이 `CH4.poscar`처럼 되어 있다면 `POSCAR`로 이름을 바꿔 열어도 된다.

```bash
$ mv CH4.poscar POSCAR
```

#### 3.6.2 ASE, XCrySDen 활용

터미널 환경에서 빠르게 구조만 보고 싶다면 `ASE`나 `XCrySDen`도 유용하다.

```bash
$ ase gui POSCAR
```

```bash
$ fdf2xcrysden.py STRUCT.fdf
```

보통은 `ASE`가 더 간단하고, `XCrySDen`은 결합 길이와 각도 측정이 필요할 때 유용하다.

## 4. 분자 계산 실습 (목표별 과제)

### 4.1 CH4: basis 크기에 따른 에너지/구조 변화(수렴성) 확인

#### 4.1.1 CH4 분자 구조 확인

튜토리얼에 포함된 `STRUCT.fdf`는 이미 최적화가 끝난 구조일 수 있다.  
이번 실습에서는 일부러 약간 어긋난 구조를 넣고, 구조 최적화를 통해 올바른 형태로 돌아가는지 확인해 본다.

$CH_4$는 네 개의 C-H 결합 길이가 모두 같아야 하는 대표적인 분자다.  
따라서 H 원자 하나의 위치를 조금 바꾼 입력 구조를 넣으면, 최적화가 제대로 되었는지 비교하기 쉽다.

```bash
NumberOfAtoms    5
NumberOfSpecies  2

%block ChemicalSpeciesLabel
 1 6 C
 2 1 H
%endblock ChemicalSpeciesLabel

LatticeConstant 10.000000000 Ang

%block LatticeVectors
    1.000000000     0.000000000     0.000000000
    0.000000000     1.000000000     0.000000000
    0.000000000     0.000000000     1.000000000
%endblock LatticeVectors

AtomicCoordinatesFormat ScaledCartesian

%block AtomicCoordinatesAndAtomicSpecies
 0.4991728  0.4991728  0.4992459  1  1
 0.6028726  0.4747901  0.4680767  2  2
 0.4747901  0.6028726  0.4680767  2  3
 0.4908115  0.4908115  0.6096102  2  4
 0.4285072  0.4285072  0.4511578  2  5
%endblock AtomicCoordinatesAndAtomicSpecies
```

이제 이 구조가 실제로 약간 비틀린 상태인지 먼저 눈으로 확인해 보자.

- `XCrySDen`으로 확인

```bash
$ cd input
$ fdf2xcrysden.py STRUCT.fdf
```

- `ASE`로 확인 (더 간단해서 권장)

```bash
$ cd input
$ fdf2poscar.py STRUCT.fdf
$ ase gui STRUCT.poscar
```

만약 `fdf2xcrysden.py` 실행 시 `LatticeVectors` 관련 에러가 난다면, `STRUCT.fdf` 안에 셀 정보가 빠졌을 가능성이 있다.  
이럴 때는 먼저 `xyz`로 바꿔 확인하는 것이 가장 간단하다.

```bash
if cell.shape == (3,3):
^^^^^^^^^^
AttributeError: 'list' object has no attribute 'shape'
```

```bash
$ cd input
$ fdf2xyz.py STRUCT.fdf
$ ls
$ xyz2xcrysden.py STRUCT.xyz
```

ASE를 통해 분자 구조를 볼 때는 `ctrl` 을 누르고 두 원자를 클릭하면 결합 길이, 세 원자를 클릭하면 결합 각도가 하단에 뜨는 것을 확인할 수 있다.

XCrySDen에서 분자 구조만 보려면 실행 후 메뉴에서  
`Display -> Unit of Repetition -> Translational asymmetric unit`을 선택한다.

- 결합 길이 확인: `Distance` 도구를 선택한 뒤 두 원자를 클릭하고 `Done`
- 결합 각도 확인: `Angle` 도구를 선택한 뒤 세 원자를 클릭하고 `Done`

입력 구조를 보면 $CH_4$ 분자의 한 결합 길이가 다른 결합과 약간 다르게 설정되어 있음을 확인할 수 있다.

#### 4.1.2 구조 최적화 실행 및 결과 확인

이제 수정한 `STRUCT.fdf`를 `input/` 폴더에 두고 계산을 실행한다.  
구조 최적화를 위해서는 `RUN.fdf`와 `BASIS.fdf`도 함께 조정한다.

- `RUN.fdf`: `MD.NumberCGsteps`를 `300` 정도로 늘려 충분히 최적화가 진행되게 한다.
- `BASIS.fdf`: `PAO.BasisSize`를 `SZ`로 바꿔 먼저 가볍게 구조를 확인한다.

예를 들어 아래 항목이 있는지 확인해 두면 편하다.

```bash
$ cd input
$ gedit RUN.fdf # or vi RUN.fdf
```

계산은 `input/` 폴더가 있는 계산 디렉토리에서 실행한다.

```bash
$ pestat
$ qsub slm_siesta_run
$ myq
```

Edison 환경이라면 같은 자리에서 아래처럼 실행한다.

```bash
$ python ../bin/run_siesta.py
```

계산이 끝나면 실행 스크립트가 만든 `output/` 또는 `OUT/` 폴더 안에 결과 파일이 생긴다.  
폴더 이름이 헷갈리면 먼저 `ls`로 확인한다.

```bash
$ ls
$ ls output
```

예제 기준으로는 최적화된 결과 구조가 `Test.xyz` 형태로 저장된다.

`Test.xyz`에는 셀 정보가 없어서 XCrySDen에서 바로 열리지 않을 수 있다.  
이 경우 기존 `STRUCT.xyz`의 셀 정보를 `Test.xyz` 상단에 복사한 뒤 열면 된다.  
간단히 구조만 확인할 목적이라면 `ASE`로 여는 쪽이 더 편하다.

```bash
$ cd output
$ ase gui Test.xyz
```

```bash
$ cd output
$ xyz2xcrysden.py Test.xyz
```

![03_010](/siesta/img/ch4-h20-nh3-09.png){: style="display:block; height:800px; margin-left:auto; margin-right:auto;" }

입력 구조에서는 결합 길이가 약 `0.001 Ang` 정도 차이 났지만, 최적화 후에는 차이가 더 작아진다.  
즉, 구조 최적화를 통해 $CH_4$에 더 적절한 대칭 구조로 수렴했음을 확인할 수 있다.

#### 4.1.3 CH4 분자 basis 확인

구조 최적화가 끝났다면, 이번에는 사용한 basis가 충분한지 확인한다.  
이 단계에서는 **구조를 크게 바꾸는 것보다, basis 선택에 따라 결과가 얼마나 달라지는지 비교**하는 것이 핵심이다.

보통은 최적화된 구조를 기준으로 `PAO.BasisSize`만 바꿔 가며 계산한다.

```bash
$ cd input
$ gedit BASIS.fdf # or vi BASIS.fdf
```

아래 그림의 `PAO.BasisSize` 항목을 `SZ`, `DZ`, `TZ`, `DZP`, `DZDP` 등으로 바꿔 테스트한다.

![03_014](/siesta/img/ch4-h20-nh3-10.jpg){: style="display:block; height:100px; margin-left:auto; margin-right:auto;" }

계산이 끝나면 각 basis에서 얻은 결합 길이와 결합각을 비교하고, 문헌값과도 함께 비교해 본다.

Reference: Handy, Nicholas C., Christopher W. Murray, and Roger D. Amos,  
"Study of methane, acetylene, ethene, and benzene using Kohn-Sham theory."  
The Journal of Physical Chemistry 97.17 (1993): 4392-4396.

![03_015](/siesta/img/ch4-h20-nh3-11.jpg){: style="display:block; height:250px; margin-left:auto; margin-right:auto;" }

|  | SZ | DZ | TZ | DZP | DZDP |
| --- | --- | --- | --- | --- | --- |
| `Bonding length [Ang]` | 1.20100 | 1.10876 | 1.11050 | 1.10977 | 1.10861 |
| `Bonding angle [degree]` | 109.4398 | 109.1358 | 109.1567 | 109.4914 | 109.3702 |

이 표를 보면 `SZ`는 다소 거칠고, `DZ` 이상부터 값이 많이 안정되는 경향을 볼 수 있다.  
실제 계산에서는 정확도와 계산 비용을 함께 고려해 적절한 basis를 선택한다.

---

### 4.2 H2O 분자 구조 최적화 및 전자 밀도 확인

#### 4.2.1 H2O 분자 구조 최적화

$H_2O$도 기본적인 구조 최적화 절차는 `4.1`과 같다.  
다만 이번에는 구조 최적화뿐 아니라 전자 밀도까지 볼 예정이므로 `RUN.fdf`에 관련 옵션을 추가한다.

```bash
# RUN.fdf
...
WriteHirshfeldPop T
SaveDeltaRho      T
...
```

- `WriteHirshfeldPop`: 원자별 전하 분포를 해석할 때 참고할 수 있는 출력을 남긴다.
- `SaveDeltaRho`: 전하 밀도 관련 후처리에 필요한 데이터를 저장한다.

계산이 끝난 뒤에는 구조가 어떻게 변했는지도 함께 확인해 보자.

```bash
$ cd output
$ ls
$ cp H2O.ANI opt.xyz
$ ase gui opt.xyz
```

여기서는 `H2O.ANI`를 복사해 `opt.xyz`라는 이름으로 열었다.  
이 파일을 보면 최적화 과정에서 원자가 step마다 어떻게 움직였는지 확인할 수 있다.

![03_016](/siesta/img/ch4-h20-nh3-13.png){: style="display:block; height:250px; margin-left:auto; margin-right:auto;" }

#### 4.2.2 H2O 전자 밀도 확인

전자 밀도를 시각화하려면 계산이 끝난 뒤 후처리 과정이 한 번 더 필요하다.

```bash
$ cd output
$ cp ../../../bin/input_rho2xsf.txt .
$ gedit input_rho2xsf.txt # or vi input_rho2xsf.txt
$ rho2xsf < input_rho2xsf.txt
```

`input_rho2xsf.txt` 안에는 현재 계산 파일 이름과 맞아야 하는 항목이 있을 수 있으므로, 실행 전에 한 번 열어서 확인하는 편이 안전하다.

![03_017](/siesta/img/ch4-h20-nh3-14.png){: style="display:block; height:250px; margin-left:auto; margin-right:auto;" }

`rho2xsf` 실행이 끝나면 `H2O.XSF`와 같은 시각화용 파일이 생성된다.  
이 파일을 로컬로 옮겨 `VESTA`에서 열면 $H_2O$의 전자 밀도를 볼 수 있다.

VESTA에서 단면을 확인하는 한 가지 방법은 다음과 같다.

1. 좌측 상단의 선택 도구로 $H_2O$ 원자들을 드래그한 뒤, 하단의 `Boundary`를 누른다.
2. `New`를 누르고 `Calculate the best plane...`를 선택한 뒤 `Apply`를 누른다.
3. 하단의 `Properties`에서 `Section` 항목으로 들어가 `isosurface level`을 `Max: 0.02`, `Min: -0.02`로 설정한다.

![03_018](/siesta/img/ch4-h20-nh3-15.png){: style="display:block; height:250px; margin-left:auto; margin-right:auto;" }

### 4.3 NH3 개별 실습 과제

이제 앞에서 했던 과정을 $NH_3$에 직접 적용해 본다.  
처음부터 완전히 새로운 작업을 하는 것이 아니라, `CH4`와 `H2O`에서 했던 절차를 스스로 다시 조합하는 연습이라고 생각하면 된다.

권장 순서는 다음과 같다.

1. 주어진 `STRUCT.fdf`에서 한 원자의 위치를 약간 바꿔 "조금 틀어진 초기 구조"를 만든다.
2. 구조 최적화를 실행해 안정한 구조로 수렴하는지 확인한다.
3. `PAO.BasisSize`를 바꿔 basis 수렴성을 간단히 비교해 본다.
4. 필요하면 전자 밀도 시각화까지 수행한다.

전자 밀도까지 확인한다면 `Properties -> Isosurfaces -> Isosurface level` 값을 `0.007` 정도로 맞춰 보자.  
결과가 너무 진하거나 너무 약하게 보이면 이 값은 조금씩 조절해도 된다.
