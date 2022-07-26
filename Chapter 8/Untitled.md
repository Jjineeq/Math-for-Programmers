## Chapter 9 움직이는 물체 시뮬레이션하기
#### 이번 챕터에서는 소행성 게임을 만들것이다. 이 게임은 플레이어가 움직이는 소행성을 피하며 우주선 방향을 조종할 수 있어야 한다.
#### 이것을 구현하기 위해 앞에서 배운 미분적분학 개념을 사용할 것이다.
#### 하나씩 구현할 것인데 우선 소행성과 우주선의 x,y 좌표 변화를 시간에 따라 x(t), y(t)로 두며 시작할 것이다.
#### 시간에 대한 위치 함수의 도함수는 속도라고 하며, 시간에 대한 속도 함수의 도함수는 가속도라고 한다. 위치함수는 x,y 즉 2개이므로 속도 함수도 2개이고, 가속도 함수도 2개가 된다. 이것으로 속도와 가속도도 벡터로 생각 할 수 있다.
#### 소행성을 움직이기 위해 랜덤하게 선택된 등속 함수를 설정할 것이다.
#### 이후 오일러 방법이라는 알고리즘을 사용해 각 속도 함수를 실시간으로 적분해 프레임별로 소행성의 위치를 얻겠다.
#### 다음으로는 플레이어가 우주선을 제어할 수 있도록 만들 것이다.
#### 키보드에서 위쪽 방향키를 누르면 우주선은 진행방향으로 가속할 것이다. '가속'의 의미는 x(t)와 y(t) 각각의 도함수의 도함수가 0이 아님을 의미한다. 이것은 속도도 변화하고 위치도 변화한다는 것을 의미한다. 
#### 정리하자면 실시간으로 가속도 함수와 속도 함수를 적분하려고 오일러 방법을 사용하는 것이다.
#### 작성한 내용이 많은데 아래서 하나씩 작성된 순서에 맞게 구현하겠다.
  
  
---
## 9.1 속도가 고정된 움직임을 시뮬레이션하기
#### 일상에서는 속도와 속력을 동일한 의미를 가진 단어로 사용한다. 하지만 실질적으로는 속도는 속력과 이동방향이라는 두가지 개념을 포함하고 있다. 즉 속도는 벡터로 표현할 수 있다는 것이다.
#### 우리가 만들 게임은 2차원에서 표현할 것이다. 고로 위치의 순서쌍과 속도의 순서쌍을 다뤄야 한다.
#### 앞으로 x(t), y(t)를 위치 함수 순서쌍이라고 하고, x'(t), y'(t)를 속도 함수 순서쌍이라고 하겠다.
#### 각각을 벡터 값 함수 s(t) = (x(t),y(t)), v(t) = (x'(t),y'(t))로 쓰기도 한다. 
    
---
### 9.1.1 소행성에 속도 부여하기
#### 소행성에 속도벡터를 부여하기위해 PolygonModel 객체에 벡터의 두 성분 vx와 vy 속성을 추가하겠다.


```python
class PolygonModel():
    def __init__(self,points):
        self.points = points
        self.rotation_angle = 0
        self.x = 0
        self.y = 0 # -- 위에는 이전에 했던 부분과 동일
        self.vx = 0
        self.vy = 0 # vx, vy 속성은 vx = x'(t), vy=y'(t)의 현재 값을 저장하게 된다. 기본 값으로 0이 들어가 있고
        # 이것은 객체가 움직이지 않는것을 의미한다.
    
```


```python
# 소행성이 불규칙하게 움직이도록 소행성의 속도 성분에 랜덤 값을 부여하겠다,
class Asteroid(PolygonModel):
    def __init__(self):
        sides = randint(5,9)
        vs = [vectors.to_cartesian((uniform(0.5,1.0), 2 * pi * i / sides))
                for i in range(0,sides)]
        super().__init__(vs)
        self.vx = uniform(-1,1) # x,y의 속도는 -1부터 1사이의 랜덤 값으로 설정
        self.vy = uniform(-1,1)
```

