# Привет! Это ревью для тебя, Дмитрий.

**Начну с общих классов, закончу конкретными.**

### 1. Main

1. Не используй [статику](https://github.com/dmitry-shuplev/gallow/blob/470f90e91cd380c82fdb3c201270c1517d39771e/src/Main.java#L16) если только речь не о вспомгательных классах, по типу: Math, HibernateUtil (где просто отдаётся свободные в пуле сессий - сессия). Почему? 

Потому что очень сложно понять зависимости класса. Когда у тебя все используемые зависимости в полях - я сразу могу прочесть твой класс и сказать, что там используется. Если ты используешь статику, значит ты не используешь поля для хранения нашей зависимости, а следовательно я не могу заранее понять что ты используешь. Это неудобно. 

Нельзя переопределить метод используя наследование и поломорфизм.

2. [Тут](https://github.com/dmitry-shuplev/gallow/blob/470f90e91cd380c82fdb3c201270c1517d39771e/src/Main.java#L19) у тебя нет вертикальных и горизонтальных отступов

Надо так:

```java
char inputedChar = getChar();
            
if (!isCharCorrect(game, inputedChar)) {
    Show.charError();
    continue;
}
            
procesedGameSession(game, inputedChar);
```

Просто сочетание ctrl + k + d на раскладке Visual Studio используй. И он сам горизонтальные отступы делать будет. А вертикальные сам используй. Когда их использовать? Когда между двумя строками находится блок: if, while, for.

3. Смотрим [этот](https://github.com/dmitry-shuplev/gallow/blob/470f90e91cd380c82fdb3c201270c1517d39771e/src/Main.java#L27) метод:

Определение модификатор доступа private если нигде больше метод не используешь (и так со всем другими методами и полями класса). package defaul позволит мне использовать эти методы и поля использовать в других классах этого же пакета

```java
    static void compareResult(GameSession game) {
        if (game.hiddenWord.equals(game.unhideenWord)) {
            Show.winGame(game);
            System.exit(0);
        }
        if (game.errors == 6) {
            Show.loseGame(game);
            System.exit(0);
        }
    }
```

Снова нет вертикального отступа.
Магические значения (цифры в данном случае), речь о "6". Определяй контстанты для таких случаев.

Не используй System.exit(0) - так ты резко можешь прервать всё приложение, а это плохо, потому что нам может потребоваться отобразить какой - то вывод в консоль или закрыть правильно ресурсы. А это не получится сделать из резкого закрытия всего приложения.

Ещё твой код можно было бы сократить:

```java
    private static void compareResult(GameSession game) {
        if (game.hiddenWord.equals(game.unhideenWord)) {
            Show.winGame(game);
        }

        if (game.errors == 6) {
            Show.loseGame(game);
        }

        System.exit(0);
    }
```
4. Теперь этот метод:

```java
    static char getChar() {
        Scanner scanner = new Scanner(new InputStreamReader(System.in, StandardCharsets.UTF_8));
        System.out.print("Введите букву: ");
        char curentChar = scanner.nextLine().toUpperCase().charAt(0);
        return curentChar;
    }
```

Тут всё также: нет модификатора. 
Вот эту логику ввода значений надо вынести в другой класс (UserInput). Удобство в томт, что можно заменять динамически реализации ввода данных. Ведь у тебя не только может быть такое, что пользователь вводит значения с консоли, но а также может быть так, что ввод происходит с другого UI.

5. Этот метод
```java
    static void procesedGameSession(GameSession game, char inputedChar) {
        int matchCounter = 0;
        String unhiddentWord = "";
        for (int i = 0; i < game.hiddenWord.length(); i++) {
            if (game.hiddenWord.charAt(i) == inputedChar) {
                unhiddentWord += inputedChar;
                matchCounter++;
            } else {
                unhiddentWord += game.unhideenWord.charAt(i);
            }
        }
        if (matchCounter == 0) {
            game.errors++;
        }
        for (int i = 0; i < game.avalibleChars.length(); i++) {
            if (game.avalibleChars.charAt(i) == inputedChar) {
                game.avalibleChars.deleteCharAt(i);
            }
        }
        game.unhideenWord = unhiddentWord;
        game.usedChar += inputedChar;
    }
```
Нет отступов.
Этот метод должен быть в классе game. Причём hidden word у тебя в виде строки, тут явно прослеживается что часть метода должна быть в классе HiddenWord. Тут вообще дофига чего в одном месте.

Давай начнём сначала:
Проверяем соответствие что в hidden word есть буквы введённая пользователем.
Смотрим количество ошибок пользователем.
Скрываем от пользователя в скрытом слове угаданные буквы.

Это слишком жирный метод. Надо его декомпозировать.

6. Этот метод:

```java
static boolean isCharCorrect(GameSession game, char currenChar) {
        for (int i = 0; i < game.avalibleChars.length(); i++) {
            if (game.avalibleChars.charAt(i) == currenChar) {
                return true;
            }
        }
        return false;

        //Можно избежать двойного вызова провеки наличия символа если возвращать не булево значение,
        // а int с номером символа.
        //В случае если символа нет возвращать некорректное значение, -1. Но поскольку мне делались замечания
        // по чистоте кода делаю так. Если не прав поправьте.
    }
```
Тут есть комментарии для тебя. В продакшене их надо убирать. Раз ты выложил проект на обозрение, то убирай их. За это минус.
Снова метод должен быть в другом классе. 

### Класс Game Session

1. 
```java
public class GameSession {

    int errors = 0;
    String hiddenWord = "";
    String unhideenWord = "";
    String usedChar = "";
    StringBuilder avalibleChars = new StringBuilder("АБВГДЕЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ");
```
Поля должны быть private как я писал выше.
Тут hiddenWord, unhideenWord, usedChar, avalibleChars - всё это в другой класс выноси исходя из логики.

```java
 new StringBuilder("АБВГДЕЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ"); // есть возможность сократить код, используя регулярные выражения.

 new StringBuilder("[А-ЯЁ]");
```
2. Вынеси метод в другой класс

```java
    void generateUnhiddenWord() {
        for (int i = 0; i < hiddenWord.length(); i++) {
            unhideenWord += '*';
        }
    }
```

3. void convencedWord() слишком дофига чего делает, декомпозируй.

И вынеси в класс Dictionary

№ Класс VocabularProcessor

1. Вынеси условие: !line.trim().isEmpty() && line.length()>=4, в отдельный метод, иначе тяжело читать. Легче название прочесть и понять его суть
2. Снова нет вертикальных отступов в методе
3. Если у тебя константы не используются нигде, то сделай их приватными
4. Да и название класс лучше Dictionary

# Show класс

1. Молодец что вынес UI в отдельный класс, но картинки вынеси в другой класс
2. Снова комментирии убирай
3. Снова нет отсутпов, снова ctrl + k + d
4. Снова магические числа, убирай
5. Снова магические строки, убирай
6. Картинки вообще лучше убери в файл и сканируй его.


Это всё




