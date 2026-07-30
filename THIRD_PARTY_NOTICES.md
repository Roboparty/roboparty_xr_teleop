# Third-Party Notices

This repository contains software derived from third-party projects and uses
third-party dependencies. The Apache License 2.0 text in `LICENSE` does not
replace the licenses or permissions that apply to third-party material.

## Unitree xr_teleoperate

Parts of this repository are based on and modified from Unitree Robotics'
`xr_teleoperate` project:

- Upstream: https://github.com/unitreerobotics/xr_teleoperate
- Copyright [2025] [HangZhou YuShu TECHNOLOGY CO.,LTD. ("Unitree Robotics")]
- Upstream license notice: Apache License 2.0

RoboParty modified the upstream software for RPO/Roboto robot teleoperation,
PICO XR input, ROS 2 integration, and the repository's current asset and launch
layout. Source files derived and changed from the upstream project should retain
a prominent modification notice.

## Other upstream projects and dependencies

The upstream project identifies the following projects as code bases or
dependencies on which it builds. Their own license terms continue to apply:

1. https://github.com/OpenTeleVision/TeleVision
2. https://github.com/dexsuite/dex-retargeting
3. https://github.com/vuer-ai/vuer
4. https://github.com/stack-of-tasks/pinocchio
5. https://github.com/casadi/casadi
6. https://github.com/meshcat-dev/meshcat-python
7. https://github.com/zeromq/pyzmq
8. https://github.com/Dingry/BunnyVisionPro
9. https://github.com/unitreerobotics/unitree_sdk2_python

CasADi is installed separately as described in `README.md` and is not vendored
in this repository. Verify the license and version actually used when
redistributing an environment or binary.

## Robot and documentation assets

The following existing assets are recorded separately because the repository
does not currently contain enough provenance information to assign them a new
license:

- `assets/Atom01_urdf/**`, including URDF, MJCF, USD and STL mesh files
- `image.png`
- `image-1.png`

This notice records these assets only. It does not apply the Apache License 2.0
to them, does not change any existing ownership, and does not grant additional
rights. Their source and redistribution permission should be confirmed before
they are distributed independently or included in a product release.