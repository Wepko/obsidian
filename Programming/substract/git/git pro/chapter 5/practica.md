Отлично! Вот пошаговые решения практических заданий с пояснениями.

---

## **Задание 1 (Модели workflow): Диспетчер интеграции через GitHub/GitLab**

### **Шаг 1: Создание upstream репозитория**
1. Зайдите на GitHub.com или GitLab.com
2. Нажмите "New repository"
3. Назовите его, например, `git-workflow-demo`
4. **Оставьте его пустым** - не добавляйте README, .gitignore или лицензию
5. Нажмите "Create repository"
6. Запомните URL вашего репозитория: `https://github.com/ВАШ_АККАУНТ/git-workflow-demo.git`

### **Шаг 2: Форк и клонирование**
1. На странице созданного репозитория нажмите кнопку **"Fork"** (в правом верхнем углу)
2. После создания форка, склонируйте его локально:
```bash
# Склонируйте ваш форк (origin)
git clone https://github.com/ВАШ_АККАУНТ/git-workflow-demo.git
cd git-workflow-demo

# Добавьте оригинальный репозиторий как upstream
git remote add upstream https://github.com/ОРИГИНАЛЬНЫЙ_АККАУНТ/git-workflow-demo.git

# Проверьте удалённые репозитории
git remote -v
# Должно показать:
# origin    https://github.com/ВАШ_АККАУНТ/git-workflow-demo.git (fetch)
# origin    https://github.com/ВАШ_АККАУНТ/git-workflow-demo.git (push)
# upstream  https://github.com/ОРИГИНАЛЬНЫЙ_АККАУНТ/git-workflow-demo.git (fetch)
# upstream  https://github.com/ОРИГИНАЛЬНЫЙ_АККАУНТ/git-workflow-demo.git (push)
```

### **Шаг 3: Создание тематической ветки и внесение изменений**
```bash
# Создайте и переключитесь на новую ветку
git checkout -b add-contributors-file

# Создайте файл CONTRIBUTORS.md
echo "# Contributors" > CONTRIBUTORS.md
echo "" >> CONTRIBUTORS.md
echo "- Ваше Имя <your.email@example.com>" >> CONTRIBUTORS.md

# Посмотрите изменения
git status
git diff

# Добавьте файл в staging и создайте коммит
git add CONTRIBUTORS.md
git commit -m "docs: add CONTRIBUTORS.md file with initial contributor"
```

### **Шаг 4: Push в origin и создание Pull Request**
```bash
# Отправьте ветку в ваш форк (origin)
git push origin add-contributors-file

# Теперь в браузере зайдите на страницу вашего форка
# GitHub обычно сам предложит создать Pull Request после push
# Или нажмите "Compare & pull request" на странице репозитория
```

### **Шаг 5: Слияние PR (как диспетчер интеграции)**
1. На странице Pull Request нажмите **"Merge pull request"**
2. Выберите опцию слияния:
   - **Create a merge commit** (рекомендуется для практики)
   - **Squash and merge** (объединяет все коммиты в один)
   - **Rebase and merge** (делает историю линейной)
3. Нажмите "Confirm merge"
4. После слияния вы можете удалить ветку через интерфейс GitHub

### **Шаг 6: Обновление локального репозитория**
```bash
# Переключитесь в main и обновите с upstream
git checkout main
git pull upstream main

# Убедитесь, что изменения появились
ls -la CONTRIBUTORS.md
cat CONTRIBUTORS.md

# Теперь обновите ваш origin
git push origin main
```

---

## **Задание 2 (Работа с патчами)**

### **Шаг 1: Подготовка репозитория**
```bash
# Создайте новый каталог для задания
mkdir git-patch-demo && cd git-patch-demo
git init

# Создайте начальный файл и первый коммит
echo "Initial content" > demo.txt
git add demo.txt
git commit -m "Initial commit"
```

### **Шаг 2: Создание веток feature-a и feature-b**
```bash
# Создаём ветку feature-a
git checkout -b feature-a

# Первый коммит в feature-a
echo "Feature A, change 1" >> demo.txt
git add demo.txt
git commit -m "feat(a): add first change for feature A"

# Второй коммит в feature-a
echo "Feature A, change 2" >> demo.txt
git add demo.txt
git commit -m "feat(a): add second change for feature A"

# Возвращаемся к main и создаём feature-b
git checkout main
git checkout -b feature-b

# Первый коммит в feature-b
echo "Feature B, change 1" >> demo.txt
git add demo.txt
git commit -m "feat(b): add first change for feature B"

# Второй коммит в feature-b
echo "Feature B, change 2" >> demo.txt
git add demo.txt
git commit -m "feat(b): add second change for feature B"
```

### **Шаг 3: Генерация патчей из feature-a**
```bash
# Переключаемся обратно в feature-a
git checkout feature-a

# Смотрим историю коммитов
git log --oneline

# Генерируем патчи для всех коммитов
git format-patch main

# Должны создаться файлы вида:
# 0001-feat-a-add-first-change-for-feature-A.patch
# 0002-feat-a-add-second-change-for-feature-A.patch

# Посмотрим содержимое одного патча
cat 0001-feat-a-add-first-change-for-feature-A.patch
```

### **Шаг 4: Применение патчей через git am**
```bash
# Переключаемся в main
git checkout main

# Применяем все патчи из feature-a
git am *.patch

# Проверяем результат
git log --oneline
cat demo.txt

# Должно быть:
# Initial content
# Feature A, change 1
# Feature A, change 2
```

