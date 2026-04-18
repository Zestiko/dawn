# План односторінкового лендінгу на Dawn 15.4.1

> **Магазин:** zestiko.myshopify.com
> **Тема:** Shopify Dawn 15.4.1 (через GitHub)
> **Workflow:** виключно Git, без правок через Theme Editor

---

## 0. Архітектура: що редагується де

Не весь homepage живе в `templates/index.json`. Перед тим як почати — зафіксуй розподіл відповідальності:

| Що змінюємо | Файл |
|---|---|
| Контент і порядок секцій homepage | `templates/index.json` |
| Header + announcement bar (sticky behavior) | `sections/header-group.json` |
| Footer (mega-menu, payment icons, лінки) | `sections/footer-group.json` |
| Глобальні кольори, типографіка, cart type, badge schemes | `config/settings_data.json` |
| Дефолти схем і нові field-и | `config/settings_schema.json` |

**Ключовий принцип односторінковика:** homepage = product page. Всі CTA ведуть або на `#main-product` (anchor), або одразу в `/cart/{variant_id}:1`, або відкривають AJAX cart drawer. Окрема `/products/[handle]` сторінка існує (її не можна видалити), але всі лінки з homepage її минають.

**Спільне для всіх варіантів — оновлення в `config/settings_data.json`:**

```json
{
  "current": {
    "cart_type": "drawer",
    "show_vendor": false,
    "predictive_search_enabled": false
  }
}
```

`cart_type: "drawer"` — критично, бо `"page"` навігує юзера геть з лендінгу, а `"notification"` — компромісний варіант (підходить для C, не дуже для A/B).

---

## Варіант A — Агресивний (High-Conversion)

### Структура

1. **Announcement bar** (закріплений зверху) — таймер / промокод / free shipping threshold
2. **Sticky header** (`on-scroll-up`) — мінімум: лого + cart icon
3. **Hero (image-banner, full-screen)** — H0 заголовок, підзаголовок з social proof number, два CTA: «Buy now» (anchor на `#main-product`) + «Why us» (anchor на `#social-proof`)
4. **Featured product** — головна секція; всі блоки оптимізовані під швидке рішення: title → price → variant picker (button-style) → buy_buttons з `show_dynamic_checkout: true` (Shop Pay) → коротке description
5. **Social proof (multicolumn)** — 3 короткі відгуки, image_ratio: circle (фото клієнтів)
6. **Email-signup-banner** з urgency hook («Hesitating? Get 10% off.»)
7. **Minimal footer** — тільки policy links + соц мережі

### `templates/index.json`

