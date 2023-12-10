---
layout: page
title: A Freestanding Rust Binary
parent: Writing an OS in Rust
grand_parent: Book
permalink: /Book/Writing_an_OS_in_Rust/A_Freestanding_Rust_Binary
---

# 1.1 A Freestanding Rust Binary

## 소개

운영체제 커널을 만들려면 다음을 사용하지 않고 구현해야한다.
- 스레드
- 파일
- 동적메모리
- 네트워크
- 난수생성기
- 표준 출력
- 기타 운영체제의 추상화
- 특정 하드웨어 기능을 필요하는 것

## Rust 표준 라이브러리 링크 해제

### ***no_std*** 속성

```Shell
cargo new blog_os_post1_2018 --bin --edition 2018
```

```Rust
#![no_std]

fn main(){
	println!("Hello, world!");
}
```

이 코드를 실행하게 되면 println 매크로를 제공하는 표준 라이브러리를 링크하지 않게 되어 에러를 낸다.


## ***Panic*** 시 호출되는 함수 구현

> 컴파일러는 **panic**이 일어날 경우 ***panic_handler*** 속성이 적용된 함수가 호출되도록 합니다.
> 표준 라이브러리는 **panic**시 호출되는 함수가 제공되지만,
>  ***no_std*** 환경에서는 우리 **panic** 시에 호출 될 함수를 직접 설정해야함
```Rust
use core::panic::PanicInfo;

#[panic_hanlder]
fn panic(_info: &PanicInfo) -> !{
	loop{}
}
```
### ***PanicInfo*** 인자
> 패닉 일어난 파일명, 패닉 일어난 코드 줄번호, 전달된 메시지 를 가진 구조체

위 panic 함수는 절대 반환하지 않기때문에 ***never*** 타입! 을 반환

### ***eh_personality*** Language Item 

Language item이란?
컴파일러가 내부적으로 요구하는 특별한 함수 및 타입
> ex) ***Copy*** trait는 어떤 타입들이 [copy semantics]()를 가지는지 컴파일러에게 알려줌.
> Copy trait 구현코드에 ***#[lang="copy"]*** 속성을 이용하여 language item으로 선언을 확인할수있다.

임의로 구현한 Language item 유의 사항
- language item 구현 코드는 매우 자주 변경되어 불안정
- 컴파일러가 타입 체크를 하지않음
-> 따라서 language item 오류를 고치는 방법 ***eh_personality*** language item

***eh_personality*** Language item : Stack unwinding 구현하는 함수

Stack unwinding 해제하는 방법
```toml
[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

## ***Start*** 속성
Rust 표준 라이브러리를 링크하는 파일의 실행 과정
1. crt0(C runtime zero)에서 실행이 시작됨
2. start language item으로 지정된 Rust 런타임 실행 시작 함수를 호출한다.
3. Rust 런타임 초기화 후 main 함수가 호출

현재 freestanding 파일은 Rust 런타임과 crt0에 접근할 수 없기에 직접 실행 시작 지점을 지정해야함.

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

/// 패닉이 일어날 경우, 이 함수가 호출됩니다.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
// name mangling 해제, Rust 컴파일러가 `_start` 함수를 만들도록 함
#[no_mangle] 
// Rust 함수 호출 규약이 아닌 C 함수 호출 규약사용
pub extern "C" fn _start() -> ! {
    loop {}
}
/*
! 리턴 타입 : diverging function.
시작지점 함수는 운영체제 or 부트로더에 의해 호출.종료시 exit 시스템콜로 종료
*/

```

`#[no_mangle]` 속성 : name mangling 해제, Rust 컴파일러가 `_start` 함수를 만들도록 함
- `#[no_mangle]` 속성 없으면 실제 함수 이름을 이상한 이름으로 바꿔 생성


## 링커 오류

위 코드는 링커에 C 런타임 링크 하지말라 해야함
### Bare Metal 시스템 목표로 빌드
`target triple`: 여러 시스템 환경들을 표현하기 위함
```Shell
rustup target add thumbv7em-none-eabihf
```

### 링커 인자

링커에 특정 인자들을 추가하여 오류를 해결하는 방법

