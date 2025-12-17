#Hangman #Виселица #Txt-reader #Python 
# Hangman.py 

```python
#Импорт библиотек в самом начале кода
#случайные величины, время, система
import random, time, os
#import words
import txt_words_reader as tws

class Hangman:

    #Конструктор класса
    def __init__(self, words_param):
        self.words = words_param
        self.mainloop()

    def _new_game(self):
        self.attempts = 12
        self.selected_word = random.choice(self.words).lower()
        self.unguessed_string = "_" * len(self.selected_word)
        self.guessed_string = self.unguessed_string 

    def mainloop(self):

        os.system("cls")
        is_running = True
        while is_running == True:
            self._new_game()
            print("Добро пожаловать в игру Висельник!")
            print("Выйти из игры можно только после её окончания.")
            user_input = input("Enter для продолжения или \"exit\" для выхода. \n")
            if user_input == "exit":
                is_running = False
                break

            while self.attempts != 0:

                if self.guessed_string.count("_") == 0:
                    print("Победа! Загаданное слово - ", self.selected_word)
                    break

                os.system("cls")
                print("Осталось попыток:", self.attempts)
                print(self.guessed_string)
                letter = input("Введите букву: ")

                if self.selected_word.count(letter) != 0:

                    self.unguessed_string = ""
                    for i in range(len(self.selected_word)):
                        if self.selected_word[i] == letter or self.guessed_string[i] != "_":
                            self.unguessed_string += self.selected_word[i]
                        else:
                            self.unguessed_string += "_"
                    self.guessed_string = self.unguessed_string

                else:
                    self.attempts -= 1

    def test(self):
        os.system("cls")
        print(self.words)
        print(self.attempts)
        print(self.selected_word)
        print(self.guessed_string)

#words_list = ["Яблоко", "Влад", "Никита", "Очки", "Кашель"]
#words_list - words.s1
words_list = tws.read_words("words.txt")
s = Hangman(words_list)
s.test()
s.mainloop()
```

Игра "Виселица" - где пользователь пытается угадать слово по буквам, имея ограниченное число попыток. Игра повторяется, пока пользователь не решит выйти или не выиграет/проиграет в текущей игре.
### Импорт библиотек

random  для случайного выбора слова
time не используется явно в коде, возможно, для задержек или тайминга
os  для управления системой (например, очистки консоли)
txt_words_reader  пользовательский модуль (не приведён в коде), для чтения слов из файла
### Конструктор init
Получает список слов words_param
Сохраняет его в self.words
Запускает главный цикл self.mainloop()
### Метод newgame
Обнуляет попытки (self.attempts = 12)
Выбирает случайное слово из списка и переводит его в нижний регистр
Создаёт строку из _ , которая показывает скрытое слово (self.unguessed_string)
Также копирует эту строку в self.guessed_string — текущий прогресс

### Метод mainloop  основная логика игры
Очищает экран (os.system("cls"))
Запускает цикл, который позволяет играть несколько раундов
Перед началом каждого раунда  создаёт новую игру (new_game()) 
Предлагает игроку начать или выйти
Внутренний цикл  пока есть попытки:
Проверяет победу (не осталось _  )
Если победа  выводит сообщение и прекращает текущий раунд
Иначе показывает количество попыток и текущий прогресс слова
Запрашивает букву для угадывания
Если буква есть в слове  обновляет прогресс (self.guessed_string)
Если нет  уменьшает количество попыток

### Основной скрипт

Загружает список слов из файла words.txt с помощью функции read_words из модуля txt_words_reader.
Создаёт объект Hangman с этим списком.
Вызывает test() для вывода текущих переменных.
Запускает основной цикл игры mainloop().

### def test(self):
os.system("cls") очищает экран команды Windows. Это делается для того, чтобы вывод был чистым и удобно видно текущие переменные.
print(self.words)  выводит список слов, из которых выбирается загаданное слово. Это помогает понять, какие слова доступны для игры.
print(self.attempts) показывает сколько попыток осталось у игрока в текущей игре.
print(self.selected_word) выводит загаданное слово. Это обычно делается для отладки, чтобы убедиться, что программа правильно выбирает слово, или чтобы знать правильный ответ во время тестирования.
print(self.guessed_string) показывает текущий прогресс угадывания: какие буквы уже угаданы и отображаются, а какие еще скрыты (подчеркивания).

# txt-reader.py

```python
def read_words(filename: str):
    words_out = []

    with open(filename, "r", encoding = "utf-8") as myfile:
        words_raw = myfile.readlines()

        for word in words_raw:
            words_out.append(word[:-1])

        myfile.close()
    return words_out
```

def read_words(filename: str):  объявляет функцию, которая принимает один параметр:
    filename  название файла из которого нужно прочитать слова.
Создание пустого списка words_out:
    words_out = [] создаёт пустой список, куда будут добавляться слова из файла.
Открытие файла для чтения:
    with open(filename, "r", encoding="utf-8") as myfile: открывает файл с указанным именем в режиме чтения ("r"), с кодировкой UTF-8.
    with обеспечивает автоматическое закрытие файла после выполнения блока.
Чтение всех строк файла:
    words_raw = myfile.readlines() читает все строки файла и сохраняет их в список words_raw. Каждая строка файла отдельный элемент этого списка.
Обработка каждой строки:
    for word in words_raw: перебирает каждую строку файла.
    words_out.append(word[:-1]) добавляет в список words_out подстроку word[:-1].
Закрытие файла:
    myfile.close() вручную закрывать файл не обязательно, так как with уже обеспечивает автоматическую закрытие файла. В этом случае вызов close() излишен и может быть удалён.
Возвращение результата:
    return words_out — функция возвращает список слов без символов переноса строки.