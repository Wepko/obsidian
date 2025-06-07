Чтобы собрать Flutter-проект в APK-файл (для Android), тебе нужно выполнить несколько шагов. Ниже пошаговая инструкция:

---

### 🛠 Шаг 1: Подготовка проекта

1. **Убедись, что Flutter установлен**  
   Проверь с помощью команды:
   ```bash
   flutter doctor
   ```
   Убедись, что нет ошибок, особенно связанных с Android SDK.

2. **Подключи устройство или настрой эмулятор** (не обязательно для билда, но полезно для тестирования):
   - Android Studio → AVD Manager
   - Либо подключи реальное устройство через USB и включи режим разработчика + USB-отладку.

---

### 📦 Шаг 2: Сборка APK

Открой терминал в папке твоего Flutter-проекта и выполни следующие команды:

#### Для **debug сборки** (для тестирования на устройстве):
```bash
flutter build apk --debug
```

#### Для **release сборки** (готовый к выпуску APK):
```bash
flutter build apk --release
```

> Release-сборка будет находиться в:
```
build/app/outputs/flutter-apk/app-release.apk
```

---

### 🔐 Опционально: Подписываем APK (для Google Play)

Если хочешь загрузить приложение в Google Play, тебе нужно подписать APK. Для этого нужно создать **keystore**.

#### 1. Создай keystore:
```bash
keytool -genkey -v -keystore ~/key.jks -keyalg RSA -keysize 2048 -storetype JKS -validity 10000
```

#### 2. Настрой файл `android/app/build.gradle`:
Добавь секцию signingConfigs и укажи путь к ключу и пароли.

#### 3. Пересобери проект:
```bash
flutter build apk --release
```

---

### 🧪 Как установить APK на телефон:

1. Скопируй файл `app-release.apk` на телефон.
2. Открой его через файловый менеджер.
3. Разреши установку из неизвестных источников, если система предупреждает.

---

### 🎥 Полезные видео на YouTube:

Вот несколько обучающих видео, которые помогут тебе наглядно собрать APK:

- [Flutter Build APK Tutorial | How to Build APK in Flutter](https://www.youtube.com/watch?v=VzFPSFMIQaI)
- [Flutter Release APK & App Bundle with Keystore Signing](https://www.youtube.com/watch?v=YOj5oHwUAxU)
- [How to Create a Signed APK for Android in Flutter](https://www.youtube.com/watch?v=FPGyOheQlFA)

---

Если ты хочешь собрать **App Bundle (.aab)** вместо APK (например, для Google Play), используй команду:
```bash
flutter build appbundle
```

Путь к файлу:
```
build/app/outputs/bundle/release/app-release.aab
```

---

Если будут вопросы — пиши! Могу помочь с подписью или настройкой Gradle.