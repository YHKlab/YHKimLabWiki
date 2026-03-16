슈도포텐셜 (Pseudopotential) 생성 & 검증
===============================
## Contents
1. 슈도포텐셜 생성
2. 슈도포텐셜 검증
3. 심화과정 (Fe 원자) 

 
---
## 1. 슈도포텐셜 생성

이번 장에서는 **전(全)전자(all-electron) 계산**에 이어 **슈도포텐셜(pseudopotential) 생성** 방법을 다룬다.  
예시로는 Si 원자를 사용하며, ATOM 프로그램의 pg 모드를 이용해 pseudopotential을 생성한다.

전전자 계산에서 확인했듯이, core 전자는 핵 근처에 강하게 국소화되어 있고 화학 결합에는 주로 valence 전자가 관여한다.
슈도포텐셜은 이러한 점을 이용하여 core 영역 부분을 단순화하면서도 valence 물리량은 유지하도록 만든 유효 포텐셜이다. 

### 1-1. 입력값 설명
우선, `ATOM`이 설치된 위치에서 `/Tutorial/1.atom`에 위치한 다음 파일을 살펴보자:
```bash
$cd Tutorial/All_electron/1.atom/1.Si/si.pg.inp
$vi Si.pg.inp
```
`Si.pg.inp`:  

```bash
#
#  Pseudopotential generation for Silicon
#  pg: simple generation
#
   pg      Silicon
        tm2      3.0             # PS flavor, logder R
 n=Si c=car                      # Symbol, XC flavor,{ |r|s}
       0.0       0.0       0.0       0.0       0.0       0.0
    3    4                       # norbs_core, norbs_valence
    3    0      2.00      0.00   # 3s2
    3    1      2.00      0.00   # 3p2
    3    2      0.00      0.00   # 3d0
    4    3      0.00      0.00   # 4f0
      1.90      1.90      1.90      1.90      0.00      0.00
#
# Last line (above): 
#    rc(s)     rc(p)     rc(d)     rc(f)   rcore_flag  rcore
#
#2345678901234567890123456789012345678901234567890123456789
```  

**첫 번째 줄**:  
- `ae`: All-electron calculation  
- `pg`: Pseudopotential generation  
- `pe`: Pseudopotential generation (core correction)  

**두 번째 줄**:  
• pseudopotential 생성 방식 (tm2, ker 등)
• 로그도함수(logarithmic derivative) 테스트 반경 설정 (예: 3.0) 

**세 번째 줄**:  
• 원소 기호  
• 교환-상관(exchange-correlation) 범함수 종류:  
- `ca`: Ceperley-Alder(LDA)  
- `pb`: Perdew, Burke, and Ernzerhof(GGA)  
> `xc` 옵션 뒤에 `s`를 붙여주면 `spin-polarized`, `non-relativistic` 계산을 수행할 수 있고,  
 `r`를 붙여주면 `spin-polarized`, `relativistic` 계산을 수행할 수 있다.

**네 번째 줄**:  
• 대부분의 계산에서 사용되지 않는 옵션이다  

**다섯번째 줄**:  
• 코어 오비탈 수(1열)  
• 원자가 오비탈 수(2열)  

**여섯번째~아홉번째 줄**:  
• 원자가 오비탈 정보(다섯번째 줄에서 지정한 수만큼)  : `n` 양자수 (1열), `l` 양자수 (2열), 전자 점유도 (3,4열)  

**마지막 줄**:   
• s, p, d, f 오비탈에 대한 슈도포텐셜 cutoff 반경 (1~4열)  
• Core correction 사용 여부 (0 또는 1) (5열), Core correction 기준 반경 값 (6열)   


### 1-2. 계산 및 결과 설명  

전(全)전자(all-electron) 계산의 입력 파일과 다른 점은 상단 왼쪽에 위치한 계산 모드가 `ae`가 아니라 `pg`라는 점이다.  
슈도포텐셜 생성을 수행하려면 반드시 첫 줄의 모드를 `pg`로 설정해야 한다.  
계산 실행은 `ae.sh`가 아니라 `pg.sh` 스크립트를 사용한다.

``` bash
$ sh ../../bin/pg.sh Si.pg.inp
```
  