<details>
<summary>링커 인자</summary>
#### Linux
```
error: linking with `cc` failed: exit code: 1
  |
  = note: "cc" […]
  = note: /usr/lib/gcc/../x86_64-linux-gnu/Scrt1.o: In function `_start':
          (.text+0x12): undefined reference to `__libc_csu_fini'
          /usr/lib/gcc/../x86_64-linux-gnu/Scrt1.o: In function `_start':
          (.text+0x19): undefined reference to `__libc_csu_init'
          /usr/lib/gcc/../x86_64-linux-gnu/Scrt1.o: In function `_start':
          (.text+0x25): undefined reference to `__libc_start_main'
          collect2: error: ld returned 1 exit status
```
기본적으로 C런타임 실행 루틴 내에 `_start` 함수가 있음.
하지만 `no_std` 속성 사용하여 `libc`를 링크 하지않으므로 에러가 난다.
해결하는 방법은 `--nostartfiles`추가한다
```Shell
cargo rustc -- -C link-arg=nostartfiles
```
#### Windows

```
error: linking with `link.exe` failed: exit code: 1561
  |
  = note: "C:\\Program Files (x86)\\…\\link.exe" […]
  = note: LINK : fatal error LNK1561: entry point must be defined
```
`entry point must be defined` : 링커가 실행 시작지점을 찾을 수 없다.
Windows의 경우 기본 실행 시작 지점 이름이 subsystem에 따라 다름
- `CONSOLE` 서브시스템의 경우 링커가 mainCRTStartup 
- `WINDOWS` 서브시스템의 경우 링커가 WinMainCRTStartUp

```Shell
cargo rustc -- -C link-args="/ENTRY:_start /SUBSYSTEM:{subsystem}"
## {subsystem}은 console | windows 로 하면된다
```


#### macOS

1. 실행 시작 지점 함수 기본값 설정
```
error: linking with `cc` failed: exit code: 1
  |
  = note: "cc" […]
  = note: ld: entry point (_main) undefined. for architecture aarch64
          clang: error: linker command failed with exit code 1 […]
```
링커가 실행 시작 지점 함수의 기본값 `main`을 찾지 못했다 는 메시지 
- macOS에서는 모든 함수들의 이름 맨 앞에 `_` 문자가 앞에 붙음(무슨 이유인지 모름)
따라서 실행 시작 지점 함수 이름을 `_start`로 새롭게 지정하자
```Shell
cargo rustc -- -C link-args="-e __start"
```
`-e`인자 : 실행시작지점 함수이름 설정
macOS에서는 모든 함수 이름앞에 추가로 `_`문자가 붙으므로 하나 더 추가함

2. static 링크 실행파일 만들기
- macOS는 기본적으로 모든 프로그램이 `libSystem`라이브러리를 링크하도록 요구함
따라서 `-static`인자를 추가해야함
```Shell
cargo rustc -- -C link-args="-e __start -static"
```

3. crt0 링크 해제하기
- macOS에서 기본적으로 `crt0`를 링크하도록함
```Shell
cargo rustc -- -C link-args="-e __start -static -nostartfiles"
```

#### 플랫폼 별 빌드 명령어들을 하나로 통합하기

위에서 살펴본 대로 호스트 플랫폼 별로 상이한 빌드 명령어가 필요한데, 
`.cargo/config.toml` 이라는 파일을 만들고 플랫폼 마다 필요한 상이한 인자들을 명시하여 
여러 빌드 명령어들을 하나로 통합할 수 있다.

```toml
# in .cargo/config.toml

[target.'cfg(target_os = "linux")']
rustflags = ["-C", "link-arg=-nostartfiles"]

[target.'cfg(target_os = "windows")']
rustflags = ["-C", "link-args=/ENTRY:_start /SUBSYSTEM:console"]

[target.'cfg(target_os = "macos")']
rustflags = ["-C", "link-args=-e __start -static -nostartfiles"]
```
</details>

## 추가 설명
- Stack Unwinding
- crt0
	- C 프로그램 환경 설정, 초기화하는 런타임 시스템
	- 스택을 만들고 프로그램에 주어진 인자를 레지스터에 배치
- Rust 런타임 실행 시작 함수
	- Stack overflow Guard 초기화
	- Panic backtrace 정보 출력
## 모르는 것들
- trait
- copy semantics