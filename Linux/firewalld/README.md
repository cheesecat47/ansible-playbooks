# firewalld

## Description
리눅스 Firewalld 활용 플레이북입니다.

## Usage

### 01.check_firewalld.playbook.yml

- 대상 호스트에 `firewalld`가 설치되어 있는지, 현재 실행 중인 상태인지 등을 조회합니다.

```shell
> ansible-playbook -i inventory.yml 01.check_firewalld.playbook.yml -k

# ...

TASK [Display firewalld version] *************************************************************************************************************
ok: [centos7.0] => {
    "msg": "firewalld-0.3.9-7.el7.noarch"
}
ok: [centos7.5] => {
    "msg": "firewalld-0.4.4.4-15.el7_5.noarch"
}
ok: [rocky8.10] => {
    "msg": "firewalld-0.9.11-4.el8.noarch"
}
ok: [centos7.9] => {
    "msg": "firewalld-0.6.3-13.el7_9.noarch"
}
ok: [rocky9.6] => {
    "msg": "firewalld-1.3.4-9.el9_5.noarch"
}

# ...

TASK [Display current firewall rules] ******************************************
ok: [centos7.0] => {
    "msg": "block\n  interfaces: \n  sources: \n  services: \n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\ndmz\n  interfaces: \n  sources: \n  services: ssh\n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\ndrop\n  interfaces: \n  sources: \n  services: \n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\nexternal\n  interfaces: \n  sources: \n  services: ssh\n  ports: \n  masquerade: yes\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\nhome\n  interfaces: \n  sources: \n  services: dhcpv6-client ipp-client mdns samba-client ssh\n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\ninternal\n  interfaces: \n  sources: \n  services: dhcpv6-client ipp-client mdns samba-client ssh\n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\npublic (default, active)\n  interfaces: eth0\n  sources: \n  services: dhcpv6-client ssh\n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\ntrusted\n  interfaces: \n  sources: \n  services: \n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t\nwork\n  interfaces: \n  sources: \n  services: dhcpv6-client ipp-client ssh\n  ports: \n  masquerade: no\n  forward-ports: \n  icmp-blocks: \n  rich rules: \n\t"
}
# ...
```

### 02.backup_xml.playbook.yml

- `/etc/firewalld/zones` 경로 아래에는 firewalld zone 별로 방화벽 규칙이 `public.xml`과 같이 저장됩니다.
- 이 파일을 백업(다른 이름으로 복사)해두는 플레이북입니다.

```shell
# zone, today 변수는 직접 입력할 수 있습니다. (주석 해제)
> ansible-playbook -i inventory.yml 02.backup_xml.playbook.yml -k # -e "zone=public today=YYYYmmdd"

# ...

TASK [Backup the original XML file] **********************************************************************************************************
fatal: [centos7.9]: FAILED! => {"changed": false, "msg": "Source /etc/firewalld/zones/private.xml not found"}
changed: [rocky9.6]
# ...
```

#### Output Details
- 위 예시에서 centos7.9 호스트는 `private`존에 규칙을 추가한 적이 없기 때문에 `private.xml` 파일도 생성되지 않았기 때문에 `failed`가 되었습니다.
- xml 파일이 존재하는 경우 `changed`로 표시됩니다.

## Tested on
- CentOS 7.0
- CentOS 7.5
- CentOS 7.9
- Rocky 8.10
- Rocky 9.6 

## References
