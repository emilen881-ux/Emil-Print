# Исправление функции generateESCPOS

## Проблема
Функция `generateESCPOS` в файле `index.html` (строки 1456-1486) использует неправильную кодировку для русского текста.

Термопринтер Xprinter XP-365B требует кодировку CP866 (DOS Cyrillic), но текущий код использует UTF-8 через `TextEncoder`.

## Решение
Замени функцию `generateESCPOS` (строки 1456-1486) на эту исправленную версию:

```javascript
function generateESCPOS(label) {
    // CP866 encoding map for Cyrillic
    const cp866map = {
        'А':128,'Б':129,'В':130,'Г':131,'Д':132,'Е':133,'Ж':134,'З':135,
        'И':136,'Й':137,'К':138,'Л':139,'М':140,'Н':141,'О':142,'П':143,
        'Р':144,'С':145,'Т':146,'У':147,'Ф':148,'Х':149,'Ц':150,'Ч':151,
        'Ш':152,'Щ':153,'Ъ':154,'Ы':155,'Ь':156,'Э':157,'Ю':158,'Я':159,
        'а':160,'б':161,'в':162,'г':163,'д':164,'е':165,'ж':166,'з':167,
        'и':168,'й':169,'к':170,'л':171,'м':172,'н':173,'о':174,'п':175,
        'р':224,'с':225,'т':226,'у':227,'ф':228,'х':229,'ц':230,'ч':231,
        'ш':232,'щ':233,'ъ':234,'ы':235,'ь':236,'э':237,'ю':238,'я':239,
        'Ё':240,'ё':241
    };
    
    function encodeCP866(text) {
        let result = [];
        for (let char of text) {
            if (cp866map[char]) {
                result.push(cp866map[char]);
            } else {
                result.push(char.charCodeAt(0));
            }
        }
        return result;
    }
    
    const ESC = 0x1B;
    const GS = 0x1D;
    
    let commands = [];
    
    // Initialize printer
    commands.push(ESC, 0x40);
    
    // Set code page to CP866 (Cyrillic)
    commands.push(ESC, 0x74, 17);
    
    // Center alignment
    commands.push(ESC, 0x61, 1);
    
    // Line 1 - Bold and larger
    if (label.line1) {
        commands.push(ESC, 0x45, 1); // Bold ON
        commands.push(GS, 0x21, 0x11); // Double size
        commands.push(...encodeCP866(label.line1));
        commands.push(0x0A); // Line feed
    }
    
    // Reset formatting
    commands.push(ESC, 0x45, 0); // Bold OFF
    commands.push(GS, 0x21, 0x00); // Normal size
    
    // Lines 2-5 - Normal
    if (label.line2) {
        commands.push(...encodeCP866(label.line2));
        commands.push(0x0A);
    }
    if (label.line3) {
        commands.push(...encodeCP866(label.line3));
        commands.push(0x0A);
    }
    if (label.line4) {
        commands.push(...encodeCP866(label.line4));
        commands.push(0x0A);
    }
    if (label.line5) {
        commands.push(...encodeCP866(label.line5));
        commands.push(0x0A);
    }
    
    // Feed and cut
    commands.push(0x0A, 0x0A, 0x0A);
    commands.push(GS, 0x56, 66, 0); // Partial cut
    
    return new Uint8Array(commands);
}
```

## Что изменилось

1. **Добавлена таблица CP866** - полная карта перекодировки русских букв в CP866
2. **Функция encodeCP866()** - конвертирует каждый символ в правильный байт CP866
3. **Команда установки кодовой страницы** - `ESC t 17` устанавливает CP866 в принтере
4. **Возврат Uint8Array** - вместо строки возвращаем массив байтов напрямую

## Инструкция

1. Открой файл `index.html`
2. Найди функцию `generateESCPOS` (строка 1456)
3. Выдели весь код функции до строки 1486 (с закрывающей скобкой `}`)
4. Замени выделенный код на код выше
5. Сохрани файл
6. Проверь печать - русский текст должен теперь отображаться правильно!

## Проверка

После замены функции:
- Подключи термопринтер Xprinter XP-365B через USB
- Нажми "Подключить USB принтер"
- Выбери бирку и нажми "Печать через USB"
- Русский текст должен печататься правильно!