```json
{
  "sections": {
    "hero": {
      "type": "image-banner",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Limited drop. Selling out fast.",
            "heading_size": "h0"
          }
        },
        "text": {
          "type": "text",
          "settings": {
            "text": "<p>Joined by 10,000+ customers. Free shipping over $50. 30-day returns.</p>",
            "text_style": "subtitle"
          }
        },
        "cta": {
          "type": "buttons",
          "settings": {
            "button_label_1": "Buy now",
            "button_link_1": "#main-product",
            "button_style_secondary_1": false,
            "button_label_2": "Why us",
            "button_link_2": "#social-proof",
            "button_style_secondary_2": true
          }
        }
      },
      "block_order": ["heading", "text", "cta"],
      "settings": {
        "image_overlay_opacity": 50,
        "image_height": "large",
        "adapt_height_first_image": false,
        "desktop_content_position": "middle-center",
        "show_text_box": true,
        "desktop_content_alignment": "center",
        "color_scheme": "scheme-4",
        "mobile_content_alignment": "center",
        "stack_images_on_mobile": false,
        "show_text_below": false
      }
    },
    "main-product": {
      "type": "featured-product",
      "blocks": {
        "title": { "type": "title" },
        "price": { "type": "price" },
        "variant_picker": {
          "type": "variant_picker",
          "settings": { "picker_type": "button" }
        },
        "quantity_selector": { "type": "quantity_selector" },
        "buy_buttons": {
          "type": "buy_buttons",
          "settings": {
            "show_dynamic_checkout": true,
            "show_gift_card_recipient": false
          }
        },
        "description": { "type": "description" },
        "share": {
          "type": "share",
          "settings": { "share_label": "Share" }
        }
      },
      "block_order": [
        "title",
        "price",
        "variant_picker",
        "quantity_selector",
        "buy_buttons",
        "description",
        "share"
      ],
      "settings": {
        "product": "your-product-handle",
        "secondary_background": false,
        "hide_variants": false,
        "enable_video_looping": false,
        "color_scheme": "scheme-1",
        "padding_top": 28,
        "padding_bottom": 28
      }
    },
    "social-proof": {
      "type": "multicolumn",
      "blocks": {
        "review-1": {
          "type": "column",
          "settings": {
            "title": "★★★★★ Sarah K.",
            "text": "<p>\"Best purchase I've made this year. Worth every penny.\"</p>",
            "link_label": "",
            "link": ""
          }
        },
        "review-2": {
          "type": "column",
          "settings": {
            "title": "★★★★★ Mike T.",
            "text": "<p>\"Quality is unreal for the price. Already ordered a second one.\"</p>",
            "link_label": "",
            "link": ""
          }
        },
        "review-3": {
          "type": "column",
          "settings": {
            "title": "★★★★★ Anna L.",
            "text": "<p>\"Fast shipping, premium packaging, product is amazing.\"</p>",
            "link_label": "",
            "link": ""
          }
        }
      },
      "block_order": ["review-1", "review-2", "review-3"],
      "settings": {
        "title": "Loved by 10,000+ customers",
        "heading_size": "h2",
        "image_width": "third",
        "image_ratio": "circle",
        "columns_desktop": 3,
        "column_alignment": "center",
        "background_style": "primary",
        "button_label": "",
        "button_link": "",
        "swipe_on_mobile": true,
        "color_scheme": "scheme-2",
        "columns_mobile": "1",
        "padding_top": 28,
        "padding_bottom": 28
      }
    },
    "urgency-cta": {
      "type": "email-signup-banner",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Hesitating? Get 10% off.",
            "heading_size": "h1"
          }
        },
        "paragraph": {
          "type": "paragraph",
          "settings": {
            "text": "<p>Subscribe and we'll send your discount code instantly.</p>",
            "text_style": "body"
          }
        },
        "email_form": { "type": "email_form" }
      },
      "block_order": ["heading", "paragraph", "email_form"],
      "settings": {
        "image_overlay_opacity": 0,
        "show_background_image": false,
        "image_height": "small",
        "desktop_content_position": "middle-center",
        "show_text_box": true,
        "desktop_content_alignment": "center",
        "color_scheme": "scheme-5",
        "mobile_content_alignment": "center",
        "show_text_below": false
      }
    }
  },
  "order": ["hero", "main-product", "social-proof", "urgency-cta"]
}
```

### Color schemes для варіанту A

| Секція | Schema | Логіка |
|---|---|---|
| Hero | `scheme-4` (чорний / білий текст) | Драматичний контраст, утримує увагу |
| Featured product | `scheme-1` (чистий білий) | Максимум фокусу на товарі, без візуального шуму |
| Social proof | `scheme-2` (light gray) | М'який розділювач |
| Email banner | `scheme-5` (синій / білий) | Action-color, асоціюється з кнопками |

Додатково в `config/settings_data.json`:
```json
"buttons_radius": 0,
"buttons_shadow_opacity": 20,
"buttons_shadow_vertical_offset": 4,
"buttons_shadow_blur": 10
```
Жорсткі кути + помітна тінь = більше кліків (кнопки виглядають «тактильно»).

### Зміни поза `templates/index.json`

**`sections/header-group.json`** — sticky behavior + announcement:
```json
{
  "sections": {
    "announcement-bar": {
      "type": "announcement-bar",
      "blocks": {
        "announcement": {
          "type": "announcement",
          "settings": {
            "text": "🔥 Free shipping on orders over $50 — today only",
            "link": ""
          }
        }
      },
      "block_order": ["announcement"],
      "settings": {
        "color_scheme": "scheme-5",
        "show_line_separator": false
      }
    },
    "header": {
      "type": "header",
      "settings": {
        "logo_position": "middle-left",
        "menu": "main-menu",
        "sticky_header_type": "on-scroll-up",
        "show_line_separator": true,
        "color_scheme": "scheme-1",
        "menu_type_desktop": "dropdown",
        "enable_country_selector": false,
        "enable_language_selector": false
      }
    }
  },
  "order": ["announcement-bar", "header"]
}
```

### Git workflow для A

