# Try Debugt on Robot Shop

One request buys a robot. Debugt shows every call it makes across five services — Node, Go,
Python and Java — with the lines each one ran and the values they changed. The rest of the shop
runs in Docker. Five is what Debugt's Free plan runs at once.

## 1. Install, once

On your machine:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Node 18 or later
- Python 3
- A JDK, 11 or 17
- Go 1.22 or later

In VS Code, these extensions:

| Extension | For |
|---|---|
| [Debugt](https://marketplace.visualstudio.com/items?itemName=debugt-labs.debugt-extension) | Debugt itself |
| Python Debugger (`ms-python.debugpy`) | `payment` |
| Language Support for Java (`redhat.java`) and Debugger for Java (`vscjava.vscode-java-debug`) | `shipping` |
| Go (`golang.go`) | `catalogue-go` |

Node needs no extension: VS Code debugs it out of the box.

## 2. Get the shop running

```bash
git clone https://github.com/debugxt/debugt-demo-robot-shop.git
cd debugt-demo-robot-shop
docker compose -f docker-compose.yaml -f docker-compose.debugt.yaml -f docker-compose.debugt-five.yaml up -d \
  mongodb redis rabbitmq mysql catalogue user cart ratings dispatch web
(cd catalogue && npm install)
(cd payment && python3 -m venv .venv && .venv/bin/pip install Flask requests pika prometheus_client opentracing instana)
code .
```

The first `docker compose` pulls the images, which takes a few minutes. On an Apple Silicon Mac,
`mongodb` and `mysql` run as amd64 images under emulation: slower to start, otherwise the same.

## 3. In Debugt

1. Debugt asks whether to use the setup this repo ships. Choose **Use it**.
2. In Debugt's header, pick the **Robot Shop · five services** canvas, and press **Run all**.
   The first run of `shipping` takes a minute or two while VS Code's Java extension builds it.
3. Open **Requests → buy-a-robot → buy robot**, and press **Send**.

The answer is the purchase: each call with its status and time, the order and its total. The
trace beside it is the request's whole journey: open a hop to see what was sent and answered,
and its execution to see the code that ran.

## Things to try

- **Mock a dependency.** `payment` calls PayPal on every order, which refuses automated calls —
  so a URL override on `payment`, **Payment gateway**, answers for it. Switch it off in payment's
  details, under URL overrides, send again, and watch the order fail on PayPal's 403.
- **Change the order.** The body takes `sku`, `qty`, `country` and `city` — try
  `{ "qty": 3, "country": "us", "city": "new" }`.
- **Use the shop.** The storefront, http://localhost:8080, goes through the same five services.

## If something fails

| What you see | Why |
|---|---|
| `fetch failed` on `localhost:8082`, `8083` or `8086` | `user`, `cart` or `ratings` is not up in Docker: run the `docker compose` line again. |
| **Run all** is greyed out | Another canvas has services running. Stop them, or pick that canvas again. |
| `shipping` fails to start | VS Code's Java extension is still building it: watch the Java status in the status bar, and run again. |
| `payment` fails to start | The Python environment is missing: run the `payment` line of step 2 again. |

Every service from VS Code, and how this setup is made: [DEBUGT.md](DEBUGT.md).
