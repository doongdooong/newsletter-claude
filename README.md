# 아침 브리핑 데스크

매일 아침 관심 분야(채용·HR·조직문화 / 국내 대기업 사업동향 / AI·테크) 뉴스를
자동 수집·요약해서, 아이폰 홈 화면에서 앱처럼 열어보는 개인용 도구.

## 시스템 구조 (3단계 파이프라인)

1. **생성**: Claude Code 스케줄 작업이 매일 오전 7시(KST) 실행 → 웹서치로 뉴스 수집 → JSON으로 구조화
   (프롬프트 원문: [briefing-task.md](./briefing-task.md))
2. **저장**: 결과를 Google Drive의 **고정 파일**(`latest-briefing.json`, 아래 파일 ID)에 `update`로 덮어쓰기
   - 파일 ID: `1o04rrBn-KDCIO-g5Tn1MEXQi8D81xcBW`
   - ⚠️ 반드시 이 ID의 파일을 **update**(내용만 교체)해야 함. 새 파일을 생성하면 안 됨 —
     기존 새 파일 생성 방식은 공유 권한이 상속되지 않아 웹앱이 계속 옛날 파일을 보는 버그가 있었음 (2026-08-27 발견, 아래 트러블슈팅 참고).
3. **표시**: `index.html` (GitHub Pages)이 Google Drive REST API로 그 파일을 읽어와 카드 UI로 렌더링
   - `https://www.googleapis.com/drive/v3/files/{FILE_ID}?alt=media&key={API_KEY}`

## 배포

- **호스팅**: GitHub Pages, 이 저장소의 `index.html` (파일명 하나로 통일됨, 과거 `index_1/2/3.html` 정리 완료)
- **URL**: https://doongdooong.github.io/newsletter-claude/
- **접근**: 아이폰 Safari → 홈 화면에 추가 (PWA, 네이티브 앱 아님)

## 데이터 소스 연결

- Google Drive 파일을 "링크가 있는 모든 사용자 → 뷰어"로 공개 설정 (이 설정은 파일을 최초 생성할 때만 하면 됨. 이후로는 같은 파일을 계속 `update`하므로 재설정 불필요)
- 웹앱은 Google Cloud **API 키**로 Drive REST API 직접 호출 (`index.html` 상단에 하드코딩)
- API 키 설정: "API 제한사항 = Google Drive API만", "애플리케이션 제한사항 = 없음"
  (HTTP 리퍼러 제한은 iOS Safari 호환성 문제로 포기함)

## JSON 스키마

```json
{
  "date": "YYYY-MM-DD",
  "categories": [
    {
      "name": "채용·HR·조직문화 | 국내 대기업 사업동향 | AI·테크 전반",
      "items": [
        {
          "headline": "string",
          "summary": "string (2문장 이내)",
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

- GitHub Public 저장소에 API 키가 노출되어 보안 경고 수신 → 키 폐기 후 재발급, 최소 권한으로 제한
- 403 (리퍼러 차단) → 원인: 브라우저가 빈 리퍼러로 요청 → 메타 태그 시도했으나 iOS Safari에서 미해결
  → 리퍼러 제한 해제, "Drive 읽기 전용" 권한으로만 방어
- 400 (Bad Request) → 원인: API 키 만료 → 재발급으로 해결
- **(2026-08-27) 브리핑이 하루씩 밀려 보임** → 원인: 매일 "파일 찾아서 없으면 생성" 방식으로 동작하면서 실제로는 매번 새 파일이 생성되고 있었고, 새 파일은 공개 공유 권한이 없어 웹앱이 계속 옛날 고정 ID의 파일(전날 데이터)만 보여주고 있었음
  → 해결: 파이프라인을 Claude Code 스케줄로 이전하면서, 항상 같은 파일 ID를 `update`(덮어쓰기)하도록 변경. 관리 주체를 Claude 앱 Scheduled Task + Drive 웹 + GitHub 웹 3곳에서 이 저장소(Claude Code) 하나로 통합

## 다음에 할 만한 작업 후보

- [ ] 오프라인에서도 마지막 브리핑이 보이도록 서비스워커 캐싱 추가 (선택)
- [ ] 아카이브 UI 개선 (현재는 `localStorage` 기반이라 기기 변경 시 기록 소실)
- [ ] Google Cloud Console에서 API 키 제한사항이 여전히 "Drive API만"으로 걸려있는지 주기적 확인
