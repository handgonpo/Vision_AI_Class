## 1. 최초 1회 — GitHub 업로드 전용 폴더 생성

PowerShell에서 실행합니다.

```powershell
cd "C:\Users\user1\Desktop\이유정강사"

git clone https://github.com/handgonpo/Vision_AI_Class.git Vision_AI_Class_upload
```

## 2. 앞으로 매번 실행할 명령어

`backEnd_2` 폴더를 업로드하는 예시

```PowerShell
# 1. 경로와 업로드 폴더 설정
$base = "C:\Users\user1\Desktop\이유정강사"
$folder = "backEnd_2"

# 2. GitHub 업로드 전용 폴더 이동
cd "$base\Vision_AI_Class_upload"

# 3. 최신 main 가져오기
git status --short
git switch main
git pull --ff-only origin main

# 4. 원하는 폴더만 복사
$source = Join-Path $base "vision_AI_7_to_15\$folder"
$dest = Join-Path $base "Vision_AI_Class_upload\$folder"

if (!(Test-Path -LiteralPath $source -PathType Container)) {
    throw "원본 폴더가 없습니다: $source"
}

robocopy $source $dest /E /XD .git .venv venv node_modules __pycache__ /XF .env .env.* *.pem *.pfx *.key /R:2 /W:2

if ($LASTEXITCODE -ge 8) {
    throw "파일 복사 오류: $LASTEXITCODE"
}

# 5. 해당 폴더만 Add
git add -- "$folder/"

# 6. 추가된 파일 확인
git -c core.quotepath=false diff --cached --name-only

# 7. Commit
git commit -m "Update $folder"

# 8. Push
git push origin main
```

## 3. 다른 폴더를 업로드하려면?

앞으로는 이 부분만 변경하면 됩니다.

```PowerShell
$folder = "교과 9 openCV 규칙기반 머신비전"
```

또는

```PowerShell
$folder = "교과 8 프로젝트"
```
