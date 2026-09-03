# ROS 2 Humble + CUDA Cross-Platform Build

[![Cross-Platform ROS2 CUDA Build](https://github.com/mashushka11-ops/ros2-cuda-crossbuild/actions/workflows/build.yml/badge.svg)](https://github.com/mashushka11-ops/ros2-cuda-crossbuild/actions/workflows/build.yml)

CI/CD пайплайн сборки кросс-платформенных Docker-контейнеров для алгоритма одометрии **FAST-LIO** и драйвера **Livox-SDK2 / livox_ros_driver2** под **ROS 2 Humble** с поддержкой **NVIDIA CUDA**.

## Поддерживаемые архитектуры

| Платформа | Архитектура | Базовый образ | Compute Capability | Назначение |
| :--- | :--- | :--- | :--- | :--- |
| **x86_64** | `linux/amd64` | Ubuntu 22.04 + CUDA 12.2 | `sm_86` | ПК / Рабочие станции |
| **Jetson AGX Orin** | `linux/arm64` | Dusty-nv L4T Jetpack 6.0 | `sm_87` | Бортовой вычислитель |
| **Jetson Orin Nano**| `linux/arm64` | Dusty-nv L4T Jetpack 6.0 | `sm_87` | Компактный бортовой вычислитель |

---

## Архитектура решения

* **GitHub Actions Matrix:** Параллельная кросс-сборка под разные архитектуры через Docker Buildx и QEMU.
* **Двухэтапная сборка:**
  1. `Dockerfile.base.*` — системные CUDA/ROS2 зависимости, PCL, Eigen и компиляция драйвера `Livox-SDK2`.
  2. `Dockerfile.package` — компиляция пакетов `livox_ros_driver2` (через скрипт `build.sh humble`) и ROS2-порта `FAST_LIO_ROS2` со встроенным деревом поиска `ikd-Tree`.
* **Оптимизация сборки:** Ограничение параллелизма (`--parallel-workers 1`) для исключения Out-Of-Memory при эмуляции ARM64.

---

## Быстрый запуск

### 1. x86_64 с GPU NVIDIA

```bash
docker pull ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest

docker run --rm -it --gpus all \
  ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest

2. NVIDIA Jetson (ARM64)
docker pull ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-jetson-agx-orin:latest

docker run --rm -it --runtime nvidia \
  ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-jetson-agx-orin:latest

3. Запуск узла внутри контейнера

ros2 launch fast_lio mapping.launch.py
