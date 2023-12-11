---
layout: page
title: A Minimal Kernel
parent: Writing an OS in Rust
grand_parent: Book
permalink: /Book/Writing_an_OS_in_Rust/A_Minimal_Kernel
---

# 1.2 A Minimal Kernel

## 컴파일 대상 환경 설정
```json
{
    "llvm-target": "x86_64-unknown-none", // 운영체제 대상
    "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
    "arch": "x86_64", // 컴파일 대상 ISA
    "target-endian": "little",
    "target-pointer-width": "64",
    "target-c-int-width": "32",
    "os": "none",
    "executables": true,
    "linker-flavor": "ld.lld",
    "linker": "rust-lld",
    "panic-strategy": "abort", // stack unwinding 지원하지 않으므로 패닉시 즉시 종료
    "disable-redzone": true, // red zone 스택 포인터 최적화 기능 해제 -> 
    "features": "-mmx,-sse,+soft-float" // 컴파일 대상 환경 기능등
}
```

## disable-redzone

### Red Zone
![[Pasted image 20231210103012.png]]
n개의 지역 변수를 가진 함수 스택 포인터
함수가 호출되었을 때, 함수의 반호나 주소 및 지역변수들을 스택에 저장할 수 있게 스택포인터 값 조정
**Red Zone**은 조정된 스택 포인터아래의 128바이트 메모리 구간을 가르킨다.
함수가 또 다른 함수를 호출하지 않는 구간에서만 사용하는 임시 데이터의 경우, 함수가 이 구간에 해당 데이터를 저장하는데 이용될 수 있다.  따라서 스택 포인터를 조정하기 위해 필요한 명령어 두 개를 생략할 수 있는 상황이 종종 있다.( 다른 함수를 호출 하지 않는 함수)

하지만 이 최적화 기법을 사용하는 도중 exception 혹은 hardware intterupt가 일어날 경우 큰 문제가 생긴다. 함수가 red zone을 사용하는 도중 예외가 발생한 상황을 가정해보자

![[Pasted image 20231210104439.png]]
CPU와 exception handler가 red zone에 있는 데이터를 덮어씁니다.
하지만 이 데이터는 interrupt된 함수가 사용중이다. 따라서 exception handler로 부터 반환되어 다시 interrupt 함수가 계속 실행하게 되었을때, 변경된 red zone의 데이터로 인해 함수가 오작동 할 수 있다.
이런 현상으로 인해 디버깅하는데 몇 주씩 걸리는 이상한 버그를 발생시킬수도 있다.

## features : mmx, sse soft-float
### Single Instruction Multiple Data(SIMD)
- CPU에서 지원되는 명령어 셋으로 하나의 명령어로 동일한 형태 / 구조의 여러 데이터를 한번에 처리하는 병렬처리기법
- x86/amd64 
	- **MMX, SSE,** SSE2, AVX, AVX2, AVX512F
- ARM
	- NEON
#### SISD VS SIMD
![[Pasted image 20231210110912.png]]
- N개의 32비트 정수 A,B 덧셈 연산을 하는 예시
(a) SISD(Single Instruction Single Data)
- 3개의 레지스터에 N번 연산
(b) SIMD(Single Instruction Multiple Data)
- 모든 변수를 한번에 처리
```javascript
// SISD
var a = [1, 2, 3, 4];
var b = [5, 6, 7, 8];
var c = [];

c[0] = a[0] + b[0];
c[1] = a[1] + b[1];
c[2] = a[2] + b[2];
c[3] = a[3] + b[3];
c; // Array[6, 8, 10, 12]


// SIMD
var a = SIMD.Float32x4(1, 2, 3, 4);
var b = SIMD.Float32x4(5, 6, 7, 8);
var c = SIMD.Float32x4.add(a,b); // Float32x4[6, 8, 10, 12]
```
행렬, 벡터 계산에 SIMD가 효율적. 따라서 그래픽 드라이버 DirectX, OpenGL은 SIMD를 지원

#### 구현 방법
- 어셈블리 / Intrinsic func(inline assembly match)
- 요즘 최신 컴파일러들은 자체적으로 SIMD instruction 사용한 코드로 변환해줌
#### Aligned memory
```c--
short a, b, c, d;                 // (1) Not aligned
short a[4];                       // (2) Not Aligned
__declspec(align(32)) short a[4]; // (3) Aligned
```

![[Pasted image 20231210113627.png]]
메모리 시작 지점을 align 한 숫자들을 배수로 맞추고 내부 원소 하나하나의 크기를 align한 크기로 맞춤

#### Intrinsic Function
```
_mm_<intrin_op>_<suffix>
```
![[Pasted image 20231210114446.png]]

```c--
	const int n = 1000000000;
	__m128i a, b, r;
	__declspec(align(16)) short v1[8] = { 1, 2, 3, 4, 5, 6, 7, 8 };
	__declspec(align(16)) short v2[8] = { 8, 1, 7, 2, 6, 3, 5, 4 };
	__declspec(align(16)) short result[8];

  // SISD
	for (int i = 0; i < n; ++i) {
		result[0] = v1[0] > v2[0] ? v1[0] : v2[0];
		result[1] = v1[1] > v2[1] ? v1[1] : v2[1];
		result[2] = v1[2] > v2[2] ? v1[2] : v2[2];
		result[3] = v1[3] > v2[3] ? v1[3] : v2[3];
		result[4] = v1[4] > v2[4] ? v1[4] : v2[4];
		result[5] = v1[5] > v2[5] ? v1[5] : v2[5];
		result[6] = v1[6] > v2[6] ? v1[6] : v2[6];
		result[7] = v1[7] > v2[7] ? v1[7] : v2[7];
	}

	// SIMD
	for (int i = 0; i < n; ++i) {
		a = _mm_loadu_si128((__m128i *)v1);
		b = _mm_loadu_si128((__m128i *)v2);
		r = _mm_max_epi16(a, b);
		_mm_storeu_si128((__m128i *)result, r);
	}
```

![[Pasted image 20231210114510.png]]

#### soft-float기능 
일반 정수 계산만을 이용해 부동소수점 계산을 SW에서 모방