```bash
# 1. Свіжий main + нова гілка
git checkout main
git pull origin main
git checkout -b feature/landing-variant-a

# 2. Замінюємо файли
#    - templates/index.json
#    - sections/header-group.json
#    - config/settings_data.json (cart_type, buttons_radius, ін.)

# 3. Локальний preview через Shopify CLI
shopify theme dev --store=zestiko.myshopify.com
# Відкриває preview URL, перевіряємо homepage + cart flow

# 4. Theme Check
shopify theme check
# Виправляємо warnings, особливо ParserBlockingScript і UnusedAssign

# 5. Коміт + push
git add templates/index.json sections/header-group.json config/settings_data.json
git commit -m "feat(landing): variant A — high-conversion homepage

- Full-screen hero with dual CTA
- Featured product with dynamic checkout
- Social proof (3 reviews, circular images)
- Email capture with urgency hook
- Sticky header with announcement bar"
git push origin feature/landing-variant-a

# 6. PR + theme preview
# GitHub UI: створюємо PR feature/landing-variant-a → main
# Shopify автоматично створить preview theme при push (якщо налаштовано Auto-deploy)
# Або вручну: Online Store → Themes → Add theme → Connect from GitHub → обираємо гілку

# 7. QA на preview theme + merge
git checkout main
git merge feature/landing-variant-a --no-ff
git push origin main
```

---

## Варіант B — Збалансований (Balanced UX)

### Структура

1. **Standard header** — без sticky behavior, без announcement bar
2. **Hero (image-banner, medium)** — H1, лаконічна підпис, **один** CTA («Shop now» → `#main-product`)
3. **Featured product** — стандартні блоки (title, price, variant picker, buy buttons, description, share); без urgency хаків
4. **Multicolumn (3 переваги)** — Free shipping / Returns / Warranty
5. **Collapsible-content (FAQ)** — 3-4 ключові питання
6. **Standard footer** — повний menu, payment icons, social, email signup

### `templates/index.json`

```json
{
  "sections": {
    "hero": {
      "type": "image-banner",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Designed for everyday excellence.",
            "heading_size": "h1"
          }
        },
        "text": {
          "type": "text",
          "settings": {
            "text": "<p>Premium quality, honest price.</p>",
            "text_style": "body"
          }
        },
        "cta": {
          "type": "buttons",
          "settings": {
            "button_label_1": "Shop now",
            "button_link_1": "#main-product",
            "button_style_secondary_1": false,
            "button_label_2": "",
            "button_link_2": "",
            "button_style_secondary_2": false
          }
        }
      },
      "block_order": ["heading", "text", "cta"],
      "settings": {
        "image_overlay_opacity": 30,
        "image_height": "medium",
        "adapt_height_first_image": false,
        "desktop_content_position": "bottom-center",
        "show_text_box": false,
        "desktop_content_alignment": "center",
        "color_scheme": "scheme-3",
        "mobile_content_alignment": "center",
        "stack_images_on_mobile": false,
        "show_text_below": false
      }
    },
    "main-product": {
      "type": "featured-product",
      "blocks": {
        "title": { "type": "title" },
        "price": { "type": "price" },
        "variant_picker": {
          "type": "variant_picker",
          "settings": { "picker_type": "button" }
        },
        "quantity_selector": { "type": "quantity_selector" },
        "buy_buttons": {
          "type": "buy_buttons",
          "settings": {
            "show_dynamic_checkout": true,
            "show_gift_card_recipient": false
          }
        },
        "description": { "type": "description" },
        "share": {
          "type": "share",
          "settings": { "share_label": "Share" }
        }
      },
      "block_order": [
        "title",
        "price",
        "variant_picker",
        "quantity_selector",
        "buy_buttons",
        "description",
        "share"
      ],
      "settings": {
        "product": "your-product-handle",
        "secondary_background": false,
        "hide_variants": false,
        "color_scheme": "scheme-1",
        "padding_top": 36,
        "padding_bottom": 36
      }
    },
    "benefits": {
      "type": "multicolumn",
      "blocks": {
        "benefit-1": {
          "type": "column",
          "settings": {
            "title": "Free shipping",
            "text": "<p>On all orders over $50.</p>"
          }
        },
        "benefit-2": {
          "type": "column",
          "settings": {
            "title": "30-day returns",
            "text": "<p>Hassle-free, no questions asked.</p>"
          }
        },
        "benefit-3": {
          "type": "column",
          "settings": {
            "title": "Made to last",
            "text": "<p>2-year warranty on everything.</p>"
          }
        }
      },
      "block_order": ["benefit-1", "benefit-2", "benefit-3"],
      "settings": {
        "title": "Why customers stay",
        "heading_size": "h2",
        "image_width": "third",
        "image_ratio": "adapt",
        "columns_desktop": 3,
        "column_alignment": "center",
        "background_style": "none",
        "button_label": "",
        "button_link": "",
        "swipe_on_mobile": false,
        "color_scheme": "scheme-1",
        "columns_mobile": "1",
        "padding_top": 36,
        "padding_bottom": 36
      }
    },
    "faq": {
      "type": "collapsible-content",
      "blocks": {
        "row-1": {
          "type": "collapsible_row",
          "settings": {
            "heading": "How long does shipping take?",
            "row_content": "<p>Orders ship within 24 hours. Domestic delivery: 2–5 business days.</p>",
            "icon": "truck"
          }
        },
        "row-2": {
          "type": "collapsible_row",
          "settings": {
            "heading": "What's your return policy?",
            "row_content": "<p>30 days, free returns. Just email us at hello@yourstore.com.</p>",
            "icon": "return"
          }
        },
        "row-3": {
          "type": "collapsible_row",
          "settings": {
            "heading": "Is there a warranty?",
            "row_content": "<p>Yes, 2-year limited warranty on all products.</p>",
            "icon": "shield"
          }
        }
      },
      "block_order": ["row-1", "row-2", "row-3"],
      "settings": {
        "caption": "",
        "heading": "Frequently asked",
        "heading_size": "h2",
        "layout": "none",
        "open_first_collapsible_row": false,
        "color_scheme": "scheme-1",
        "container_color_scheme": "scheme-2",
        "padding_top": 36,
        "padding_bottom": 36
      }
    }
  },
  "order": ["hero", "main-product", "benefits", "faq"]
}
```

