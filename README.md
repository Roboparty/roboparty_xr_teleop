# Roboparty XR Teleop

基于 [Unitree xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate) 修改，面向 RPO/Roboto 机器人与 PICO VR 的双臂遥操作。

## 环境要求

- Ubuntu 20.04 / 22.04
- Python 3.10
- ROS2 Python 环境
- PICO VR 设备
- RPO/Roboto 机器人控制链路

## 安装

### 1. 创建环境

推荐先创建独立环境：

```bash
conda create -n roboparty_xr python=3.10 -y
conda activate roboparty_xr
```

### 2. 安装 IK 依赖

`pinocchio` 和 `casadi` 建议通过 `conda-forge` 安装：

```bash
conda install -c conda-forge pinocchio casadi numpy=1.26.4 -y
```

### 3. 安装仓库依赖

在仓库根目录执行：

```bash
pip install -r requirements.txt
```

### 4. 准备 ROS2

运行本项目前，请确认当前 Python 环境与 ROS2 使用的 Python 小版本一致。

本项目当前推荐/测试组合为：

- Python `3.10`
- 与 Python `3.10` 匹配的 ROS2 Python 环境

运行前请确认当前终端已经 `source` 过对应的 ROS2 环境，并且可以正常导入：

- `rclpy`
- `sensor_msgs`

可以直接用下面命令自检：

```bash
python -c "import rclpy; from sensor_msgs.msg import JointState; print('ROS2 Python OK')"
```

注意：

- 如果 ROS2 是基于 Python `3.12` 安装或构建的，而当前项目运行在 Python `3.10` 环境中，通常无法正常导入 `rclpy`
- 反过来也是一样，Python `3.12` 环境通常也不能直接使用基于 Python `3.10` 的 ROS2 Python 包
- 不要混用不同 Python 小版本的 ROS2 和本项目运行环境

## 证书配置

PICO 通过浏览器访问遥操作页面时，需要 HTTPS 证书。

