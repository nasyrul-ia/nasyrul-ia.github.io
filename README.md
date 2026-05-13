# IA System Documentation Hub

A central documentation portal for the **Intelligence Analytics (IA)** system suite. This repository serves as the landing page and index for all IA-related project documentation.

## Overview

This repo hosts the main index page (`Index.html`) that acts as a navigation hub — providing a single entry point for users to access documentation across all IA modules and subsystems.

## Modules

| # | Module | File |
|---|--------|------|
| 01 | IA Violation System | `Violation_Project_Documentation.html` |
| 02 | IA Monitoring System | `IA_System_Monitoring_Documentation.html` |

> Additional modules will be added here as documentation is developed.

## Live Site

This repository is published via **GitHub Pages** and accessible at:  
[https://nasyrul-ia.github.io](https://nasyrul-ia.github.io)

## Usage

Each module card on the index page links directly to its corresponding HTML documentation file. All documentation files should be placed in the root of this repository.

To add a new module:
1. Add the HTML documentation file to the root of the repo
2. Update the `modules` array in `Index.html` with the new entry
3. Commit and push

## Features

- Dark / Light theme toggle (preference saved in browser)
- Fully responsive — works on desktop and mobile
- No build tools required — pure HTML, CSS and JavaScript

## Structure

```
/
├── Index.html                                  ← Main navigation hub
├── Violation_Project_Documentation.html        ← IA Violation System docs
├── IA_System_Monitoring_Documentation.html     ← IA Monitoring System docs
└── README.md
```