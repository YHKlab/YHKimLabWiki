Graphene (2D)
=========================
## Contents
1. Graphene의 양자수송 특성 계산

> Input: [Link](https://1drv.ms/u/s!ArHICXVqPANVux6jWW0x2QlTFre0?e=88oKbL)

## Graphene의 양자수송 특성 계산

<center><img src="../../transiesta/img/graphene-01.png" width="80%" height="80%"></center>

이번 장에서는 2차원 물질인 그래핀에 대해서 양자수송 특성을 계산한다.


### Step 1: Electrode calculation

<center><img src="../../transiesta/img/graphene-02.png" width="30%" height="30%"></center>

Electrode에 대해 DFT 계산을 통해 `elec.TSHS` 를 얻는다. 우리의 모델의 경우 x축 방향으로 periodic 구조를 가지지 않는다. 때문에 x방향으로의 k-point를 1로 지정해주었다. Eleectrode 계산 시에 사용된 k-point가 추후에 scattering 계산을 통한 양자수속 특성 계산의 정확도 큰 영향을 미치기 때문에 반드시 **k-point에 대해 수렴성 테스트**를 진행해야한다. 특히 양자 수송 방향(z-축)으로 더욱 엄격히 k-point를 주어야 보다 정확한 전극의 self-energy를 얻을 수 있다.

```
$ vi KPT.fdf
%block kgrid_Monkhorst_Pack
 1    0    0    0.0
 0   60    0    0.0
 0    0   35    0.0
%endblock kgrid_Monkhorst_Pack
```
electrode 계산시, `SolutionMethod`는 `diagon`으로 설정되어야 하며, `RUN.fdf`에 `TSHS`파일을 저장하기 위한 옵션을 추가하였다.
```
$ vi RUN.fdf
SolutionMethod      diagon
TS.HS.Save .true.

$ qsub slm_siesta_run
``` 

계산이 완료되면 output 폴더에서 `elec.TSHS` 파일을 확인할 수 있다.

### Step 2: Scattering region calculation
앞서 구한 `elec.TSHS` 파일을 NEGF 계산할 input 폴더로 복사한 후 transiesta를 통해 `.TSHS` 파일을 구한다.
```
$ cp ../1.Electrode_k060/OUT/elec.TSHS input/.
```
수송 방향인 z축 방향으로의 k-point는 1이어야 한다. 나머지 k-point는 반드시 electrode의 `.TSHS` 파일을 계산할 때 사용한 k-point와 일치해야한다.


```
$ vi KPT.fdf
block kgrid_Monkhorst_Pack
 1    0    0    0.0
 0   60    0    0.0
 0    0    1    0.0
%endblock kgrid_Monkhorst_Pack
```
Electrode 계산과 다르게, Scattering 영역 계산 시 NEGF를 이용하기 때문에, SolutionMethod는 diagon이 아닌 transiesta이어야 한다. 계산 과정에서, SIESTA SCF cycle이 먼저 수행되고 난 뒤 TS SCF cycle이 수행되어 SCF DM을 얻을 수 있다.
```
$ vi RUN.fdf
SolutionMethod      transiesta
```
NEGF 계산에서는 electrode 영역과 voltage에 대한 정보를 담고 있는 `TS.fdf`로 옵션을 조절한다.
```
$ vi TS.fdf
TS.Voltage    0.00000 eV
%block TS.Elec.Left
  HS elec.TSHS
  chem-pot Left
  semi-inf-dir -a3
  elec-pos begin 1
  used-atoms 4
%endblock TS.Elec.Left
%block TS.Elec.Right
  HS elec.TSHS
  chem-pot Right
  semi-inf-dir +a3
  elec-pos end -1
  used-atoms 4
%endblock TS.Elec.Right
```
TranSIESTA를 실행하여 scattering 영역에 대한 `.TSHS` 파일을 얻는다. 
```
$ qsub slm_siesta_run
```
### Step 3: Post-processing

TBTrans를 이용하여 transmission function을 구한다.<br/>앞서 구한 `scat.TSHS` 파일이 input으로 필요하며, 그 외 모든 input은 Step 2와 동일하다. 이때 실행 파일은 transiesta가 아닌 tbtrans이다.
```
$ cp ../2.Graphene_k060/OUT/scat.TSHS input/.
```
Tbtrans 계산시, TS.fdf에서 관련 옵션을 조절한다.<br/> DOS과 transmission 분석을 위한 에너지 범위를 지정할 수 있다. 
```
$ vi TS.fdf
%block TBT.Contour.neq
  part line
   from  -5.00000 eV to    5.00000 eV
    points    501
     method mid-rule
%endblock TBT.Contour.neq
```

```
$ qsub slm_siesta_run_tbt
```
파이썬 코드를 이용하여 transmission function을 시각화해본다.
```
$ python show_trans_rev.py scat.TBT.AVTRANS_Left-Right
```

예제 파일에 포함되어 있는 show_trans.py는 k vector가 단위 벡터인 경우에만 작동하는 코드이므로,
kgrid에 1이 아닌 값을 준 경우 아래와 같이 함수 부분을 수정해준다.

```python
#show_trans_minsu.py
def get_xy_array(filename):

    lines = open(filename).readlines()
    x = []; y = []; y_temp = []
    weight = 0
    firstLoop = 1
    y_index = 0
    i = 0

    for line in lines:
        data=line.split()

    #refresh and continue at  empty line
        if data == []:
            firstLoop = firstLoop - 1
            i = 0
            continue

    #find weight for each k vector
        if  (data[0]  == '#') and (len(data) > 3) :
            weight = data[ len(data) - 1]

    #
        if firstLoop == 0 :
            if data[0] != '#':
                x.append(float(data[0]))
                y.append(float(data[1]) * float(weight))

        #sum y values with weight for each k vector
        else :
            if data[0] != '#':
                y[i] += ( float(data[1]) * float(weight) )
                i += 1

    x = np.array(x)
    y = np.array(y)

    return x, y

```

<center><img src="../../transiesta/img/graphene-03.png" width="60%" height="60%"></center>


## Exercise 1: K-point effect


Post-processing 단계에서 TBtrans 계산을 할때는 electrode나 scattering region에 대한 계산을 할 때 사용한 k-point와 다른 k-point 값을 사용해도 된다. 일반적으로 transmission 그래프을 "매끄럽게" 하기 위해 **post-processing** 단계는  k-point를 scattering 계산보다 더욱 크게 주는 것이 일반 적이다.<br/>
참고로 tbtrans 계산의 input인 `scat.TSHS` 파일은 k-point를 1x60x1일때 얻은 결과값이다.
```
$ vi KPT.fdf
%block kgrid_Monkhorst_Pack
 1    0    0    0.0
 0   240    0    0.0
 0    0    1    0.0
%endblock kgrid_Monkhorst_Pack
```

<center><img src="../../transiesta/img/graphene-04.png" width="60%" height="60%"></center>

k-point를 증가시키자 transmission 그래프가 매끄러워진 것을 확인할 수 있다. 

## Exercise 2: Channel width effect
폭이 2배 넓은 모델의 T(E) 과 원래 T(E) 그래프를 비교해 보자.

<center><img src="../../transiesta/img/graphene-05.png" width="60%" height="60%"></center>

electrode의 구조가 달라졌으므로, electrode calculation부터 진행해야한다.<br/> STRUCT.fdf 파일을 간략하게 보면, atom 개수와 lattice vector의 y축 길이가 늘어난 것을 확인할 수 있다.
```
$ vi 3.Electrode_wider_k30/input/STRUCT.fdf
NumberOfAtoms    8           # Number of atoms
NumberOfSpecies  1           # Number of species
LatticeConstant       1.000000000 Ang
%block LatticeVectors
   20.000000000     0.000000000     0.000000000
    0.000000000     4.901704000     0.000000000
    0.000000000     0.000000000     4.245000000
%endblock LatticeVectors
...

$ vi 4.Graphene_wider_k30/input/STRUCT.fdf
NumberOfAtoms    48           # Number of atoms
NumberOfSpecies  1           # Number of species
LatticeConstant       1.000000000 Ang
%block LatticeVectors
   20.000000000     0.000000000     0.000000000
    0.000000000     4.901704000     0.000000000
    0.000000000     0.000000000    25.470000000
%endblock LatticeVectors
...
```
또한 real space 상에서 unit cell이 y축 방향으로 2배 늘어났으므로, y축의 k-point를 바꿔주었다.
```
$ vi 3.Electrode_wider_k30/input/KPT.fdf
%block kgrid_Monkhorst_Pack
 1    0    0    0.0
 0   30    0    0.0
 0    0   35    0.0
%endblock kgrid_Monkhorst_Pack

$ vi 4.Graphene_wider_k30/input/KPT.fdf
%block kgrid_Monkhorst_Pack
 1    0    0    0.0
 0   30    0    0.0
 0    0    1    0.0
%endblock kgrid_Monkhorst_Pack
```
<center><img src="../../transiesta/img/graphene-06.png" width="60%" height="60%"></center>

위와 같이 transmission 값이 두배가 되는 것을 확인할 수 있다.