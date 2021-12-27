# Yet another localization approach in Flutter

*Online talk at [the Flutter Global Summit 21 Vol.2](https://geekle.us/schedule/flutter2) on December 9, 2021.*

*Watch on [YouTube](https://youtu.be/DTGp84cEnr4).*

![](images/cover_image.png)

When it comes to localizing Flutter applications, there is already quite a number of approaches. Some offer strongly typed message keys, while others leave you guessing whether a message exists and with no refactoring support. Some use custom commands for generating code and don't offer a file system watching, while others imply only manual work. They differ by supported file formats, pluralization support, a mechanism for locales switching, etc.

In this session, I will share what I am looking for when it comes to localizing Flutter applications (spoiler alert: code generation with build_runner is one of the key requirements), explain the solution that combines pros and avoids cons of existing approaches, and implement it in a multi-language Flutter app live.

*The source code and slides are available in [flutter_global_summit_2021 GitHub repository](https://github.com/foxanna/flutter_global_summit_2021).*