---
title: GeoTIFF와 좌표계
tags: [gis, crs]
---

- GeoTIFF: 좌표계 정보가 붙은 TIFF. 픽셀↔위경도 변환이 된다. rasterio/GDAL로 읽는다.
- WGS-84(EPSG:4326): 위경도. 면적 계산에 맞지 않다.
- UTM 52N(EPSG:32652) 또는 한국 중부원점(EPSG:5186): m 단위 평면좌표. 면적은 여기서 잰다.
- 라벨 마스크도 GeoTIFF로 만들면 좌표가 유지된다.

관련: [[정사영상]], [[추론과 면적 계산]]