계산이 완료되면 SIESTA에서 사용할 수 있는 Si.pg.psf 파일이 생성되며,
슈도포텐셜 계산 결과는 Si.pg/ 디렉토리에 저장된다.  
생성된 `Si.pg` 위치에 가면 다음과 같은 결과 파일들이 생성되어 있다:  

• `OUT`: 계산 결과 정보를 담은 출력 파일  
• `PSCHARGE`: 전자 밀도(multiplied by 4πr²)   
• `PSPOTR0` ~ `PSPOTR3`: 슈도포텐셜(s, p, d, f 오비탈 순)  
• `PSPOTQ0` ~ `PSPOTQ3`: 퓨리에 변환된 슈도포텐셜 
• `PSPOTR0` ~ `PSPOTR3`: 슈도포텐셜(s, p, d, f 오비탈 순)  
• `PSWFNR0` ~ `PSWFNR3`: 원자가 전자에 대한 슈도 파동함수  
• `PSWFNQ0` ~ `PSWFNQ3`: 퓨리에 변환된 원자가 전자에 대한 슈도 파동함수  


또한 gnuplot를 사용하면 생성된 슈도포텐셜을 시각화할 수 있다  

```bash
$ gnuplot --perist pots.gplot
```

<img src="../img/pseudopotential-03.jpg" width="500"/>


추가로 자주 사용하는 시각화 스크립트는 다음과 같다.

• pseudo : Norm-conserving 조건 확인용 그래프  
• charge : all-electron vs pseudopotential 전자밀도 비교


## 2. 슈도포텐셜 테스트
생성된 **슈도포텐셜(pseudopotential)** 은 단순히 파일이 만들어졌다고 해서 바로 사용하면 안 된다.  
전(全)전자 계산 결과와 비교하여 **정확도**와 **transferability**를 검증해야 한다.

이 파트에서는 ATOM 프로그램으로 다음을 확인한다.

• Norm-conserving pseudopotential 조건  
• Transferability test (전자 배치를 바꾼 경우에도 잘 맞는지)

### 2-1. Norm-conserving pseudopotentials  

**Norm-Conserving Pseudopotentials, D. R. Hamann, M. Schlüter, and C. Chiang PRL (1979)** [2](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.43.1494)   
논문에서 제시한 pseudopotential의 조건은 다음과 같다.  


> 조건 1. Pseudo와 real의 **원자가 고유값 (valence eigenvalue)** 이 일치한다  
> 조건 2. Pseudo와 real의 **파동함수 (wavefunction)** 가 $r_c$ (core radius) 이후에서 일치한다  
> 조건 3. Pseudo와 real의 **전하 밀도 (charge density) 적분값**이 $r_c$ 이후에서 일치한다 (norm conservation)  
> 조건 4. Pseudo와 real의 **로그도함수 (logarithmic derivatives)** 가 $r_c$ 이후에서 일치한다  


`ATOM`의 pg 계산 결과(OUT, AE*, PS* 파일)를 이용해 위 조건들을 직접 확인할 수 있다.

#### **조건 1. 원자가 고유함수 (valence eigenvalue)**  

pg 계산의 OUT 파일에는 all-electron과 pseudopotential 생성 결과가 함께 포함되어 있으므로,  
valence eigenvalue 비교는 OUT 파일에서 바로 확인할 수 있다.

```bash
(Pseudopotential generation 결과 폴더에서)
$ grep ‘&v’ OUT
```

<img src="../img/pseudopotential-04.jpg" width="500" />

출력 결과에는 all-electron 영역과 pseudopotential 영역의 valence 오비탈 정보가 나타나며,  
&v 라벨이 붙은 줄에서 원자가 오비탈의 고유값을 비교할 수 있다.

확인 포인트:  
• pseudo와 real의 valence eigenvalue가 잘 일치하는가?

#### **조건 2. 파동함수 (Wavefunctions)**  

pg 계산 폴더에는 all-electron / pseudo 파동함수가 함께 저장된다.

