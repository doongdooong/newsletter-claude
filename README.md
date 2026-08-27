# 아침 브리핑 데스크

매일 아침 관심 분야(채용·HR·조직문화 / 국내 대기업 사업동향 / AI·테크) 뉴스를
자동 수집·요약해서, 아이폰 홈 화면에서 앱처럼 열어보는 개인용 도구.

## 시스템 구조 (전부 이 저장소 하나로 통합)

1. **생성**: Claude Code 클라우드 스케줄 작업(routine)이 매일 오전 7시(KST) 실행 → 웹서치로 뉴스 수집 → JSON으로 구조화
   (프롬프트 원문: [briefing-task.md](./briefing-task.md))
2. **저장**: 결과를 이 저장소의 `latest-briefing.json` 파일로 커밋 & push
3. **표시**: `index.html` (GitHub Pages)이 같은 저장소의 `latest-briefing.json`을 `fetch`로 읽어와 카드 UI로 렌더링

> 2026-08-27 이전에는 Google Drive에 데이터를 저장하고 웹앱이 Drive API(+ API 키)로 읽어오는
> 3단계(Claude 앱 Scheduled Task / Google Drive / GitHub Pages) 구조였음. 관리 지점이 세 군데로
> 흩어져 있고 수정이 번거로워서, 전부 이 GitHub 저장소 + Claude Code 스케줄 하나로 합침.
> Drive, API 키, 공유 권한 설정이 전부 필요 없어짐. (자세한 배경은 아래 트러블슈팅 이력 참고)

## 배포

- **호스팅**: GitHub Pages, 이 저장소의 `index.html`
- **URL**: https://doongdooong.github.io/newsletter-claude/
- **접근**: 아이폰 Safari → 홈 화면에 추가 (PWA, 네이티브 앱 아님)

## 자동화 (Claude Code 스케줄)

- claude.ai의 [routines](https://claude.ai/code/routines)에 `morning-briefing`이라는 이름으로 등록되어 있음
- 매일 07:00 KST(cron `0 22 * * *`, UTC 기준)에 클라우드에서 실행
- 실행 내용: 이 저장소를 체크아웃 → `briefing-task.md` 지시대로 뉴스 수집·JSON 작성 → `latest-briefing.json`을 커밋하고 push
- 스케줄 시간이나 실행 내용을 바꾸려면: Claude Code에서 `/schedule` (또는 "스케줄 수정해줘") 라고 말하면 됨

## JSON 스키마

- 카테고리당 헤드라인 최대 2개 (조건에 맞는 기사가 없으면 그 카테고리 자체가 배열에서 빠짐)
- 실행 시점 기준 최근 72시간 이내 발행된 기사만 포함 (선정 규칙 전체는 [briefing-task.md](./briefing-task.md) 참고)

```json
{
  "date": "YYYY-MM-DD",
  "categories": [
    {
      "name": "채용·HR·조직문화 | 국내 대기업 사업동향 | AI·테크 전반",
      "items": [
        {
          "headline": "string (후속 업데이트인 경우 \"[업데이트] \" 접두어)",
          "summary": "string (2문장 이내, 업데이트는 1문장)",
          "source": "string",
          "url": "string",
          "hrComment": "string (선택)"
        }
      ]
    }
  ]
}
```

## 디자인 컨셉

- 뉴스룸/디스패치 보드 느낌의 다크 테마
- 폰트: Noto Serif KR(본문), JetBrains Mono(태그·날짜 등)
- 카테고리별 색상 태그: HR=teal, 산업동향=amber, AI=violet
- 로컬 아카이브: `localStorage`에 날짜별로 브리핑 저장 (서버 DB 없음)

## 트러블슈팅 이력

- GitHub Public 저장소에 API 키가 노출되어 보안 경고 수신 → 키 폐기 후 재발급, 최소 권한으로 제한 (2026-08-27 Drive 방식 폐기로 API 키 자체가 필요 없어져 이 문제는 해소됨)
- 403 (리퍼러 차단) → 원인: 브라우저가 빈 리퍼러로 요청 → 메타 태그 시도했으나 iOS Safari에서 미해결
- 400 (Bad Request) → 원인: API 키 만료 → 재발급으로 해결
- **(2026-08-27) 브리핑이 하루씩 밀려 보임** → 원인: Google Drive에 매일 "파일 찾아서 없으면 생성" 방식으로 저장했는데, 실제로는 매번 새 파일이 생성되고 있었고 새 파일은 공개 공유 권한이 없어 웹앱이 계속 옛날 고정 ID의 파일(전날 데이터)만 보여주고 있었음
  → 해결: Drive를 아예 제거하고, 데이터를 이 GitHub 저장소 안에 직접 저장하는 구조로 전환. Claude 앱 Scheduled Task + Drive 웹 + GitHub 웹 3곳에 흩어져 있던 관리 지점을 이 저장소 + Claude Code 스케줄 하나로 통합

## 다음에 할 만한 작업 후보

- [ ] 오프라인에서도 마지막 브리핑이 보이도록 서비스워커 캐싱 추가 (선택)
- [ ] 아카이브 UI 개선 (현재는 `localStorage` 기반이라 기기 변경 시 기록 소실)
- [ ] (완료 후 정리) 기존 Claude 앱 Scheduled Task 삭제, Google Cloud의 옛 API 키 폐기, Drive의 옛 `latest-briefing.json` 파일 정리
