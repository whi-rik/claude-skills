---
name: render-analysis
description: "RenderForge 렌더링 데이터 분석 스킬. 무음률, RMS, stem 구성, VST 성능, 워커별 통계, GM 커버리지 분석 및 보고서 작성. '무음 분석', '통계', 'RMS', '보고서', '현황 확인', '수량 확인' 등의 요청 시 사용."
---

# render-analysis

RenderForge 렌더링 데이터를 분석하고 보고서를 작성한다.

## 분석 기준

- 모든 무음률은 **RMS 체크 완료(`rms_status='done'`)** 데이터만 사용
- status 기반이 아닌 RMS 기반이 정확한 무음 판정

## 쿼리 패턴

### 벌크 쿼리 (필수)
```python
# Good: 한번에 집계
TB_330.objects.filter(...).values('queue_type').annotate(
    done=Count('render_job_id', filter=Q(status='done')),
    silent=Count('render_job_id', filter=Q(status='silent')),
)

# Bad: 해시별 개별 쿼리 (N+1)
for h in hashes:
    TB_330.objects.filter(track__midi__hash=h)...
```

### 다이나믹 적용 구분
```python
# 원본
TB_330.objects.exclude(midi_path__endswith='_r.mid')
# 다이나믹 적용
TB_330.objects.filter(midi_path__endswith='_r.mid')
# 재렌더 테스트 (_rf)
TB_330.objects.filter(wav_path__contains='_rf')
```

### GM remap 적용
```python
remap = {g.program_number: g.remap_program or g.program_number for g in TB_200.objects.all()}
```

## 보고서 작성

- 경로: `debug/{report_name}.md`
- 형식: 마크다운, 표 사용
- 필수 항목: 작성일, 기준(RMS done 등), 전체 현황, 세분화 분석, 결론/대책
- 수치: 절대값 + 비율(%) 병기

## Stem 분류 참조

메모리 `reference_stem_classification.md` — 13개 stem, 125개 GM 악기 매핑.