### Color schemes для варіанту B

| Секція | Schema | Логіка |
|---|---|---|
| Hero | `scheme-3` (navy + білий) | Помірний контраст, не «кричить» |
| Featured product | `scheme-1` (білий) | Класика, фокус на товарі |
| Benefits | `scheme-1` (білий) | Без розділювачів — плавний flow |
| FAQ | `scheme-1` контейнер `scheme-2` | Картки виділяються легким сірим |

Глобально залишаємо стандартний Dawn-тон:
```json
"buttons_radius": 0,
"buttons_shadow_opacity": 0,
"card_border_thickness": 0,
"card_shadow_opacity": 0
```

### Зміни поза `templates/index.json`

**`sections/header-group.json`** — без announcement bar:
```json
{
  "sections": {
    "header": {
      "type": "header",
      "settings": {
        "logo_position": "middle-left",
        "menu": "main-menu",
        "sticky_header_type": "none",
        "show_line_separator": true,
        "color_scheme": "scheme-1"
      }
    }
  },
  "order": ["header"]
}
```

`cart_type` можна залишити `"notification"` — для UX-балансу не треба перебивати поточну сторінку drawer-ом, маленький toast достатній.

### Git workflow для B

```bash
git checkout main && git pull
git checkout -b feature/landing-variant-b

# Редагуємо: templates/index.json, sections/header-group.json
# (settings_data.json — мінімальні зміни)

shopify theme dev --store=zestiko.myshopify.com
shopify theme check

git add templates/index.json sections/header-group.json
git commit -m "feat(landing): variant B — balanced UX homepage

- Medium hero with single CTA
- Standard featured-product configuration
- 3-column benefits block
- 3-row FAQ via collapsible-content"
git push origin feature/landing-variant-b

# PR → preview theme → merge
```

---

## Варіант C — Інформативний (Content-First)

### Структура

1. **Simple header** — лого + menu, без sticky
2. **Hero (image-banner, medium)** — описовий H1, без CTA (читач має проскролити)
3. **Rich-text intro** — короткий бренд-сторі (2-3 речення)
4. **Featured product** — розгорнутий: vendor → title → caption (subtitle) → price → variant_picker → buy_buttons → description → 4× collapsible_tab (Materials / Shipping / Dimensions / Care) → share
5. **Image-with-text #1** — детальний benefit / feature (image_first)
6. **Image-with-text #2** — другий benefit (text_first, інша color scheme)
7. **Multicolumn** — технічні specs у трьох колонках
8. **Collapsible-content** — розгорнутий FAQ (5 питань)
9. **Standard footer** — повний

