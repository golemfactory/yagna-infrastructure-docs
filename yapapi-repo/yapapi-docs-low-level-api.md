# Low Level Api

## General Concept

![](https://github.com/golemfactory/yagna-infrastructure-docs/tree/6f4941306b695ec5b67c72a8b64884b6ec8f33cb/art/gc-nodes.svg)

```python
from yapapi.rest import Configuration, Market, Activity, Payment
from yapapi import props as yp
from yapapi.props.builder import DemandBuilder
from datetime import datetime, timezone


async def list_offers(conf: Configuration):
    async with conf.market() as client:
        market_api = Market(client)
        dbuild = DemandBuilder()
        dbuild.add(yp.Identification(name="some scanning node"))
        dbuild.add(yp.Activity(expiration=datetime.now(timezone.utc)))

        async with market_api.subscribe(
            dbuild.props, dbuild.cons
        ) as subscription:
            async for event in subscription.events():
                print("event=", event)
        print("done")
```

## Access Configuration

### Class `yapapi.rest.Configuration(...)`

**Initialization Arguments**

`app_key`: \(optional\) str : Defines access token to API Gateway

