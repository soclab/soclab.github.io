# 업적 소식 자동 생성

`main` 브랜치에 업적 JSON을 올리면 GitHub Actions가 변경된 항목만 확인해
`news.json`에 연구 게시글을 추가합니다. 국제논문은 매번 최신 10편을 기존 공지와
대조해 누락된 단계도 복구하지만, 과거 업적 전체를 다시 게시하지는 않습니다.
자동 커밋과 같은 실행에서 정적 사이트도 GitHub Pages에 배포합니다.

## 최초 1회 설정

저장소의 **Settings → Pages → Build and deployment → Source**를
**GitHub Actions**로 설정합니다. GitHub Actions가 만든 커밋은 기존의
`Deploy from a branch` 빌드를 다시 실행하지 않기 때문에 필요한 설정입니다.

저장소의 **Settings → Actions → General → Workflow permissions**에서
**Read and write permissions**도 허용해야 `news.json` 자동 커밋이 가능합니다.

## 국제논문

- 새 레코드가 `status: "accepted"`이고 `date`, `volume`, `issue`, `pages`가 비어
  있으면 Acceptance 게시글을 한 번 생성합니다.
- 이후 같은 `id`에 실제 `date`, `volume`, `issue`, `pages`를 모두 입력하면
  Publication 게시글을 별도로 생성합니다. `status`도 `published`로 바꾸는 것을
  권장하지만, 게시 판정은 실제 게재 정보 네 필드를 기준으로 합니다.
- 확정 DOI가 있는 `accepted` 논문은 매일 03:15(KST) 실행되는
  `Check publication DOIs daily`가 Crossref에서 해당 DOI를 조회해, 있는 값으로
  비어 있는 `volume`, `issue`, `pages`, `date`, `month`를 채웁니다. volume·pages·
  date가 모두 채워지면 `status`를 `published`로 바꾸고, Publication 게시글 생성과
  사이트 배포를 이어서 실행합니다. Crossref에 아직 정보가 없으면(Early Access)
  기다리며, 직접 입력한 값은 덮어쓰지 않습니다.
- 자동화 설치 전부터 `published`였던 최신 10편도 공지를 대조합니다. Publication
  글이 빠져 있으면 최초 실행에서 자동 생성합니다.
- 제목의 학술지 약칭은 내장된 표를 사용합니다. 새 학술지의 약칭을 직접 정하려면
  해당 논문 레코드에 `"journal_abbr": "약칭"`을 추가합니다.
- Acceptance의 게시 날짜와 본문 월은 JSON에 실제 게재일이 아직 없으므로 업적
  변경 커밋 날짜를 사용합니다.
- Publication의 게시 날짜는 공지가 생성된 날(한국 시간)입니다. 본문의 월은 실제
  게재월(`date`)을 사용합니다. 이미 생성된 공지의 게시 날짜는 바뀌지 않습니다.

예시:

```json
{
  "id": 238,
  "type": "international-journal",
  "status": "accepted",
  "year": 2026,
  "month": null,
  "title": "Paper title",
  "authors": ["First Author", "Sungho Kang"],
  "journal": "IEEE Transactions on Very Large Scale Integration Systems",
  "volume": "",
  "issue": "",
  "pages": "",
  "date": ""
}
```

발행 후에는 같은 레코드를 다음처럼 갱신합니다.

```json
{
  "status": "published",
  "volume": "34",
  "issue": "10",
  "pages": "3100–3112",
  "date": "2026.10"
}
```

## 국제학회

- `international_conferences.json`의 논문을 같은 연도·학회명끼리 묶어 학회마다
  게시글 하나를 만듭니다. 게시판은 연구, 카테고리는 행사(`event`)입니다.
- 제목은 `2026 ISOCC Presentation`, 본문은 `The following papers have been
  presented in ISOCC 2026`으로 시작하고 논문별 `Title`/`Authors`를 나열한 뒤
  `Congratulations to the authors`로 끝납니다. 논문이 한 편이면 `paper has`로
  씁니다.
- 학회 약칭은 학회명 괄호 안의 값(예: `(ISOCC 2026)`)을 사용합니다. 괄호가 없는
  학회는 레코드에 `"conference_abbr": "약칭"`을 추가하면 됩니다.
- 학회 개최일(`date`)이 지난 뒤, 개최일로부터 90일 안에 있는 학회만 새로 게시합니다.
  개최 전에 입력한 논문은 개최일 이후 첫 실행에서 게시되며, 오래된 학회는 소급
  게시하지 않습니다.
- 같은 학회 논문을 나중에 추가·수정하면 기존 자동 게시글 본문이 갱신됩니다.
  같은 제목의 수동 게시글이 이미 있으면 새로 만들지 않습니다.
- 게시 날짜는 게시글이 생성된 날(한국 시간)입니다.

## 특허

- `patents_domestic_granted.json` 또는 `patents_international_granted.json`에 새
  등록 특허를 추가하면 Patent 게시글을 한 번 생성합니다.
- 출원 JSON은 자동 게시 대상이 아닙니다.
- 제목의 영문 제1발명자명은 `members.json`의 한글/영문 이름 대응표를 사용합니다.
  대응되는 사람이 없으면 특허 레코드에 `"lead_inventor_en": "English Name"`을
  추가할 수 있습니다.
- 국제특허 본문은 기존 사이트와 동일하게 `World Intellectual Property Office`
  문구를 사용합니다.

## 중복 및 수정

- 자동 게시글에는 `source_key`가 저장되어 같은 단계의 글이 두 번 생기지 않습니다.
- 같은 업적에 대한 기존 수동 게시글도 제목과 본문을 확인해 중복 생성을 피합니다.
- 업적 JSON을 고치면 이미 자동 생성된 게시글의 제목과 본문도 갱신됩니다. 게시글
  번호, 조회수, 이미지, 첨부파일은 그대로 유지합니다.
- 실행 결과는 저장소의 **Actions → Sync achievement news** 로그에서 확인합니다.
