---
layout: page
title: 코멘토 SoC 개발 1주차
parent: 일상
permalink: /일상/코멘토SoC/1주차
---
# 커널 / 빌드루트 / QEMU
cpio the root filesystem 
Compression method : gzip
linux kernel 전통적으로 gcc, 안드로이드 llvm clang 사용
# GIC
Generic Interrupt Controller
존재 이유 : Linux 첫 부팅시 Timer가 Interrupt를 걸면서 Context Switching을 해주기때문

# CPU 파악하기
 arm사 IP 종류 cortext
 A - Application SoC CPU
 R - 실시간성 CPU, MMU 없음
 M - MCU

## Numa vs SMP

- Numa (Non-Uniform Memory Access) 
- SMP(Symmetric )
	- system bus에서 캐시를 사용해서 응답을 빠르게 할수있으나 coherency 문제 발생
		- https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/extended-system-coherency---part-1---cache-coherency-fundamentals
## PSCI 
- 사용하지 않는 불필요한 코어를 끄기
- 최근 정의된 것
http://www.iamroot.org/xe/index.php?mid=Note&document_srl=216084