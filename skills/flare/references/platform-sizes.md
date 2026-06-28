# Platform Sizes

Use this reference when the user names a publishing, commerce, or ad platform and asks to create, generate, import, or export a visual for that platform.

Last reviewed: 2026-06-29. Platform specs change. Prefer an explicit user-provided spec, campaign manager requirement, or dynamic MCP/tool response over these defaults.

## How To Use

1. Choose the platform and placement before writing the image prompt, HTML, or frame.
2. Use the recommended canvas size as the intended final delivery size.
3. If the image generation tool supports the exact aspect ratio but not exact pixels, generate the closest ratio, then place it in a Flare frame at the target size for crop/export.
4. If the generation tool supports only square/portrait/landscape presets, choose the closest ratio and write the prompt so important subjects fit within a safe crop.
5. For marketplace main images, compliance matters more than visual drama: neutral product view, no extra props unless allowed, no text overlays, and enough margin.

## Default Decision Rules

- If the user says "cover", "thumbnail", "poster", or "banner" with a named platform, use the primary preset for that platform.
- If the user says "ad" but no placement, prefer the platform's common mobile-first feed placement.
- If the user says "main image" for a marketplace, use a square white-background product image.
- If the platform has both organic and ad placements, do not assume ad policy unless the user says ad, campaign, sponsored, marketplace, or listing.
- When the target is unknown but the user says "social post", default to `1080x1350` (`4:5`) for feed-first content.
- When the user asks for "short video cover", "story", "reels", "shorts", "TikTok", "Douyin", "Kuaishou", or "Snapchat", default to `1080x1920` (`9:16`).
- When the user asks for app store assets, choose the specific store and asset type before generation: screenshot, app icon, Google Play feature graphic, or promotional hero.
- When the user asks for print, packaging, apparel, or physical merchandise, ask for vendor template/specs if production-ready output is required. Use the defaults below only for ideation or common drafts.

## Social And Content Platforms

| Platform / intent                             |         Recommended canvas |         Aspect | Notes                                                                                                                                              |
| --------------------------------------------- | -------------------------: | -------------: | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xiaohongshu / Rednote / 小红书 note cover     |                `1080x1440` |          `3:4` | Best default for feed covers and carousel images. Keep title/key subject inside central safe area. Use `1080x1080` only when user asks for square. |
| Xiaohongshu / Rednote / 小红书 landscape note |                `1440x1080` |          `4:3` | Use for horizontal photography or product scenes.                                                                                                  |
| Instagram feed portrait                       |                `1080x1350` |          `4:5` | Best default for feed reach. Keep text away from edges.                                                                                            |
| Instagram feed square                         |                `1080x1080` |          `1:1` | Use for product grids or when the user says square.                                                                                                |
| Instagram feed landscape                      |                 `1080x566` |       `1.91:1` | Use for wide photos or link-style creative.                                                                                                        |
| Instagram Stories / Reels cover               |                `1080x1920` |         `9:16` | Keep text away from top/bottom UI zones.                                                                                                           |
| Facebook / Meta feed image                    | `1080x1350` or `1080x1080` | `4:5` or `1:1` | Prefer `4:5` for mobile feed unless user asks square.                                                                                              |
| Facebook / Meta link image                    |                 `1200x628` |       `1.91:1` | Use for link preview and website traffic creative.                                                                                                 |
| Facebook / Meta Stories / Reels               |                `1080x1920` |         `9:16` | Mobile full-screen placement.                                                                                                                      |
| TikTok organic/ad image or cover              |                `1080x1920` |         `9:16` | Mobile-first. Leave safe margins for captions, buttons, and profile UI.                                                                            |
| Douyin / 抖音 vertical cover/ad               |                `1080x1920` |         `9:16` | Same creative posture as TikTok unless a Chinese ad spec is provided.                                                                              |
| Kuaishou / 快手 vertical cover/ad             |                `1080x1920` |         `9:16` | Use for short-video style creative.                                                                                                                |
| YouTube thumbnail                             |  `1280x720` or `1920x1080` |         `16:9` | Use `1280x720` for standard thumbnail work, `1920x1080` for higher-quality Flare design/export.                                                    |
| YouTube Shorts cover/frame                    |                `1080x1920` |         `9:16` | Treat as a vertical cover frame; platform support for custom Shorts thumbnails can vary.                                                           |
| Pinterest standard Pin                        |                `1000x1500` |          `2:3` | Best default. Avoid very tall pins unless requested.                                                                                               |
| Pinterest square Pin                          |                `1000x1000` |          `1:1` | Use for product catalog consistency.                                                                                                               |
| LinkedIn feed single image                    |                 `1200x627` |       `1.91:1` | Default for professional feed/link-style creative.                                                                                                 |
| LinkedIn square image                         |                `1080x1080` |          `1:1` | Use for simple product or announcement cards.                                                                                                      |
| LinkedIn vertical image                       |   `720x900` or `1080x1350` |          `4:5` | Use for mobile-first feed posts.                                                                                                                   |
| X / Twitter image post                        |   `1600x900` or `1200x675` |         `16:9` | Good default for timeline visuals.                                                                                                                 |
| X / Twitter square post                       |                `1200x1200` |          `1:1` | Use for product or quote cards.                                                                                                                    |
| X / Twitter website card                      |                 `1200x628` |       `1.91:1` | Use for link/ad card creative.                                                                                                                     |
| Snapchat ad/story                             |                `1080x1920` |         `9:16` | Full-screen vertical; keep UI-safe margins.                                                                                                        |
| WeChat / 微信 Official Account cover          |                  `900x383` |       `2.35:1` | Common article cover default. If the account/editor requires a different crop, follow that spec.                                                   |
| WeChat / 微信 share/social card               |                 `1200x628` |       `1.91:1` | Use for general link-card style sharing creative.                                                                                                  |
| Weibo / 微博 feed image                       | `1080x1080` or `1080x1350` | `1:1` or `4:5` | Prefer square for broad compatibility, portrait for mobile visual impact.                                                                          |
| Bilibili / B 站 video cover                   |                 `1146x717` | approx `16:10` | Common cover working size. Use `1920x1080` if the user explicitly wants 16:9 video artwork.                                                        |

