# VMG Support Team — кастомизация RustDesk

Актуально на: 23.07.2026  
Рабочая ветка: `vmgroup-brand`  
Проверенная функциональная точка: `fc6c0d2b0`

## Назначение

Форк RustDesk используется как клиент удалённой поддержки VMG:

- собственные серверы `rustdesk.groupvm.ru`;
- имя подключения `VMG Support Team`;
- собственные иконки Windows и Android;
- логотип VMG в карточке входящего подключения;
- сборка Windows x64 и Android ARM64;
- выдача сборок через GitHub Actions Artifacts.

## Ветка

Все кастомные изменения находятся в ветке:

```text
vmgroup-brand
```

`origin` — собственный форк.  
`upstream` — официальный репозиторий RustDesk.

---

## 1. Серверы по умолчанию

Файл:

```text
src/common.rs
```

В начале `load_custom_client()` вызывается:

```rust
apply_vmgroup_default_servers();
```

Функция задаёт:

```text
custom-rendezvous-server = rustdesk.groupvm.ru
relay-server             = rustdesk.groupvm.ru
api-server               = https://rustdesk.groupvm.ru
key                      = публичный ключ сервера
```

Используется `or_insert_with`, поэтому уже сохранённые пользовательские параметры не перезаписываются.

Проверка:

```bash
git grep -n -C8 -E \
  'apply_vmgroup_default_servers|rustdesk\.groupvm\.ru' \
  -- src/common.rs
```

---

## 2. Имя исходящего подключения

Файл:

```text
src/client.rs
```

В `LoginRequest` установлено:

```rust
my_name: "VMG Support Team".to_owned(),
```

Именно это имя отображается на принимающем компьютере.

Системный логин ОС должен оставаться:

```rust
os_login: Some(OSLogin {
    username: os_username,
    password: os_password,
    ..Default::default()
})
```

Нельзя заменять `os_username` на `VMG Support Team`, иначе могут перестать работать:

- авторизация ОС;
- elevation;
- терминальные подключения;
- вход с системными учётными данными.

Проверка:

```bash
git grep -n -C5 -E \
  'my_name:|os_login:|username: os_username' \
  -- src/client.rs
```

---

## 3. PeerInfo

Файл:

```text
src/server/connection.rs
```

Изменены только отображаемые поля:

```rust
username: "VMG Support Team".to_owned();
pi.hostname = "VMG Support Team".to_owned();
```

Правка применяется к desktop и Android.

Нельзя глобально заменять:

```rust
crate::username()
common::username()
```

Они используются во внутренней логике RustDesk.

Проверка:

```bash
git grep -n -C5 "VMG Support Team" \
  -- src/server/connection.rs
```

---

## 4. Логотип входящего подключения

Локальный Flutter asset:

```text
flutter/assets/vmg_logo.png
```

Каталог подключён в:

```text
flutter/pubspec.yaml
```

через:

```yaml
assets:
  - assets/
```

В корневом `.gitignore` присутствует правило:

```text
*png
```

Поэтому новый PNG добавляется принудительно:

```bash
git add -f flutter/assets/vmg_logo.png
```

### Desktop

Файл:

```text
flutter/lib/desktop/pages/server_page.dart
```

Для подключения с именем VMG принудительно используется локальный логотип:

```dart
if (client.name == 'VMG Support Team') {
  return _buildInitialAvatar();
}
```

В `_buildInitialAvatar()` используется:

```dart
Image.asset(
  'assets/vmg_logo.png',
  width: 70,
  height: 70,
  fit: BoxFit.cover,
)
```

### Android/mobile

Файл:

```text
flutter/lib/mobile/pages/server_page.dart
```

Используется тот же asset размером 40×40.

### Особенность тестирования

Окно с кнопками «Принять» и «Отклонить» отрисовывает принимающий компьютер.

Поэтому для проверки нового логотипа свежая сборка должна быть установлена именно на принимающей стороне.

Обновление только исходящего ноутбука не меняет интерфейс старой версии на приёмнике.

Проверка:

```bash
git grep -n -C5 \
  "client.name == 'VMG Support Team'" \
  -- flutter/lib/desktop/pages/server_page.dart \
     flutter/lib/mobile/pages/server_page.dart
```

---

## 5. Windows-брендинг

Изменены:

```text
Cargo.toml
res/icon.ico
res/icon.png
res/tray-icon.ico
flutter/windows/runner/Runner.rc
flutter/windows/runner/main.cpp
flutter/windows/runner/resources/app_icon.ico
```

Результат:

- собственная иконка EXE;
- собственная иконка окна;
- собственная иконка трея;
- Windows metadata VMG;
- название VMG Support Team.

Windows Explorer может некоторое время показывать старую иконку из-за кэша. Это не означает, что ресурс не попал в сборку.

---

## 6. Android-брендинг

Основные файлы:

