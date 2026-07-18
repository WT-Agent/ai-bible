# NetTeng Infinite AI Daily Bible Verses

## Project Introduction

**NetTeng Infinite AI Daily Bible Verses** is an interactive spiritual evaluation and prayer micro-application inspired by the structured data style of nomads.com (Nomad List). Built on Vue 3 + Vite + Vanilla CSS, the application allows users to rate their current spiritual metrics (Faith, Peace, Love, Wisdom, and Trial) and share their daily prayer requests. The AI then calculates its own consensus rating, displays a comparison dashboard (Self-Rating vs. AI Consensus), and generates a detailed, engaging scripture meditation in Markdown format based on the selected narrative style (such as Apostolic Meditations, 9-Figure Theological roundtable, or Cyber Prayer). A responsive Virtual Prayer Candle is built-in to support 528Hz sanctuary bell chime sound synthesis and grace accumulation.

### Key Features
- **Spiritual Rating Sliders**: Interactive 1-to-5 star sliders for Faith, Peace, Love, Wisdom, and Trial.
- **Virtual Prayer Candle**: Local sound synthesis using the Web Audio API (528Hz Solfeggio bell chime). Clicking the candle increments the grace count and triggers a floating "Grace +1" animation.
- **AI Consensus & Comparison Dashboard**: Automatic extraction of AI ratings from generated text to render side-by-side progress bars.
- **Scripture Card Style History**: Local persistence of generated prayers and grace counts in the browser, displayed as beautiful holy cards with reload, single deletion, and clear all functions.
- **Floating Share Button**: A sleek glassmorphism share button at the top-right corner to invoke WeChat moments sharing guidance.
- **Side-by-Side QR Codes**: Parallel WeChat and Alipay payment codes in the donation section, and WeChat and DingTalk codes in the Contact Us modal.
- **Adaptive Modal Views**: Terms and privacy modals support vertical scrolling with max-height limits, while QR code modals adjust height automatically to prevent nested scrollbars.

## Quick Start

### 1. Clone the Project
```bash
git clone https://github.com/WT-Agent/ai-bible.git
cd ai-bible
```

### 2. Install Dependencies
This project enforces pnpm as the package manager:
```bash
pnpm install
```

### 3. Local Environment Configuration
Copy and configure the environment variables:
```bash
cp .env.example .env
```
Fill in your API key in the `.env` file:
- `DEEPSEEK_API_KEY`: Your DeepSeek developer API key (used for text generation tasks)

### 4. Development & Build
Start the local development server (with reverse proxy support to prevent API key leaks):
```bash
pnpm dev
```
Build static production assets (outputs to the `dist/` directory, suitable for Vercel, GitHub Pages, or CDN bucket hosting):
```bash
pnpm build
```

## Contact Us

If you have any questions, suggestions, or business cooperation proposals during use, feel free to contact us via WeChat or DingTalk.

## Donation Support

If this project helps you, feel free to support the author. Your support is the driving force for continuous maintenance and optimization.

| WeChat Pay | Alipay |
| :---: | :---: |
| <img src="asset/tenpay.png" width="180" alt="WeChat Pay" /> | <img src="asset/alipay.png" width="180" alt="Alipay" /> |

## License

This project is licensed under the MIT License.

Copyright (c) 2026. All rights reserved.
