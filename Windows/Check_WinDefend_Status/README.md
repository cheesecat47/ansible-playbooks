# Get_WinDefend_Status

## Description
Windows Server에 Windows Defender 서비스가 설치되어 있는지 / 실행 중인지 확인합니다.

## Usage

```shell
> ansible-playbook -i inventory.yml check_windefend_status.playbook.yml -k

# ...

TASK [Show Defender service status] **********************************************************************************************************
ok: [winsvr2012_vm01] => {
    "msg": "winsvr2012_vm01:  Not Installed "
}
ok: [winsvr2016_vm02] => {
    "msg": "winsvr2016_vm02:  Stopped "
}
ok: [winsvr2019_vm03] => {
    "msg": "winsvr2019_vm03:  Running "
}
```

### Output Details
- `Not Installed`
  - Defender 서비스는 Windows Server 2012 R2 버전 이상을 지원.
  - Windows Server 2012 R2는 Defender 지원은 되나 기본으로 설치되지 않음.
- `Running` / `Stopped`
  - Windows Server 2016 버전 이상에서는 Defender가 기본 설치되므로 이 둘 중 한 개 출력.

## Tested on
- Windows Server 2012 R2 Standard
- Windows Server 2016 Standard
- Windows Server 2019 Standard

## References
- <https://learn.microsoft.com/en-us/defender-endpoint/microsoft-defender-antivirus-on-windows-server#verify-microsoft-defender-antivirus-is-running>