---
### 9.1.2 소행성이 움직이도록 게임 엔진 업데이트하기
#### 프레임이 다음 프레임으로 전환되는 동안 시간이 t만큼 흘렀다면 x는 vx*t로, y는 vy*t로 갱신한다. -- 유량이 거의 변화하지 않을 때 부피의 작은 변화량을 구하는 근삿값 계산 방법과 동일
#### PolygonModel에서 move 메서드를 추가해보겠다.


```python
 def move(self, milliseconds):
        dx, dy = self.vx * milliseconds / 1000.0, 
        self.vy * milliseconds / 1000.0 # dx,dy 즉 x와 y의 변화량은 소행성 속도와 초단위 경과 시간을 곱해서 계산한다.
        self.x, self.y = vectors.add((self.x,self.y), (dx,dy)) # 변화량을 본래 위치에 더해 움직임을 처리한다.
```

#### 위에는 오일러 방법 알고리즘을 적용한 사례인데 이 알고리즘은 하나 이상의 함숫값을 계속 유지하며, 매 타임스텝마다 도함수에 따라 함수를 갱신한다.
#### 하지만 이 방법은 도함수가 일정하면 완벽하게 작동하지만 도함수가 변화하면 근삿값을 계산하는 한계가 있다.
---
### 9.1.3 화면에 소행성 유지하기
#### 소행성을 스크린 영역 내에 있도록 하려면 두 좌표를 최솟값과 최댓값 사이에 있도록 로직을 만들어야 한다. 
#### 예시를 사용하자면 x속성이 10에서 10.1로 증가하면 20을 빼서 -9.9로 보내는 것이다. 이런 작업은 스크린의 우측에서 좌측으로 텔레포트 시키는 효과가 있다. 


```python
def move(self, milliseconds):
        dx, dy = self.vx * milliseconds / 1000.0, self.vy * milliseconds / 1000.0
        self.x, self.y = vectors.add((self.x,self.y), (dx,dy)) # 위와 동일
        if self.x < -10: # 텔레포트하는 코드 벗어나는 지점에따라 다시 지정할 결과를 넣어줌
            self.x += 20
        if self.y < -10:
            self.y += 20
        if self.x > 10:
            self.x -= 20
        if self.y > 10:
            self.y -=20
```


```python
milliseconds = clock.get_time() # 게임 시작후 시간이 얼마나 경과했는가
for ast in asteroids: # 모든 소행성에 각 소행성의 속도에 따라 시간에 맞춰 위치 갱신
    ast.move(milliseconds)
```


```python
from asteroids import *
```


```python
main(asteroids=default_asteroids) # 실행하면 우주선이 움직이는 것을 볼 수 있다.
```

---
## 9.2 가속도 시뮬레이션하기
#### 로켓이 스러스터를 점화하면 우주선이 현재 진행 방향을 유지하며 일정한 가속도로 가속한다고 해보자
#### 가속도는 속도의 도함수라고 정의했으니, 일정한 가속도는 시간에 대해 x,y 방향으로 속도를 일정 비율로 변화시킨다. 
#### 가속도가 영벡터가 아니라면 속도 vx와 vy는 상수가 아니다.
#### 시간에 따라 변화하는 속도함수 vx(t),vy(t)가 있다고 하면 vx'(t) = ax, vt'(x) = ay를 만족하는 ax와 ay가 존재한다고 할 수 있고 가속도는 a =(ax,ay) 와 같은 벡터로 표현할 수 있다.
#### 플레이어가 아무런 키도 누르지 않으면 우주선의 가속도는 0이고 위쪽 화살표를 누르면 가속도가 갱신되어 (ax,ay)는 우주선의 진행 방향과 같은 벡터가 생성되어야 한다. 
---
### 9.2.1 우주선 가속화하기
#### 게임을 하던중 위쪽 화살표를 누르고 있게되면 우주선은 가속될 것이다. 하지만 우리가 항상 y축, x축의 방향으로 우주선을 조종하지는 않을 것이다. 즉 모든 방향에 대해 가속도로 표현할수 있어야 하는 것인데 이러기 위해 삼각함수를 사용하면 된다. 
#### 우주선이 x축을 기준으로 반시계방향으로 θ만큼 회전해 진행한다고 해보자
#### 이러면 수평 성분은 abs(a)*cos(θ)이 될것이고 수직성분은 abs(a)*sin(θ)가 될 것이다.  정리하면 가속도 벡터는 (abs(a)*cos(θ),abs(a)*sin(θ))과 같은 순서쌍으로 표현할 수 있다. 
#### 이제 위에 정리한 내용을 코드로 작성해보겠다.


