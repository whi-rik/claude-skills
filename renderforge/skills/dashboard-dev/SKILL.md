---
name: dashboard-dev
description: "RenderForge React 대시보드 개발 스킬. 페이지/컴포넌트 추가, API 연동, UI 수정. '대시보드', '버튼 추가', '페이지 만들어', '프론트', 'UI' 등 요청 시 사용."
---

# dashboard-dev

RenderForge React SPA 대시보드 개발 패턴.

## 기술 스택

- React + TypeScript
- React Query (상태관리)
- shadcn/ui (UI 컴포넌트)
- axios (API 클라이언트)
- Vite (빌드)

## 파일 구조

```
dashboard/src/
├── api/
│   ├── client.ts        — axios 인스턴스
│   ├── types.ts         — 응답 타입 정의
│   └── endpoints/       — API 함수
├── components/
│   ├── ui/              — shadcn 기본 컴포넌트
│   ├── layout/          — PageContainer 등
│   └── {domain}/        — 도메인별 컴포넌트
├── hooks/               — React Query 훅 (use*.ts)
├── pages/               — 페이지 컴포넌트
└── lib/                 — 유틸리티, 상수
```

## API 연동 패턴

```typescript
// 1. types.ts — 타입 정의
export interface MyResult { task_id: string }

// 2. endpoints/xxx.ts — API 함수
export async function myApi(params): Promise<MyResult> {
  const { data } = await client.post<ApiResponse<MyResult>>('/path', params)
  return data.data
}

// 3. hooks/useXxx.ts — React Query 훅
export function useMyMutation() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: (params) => myApi(params),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['key'] }),
  })
}
```

## 주의사항

- Django admin dashboard (`templates/admin/`) — 수정 불필요
- 대시보드 관련 수정은 항상 React SPA(`dashboard/src/`) 기준
- Celery 태스크 위임 API는 `{task_id}` 반환 → UI에서 task_id 표시
