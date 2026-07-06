# INC-001: tar bzip2 압축 실패 — bzip2 패키지 미설치

## Summary
`/usr/local` 디렉토리를 bzip2로 압축하여 `/root/backup.tar.bz2`로 저장하는 작업(RHCSA 11번 문항) 중, `tar -cvjf` 명령이 `bzip2: command not found` 에러와 함께 실패함. 시스템에 `bzip2` 패키지가 설치되어 있지 않아 발생한 문제로, `dnf install bzip2` 후 재실행하여 정상 복구됨.

## Severity
🟡 Medium — 서비스 장애는 아니지만, 실습/시험 진행이 중단되는 수준의 이슈. 시험 환경에서도 동일하게 발생 가능성 있어 사전 점검 필요.

## Symptoms
```bash
[root@localhost reallinux]# tar -cvjf /root/backup.tar.bz2 /usr/local
tar: Removing leading `/' from member names
/usr/local/
/usr/local/bin/
...
/bin/sh: line 1: bzip2: command not found
...
tar: /root/backup.tar.bz2: Cannot write: Broken pipe
tar: Child returned status 127
tar: Error is not recoverable: exiting now
```
- `tar` 명령 실행 중간에 `bzip2: command not found` 출력
- 이어서 `Cannot write: Broken pipe`, `Child returned status 127` 발생
- 최종적으로 `/root/backup.tar.bz2` 파일은 생성되지만 불완전한 상태

## Root Cause
`tar -j` 옵션은 tar 자체가 압축을 수행하는 게 아니라, 시스템에 설치된 외부 `bzip2` 프로그램을 파이프로 호출하는 방식으로 동작함. Rocky Linux 9.8 최소 설치 환경에는 `bzip2` 패키지가 기본으로 포함되어 있지 않아, tar가 호출하려는 프로그램 자체가 존재하지 않아 실패함.

**핵심:** 명령어 문법 자체는 문제가 없었음 (`tar -cvjf` 옵션 사용 정확). 원인은 순수하게 **환경(패키지 미설치)** 문제였음.

## Recovery
```bash
# 1. bzip2 설치 여부 확인
which bzip2
# (출력 없음 → 미설치 확인)

# 2. 패키지 설치
dnf install -y bzip2

# 3. 설치 확인
which bzip2
bzip2 --version

# 4. 이전 실패로 생성된 불완전한 아카이브 삭제
rm -f /root/backup.tar.bz2

# 5. tar 재실행
tar -cvjf /root/backup.tar.bz2 /usr/local

# 6. 결과 검증
ls -lh /root/backup.tar.bz2
file /root/backup.tar.bz2
```

### Evidence (복구 확인)
```bash
[root@localhost reallinux]# file /root/backup.tar.bz2
/root/backup.tar.bz2: bzip2 compressed data, block size = 900k
```
매직 넘버 기준으로 실제 bzip2 압축 데이터임을 확인 → 정상 복구 완료.

## Prevention
- **작업 전 사전 점검 습관화:** 압축/네트워크 등 외부 프로그램에 의존하는 명령을 쓰기 전, `which {프로그램명}`으로 존재 여부를 먼저 확인
- **RHCSA 시험 환경 대비:** 시험 환경이 최소 설치 기반일 가능성이 있으므로, `-z`(gzip)는 되는데 `-j`(bzip2)만 안 될 수 있다는 점을 인지하고 있어야 함. 시험 중 이런 에러를 만나면 당황하지 말고 `dnf install bzip2`로 즉시 해결 가능
- **실무 확장:** 배포/자동화 스크립트 작성 시, 스크립트 시작 부분에 필요한 패키지 존재 여부를 `command -v` 등으로 사전 체크하는 로직을 넣는 습관 (Ansible/Terraform provisioning에서도 동일한 패턴 적용됨)

## 관련 문항
- RHCSA 덤프 11번 (tar bzip2 압축) — `notes/2026-07-06-day4.md` 참고