```python
if keys[pygame.K_UP]: # key가 눌렸는지 확인
            ax = acceleration * cos(ship.rotation_angle) # 진행방향과 가속도 크기를 곱해 계산
            ay = acceleration * sin(ship.rotation_angle)
            ship.vx += ax * milliseconds/1000 # x,y 속도에 ax,y*t만큼 더해 값 갱신
            ship.vy += ay * milliseconds/1000
ship.move(milliseconds) # 갱신한 속도를 사용해 위치 이동
```

#### 위의 예시는 이계도함수가 x''(t) = vx'(t) = ax와 y''(t) = vy'(t) = ay일 떄 오일러 방법을 심화 적용한 것이다.
#### 각 타임스텝에 속도를 먼저 갱신한 뒤, move메서드에서 갱신한 속도를 사용해 위치를 갱신했다.
---
## 9.3 오일러 방법 깊게 살펴보기
#### 어떤 객체가 t=0시에 위치 (0,0)에 있고, 초기 속도가 (1,0)이며 가속도가 (0,0.2)로 일정하다고 해보자 초기 속도는 양의 x방향을 가리키고, 가속도는 양의 y방향을 가리킨다.
#### 이것은 객체가 곧바로 오른쪽으로 움직이기 시작하지만 시간이 흐르면서 위쪽으로 움직인다는 것을 의미한다.
### 9.3.1 오일러 방법 직접 해보기
#### 위에서 사용한 예시를 가지고 t가 0부터 10까지 증가할 때 2초마다 위치, 속도, 가속도의 상황을 보겠다.
#### 위치, 속도, 가속도 함수를 s(t), v(t), a(t)라고 표현하겠다. 함수들의 관계를 보면 s(t) = (x(t),y(t)), v(t) = (x'(t),y'(t)), a(t) = (x''(t), y''(t))로 나타낼 수 있다. 
#### 0초 일때는 위에서 본 것과 같이 s(0) = (0,0), v(0) = (1,0), a(0) = (0,0.02)라고 할 수 있다.
#### 그렇다면 2초일때는 어떻게 될까 직접 계산해보겠다.
#### 가속도는 일정하다고 했으니 넘어가고 속도를 보자 vx(2) = vx(0) + ax(0)*Δt 라고 할 수 있다. vy도 동일하게 vy(2) = vy(0) + ay(0)*Δt 가 된다. 계산해보면 vx(2) = 1, vy(2) = 0.4가 나오게 된다. 즉 v(2) = (1,0.4)이 된다.
#### 이어서 위치를 계산해보자
#### x(2) = x(0) + vx(0)*Δt = 2, y(2) = y(0)+vy(0)*Δt = 0이 된다. 
#### 오일러 방법을 사용해 실제로 구해보았다. 위와 동일한 절차로 계속 계산하면 t =4, 6, 8, 10일때 값을 모두 구할 수 있을 것이다.
   
---
### 9.3.2 파이선에서 오일러 방법 알고리즘 구현하기
#### 직접 구해봤으니 이제 파이썬으로 구현해보자.


```python
t = 0
s = (0,0)
v = (1,0)
a = (0,0.2)
# 관찰하려는 순간은 2,4,6,8,10초이다 즉 그 사이는 2초이고 총 5개를 관찰하면 된다.
dt = 2
steps = 5
```


```python
# 시간, 위치, 속도를 매 타임스텝마다 갱신해야 한다.
from vectors import add, scale
positions = [s]
for _ in range(0,5):
    t += 2
    s = add(s,scale(dt,v))
    v = add(v,scale(dt,a))
    positions.append(s)
```


