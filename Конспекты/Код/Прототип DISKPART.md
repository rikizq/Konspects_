```python
import re

class DiskPartSimulator:
    def __init__(self):
        # Состояние системы после предыдущего упражнения
        self.disks = {
            0: Disk(0, size_gb=120, style='MBR', partitions=[
                Partition(1, 'Основной', 50, 'NTFS', 'C', active=True, system=True),
                Partition(2, 'Расширенный', 70, '', '', extended=True, children=[
                    Partition(3, 'Логический', 40, 'NTFS', 'G', label='Log2'),
                    Partition(4, 'Логический', 20, 'exFAT', 'X', label='Temp'),
                    # свободно 10 ГБ внутри расширенного
                ])
            ]),
            1: Disk(1, size_gb=500, style='GPT', partitions=[
                Partition(1, 'Основной', 448, 'NTFS', 'D', label='Storage'),
                Partition(2, 'Основной', 30, 'FAT32', 'Z', label='Share'),
                # свободно 20 ГБ
            ]),
            2: Disk(2, size_gb=250, style='GPT', partitions=[
                Partition(1, 'Основной', 250, 'NTFS', 'W', label='Fresh'),
            ]),
        }
        self.current_disk = None
        self.current_partition = None
        self.current_volume = None
        self.shadows = []  # список теневых копий (как тома)
        self.next_volume_num = 5  # для томов (том 0-4 были в предыдущих упражнениях)

    def run(self):
        print("DiskPart симулятор запущен. Введите 'exit' для выхода.")
        while True:
            cmd = input("DISKPART> ").strip()
            if not cmd:
                continue
            if cmd.lower() == 'exit':
                print("Выход из DiskPart...")
                break
            self.process_command(cmd)

    def process_command(self, cmd):
        parts = cmd.lower().split()
        if not parts:
            return

        # list disk
        if parts[0] == 'list' and len(parts) >= 2 and parts[1] == 'disk':
            self.list_disk()
        # list partition
        elif parts[0] == 'list' and len(parts) >= 2 and parts[1] == 'partition':
            self.list_partition()
        # list volume
        elif parts[0] == 'list' and len(parts) >= 2 and parts[1] == 'volume':
            self.list_volume()
        # select disk N
        elif parts[0] == 'select' and parts[1] == 'disk' and len(parts) == 3:
            try:
                n = int(parts[2])
                self.select_disk(n)
            except:
                print("Неверный номер диска.")
        # select partition N
        elif parts[0] == 'select' and parts[1] == 'partition' and len(parts) == 3:
            try:
                n = int(parts[2])
                self.select_partition(n)
            except:
                print("Неверный номер раздела.")
        # select volume X
        elif parts[0] == 'select' and parts[1] == 'volume' and len(parts) == 3:
            vol = parts[2].upper()
            self.select_volume(vol)
        # create partition primary [size=N] [offset=N]
        elif parts[0] == 'create' and parts[1] == 'partition' and parts[2] == 'primary':
            self.create_primary(cmd)
        # create partition extended [size=N]
        elif parts[0] == 'create' and parts[1] == 'partition' and parts[2] == 'extended':
            self.create_extended(cmd)
        # create partition logical [size=N]
        elif parts[0] == 'create' and parts[1] == 'partition' and parts[2] == 'logical':
            self.create_logical(cmd)
        # delete partition [override]
        elif parts[0] == 'delete' and parts[1] == 'partition':
            self.delete_partition()
        # format fs=... label="..." quick
        elif parts[0] == 'format':
            self.format_volume(cmd)
        # assign letter=X
        elif parts[0] == 'assign' and len(parts) >= 2 and parts[1].startswith('letter='):
            letter = parts[1].split('=')[1].upper()
            self.assign_letter(letter)
        # shrink desired=N
        elif parts[0] == 'shrink':
            self.shrink_volume(cmd)
        # extend size=N
        elif parts[0] == 'extend':
            self.extend_volume(cmd)
        # gpt attributes=value
        elif parts[0] == 'gpt':
            self.set_gpt_attributes(cmd)
        # offline disk
        elif parts[0] == 'offline' and parts[1] == 'disk':
            self.offline_disk()
        # online disk
        elif parts[0] == 'online' and parts[1] == 'disk':
            self.online_disk()
        # clean [all]
        elif parts[0] == 'clean':
            self.clean_disk(cmd)
        # convert gpt / mbr
        elif parts[0] == 'convert' and len(parts) >= 2:
            self.convert_disk(parts[1])
        # create shadow
        elif parts[0] == 'create' and parts[1] == 'shadow':
            self.create_shadow()
        # delete shadow
        elif parts[0] == 'delete' and parts[1] == 'shadow':
            self.delete_shadow(cmd)
        # list shadow
        elif parts[0] == 'list' and parts[1] == 'shadow':
            self.list_shadow()
        else:
            print(f"Нераспознанная команда: {cmd}")

    # ---------- Вспомогательные методы ----------
    def get_volume_by_letter(self, letter):
        for disk in self.disks.values():
            if disk.online:
                for p in disk.partitions:
                    if p.letter == letter:
                        return p
        return None

    def get_partition_by_number(self, disk, num):
        # простая нумерация разделов диска
        count = 0
        for p in disk.partitions:
            if not p.extended:
                count += 1
                if count == num:
                    return p
            else:
                # расширенный раздел сам не считается как отдельный номер в list partition
                # но в DISKPART расширенный раздел имеет номер, а логические - следующие номера
                # Упростим: будем нумеровать все разделы подряд
                pass
        # упрощённо: линейный поиск по атрибуту number (нужно задавать номера)
        for p in disk.partitions:
            if hasattr(p, 'number') and p.number == num:
                return p
        return None

    def find_free_space(self, disk, size_mb, partition_type='primary'):
        # Поиск непрерывного свободного места на диске (упрощённо)
        # Возвращает начало в МБ или None
        # В реальности нужно учитывать границы разделов
        # Для MBR и GPT логика разная
        # Упростим: считаем, что свободное место всегда есть в конце диска,
        # а для логических - внутри расширенного
        pass

    # ---------- Команды ----------
    def list_disk(self):
        print("\n  Диск ###  Состояние      Размер  Свободно  Дин  GPT")
        print("  ---------  -------------  -------  --------  ---  ---")
        for num, d in self.disks.items():
            free = d.free_space_gb()
            gpt = '*' if d.style == 'GPT' else ''
            din = ' '  # динамический не поддерживаем
            print(f"  Диск {num}     {'В сети' if d.online else 'Нет сети'}          {d.size_gb} ГБ     {free} ГБ        {din}   {gpt}")
        print()

    def list_partition(self):
        if not self.current_disk:
            print("Нет выбранного диска. Используйте select disk.")
            return
        d = self.disks[self.current_disk]
        print("\n  Раздел ###  Тип              Размер  Смещение")
        print("  ----------  ----------------  -------  --------")
        # Пронумеруем разделы подряд
        num = 1
        for p in d.partitions:
            typ = p.type
            if p.extended:
                typ = "Расширенный"
            elif p.type == 'Основной':
                typ = "Основной"
            elif p.type == 'Логический':
                typ = "Логический"
            print(f"  Раздел {num}    {typ:<16} {p.size} ГБ    {p.offset or 0} КБ")
            p.number = num
            num += 1
            if p.extended and p.children:
                for child in p.children:
                    print(f"  Раздел {num}    Логический        {child.size} ГБ    {child.offset or 0} КБ")
                    child.number = num
                    num += 1
        print()

    def list_volume(self):
        print("\n  Том ###  Имя         Метка         ФС     Тип        Размер  Состояние  Инфо")
        print("  ---------  ----------  ------------  -----  ----------  -------  ---------  -----------")
        # Соберём все тома
        volumes = []
        for disk in self.disks.values():
            for p in disk.partitions:
                if p.fs and p.letter:
                    volumes.append(p)
                if p.extended and p.children:
                    for c in p.children:
                        if c.fs and c.letter:
                            volumes.append(c)
        # Добавим теневые
        for sh in self.shadows:
            volumes.append(sh)

        for idx, v in enumerate(volumes, start=1):
            info = "Системный" if getattr(v, 'system', False) else ""
            dinfo = "Теневая копия" if hasattr(v, 'shadow') and v.shadow else ""
            print(f"  Том {idx}      {v.letter:<9} {v.label or '':<12} {v.fs:<5}  Основной     {v.size:>4} ГБ  Исправен   {info} {dinfo}")
        print()

    def select_disk(self, n):
        if n in self.disks:
            self.current_disk = n
            self.current_partition = None
            self.current_volume = None
            print(f"Диск {n} выбран.")
        else:
            print(f"Диск {n} не найден.")

    def select_partition(self, n):
        if not self.current_disk:
            print("Сначала выберите диск.")
            return
        # Ищем раздел с номером n на текущем диске (упрощённо)
        d = self.disks[self.current_disk]
        found = None
        for p in d.partitions:
            if hasattr(p, 'number') and p.number == n:
                found = p
                break
        if not found:
            print(f"Раздел {n} не найден на диске {self.current_disk}.")
        else:
            self.current_partition = found
            self.current_volume = found if found.fs else None
            print(f"Раздел {n} выбран.")

    def select_volume(self, letter):
        # поиск по букве
        for disk in self.disks.values():
            for p in disk.partitions:
                if p.letter == letter:
                    self.current_volume = p
                    self.current_disk = disk.num
                    self.current_partition = p
                    print(f"Том {letter} выбран.")
                    return
            if disk.extended and disk.children:
                for c in disk.children:
                    if c.letter == letter:
                        self.current_volume = c
                        self.current_disk = disk.num
                        self.current_partition = c
                        print(f"Том {letter} выбран.")
                        return
        for sh in self.shadows:
            if sh.letter == letter:
                self.current_volume = sh
                print(f"Том {letter} (теневая копия) выбран.")
                return
        print(f"Том {letter} не найден.")

    def create_primary(self, cmd):
        if not self.current_disk:
            print("Сначала выберите диск.")
            return
        d = self.disks[self.current_disk]
        # Парсим размер
        size_mb = None
        m = re.search(r'size=(\d+)', cmd, re.IGNORECASE)
        if m:
            size_mb = int(m.group(1))
        # Если размер не указан, занимаем всё свободное место
        free = d.free_space_mb()
        if size_mb is None:
            size_mb = free
        if size_mb > free:
            print(f"Недостаточно свободного места. Доступно: {free} МБ")
            return
        # Для простоты создаём раздел в конце диска
        new_part = Partition(num=None, type='Основной', size=size_mb/1024, fs='', letter=None, label='')
        d.partitions.append(new_part)
        # Обновим нумерацию позже
        print("DiskPart выполнил создание раздела.")
        self.current_partition = new_part

    def create_extended(self, cmd):
        # Аналогично, но создаём расширенный раздел (только MBR)
        print("Создание расширенного раздела пока не реализовано в симуляторе.")
        pass

    def create_logical(self, cmd):
        # Внутри расширенного
        print("Создание логического раздела пока не реализовано.")
        pass

    def delete_partition(self):
        if not self.current_partition:
            print("Нет выбранного раздела.")
            return
        # Удаляем раздел из диска
        disk = self.disks[self.current_disk]
        if self.current_partition in disk.partitions:
            disk.partitions.remove(self.current_partition)
        elif hasattr(disk, 'children') and self.current_partition in disk.children:
            # для логических
            pass
        self.current_partition = None
        self.current_volume = None
        print("DiskPart выполнил удаление раздела.")

    def format_volume(self, cmd):
        if not self.current_volume:
            print("Нет выбранного тома.")
            return
        fs_match = re.search(r'fs=(\w+)', cmd, re.IGNORECASE)
        label_match = re.search(r'label="([^"]+)"', cmd, re.IGNORECASE)
        quick = 'quick' in cmd.lower()
        if fs_match:
            fs = fs_match.group(1).upper()
            self.current_volume.fs = fs
        if label_match:
            self.current_volume.label = label_match.group(1)
        print("  100 процентов выполнено")
        print("DiskPart успешно отформатировал том.")

    def assign_letter(self, letter):
        if not self.current_volume:
            print("Нет выбранного тома.")
            return
        # Проверим, не занята ли буква
        for disk in self.disks.values():
            for p in disk.partitions:
                if p.letter == letter:
                    print(f"Буква {letter} уже используется.")
                    return
        self.current_volume.letter = letter
        print("DiskPart успешно назначил букву диска или точку подключения.")

    def shrink_volume(self, cmd):
        if not self.current_volume:
            print("Нет выбранного тома.")
            return
        desired_match = re.search(r'desired=(\d+)', cmd, re.IGNORECASE)
        if desired_match:
            desired_mb = int(desired_match.group(1))
            # Уменьшаем размер
            current_mb = self.current_volume.size * 1024
            if desired_mb >= current_mb:
                print("Ошибка: желаемый размер сжатия больше размера тома.")
                return
            new_mb = current_mb - desired_mb
            self.current_volume.size = new_mb / 1024
            print(f"DiskPart успешно уменьшил размер тома:\n    Новый размер тома: {int(new_mb)} МБ")
        else:
            print("Не указан размер сжатия.")

    def extend_volume(self, cmd):
        if not self.current_volume:
            print("Нет выбранного тома.")
            return
        size_match = re.search(r'size=(\d+)', cmd, re.IGNORECASE)
        if size_match:
            size_mb = int(size_match.group(1))
            # Увеличиваем размер (проверка свободного места не делается)
            self.current_volume.size += size_mb / 1024
            print("DiskPart успешно расширил том.")

    def set_gpt_attributes(self, cmd):
        if not self.current_volume:
            print("Нет выбранного тома.")
            return
        attr_match = re.search(r'attributes=([0-9a-fA-Fx]+)', cmd, re.IGNORECASE)
        if attr_match:
            attr = attr_match.group(1)
            self.current_volume.gpt_attrs = attr
            print("DiskPart успешно установил атрибуты GPT.")

    def offline_disk(self):
        if not self.current_disk:
            print("Нет выбранного диска.")
            return
        self.disks[self.current_disk].online = False
        print("DiskPart успешно перевел выбранный диск в автономный режим.")

    def online_disk(self):
        if not self.current_disk:
            print("Нет выбранного диска.")
            return
        self.disks[self.current_disk].online = True
        print("DiskPart успешно перевел выбранный диск в онлайн.")

    def clean_disk(self, cmd):
        if not self.current_disk:
            print("Нет выбранного диска.")
            return
        d = self.disks[self.current_disk]
        d.partitions = []
        if 'all' in cmd.lower():
            print("DiskPart выполнил очистку диска. (Все сектора обнулены.)")
        else:
            print("DiskPart выполнил очистку диска.")

    def convert_disk(self, style):
        if not self.current_disk:
            print("Нет выбранного диска.")
            return
        if style.upper() in ('GPT', 'MBR'):
            self.disks[self.current_disk].style = style.upper()
            print(f"DiskPart успешно преобразовал выбранный диск в формат {style.upper()}.")
        else:
            print("Неверный стиль. Используйте GPT или MBR.")

    def create_shadow(self):
        if not self.current_volume:
            print("Нет выбранного тома.")
            return
        # Создаём теневую копию как новый том
        shadow = Partition(num=None, type='Теневая копия', size=self.current_volume.size,
                           fs=self.current_volume.fs, letter=None, label='')
        shadow.shadow = True
        self.shadows.append(shadow)
        print("Теневая копия создана.")

    def delete_shadow(self, cmd):
        # Удаляем теневую копию
        if not self.shadows:
            print("Нет теневых копий.")
            return
        # Для простоты удалим последнюю
        self.shadows.pop()
        print("Теневая копия удалена.")

    def list_shadow(self):
        if not self.shadows:
            print("На этом компьютере нет теневых копий.")
        else:
            for i, s in enumerate(self.shadows):
                print(f"Теневая копия {i+1}: размер {s.size} ГБ, FS {s.fs}")

class Disk:
    def __init__(self, num, size_gb, style, partitions=None, online=True):
        self.num = num
        self.size_gb = size_gb
        self.style = style  # 'MBR' or 'GPT'
        self.partitions = partitions if partitions else []
        self.online = online

    def free_space_gb(self):
        used = sum(p.size for p in self.partitions if not p.extended)  # расширенный не занимает места отдельно
        return self.size_gb - used

    def free_space_mb(self):
        return self.free_space_gb() * 1024

class Partition:
    def __init__(self, num, type, size, fs, letter, label='', active=False, system=False, extended=False, children=None, offset=None):
        self.number = num
        self.type = type  # 'Основной', 'Расширенный', 'Логический'
        self.size = size  # в ГБ
        self.fs = fs
        self.letter = letter
        self.label = label
        self.active = active
        self.system = system
        self.extended = extended
        self.children = children if children else []
        self.offset = offset
        self.gpt_attrs = None

if __name__ == "__main__":
    sim = DiskPartSimulator()
    sim.run()
```