## Commerce And Marketplace Platforms

| Platform / intent                         |         Recommended canvas |   Aspect | Notes                                                                                                                                                                                 |
| ----------------------------------------- | -------------------------: | -------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Amazon main image                         |                `2000x2000` |    `1:1` | White background (`#ffffff`), product only, no text/logo/watermark/borders, product should fill most of the frame while staying uncropped. Use at least `1000x1000` when constrained. |
| Amazon secondary/lifestyle image          | `2000x2000` or `1600x1600` |    `1:1` | Props, context, callouts, or lifestyle scenes are usually secondary images, not the main image.                                                                                       |
| Shopify product image                     |                `2048x2048` |    `1:1` | Strong general product default; keep all product images in a collection visually consistent.                                                                                          |
| Shopify collection/hero image             |  `1920x1080` or `1600x900` |   `16:9` | Use for storefront banners and category hero art.                                                                                                                                     |
| Etsy listing hero                         | `3000x2250` or `2000x1500` |    `4:3` | Good listing-card crop. Keep product centered with margin.                                                                                                                            |
| Etsy square product                       |                `2000x2000` |    `1:1` | Use when the shop grid needs square consistency.                                                                                                                                      |
| eBay main image                           |                `1600x1600` |    `1:1` | White or clean background, no overlays. Use at least `500x500`; larger is better for zoom.                                                                                            |
| Walmart Marketplace main image            |                `2000x2000` |    `1:1` | Product-only white background is safest.                                                                                                                                              |
| Google Shopping / Merchant image          |                `1200x1200` |    `1:1` | Use clean product imagery; minimums vary by category, but larger square assets are safer for reuse.                                                                                   |
| Shopee / Lazada main image                | `1024x1024` or `1200x1200` |    `1:1` | Marketplace grid default; use clean centered product.                                                                                                                                 |
| Taobao / Tmall / 淘宝 / 天猫 main image   |                `1000x1000` |    `1:1` | Common main-image working size. Use marketplace-specific campaign specs when provided.                                                                                                |
| Taobao / Tmall / 淘宝 / 天猫 detail image |         `750px` wide frame | variable | For long detail pages, create stacked frames or a long poster with a fixed `750px` width when the user asks for detail-page content.                                                  |

## Ad And Banner Fallbacks

| Placement                      | Recommended canvas |   Aspect | Notes                                                                |
| ------------------------------ | -----------------: | -------: | -------------------------------------------------------------------- |
| Generic mobile vertical ad     |        `1080x1920` |   `9:16` | TikTok/Reels/Stories/Snapchat style.                                 |
| Generic feed ad                |        `1080x1350` |    `4:5` | Best all-purpose mobile feed default.                                |
| Generic square ad              |        `1080x1080` |    `1:1` | Good cross-platform fallback.                                        |
| Generic link/display ad        |         `1200x628` | `1.91:1` | Good for link cards and responsive display ads.                      |
| Wide banner / hero             |        `1920x1080` |   `16:9` | Use for site hero, YouTube-style art, and presentation-like visuals. |
| Marketplace product main image |        `2000x2000` |    `1:1` | White background, product centered, no overlays.                     |