```python
from draw2d import * 
draw2d(Points2D(*positions))
# 오일러 방법으로 계산한 객체의 시간별 궤적
```


    
![output_15_0](https://user-images.githubusercontent.com/100830660/181004041-d3b23d25-d3a5-487a-b682-8610b02136b0.png)
    


---
## 9.4 작은 타임스텝으로 오일러 방법 실행하기
#### 타입스텝을 두배 늘려 dt = 1, steps = 10으로 설정하고 계산, dt = 0.1, steps = 100으로 늘려 차이가 있는지 확인해 보겠다.


```python
# 확인을 위한 함수 코드
from draw2d import *
def pairs(lst): 
    return list(zip(lst[:-1],lst[1:])) # point를 찍기위해 리스트에서 zip으로 xy좌표 추출해 순서쌍 제작

def eulers_method(s0,v0,a,total_time,step_count): # 처음위치, 처음속도, 가속도, 측정할 시간, step 개수
    positions = [s0] # 초기 좌표값
    s = s0 # 초기위치
    v = v0 # 초기속도
    dt = total_time/step_count 
    for _ in range(0,step_count):
        s = add(s,scale(dt,v)) # 위치에 대해 변화한 속도를 반영해 위치 변경
        v = add(v,scale(dt,a)) # 속도에 대해 변화한 가속도를 반영해 속도 변경
        positions.append(s) # 좌표값에 추가
    return positions

approx5 = eulers_method((0,0),(1,0),(0,0.2),10,5)
approx10 = eulers_method((0,0),(1,0),(0,0.2),10,10)
approx100 = eulers_method((0,0),(1,0),(0,0.2),10,100)
approx1000 = eulers_method((0,0),(1,0),(0,0.2),10,1000)
```


```python
draw2d(
    Points2D(*approx5, color = 'C0'),
    *[Segment2D(t,h,color='C0') for (h,t) in pairs(approx5)],
    Points2D(*approx10, color = 'C1'),
    *[Segment2D(t,h,color ='C1') for (h,t) in pairs(approx10)])
```


    
![output_18_0](https://user-images.githubusercontent.com/100830660/181004050-2db06057-ce13-48d1-9013-9fe8050886b5.png)
    



```python
draw2d(
    Points2D(*approx5, color='C0'),
    *[Segment2D(t,h,color='C0') for (h,t) in pairs(approx5)],
    Points2D(*approx10, color='C1'),
    *[Segment2D(t,h,color='C1') for (h,t) in pairs(approx10)],
    *[Segment2D(t,h,color='C2') for (h,t) in pairs(approx100)],
    *[Segment2D(t,h,color='C3') for (h,t) in pairs(approx1000)],
    )
```


    
![output_19_0](https://user-images.githubusercontent.com/100830660/181004052-88c01b82-003a-46ed-8a0b-8de901a92d07.png)
    


#### 같은 방식으로 네가지 계산을 수행했지만 두개는 같지만 나머지와 다른 결과가 나왔다. 
#### 더 작은 dt(타임스텝이 커질수록)를 가질수록 y좌표 값이 더 커졌다.
#### 처음 2초를 확대해서 보면 왜 이런 결과가 나왔는지 알 수 있다.
#### 2초동안 타임스텝이 10개를 사용한 방법에서는 객체가 속도를 바꿀 기회를 얻어 한번 x축보다 위쪽에 위치한다. 하지만 5개의 타임스텝을 가진 방법에서는 가속되지 않았다. 또한 더 많은 타임스텝을 가질 수록 2초동안 더 많이 속도를 바꿀 기회를 얻어 속도 증가량을 많이 가지고 있게 된다.
#### 즉 위에서 구한 Δs = v * Δt는 속도가 상수일때만 맞게 된다는 것이다.
#### 시간 구간을 작게하면 각 구간의 속도가 그다지 변하지 않기에 오일러 방법이 좋은 근사값을 준다. 이를 확인하기 위해서는 dt를 매우 작은 값으로 두고(타임스텝을 많이 만들고) 실행하면 된다.
#### 정리하자면 타임스텝을 더 많이 주고 근사값을 계산할수록 결과는 이 값에 수렴하게 된다.
