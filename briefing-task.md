# 아침 브리핑 수집 프롬프트

Claude Code 클라우드 스케줄 작업(`morning-briefing` routine, 매일 오전 7시 KST)이 실행하는 원문.
수정할 때는 **이 파일만 고치면 됨** — routine은 매번 이 파일을 새로 읽어서 실행하므로 별도로
routine 설정을 바꿀 필요 없음.

---

오늘 아침 브리핑을 만들어줘. 오늘 날짜는 셸에서 직접 확인해(`date` 명령 등). 다음 3개 카테고리로
최신 뉴스를 웹서치해서 수집해:

1. 채용·HR·조직문화
2. 국내 대기업 사업동향
3. AI·테크 전반

각 카테고리당 헤드라인 3~5개. 각 항목은:
- headline (제목)
- summary (2문장 이내 요약)
- source (출처 매체명)
- url (기사 링크)
- hrComment (선택, HR 관점 코멘트 — 있으면만 추가)

전체 분량은 5분 내 읽을 수 있는 정도로 제한해줘. 해당 카테고리에 특별한 소식이 없으면
items를 빈 배열로 둬도 돼.

다 모았으면 아래 스키마의 JSON으로 구조화해:

```json
{
  "date": "YYYY-MM-DD",
  "categories": [
    {"name": "채용·HR·조직문화", "items": [...]},
    {"name": "국내 대기업 사업동향", "items": [...]},
    {"name": "AI·테크 전반", "items": [...]}
  ]
}
```

**저장 방법 (중요):**
이 저장소(현재 체크아웃되어 있는 `newsletter-claude`) 루트의 `latest-briefing.json` 파일 내용을
위 JSON으로 완전히 덮어써. 그 다음 아래 순서로 커밋하고 push까지 반드시 끝내:

```
git add latest-briefing.json
git commit -m "브리핑 업데이트: YYYY-MM-DD"
git push origin main
```

push까지 성공해야 웹앱(GitHub Pages)에 반영돼. push가 실패하면(권한 오류 등) 그 에러 메시지를
그대로 결과에 남겨줘 — 사용자가 원인을 파악할 수 있어야 해.
