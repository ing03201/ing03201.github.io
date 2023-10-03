---
layout: page
title: SmartThings Edge Driver
parent: IoT
grand_parent: Tech
permalink: /Tech/IoT/SmartThingsEdgeDriver
---

# SmartThings Edge Driver
![[Pasted image 20231003161617.png]]
## 개요
>SmartThings Edge driver(스마트싱스 엣지 드라이버)는 [SmartThings](https://namu.wiki/w/SmartThings "SmartThings")의 제품의 타입을 식별하는 [SmartThings Edge](https://namu.wiki/w/SmartThings%20Edge "SmartThings Edge") 베이스의 소프트웨어 드라이버이다. 
> -나무위키 펌


요약하면 SmartThings 허브에 설치할 수 있는 드라이버를 뜻한다.
Work With SmartThings 인증을 받은 제품들, 
예를 들어 [Philips Hue]([SmartThingsEdgeDrivers/drivers/SmartThings/philips-hue at main · SmartThingsCommunity/SmartThingsEdgeDrivers (github.com)](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers/tree/main/drivers/SmartThings/philips-hue)), [Bose]([SmartThingsEdgeDrivers/drivers/SmartThings/bose at main · SmartThingsCommunity/SmartThingsEdgeDrivers (github.com)](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers/tree/main/drivers/SmartThings/bose)) [sonos]([SmartThingsEdgeDrivers/drivers/SmartThings/sonos at main · SmartThingsCommunity/SmartThingsEdgeDrivers (github.com)](https://github.com/SmartThingsCommunity/SmartThingsEdgeDrivers/tree/main/drivers/SmartThings/sonos)) 등은 공식으로 허브에서 드라이버를 지원한다.

*사용하려면 필수적으로 SmartThings Hub가 있어야한다.*
## 커스텀 드라이버 만들기
> SmartThings 계정, CLI, 개발 환경설정을 다 한 상태를 가정하고 작성합니다.

