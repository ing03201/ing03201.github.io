---
layout: page
title: VGA Text Mode
parent: Writing an OS in Rust
grand_parent: Book
permalink: /Book/Writing_an_OS_in_Rust/VGA_Text_Mode
---

# 1.3 VGA Text Mode

## VGA Text Buffer

VGA 텍스트 모드에서 화면을 출력할때 VGA 하드웨어의 텍스트 버퍼에 문자를 저장해야한다
텍스트 버퍼는 보통 25행 80열 크기의 2차원 배열이며, 해당 버퍼에 저장된 값들은 즉시 화면에 렌더링 된다. 배열의 각 원소는 화면에 출력될 문자들을 아래의 형식으로 표현한다

|비트|값|
|---|---|
|0-7|ASCII 코드|
|8-11|전경색|
|12-14|배경색|
|15|깜빡임 여부|

두 번째 바이트는 표현하는 문자가 어떻게 표시될 것인가를 정의한다.
첫 4비트는 전경색을 나타내고 그 다음 3비트는 배경색, 마지막 비트는 해당 문자의 blink 여부를 결정한다. 