## Display Ads And IAB Units

| Placement / unit                  | Recommended canvas |   Aspect | Notes                                                                         |
| --------------------------------- | -----------------: | -------: | ----------------------------------------------------------------------------- |
| Medium rectangle                  |          `300x250` |    `6:5` | Common display ad workhorse. Keep copy short and CTA prominent.               |
| Leaderboard                       |           `728x90` | `8.09:1` | Wide horizontal banner. Use simple hierarchy; avoid small body copy.          |
| Mobile banner                     |           `320x50` |   `32:5` | Very small. Usually logo, short offer, and CTA only.                          |
| Large mobile banner               |          `320x100` |   `16:5` | More readable mobile fallback when supported.                                 |
| Wide skyscraper                   |          `160x600` |   `4:15` | Vertical display unit. Stack message, product, CTA.                           |
| Half-page ad                      |          `300x600` |    `1:2` | Strong for product storytelling and high-impact display placements.           |
| Billboard / large leaderboard     |          `970x250` | `3.88:1` | Use for premium desktop display inventory.                                    |
| Responsive display ad source 1:1  |        `1200x1200` |    `1:1` | Useful source asset for Google responsive display ads.                        |
| Responsive display ad source wide |         `1200x628` | `1.91:1` | Useful source asset for Google responsive display ads and link-card creative. |

## App Stores And App Icons

| Platform / asset                     |                        Recommended canvas |   Aspect | Notes                                                                                                                                         |
| ------------------------------------ | ----------------------------------------: | -------: | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Apple App Store app icon             |                               `1024x1024` |    `1:1` | Master app icon. Do not add rounded corners manually; avoid transparency unless the current Apple icon workflow explicitly requires variants. |
| Google Play app icon                 |                                 `512x512` |    `1:1` | 32-bit PNG, sRGB. Upload a full square; Google Play applies masking dynamically. Avoid badges, ranking text, pricing, and misleading claims.  |
| Google Play feature graphic          |                                `1024x500` | `2.05:1` | Store-listing hero. Use concise app value prop and clear brand/app visual.                                                                    |
| Google Play phone screenshot         | `1080x1920`, `1440x2560`, or device ratio | variable | Use real UI screenshots inside device/mockup frames when possible; follow Play Console's device-category requirements.                        |
| Apple App Store iPhone screenshot    |       `1290x2796` or `1320x2868` portrait |   device | Use when the user asks for App Store screenshots and no device is specified. Apple allows multiple device-specific sizes.                     |
| Apple App Store iPad screenshot      |                      `2064x2752` portrait |   device | Use for iPad app listing drafts; confirm current App Store Connect device group before final export.                                          |
| App store promo / ASO creative draft |                               `1290x2796` |   device | Good iPhone-first default for screenshot-style promotional cards.                                                                             |
| PWA / web app icon source            |                                 `512x512` |    `1:1` | Generate a clean source icon that can be resized to `192x192`, `512x512`, maskable, and favicon variants.                                     |

## Web Sharing, SEO, And Launch Assets

| Platform / intent                | Recommended canvas |   Aspect | Notes                                                                                           |
| -------------------------------- | -----------------: | -------: | ----------------------------------------------------------------------------------------------- |
| Open Graph / link preview        |         `1200x630` | `1.91:1` | Default for website sharing cards across Facebook, LinkedIn, Slack, Discord, and many crawlers. |
| X large summary card             |         `1200x628` | `1.91:1` | Use for website card or summary-large-image style previews.                                     |
| GitHub repository social preview |         `1280x640` |    `2:1` | Keep important content inside margins; GitHub recommends at least `640x320`.                    |
| Product Hunt gallery image       |         `1270x760` | `1.67:1` | Use for launch gallery slides and product story frames.                                         |
| Blog cover / article hero        |         `1200x630` | `1.91:1` | Good default when no CMS-specific size is provided.                                             |
| Website hero image               |        `1920x1080` |   `16:9` | Good general hero/landing-page source. Confirm actual site crop before final export.            |
| Website wide banner              |         `1600x600` |    `8:3` | Common marketing-site banner source.                                                            |
| Email newsletter header          |         `1200x600` |    `2:1` | Design at 2x for common 600px email containers; keep file weight low after export.              |
| Email hero / promotion           |         `1200x800` |    `3:2` | Good default for responsive email hero imagery.                                                 |

