English | [简体中文](README.zh-CN.md)

# 🚀 Amazon Data Collection API

[![API](https://img.shields.io/badge/API-online-brightgreen)](https://docs.pangolinfo.com/en-index)
[![Platform](https://img.shields.io/badge/Platform-Amazon-blue)](https://www.pangolinfo.com/)
[![Formats](https://img.shields.io/badge/Formats-JSON%20%7C%20HTML%20%7C%20Markdown-orange)](https://docs.pangolinfo.com/en-index)

## Plans and contact

- Sign up at [tool.pangolinfo.com](https://tool.pangolinfo.com/) to get the Free plan with 60 credits every month.
- Enterprise users can scan the QR code to contact us for a demo or a customized test.

<img src="assets/whatsapp-qr.png" alt="WhatsApp QR code" width="200" />

*Scan the WhatsApp QR code to contact us.*

### Email

- **Technical support**: support@pangolinfo.com
- **Business**: charon@pangolinfo.com
- **Response time**: within 24 hours

### Links

- **Website**: [www.pangolinfo.com](https://www.pangolinfo.com/)
- **Documentation**: [docs.pangolinfo.com/en-index](https://docs.pangolinfo.com/en-index)
- **Sign up / API key**: [tool.pangolinfo.com](https://tool.pangolinfo.com/)

---

## About

**Pangolinfo Scrape API** is built by **PANGOLIN INFO TECH PTE. LTD.** (Singapore, founded in 2022) for cross-border sellers, data providers, and developers.

The API collects **all public Amazon data**. It follows Amazon page changes and returns structured fields such as title, price, discount, availability, description, and reviews, as well as raw HTML or Markdown. You call it with an API token; the parsing logic is maintained on our side.

Output formats: **JSON**, **HTML**, and **Markdown**.

### Highlights

- **Dynamic parsing**: adapters track Amazon DOM changes, so your integration does not have to
- **Success rate**: 99% overall; Sponsored Product (SP) ad capture rate 90%+
- **Amazon coverage**: all public Amazon data, including product detail, keyword search, category listings, seller listings, Best Sellers, and New Releases
- **Formats**: JSON, HTML, and Markdown
- **Low maintenance**: parsing stays up to date without a parser rewrite on your side

## Supported Amazon data

Pangolinfo supports all public Amazon data.

| Page | Fields |
|------|--------|
| **Amazon** product detail, keyword search, category listing, seller listing, Best Sellers, New Releases | ASIN, title, price, rating, reviews, review count, images, sales, seller, description, and other public fields |

The open-source code in this repository is narrower than the hosted API. It parses **Amazon keyword search result HTML** that you already have. Regions in the tree: United States, United Kingdom, Germany, France, and Canada (`parsermgr/amazon`).

---

## Open-source Amazon keyword parser

This repository shares a parser for **Amazon keyword search result pages**. It is meant to show how structured fields are read from that HTML.

**Please note:**

* It parses a **local HTML file**. It does not fetch pages.
* It does **not** include HTTP requests, proxies, or captcha handling.
* It follows the Amazon layout it was written for. Amazon changes that layout. See `parsermgr/parser_test.go` for how to run it.

Fields on each product (`model.AmzProduct`): ASIN, title, price, star, rating, image, sales, Prime, and sponsored. The page result (`model.AmzKeywordModel`) also includes the keyword, page index, and next-page URL.

## Quick start

### 1. Open-source parser

```bash
git clone https://github.com/Pangolin-spg/amazon-scraper-api.git
cd amazon-scraper-api

go mod tidy
go test ./parsermgr/
```

### 2. Scrape API

1. [Sign up](https://tool.pangolinfo.com/) for the Free plan (60 credits every month).
2. Exchange your email and password for a token: `POST https://scrapeapi.pangolinfo.com/api/v1/auth`. See [Authentication](https://docs.pangolinfo.com/en-api-reference/authApi/authApi). [`examples/python/get_amazon_product_sync.py`](examples/python/get_amazon_product_sync.py) is a longer sign-in sample. The snippets below follow the current API reference: `format` is a string, and the task URL is `POST https://scrapeapi.pangolinfo.com/api/v1/scrape`.
3. Submit a task: `POST https://scrapeapi.pangolinfo.com/api/v1/scrape`.

#### Python

```python
import requests
import json

# Token from POST https://scrapeapi.pangolinfo.com/api/v1/auth
TOKEN = "your_token_here"

def get_amazon_product(asin):
    """Fetch Amazon product detail as JSON."""
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

`format` accepts `json`, `rawHtml`, or `markdown`. Other `parserName` values (`amzKeyword`, `amzProductOfCategory`, `amzProductOfSeller`, `amzBestSellers`, `amzNewReleases`) are listed in the [Amazon Scraper API reference](https://docs.pangolinfo.com/en-api-reference/amazonApi/amazonScrapeAPI).

## Simplified parser sketch

The maintained parser is `parsermgr` (it uses `htmlquery`). The sketch below only shows the shape of a keyword-page parse:

```go
// Simplified Amazon keyword-search parser sketch.
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

        // Price from .a-price-whole, rating from .a-icon-alt.
        products = append(products, product)
    })

    return products, nil
}
```

## Feature comparison

| Feature | Open-source template | Free API | Paid API |
|---------|----------------------|----------|----------|
| What you get | Keyword search HTML parser | Amazon public data | Amazon public data |
| Quota | Local parsing, no API credits | Free plan, 60 credits every month | Paid plans |
| Support | Community | Official support | Priority support |
| Customize the parser | Full source access | No | Custom work available |
| Anti-bot | You build it | Included | Included |
| Success rate | You maintain it | 99% overall | 99% overall |

Hosted API Sponsored Product (SP) ad capture rate: **90%+**. Plan sizes are on the [billing page](https://docs.pangolinfo.com/en-help-center/amz-billing-subscription/basePlans).

## Contributing

1. **Fork** this repository
2. **Create** a branch (`git checkout -b feature/AmazingFeature`)
3. **Commit** your change (`git commit -m 'Add some AmazingFeature'`)
4. **Push** (`git push origin feature/AmazingFeature`)
5. **Open** a pull request

### Contributor thanks

- A mention for your GitHub profile
- API credits for contributors
- A direct line to the maintainers
- Priority help on issues you hit

### Useful contributions

- Bug fixes in the keyword parser
- Docs and examples
- Tests
- More Amazon marketplaces or languages in the parser

## Use cases

### Learning

- How HTML parsing works
- Data projects
- Crawler internals, using a local HTML fixture
- Market research on data you already saved

### Personal projects

- Price monitoring
- Product trend analysis
- Product research helpers
- A personal dashboard

### Commercial use

- Competitor research
- Amazon data services
- Ops automation
- Market reports

## Documentation

- **Docs home**: [docs.pangolinfo.com/en-index](https://docs.pangolinfo.com/en-index)
- **Amazon Scraper API**: [amazonScrapeAPI](https://docs.pangolinfo.com/en-api-reference/amazonApi/amazonScrapeAPI)
- **Authentication**: [authApi](https://docs.pangolinfo.com/en-api-reference/authApi/authApi)
- **Plans and billing**: [basePlans](https://docs.pangolinfo.com/en-help-center/amz-billing-subscription/basePlans)
- **Billing FAQ**: [faq](https://docs.pangolinfo.com/en-help-center/amz-billing-subscription/faq)

## Roadmap

- [ ] Finish the Amazon parser docs
- [ ] Docker packaging
- [ ] SDKs in more languages
- [ ] Broader Amazon public-data and marketplace coverage in this repo
- [ ] Parsing improvements
- [ ] Hosted deployment options
- [ ] A visual setup UI

## Star history

[![Star History Chart](https://api.star-history.com/svg?repos=Pangolin-spg/amazon-scraper-api&type=Date)](https://star-history.com/#Pangolin-spg/amazon-scraper-api&Date)

## License

[MIT License](LICENSE).

---

## Get started

1. Star this repository
2. [Sign up](https://tool.pangolinfo.com/) for the Free plan — 60 credits every month
3. For a demo or a customized test, scan the WhatsApp QR code above

---

*If this project helps you, a Star is welcome.*
