# 서버 취약점 점검 Playbook

- KISA ["주요정보통신기반시설 기술적 취약점 분석ㆍ평가 방법 상세가이드"](https://krcert.or.kr/kr/bbs/view.do?searchCnd=1&bbsId=B0000127&searchWrd=취약점&menuNo=205021&pageIndex=1&categoryCode=&nttId=35988)를 다중 서버 대상으로 동시 실행 가능하도록 Ansible Playbook으로 구현

|  OS   | Code  |                            Name                            |
| :---: | :---: | :--------------------------------------------------------: |
| Linux | U-01  | [root 계정 원격접속 제한](./Linux/U-01.check.playbook.yml) |
