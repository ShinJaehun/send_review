
# send_review — Git Commit Diff 패키징 & 전송 도구

X201(코딩 머신)에서 작업한 Git 커밋의 STAT/DIFF를 텍스트 파일로 생성한 뒤,
X230(리뷰/설계 머신)으로 scp를 통해 전송하는 간단한 자동화 도구입니다.

---

## 개요

- X201: 코딩 + 커밋 + push (브라우저 없이)
- X230: diff 파일을 받아 설계/리뷰 대화 진행
- GitHub 웹 UI 복붙 대신 정확한 raw diff 사용

---

## 요구 사항

### 보내는 쪽 (X201)
- git
- ssh / scp (OpenSSH)
- bash

### 받는 쪽 (X230)
- SSH 서버 실행 중
- ~/.ssh/authorized_keys 에 X201 공개키 등록

---

## SSH 서버 설치 (X230)

### Ubuntu / Debian
    sudo apt update
    sudo apt install -y openssh-server
    sudo systemctl enable --now ssh

### Fedora
    sudo dnf install -y openssh-server
    sudo systemctl enable --now sshd

상태 확인:
    systemctl status ssh
    ss -lntp | grep :22

---

## SSH 키 생성 (X201)

키 확인:
    ls -l ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub

없다면 생성:
    ssh-keygen -t ed25519 -C "x201->x230" -f ~/.ssh/id_ed25519

---

## 공개키 등록

### 방법 A (권장)
    ssh-copy-id -i ~/.ssh/id_ed25519.pub USER@X230_HOST

### 방법 B (수동)
X201에서:
    cat ~/.ssh/id_ed25519.pub

출력된 한 줄을 X230의 ~/.ssh/authorized_keys에 추가:

    mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    nano ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/authorized_keys

---

## SSH 테스트

X201에서:
    ssh USER@X230_HOST

비밀번호 없이 접속되면 성공.

---

## SSH config 설정 (권장)

X201의 ~/.ssh/config:

Host x230
    HostName 192.168.0.23
    User shin
    IdentityFile ~/.ssh/id_ed25519

권한:
    chmod 600 ~/.ssh/config

이제:
    ssh x230

---

## send_review 스크립트 설정

스크립트 상단:

TARGET_USER="shin"
TARGET_HOST="x230"
TARGET_DIR="~/review_diffs"

---

## 사용법

최근 커밋:
    send_review HEAD

특정 SHA:
    send_review 9b59ee1

전송 결과:
- X201: /tmp/review_<SHA>.txt 생성
- X230: ~/review_diffs/ 로 업로드

---

## 문제 해결

Permission denied (publickey)
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    ssh -v x230

Connection refused
    systemctl status ssh

방화벽 (Ubuntu)
    sudo ufw allow ssh

---

## 보안 참고

패스프레이즈 사용 권장.
ssh-agent 사용:

    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519"

