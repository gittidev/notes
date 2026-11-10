---
title: EXIF와 XMP
tags: [drone, metadata]
---

- **EXIF**: 표준 카메라 정보. 기종, 시각, 노출, 초점거리, GPS, 시리얼.
- **XMP drone-dji**: DJI 전용 정보. RelativeAltitude, AbsoluteAltitude, Gimbal Pitch/Yaw/Roll, 비행 자세·속도, RtkFlag·RtkStd, LRFTargetDistance, DewarpData, CalibratedFocalLength(픽셀 단위 초점거리), ShutterCount, SurveyingMode, UTCAtExposure 등.
- 읽는 법
  - Python Pillow 로 EXIF.
  - XMP 는 파일 앞부분(수백 KB)에서 `<x:xmpmeta>` 블록을 정규식으로 꺼낸다.
  - 가장 편한 건 `exiftool`.

```bash
exiftool -csv -r -XMP-drone-dji:all -EXIF:all ./photos > metadata.csv
```

관련: [[고도 종류]], [[RtkFlag]], [[GSD]]