```text
flutter/android/app/src/main/AndroidManifest.xml
flutter/android/app/src/main/res/values/strings.xml
```

Название приложения:

```text
VM Group Support Team
```

Accessibility Service:

```text
VM Group Input
```

Launcher icons заменены в каталогах:

```text
mipmap-mdpi
mipmap-hdpi
mipmap-xhdpi
mipmap-xxhdpi
mipmap-xxxhdpi
```

В каждом каталоге изменены:

```text
ic_launcher.png
ic_launcher_round.png
ic_launcher_foreground.png
```

Некоторые внутренние строки RustDesk пока остаются в Kotlin-коде:

```text
RustDesk is Open
RustDesk Service
RustDeskVD
Show RustDesk
```

Они не влияют на основное название установленного приложения.

---

## 7. GitHub Actions

Изменены:

```text
.github/workflows/flutter-build.yml
.github/workflows/flutter-ci.yml
```

Запускаемый workflow:

```text
Full Flutter CI
```

Ветка:

```text
vmgroup-brand
```

Собираются:

- Windows x64;
- Android ARM64.

Отключены:

- Windows ARM64;
- Windows Sciter;
- Android ARMv7;
- Android x86_64;
- Android universal;
- Linux;
- Linux Sciter;
- AppImage;
- Flatpak;
- macOS;
- iOS;
- автоматическая публикация GitHub Release.

Артефакты:

```text
vmgroup-windows-x86_64
vmgroup-android-arm64
```

Публикация через `softprops/action-gh-release` отключена, чтобы workflow не завершался ошибкой 403.

---

## 8. Ручная сборка

1. Проверить состояние:

```bash
git checkout vmgroup-brand
git status --short
git push
```

2. В GitHub открыть:

```text
Actions → Full Flutter CI
```

3. Нажать:

```text
Run workflow
```

4. Выбрать:

```text
vmgroup-brand
```

5. Скачать артефакты:

```text
vmgroup-windows-x86_64
vmgroup-android-arm64
```

6. Установить новую Windows-сборку поверх старой.

7. Для проверки логотипа обновить принимающий компьютер.

---

## 9. Обновление после выхода новой версии RustDesk

Переключиться на рабочую ветку:

```bash
git checkout vmgroup-brand
git status --short
```

Рабочее дерево должно быть чистым.

Создать резервный тег:

```bash
git tag -a "backup-before-upstream-$(date +%Y%m%d)" \
  -m "Backup before RustDesk upstream update"

git push origin --tags
```

Получить изменения официального репозитория:

```bash
git fetch upstream
```

Посмотреть свежие коммиты:

```bash
git log --oneline --decorate -10 upstream/master
```

Выполнить rebase:

```bash
git rebase upstream/master
```

При конфликте:

```bash
git status
```

Исправить файл, удалить маркеры:

```text
<<<<<<<
=======
>>>>>>>
```

Затем:

```bash
git add <исправленный-файл>
git rebase --continue
```

Отмена rebase:

```bash
git rebase --abort
```

После успешного обновления:

```bash
git push --force-with-lease origin vmgroup-brand
```

Не использовать обычный `--force`, если нет отдельной причины.

---

## 10. Проверка после rebase

Серверы:

```bash
git grep -n -C8 -E \
  'apply_vmgroup_default_servers|rustdesk\.groupvm\.ru' \
  -- src/common.rs
```

Имя подключения и системный логин:

```bash
git grep -n -C5 -E \
  'my_name: "VMG Support Team"|username: os_username' \
  -- src/client.rs
```

PeerInfo:

```bash
git grep -n -C5 "VMG Support Team" \
  -- src/server/connection.rs
```

Логотип:

```bash
git grep -n -C5 \
  "client.name == 'VMG Support Team'" \
  -- flutter/lib/desktop/pages/server_page.dart \
     flutter/lib/mobile/pages/server_page.dart
```

Ресурсы:

```bash
git ls-files | grep -E \
  'vmg_logo|icon\.ico|tray-icon|ic_launcher' | sort
```

После этого запустить:

```text
Actions → Full Flutter CI → vmgroup-brand
```

---

## 11. Контрольный список

```text
[ ] git status чистый
[ ] активна ветка vmgroup-brand
[ ] встроен rustdesk.groupvm.ru
[ ] встроен публичный ключ сервера
[ ] my_name = VMG Support Team
[ ] os_login.username = os_username
[ ] PeerInfo username и hostname брендированы
[ ] Windows-иконки присутствуют
[ ] Android-иконки присутствуют
[ ] flutter/assets/vmg_logo.png отслеживается Git
[ ] desktop avatar override присутствует
[ ] mobile avatar override присутствует
[ ] Full Flutter CI завершён успешно
[ ] Windows-сборка проверена
[ ] логотип проверен на принимающем ПК
[ ] Android ARM64 APK проверен
```
