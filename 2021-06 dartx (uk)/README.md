# Як покращити Dart-колекції з бібліотекою dartx

У мові програмування Dart класи `Iterable` та `List` надають базовий набір методів для модифікації та фільтрації списків. Однак відсутність бібліотеки на кшталт [LINQ-to-Objects](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects) у C# у багатьох випадках залишає розробників без лаконічних однорядкових операцій. Звісно ж, будь-яку задачу можна вирішити, використовуючи лише методи з `dart.core`, проте нерідко це потребує рішення в декілька рядків, і такий код виходить неочевидним і складнішим для розуміння.

![](images/cover_image.png)

Бібліотека **[dartx](https://pub.dev/packages/dartx)** надає розробникам можливість комбінування і використання елегантних та читабельних однорядкових операцій над колекціями (та іншими типами у Dart) завдяки механізму екстеншенів.

Зараз ми розглянемо, як наступні задачі вирішуються за допомогою базових методів з мови програмування Dart та з використанням методів бібліотеки [dartx](https://pub.dev/packages/dartx):

* отримати перший / останній елемент списку або null / значення за замовчуванням / за умовою;
* трансформувати / відфільтрувати / відвідати елементи списку в залежності від їх індексу;
* перетворити список на мапу;
* впорядкувати список;
* виокремити унікальні елементи списку;
* отримати найменше / найбільше / середнє арифметичне значення певного атрибута елементів списку;
* вилучити елементи списку, що є `null`.

Та розглянемо декілька додаткових корисних екстеншенів.

Всі приклади вимагають посилання на пакет [dartx](https://pub.dev/packages/dartx) у `pubspec.yaml` файлі, а також `import` у файлі з кодом:

```dart
import 'package:dartx/dartx.dart';
```

![](images/break_image.png)

## Отримання першого / останнього елемента списку...

### … або null

Цей типовий код для отримання першого та останнього елементу списку у чистому Dart:

```dart
final first = list.first;

final last = list.last;
```

призведе до ексепшену `StateError` під час виконання програми, якщо список `list` порожній. Щоб його позбутися, доведеться перевіряти чи список не порожній і явно повертати `null`:

```dart
final firstOrNull = list.isNotEmpty ? list.first : null;

final lastOrNull = list.isNotEmpty ? list.last : null;
```

З використанням [dartx](https://pub.dev/packages/dartx) ця задача спрощується до:

```dart
final firstOrNull = list.firstOrNull;

final lastOrNull = list.lastOrNull;
```

Аналогічно, цей код поверне `null`, якщо `index` поза межами `list`:

```dart
final elementAtOrNull = list.elementAtOrNull(index);
```

### … або значення за замовчуванням

Пам’ятаючи, що гетери `.first` та `.last` призведуть до ексепшена, коли список `list` порожній, отримання першого та останнього елементів або вказаного значення у чистому Dart має такий вигляд:

```dart
final firstOrDefault = (list.isNotEmpty ? list.first : null) ?? defaultValue;

final lastOrDefault = (list.isNotEmpty ? list.last : null) ?? defaultValue;
```

З пакетом [dartx](https://pub.dev/packages/dartx):

```dart
final firstOrDefault = list.firstOrDefault(defaultValue);

final lastOrDefault = list.lastOrElse(defaultValue);
```

Аналогічно, цей код поверне `defaultValue`, якщо `index` поза межами `list`:

```dart
final elementAtOrDefault = list.elementAtOrDefault(index, defaultValue);
```

### … за умовою

Таким є код для одержання першого та останнього елементів списку, що відповідають деякій умові або `null` у чистому Dart:

```dart
final firstWhere = list.firstWhere((x) => x.matches(condition));

final lastWhere = list.lastWhere((x) => x.matches(condition));
```

Неважко здогадатись, що і цей код може призводити до `StateError` ексепшену. Цю ситуацію можна виправити параметром ``orElse``:

```dart
final firstWhereOrNull = list.firstWhere((x) => x.matches(condition), orElse: () => null);

final lastWhereOrNull = list.lastWhere((x) => x.matches(condition), orElse: () => null);
```

З [dartx](https://pub.dev/packages/dartx) імплементація скорочується до:

```dart
final firstWhereOrNull = list.firstOrNullWhere((x) => x.matches(condition));

final lastWhereOrNull = list.lastOrNullWhere((x) => x.matches(condition));
```

![](images/break_image.png)

## … елементів списку в залежності від їх індексу

### Трансформація…

Нерідкою є необхідність формування нового списку, де кожен елемент певним чином залежить від свого індексу в оригінальному списку. Для прикладу, розглянемо задачу трансформації кожного елементу у рядок `String` з додаванням його індексу.

Це один з варіантів реалізації на чистому Dart:

```dart
final newList = list.asMap()
  .map((index, x) => MapEntry(index, '$index $x'))
  .values
  .toList();
```

З [dartx](https://pub.dev/packages/dartx) все набагато простіше:

```dart
final newList = list.mapIndexed((index, x) => '$index $x').toList();
```

Цей код містить `.toList()`, бо цей та більшість інших методів повертають ліниві `Iterable`.

### Фільтрація…

Для іншого прикладу розглянемо задачу колекціонування тільки елементів з непарними індексами.

На чистому Dart це:

```dart
final oddItems = [];
for (var i = 0; i < list.length; i++) {
  if (i.isOdd) {
    oddItems.add(list[i]);
  }
}
```

Або в одному ланцюзі:

```dart
final oddItems = list.asMap()
  .entries
  .where((entry) => entry.key.isOdd)
  .map((entry) => entry.value)
  .toList();
```

З [dartx](https://pub.dev/packages/dartx) це:

```dart
final oddItems = list.whereIndexed((x, index) => index.isOdd).toList();
```

або ж:

```dart
final oddItems = list.whereNotIndexed((x, index) => index.isEven).toList();
```

### Відвідання…

Як вивести вміст списку, вказуючи серед іншого індекси елементів?

На чистому Dart:

```dart
for (var i = 0; i < list.length; i++) {
  print('$i: ${list[i]}');
}
```

З [dartx](https://pub.dev/packages/dartx):

```dart
list.forEachIndexed((element, index) => print('$index: $element'));
```

![](images/break_image.png)

## Перетворення списку на мапу

Припустимо, необхідно конвертувати список унікальних об’єктів `Person` на `Map<String, Person>`, де ключами є `person.id`, а значеннями — повні об’єкти `Person`.

На чистому Dart це має такий вигляд:

```dart
final peopleMap = people.asMap().map((index, person) => MapEntry(person.id, person));
```

З [dartx](https://pub.dev/packages/dartx):

```dart
final peopleMap = people.associate((person) => MapEntry(person.id, person));
```

або:

```dart
final peopleMap = people.associateBy((person) => person.id);
```

А ось як виглядає реалізація задачі формування мапи, де ключами є дата `DateTime`, а значеннями — список `List<Person>` людей, які народились у цей день.

На чистому Dart:

```dart
final peopleMapByBirthDate = people.fold<Map<DateTime, List<Person>>>(
  <DateTime, List<Person>>{}, 
  (map, person) {
    if (!map.containsKey(person.birthDate)) {
      map[person.birthDate] = <Person>[];
    }
    map[person.birthDate].add(person);
    return map;
  },
);
```

Значно простішим є варіант з використанням [dartx](https://pub.dev/packages/dartx):

```dart
final peopleMapByBirthDate = people.groupBy((person) => person.birthDate);
```

![](images/break_image.png)

## Впорядкування списку

При використанні методу з чистого Dart потрібно пам’ятати, що:

```dart
list.sort();
```

модифікує оригінальний список `list`, та щоб отримати нову колекцію, потрібно змінити код на:

```dart
final orderedList = [...list]..sort();
```

[dartx](https://pub.dev/packages/dartx) надає метод, що повертає новий екземпляр списку:

```dart
final orderedList = list.sorted();
```

та:

```dart
final orderedDescendingList = list.sortedDescending();
```

Припустимо, необхідно впорядкувати список на основі деякої властивості елементів.

На чистому Dart:

```dart
final orderedPeople = [...people]
  ..sort((person1, person2) => person1.birthDate.compareTo(person2.birthDate));
```

З [dartx](https://pub.dev/packages/dartx):

```dart
final orderedPeople = people.sortedBy((person) => person.birthDate);
```

та:

```dart
final orderedDescendingPeople = people.sortedByDescending((person) => person.birthDate);
```

До речі, з [dartx](https://pub.dev/packages/dartx) також можливо впорядковувати списки за кількома ознаками:

```dart
final orderedPeopleByAgeAndName = people
  .sortedBy((person) => person.birthDate)
  .thenBy((person) => person.name);
```

та:

```dart
final orderedDescendingPeopleByAgeAndName = people
  .sortedByDescending((person) => person.birthDate)
  .thenByDescending((person) => person.name);
```

![](images/break_image.png)

## Виокремлення унікальних елементів списку

Один з варіантів вирішення цієї задачі на чистому Dart такий:

```dart
final unique = list.toSet().toList();
```

Ця реалізація не гарантує збереження порядку розташування елементів, а альтернативою є тільки рішення у кілька рядків.

З [dartx](https://pub.dev/packages/dartx) реалізація спрощується до:

```dart
final unique = list.distinct().toList();
```

та:

```dart
final uniqueFirstNames = people.distinctBy((person) => person.firstName).toList();
```

![](images/break_image.png)

## Найменше / найбільше / середнє арифметичне значення певного атрибута елементів списку

Для отримання найменшого / найбільшого елемента списку можна, наприклад, його впорядкувати й взяти перший / останній елементи:

```dart
final min = ([...list]..sort()).first;

final max = ([...list]..sort()).last;
```

Той самий підхід можна використати для впорядкування за певною ознакою:

```dart
final minAge = (people.map((person) => person.age).toList()..sort()).first;

final maxAge = (people.map((person) => person.age).toList()..sort()).last;
```

або ж:

```dart
final youngestPerson = 
  ([...people]..sort((person1, person2) => person1.age.compareTo(person2.age))).first;

final oldestPerson = 
  ([...people]..sort((person1, person2) => person1.age.compareTo(person2.age))).last;
```

Неважко здогадатись, що [dartx](https://pub.dev/packages/dartx) пропонує значно простіші рішення, які, до речі, повернуть `null` для порожнього списку:

```dart
final youngestPerson = people.minBy((person) => person.age);

final oldestPerson = people.maxBy((person) => person.age);
```

Якщо елементи списку реалізують `Comparable`, можна також використовувати методи без селекторів:

```dart
final min = list.min();

final max = list.max();
```

Також не є складними задачі отримання середнього арифметичного:

```dart
final average = list.average();
```

або ж:

```dart
final averageAge = people.averageBy((person) => person.age);
```

та суми `num` елементів списку:

```dart
final sum = list.sum();
```

або `num` властивостей комплексних елементів:

```dart
final totalChildrenCount = people.sumBy((person) => person.childrenCount);
```

![](images/break_image.png)

## Вилучення елементів списку, що є null

У чистому Dart:

```dart
final nonNullItems = list.where((x) => x != null).toList();
```

З [dartx](https://pub.dev/packages/dartx):

```dart
final nonNullItems = list.whereNotNull().toList();
```

![](images/break_image.png)

## Більше корисних екстеншенів

Бібліотека [dartx](https://pub.dev/packages/dartx) надає широкий набір корисних методів. Надалі ми не будемо зупинятися на кожному детально, проте я сподіваюсь, що назви і код достатньо очевидні.

### joinToString

```dart
final report = people.joinToString(
  separator: '\n',
  transform: (person) => '${person.firstName}_${person.lastName}',
  prefix: '<<️',
  postfix: '>>',
);
```

### all (every) / none

```dart
final allAreAdults = people.all((person) => person.age >= 18);

final allAreAdults = people.none((person) => person.age < 18);
```

### first / second / third / fourth collection items

```dart
final first = list.first;
final second = list.second;
final third = list.third;
final fourth = list.fourth;
```

### takeFirst / takeLast

```dart
final youngestPeople = people.sortedBy((person) => person.age).takeFirst(5);

final oldestPeople = people.sortedBy((person) => person.age).takeLast(5);
```

### firstWhile / lastWhile

```dart
final orderedPeopleUnder50 = people
  .sortedBy((person) => person.age)
  .firstWhile((person) => person.age < 50)
  .toList();

final orderedPeopleOver50 = people
  .sortedBy((person) => person.age)
  .lastWhile((person) => person.age >= 50)
  .toList();
```

![](images/break_image.png)

## І наостанок

Додам, що бібліотека [dartx](https://pub.dev/packages/dartx) пропонує ще більше зручних екстеншенів для [`Iterable`](https://github.com/leisim/dartx/blob/master/lib/src/iterable.dart), [`List`](https://github.com/leisim/dartx/blob/master/lib/src/list.dart) та [інших типів](https://github.com/leisim/dartx/tree/master/lib/src) у Dart. Найкращим способом ознайомитися з усіма її можливостями є перегляд [програмного коду](https://github.com/leisim/dartx).

Викажіть авторам пакету підтримку, поставивши йому 👍🏻 на [pub.dev](https://pub.dev/packages/dartx) та ⭐️ на [GitHub](https://github.com/leisim/dartx).

![](images/break_image.png)

*[Опубліковано](https://dou.ua/forums/topic/34098/) на DOU у червні 2021.*