## Video, Streaming, And Podcast

| Platform / intent                 | Recommended canvas |   Aspect | Notes                                                                                                |
| --------------------------------- | -----------------: | -------: | ---------------------------------------------------------------------------------------------------- |
| YouTube channel banner            |        `2560x1440` |   `16:9` | Keep text/logo in the central safe area because TV, desktop, and mobile crop differently.            |
| YouTube thumbnail                 |         `1280x720` |   `16:9` | Standard thumbnail; already listed above, repeated here for video workflows.                         |
| Twitch profile banner             |         `1200x480` |    `5:2` | Keep key content toward center/left because wide browser windows can stretch/crop.                   |
| Twitch offline screen             |        `1920x1080` |   `16:9` | Use as full-screen "stream offline" artwork.                                                         |
| Twitch panel                      |       `320px` wide | variable | Height depends on content. Keep text large and concise.                                              |
| Stream overlay / scene background |        `1920x1080` |   `16:9` | Base canvas for webcam frames, starting-soon, BRB, and ending screens.                               |
| Podcast show cover                |        `3000x3000` |    `1:1` | Apple Podcasts accepts `1400x1400` to `3000x3000`; largest size is preferred. Use JPG/PNG, no alpha. |
| Podcast episode art               |        `3000x3000` |    `1:1` | Same square format; ensure it reads at tiny sizes.                                                   |
| Spotify album / track cover       |        `3000x3000` |    `1:1` | Use high-res square source; Spotify accepts a broad range but square sRGB artwork is safest.         |
| Spotify Canvas visual             |        `1080x1920` |   `9:16` | Use for vertical looping video/visual source.                                                        |

## Documents And Print

| Deliverable              |                       Recommended canvas |   Aspect | Notes                                                                                                 |
| ------------------------ | ---------------------------------------: | -------: | ----------------------------------------------------------------------------------------------------- |
| Presentation slide 16:9  |                              `1920x1080` |   `16:9` | Best default for slides, decks, pitch covers, and webinar title cards.                                |
| Presentation slide 4:3   |                              `1600x1200` |    `4:3` | Use only when the user asks for legacy projector or 4:3 slides.                                       |
| A4 document/poster       |                   `2480x3508` at 300 DPI |   ISO A4 | Use for print-quality A4. Physical size is 210mm x 297mm.                                             |
| A3 poster                |                   `3508x4961` at 300 DPI |   ISO A3 | Use for print-quality A3.                                                                             |
| A5 flyer                 |                   `1748x2480` at 300 DPI |   ISO A5 | Use for small flyer/handout.                                                                          |
| US Letter document       |                   `2550x3300` at 300 DPI |   Letter | Use for US documents, reports, resumes, and print drafts.                                             |
| Business card, US        |                    `1050x600` at 300 DPI |    `7:4` | 3.5in x 2in. Add bleed if production-ready.                                                           |
| Business card, common EU |                    `1004x650` at 300 DPI |  `85:55` | 85mm x 55mm. Confirm local printer specs.                                                             |
| Postcard                 |                   `1800x1200` at 300 DPI |    `3:2` | 6in x 4in. Add bleed when printing.                                                                   |
| US flyer                 |                   `2550x3300` at 300 DPI |   Letter | Same as US Letter; ask if the printer expects bleed.                                                  |
| Roll-up banner draft     | design in physical units, often 85x200cm | variable | Vendor-specific. Ask for printer template before final production; use this only as concept guidance. |

Print rule: for production print, ask for final physical size, region, printer/vendor template, bleed, color mode, and whether Flare should export PDF or image. Use 300 DPI for close-view print, and add standard bleed of `0.125in` / `3mm` unless the vendor says otherwise.

## Books And Publishing

| Platform / asset              |          Recommended canvas |   Aspect | Notes                                                                                                                       |
| ----------------------------- | --------------------------: | -------: | --------------------------------------------------------------------------------------------------------------------------- |
| Kindle / KDP ebook cover      |                 `1600x2560` |  `1:1.6` | KDP ideal ebook cover size. Keep title/author legible at thumbnail size.                                                    |
| Generic ebook cover           |                 `1600x2560` |  `1:1.6` | Good cross-platform ebook default.                                                                                          |
| KDP paperback/hardcover cover | use KDP calculator/template | variable | Print cover depends on trim size, page count, paper color, and bleed. Do not guess spine width for production-ready output. |
| Audiobook cover               |                 `3000x3000` |    `1:1` | Square cover works well across audiobook and podcast-like directories.                                                      |
| Magazine cover concept        |                 `2480x3508` |   ISO A4 | Use A4 unless the user provides trim size.                                                                                  |

