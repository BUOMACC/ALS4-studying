# ALS4-studying

언리얼 엔진의 `ALS4` 프로젝트에 대해 궁금한 내용들을 다룹니다.  

문서 내용은 향후 더 추가될 수 있습니다.  

> This project is based on my personal analysis and may contain inaccuracies.

<a name="table-contents"></a>
## 목차
> 1. [8-Way-Movement](#section_01)
> 2. [발 미끄러짐 방지](#section_02)


<a name="section_01"></a>
## 1. 8-Way-Movement
<hr>

ALS에서는 시선을 정면을 유지한채 이동하는 `8-Way-Movement` 모드를 지원한다.

### Animation List
<hr>

|<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_F.png" width="100%" height="100%"/>|<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_B.png" width="100%" height="100%"/>|<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_LF.png" width="100%" height="100%"/>|<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_LB.png" width="100%" height="100%"/>|<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_RF.png" width="100%" height="100%"/>|<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_RB.png" width="100%" height="100%"/>|
|  --------------------------  |--------------------------|--------------------------|--------------------------|--------------------------|--------------------------|
|(F) 앞으로 달리는 동작    |(B) 뒤로 달리는 동작    |(LF) 앞으로 달리는 도중 왼쪽으로 달리는 동작    |(LB) 뒤로 달리는 도중 왼쪽으로 달리는 동작    |(RF) 앞으로 달리는 도중 오른쪽으로 달리는 동작    |(RB) 뒤로 달리는 도중 오른쪽으로 달리는 동작|

애니메이션은 총 6개를 사용하며 각 방향에 따른 앞 / 뒤 / 좌 / 우 이동 동작으로 구성된다.  
또한 애니메이션을 보다시피 8방향으로 이동하는 이름이 무색하게 대각선 이동이 보이지 않는다.  


### 8방향 이동의 구현 - 대각선 처리
<hr>

ALS4 에서는 `Blending`과 캐릭터를 회전시켜 대각선 이동을 구현했다.  
예를 들어, 뒤 우측 대각선(RB)으로 이동한다고 했을때 뒤(B) 이동과 우측(R) 이동을 `Blending`하여 대각선 움직임을 구현한다.  

하지만 지속적으로 뒤 우측 대각선(RB) 애니메이션을 재생하지 않고 캐릭터를 회전시키고 뒤(B)로 이동하는 애니메이션을 재생하여 자연스럽게 전환되도록 구현되었다.  


<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_Rotating.png" width="50%" height="50%"/>   

> 처음엔 대각선 이동을 하지만, 이후에는 캐릭터가 회전되고 뒤(B)로 이동하는 애니메이션으로 전환된다.


### 어떻게 회전하는가?
<hr>

<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_Rotating_2.png" width="75%" height="75%"/>  

`Yaw` 값을 기반으로 캐릭터를 직접 회전시키는 방식으로, 뒤 우측 대각선(RB)를 기준으로 계산하자면 커브값의 입력(Time)으로 135.0이 들어오며 반환(Y)로는 -45.0 값이 `BYaw` 변수에 저장된다.

<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_Rotating_3.png" width="30%" height="30%"/>  

계산된 `BYaw` 값은 현재 애니메이션(Move B)의 `YawOffset` 커브값으로 저장한다.

<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_Rotating_4.png" width="60%" height="60%"/>  

마지막으로 저장된 값을 `UpdateGroundedRotation()` 내부에서 사용하여 캐릭터를 이동 방향의 반대 방향을 쳐다보도록 회전이 처리된다.  


### 방향별 BlendSpace 블렌딩 처리
<hr>

각각 방향별로 구현된 `BlendSpace`를 서로 블렌딩하는 과정에서 `VelocityBlend`라는 가중치 값을 사용하여 블렌딩한다.

<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_01/Image_BlendSpaces.png" width="75%" height="75%"/>  

위 이미지를 보듯이 `VelocityBlend` 값을 이용해 앞 / 뒤 / 좌 / 우 애니메이션들을 서로 혼합하여 적합한 애니메이션이 출력된다.


`VelocityBlend`는 다음과 같이 계산한다.
- 캐릭터 기준에서의 현재 이동 방향을 구한다. (Relative Velocity)
- Relative Velocity의 요소의 절댓값을 더한 값으로 나누어 요소의 총 합이 1이 되도록 만든다.
  > ex: 대각선 이동 (0.707, 0.707..) → (0.5, 0.5)로 변환

이렇게 구해진 `VelocityBlend`의 모든 요소(F, B, L, R)의 합은 항상 1이 되며, 이는 블렌딩의 `Alpha`값으로 사용하기 용이해진다.

[목차로 이동](#table-contents)

<br>

<a name="section_02"></a>
## 2. 발 미끄러짐 방지

ALS4에서는 이동속도와 애니메이션 속도의 불일치로 발이 미끄러지는 현상을 방지하기 위해 주로 보폭(`Stride`)과 재생속도(`PlayRate`)를 조절하여 이를 해결한다.

### StrideBlend
<hr>

<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_02/Image_Stride.png" width="70%" height="70%"/>  

`StrideBlend`는 구성된 `BlendSpace`에서 발의 보폭을 얼마나 사용할지 가중치를 의미한다.  
0인 경우 가만히 있는 `Idle` 포즈를 사용하고, 1인 경우 움직이는 애니메이션 `Walk 또는 Run`을 사용한다.  


<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_02/Image_StrideCurve.png" width="50%" height="50%"/>  

`StrideBlend` 값은 별도의 커브를 이용해서 속도(Time = `Speed`)에 따라 `0.2 ~ 1.0`) 범위 사이의 값을 반환한다.  
예를 들어, 달리기의 경우 초당 `350` 만큼의 속도를 가지고 이동할 것으로 만들어졌기에, 최대인 `350`에서는 `Stride` 값은 `1.0`이다.


### PlayRate
<hr>

재생속도(`PlayRate`)는 보폭(`Stride`)만으로는 발 미끄러짐을 해결할 수 없을때 ㅐ생속도를 늘려 커버하기 위한 장치다.  
예를 들어, 초당 `350`의 속도로 이동하는 애니메이션을 이용해 초당 `700`으로 이동하고 싶다면 애니메이션의 속도를 2배로 하면 된다.  

계산 방식은 단순하다. 현재 속도(`Speed`)를 애니메이션이 호환하는 속도(달리기의 경우 350)로 나누어 `PlayRate`를 구한다.  
속도가 절반(`175`)인 경우에는 `PlayRate`는 `0.5`, 반면에 두배(`350`)인 경우에는 `PlayRate`는 2가 될 것이다.  


### 대각선 이동의 문제
<hr>

기존 `StrideBlend`와 `PlayRate`를 이용해 발 미끄러짐을 일부 해결하더라도 대각선 이동에서는 한 가지 문제점이 있다.  
대각선에서 이동을 처리하기 위해 `Normalized`된 `Vector`값을 사용하는데, 이 경우 오른쪽 대각선으로 이동한다고 가정하면 `(0.707, 0.707)` 같은 입력이 나온다.  

<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_02/Image_Diagonal.png" width="80%" height="80%"/>  

> 정면 이동시 (375, 0)의 속도를 가진다.


<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_02/Image_Diagonal_2.png" width="80%" height="80%"/>  

> 대각선 이동시 (265.165, 265.165)의 속도를 가지며 크기는 약 375로 정면 이동과 동일한 속도다.


하지만, 앞에서 구한 `VelocityBlend`값을 이용해 `0.5`씩 블렌딩하면 생각한것과 다르게 (375 * 0.5, 375 * 0.5)의 속도로 이동하는 애니메이션이 되어버린다.  
이때 발의 보폭(`Stride`)과 속도(`Speed`)를 조절하기에는 같은 움직임에도 불구하고 부자연스럽게 보이게 될 것이다.  
- 달리는 상태의 발의 보폭(`Stride`)는 이미 최대값(1.0 = 375의 속도 커버)으로, 더 이상 증가시킬 수 없다.
- `PlayRate`를 이용하기에는 대각선으로 이동하는 애니메이션만 속도가 빨라보일 것이다.  


<img src="https://github.com/BUOMACC/ALS4-studying/blob/main/Images/Section_02/Image_Diagonal_3.png" width="100%" height="100%"/>  

ALS 에서는 이 문제를 해결하기 위해 달리는 상태에 `Foot IK Bone`의 스케일 대각선 이동 배수(약 1.414배)만큼 스케일하여 기존 이동과 보폭이 일치하도록 처리했다.

[목차로 이동](#table-contents)
