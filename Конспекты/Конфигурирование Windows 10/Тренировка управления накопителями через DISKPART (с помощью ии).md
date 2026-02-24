
### Сценарий тренировки DISKPART

 Диск 0 (MBR, 120 ГБ):
- C: (NTFS, 50 ГБ, системный, активный)
- F: (FAT32, 30 ГБ, Log1, логический)
- G: (NTFS, 40 ГБ, Log2, логический)
Свободно: 0 Б

Диск 1 (GPT, 500 ГБ):
- D: (NTFS, 500 ГБ, Storage)
Свободно: 0 Б

Диск 2 (GPT, 250 ГБ):
- H: (NTFS, 150 ГБ, Data)
- I: (NTFS, 80 ГБ, Extra, скрыт атрибутом GPT)
Свободно: 20 ГБ

Задание:

1. На диске 0 удалите логический диск F: (Log1), чтобы освободить 30 ГБ. 

2. В освободившемся пространстве создайте новый основной раздел (нелогический!). Для этого сначала преобразуйте диск 0 из MBR в GPT. 

3. На диске 2 сожмите том H: (Data) на 10 ГБ (освободите место в конце тома). 

4. В освободившемся месте на диске 2 создайте новый простой том размером 15 ГБ, отформатируйте в NTFS, назначьте букву Y, метка "NewData". 

5. На диске 1 сожмите том D: (Storage) на 50 ГБ.

6. В освободившемся пространстве диска 1 создайте раздел размером 30 ГБ, отформатируйте в FAT32, назначьте букву Z, метка "Share". Остаток 20 ГБ оставьте свободным.

7. На диске 1 сделайте раздел Z активным (для GPT это не имеет смысла, но команда active для GPT выдаст ошибку - проверьте реакцию). 

8. Для раздела Y на диске 2 установите атрибут GPT "только для чтения" (атрибут 0x1000000000000000). 

9. Переведите диск 1 в автономное состояние (offline), а затем снова в онлайн (online).

10. Очистите диск 2 полностью (команда clean all), чтобы обнулить все сектора.

11. Инициализируйте диск 2 как GPT заново, создайте на нём один раздел на всё пространство, отформатируйте в NTFS с меткой "Fresh" и назначьте букву W.

### Выполнение
1. select disk 0, select partition 3, delete partition
2. convert gpt, create partition primary size=30000
3. select volume H, shrink desired=10000
4. select disk 2, create partition primary size=15000, format fs=ntfs quick label="NewData", assign letter=Y
5. select volume D, shrink desired=50000
6. select disk 1, create partition primary size=30000, format fs=fat32 quick label="Share", assign letter=Z
7. active
8. select disk 2, select partition 2, gpt attributes=0x1000000000000000
9. select disk 1, offline disk, online disk
10. select disk 2, clean all
11. convert gpt, create partition primary, format fs=ntfs quick label="Fresh", assign letter=W