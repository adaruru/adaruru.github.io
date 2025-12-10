---
title: Getting Started with Presentations
date: 2025-11-26T14:50:00+08:00
description: How to embed presentations in this site.
menu:
  sidebar:
    name: Getting Started
    identifier: presentations-getting-started
    parent: presentations
    weight: 10
tags: ["Presentations", "Guide"]
categories: ["Presentations"]
---

This page demonstrates how to embed presentations and slides.

## Supported Methods

### 1. Embedded PDF

Use the embed-pdf shortcode to display PDF slides:

```
{{< embed-pdf src="/files/your-presentation.pdf" >}}
```

### 2. Video Presentations

Embed recorded presentations using the video shortcode:

```
{{< video src="/videos/your-presentation.mp4" >}}
```

### 3. Online Slides (iframe)

Embed slides from platforms like Google Slides, SlideShare, or Speaker Deck:

```html
<iframe src="https://docs.google.com/presentation/d/e/YOUR_ID/embed" 
        frameborder="0" 
        width="960" 
        height="569" 
        allowfullscreen="true" 
        mozallowfullscreen="true" 
        webkitallowfullscreen="true">
</iframe>
```

### 4. Mermaid Diagrams

For technical diagrams in presentations:

```
{{< mermaid >}}
graph LR
    A[Start] --> B[Process]
    B --> C[End]
{{< /mermaid >}}
```
