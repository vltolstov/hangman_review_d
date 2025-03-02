# Минусы

## 1. Отступы

![image](https://github.com/user-attachments/assets/854cbb3b-ac37-46c7-80bb-d05ca727e8ee)
If - ы должны быть разделены отступами

**В конце ты оставляешь пустой отступ перед окончанием класса, так выглядит ужасно**

_Вот как надо:_

![image](https://github.com/user-attachments/assets/49fecd14-b60e-416c-aa89-5598eb44c6bf)

Тут также:
![image](https://github.com/user-attachments/assets/4cb94df0-6b8c-4bcb-ba54-35c8dc294b5f)

## 2. Исключения должны быть написаны на английском. Щас у тебя они описаны на русском

![image](https://github.com/user-attachments/assets/15dc11d2-172f-417e-b8e0-7f8bdb0b71af)

## 3. Присутствуют магические значения (я про 1 и regex шаблон):

```java
  if (letter.length() != 1) {
            throw new IllegalArgumentException("Введите только одну букву");
  }

  if (!letter.matches("[а-яё-]+")) {
            throw new IllegalArgumentException("Только кириллица маленькими буквами");
  }
```

# 4. Переменная слиплась с блоком if:

```java

    @Override
    public void validate(String file) {
        File filePath = new File(file);
        if (!filePath.exists()) {
            throw new IllegalArgumentException("Такой файл не существует");
        }
```
Надо так:

```java

    @Override
    public void validate(String file) {
        File filePath = new File(file);

        if (!filePath.exists()) {
            throw new IllegalArgumentException("Такой файл не существует");
        }
```

# 5. Разделитель

```java
    public HiddenWord(String word) {
        this.word = word;
        this.revealedWord = new StringBuilder("*".repeat(word.length()));
    }
```
`*` стоит определить в константу. Почему? Так легче будет найти separator и исправить его в одном месте. Вполне возможно ты будешь его использовать. Да и в целом код будет читабельней если подберёшь правильное название.

# 6. Выделяй методы для больших строк

```java
        this.revealedWord = new StringBuilder("*".repeat(word.length()));
```
Определение `revealWord` должно быть короче. Не то чтобы это сложно читать, но если как - то логика усложнится или в целом я не хочу вдумываться, то дай название нормальное методу, и пусть он возвращает значение.

# 7. Определяй названия каких - либо сущностей наиболее простыми и часто встречаемыми словами. 
Тут лучше назвать mask. Либо hiddenWord. Дело вкусовщины.

# 8. Определяй параметры методов как final. Они не должны меняться в процессе работы метода.

# 9. Непонятная логика

```java
public interface WordSource {

    String getRandomWord();

}
```

Я когда читал код реализующий данный метод, думал, что интерфейс - это источник слов. В то время как тут метод `getRandomWord()`. Создай лучше генератор рандомных слов и его используй. А то тут вообще хз что это такое. То что выделяешь абстракцию молодец, но так делать не надо. Логично было бы увидеть интерфейс `Dictionary` и класс его реализующий `FileDictionaryProvider`. 

# 10. try-catch-finally --- урод из цирка

```java
    private void loadDictionary() {
        try {
            dictionary = Files.readAllLines(dictionaryFilePath);
        } catch (IOException e) {
            dictionary = List.of("акула", "рюкзак", "тетрадь", "шимпанзе", "клавиатура", "человеконенавистничество");
            throw new RuntimeException("Ошибка чтения словаря: " + e.getMessage());
        }
        dictionary = filterDictionaryByDifficult(dictionary, difficulty);
    }
```

Взгляни на это, неправда ли хочется просраться после такого? 
Я лично сам посрал с кайфом. А ты? 
Так вот, в try-catch-finally надо использовать только методы. Что я имею в виду? В нём не должно быть несколоько строк кода или полноценный код. Только метод. 

```java
    private void loadDictionary() {
        try {
            dictionary = readAllLines(dictionaryFilePath);
        } catch (IOException e) {
            dictionary = getDefaultDictionary();
            throw new RuntimeException("Ошибка чтения словаря: " + e.getMessage());
        }

        dictionary = filterDictionaryByDifficult(dictionary, difficulty);
    }
```

# 11. Неправильно поля расставил:

Было:

```java
public class FileWordProvider implements WordSource {

    private final Path dictionaryFilePath;
    private List<String> dictionary;
    private final Difficulty difficulty;

    private final Random random;
```

Стало: 

```java
public class FileWordProvider implements WordSource {

    private final Path dictionaryFilePath;
    private final Random random;
    private final Difficulty difficulty;

    private List<String> dictionary;
```

Всё что с `final` в начале. Всё что без final идёт после, причём через отступ (можно с ним, но мне так больше нравится)

# 12. Магические цифры, убирай

```java
    private List<String> filterDictionaryByDifficult(List<String> dictionary, Difficulty difficulty) {
        return dictionary.stream()
                .filter(word -> switch (difficulty) {
                    case EASY -> word.length() >= 5 && word.length() <= 6;
                    case MEDIUM -> word.length() >= 7 && word.length() <= 9;
                    case HARD -> word.length() >= 10;
                })
                .toList();
    }
```

# 13. Класс Game имеет сликшом много полей. Создавай сущности которые внедрят в себя поля, и таким образом уменьши их до 3-х. Сам код тоже очень жирный. Даже читать не хочется.

# 14. Я не понимаю разницы между двумя этими методами о `print`. Что значит guessed и просто letter? Пс

```java
    private int processUserGuess(HiddenWord hiddenWord, char letter, int attempts) {
        if (hiddenWord.guessLetter(letter)) {
            printer.printLetterGuessedMessage();
        } else {
            printer.printWrongLetterMessage();
            attempts--;
        }

        return attempts;
    }
```
 
# 15. В экран не помещается больше 3-х слов в параметрах. Делай так и жизнь станет проще:

```java
    private int processUserGuess(final HiddenWord hiddenWord,
                                 final char letter,
                                 final int attempts) {
        
```

# 16. Снова ошибка с отступами и try-catch (читай выше)

```java
            String input = scanner.nextLine();
            try {
                validator.validate(input);
                validator.checkLetterNotUsed(input.charAt(0), usedLetters);
            } catch (IllegalArgumentException e) {
                System.out.println("Ошибка: " + e.getMessage() + "\n");
                continue;
            }
```

# 17. Если ты так делаешь отступы между полями, значит они явно требуют чтобы был создан класс который их вмещает

```java
public class GameLauncher {

    private final Settings settings;
    private final SettingsConsoleUI settingsUI;

    private final ConsolePrinter printer;
    private final Scanner scanner;
```

# 18. Не понимаю, что значат цифры в кавычках. Сделай константы. И зачем ты вообще сунул пункт 4, чтобы потом использовать `return`? Убирай этот пункт.

```java
    public void run() {
        while (true) {
            printer.printGameMenu();

            String action = scanner.nextLine();

            switch (action) {
                case "1" -> new Game(settings).loop();
                case "2" -> settingsUI.showActionMenu(settings);
                case "3" -> new Statistics().printStats();
                case "4" -> {
                    return;
                }
                default -> printer.printWrongInputError();
            }
        }
    }
```

# 19. Почему не юзаешь this для полей?

```java
    public Game(Settings settings) {
        this.usedLetters = new ArrayList<>();
        this.settings = settings;
        this.attempts = settings.getMaxAttempts();
        statistics = new Statistics();
```
речь о `statistics`

Тут тоже самое. Выглядит колхозно, без обид.

```java

        validator = new LetterValidatorImpl();
        printer = new ConsolePrinter();
```

# 20. Если переменная может быть final, то делай её final. Так станет яснее, что она не меняется:

```java
         boolean won = hiddenWord.isWordGuessed();
```

# 21. Вытаскивай try-catch-finally в методы. Посмотри как много места он занимает.

![image](https://github.com/user-attachments/assets/af61e7e4-3479-42b1-9754-bb919b74ae8c)

# 22. Я понял, что такое "4" - ты так игру закрываешь. Сразу наткунлся на мысль, что это меню, и оно не должно быть в GameLauncher, у него другая ответственность. И лучше сделай булевую переменную, по типу `isRunning`.

![image](https://github.com/user-attachments/assets/d33b4dd0-efaa-41b3-9f7e-54dbbd683b77)

# 23. Statistics выглядит жирным. Причём несколько методов не его ответственность: `getAverageAttempts` и `getAverageTimeSpent`, и `printStats`

# 24. Слишком здоровый интерфейс. Прочти про 4-й принцип SOLID.

![image](https://github.com/user-attachments/assets/75f26127-b3ce-493f-a8d8-141ae22567dd)

# 25. Посмотри все замечания и каждый класс поправь, окей? Там магические числа, try-catch и т.д.

# 26. Почему мы бы для статов не использовать комментарии? (для многих вещей их юзать не стоит, но я не понимаю, чё такое 0)

![image](https://github.com/user-attachments/assets/2ef84bd7-4559-4c84-9be0-74f3c01b7bef)


