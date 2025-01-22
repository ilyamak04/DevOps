- `firewall-cmd --zone=public --add-masquerade --permanent`
- `firewall-cmd --reload`

Базовые права устанавливаются на уровне файловой системы Linux.

Пользователь Linux добавляется в базу Samba с помощью smbpasswd.

В конфигурации Samba (smb.conf) задаются дополнительные ограничения и правила.

Все уровни (Linux, база Samba, smb.conf) должны соответствовать, чтобы пользователь получил доступ.