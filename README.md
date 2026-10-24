# amazon scraper python：代理和验证码全托管，Hobby $49/月够抓 2 万个亚马逊页面（附全套餐价格与选档规则）

用 Python 抓亚马逊数据，写解析代码只占工作量的一小部分，大头全花在反爬上：换 User-Agent、维护住宅代理池、解验证码、模拟浏览器 TLS 指纹。ScraperAPI 的思路是把这一整层外包出去——你把目标 URL 交给它，它用 4,000 多万 IP 的代理池和自动验证码处理把页面带回来，按量扣 API credits。关键在于它的域名计价：amazon.com 每次请求固定扣 5 credits，Hobby 套餐 $49/月包含 100,000 credits，换算下来够抓 2 万个亚马逊页面，折合约 $2.45/千页。本文按官网当前价格把 8 个套餐全部列出来，算清真实成本，再讲清楚几个定价页上不显眼的限制。

## 选购速览

> 大多数个人开发者和中小项目：选 **Hobby（$49/月，年付折合 $44.10/月）**，10 万 credits 按亚马逊 5 credits/请求算，月抓 2 万个页面以内够用，而且额度用完直接停止、不会产生超额账单。不确定用量的话先跑免费试用——注册 7 天内 5,000 credits，不绑信用卡。👉 [领取 ScraperAPI 7 天免费试用（5,000 credits，无需信用卡）](https://www.scraperapi.com/?fp_ref=coupons)

## 用 Python 抓亚马逊，卡住你的到底是什么

直接用 `requests` 请求亚马逊搜索页，返回的通常不是商品列表，而是一段让你联系 api-services-support@amazon.com 申请自动化访问的反爬提示。加上 Chrome 的 User-Agent 和 `Accept-Language` 请求头，偶尔能过，但同一 IP 多请求几次就会被丢验证码。

GitHub 上排名靠前的开源方案基本都在这个思路上延伸：Scrapy 版的亚马逊爬虫（比如 ian-kerins 的 amazon-python-scrapy-scraper 及各种 fork）、requests + BeautifulSoup 的教程仓库、scrape-do 和 Thordata 放出的命令行工具。低频抓取时确实能用，共同前提是你要自己解决 IP 和指纹问题——一般做法是再接一个住宅代理服务，或者用 `curl_cffi` 模拟真实浏览器的 TLS 握手。亚马逊的 TLS 指纹检测能识别 Python 默认 `requests` 库的 OpenSSL 特征，这正是很多教程代码“本地跑得通、挂到服务器第二天就废”的原因。

ScraperAPI 把这些收进一个 API：请求发到 `api.scraperapi.com`，带你的 API key，它负责选代理、重试、解验证码，代码里不再出现代理地址。超并发时返回 429 直接拒绝（不扣 credits 也不排队），所以客户端代码要带重试逻辑，这是迁移过来后唯一需要自己处理的部分。

## ScraperAPI 全套餐价格表（官网当前在售的全部方案）

官方现在共 8 档：Free 加 6 个自助付费档，再加定制报价的 Enterprise。所有付费套餐功能一致——JS 渲染、住宅/移动 IP、反爬绕过、结构化数据端点都不锁套餐，差别只在 credits、并发线程和地理定向范围。年付统一 10% off。

| 套餐 | 月付 | 年付折合 | Credits | 并发线程 | 购买入口 |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [免费注册领 1,000 credits](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10/月 | 100,000 | 20 | [开通 Hobby 套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10/月 | 1,000,000 | 50 | [开通 Startup 套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10/月 | 3,000,000 | 100 | [开通 Business 套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling（最受欢迎） | $475 | $427.50/月 | 5,000,000 | 200 | [开通 Scaling 套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50/月 | 10,500,000 | 300 | [开通 Professional 套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50/月 | 21,500,000 | 500 | [开通 Advanced 套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | 定制报价 | 定制 | 22,000,000+ | 500+ | [联系销售定制方案](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

几个影响选档但容易被忽略的细节：

- **地理定向**：Hobby 和 Startup 只支持 US & EU 两个区域，Business 起才开放 50+ 国家定向。要抓日本站（.co.jp）、德国站（.de）的本地化价格，Hobby 档的定向能力不够。
- **分析面板**：Hobby 的数据面板只保留最近 30 天记录，Business 起无限制。
- **年付不需要优惠码**：10% off 是定价页直接标出的价格。第三方渠道流传的 START10 之类首月九折码力度与年付相当，是否有额外叠加以结账页实际显示为准。

## 抓亚马逊的真实成本：把 credits 换算成页面数

这是多数测评文章没算清的一层。ScraperAPI 对特殊域名单独计价：普通页面 1 credit，**amazon.com、Walmart、eBay 每次请求 5 credits**，Google/Bing 25，LinkedIn 30。所以表格里的 credits 要除以 5 才是你能抓的亚马逊页面数：

| 套餐 | 月 credits | 可抓亚马逊页面 | 折合每千页成本 |
| --- | --- | --- | --- |
| Hobby | 100,000 | 20,000 | $2.45 |
| Startup | 1,000,000 | 200,000 | $0.745 |
| Business | 3,000,000 | 600,000 | $0.498 |
| Scaling | 5,000,000 | 1,000,000 | $0.475 |

同样的 credits 拿去抓普通网站就是 10 万次请求（$0.49/千次），拿去抓 Google SERP 只够 4,000 次（$12.25/千次）。同一个套餐在不同目标上的成本差 30 倍，只看“每千 credits 单价”会严重误判。

参数也有成本差异。`render=true`（JS 渲染）、`premium=true`（住宅代理）、`screenshot=true` 各加 10 credits；组合计价非线性叠加，premium + render 是 25 而非 20，ultra_premium + render 是 75。抓亚马逊普通搜索页和商品页一般用不到这些。真正的好消息是这几个参数免费：`country_code`、`session_number`（同一 IP 会话保持）、`device_type`、`output_format`、`keep_headers`、`autoparse`。地理定向和会话粘性不额外收费，在同价位产品里少见。

计费口径对失败重试友好：只对 HTTP 200 和 404 的成功请求收费，以及你在 70 秒时限内主动取消的请求；403 或超时不扣 credits。动手前可以用 dashboard 里的 Domain Cost Estimator 查任意 URL 的单次成本，也能给每次抓取设 `max_cost` 上限；不喜欢图形界面就调 `https://api.scraperapi.com/account/urlcost` 这个端点，支持全部参数。

👉 [查看全部套餐的年付价格与额度规则](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## 三种用法：从原始 HTML 到直接返回 JSON

按投入的解析工作量从多到少，ScraperAPI 有三种用法。

**第一种，当代理用，返回 HTML。** 拿到 API key 后一个标准请求长这样：

python
import requests

payload = {
    "api_key": "YOUR_SCRAPERAPI_KEY",
    "url": "https://www.amazon.com/s?k=mechanical+keyboard",
    "output_format": "markdown",
}
response = requests.get("https://api.scraperapi.com", params=payload)
print(response.text)


后续解析和普通页面没区别：亚马逊搜索结果项包裹在带 `data-component-type="s-search-result"` 属性的 `div` 里，商品名在 class 为 `a-size-medium a-color-base a-text-normal` 的 `span` 中，推荐先取全部结果块再循环提取字段，方便发现缺数据的条目。

**第二种，加 `autoparse=true`。** 对解析质量要求不高时，这个参数让请求直接返回初步结构化的结果，不加价。

**第三种，Amazon 结构化数据端点，完全不写解析代码。** 官方提供 Amazon Product、Amazon Search、Amazon Offers 三个预构建端点，直接返回 JSON。Product 端点按 ASIN 查询，返回商品名、价格、库存、评分、评论数、图片、卖家信息等 18 个以上字段；Search 端点返回搜索结果及商品位置，含 sponsored 标记；Offers 端点返回第三方卖家报价，做 Buy Box 监控和竞品定价分析用。计费和核心 API 一致，Amazon 端点每请求同样是 5 credits，且所有套餐（包括 Hobby 和 Startup）都能用。

支持 21 个国际亚马逊站点（.com、.co.uk、.de、.co.jp、.com.au 等），美国站还支持 ZIP 级地理定向，可以拉取指定收货地址区域的区域价格和物流数据——做亚马逊跨区比价或价格监控的话，这个功能基本是刚需。抓评论另有对应的 Review API，按 ASIN 拉取，省掉翻页和懒加载处理。

一个具体返回示例（官方教程数据）：商品 "iPhone 15 Charger, [Apple Certified] 20W USB C"，ASIN `B0CD6YLMBK`，4.3 星，34 条评论，`price_string` 为 "$16.99"，`has_prime` 为 true，还带 `is_best_seller`、`is_amazon_choice`、`is_limited_deal` 等标记字段。

想验证自己目标页面的实际消耗和返回质量，注册后在 API Playground 里贴 URL 测一下就行。👉 [用免费试用实测你的亚马逊目标页面](https://www.scraperapi.com/?fp_ref=coupons)

## 额度用完会发生什么：低价套餐是“硬停”

这一条对预算控制最关键，但定价页上很不显眼。**Hobby、Startup、Business 三档的 credits 用到 100% 后，请求直接停止**，不会产生超额账单，只能等下周期刷新、手动升级或找客服；官方原话是 hitting 100 percent of your credits "does not produce a bigger invoice. It produces a stop"。只有 Scaling 及以上提供 pay-as-you-go（按固定单价继续抓、可设月度消费上限），auto-upgrade 自动升档需手动开启，覆盖 Hobby 到 Professional 五档。

另一个限制：credits 不滚存，订阅续期时余额重置。闲置月份的额度攒不到繁忙月份用，所以选套餐应该按峰值月的量定，不是按月平均——月均 1.5 万页但峰值 3 万页的项目，Hobby 会在峰值月断粮，Startup（月抓 20 万个亚马逊页面）才是安全线。

👉 [对照各档年付价与超额规则](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## 和其他方案比，ScraperAPI 适合谁

横向有三类选择，按抓取规模说。

**每月几百个页面的低频需求**：自建最省钱。开源 Scrapy 爬虫加免费额度就能跑，但代理是绕不开的开销，选择器也要跟着亚马逊改版维护。这个量级没必要付费。

**每月几千到几十万个页面、需要稳定 JSON 输出**：ScraperAPI 的甜区。第三方在 2025 年 7 月做过一次四家亚马逊抓取 API 的实测（对 com、com.br、in、ca 四个站点各发 10 次请求），ScraperAPI 成功率 100%，平均响应 9.61 秒；更早的独立基准测试也录得 100% 成功率、约 11.8 秒平均响应。速度在同类里偏慢（同次测试中 Scrapingdog 平均 3.55 秒、Oxylabs 约 5.38 秒、Bright Data 面板宣称约 9 秒），但对批量任务来说成功率通常比单次速度更影响总成本——失败重试在 ScraperAPI 这里不扣钱。

**单价敏感的大批量场景**：要说清楚，ScraperAPI 入门单价确实偏高。同一次测评的换算口径下，Scrapingdog 约 $0.20/1K 起、量大降到 $0.063/1K；Bright Data 固定约 $0.90/1K；Oxylabs $1.35→$1.25/1K；ScraperAPI $2.45→$0.475/1K。不过那份测评出自 Scrapingdog 官方博客，立场不中立，且各家对亚马逊的域名计价和结构化支持差异很大，结论只当参考。ScraperAPI 相对的强项在“只对成功请求计费 + 地理定向免费 + 21 个站点 + ZIP 级定向”的组合，以及结构化端点的字段稳定性。月抓百万页以上且对单价极度敏感的项目，值得先用各家免费额度实测自己的目标页面再换算。

## FAQ

**ScraperAPI 有免费版吗？**
有。注册即得 1,000 个免费 credits，最多 5 个并发，不用信用卡；所有账号另有 7 天试用，含 5,000 credits。免费额度够验证目标页面能否抓通、确认解析字段。

**年付和月付差多少？**
所有付费套餐年付统一 10% off。Hobby 从 $49/月降到 $44.10/月，一年省 $58.80；Scaling 从 $475 降到 $427.50，一年省 $570。年付可提前续期获取新额度。

**结构化数据端点要另外付费吗？**
不用。按域名计费，和核心 API 用同一套 credits——Amazon 端点每请求 5 credits，与直接抓亚马逊 HTML 相同。区别只在于端点直接返回解析好的 JSON，省掉自己写选择器和维护的成本。

**怎么估算我的项目该选哪档？**
峰值月的亚马逊页面数乘以 5 就是所需 credits。月抓 2 万页以内 Hobby 够用，硬停机制兜底不会超支；2 万到 20 万页看 Startup；需要 50+ 国家定向或超额消费能力，从 Business/Scaling 起步。

## 下单前的最后确认

按情况直接给结论：

- 只是偶尔查几十个商品价格、能接受失败重试：先用 1,000 credits 的免费版跑，别急着付费。
- 每月稳定抓几千到 2 万个亚马逊页面、只看美欧站点：Hobby 月付 $49（年付 $44.10/月）是最常见的落点，额度用完自动停，不会有意外账单。
- 要抓 .co.jp、.de 等 21 个站点之外更细的区域价格，或用量会冲破 Hobby 上限：直接上 Business 以上，地理定向和超额策略才完整。
- 预算极度敏感、月抓百万页以上：拿各家免费额度实测对比后再定，ScraperAPI 在这个量级的卖点在成功率、ZIP 级定向和结构化输出，而非单价。

确认完峰值月的页面数，就可以去选档了：👉 [按年付价开通 ScraperAPI（Hobby 折合 $44.10/月）](https://www.scraperapi.com/pricing/?fp_ref=coupons)
