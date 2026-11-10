---
title: DJI 파일 종류
tags: [drone, dji]
---

DJI 매핑 미션을 마치면 SD 카드에 이런 파일이 남는다.

- `DJI_YYYYMMDDhhmmss_NNNN_V.JPG`: 광각 가시광 사진. NNNN 은 미션 안 순번.
- `_Z.JPG`: 줌 카메라. 기체에 따라 `_V` 로 저장되기도 한다.
- `_T.JPG`: 열화상. 열화상 카메라가 달린 기체만.
- `_0002_D.MRK/.OBS/.RTK/.NAV`: 사진이 아니라 PPK 데이터. 그래서 사진 순번에 0002 가 비어 있다. `ppk_file_name.pbk` 는 이 파일들의 이름 목록.
- `.MP4`: 동영상. 압축·블러가 심해 학습용으로는 잘 안 쓴다.

관련: [[RTK와 PPK]], [[PPK 파일 MRK NAV OBS RTK]], [[JPG와 RAW]]
