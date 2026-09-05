# ROS 2 Humble + CUDA Cross-Platform Build

CI/CD пайплайн автоматической сборки кросс-платформенных Docker-контейнеров для алгоритма одометрии FAST-LIO и драйвера Livox-SDK2 / livox_ros_driver2 под ROS 2 Humble с поддержкой NVIDIA CUDA.

Разработано в рамках технического задания компании **«Априорные Решения Машин»** ([armmeh.ru](https://armmeh.ru/)).

---

## 1. Поддерживаемые архитектуры

| Платформа | Архитектура | Базовый образ | Compute Capability | Назначение |
| :--- | :--- | :--- | :--- | :--- |
| **x86_64** | `linux/amd64` | Ubuntu 22.04 + CUDA 12.2 | `86` (sm_86) | ПК / Рабочие станции |
| **Jetson AGX Orin** | `linux/arm64` | Dusty-nv L4T Jetpack 6.x | `87` (sm_87) | Бортовой вычислитель |
| **Jetson Orin Nano** | `linux/arm64` | Dusty-nv L4T Jetpack 6.x | `87` (sm_87) | Компактный бортовой вычислитель |

---

## 2. Архитектура решения

* **GitHub Actions Matrix:** Параллельная кросс-сборка под целевые платформы через Docker Buildx и эмуляцию QEMU.
* **Триггеры сборки:** Автоматический запуск при `push` в ветку `main`, создание версионных тегов (`v*.*.*`), а также ручной запуск через `workflow_dispatch`.
* **Двухэтапная модульная сборка:**
  * `docker/Dockerfile.base.*` — системные зависимости ROS 2 Humble, CUDA Toolkit, PCL, Eigen и компиляция драйвера Livox-SDK2.
  * `docker/Dockerfile.package` — сборка интерфейсов `ros2_interface_livox`, драйвера `livox_ros_driver2` и алгоритма `FAST_LIO_ROS2` с деревом `ikd-Tree`.
* **Оптимизация сборки:** 
  * Ограничение параллелизма (`--parallel-workers 1`, `MAKEFLAGS="-j2"`) для предотвращения Out-Of-Memory под QEMU ARM64.
  * Флаг `-DBUILD_TESTING=OFF` для исключения неиспользуемых зависимостей линтеров (`ament_lint_auto`).
  * Безопасное перенаправление вызовов `/sbin/ldconfig` через `dpkg-divert` для предотвращения сбоев эмулятора QEMU.

---

## 3. Быстрый запуск

### Запуск на x86_64 с GPU NVIDIA

```bash
docker pull ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest

docker run --rm -it --gpus all \
  ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest
