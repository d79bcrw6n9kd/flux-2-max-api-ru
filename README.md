# FLUX 2 Max API (`flux-2-max`)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

<p align="center">
  <img src="assets/01-product-hero-perfume.jpg" width="820" alt="Sample render via the APIMart image API">
</p>

> **from $0.056 per image** (1MP $0.056 · 2MP $0.08 · 3MP $0.104) — одна OpenAI-совместимая конечная точка на `https://api.apimart.ai/v1`, ежемесячный тариф не нужен. *(данные на 2026-10-12)*

**[Получить API-ключ](https://go.apimart.ai/k-860501)** · **[Актуальные цены](https://go.apimart.ai/k-ff4692)** · **[Страница модели](https://go.apimart.ai/k-7b7bab)** · [⚡ Быстрый старт за 60 секунд](#quickstart)

**Почему команды используют FLUX 2 Max (`flux-2-max`) через APIMart**

- **Один ключ — весь каталог.** Тот же base URL `https://api.apimart.ai/v1` и тот же заголовок `Authorization` дают доступ к FLUX 2 Max (`flux-2-max`) и ещё 300+ моделям для изображений, видео и языка — менять нужно поле `model`, а не клиент.
- **Минимум $1, оплата по факту.** Ни подписки, ни предоплаченного тарифа, который нужно заранее рассчитать: пополняйте от $1 и расходуйте на вызовы.
- **Сумма списания возвращается в ответе.** Каждая завершённая задача сообщает списанную сумму (`cost` / `credits_cost`); задача со статусом `failed` возвращается на баланс.
- **Асинхронность по умолчанию.** Отправьте задачу, получите `task_id`, опрашивайте `GET /v1/tasks/{id}` — пакетная обработка и повторы это обычная работа с очередью, а не отдельная интеграция.

<!-- /conv-kit:v1 -->

**Также известен как:** Флюкс

FLUX 2 Max is the top FLUX 2 tier, priced per megapixel of output.

## Model id and routes

| Route | `model` value | Billing | Notes |
| --- | --- | --- | --- |
| Per-image | `flux-2-max` | per delivered image, by resolution | see the [model page](https://go.apimart.ai/k-7b7bab) for the exact tiers |

Endpoint: `POST https://api.apimart.ai/v1/images/generations`, then poll `GET /v1/tasks/{task_id}`.

## Pricing

<!-- pricing:model:start -->
| Output | List price | Effective price |
| --- | --- | --- |
| default | 0.1 | 0.08 |
| 1MP | 0.07 | 0.056 |
| 2MP | 0.1 | 0.08 |
| 3MP | 0.13 | 0.104 |
| 4MP | 0.16 | 0.128 |


<!-- pricing:model:end -->

Prices are a snapshot; the [pricing page](https://go.apimart.ai/k-ff4692) and [`data/model.json`](data/model.json) are refreshed by CI, and a completed task reports the exact amount in its `cost` field.

## Request parameters

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `model` | string | required | the model id shown in the route table above |
| `prompt` | string | required | what to generate, or what to change when editing |
| `size` | string | `auto` / `1:1` | aspect ratio, or `auto` where the route supports it |
| `resolution` | string | `1K` | output resolution tier — cost scales with it |
| `n` | integer | `1` | number of images per task, where the route allows more than one |
| `image_urls` | string[] | — | reference images as public HTTP(S) URLs or base64 data URLs |
| `nsfw_check` | boolean | `false` | runs `omni-moderation-latest` before submitting |

The table lists the fields accepted by the APIMart `images/generations` endpoint. Check the [model page](https://go.apimart.ai/k-7b7bab) for anything this model adds on top.

## Quickstart

```bash
curl --request POST --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" --header 'Content-Type: application/json' \
  -d '{"model":"flux-2-max","prompt":"A bamboo forest at sunrise, slow push-in","size":"1:1","resolution":"1K"}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

created = requests.post(f"{BASE}/images/generations", headers=HEADERS, timeout=60, json={
    "model": "flux-2-max", "prompt": "A bamboo forest at sunrise, slow push-in",
}).json()
task_id = created["data"]["id"]

while True:
    task = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json()["data"]
    if task["status"] in ("completed", "failed"):
        break
    time.sleep(5)
print(task.get("cost"), task["result"]["images"][0]["url"])
```

Runnable versions: [`examples/`](examples). The task lifecycle is `pending → processing → completed | failed`.

## FAQ

**Какой id у модели FLUX 2 Max?**

`flux-2-max`. Смотрите таблицу маршрутов выше.

**Как тарифицируется?**

**from $0.056 per image** (1MP $0.056 · 2MP $0.08 · 3MP $0.104). Завершённая задача указывает точную сумму в поле `cost`; неудачная задача возвращается на баланс.

**How do I get a key?**

Use the [Get an API key](https://go.apimart.ai/k-860501) link — $1 minimum top-up, no plan required.

<!-- related-repos:auto -->
## Related repositories

- **Same model, this language** — the other half of this pair: [flux-2-max-prompts-ru](https://github.com/d79bcrw6n9kd/flux-2-max-prompts-ru)
- **Same model, other languages** — 9 more language builds: 中文 [flux-2-max-api-cn](https://github.com/apimartg583b51/flux-2-max-api-cn) · Deutsch [flux-2-max-api-de](https://github.com/nk7pvzvbm3b8/flux-2-max-api-de) · Español [flux-2-max-api-es](https://github.com/xmob24gstm0hwf/flux-2-max-api-es) · Français [flux-2-max-api-fr](https://github.com/apimartg583b51/flux-2-max-api-fr) · Relay/gateway [flux-2-max-api-gateway](https://github.com/apimartpzjt/flux-2-max-api-gateway) · Bahasa [flux-2-max-api-id](https://github.com/rtkz3/flux-2-max-api-id) · 日本語 [flux-2-max-api-ja](https://github.com/nbkj6/flux-2-max-api-ja) · 한국어 [flux-2-max-api-ko](https://github.com/ibmf1yp-x2df/flux-2-max-api-ko) · Português [flux-2-max-api-pt](https://github.com/j3jxmbmj/flux-2-max-api-pt)
- **More models in this language** — [midjourney-api-ru](https://github.com/sfpxvicw/midjourney-api-ru) · [gemini-2-5-flash-image-preview-api-ru](https://github.com/zwtdva8/gemini-2-5-flash-image-preview-api-ru) · [nano-banana-api-ru](https://github.com/ggpbi40ss/nano-banana-api-ru) · [gemini-3-1-flash-image-preview-api-ru](https://github.com/lu60av/gemini-3-1-flash-image-preview-api-ru) · [nano-banana-2-api-ru](https://github.com/jnp3hlry/nano-banana-2-api-ru) · [gemini-3-pro-image-preview-api-ru](https://github.com/dbxz24/gemini-3-pro-image-preview-api-ru)
- **Topic hubs** — every build of this model, all languages: [flux-2-max](https://github.com/topics/flux-2-max) · [flux-2-max-api](https://github.com/topics/flux-2-max-api)

This repository is one build in a cross-linked set of API guides for the same model family; each language build links back to the others.
<!-- /related-repos:auto -->

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Model page | <https://go.apimart.ai/k-7b7bab> | `apimart.ai/ru/model/flux-2` |
| Live pricing | <https://go.apimart.ai/k-ff4692> | `apimart.ai/ru/pricing` |
| Get an API key | <https://go.apimart.ai/k-860501> | `apimart.ai/ru/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

FLUX 2 Max is a third-party model served through APIMart; this repository documents how to call it and publishes
model ids and prices, and does not claim official status. Model names, prices and documentation belong to their
respective owners.

## License

MIT — see [LICENSE](LICENSE).
