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

* **GitHub Actions Matrix:** Параллельная кросс-сборка под целевые платформы через Docker Buildx и эмуляцию QEMU.
* **Триггеры сборки:** Автоматический запуск при push в ветку `main`, создание версионных тегов (`v*.*.*`), а также ручной запуск через `workflow_dispatch`.
* **Двухэтапная сборка:**
  1. `Dockerfile.base.*` — системные зависимости ROS 2 Humble, CUDA Toolkit, PCL, Eigen и компиляция драйвера `Livox-SDK2`.
  2. `Dockerfile.package` — компиляция пакетов `livox_ros_driver2` и ROS2-порта `FAST_LIO_ROS2` с поисковым деревом `ikd-Tree`.
* **Оптимизация компиляции:** Ограничение параллелизма сборщика (`--parallel-workers 1`) для предотвращения Out-Of-Memory при сборке под QEMU ARM64.

---

## Быстрый запуск

### 1. x86_64 с GPU NVIDIA

```bash
docker pull ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest

docker run --rm -it --gpus all \
  ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest
```

### 2. NVIDIA Jetson (ARM64)

```bash
docker pull ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-jetson-agx-orin:latest

docker run --rm -it --runtime nvidia \
  ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-jetson-agx-orin:latest
```

### 3. Запуск узла внутри контейнера

```bash
ros2 launch fast_lio mapping.launch.py
```

---

## Подтверждение наличия CUDA

Проверка компилятора CUDA (`nvcc`) и доступности библиотек в собранных образах:

```bash
docker run --rm ghcr.io/mashushka11-ops/ros2-cuda-crossbuild/fastlio-x86:latest nvcc --version
```

---

## Добавление нового ROS 2 пакета

Система спроектирована по модульной двухэтапной схеме, что позволяет подключать новые ROS 2 пакеты без необходимости повторной компиляции базового CUDA/ROS2 окружения:

1. **Базовый образ:** В качестве родительского слоя используется готовый базовый образ нужной архитектуры (`base-x86`, `base-jetson-agx-orin` или `base-jetson-orin-nano`).
2. **Параметризация сборки:** В шаблон передаются аргументы репозитория и целевой микроархитектуры:
   * `REPO_URL` — ссылка на репозиторий пакета.
   * `BRANCH` — ветка для клонирования.
   * `CUDA_ARCH` — флаг архитектуры (`sm_86` для Ampere x86, `sm_87` для Orin).
3. **Шаблон сборки через colcon:**
   ```dockerfile
   ARG REPO_URL
   ARG BRANCH=main
   ARG CUDA_ARCH=sm_87

   WORKDIR /workspace/ros2_ws/src
   RUN git clone -b ${BRANCH} ${REPO_URL} my_new_package

   WORKDIR /workspace/ros2_ws
   RUN /bin/bash -c "source /opt/ros/humble/setup.bash && \
       colcon build --cmake-args -DCMAKE_CUDA_ARCHITECTURES=${CUDA_ARCH} --parallel-workers 1"
   ```
4. **Матрица CI/CD:** Для сборки нового пакета достаточно добавить новую запись в матрицу `matrix` файла `.github/workflows/build.yml`.
