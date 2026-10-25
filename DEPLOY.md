# 휴대폰에서 확인하기 — GitHub Pages

`hktower226-lang/hiring-ratio-v2`의 `main` 브랜치 루트에 다음 파일이 있어야 합니다.

```
index.html
data/
  sheet1.json
```

1. 저장소의 **Settings → Pages**를 엽니다.
2. **Build and deployment → Source**를 **Deploy from a branch**로 선택합니다.
3. 브랜치를 **main**, 폴더를 **/ (root)**로 선택하고 **Save**를 누릅니다.
4. GitHub Pages 배포가 완료되면 Pages 설정에 표시되는 웹주소를 휴대폰에서 엽니다.

기본 프로젝트 주소는 `https://hktower226-lang.github.io/hiring-ratio-v2/`입니다. **저장소 업로드와 Pages 배포가 완료되기 전에는 이 주소로 화면을 볼 수 없습니다.** GitHub 계정·저장소의 플랜과 공개 여부에 따라 Pages 사용 가능 여부가 달라질 수 있습니다.

이후 GitHub에서 `data/sheet1.json`만 수정해 `main`에 커밋하면 됩니다. Pages 배포 완료 후 화면의 **데이터 새로고침**을 누르거나 최대 1분 기다리면 반영됩니다. 화면에 표시된 집계와 업체 분류는 JSON으로부터 자동 계산됩니다.

## 컴퓨터에서 개발 중 확인

저장소 폴더에서 다음 명령을 실행합니다.

```sh
python -m http.server 8000 --bind 0.0.0.0
```

휴대폰과 컴퓨터를 같은 Wi-Fi에 연결한 뒤, 휴대폰 브라우저 주소창에 `http://컴퓨터의내부IP:8000/`을 입력합니다. 컴퓨터의 방화벽이 해당 포트 연결을 허용해야 합니다. 이 방법은 사용자의 컴퓨터에서 서버를 실행할 때에만 해당하며, 클라우드 작업 환경의 주소를 휴대폰에서 열 수 있다는 의미는 아닙니다.
