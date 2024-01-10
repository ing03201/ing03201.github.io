---
layout: page
title: 코멘토 SoC 개발 1주차
parent: 일상
permalink: /일상/코멘토 SoC 개발 1주차
---
# 커널 / 빌드루트 / QEMU
cpio the root filesystem 
Compression method : gzip
linux kernel 전통적으로 gcc, 안드로이드 llvm clang 사용
# GIC
Generic Interrupt Controller
존재 이유 : Linux 첫 부팅시 Timer가 Interrupt를 걸면서 Context Switching을 해주기때문
