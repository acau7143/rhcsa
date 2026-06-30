## Day 1 Study Note
### 2026-07-01 | 네트워크 · YUM repo · NTP

---

## 왜 이 명령어/개념을 배우는가

| 항목 | 선택한 것 | 대안 | 선택 이유 |
|------|-----------|------|-----------|
| 네트워크 설정 | `nmcli` | `nmtui`, `ifcfg` 파일 직접 수정 | RHEL 10 표준, ifcfg deprecated |
| 패키지 저장소 | `.repo` 파일 생성 | `dnf config-manager` | 시험에서 직접 파일 작성이 기본 |
| 시간 동기화 | `chronyd` | `ntpd` | RHEL 기본 데몬, 가상화 환경에 강함 |

---

## 오늘 공부한 핵심 명령어

### nmcli

```bash
# 연결 상태 확인
nmcli connection show
nmcli device status

# 고정 IP 설정
sudo nmcli connection modify enp0s3 \
  ipv4.method manual \
  ipv4.addresses 10.0.2.15/24 \
  ipv4.gateway 10.0.2.2 \
  ipv4.dns 8.8.8.8

# 설정 적용
sudo nmcli connection up enp0s3

# 검증
ip a show enp0s3
nmcli connection show enp0s3 | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway|ipv4.dns"
```

### YUM repo

```bash
# repo 파일 생성
sudo vim /etc/yum.repos.d/myrepo.repo

# repo 목록 확인
dnf repolist
```

```ini
# .repo 파일 형식
[repo-id]
name=설명
baseurl=http://주소
enabled=1
gpgcheck=0
```

### chronyd

```bash
# 상태 확인
systemctl status chronyd
timedatectl

# 설정 파일
sudo vim /etc/chrony.conf

# 재시작
sudo systemctl restart chronyd

# 동기화 확인
chronyc sources -v
```

---

## 오늘 처음 안 사실

| 몰랐던 것 | 알게 된 것 | 왜 그렇게 되는지 |
|-----------|-----------|----------------|
| modify 후 왜 up을 따로 해야 하나 | modify는 저장, up은 적용 | 저장과 적용이 분리돼 있음 |
| repo 파일 등록 명령이 따로 있는 줄 알았음 | 파일만 만들면 DNF가 자동으로 읽음 | `/etc/yum.repos.d/*.repo` 를 DNF가 자동 스캔 |
| gpgcheck=0 이 그냥 옵션인 줄 알았음 | GPG 키 없으면 설치 실패함 | 키 없이 gpgcheck=1 하면 검증 실패 |
| chronyc sources `^*` 의미 몰랐음 | 현재 동기화 중인 서버 표시 | `*` = best, `?` = 연결 안 됨 |

---

## Incident

### INC-001 — nmcli 권한 오류

| 항목 | 내용 |
|------|------|
| **증상** | `GDBus.Error: The name is not activatable` |
| **원인** | 일반 유저(uid=1000)로 nmcli 실행 |
| **확인** | `systemctl status NetworkManager` 로그에서 `result="fail" uid=1000` 확인 |
| **해결** | `sudo nmcli connection modify ...` |
| **예방** | nmcli 설정 변경은 항상 sudo 필요 |

---

## 검증 명령어 정리

| 검증 대상 | 명령어 |
|-----------|--------|
| 네트워크 IP | `ip a show enp0s3` |
| 고정 IP 설정값 | `nmcli connection show enp0s3 \| grep ipv4` |
| repo 등록 확인 | `dnf repolist` |
| NTP 동기화 상태 | `chronyc sources -v` |
| 시간/NTP 전체 상태 | `timedatectl` |

---

## man 페이지 치트시트

| 명령어 | man 페이지 |
|--------|-----------|
| nmcli 예시 | `man nmcli-examples` |
| repo 파일 형식 | `man 5 dnf.conf` |
| chrony 설정 | `man chrony.conf` |
| chronyc 명령 | `man chronyc` |

---

## 혼자 다시 할 수 있나? (내일 자문)

- [ ] nmcli로 고정 IP 설정하고 검증까지 혼자 할 수 있다
- [ ] .repo 파일을 형식에 맞게 처음부터 작성할 수 있다
- [ ] chrony.conf에서 NTP 서버 변경하고 동기화 확인할 수 있다