# Function and Console I/O

### 함수
- 어떤 일을 수행하는 코드의 덩어리
- 반복적인 수행을 1회만 작성 후 호출
- 코드를 논리적인 단위로 분리
- 캡슐화: 인터페이스만 알면 타인의 코드 사용 가능


```python
'''
사용법
def 함수 이름(parameter1, ...):
    수행문
    수행문
    ....
    return 반환값
'''

def calculate_rectangle_area(x,y):
    result = x*y
    return result

x = 10
y = 20
print('x:', x)
print('y:', y)
print('넓이:', calculate_rectangle_area(x,y))
```

    x: 10
    y: 20
    넓이: 200
    

### 함수 수행 순서
- 함수 부분을 제외한 메인프로그램부터 시작
- 함수 호출시 함수 부분을 수행 후 되돌아옴
1. 메인 프로그램 수행
2. 함수 호출
3. 함수 수행
4. 메인 프로그램 수행

### Parameter and Argument
- parameter: 함수의 입력 값 인터페이스
- argument: 실제 parameter에 대입된 값


```python
# 파라미터와 반환값 유무에 따라 함수의 형태

# 파라미터 존재, 반환값 존재
def f(x):
    result = x + 10
    return result

print('f(10):', f(10))


# 파라미터 X, 반환값 존재
def f():
    return 100

print('f():', f())


# 파라미터 존재, 반환값 X
def f(s):
    print(s + "!")

f('Hello')

    
# 파라미터 X, 반환값 X
def f():
    print("Hi")

f()

```

    f(10): 20
    f(): 100
    Hello!
    Hi
    

### Console I/O
- input() 함수로 입력을 받음
- print() 함수로 출력


```python
print('enter anything :')
anything = input()
print('input:', anything)
```

    enter anything :
    apple
    input: apple
    


```python
# 한번에 여러 번 입력 받기
a, b = input().split() # split안에 나누는 기준을 입력할 수 있음
print('first:', a)
print('second:', b)
```

    apple banana
    first: apple
    second: banana
    


```python
num = input()
print(num)
print(type(num), end='\n\n')

# 일반적으로 입력은 문자열로 받는다
# 숫자형을 입력받고 싶으면 형변환을 해주면 된다

num = int(input())
print(num)
print(type(num))
```

    10
    10
    <class 'str'>
    
    10
    10
    <class 'int'>
    

### print formatting
- %string(%s, %d, %c. %f, %o, %x 등이 존재)
- format함수
- fstring


```python
# %string
print('%%string\nI have %s. My score is %d' %('grape', 99), end = '\n\n')

# format
print('format함수\n{} {}'.format('a', 13), end = '\n\n')
```

    %string
    I have grape. My score is 99
    
    format함수
    a 13
    
    

###### %string의 %f
- %글자수.소수점f
- 이런식으로 글자수와 소수점 몇째자리까지 나타낼지 정할 수 있다.(반올림이 적용됨)


```python
print('%5.3f' %(3.493921)) 
```

    3.494
    

###### format함수에서 소수점
- print({:.소수점자리수f}.format())
- 반올림 적용됨


```python
print('{:.3f}'.format(3.493921))
```

    3.494
    

### Padding
- 여유 공간을 지정하여 글자배열 + 소수점 자릿수 맞추기


```python
print('Product: %10s, Price per unit: %10.3f' %('Banana', 3.4939))

# format함수에서 <. >로 정렬방향 선택 가능
print('Product: {0:<10s}, Price per unit: {1:>10.3f}'.format('Banana', 3.4939))
```

    Product:     Banana, Price per unit:      3.494
    Product: Banana    , Price per unit:      3.494
    

### f-string
- print(f"{변수명}")
- print(f"{변수명:공간 수}")
- print(f"{변수명:>}") : 오른쪽 정렬
- print(f"{변수명:<}") : 왼쪽 정렬
- print(f"{변수명:^}") : 가운데 정렬
- print(f"{변수명:*<}:") : 왼쪽 정렬후 빈칸은 *로 채움


```python
name = 'orange'

print(f'My name is {name}')
print(f'My name is {name:20}')
print(f'My name is {name:-<20}')
print(f'My name is {name:*>20}')
print(f'My name is {name:#^20}')

# 소수점 다루기(반올림 적용됨)
num = 3.149592
print(f'{num:.2f}')
```

    My name is orange
    My name is orange              
    My name is orange--------------
    My name is **************orange
    My name is #######orange#######
    3.15
    
