#NET_USER #NET_LOCALGROUP
## NET - это набор консольных утилит Windows для управления:

1. Просмотр Списка пользователей - net user
2. Изменение пароля пользователя - net user name
3. Просмотр данных пользователя - net user <имя_пользователя>
4. Просмотр локальных групп - net localgroup
5. Назначение пользователя в локальную группу - net localgroup group user /add
6. Запрет на изменение пароля / «бесконечность» - net user «Пользователь» /passwordchg:no
7. Удаление пользователя - net user username /delete