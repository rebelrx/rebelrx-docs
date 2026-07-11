# RebelRx Guides

Practical guides for Linux, self-hosting, privacy, homelab infrastructure, and evidence-based health education.

[![Live Site](https://img.shields.io/badge/docs-live-brightgreen)](https://docs.rebelrx.tech)
[![English](https://img.shields.io/badge/language-English-blue)](https://docs.rebelrx.tech)
[![Russian](https://img.shields.io/badge/language-Russian-blue)](https://docs.rebelrx.tech/ru/)
[![Spanish](https://img.shields.io/badge/language-Spanish-blue)](https://docs.rebelrx.tech/es/)
[![Chinese](https://img.shields.io/badge/language-Chinese-blue)](https://docs.rebelrx.tech/zh/)

## 🚀 Live Site

👉 <https://docs.rebelrx.tech>

## 🌐 Available Languages

- 🇺🇸 English
- 🇷🇺 Russian: <https://docs.rebelrx.tech/ru/>
- 🇪🇸 Spanish: <https://docs.rebelrx.tech/es/>
- 🇨🇳 Chinese: <https://docs.rebelrx.tech/zh/>

## Topics

- **Linux** — Why Linux, Artix desktop install, Devuan server install
- **Homelab** — Docker Compose, Nginx Proxy Manager, Tailscale VPN, NAS mounting, backup & recovery
- **Privacy** — Why privacy matters, app recommendations, mobile privacy, migration guide
- **Health** — Evidence-aware health education and the Non-Toxic Grocery Guide

## 🛠️ Building Locally

```bash
pip install mkdocs-material mkdocs-static-i18n
mkdocs serve        # live preview at http://127.0.0.1:8000
mkdocs build        # static output to site/
```

Pre-commit hooks (markdownlint, codespell, strict build) run automatically on
commit if you have [pre-commit](https://pre-commit.com/) installed:

```bash
pip install pre-commit
pre-commit install
```

## About

This repository contains the source for the RebelRx Guides documentation site, built with MkDocs Material.
