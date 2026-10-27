[English](README.md) | 简体中文

# 🚀 Amazon 数据采集 API

[![API 状态](https://img.shields.io/badge/API-在线-brightgreen)](https://docs.pangolinfo.com/cn-index)
[![平台](https://img.shields.io/badge/平台-Amazon-blue)](https://www.pangolinfo.com/zh/)
[![数据格式](https://img.shields.io/badge/数据格式-JSON%20%7C%20HTML%20%7C%20Markdown-orange)](https://docs.pangolinfo.com/cn-index)

## 计划与联系

- 在 [tool.pangolinfo.com](https://tool.pangolinfo.com/) 注册即可获得免费计划，每月 60 credits。
- 企业用户可扫码联系我们，获取产品演示或个性化测试等。

<img src="wechat-qrcode.png" alt="微信二维码" width="200" />

*扫码添加微信，联系我们。微信：`Pangolin-Scraper`*

### 邮件

- **技术支持**: support@pangolinfo.com
- **商务合作**: charon@pangolinfo.com
- **响应时间**: 24 小时内回复

### 链接

- **官网**: [www.pangolinfo.com/zh](https://www.pangolinfo.com/zh/)
- **文档**: [docs.pangolinfo.com/cn-index](https://docs.pangolinfo.com/cn-index)
- **注册 / API key**: [tool.pangolinfo.com](https://tool.pangolinfo.com/)

---

## 项目简介

**Pangolinfo Scrape API** 由 **PANGOLIN INFO TECH PTE. LTD.**（新加坡，成立于 2022 年）提供，面向跨境卖家、数据服务商和工具开发者。

接口覆盖 **全部 Amazon 公开数据**。页面结构变化时，解析逻辑由我们持续维护，可返回标题、价格、折扣、库存、描述、评论等结构化字段，也可返回原始 HTML 或 Markdown。使用 API token 即可调用。

输出格式：**JSON**、**HTML**、**Markdown**。

### 核心优势

- **动态适配**: 自动跟随 Amazon 页面结构变化，集成方不必自己改 DOM
- **成功率**: 整体成功率 99%，SP 广告捕获率 90%+
- **Amazon 覆盖**: 全部公开数据，包括商品详情、关键词搜索、分类列表、卖家商品、热销榜、新品榜
- **多种格式**: JSON、HTML、Markdown
- **少维护**: 解析逻辑由服务端持续维护

## 支持的 Amazon 数据

Pangolinfo 支持全部 Amazon 公开数据。

| 页面 | 字段 |
|------|------|
| **Amazon** 商品详情、关键词搜索、分类列表、卖家商品、热销榜、新品榜 | ASIN、标题、价格、评分、评论、评论数、图片、销量、卖家信息、商品描述等公开字段 |

本仓库里的开源代码范围小于托管 API。它只解析你**已经拿到的 Amazon 关键词搜索结果 HTML**。代码里的站点：美国、英国、德国、法国、加拿大（`parsermgr/amazon`）。

---

## 开源 Amazon 关键词解析器

本仓库公开了 **Amazon 关键词搜索结果页** 的解析代码，用来说明如何从这类 HTML 抽出结构化字段。

**请注意：**

* 它解析的是**本地 HTML 文件**，不会发起网络请求。
* **不包含**抓取、代理、验证码处理。
* 代码按编写时的 Amazon 页面结构实现。页面改版后可能需要调整。用法见 `parsermgr/parser_test.go`。

单条商品字段（`model.AmzProduct`）：ASIN、标题、价格、星级、评论数、图片、销量、Prime、是否广告。页面结果（`model.AmzKeywordModel`）还包括关键词、页码和下一页 URL。

## 快速开始

### 1. 开源解析器

```bash
git clone https://github.com/Pangolin-spg/amazon-scraper-api.git
cd amazon-scraper-api

go mod tidy
go test ./parsermgr/
```

### 2. Scrape API

1. 在 [tool.pangolinfo.com](https://tool.pangolinfo.com/) 注册免费计划（每月 60 credits）。
2. 用邮箱和密码换 token：`POST https://scrapeapi.pangolinfo.com/api/v1/auth`。说明见[认证文档](https://docs.pangolinfo.com/cn-api-reference/authApi/authApi)。[`examples/python/get_amazon_product_sync.py`](examples/python/get_amazon_product_sync.py) 是更长的登录示例。下面的片段与当前 API 文档一致：`format` 为字符串，任务地址为 `POST https://scrapeapi.pangolinfo.com/api/v1/scrape`。
3. 提交任务：`POST https://scrapeapi.pangolinfo.com/api/v1/scrape`。

#### Python

```python
import requests
import json

# token 来自 POST https://scrapeapi.pangolinfo.com/api/v1/auth
TOKEN = "your_token_here"

def get_amazon_product(asin):
    """获取 Amazon 商品详情 JSON。"""
    url = "https://scrapeapi.pangolinfo.com/api/v1/scrape"
    headers = {
        "Authorization": f"Bearer {TOKEN}",
        "Content-Type": "application/json",
    }
    payload = {
        "url": f"https://www.amazon.com/dp/{asin}",
        "parserName": "amzProductDetail",
        "format": "json",
    }
    response = requests.post(url, headers=headers, json=payload)
    return response.json()

if __name__ == "__main__":
    result = get_amazon_product("B0DYTF8L2W")
    print(json.dumps(result, indent=2, ensure_ascii=False))
```

#### Node.js

```javascript
const axios = require("axios");

class EcommerceAPI {
    constructor(token) {
        this.token = token;
        this.baseURL = "https://scrapeapi.pangolinfo.com/api/v1/scrape";
    }

    async getAmazonProduct(asin) {
        const payload = {
            url: `https://www.amazon.com/dp/${asin}`,
            parserName: "amzProductDetail",
            format: "json",
        };

        const response = await axios.post(this.baseURL, payload, {
            headers: {
                Authorization: `Bearer ${this.token}`,
                "Content-Type": "application/json",
            },
        });

        return response.data;
    }
}

const api = new EcommerceAPI("your_token");
api.getAmazonProduct("B0DYTF8L2W")
    .then((data) => console.log(data))
    .catch((err) => console.error(err));
```

`format` 可选 `json`、`rawHtml`、`markdown`。其他 `parserName`（`amzKeyword`、`amzProductOfCategory`、`amzProductOfSeller`、`amzBestSellers`、`amzNewReleases`）见 [Amazon Scraper API 文档](https://docs.pangolinfo.com/cn-api-reference/amazonApi/amazonScrapeAPI)。

## 简化解析示例

仓库里实际维护的解析器在 `parsermgr`（使用 `htmlquery`）。下面只保留关键词页解析的大致形状：

```go
// 简化的 Amazon 关键词搜索解析示例。
package parser

import (
    "github.com/PuerkitoBio/goquery"
    "strings"
)

type AmazonProduct struct {
    ASIN     string  `json:"asin"`
    Title    string  `json:"title"`
    Price    float64 `json:"price"`
    Rating   float64 `json:"rating"`
    Reviews  int     `json:"reviews"`
    ImageURL string  `json:"image_url"`
}

func ParseAmazonSearchPage(html string) ([]AmazonProduct, error) {
    doc, err := goquery.NewDocumentFromReader(strings.NewReader(html))
    if err != nil {
        return nil, err
    }

    var products []AmazonProduct

    doc.Find("[data-asin]").Each(func(i int, s *goquery.Selection) {
        asin, _ := s.Attr("data-asin")
        if asin == "" {
            return
        }

        product := AmazonProduct{
            ASIN:     asin,
            Title:    s.Find("h2 a span").Text(),
            ImageURL: s.Find("img").AttrOr("src", ""),
        }

        // 价格来自 .a-price-whole，评分来自 .a-icon-alt。
        products = append(products, product)
    })

    return products, nil
}
```

## 功能对比

| 功能 | 开源模板 | 免费 API | 付费 API |
|------|----------|----------|----------|
| 能力范围 | 关键词搜索页 HTML 解析 | Amazon 公开数据 | Amazon 公开数据 |
| 额度 | 本地解析，不消耗 API credits | 免费计划，每月 60 credits | 按付费套餐 |
| 技术支持 | 社区 | 官方支持 | 优先支持 |
| 自行改解析 | 源码可改 | 不支持 | 可谈定制 |
| 反爬 | 需自建 | 已包含 | 已包含 |
| 成功率 | 自行维护 | 整体 99% | 整体 99% |

托管 API 的 SP 广告捕获率为 **90%+**。套餐额度见[计费说明](https://docs.pangolinfo.com/cn-help-center/amz-billing-subscription/basePlans)。

## 参与贡献

1. **Fork** 本仓库
2. **创建**分支（`git checkout -b feature/AmazingFeature`）
3. **提交**更改（`git commit -m 'Add some AmazingFeature'`）
4. **推送**（`git push origin feature/AmazingFeature`）
5. **提交** Pull Request

### 贡献者回馈

- GitHub 主页上的致谢
- 贡献者 API 额度
- 与维护者直接交流
- 问题优先处理

### 欢迎的贡献

- 关键词解析器的缺陷修复
- 文档和示例
- 测试
- 解析器里的更多 Amazon 站点或语言

## 使用场景

### 学习研究

- 理解 HTML 解析
- 数据项目
- 用本地 HTML 样本看采集链路
- 基于已有数据做市场分析

### 个人项目

- 价格监控
- 商品趋势分析
- 选品辅助
- 个人数据看板

### 商业应用

- 竞品分析
- Amazon 数据服务
- 运营自动化
- 市场研究报告

## 文档

- **文档首页**: [docs.pangolinfo.com/cn-index](https://docs.pangolinfo.com/cn-index)
- **Amazon Scraper API**: [amazonScrapeAPI](https://docs.pangolinfo.com/cn-api-reference/amazonApi/amazonScrapeAPI)
- **认证**: [authApi](https://docs.pangolinfo.com/cn-api-reference/authApi/authApi)
- **套餐与计费**: [basePlans](https://docs.pangolinfo.com/cn-help-center/amz-billing-subscription/basePlans)
- **计费常见问题**: [faq](https://docs.pangolinfo.com/cn-help-center/amz-billing-subscription/faq)

## 路线图

- [ ] 完善 Amazon 解析器文档
- [ ] 提供 Docker 部署方式
- [ ] 更多编程语言的 SDK
- [ ] 在本仓库覆盖更多 Amazon 公开数据与站点
- [ ] 改进解析
- [ ] 云端部署方案
- [ ] 可视化配置界面

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=Pangolin-spg/amazon-scraper-api&type=Date)](https://star-history.com/#Pangolin-spg/amazon-scraper-api&Date)

## 开源协议

[MIT License](LICENSE)。

---

## 立即开始

1. 给本仓库点一个 Star
2. [注册](https://tool.pangolinfo.com/)免费计划，每月 60 credits
3. 企业用户如需产品演示或个性化测试，请扫上方微信二维码

---

*如果这个项目对你有帮助，欢迎点一个 Star。*