在仓库根目录执行：

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem
```

如果主机开了防火墙，请放行 `8012` 端口：

```bash
sudo ufw allow 8012
```

## 机器人侧平衡 Policy

本项目负责 `VR -> 上半身遥操作` 控制链路。真机运行时，人形机器人本体仍需要底层 `policy` 负责站立与平衡控制，因此通常需要配合 [`roboparty_deploy`](https://github.com/Roboparty/roboparty_deploy) 一起使用。

如果需要切换机器人底层策略模型，请先修改 `roboparty_deploy` 中的 `src/inference/launch/inference.launch.py`，把加载的配置文件改成目标 policy：

```python
configs = [
    os.path.join(
        get_package_share_directory("inference"),
        "config",
        "inference_interrupt.yaml",
    ),
]
```

常见可选配置文件包括：

- `inference_amp.yaml`
- `inference_attn_enc.yaml`
- `inference_beyondmimic.yaml`
- `inference_getup.yaml`
- `inference_interrupt.yaml`

本项目当前建议使用：

- `inference_interrupt.yaml`

修改完成后，重新运行 `./tools/start_robot.sh`，机器人启动时就会加载对应配置，并使用相应的底层 policy。

详细说明请参考 [`roboparty_deploy` 的 README_CN](https://github.com/Roboparty/roboparty_deploy/blob/main/README_CN.md)。

机器人侧遥控器按键操作也请参考该文档。结合本项目，主要会用到下面几个按键：

- `X`：使能 / 失能电机
- `A`：复位
- `B`：进入推理模式。进入后机器人可以保持平衡并正常行走，但此时上半身控制接口还没有暴露给 VR 遥操作
- `LB`：将上半身控制接口暴露给 VR 遥操作

推荐的真机联调流程如下：

1. 在机器人侧启动 `roboparty_deploy`，并确认已经加载 `inference_interrupt.yaml`
2. 使用机器人遥控器按 `X` 使能电机，按 `A` 让机器人复位
3. 按遥控器 `B` 进入推理模式，确认机器人已经能够稳定站立、保持平衡并正常行走
4. 在本仓库中启动 VR 遥操作程序，并按照下文流程完成 PICO 连接，使仿真机器人开始跟随操作者动作
5. 先不要立即把真机暴露给 VR，先观察仿真机器人与真机机器人的上半身姿态，尽量让两者接近
6. 当仿真机器人和真机机器人姿态已经基本对齐后，按机器人遥控器的 `LB`，将上半身控制接口暴露给 VR
7. 最后在 VR 遥操作端继续执行到按下 VR controller 的 `A`，开始向真机发送上半身控制命令

也就是说，真机真正进入上半身遥操作，需要同时满足两边条件：

- 机器人侧已经按下遥控器 `LB`，允许 VR 接管上半身接口
- 遥操作侧已经按下 VR controller 的 `A`，开始发送控制命令

## 启动

在仓库根目录执行：

```bash
python teleop/xr_control_rpo.py --xr-mode controller
```

常用参数：

- `--xr-mode {hand,controller}`：选择 XR 跟踪模式，默认 `controller`
- `--frequency`：控制频率，默认 `30`
- `--headless`：关闭 IK 可视化。真机联调时建议开启，以减少 Meshcat 带来的额外开销
- `--profile-loop`：打印控制环耗时统计，便于分析 `XR -> IK -> ROS` 各环节耗时

如果你想在真机前先做性能检查，推荐先使用：

```bash
python teleop/xr_control_rpo.py --xr-mode controller --headless --profile-loop
```

终端会打印类似下面的统计信息：

```text
Loop timing avg over N iters: motion=... ms, ik=... ms, send=... ms, loop=... ms, rate=... Hz, ipopt_iter_avg=..., ipopt_iter_max=..., ipopt_hit_max=.../N, ipopt_last_status=...
```

字段含义：

- `motion`：XR 数据读取与坐标转换耗时
- `ik`：`arm_ik.solve_ik()` 耗时
- `send`：ROS 关节命令发布耗时
- `loop`：整轮控制循环耗时
- `rate`：实际控制频率
- `ipopt_iter_*`：单次 IK 求解的 Ipopt 迭代统计

## PICO 使用流程

以下流程默认：

- PICO 与运行程序的主机在同一局域网
- 主机 IP 为 `192.168.123.2`
- 使用 `controller` 模式

如果你的主机 IP 不同，请把下面 URL 中的 `192.168.123.2` 替换成实际地址。

### 1. 启动控制程序

```bash
python teleop/xr_control_rpo.py --xr-mode controller
```

程序启动后会等待开始信号，同时浏览器会弹出 Meshcat 仿真界面。

<p align="center">
  <img src="image.png" alt="Meshcat 仿真界面" width="80%">
</p>

### 2. 在 PICO 浏览器打开页面

在 PICO 浏览器访问：

```text
https://192.168.123.2:8012/?ws=wss://192.168.123.2:8012
```

如果浏览器提示证书不安全，点击继续访问即可。

### 3. 进入 VR

进入页面后：

1. 点击 `Pass-Through`
2. 接受浏览器和设备弹出的权限请求
3. 等待 XR 会话建立完成

终端出现连接日志后，说明 PICO 已接入成功。

### 4. 开始遥操作

推荐真机使用下面这条命令启动：

```bash
python teleop/xr_control_rpo.py --xr-mode controller --headless
```

终端键位：

- `r`：开始主循环。按下后程序开始接收 VR 传输的数据，Meshcat 中的机器人会跟随操作者动作，但此时真机不会运动。
- `a`：允许发送机械臂命令。按下后，真机开始接收电脑发送的关节位置。
- `q`：退出程序。

建议在按下 `a` 之前，先让 Meshcat 中的机器人姿态尽量接近真机复位姿态。

<p align="center">
  <img src="image-1.png" alt="开始发送命令前的姿态对齐示意" width="80%">
</p>

VR 控制器键位：

- 右手 `A`：开始发送命令。效果与键盘 `a` 一致。
- 右手 `B`：停止发送命令。停止后仍可再次按 `A` 恢复。

建议先让操作者手臂接近机器人真机姿态，再开始发送控制命令。

## 运行建议

- 首次使用时，先在安全空间内做小范围动作测试
- 开始发送命令前，确认机器人周围无人靠近
- 退出前，建议先让机械臂回到较自然的位置

## 目录结构

```text
roboparty_xr_teleop/
├── assets/
│   └── Atom01_urdf/  # compatibility asset path
├── teleop/
│   ├── robot_control/
│   │   └── robot_arm_ik.py
│   ├── utils/
│   │   └── weighted_moving_filter.py
│   ├── xr_control_rpo.py
│   └── xr_Control_Atom.py  # compatibility entrypoint
├── televuer/
├── LICENSE
├── README.md
└── requirements.txt
```

## 致谢

本项目基于 [Unitree xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate) 修改实现。相关许可及第三方归属信息见 [LICENSE](LICENSE) 和 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
