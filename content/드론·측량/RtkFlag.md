---
title: RtkFlag
tags: [drone, gnss]
---

| 값 | 상태 | 오차 |
|---|---|---|
| 50 | 고정해(Fixed) | 1~3 cm |
| 34 | Float | 수십 cm |
| 16 | 단독 GPS(Single) | 수 m |

- 단독(16) 구간은 AbsoluteAltitude 가 수 m 씩 틀어질 수 있다. RelativeAltitude·LRF 거리는 영향이 없다.
- RtkStdLat/Lon/Hgt = 위치 표준편차(m). 고정해면 보통 0.002~0.007.
- 미션 전체 사진의 RtkFlag 비율을 먼저 세어 보면 PPK 가 필요한지 바로 알 수 있다.

관련: [[RTK와 PPK]], [[EXIF와 XMP]], [[고도 종류]]
