#Код #Except #ZeroDivisionError #raw #try #Python 
```python
is_running = True

while is_running:
    raw = input("Программа для проверки DivisionByZero. Выход или Продолжить?\n")

    if raw == "Выход":
        is_running = False
        print("Программа завершена.")

    elif raw == "Продолжить":
        x = int(input("Делимое: "))
        y = int(input("Делитель: "))
        z = 0

        try:
            z = x/y
            print("Программа работает штатно", z)
            
        except ZeroDivisionError:
            z = 0
            print("Исключение: деление на 0!", z)

    else:
        print("Неккоректная команда.")
```

