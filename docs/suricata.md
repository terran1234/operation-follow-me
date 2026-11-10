# Suricata (IDS/IPS) 커스텀 룰

## 왜 필요했나

기본 오픈소스 룰셋(ET Open 등)은 일반적인 공격 패턴 위주라, FM 8+ 서비스에 실제로 존재하는 취약점(SQLi, XSS, IDOR, 파일 업로드)에 맞춘 탐지는 따로 만들어야 했습니다.

## 적용 과정

1. 로컬 VM(Ubuntu + DVWA + ModSecurity + Suricata)에서 룰 문법과 동작을 먼저 검증
2. pfSense WAN 인터페이스의 `custom.rules`에 최종 룰 15개 적용 (WAN 10 + LAN 5)
3. 처음에는 **전부 alert(탐지만)** 로 시작
4. 정상 동작을 확인한 뒤, **오탐 위험이 없는 8개만 drop(차단)** 으로 전환
5. 관리자 페이지 접근, `member_id` 파라미터 2개는 정상 사용과 공격을 구분할 수 없어 **alert로만 유지**
   (차단하면 정상 기능이 같이 막히기 때문)

## WAN 룰 10개

| # | SID | 탐지 대상 | 동작 |
|---|---|---|---|
| 1 | 1000001 | SQL Injection (UNION SELECT) | drop |
| 2 | 1000002 | Stored XSS (`<script`) | drop |
| 3 | 1000003 | 로그인 반복 시도 (Brute Force) | drop |
| 4 | 1000004 | PHP 웹쉘 업로드 | drop |
| 5 | 1000005 | 관리자 문의 페이지 접근 | alert |
| 6 | 1000006 | IDOR (`member_id` 파라미터) | alert |
| 7 | 1000007 | Directory Traversal | drop |
| 8 | 1000008 | sqlmap 스캐너 | drop |
| 9 | 1000009 | 민감 파일 접근 (`.env`) | drop |
| 10 | 1000010 | Command Injection | drop |

## 배운 점

탐지 룰은 "얼마나 많이 잡느냐"보다 **정상 트래픽을 막지 않는 것**이 중요합니다.
alert로 먼저 검증하고 확신이 있는 룰만 drop으로 올리는 단계적 적용을 택했습니다.