### `templates/index.json`

```json
{
  "sections": {
    "hero": {
      "type": "image-banner",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Meet [Product Name]",
            "heading_size": "h1"
          }
        },
        "text": {
          "type": "text",
          "settings": {
            "text": "<p>The story, the craft, and why it matters.</p>",
            "text_style": "subtitle"
          }
        }
      },
      "block_order": ["heading", "text"],
      "settings": {
        "image_overlay_opacity": 20,
        "image_height": "medium",
        "desktop_content_position": "middle-center",
        "show_text_box": false,
        "desktop_content_alignment": "center",
        "color_scheme": "scheme-3",
        "mobile_content_alignment": "center",
        "stack_images_on_mobile": false,
        "show_text_below": false
      }
    },
    "intro": {
      "type": "rich-text",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "A short story behind the product",
            "heading_size": "h2"
          }
        },
        "text": {
          "type": "text",
          "settings": {
            "text": "<p>We started this in [year] because [reason]. Every detail — from materials to packaging — was chosen with intention.</p>"
          }
        }
      },
      "block_order": ["heading", "text"],
      "settings": {
        "color_scheme": "scheme-1",
        "full_width": false,
        "padding_top": 56,
        "padding_bottom": 28
      }
    },
    "main-product": {
      "type": "featured-product",
      "blocks": {
        "vendor": {
          "type": "text",
          "settings": {
            "text": "{{ product.vendor }}",
            "text_style": "uppercase"
          }
        },
        "title": { "type": "title" },
        "caption": {
          "type": "text",
          "settings": {
            "text": "{{ product.metafields.descriptors.subtitle.value }}",
            "text_style": "subtitle"
          }
        },
        "price": { "type": "price" },
        "variant_picker": {
          "type": "variant_picker",
          "settings": { "picker_type": "button" }
        },
        "quantity_selector": { "type": "quantity_selector" },
        "buy_buttons": {
          "type": "buy_buttons",
          "settings": {
            "show_dynamic_checkout": true,
            "show_gift_card_recipient": false
          }
        },
        "description": { "type": "description" },
        "materials": {
          "type": "collapsible_tab",
          "settings": {
            "heading": "Materials",
            "icon": "leather",
            "content": "<p>Sustainably sourced, 100% [material].</p>",
            "page": ""
          }
        },
        "shipping": {
          "type": "collapsible_tab",
          "settings": {
            "heading": "Shipping & returns",
            "icon": "truck",
            "content": "<p>Ships in 24h. Free returns within 30 days.</p>",
            "page": ""
          }
        },
        "dimensions": {
          "type": "collapsible_tab",
          "settings": {
            "heading": "Dimensions",
            "icon": "ruler",
            "content": "<p>Width: ... Height: ... Weight: ...</p>",
            "page": ""
          }
        },
        "care": {
          "type": "collapsible_tab",
          "settings": {
            "heading": "Care instructions",
            "icon": "heart",
            "content": "<p>Wipe with damp cloth. Avoid direct sunlight.</p>",
            "page": ""
          }
        },
        "share": {
          "type": "share",
          "settings": { "share_label": "Share" }
        }
      },
      "block_order": [
        "vendor",
        "title",
        "caption",
        "price",
        "variant_picker",
        "quantity_selector",
        "buy_buttons",
        "description",
        "materials",
        "shipping",
        "dimensions",
        "care",
        "share"
      ],
      "settings": {
        "product": "your-product-handle",
        "secondary_background": false,
        "hide_variants": false,
        "color_scheme": "scheme-1",
        "padding_top": 36,
        "padding_bottom": 36
      }
    },
    "feature-1": {
      "type": "image-with-text",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Built with intention",
            "heading_size": "h2"
          }
        },
        "text": {
          "type": "text",
          "settings": {
            "text": "<p>Detailed description of the first key feature or benefit. Talk about craftsmanship, materials, or process.</p>",
            "text_style": "body"
          }
        }
      },
      "block_order": ["heading", "text"],
      "settings": {
        "height": "medium",
        "desktop_image_width": "medium",
        "layout": "image_first",
        "desktop_content_position": "middle",
        "desktop_content_alignment": "left",
        "content_layout": "no-overlap",
        "color_scheme": "scheme-1",
        "mobile_content_alignment": "left",
        "padding_top": 36,
        "padding_bottom": 36
      }
    },
    "feature-2": {
      "type": "image-with-text",
      "blocks": {
        "heading": {
          "type": "heading",
          "settings": {
            "heading": "Crafted to last",
            "heading_size": "h2"
          }
        },
        "text": {
          "type": "text",
          "settings": {
            "text": "<p>Description of the second key feature. Reverse layout for visual rhythm.</p>",
            "text_style": "body"
          }
        }
      },
      "block_order": ["heading", "text"],
      "settings": {
        "height": "medium",
        "desktop_image_width": "medium",
        "layout": "text_first",
        "desktop_content_position": "middle",
        "desktop_content_alignment": "left",
        "content_layout": "no-overlap",
        "color_scheme": "scheme-2",
        "mobile_content_alignment": "left",
        "padding_top": 36,
        "padding_bottom": 36
      }
    },
    "specs": {
      "type": "multicolumn",
      "blocks": {
        "spec-1": {
          "type": "column",
          "settings": {
            "title": "Material",
            "text": "<p>100% organic cotton, GOTS certified.</p>"
          }
        },
        "spec-2": {
          "type": "column",
          "settings": {
            "title": "Origin",
            "text": "<p>Designed in [city], made in [country].</p>"
          }
        },
        "spec-3": {
          "type": "column",
          "settings": {
            "title": "Warranty",
            "text": "<p>2-year warranty included.</p>"
          }
        }
      },
      "block_order": ["spec-1", "spec-2", "spec-3"],
      "settings": {
        "title": "Specifications",
        "heading_size": "h2",
        "image_width": "third",
        "image_ratio": "adapt",
        "columns_desktop": 3,
        "column_alignment": "left",
        "background_style": "none",
        "swipe_on_mobile": false,
        "color_scheme": "scheme-1",
        "columns_mobile": "1",
        "padding_top": 36,
        "padding_bottom": 36
      }
    },
    "faq": {
      "type": "collapsible-content",
      "blocks": {
        "row-1": {
          "type": "collapsible_row",
          "settings": {
            "heading": "How is it made?",
            "row_content": "<p>Each piece is hand-finished by [process description].</p>",
            "icon": "leather"
          }
        },
        "row-2": {
          "type": "collapsible_row",
          "settings": {
            "heading": "Is it sustainable?",
            "row_content": "<p>Yes — sourced from certified suppliers and shipped in plastic-free packaging.</p>",
            "icon": "plant"
          }
        },
        "row-3": {
          "type": "collapsible_row",
          "settings": {
            "heading": "How do I care for it?",
            "row_content": "<p>See our care guide above. TL;DR: gentle cleaning, no harsh chemicals.</p>",
            "icon": "heart"
          }
        },
        "row-4": {
          "type": "collapsible_row",
          "settings": {
            "heading": "Shipping & returns",
            "row_content": "<p>24h dispatch, 2–5 day delivery, 30-day free returns.</p>",
            "icon": "truck"
          }
        },
        "row-5": {
          "type": "collapsible_row",
          "settings": {
            "heading": "What's covered by warranty?",
            "row_content": "<p>Manufacturing defects, 2 years from purchase date.</p>",
            "icon": "shield"
          }
        }
      },
      "block_order": ["row-1", "row-2", "row-3", "row-4", "row-5"],
      "settings": {
        "caption": "",
        "heading": "Questions, answered",
        "heading_size": "h2",
        "layout": "none",
        "open_first_collapsible_row": false,
        "color_scheme": "scheme-1",
        "container_color_scheme": "scheme-2",
        "padding_top": 36,
        "padding_bottom": 36
      }
    }
  },
  "order": [
    "hero",
    "intro",
    "main-product",
    "feature-1",
    "feature-2",
    "specs",
    "faq"
  ]
}
```