• AEWFNR# : all-electron 원자가 파동함수  
• PSWFNR# : pseudo 원자가 파동함수
(# = 0,1,2,3 → s,p,d,f)

예를 들어 Si의 3p 오비탈(R1)을 비교하려면 다음과 같이 그릴 수 있다.

```bash
$ gnuplot
gnuplot > plot 'AEWFNR1' w l, 'PSWFNR1' w l
```


<img src="../img/pseudopotential-05.jpg" width="500"/>

확인 포인트:

core 근처에서는 pseudo 파동함수가 더 부드럽다
$r_c$ 이후에서는 all-electron과 pseudo 파동함수가 겹친다

#### **조건 3. 전하 밀도 (Charge density)**  

pg 계산 폴더에서  

• AECHARGE : all-electron 전자밀도  
• PSCHARGE : pseudo valence 전자밀도

를 비교하여 norm-conserving 조건을 확인한다.

AECHARGE에서 마지막 열은 core 전자밀도이므로,  
valence 전자밀도를 비교할 때는 전체 전자밀도에서 core 전자밀도를 빼야 한다.

```bash
$ gnuplot
gnuplot >plot 'PSCHARGE' u 1:($2+$3) w l		($2+$3 : 전체 원자가 전자밀도)
gnuplot > set xrange [0:20]
gnuplot >replot 'AECHARGE' u 1:($2+$3-$4) w l    ($4 : 핵 전자밀도)
```

<img src="../img/pseudopotential-06.jpg" width="500" height="400"/>

확인 포인트:  
• pseudo valence 전하 밀도와 AE valence 전하 밀도가 $r_c$ 이후에서 잘 일치하는가?  
• pseudo 쪽이 core 근처에서 더 부드러운가?

#### **조건 4: 로그도함수 (Logarithmic derivatives of wavefunctions)**  

입력 파일 두 번째 줄의 `tm2` `3.0`와 같은 옵션은 로그도함수 검증과 관련이 있다.  
`pg` 계산 결과에는 all-electron / pseudo 파동함수의 로그도함수가 저장된다.

• AELOGD# : all-electron 로그도함수  
• PSLOGD# : pseudo 로그도함수

비교 방법은 다음과 같다.

```bash
$ gnuplot
gnuplot >plot ' AELOGD1' w l
gnuplot >replot ' PSLOGD1' w l
```

<img src="../img/pseudopotential-07.jpg" width="500" height="400"/>

확인 포인트:

• 로그도함수 곡선이 잘 겹치는가?
• 특히 $r_c$ 이후에서 일치하는가?

### 2-2. Transferability test  

위 조건들(특히 전하 밀도/로그도함수 일치)은 transferability와 관련이 깊다.  
Transferability란 pseudopotential이 바닥상태뿐 아니라 다른 전자 배치에서도 정확한지를 의미한다.  

이를 확인하려면 전자 배치를 바꾼 여러 case에 대해  
• all-electron 계산 (ae)  
• pseudopotential test 계산 (pt)  
을 같은 입력 파일 안에 함께 넣고 비교하면 된다.

`Tutorial/1.atom/1.Si` 위치에 있는 `Si.test.inp` 파일을 확인해본다.

```bash
$vi Si.test.inp
```
Si.test.inp에는 같은 전자배치에 대해 ae 계산 영역과 pt 계산 영역이 모두 들어 있다.
예를 들어 다음과 같은 배치를 포함할 수 있다.

• GS: 3s2 3p2  
• Test 1: 3s2 3p1 3d1  
• Test 1: 3s1 3p3  
• Test 1: 3s1 3p2 3d1  
• Test 1: 3s0 3p3 3d1  

여기서 `pt`는 transferability test를 위한 계산 모드이다.
`pt` 계산은 기존 `pg` 계산에서 생성된 `.vps` 파일을 입력으로 사용한다.
따라서 먼저 `pg` 계산이 완료되어 `Si.pg.vps` 파일이 있어야 한다.

```bash
$ cp ../si.pg/si.pg.vps .
$ sh ../../Utils/pt.sh Si.test.inp Si.pg.vps
```

**Total energy 비교**:  
```
$ grep ‘&d’ OUT
```
<img src="../img/pseudopotential-08.jpg" width="500" height="300"/>
 


**Eigenstate 비교**:  
```
$ grep ‘&v’ OUT
```
<img src="../img/pseudopotential-09.jpg" width="500" height="500"/>

 

위 과정을 통해 `Si` 원자의 all-electron과 pseudopotential에 대한 여러 전자배치의 `total energy`와 `eigenvalue`를 비교하여, transferability를 시험할 수 있다.

## 3. 심화과정

### 3-1. Fe 원자

이제 실제 `SIESTA` 프로그램을 위한 pseudopotential 입력파일을 만들어보자. 위에서 언급했듯 `SIESTA`는 basis가 되는 오비탈이 (l =3)까지 있어야한다. 이런 점을 만족시키는 pseudopotential generations의 입력파일은 다음 사이트에서 얻을 수 있다. [pseudo](https://departments.icmab.es/leem/SIESTA_MATERIAL/Databases/Pseudopotentials/periodictable-intro.html) (현재는 사용불가).

위 사이트에 가면 pseudopotential을 만드는 방법으로 `LDA` (localized density approximation)과 ‘GGA’ (generalized gradient approximation)을 선택할 수 있다. `LDA` 방법은 `Au` 원자와 같은 전위금속의 구조를 정확히 예측하지 못한다[3](https://iopscience.iop.org/article/10.1088/0953-8984/13/42/307/meta). 따라서 `GGA` 방법을 선택하여 `Fe`의 `ATOM` 입력파일 (input file for ATOM program)을 설치한다.

`Fe.inp` :
```bash
pe                 -- file generated from Fe ps file
        tm2
   Fe   pb
     0.000     0.000     0.000     0.000     0.000     0.000
    5    4
    4    0     2.000     0.000    #4s
    4    1     0.000     0.000    #4p
    3    2     6.000     0.000    #3d
    4    3     0.000     0.000    #4f
   2.41000   2.53000   2.29000   2.29000   0.01000  -1.00000 small core charge

#23456789012345678901234567890123456789012345678901234567890      Ruler
```

앞서 입력 옵션을 설명했듯이, 첫번째 줄에 `pe`는 **core correction**을 위한 옵션이다. 원자의 코어 전자와 최외각 전자의 파동함수가 상당부분 겹칠 경우에, core correction를 통해 더욱 안정적으로 pseudopotential을 만들 수 있다. 두번째 줄의 `tm2`은 이전에 설명한 transferability를 위해 로그도함수를 시험하기 위한 옵션이다. 또한 세번째 줄에 `pb` 옵션을 볼 수 있는데, 이는 `GGA` 방법의 종류로 ` PBE` ((Perdew, Burke, and Ernzerhof) 방법을 의미한다. <br>다른 방법들은 메뉴얼을 통해서 확인하도록 한다.
<br>`Au`와 같이 원자번호가 큰 금속은 상대론적 효과가 나타난다 (원자핵 주변으로 핵 전자들이 더욱 구속되면서 가림 효과 (shielding effect)가 커지고, d나 f 오비탈의 전자들의 구속력이 약해진다)[4]( https://link.springer.com/article/10.1007/BF03215471). 따라서 계산에서도 상대론적 효과를 고려하는 것이 좋다. 
<br>`pb` 옵션 뒤에 `s`를 붙여주면 `spin-polarized`, `non-relativistic` 계산을 수행할 수 있고, `r`를 붙여주면 `spin-polarized`, `relativistic` 계산을 수행할 수 있다. `pb`를 `pbr`로 바꾸어 주어 상대론적 효과를 고려한 pseudopotential을 만들어보자.

```bash
$ <ATOM 프로그램 위치>/Tutorial/bin/pg.sh Fe.inp
```

`SIESTA` 계산을 위한 `Fe.psf` 파일이 생성되었다.

### 3-2. Core correction

`ATOM` 프로그램은 **non-linear exchange-correlation correction** [1](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.26.1738) 계산을 수행할 수 있다.  
*Pseudo core의 전하밀도가 특정한 pseudo radius 밖에서 전하밀도도가 일치하고 경계부에서 매끈한 형태를 가지게 맞추어 주는 일련의 과정*이 `core correction`이다. Pseudopotential를 만드는 계산에서 *이 계산*을 포함시키기 위해서 앞서 실행한 입력 파일에서 `pg` 옵션을 `pe`로 바꾸어 주면 된다. 또한 마지막 줄에서 6번째 해당하는 값인 pseudo radius을 넣어준다. 
<br>만약 이 값이 음수이거나 0이면 5번째 값인 valence charge density를 통해 이 반경을 직접 계산하게 되는데, 이 값 역시 음수이면 전체 핵 전하를 기준으로, 0이면 1 값을 기준으로 계산하도록 되어있다. 따라서 pseudo radius을 실험적으로 얻은 외부 참조 값을 이용하여 넣어주는 것을 강력히 추천한다.

### 3-3. 상대론적 효과

> 해당 강의를 수행하기 앞서 기본적인 [SIESTA 사용법](../siesta/basic.md)에 대해서 숙지한다.

Pseudopotential을 만든 이유는 보통 계산에 사용하기 위에서이다. 일반적으로는 pseudopotential의 4가지 조건을 확인한 것 만으로 정확한 계산을 할 수 있다. 그러나, Au와 같이 전자수가 많은 경우 그렇지 않다. 이번에는 두가지 Au pseudopotential을 만들어 보고, 만든 pseudopotential이 계산에 적합한지 확인해보겠다.

### (1) 슈도포텐셜 생성  

위에서 언급했듯이 원자번호가 큰 원자는 상대론적 효과가 나타난다. 원자의 무게가 크면 클수록 상대론적 효과는 더 크게 나타나고, 상대론적 효과가 커질수록 상대론적 효과를 고려한 pseudopotential과 고려하지 않은 pseudopotential의 결과의 차이가 커진다. 이번에는 79번 Au 원소를 사용해 상대론적 효과를 고려한 pseudopotential과 고려하지 않은 pseudopotential을 만들고 계산을 수행한 후 그 결과를 비교해보겠다.  

우선 atom코드에서 사용할 input 코드를 가져와야 한다. atom에서 사용할 reference 코드는 [pseudo](https://departments.icmab.es/leem/SIESTA_MATERIAL/Databases/Pseudopotentials/periodictable-intro.html)에서 가져올 수 있다. 페이지에서 LDA를 선택한 후, Au를 선택하고, "input file for the ATOM program"을 선택한다. 그러면 Au.inp파일을 받을 수 있다.

`Au.inp`:

```bash
   pg                 -- file generated from Au ps file
        tm2
   Au   ca 
     0.000     0.000     0.000     0.000     0.000     0.000
   12    4
    6    0     1.000     0.000    #6s
    6    1     0.000     0.000    #6p
    5    2    10.000     0.000    #5d
    5    3     0.000     0.000    #5f
   2.63000   2.77000   2.63000   2.63000   0.00000   0.00000

#23456789012345678901234567890123456789012345678901234567890      Ruler
```

Au pseudopotential에 상대론적 효과 옵션을 넣기 위해서는 r옵션을 넣어주어야 한다. 이 옵션은 ca오른쪽에 넣어주면 된다. 주의할 점은 위치이다. r옵션은 Ruler의 첫번째 0 위에 반드시 위치해야한다. 만약 이 위치가 다르다면 atom으로 pseudopotential을 생성할 때 에러 메세지가 생길 것이다.
*(tm2의 시작 위치에 맞추어 입력해주면 된다.)*

`Au.inp` (잘못된 예시):  

```bash
   pg                 -- file generated from Au ps file
        tm2    2.63
   Au    car  # <---위치가 잘못됨
     0.000     0.000     0.000     0.000     0.000     0.000
   12    4
    6    0     1.000     0.000    #6s
    6    1     0.000     0.000    #6p
    5    2    10.000     0.000    #5d
    5    3     0.000     0.000    #5f
   2.63000   2.77000   2.63000   2.63000   0.00000   0.00000

#23456789012345678901234567890123456789012345678901234567890      Ruler
```

상대론적 효과 이외에도 계산 과정에서 xc를 바꾸거나 rc를 바꾸거나 해야할 수 있다. 이런 옵션들을 알아보려면 [atom manual](https://siesta-project.org/SIESTA_MATERIAL/Pseudos/Code/atom-4.2.0.pdf)을 찾아보면 된다.

### (2) 고체 계산(SIESTA) 

#### 1. Lattice constant

우선적으로 두가지 Au pseudopotential로 만든 fcc 구조 bulk cell의 lattice constant를 각각 구해볼 것이다. bulk model은 Tutorial 1에서 사용한 fcc모델을 사용할 것이다. Tutorial 1에서 k-point test를 이미 거쳤기 때문에 tutorial 1에서 사용한 k인 k=35를 사용해서 lattice constant를 계산했다.

|     Basis size     |      DZP       |
| :----------------: | :------------: |
| Basis energy shift |   100 [meV]    |
|         XC         |      LDA       |
|       DM.tol       | $10^{-3}$ [eV] |

lattice constant는 Tutorial 1에서와 마찬가지로 일정한 간격으로 lattice constant를 변화시키면서 
에너지가 가장 낮은 lattice constant를 찾으면 된다.
계산의 효율을 위해 처음에는 sparse하게 계산한 후 에너지가 가장 낮은 값 근처에서 dense하게 계산하면 좋다. 
이후 3d 물질에서 lattice constant를 찾기에 적합한 murnaghan fitting을 통해 에너지가 가장 낮은 lattice constant를 찾았다

![lattice_compare](../img/pseudopotential-01.png)

relativistic 효과를 킨 경우 최적화된 volume은 69.28785 Å 이고, 
relativistic 효과를 끈 경우 최적화된 volume은 79.22283 Å이다. 
이를 통해 lattice constant를 구하려면 부피의 세제곱근을 하면 된다. 
relativistic 효과를 킨 경우 lattice constant는 4.107 Å이고, 
relativistic 효과를 끈 경우 lattice constant는 4.295 Å이다. 
[reference](https://www.sciencedirect.com/science/article/pii/S0927025614007940#t0015)에서 구한 
실험의 lattice constant는 4.080 Å이므로 relativistic 효과를 킨 경우가 더 정확함을 알 수 있다.


#### 2. Band structure

이번에는 Au bulk의 band를 그려서 두 pseudopotential에 어떤 차이가 있는지 알아보자. 
band structure를 그릴 때 사용할 [reference](https://www.sciencedirect.com/science/article/pii/S0927025614007940#t0015)는 
Tutorial 1에서도 사용한 이 band 그래프이다. 
band path는 모두 $\Gamma-X-W-L-\Gamma-K$ 로 설정해준다. 보고싶은 에너지 범위는 $E_F \pm 10\,\mathrm{eV}$이다. 
이 조건들을 만족시키면서 band 그래프를 그려보면 결과는 다음과 같다.

![band_result](../img/pseudopotential-02.png)

페르미 에너지 위 5eV부분을 보면 relativistic의 band structure은 reference와 비슷하지만, 
non-relativistic의 band structure은 reference와 상이함을 알 수 있다. 
따라서 Au pseudopotential은 relativistic 옵션을 켜야하고, 이를 키지 않을시 문제가 생길 수 있다는 것을 확인할 수 있다.

#### 3. Work function

마지막으로 두 경우의 work function을 확인해보자. Work function을 구하기 위해서는 slab 모델을 만들어주어야 한다. 
<br><br>

**(1) Slab 모델은 위에서 구한 최적화된 bulk model을 이용**해서 [1, 1, 1]면을 표면으로 설정해줄 것이다. 
work function의 차이만 비교해보기 위해서 다른 변수들은 전부 고정해주었다.

<br>

**(2) 혹은 ase를 통해 직접 생성** 한다.

우선

```bash
$ase gui
```

를 입력하면 xcryden 창이 뜨게 된다.


<img src="../img/pseudopotential-10.png" width="300"/>

`setup-Surface slab`을 선택하면 원하는 구조의 slab의 .xyz 파일을 생성할 수 있다.


| <img src="../img/pseudopotential-11.png" width="400"/> | <img src="../img/pseudopotential-12.png" width="300"/> |
| --------------------------------------------------- | ---------------------------------------------------- |

원하는 `Element, Lattice Structure and Constant, Vacuum length`를 입력한 뒤 Apply


<img src="../img/pseudopotential-13.png" width="500"/>


`File-Save`를 눌러 원하는 directory에 `STRUCT.xyz` 로 저장하면 파일이 생성된다.


<img src="../img/pseudopotential-14.png" width="500"/>

생성된 `STRUCT.xyz`에는 Cell 정보가 vector 좌표로 표현되어 있으므로,
이를 

```bash
CELL   <size0>   <size1>   <size2>   <angle0>   <angle1>   <angle2>
```
과 같은 형식으로 바꾸어준다.  여기서 `size`는 각 벡터의 크기이고, `angle`은 각 벡터 사이의 각이다.


<br>

**(3) Tutorial-SIESTA 계산-A. 기초과정-(2) Au 벌크/슬랩 구조 에서 사용하는 make_bulk.py 파일을 이용**한다.
<br> ase 방법에서와 동일한 parameter들을 파일 안에서 수정할 수 있으니 확인하기 바란다.


<br>


<img src="../img/pseudopotential-15.png" width="600"/>

<br><br><br>

Slab 모델을 만든 후에는 basis를 설정해준다. work function을 구할 때는 100meV를 사용하면 reference와의 차이가 매우 크기 때문에 PAO.EnergyShift 항목을 50meV로 바꾸고 계산을 해야 한다.

|     Basis size     |      DZP       |
| :----------------: | :------------: |
| Basis energy shift |    50 [meV]    |
|         XC         |      LDA       |
|       DM.tol       | $10^{-3}$ [eV] |

slab model을 만들면 bulk model에서 했던 것과 마찬가지로 k-point test를 해야한다. slab model의 경우 z축의 k-point는 1로 고정하고 x, y값만 변화시켜가며 측정하면 된다. tutorial 1에서 slab model에 대한 k-point를 구했기 때문에 구했던 [31,31,1]의 k-point를 그대로 사용하겠다. relativistic 효과를 준 경우와 주지 않은 경우 work function을 측정해보면 결과는 다음과 같다.

|                  | Fermi [eV] | Vacuum [eV] | Work Function [eV] |
| :--------------: | :--------: | :---------: | :----------------: |
|   relativistic   | -4.506018  |  0.750869   |      5.256887      |
| non-relativistic | -4.201008  |  0.074519   |      4.275527      |

Metal work function의 [reference](https://public.wsu.edu/~pchemlab/documents/Work-functionvalues.pdf)에서 Au의 111면에서 work function은 5.31eV임을 알 수 있다. Relativistic의 경우 0.06 eV정도 차이가 나지만, non-relativistic의 경우 1.04 eV로 차이가 매우 심하게 나게 됨을 알 수 있다.

Au Tutorial에서 했던 것처럼 macroave.in을 통해 Wave function을 시각화 해보면 다음과 같은 결과를 얻을 수 있다.
Marcoave.in 은 /home/(username)/bin에 있다



<img src="../img/pseudopotential-16.png" width="700"/>

Vacuum level의 경우 그래프에서, 위치를 고정하지 않았던 쪽의 원자들에 가까운 쪽 level을 직접 그래프에서 측정하는 것이 더 정확하다.

<br>
이와 같은 일련의 과정들을 통해 Pseudopotential을 테스트할 수 있다. 테스트 결과 Au같은 원자번호가 큰 원자의 경우 Relativistic 효과를 고려하는 경우 reference와 거의 일치하지만 Relativistic 효과를 고려하지않는 경우는 reference와 큰 차이가 난다. 따라서 Au의 경우 relativistic 효과를 고려해줘야 한다는 것을 알 수 있다.

## 참고문헌
[1](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.26.1738): S. G. Louie, S. Froyen, and M. L. Cohen, Phys. Rev. B 26, 1738 (1982)  
[2](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.43.1494): Norm-Conserving Pseudopotentials, D. R. Hamann, M. Schlüter, and C. Chiang PRL (1970)  
[3](https://iopscience.iop.org/article/10.1088/0953-8984/13/42/307/meta): J. Phys.: Condens. Matter13 (2001) 9463–9470  
[4](https://link.springer.com/article/10.1007/BF03215471): Relativistic effects and the chemistry of gold (1998)  
[5](https://www.sciencedirect.com/science/article/pii/S0927025614007940#t0015): P Rivero et al. Comput. Mater. Sci. (2015), 98, 372