## Packaging And Merchandise

| Deliverable                |      Recommended canvas |   Aspect | Notes                                                                                                       |
| -------------------------- | ----------------------: | -------: | ----------------------------------------------------------------------------------------------------------- |
| Sticker / label concept    |             `2000x2000` |    `1:1` | Use for concept art. Ask for dieline/cutline and bleed before production.                                   |
| Product label, bottle/can  | vendor dieline/template | variable | Dimensions vary by container and printer. Ask for dieline, wrap width, height, bleed, and safe area.        |
| Packaging box artwork      | vendor dieline/template | variable | Do not invent panels/spine/flaps for production. Ask for packaging dieline or create a concept mockup only. |
| T-shirt front print        |             `4500x5400` |    `5:6` | Common print-on-demand source. Confirm supplier print area and garment size.                                |
| Hoodie/front apparel print |             `4500x5400` |    `5:6` | Same as T-shirt source unless supplier template differs.                                                    |
| Mug wrap concept           |             `2700x1050` |   `18:7` | Common 9in x 3.5in wrap at 300 DPI; confirm supplier template for final.                                    |
| Tote bag print concept     |             `3600x3600` |    `1:1` | Common square high-res source. Confirm supplier print area.                                                 |

## Prompting Guidance

For social images:

- Include the platform, aspect ratio, and intended crop in the generation prompt.
- Ask for safe margins when the platform has UI overlays: "keep important text and faces inside the central safe area".
- For text-heavy creative, prefer generating background/product imagery first, then add final text in Flare as editable layers.

For marketplace images:

- Say "clean product photography", "white background", "centered product", "no text", "no watermark", and "no extra props" for main images.
- Do not invent compliance details if the platform or category has stricter rules. If a specific marketplace upload is the goal and the exact spec is missing, ask one short question or use the marketplace main-image default.
- For secondary listing images, allow lifestyle scenes and callouts only when the user asks for them.

## Flare Delivery Pattern

- For generated images, keep the image at the generated file's intrinsic dimensions during MCP upload. Do not pass `width` or `height` to shrink it on insertion just because the target platform has a delivery size.
- If the user needs an exact platform deliverable, create a Flare frame at the target canvas size and place/crop the generated asset into that frame.
- For HTML imports, set the root frame to the target platform size before writing HTML.
- For exports/renders, use the platform canvas size as the render frame size.

## Source Notes

Primary public references checked for representative defaults:

- Meta Ads Guide: https://www.facebook.com/business/ads-guide
- TikTok Ads creative specifications: https://ads.tiktok.com/help/article/video-ads-specifications
- Amazon product image requirements: https://sell.amazon.com/blog/amazon-product-image-requirements
- YouTube thumbnail help: https://support.google.com/youtube/answer/72431
- Pinterest product specs: https://help.pinterest.com/en/business/article/pinterest-product-specs
- LinkedIn ad specs: https://business.linkedin.com/marketing-solutions/success/marketing-terms/ad-specs
- X Ads creative specifications: https://business.x.com/en/help/campaign-setup/creative-ad-specifications.html
- Google Merchant Center image requirements: https://support.google.com/merchants/answer/6324350
- Shopify product media help: https://help.shopify.com/en/manual/products/product-media/product-media-types
- Etsy listing image help: https://help.etsy.com/hc/en-us/articles/115015663347
- eBay picture requirements: https://www.ebay.com/help/selling/listings/creating-managing-listings/adding-pictures-listings?id=4148
- Apple App Store screenshot and icon specifications: https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications
- Apple app icon guidance: https://developer.apple.com/design/human-interface-guidelines/app-icons
- Google Play app icon, feature graphic, and screenshot guidance: https://support.google.com/googleplay/android-developer/answer/9866151
- IAB ad portfolio: https://www.iab.com/newadportfolio/
- Open Graph image guidance: https://developers.facebook.com/docs/sharing/webmasters/images
- GitHub social preview docs: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/customizing-your-repositorys-social-media-preview
- YouTube channel banner help: https://support.google.com/youtube/answer/2972003
- Twitch channel page setup guidance: https://help.twitch.tv/s/article/channel-page-setup
- Apple Podcasts artwork requirements: https://podcasters.apple.com/support/896-artwork-requirements
- KDP cover image guidance: https://kdp.amazon.com/en_US/help/topic/G200645690
- Google responsive display ad specs: https://support.google.com/google-ads/answer/7005917

Some China-platform organic content sizes are industry defaults rather than stable public API contracts; use campaign/editor specs when the user provides them.