### Color schemes для варіанту C

| Секція | Schema | Логіка |
|---|---|---|
| Hero | `scheme-3` (navy) | М'який вступ, не overwhelming |
| Intro (rich-text) | `scheme-1` | Чистий фон для читання |
| Featured product | `scheme-1` | Стандарт |
| Image-with-text #1 | `scheme-1` | |
| Image-with-text #2 | `scheme-2` | Альтернація для візуального ритму |
| Specs | `scheme-1` | |
| FAQ | `scheme-1` контейнер `scheme-2` | |

Глобально — підняти типографіку для readability:
```json
"body_scale": 110,
"heading_scale": 110,
"page_width": 1200
```

### Зміни поза `templates/index.json`

**`sections/header-group.json`** — простий, без sticky, без оголошень:
```json
{
  "sections": {
    "header": {
      "type": "header",
      "settings": {
        "logo_position": "top-center",
        "menu": "main-menu",
        "sticky_header_type": "none",
        "show_line_separator": true,
        "color_scheme": "scheme-1"
      }
    }
  },
  "order": ["header"]
}
```

`cart_type: "notification"` — для контентного флоу не варто переривати читання drawer-ом.

### Git workflow для C

```bash
git checkout main && git pull
git checkout -b feature/landing-variant-c

# Редагуємо більше файлів через додаткові collapsible_tab + image-with-text
# - templates/index.json
# - sections/header-group.json
# - config/settings_data.json (body_scale, heading_scale)

shopify theme dev --store=zestiko.myshopify.com
shopify theme check

# Перевір окремо: image-with-text потребує реальних зображень.
# Завантаж їх через Files API або в Theme Editor → Files
# (це єдиний крок, де Theme Editor допустимий — для медіа)

git add templates/index.json sections/header-group.json config/settings_data.json
git commit -m "feat(landing): variant C — content-first homepage

- Descriptive hero (no CTA, scroll-driven)
- Brand intro (rich-text)
- Expanded featured-product with 4 collapsible tabs
- 2× image-with-text feature blocks (alternating layout)
- Specs (3-column)
- 5-row FAQ
- Bumped body/heading scale to 110% for readability"
git push origin feature/landing-variant-c

# PR → preview → merge
```

