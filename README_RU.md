# Сборка TWRP для Poco X3 Pro (vayu) на Ubuntu

Это руководство предоставляет пошаговые инструкции по сборке TWRP (Team Win Recovery Project) для Poco X3 Pro (кодовое имя: vayu) на системе Ubuntu. TWRP — это кастомное рекавери, которое позволяет устанавливать пользовательские прошивки, создавать резервные копии и выполнять расширенные операции на устройствах Android.

## Требования

- **Операционная система**: 64-битная версия Ubuntu (рекомендуется 20.04 или новее).
- **Оборудование**: Минимум 8 ГБ оперативной памяти, 100 ГБ свободного места на диске и стабильное интернет-соединение.
- **Устройство**: Poco X3 Pro с **разблокированным загрузчиком**. Разблокировка стирает данные, поэтому сделайте резервную копию.
- **Знания**: Знакомство с командами терминала Linux и процессом сборки Android.

## Шаг 1: Установка необходимых пакетов

Обновите систему и установите необходимые инструменты для сборки TWRP.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl bison build-essential ccache flex libesd0-dev libssl-dev libxml2-utils openjdk-8-jdk python3 python3-distutils python3-dev python3-pip rsync schedtool squashfs-tools uuid-dev zlib1g-dev
```

Установите инструмент `repo` для управления исходным кодом:

```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
```

## Шаг 2: Клонирование исходного кода TWRP

Создайте рабочую директорию и клонируйте минимальный манифест TWRP для Android 11 (ветка twrp-11.0).

```bash
mkdir ~/twrp
cd ~/twrp
repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-11.0
repo sync -c -j8 --no-clone-bundle --no-scheduler
```

**Примечание**: Команда `repo sync` может занять время в зависимости от скорости вашего интернета. Флаг `--no-clone-bundle` помогает уменьшить проблемы с загрузкой.

## Шаг 3: Добавление дерева устройства для Poco X3 Pro (vayu)

Клонируйте дерево устройства для vayu из репозитория TeamWin.

```bash
mkdir -p device/xiaomi
git clone https://github.com/TeamWin/android_device_xiaomi_vayu.git device/xiaomi/vayu
```

Убедитесь, что директория `device/xiaomi/vayu` содержит файлы, такие как `Android.mk`, `BoardConfig.mk` и `device.mk`.

## Шаг 4: Настройка среды сборки

Инициализируйте среду сборки и выберите конфигурацию для vayu.

```bash
source build/envsetup.sh
lunch omni_vayu-eng
```

Команда `lunch` настраивает сборку для Poco X3 Pro в инженерном режиме (`-eng`).

## Шаг 5: Сборка TWRP рекавери

Скомпилируйте образ рекавери с помощью следующей команды:

```bash
mka recoveryimage
```

Этот процесс может занять от 30 минут до нескольких часов в зависимости от производительности вашей системы.

## Шаг 6: Поиск собранного образа рекавери

После успешного завершения сборки образ рекавери будет находиться по пути:

```
out/target/product/vayu/recovery.img
```

Проверьте его наличие:

```bash
ls out/target/product/vayu/recovery.img
```

## Шаг 7: Установка TWRP на устройство

1. **Убедитесь, что загрузчик разблокирован**: Если нет, включите параметры разработчика, отладку по USB и разблокировку OEM в настройках устройства, затем выполните:

   ```bash
   adb reboot bootloader
   fastboot flashing unlock
   ```

   **Предупреждение**: Разблокировка загрузчика удаляет все данные на устройстве.

2. **Прошейте образ рекавери**:

   ```bash
   cd ~/twrp/out/target/product/vayu
   fastboot flash recovery recovery.img
   fastboot reboot recovery
   ```

Устройство должно загрузиться в TWRP рекавери. Проверьте основные функции, такие как создание резервной копии и управление файлами.

## Устранение неполадок

- **Ошибки сборки**: Проверьте логи терминала на наличие отсутствующих зависимостей или некорректных файлов дерева устройства. Убедитесь, что все пакеты установлены, а дерево устройства правильно клонировано.
- **TWRP не загружается**: Убедитесь, что образ рекавери прошит корректно и загрузчик разблокирован.
- **Медленная синхронизация**: Если `repo sync` работает медленно, уменьшите значение `-j` (например, `-j4`) или проверьте интернет-соединение.

## Дополнительные замечания

- **Резервное копирование**: Всегда создавайте резервную копию данных устройства перед прошивкой.
- **Обновления**: Чтобы использовать более новую версию TWRP, проверьте доступные ветки в [репозитории манифеста TWRP](https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni).
- **Поддержка**: При возникновении проблем обратитесь к [дереву устройства vayu от TeamWin](https://github.com/TeamWin/android_device_xiaomi_vayu) или форумам XDA для поддержки сообщества.

## Ссылки

- [Минимальный манифест TWRP](https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni)
- [Дерево устройства Poco X3 Pro](https://github.com/TeamWin/android_device_xiaomi_vayu)
- [Руководство по сборке TWRP](https://github.com/twrpdtgen/twrpdtgen/wiki/4.-Build-TWRP-from-source)
