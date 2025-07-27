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

### 03.open_port.playbook.yml

- `host_vars` 디렉토리에 host 이름과 동일한 yml 파일을 생성하고, 방화벽에서 허용할 port, protocol, zone을 지정합니다.
- `host_vars`에 파일이 없는 host에 대해서는 방화벽 규칙 추가를 실행하지 않습니다.
- zone은 미지정 시 기본값으로 `public`으로 입력되나, port, protocol 정보는 필수적으로 입력해야 합니다.
  - 만약 이 둘 중 하나 이상의 값이 지정되지 않으면 해당 host에 대해서는 firewalld 서비스 재시작을 skip합니다.

```shell
> ansible-playbook -i inventory.yml 03.open_port.playbook.yml -k 
# ...
TASK [Open firewalld ports] ******************************************************************************************************************
skipping: [centos7.5]
skipping: [centos7.9]
changed: [centos7.0] => (item={'port': 22, 'protocol': 'tcp'})
ok: [rocky9.6] => (item={'port': 3306, 'protocol': 'tcp', 'zone': 'private'})
skipping: [rocky9.6] => (item={'port': 5432}) 
changed: [rocky8.10] => (item={'port': 20021, 'protocol': 'tcp'})
changed: [rocky8.10] => (item={'port': '20022-20029', 'protocol': 'tcp'})

RUNNING HANDLER [Restart firewalld] **********************************************************************************************************
changed: [centos7.0]
changed: [rocky8.10]
# ...
```

#### Output Details
- `host_vars`에 파일이 없는 centos7.5, centos7.9 호스트에 대해서는 task를 실행하지 않습니다.
- rocky9.6 호스트는 private 존에 3306/tcp 포트에 대한 규칙은 추가가 되었으나, 5432번 포트에 대한 룰을 잘못 정의하여 재시작하지 않았습니다.
  - private.xml에는 3306/tcp 규칙이 추가된 상태이므로, 이 상태에서 서버에 접속해 `systemctl restart firewalld` 명령을 실행하면 private 존에서 3306/tcp 규칙은 활성화될 것입니다.

### 04.restore_firewalld.playbook.yml

- `/etc/firewalld/zones` 경로에 백업 파일이 있는 경우에 한해 해당 상태로 원복합니다.

```shell
> ansible-playbook -i inventory.yml 04.restore_firewalld.playbook.yml -k # -e "today=20250723 zone=private"
# ...
TASK [Restore from backup file] ***************************************************************************************************
fatal: [centos7.5]: FAILED! => {"changed": false, "msg": "Source /etc/firewalld/zones/private.xml.bak.20250723 not found"}
changed: [rocky9.6]
# ...
```

## Tested on
- CentOS 7.0
- CentOS 7.5
- CentOS 7.9
- Rocky 8.10
- Rocky 9.6 

## References
