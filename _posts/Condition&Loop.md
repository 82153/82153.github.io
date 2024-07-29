---
layout: single
title: "[Python Basic] Condition & Loop"
categories: Python_Basic
---
# 조건문과 반복문

### 조건문
- 조건에 따라 특정한 동작을 하게하는 명령어
- 조건문은 <span style = 'color:#0000FF'>조건을 나타내는 기준</span>과 <span style = 'color:#0000FF'>실행해야 할 명령</span>으로 구성
- if, else, elif등의 예약어 사용


```python
'''
if [조건]: 
    수행문

else:
    수행문
'''

score = 90
if score >= 70:
    print('Pass')
else:
    print('Fail')
```

    Pass
    

### 조건 연산자
- <, >, <=, >= : 대소 비교
- ==, is : 같은지 비교
- !=, is not : 다른지 비교
- is와 is not은 <span style = 'color:#0000FF'> 메모리 주소</span>를 비교한다.
- 1은 True, 0은 False
- list가 비어있으면 False, 비어있지 않으면 True


```python
a = [1,2,3,4,5]
b = a[:]
print('값만 복사하여 is로 비교\n',a is b)

b = a
print('메모리 주소를 같게 복사\n', a is b)
```

    값만 복사하여 is로 비교
     False
    메모리 주소를 같게 복사
     True
    


```python
# -5 ~ 256까지는 같은 메모리 주소를 같게 한다
a = 30
b = 30
print('범위 내 비교\n',a is b)

a = -6
b = -6
print('범위 밖 비교\n',a is b)
```

    범위 내 비교
     True
    범위 밖 비교
     False
    

### 논리 연산자
- and: 모든 조건이 True여야 True
- or: 하나라도 True라면 True
- not: True는 False로, False는 True로 반대로 바꿈


```python
a = True
b = True

print('and:',a and b)
print('or:',a or b, end = '\n\n')

a = True
b = False
print('and:',a and b)
print('or:',a or b)
```

    and: True
    or: True
    
    and: False
    or: True
    

###### 조건이 여러개일 경우
- and -> all
- or -> any


```python
bool_ls = [True, False, False, True]
print('all:',all(bool_ls))
print('any:', any(bool_ls))
```

    all: False
    any: True
    

### 삼항 연산자
- 조건문을 사용하여 참일 경우와 거짓일 경우의 결과를 한줄로 표현


```python
val = 100
tmp = 'Pass' if val >= 70 else 'Fail'
print(tmp)
```

    Pass
    

###### 예제1
- 점수가 주어졌을 때, 학점 계산


```python
score = 83
if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
else:
    grade = 'F'
print('83점:',grade)
```

    83점: B
    

###### 예제2
- 태어난 연도를 계산하여 학교 종류를 맞추는 프로그램


```python
year = int(input())

age = 2024 - year + 1

if age <= 26 and age >= 20:
    school = '대학생'
elif age < 20 and age >= 17:
    scool = '고등학생'
elif age < 17 and age >= 14:
    scool = '중학생'
elif age < 14 and age >= 8:
    school = '초등학생'
else:
    school = '학생이 아닙니다.'

print(school)
```

    2000
    대학생
    

### 반복문
- 정해진 동작을 반복적으로 수행하게 하는 명령문
- 반복문은 <span style = 'color:#0000FF'>반복 시작 조건, 종료 조건, 수행명령</span>으로 구성됨
- for, while등의 명령어를 사용

###### for문
- 반복 범위를 지정하여 반복문 수행
- 반복 실행 횟수를 명확히 알 때 사용


```python
'''
for [변수] in [반복범위]:
    수행문
'''

for i in [1,2,3,4,5]:
    print(f'Hi {i}')
```

    Hi 1
    Hi 2
    Hi 3
    Hi 4
    Hi 5
    


```python
# range 사용
# range(시작점, 끝점, 간격)
for i in range(0,5): # 끝 범위는 제외
    print(f'Hi {i}')
    
print()
for i in range(1,10, 2):
    print(f'odd {i}')
```

    Hi 0
    Hi 1
    Hi 2
    Hi 3
    Hi 4
    
    odd 1
    odd 3
    odd 5
    odd 7
    odd 9
    


```python
# 문자열의 경우 한글자씩 처리
for i in 'abcde':
    print(f'{i}')
```

    a
    b
    c
    d
    e
    

###### while문
- 조건이 만족하는 동안 반복문 수행
- 반복 실행 횟수가 명확하지 않을 때 사용


```python
i = 1

while i < 10:
    print(f'now_i: {i}')
    i+=1
```

    now_i: 1
    now_i: 2
    now_i: 3
    now_i: 4
    now_i: 5
    now_i: 6
    now_i: 7
    now_i: 8
    now_i: 9
    

###### 반복문 제어
- break: 특정 조건에서 반복 종료
- cotinue: 특정 조건에서 남은 반복 명령 skip


```python
for i in range(10):
    if i == 5:
        break
    print(f'break {i}')
```

    break 0
    break 1
    break 2
    break 3
    break 4
    


```python
for i in range(10):
    if i == 5:
        continue
    print(f'continue {i}')
```

    continue 0
    continue 1
    continue 2
    continue 3
    continue 4
    continue 6
    continue 7
    continue 8
    continue 9
    

###### 예제1
- 구구단 계산기


```python
print('구구단 몇단을 계산할까요?')

dan = int(input())
print(f'구구단 {dan}단을 계산합니다.')

for i in range(1,10):
    print(f'{dan} X {i} = {dan*i}')
```

    구구단 몇단을 계산할까요?
    8
    구구단 8단을 계산합니다.
    8 X 1 = 8
    8 X 2 = 16
    8 X 3 = 24
    8 X 4 = 32
    8 X 5 = 40
    8 X 6 = 48
    8 X 7 = 56
    8 X 8 = 64
    8 X 9 = 72
    

###### 예제2
- 1 ~ 100 사이의 임의의 숫자 맞추기


```python
import random
ans = random.randint(1,100)
while True:
    guess = int(input())
    if guess > ans:
        print('숫자가 너무 큽니다.')
    elif guess < ans:
        print('숫자가 너무 작습니다.')
    elif guess == ans:
        print(f'정답입니다. 정답은 {ans}입니다.')
        break
```

    50
    숫자가 너무 큽니다.
    25
    숫자가 너무 큽니다.
    12
    숫자가 너무 큽니다.
    6
    정답입니다. 정답은 6입니다.
    

###### 예제3
- 사람별 평균을 구하라


```python
kor_score = [49,79,20,100,80]
math_score = [43,59,85,30, 90]
eng_score = [49,79,48,60,100]
midterm_score = [kor_score, math_score, eng_score]

student_avg = [0,0,0,0,0]
i = 0
for kor, math, eng in zip(kor_score, math_score, eng_score):
    student_avg[i] = (kor+math+eng) / 3
    i+=1
print(student_avg)
```

    [47.0, 72.33333333333333, 51.0, 63.333333333333336, 90.0]
    