### **Шаг 5: Cherry-pick из feature-b**
```bash
# Смотрим коммиты в feature-b
git checkout feature-b
git log --oneline

# Копируем хэш самого важного коммита (например, первого)
# Например: abc1234 feat(b): add first change for feature B

# Возвращаемся в main
git checkout main

# Делаем cherry-pick выбранного коммита
git cherry-pick <хэш_коммита>

# Если возник конфликт (т.к. файл уже изменён feature-a), разрешаем его:
# 1. Редактируем demo.txt, оставляя нужные изменения
# 2. git add demo.txt
# 3. git cherry-pick --continue

# Проверяем результат
git log --oneline
cat demo.txt
```

---

## **Задание 3 (Rerere в действии)**

### **Шаг 1: Включение rerere**
```bash
# Включаем rerere глобально
git config --global rerere.enabled true

# Проверяем, что включено
git config --get rerere.enabled
# Должно вернуть: true
```

### **Шаг 2: Подготовка репозитория и создание конфликта**
```bash
# Создаём новый каталог
mkdir git-rerere-demo && cd git-rerere-demo
git init

# Создаём файл и первый коммит в main
echo "Line 1: Original content" > testfile.txt
echo "Line 2: Some text" >> testfile.txt
git add testfile.txt
git commit -m "Initial commit with testfile"
```

### **Шаг 3: Создание ветки long-lived с изменением**
```bash
# Создаём ветку long-lived от main
git checkout -b long-lived

# Изменяем первую строку в ветке long-lived
echo "Line 1: Modified in long-lived branch" > testfile.txt
echo "Line 2: Some text" >> testfile.txt
git add testfile.txt
git commit -m "Change line 1 in long-lived branch"

# Проверяем
cat testfile.txt
```

### **Шаг 4: Изменение той же строки в main**
```bash
# Возвращаемся в main
git checkout main

# Изменяем ту же первую строку, но по-другому
echo "Line 1: Modified in main branch" > testfile.txt
echo "Line 2: Some text" >> testfile.txt
git add testfile.txt
git commit -m "Change line 1 in main"

# Проверяем
cat testfile.txt
```

### **Шаг 5: Первое слияние с ручным разрешением конфликта**
```bash
# Переключаемся в long-lived
git checkout long-lived

# Пробуем слить main в long-lived (конфликт гарантирован!)
git merge main

# Должен возникнуть конфликт:
# Auto-merging testfile.txt
# CONFLICT (content): Merge conflict in testfile.txt
# Automatic merge failed; fix conflicts and then commit the result.

# Проверяем статус
git status

# Смотрим файл с конфликтом
cat testfile.txt
# Увидим что-то вроде:
# <<<<<<< HEAD
# Line 1: Modified in long-lived branch
# =======
# Line 1: Modified in main branch
# >>>>>>> main
# Line 2: Some text

# РУЧНОЕ разрешение конфликта - выбираем вариант из long-lived
echo "Line 1: Modified in long-lived branch" > testfile.txt
echo "Line 2: Some text" >> testfile.txt

# Добавляем разрешённый файл и завершаем слияние
git add testfile.txt
git commit -m "Merge main into long-lived (resolved conflict)"
```

### **Шаг 6: Откат слияния**
```bash
# Откатываем последний коммит (слияние)
git reset --hard HEAD^

# Проверяем, что мы откатились
git log --oneline
# Должен быть только: [хэш] Change line 1 in long-lived branch
cat testfile.txt
# Должно быть: Line 1: Modified in long-lived branch
```

### **Шаг 7: Повторное слияние с автоматическим разрешением**
```bash
# Пробуем снова слить main в long-lived
git merge main

# MAGIC! Конфликт должен разрешиться АВТОМАТИЧЕСКИ
# Вы увидите сообщение:
# Auto-merging testfile.txt
# CONFLICT (content): Merge conflict in testfile.txt
# Resolved 'testfile.txt' using previous resolution.
# Automatic merge failed; fix conflicts and then commit the result.

# Проверяем содержимое файла
cat testfile.txt
# Должно быть АВТОМАТИЧЕСКИ разрешённое состояние:
# Line 1: Modified in long-lived branch
# Line 2: Some text

# Просто завершаем слияние
git add testfile.txt
git commit -m "Merge main into long-lived (auto-resolved by rerere)"
```

### **Шаг 8: Проверка сохранённых разрешений**
```bash
# Посмотреть сохранённые разрешения конфликтов
# (Для Linux/Mac, в Windows может отличаться)
cat .git/rr-cache/*/postimage

# Или посмотреть состояние кеша rerere
git rerere status
git rerere diff
```

---

## **Ключевые выводы из практики:**

1. **Задание 1:** Показало полный цикл open-source контрибуции через форки и PR - основную модель современной разработки.

2. **Задание 2:** Продемонстрировало "старую школу" обмена кодом через патчи, что до сих пор актуально в некоторых проектах. Вы увидели:
   - Как генерировать патчи из коммитов
   - Как применять их через `git am`
   - Как работает выборочное применение изменений через `cherry-pick`

3. **Задание 3:** Показало мощь `rerere` в действии. Это особенно полезно для:
   - Долгоживущих feature-веток
   - Частого rebase/merge
   - Ситуаций, когда конфликты повторяются

**Совет:** После выполнения заданий, попробуйте изменить разрешение конфликта в шаге 5 задания 3 (например, выбрать вариант из main) и посмотреть, как `rerere` запомнит новое разрешение.