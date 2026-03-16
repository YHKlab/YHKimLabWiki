Si-SiO<sub>2</sub>-Si junction model
=========================
## Contents
1. 수렴성 테스트


> Input: [Link](https://1drv.ms/u/s!ArHICXVqPANVuyG8V_dx7W3FLClr?e=lmyFcX)

## 수렴성 테스트

<center><img src="../../transiesta/img/si-sio2-si-01.png" width="80%" height="80%"></center>

이번 강의에서는 보다 정확하게 계산하기 위해서 필요한 사항에 대해서 논의해보기 위해 3차원의 Si-SiO<sub>2</sub>-Si 구조에 대해서 양자 수송 특성을 계산한다.

### Exercise 1: Buffer layer 테스트

Si-SiO<sub>2</sub>-Si junction model을 이용하여 buffer layer 테스트를 진행한다.

<center><img src="../../transiesta/img/si-sio2-si-02.png" width="80%" height="80%"></center>

#### Step 1: Electrode calculation
electrode의 unit cell 크기는 유지한 채로 buffer layer만을 조절하여, transmission을 확인한다. 이때, electrode의 unitcell은 principal cell이 되도록 크기를 설정하였다.

<center><img src="../../transiesta/img/si-sio2-si-03.png" width="30%" height="30%"></center>

```
$ vi STRUCT.fdf
NumberOfAtoms    32           # Number of atoms
NumberOfSpecies  1           # Number of species
%block ChemicalSpeciesLabel
 1 14 Si
%endblock ChemicalSpeciesLabel
LatticeConstant       1.000000000 Ang
%block LatticeVectors
    7.840000000     0.000000000     0.000000000
    0.000000000     7.840000000     0.000000000
    0.000000000     0.000000000    11.088000000
%endblock LatticeVectors
```
```
$ qsub slm_siesta_run
```
Left electrode와 Right electrode에 대해서 각자 DFT 계산을 진행하여 `.TSHS` 파일을 얻는다.

#### Step 2: Scattering region calculation

electode 계산에서 구한 `.TSHS` 파일을 이용하여 transiesta 계산을 진행한다.<br/>
```
$ cp ../1.elec_left/OUT/Left.TSHS input/.
$ cp ../2.elec_right/OUT/Right.TSHS input/.
```
Buffer layer 개수에 따른 구조는 다음과 같다.

<center><img src="../../transiesta/img/si-sio2-si-04.png" width="80%" height="80%"></center>

```
$ vi STRUCT.fdf
# No buffer layer                               # 3 buffer layers
NumberOfAtoms    122                            NumberOfAtoms    218
NumberOfSpecies  2                              NumberOfSpecies  2 
...                                             ...
%block LatticeVectors                           %block LatticeVectors
    7.8404   0.0000    0.0000000                7.8404   0.0000    0.0000000 
    0.0000   7.8404    0.0000000                7.8404   0.0000    0.0000000 
    0.0000   0.0000    56.001052                0.0000   0.0000   89.2731250
%endblock LatticeVectors                        %endblock LatticeVectors
...                                             ...

$ qsub slm _siesta_run
```

#### Step 3: Post-processing
TBTrans를 이용하여 transmission을 구한다.
```
$ cp OUT/siesta.TSHS input/.
$ qsub slm _siesta_run_tbt
```

<center><img src="../../transiesta/img/si-sio2-si-05.png" width="60%" height="60%"></center>


### Exercise 2: K-point 테스트
Si-SiO<sub>2</sub>-Si junction 모델은 3차원이므로 electrode 계산과 scattering 계산에서 k<sub>x</sub>, k<sub>y</sub>, k<sub>z</sub> k-points 증가시키며 transmission이 수렴되는 것을 확인한다.

- Electrode calculation
```
$ vi 1.elec_left/input/KPT.fdf
# 3 x 3 x 3 k-points
%block kgrid_Monkhorst_Pack
3   0   0   0.0
0   3   0   0.0
0   0   3   0.0
%endblock kgrid_Monkhorst_Pack

# 6 x 6 x 6 k-points
%block kgrid_Monkhorst_Pack
6   0   0   0.0
0   6   0   0.0
0   0   6   0.0
%endblock kgrid_Monkhorst_Pack

# 15 x 15 x 15 k-points
%block kgrid_Monkhorst_Pack
15  0   0   0.0
0   15  0   0.0
0   0   15   0.0
%endblock kgrid_Monkhorst_Pack
```

- Scattering region calculation
```
vi 3.total/input/KPT.fdf
# 3 x 3 x 1 k-points
%block kgrid_Monkhorst_Pack
3   0   0   0.0
0   3   0   0.0
0   0   1   0.0
%endblock kgrid_Monkhorst_Pack

# 6 x 6 x 1 k-points
%block kgrid_Monkhorst_Pack
6   0   0   0.0
0   6   0   0.0
0   0   1   0.0
%endblock kgrid_Monkhorst_Pack

# 15 x 15 x 15 k-points
%block kgrid_Monkhorst_Pack
15  0   0   0.0
0   15  0   0.0
0   0   1   0.0
%endblock kgrid_Monkhorst_Pack
```

Electrode, scatering region 계산 후 tbtrans를 통해 transmision을 비교하면 다음과 같다. 

<center><img src="../../transiesta/img/si-sio2-si-06.png" width="60%" height="60%"></center>

### Exercise 3: Basis 테스트

위에서 구한 수렴된 k-points (`6 x 6 x 6`, `6 x 6 x 1`)를 이용하여 Basis size에 따라 transmission 그래프를 확인하여 적절한 basis를 찾는다. 계산시 electrode, Transiesta, TBtrans 계산 모두 동일한 basis size로 해야한다. input 폴더안의 `BASIS.fdf` 파일에서 옵션을 바꿔가며 baiss test를 진행한다.
``` 
 $ vi BASIS.fdf
 PAO.BasisSize    SZ          # SZP or DZ, DZP, TZP
```
basis에 따른 transmission 그래프는 다음과 같다.  

<center><img src="../../transiesta/img/si-sio2-si-07.png" width="60%" height="60%"></center>



