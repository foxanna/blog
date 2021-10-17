# Dart collections with DartX extensions

Dart's `Iterable` and `List` offer a basic set of methods to modify and query collections. However, coming from the C# background, which has a marvelous [LINQ-to-Objects](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/linq-to-objects) library, I felt that a part of the functionality that I was using on a daily basis is missing. Of course, any task can be solved with the `dart.core` methods only, but it sometimes requires multi-line solutions, and the code is not so obvious and takes effort to understand. We all know that developers spend a lot of time reading the code, and it is crucial to have it concise and obvious.

![](images/cover_image.png)

**[dartx](https://pub.dev/packages/dartx)** package allows developers to use elegant and readable single-line operations on collections (and other Dart types).

Below we’ll compare how these tasks are solved with plain Dart vs. [dartx](https://pub.dev/packages/dartx):

* first / last collection item or null / default value / by condition;
* map / filter / iterate collection items depending on their index;
* converting a collection to a map;
* sorting collections;
* collecting unique items;
* min / max / average by items property;
* filtering out null items;

and look at some additional useful extensions.

All examples require `dartx` dependency in `pubspec.yaml` file and

```dart
import 'package:dartx/dartx.dart';
```

![](images/break_image.png)

## First / last collection item…

### … or null

To get the first and last collection items in plain Dart one would write:

```dart
final first = list.first;

final last = list.last;
```

which throws a `StateError` if `list` is empty. Or explicitly return `null`:

```dart
final firstOrNull = list.isNotEmpty ? list.first : null;

final lastOrNull = list.isNotEmpty ? list.last : null;
```

Using [dartx](https://pub.dev/packages/dartx) allows:

```dart
final firstOrNull = list.firstOrNull;

final lastOrNull = list.lastOrNull;
```

Similarly:

```dart
final elementAtOrNull = list.elementAtOrNull(index);
```

returns `null` if the `index` is out of bounds of the `list`.

### … or default value

Given you now remember that `.first` and `.last` getters throw errors when `list` is empty, to get the first and last collections items or default value, in plain Dart you’d write:

```dart
final firstOrDefault = (list.isNotEmpty ? list.first : null) ?? defaultValue;

final lastOrDefault = (list.isNotEmpty ? list.last : null) ?? defaultValue;
```

Using [dartx](https://pub.dev/packages/dartx):

```dart
final firstOrDefault = list.firstOrDefault(defaultValue);

final lastOrDefault = list.lastOrElse(defaultValue);
```

Similar to `elementAtOrNull`:

```dart
final elementAtOrDefault = list.elementAtOrDefault(index, defaultValue);
```

returns `defaultValue` if the `index` is out of bounds of the `list`.

### … by condition

To get the first and last collection items that match some condition or `null`, a plain Dart implementation would be:

```dart
final firstWhere = list.firstWhere((x) => x.matches(condition));

final lastWhere = list.lastWhere((x) => x.matches(condition));
```

which will throw `StateError` for empty list unless `orElse` is provided:

```dart
final firstWhereOrNull = list.firstWhere((x) => x.matches(condition), orElse: () => null);

final lastWhereOrNull = list.lastWhere((x) => x.matches(condition), orElse: () => null);
```

[dartx](https://pub.dev/packages/dartx) shortens the implementation to:

```dart
final firstWhereOrNull = list.firstOrNullWhere((x) => x.matches(condition));

final lastWhereOrNull = list.lastOrNullWhere((x) => x.matches(condition));
```

![](images/break_image.png)

## … collection items depending on their index

### Map…

It’s not a rare case when you need to obtain a new collection where each item somehow depends on its index. For example, each new item is a string representation of an item from the original collection + its index.

If you like me prefer one-liners, in plain Dart it would be:

```dart
final newList = list.asMap()
  .map((index, x) => MapEntry(index, '$index $x'))
  .values
  .toList();
```

With [dartx](https://pub.dev/packages/dartx) it’s much simpler:

```dart
final newList = list.mapIndexed((index, x) => '$index $x').toList();
```

I applied `.toList()` because this and most other extension methods return lazy `Iterable`.

### Filter…

For another example, let’s say there is a need to collect only odd-indexed items. With plain Dart, this can be implemented like:

```dart
final oddItems = [];
for (var i = 0; i < list.length; i++) {
  if (i.isOdd) {
    oddItems.add(list[i]);
  }
}
```

Or in one line:

```dart
final oddItems = list.asMap()
  .entries
  .where((entry) => entry.key.isOdd)
  .map((entry) => entry.value)
  .toList();
```

With [dartx](https://pub.dev/packages/dartx) it’s only:

```dart
final oddItems = list.whereIndexed((x, index) => index.isOdd).toList();
```

or:

```dart
final oddItems = list.whereNotIndexed((x, index) => index.isEven).toList();
```

### Iterate…

How would you log collection content specifying item indexes?

In plain Dart:

```dart
for (var i = 0; i < list.length; i++) {
  print('$i: ${list[i]}');
}
```

With [dartx](https://pub.dev/packages/dartx):

```dart
list.forEachIndexed((element, index) => print('$index: $element'));
```

![](images/break_image.png)

## Converting a collection to a map

For example, there is a need to convert a list of distinct `Person` objects to a `Map<String, Person>` where keys are `person.id`, and values are whole person instances.

In plain Dart:

```dart
final peopleMap = people.asMap().map((index, person) => MapEntry(person.id, person));
```

With [dartx](https://pub.dev/packages/dartx):

```dart
final peopleMap = people.associate((person) => MapEntry(person.id, person));
```

or:

```dart
final peopleMap = people.associateBy((person) => person.id);
```

To get a map where keys are `DateTime` and values are `List<Person>` of people who were born that day, in plain Dart you’d write:

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

It’s much simpler with [dartx](https://pub.dev/packages/dartx):

```dart
final peopleMapByBirthDate = people.groupBy((person) => person.birthDate);
```

![](images/break_image.png)

## Sorting collections

How would you sort a collection in plain Dart? You have to keep in mind that

```dart
list.sort();
```

modifies the source collection, and to get a new instance you’d have to write:

```dart
final orderedList = [...list]..sort();
```

[dartx](https://pub.dev/packages/dartx) provides an extension to get a new `List` instance:

```dart
final orderedList = list.sorted();
```

and:

```dart
final orderedDescendingList = list.sortedDescending();
```

And how would you sort collection items based on some property?

Plain Dart:

```dart
final orderedPeople = [...people]
  ..sort((person1, person2) => person1.birthDate.compareTo(person2.birthDate));
```

With [dartx](https://pub.dev/packages/dartx):

```dart
final orderedPeople = people.sortedBy((person) => person.birthDate);
```

and:

```dart
final orderedDescendingPeople = people.sortedByDescending((person) => person.birthDate);
```

To go even further, you can sort collection items by multiple properties:

```dart
final orderedPeopleByAgeAndName = people
  .sortedBy((person) => person.birthDate)
  .thenBy((person) => person.name);
```

and:

```dart
final orderedDescendingPeopleByAgeAndName = people
  .sortedByDescending((person) => person.birthDate)
  .thenByDescending((person) => person.name);
```

![](images/break_image.png)

## Collecting unique items

To get distinct collection items one might use this plain Dart implementation:

```dart
final unique = list.toSet().toList();
```

which does not guarantee to preserve items order or come up with a multi-line solution.

With [dartx](https://pub.dev/packages/dartx), it’s as easy as:

```dart
final unique = list.distinct().toList();
```

and:

```dart
final uniqueFirstNames = people.distinctBy((person) => person.firstName).toList();
```

![](images/break_image.png)

## Min / max / average by item property

To find a min / max collection item, we could for example sort it and take `first` / `last` item:

```dart
final min = ([...list]..sort()).first;

final max = ([...list]..sort()).last;
```

The same approach can be applied to sort by items' property:

```dart
final minAge = (people.map((person) => person.age).toList()..sort()).first;

final maxAge = (people.map((person) => person.age).toList()..sort()).last;
```

or:

```dart
final youngestPerson = 
  ([...people]..sort((person1, person2) => person1.age.compareTo(person2.age))).first;

final oldestPerson = 
  ([...people]..sort((person1, person2) => person1.age.compareTo(person2.age))).last;
```

But with [dartx](https://pub.dev/packages/dartx) it’s much easier:

```dart
final youngestPerson = people.minBy((person) => person.age);

final oldestPerson = people.maxBy((person) => person.age);
```

which by the way will return `null` for empty collections.

And if collection items implement `Comparable`, methods without selectors can be applied:

```dart
final min = list.min();

final max = list.max();
```

You can also easily obtain the average:

```dart
final average = list.average();
```

and:

```dart
final averageAge = people.averageBy((person) => person.age);
```

and the sum of `num` collections:

```dart
final sum = list.sum();
```

or `num` items' properties:

```dart
final totalChildrenCount = people.sumBy((person) => person.childrenCount);
```

![](images/break_image.png)

## Filtering out null items

With plain Dart:

```dart
final nonNullItems = list.where((x) => x != null).toList();
```

With [dartx](https://pub.dev/packages/dartx):

```dart
final nonNullItems = list.whereNotNull().toList();
```

![](images/break_image.png)

## More useful extensions

There are other useful extensions in [dartx](https://pub.dev/packages/dartx). We won’t go into deeper details here, but I hope the naming and the code are self-explanatory.

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

## Final words

The [dartx](https://pub.dev/packages/dartx) package contains many more extensions for [`Iterable`](https://github.com/leisim/dartx/blob/master/lib/src/iterable.dart), [`List`](https://github.com/leisim/dartx/blob/master/lib/src/list.dart), and [other Dart types](https://github.com/leisim/dartx/tree/master/lib/src). The best way to explore its capabilities is by browsing the [source code](https://github.com/leisim/dartx).

If you, as me, find the package useful, remember to give it 👍🏻 on [pub.dev](https://pub.dev/packages/dartx) and ⭐️ on [GitHub](https://github.com/leisim/dartx).

Thanks to package authors [Simon Leier](https://www.linkedin.com/in/simon-leier/) and [Pascal Welsch](https://twitter.com/passsy).

![](images/break_image.png)

*[Originally published](https://medium.com/flutter-community/dart-collections-with-dartx-extensions-959a0b42849e) on May 2021 under "Flutter community" Medium publication.*
