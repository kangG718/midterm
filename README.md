# 모바일/웹서비스 프로젝트 — 공통평가 01

Django 이미지 블로그 서버와 Java Android 클라이언트입니다.

## 제출 구조

```text
ROOT/
├── PhotoBlogServer/     # Django Server
├── PhotoViewer/         # Android Client
└── 모바일웹서비스 프로젝트_공통평가 01_수행 결과 보고서.docx
```

보고서는 현재 진행 상황을 포함하며, 클라우드 동작 검증 후 최종 캡처를 반영합니다.
가상환경, 빌드 산출물, 개인 계정이 들어 있는 DB, 업로드 사진, HTTP 요청 기록,
로컬 비밀키와 Word 임시 잠금 파일은 Git에 포함하지 않습니다.
이미지 필드의 기본 파일 `media/images/default_error.png`는 실행에 필요한 자산으로 포함합니다.

## 서버 실행

Python 3.13 환경에서 확인했습니다. 저장소를 받은 뒤 `ROOT/PhotoBlogServer`에서 실행합니다.

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

처음 실행하는 환경에서만 다음 명령으로 비밀키를 생성합니다. 기존 키 파일은 덮어쓰지 않습니다.

```bash
python -c 'from pathlib import Path; from django.core.management.utils import get_random_secret_key; p = Path(".django-secret-key"); f = p.open("x"); f.write(get_random_secret_key()); f.close(); p.chmod(0o600)'
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

환경변수 `DJANGO_SECRET_KEY`가 있으면 해당 값을 우선 사용합니다.
로컬에서는 `http://127.0.0.1:8000/`에 접속합니다.
새 DB의 관리자 계정과 게시글은 해당 환경에서 직접 생성합니다.

## PythonAnywhere 배포

이번 공통평가 계정은 `garamkang`입니다. 기존 실습 계정과 별도로 운영합니다.
실습 3과 같이 GitHub 코드를 Bash 콘솔에서 가져와 배포합니다.

```bash
git clone https://github.com/kangG718/photoblog-assessment.git
cd photoblog-assessment/ROOT/PhotoBlogServer
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

위 서버 실행 절차의 비밀키 생성, `migrate`, `createsuperuser`를 수행한 뒤 정적 파일을 모읍니다.

```bash
python manage.py collectstatic --noinput
```

Web 탭에서 **Manual configuration → Python 3.13**으로 웹앱을 생성합니다.

| 설정 | 값 |
|---|---|
| Source code / Working directory | `/home/garamkang/photoblog-assessment/ROOT/PhotoBlogServer` |
| Virtualenv | `/home/garamkang/photoblog-assessment/ROOT/PhotoBlogServer/.venv` |
| `/static/` | `/home/garamkang/photoblog-assessment/ROOT/PhotoBlogServer/staticfiles` |
| `/media/` | `/home/garamkang/photoblog-assessment/ROOT/PhotoBlogServer/media` |

Web 탭에 연결된 WSGI 설정 파일에 `PhotoBlogServer/deploy/pythonanywhere_wsgi.py`의 내용을 적용하고 Reload합니다.
이 WSGI 설정은 공개 서버에서 `DEBUG=False`를 적용합니다.
배포 목표 주소는 `https://garamkang.pythonanywhere.com/`이며, 실제 배포 완료 여부는 별도로 확인해야 합니다.

## Android 클라이언트

Android Studio에서 `ROOT/PhotoViewer`를 엽니다. Java와 XML Views를 사용합니다.
현재 `MainActivity.java`의 `site_url`은 로컬 서버 주소입니다.
실제 휴대폰에서 USB로 로컬 서버에 연결할 때는 `adb reverse tcp:8000 tcp:8000`을 사용합니다.
클라우드 검증 시 `site_url`을 `https://garamkang.pythonanywhere.com`으로 변경하고 다시 빌드합니다.

- 동기화: `GET /api_root/Post/`로 게시물 목록과 이미지를 받아 표시합니다.
- 새로운 이미지 게시: 제목, 내용, 이미지와 서버 계정을 입력하는 UI를 제공합니다.
- 인증: `/api-token-auth/`에서 발급받은 토큰으로 `POST /api_root/Post/` 요청을 보냅니다.
- 업로드: `multipart/form-data`로 제목, 내용, 선택한 이미지를 전송합니다.

## 확인 방법

```bash
cd ROOT/PhotoBlogServer
source .venv/bin/activate
python manage.py check
python manage.py test blog
```

- 12번: 클라우드 주소의 브라우저에서 게시글과 이미지가 표시되는지 확인합니다.
- 13번: Android 앱이 같은 클라우드 서버에서 이미지를 받아 동기화하는지 확인합니다.
- 화면 캡처는 `ROOT`의 수행 결과 보고서에 반영합니다.
