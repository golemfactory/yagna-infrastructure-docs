# yapapi readme

[![Tests - Status](https://img.shields.io/github/workflow/status/golemfactory/yapapi/Continuous%20integration/master?label=tests)](https://github.com/golemfactory/yapapi/actions?query=workflow%3A%22Continuous+integration%22+branch%3Amaster) ![PyPI - Status](https://img.shields.io/pypi/status/yapapi) [![PyPI version](https://badge.fury.io/py/yapapi.svg)](https://badge.fury.io/py/yapapi) [![GitHub license](https://img.shields.io/github/license/golemfactory/yapapi)](https://github.com/golemfactory/yapapi/blob/master/LICENSE) [![GitHub issues](https://img.shields.io/github/issues/golemfactory/yapapi)](https://github.com/golemfactory/yapapi/issues)

## How to use

Rendering

```python
from yapapi.runner import Engine, Task, vm
from datetime import timedelta


async def main():
    package = await vm.repo(
        image_hash="ef007138617985ebb871e4305bc86fc97073f1ea9ab0ade9ad492ea995c4bc8b",
        min_mem_gib=0.5,
        min_storage_gib=2.0,
    )

    async def worker(ctx, tasks):
        ctx.send_file("./scene.blend", "/golem/resource/scene.blend")
        async for task in tasks:
            frame = task.data
            ctx.begin()
            crops = [
                {
                    "outfilebasename": "out",
                    "borders_x": [0.0, 1.0],
                    "borders_y": [0.0, 1.0],
                }
            ]
            ctx.send_json(
                "/golem/work/params.json",
                {
                    "scene_file": "/golem/resource/scene.blend",
                    "resolution": (800, 600),
                    "use_compositing": False,
                    "crops": crops,
                    "samples": 100,
                    "frames": [frame],
                    "output_format": "PNG",
                    "RESOURCES_DIR": "/golem/resources",
                    "WORK_DIR": "/golem/work",
                    "OUTPUT_DIR": "/golem/output",
                },
            )
            ctx.run("/golem/entrypoints/render_entrypoint.py")
            ctx.download_file("/golem/output/out.png", f"output_{frame}.png")
            yield ctx.commit(task)
            # TODO: Check if job is valid
            # and reject by: task.reject_task(reason = 'invalid file')
            task.accept_task()

        ctx.log("no more frame to render")

    async with Engine(
        package=package,
        max_worker=10,
        budget=10.0,
        timeout=timedelta(minutes=5),
    ) as engine:
        async for progress in engine.map(
            worker, [Task(data=frame) for frame in range(1, 101)]
        ):
            print("progress=", progress)
```

