В Dart/Flutter нет классических namespace как в других языках, но есть несколько эффективных подходов для организации кода. Вот лучшие практики:

## 1. **Пакеты и библиотеки (Packages)**

Создавайте локальные пакеты для разных модулей:

```yaml
# pubspec.yaml
dependencies:
  core_module:
    path: packages/core_module
  auth_module:
    path: packages/auth_module
  user_module:
    path: packages/auth_module
```

## 2. **Экспорт через barrel-файлы**

Создавайте файлы `export.dart` в каждой папке:

```dart
// lib/src/core/export.dart
export 'constants/app_constants.dart';
export 'theme/app_theme.dart';
export 'utils/validators.dart';
export 'widgets/custom_button.dart';
```

```dart
// lib/src/features/auth/export.dart
export 'data/auth_repository.dart';
export 'domain/auth_service.dart';
export 'presentation/login_screen.dart';
export 'presentation/register_screen.dart';
```

## 3. **Использование part/part of**

Для тесно связанных файлов:

```dart
// user.dart
part 'user.g.dart';
part 'user_mapper.dart';

class User {
  final String name;
  final int age;
  
  User(this.name, this.age);
}
```

```dart
// user_mapper.dart
part of 'user.dart';

extension UserMapper on User {
  Map<String, dynamic> toMap() {
    return {'name': name, 'age': age};
  }
}
```

## 4. **Псевдонимы импорта (Import Aliases)**

```dart
import 'package:myapp/src/core/constants.dart' as constants;
import 'package:myapp/src/features/auth/auth_service.dart' as auth;

void main() {
  print(constants.appName);
  auth.AuthService().login();
}
```

## 5. **Структура проекта**

Рекомендуемая структура:

```
lib/
├── src/
│   ├── core/
│   │   ├── constants/
│   │   ├── theme/
│   │   ├── utils/
│   │   └── widgets/
│   ├── features/
│   │   ├── auth/
│   │   ├── profile/
│   │   └── dashboard/
│   ├── data/
│   │   ├── repositories/
│   │   └── models/
│   └── domain/
│       ├── entities/
│       └── usecases/
├── main.dart
└── export.dart
```

## 6. **Главный файл экспорта**

```dart
// lib/export.dart
export 'src/core/export.dart' show AppTheme, Validators;
export 'src/features/auth/export.dart' show LoginScreen, AuthService;
export 'src/features/profile/export.dart' show ProfileScreen;
```

## 7. **Пример использования**

```dart
// Вместо этого:
import '../../../../core/constants/app_constants.dart';
import '../../../features/auth/data/auth_repository.dart';

// Делайте так:
import 'package:myapp/export.dart' as app;

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: app.AppConstants.appName,
      theme: app.AppTheme.light,
      home: app.LoginScreen(),
    );
  }
}
```

## 8. **Плагины и инструменты**

- **build_runner** - для кодогенерации
- **freezed** - для создания моделей с namespace
- **injectable** - для dependency injection
- **flutter_gen** - для генерации путей к ресурсам

## 9. **Абсолютные импорты**

Добавьте в `analysis_options.yaml`:

```yaml
analyzer:
  language:
    strict-casts: true
    strict-inference: true
  errors:
    avoid_relative_lib_imports: error
```

## 10. **Пример полной структуры**

```dart
// lib/main.dart
import 'package:myapp/export.dart';

void main() => runApp(MyApp());

// lib/src/core/constants/app_constants.dart
class AppConstants {
  static const String appName = 'MyApp';
}

// lib/src/features/auth/presentation/login_screen.dart
class LoginScreen extends StatelessWidget {
  // ...
}
```

Этот подход дает вам преимущества namespace без их прямого наличия в языке, делая код чище и более организованным.****