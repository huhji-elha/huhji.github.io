---
layout: post
title:  "[DevOps: Golang] 기본 자료형 톺아보기"
date:   2021-09-13 23:44:14 +0530
categories: DevOps Golang 
use_math: true
---
☄ Golang의 특징에 따른 자료형과 변수 선언하기!

_____________________________________



# Golang - 자료형

### Go 언어 자료형은 다음과 같다.

1. **bool** — true, false를 저장
2. **string** — 문자, 문자열을 저장
3. **int** (+ int8, int16, int32, int64) <br>
**uint** (unsigned int + uint8, uint16, uint32, uint64) <br>
**uintptr —** uint와 같은 크기를 갖는 포인터 자료형
    1. int, uint, uintptr 타입은 32 비트 시스템에서는 32비트 길이이고, 64비트 시스템에서는 64비트 길이이다.
    2.  특별히 정수의 크기나 부호(unsigned)를 지정할 이유가 없다면 `int`를 쓰면 된다.


    💡 부호 비트가 있는 정수를 signed, 부호 비트가 없는 정수를 unsigned라고 한다.
    기본적으로 C에서 정수형변수는 signed이다.

    ```go
    //uint64 예시(unsigned long long int)
    // <<와 >>는 비트 이동 연산자이다. 정수형에만 사용된다.
    var max_int uint64 = 1<<64 - 1 
    ```

4. **byte** — unit8과 같다. 8bit 자료형을 나타낸다.
5. **rune** — int32와 같다. 유니코드 저장을 위한 자료형을 나타낸다.
6. **float32** — IEEE-754 32비트 부동소수점. 7자리 정밀도를 갖는다. <br> **float64** — IEEE-754 64비트 부동소수점. 12자리 정밀도를 갖는다.
7. **complex64**, **complex128** — float32(or 64) 크기의 실수부와 허수부로 구성된 복소수

    
# 
### Go는 묵시적 형변환을 하지않는다.

따라서, 타입이 맞지 않으면 값을 할당할 수 없다. 이때는 변환할 값과 같은 타입으로 형 변환을 해주어야 한다. `int`와 `uint` 의 변환도 불가하다.

```go
var (
	int_sample int
	float_sample float64
)

func main() {
	// int에서 float64로 묵시적 형번환을 할 수 없다.
	float_sample = int_sample

	// 다른 타입을 변수에 할당하려면 다음과 같이 형 변환을 해주어야한다.
	float_sample = float64(int_sample)
}
```

# 
### 선언만 하고 변수를 초기화하지 않으면 zero value로 초기화된다.

Golang에서 zero value는 변수의 타입에 따라 다음과 같이 나뉜다.

- 숫자 타입 : `0`
- boolean 타입 : `false`
- string 타입 : `""`

```go
//zero value 예시
var (
	i int // zero value = 0
	f float64 // zero value = 0
	b bool // zero value = false
	s string // zero value = ""
)
```

# 
### Golang의 상수 선언

상수 선언은 변수 선언의 `var` 대신 `const` 를 사용한다.

상수가 될 수 있는 타입으로는 **character**, **string**, **boolean**, **numeric** 타입이 있다.

상수와 변수는 다음의 3가지 차이점이 있다.

1. 상수는 값을 변경할 수 없다.
2. 숫자형 상수는 var로는 표현할 수 없는 범위를 저장하는 등 수를 정밀하게 표현할 수 있다.
3. 타입을 지정하지 않은 상수는 맥락에 따라 타입이 변한다. (묵시적 형변환이 가능한가?)
4. 상수 선언은 const를 명시해야하기 때문에 `:=` 로는 선언할 수 없다.

```go
import "fmt"

// 1
const Pi1 float32 = 3.14
const Pi2 = 3.14 // 선언과 동시에 초기화하면 타입을 지정하지 않아도 된다.
func main() {
	Pi2 = 64 // cannot assign to Pi2 에러가 발생한다.
}

// 2
var Big_var = 1<<100 // overflow 에러가 발생한다.
var Big_const = 1<<100

// 3
const small_const = Big_const >> 99 // 2^1이 된다.
func needInt(x int) int { return x*10 + 1 }
func main() {
	fmt.Println("needInt(small_const) : ", needInt(small_const))
} // small_const 상수 선언 시 타입을 지정하지 않았기 때문에 int 형으로 자동 변환된다.
```

Reference: 

- [Programmers 30분 Go](https://programmers.co.kr/learn/courses/13)
- [C 프로그래밍: 현대적 접근](https://wikidocs.net/26940)