---

## Порівняльна таблиця

| Параметр | A — Aggressive | B — Balanced | C — Informative |
|---|---|---|---|
| Кількість секцій | 4 | 4 | 7 |
| Кількість CTA | 3+ | 1 | 0 (scroll-driven) |
| Sticky header | ✅ on-scroll-up | ❌ none | ❌ none |
| Announcement bar | ✅ | ❌ | ❌ |
| `cart_type` | `drawer` | `notification` | `notification` |
| Featured-product blocks | 7 (мінімум) | 7 | 13 (full) |
| Hero color scheme | `scheme-4` (чорний) | `scheme-3` (navy) | `scheme-3` (navy) |
| Очікуваний LCP | <2.5s | <2.5s | 2.5–3s (більше зображень) |
| Найкраще для | Імпульсивний продукт, низький AOV | Універсальний D2C | Преміум, high-AOV, складний продукт |

---

## Спільні правила (для всіх варіантів)

1. **Замінити `your-product-handle`** в `featured-product` settings на реальний handle товару (формат як у URL: `my-product-name`).
2. **Інші templates не чіпаємо** — `templates/product.json` має лишитись робочим (Shopify все одно роутить юзерів туди по `/products/[handle]`, навіть якщо ми з homepage не лінкуємо).
3. **Додати redirect** (опціонально, через Online Store → Navigation → URL Redirects) з `/products/[handle]` на `/` — якщо хочемо повністю перевести трафік на лендінг. Це робиться через Shopify Admin, не через Git, бо redirects не зберігаються в темі.
4. **`shopify theme check`** перед кожним push — особливо стежимо за warning'ами `MissingTemplate`, `UnusedAssign`, `ParserBlockingScript`.
5. **GitHub Actions** (`/.github/workflows/ci.yml`) уже налаштовані в Dawn — при кожному PR прогоняється Theme Check + Lighthouse CI. Не мерджимо PR з failing checks.
6. **A/B-тестування варіантів** — створи три preview themes (одна на гілку), дай їм різні `?preview_theme_id=` посилання і ганяй трафік через Shopify Audiences чи стороній split-test app.

---

## Чек-лист перед мерджем у `main`

- [ ] `featured-product` показує реальний товар (handle коректний)
- [ ] Всі CTA працюють (anchor links прокручують, buy buttons ведуть у checkout)
- [ ] Cart drawer відкривається без перезавантаження сторінки (для A)
- [ ] Mobile preview перевірений (375px, 414px viewports)
- [ ] Lighthouse Performance ≥ 80 на mobile
- [ ] `shopify theme check` без errors
- [ ] Перевірені locales — якщо магазин мультимовний, додати переклади в `locales/*.json`
- [ ] Зображення оптимізовані (WebP, ≤200KB для hero)
