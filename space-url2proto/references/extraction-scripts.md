# DOM & Style Extraction Scripts

These JavaScript snippets are designed to be run via `mcp__Claude_in_Chrome__javascript_tool` on the target page. Use them to extract the visual design data needed to recreate the page.

## Table of Contents

1. [Design Tokens Extraction](#design-tokens-extraction) — Colors, fonts, spacings
2. [Section Structure Extraction](#section-structure-extraction) — Major page sections and their layout
3. [Component Detail Extraction](#component-detail-extraction) — Deep-dive into specific elements
4. [Responsive Breakpoints](#responsive-breakpoints) — Media query detection

---

## Design Tokens Extraction

Collects all unique colors, fonts, font sizes, and CSS custom properties from the page.

```javascript
(() => {
  const elements = document.querySelectorAll('*');
  const colors = new Map();
  const bgColors = new Map();
  const fonts = new Map();
  const fontSizes = new Map();
  const fontWeights = new Map();
  const borderRadii = new Map();
  const shadows = new Set();

  elements.forEach(el => {
    const s = getComputedStyle(el);
    const tag = el.tagName.toLowerCase();

    // Track colors with their usage context
    if (s.color !== 'rgba(0, 0, 0, 0)' && s.color !== 'transparent') {
      colors.set(s.color, (colors.get(s.color) || 0) + 1);
    }
    if (s.backgroundColor !== 'rgba(0, 0, 0, 0)' && s.backgroundColor !== 'transparent') {
      bgColors.set(s.backgroundColor, (bgColors.get(s.backgroundColor) || 0) + 1);
    }
    fonts.set(s.fontFamily, (fonts.get(s.fontFamily) || 0) + 1);
    fontSizes.set(s.fontSize, (fontSizes.get(s.fontSize) || 0) + 1);
    fontWeights.set(s.fontWeight, (fontWeights.get(s.fontWeight) || 0) + 1);
    if (s.borderRadius !== '0px') {
      borderRadii.set(s.borderRadius, (borderRadii.get(s.borderRadius) || 0) + 1);
    }
    if (s.boxShadow !== 'none') {
      shadows.add(s.boxShadow);
    }
  });

  // Get CSS custom properties
  const cssVars = {};
  for (const sheet of document.styleSheets) {
    try {
      for (const rule of sheet.cssRules) {
        if (rule.selectorText === ':root' || rule.selectorText === ':root, :host') {
          for (const prop of rule.style) {
            if (prop.startsWith('--')) {
              cssVars[prop] = rule.style.getPropertyValue(prop).trim();
            }
          }
        }
      }
    } catch(e) {}
  }

  const sortByCount = (map) => [...map.entries()].sort((a, b) => b[1] - a[1]).map(([k, v]) => ({ value: k, count: v }));

  return JSON.stringify({
    textColors: sortByCount(colors).slice(0, 20),
    backgroundColors: sortByCount(bgColors).slice(0, 20),
    fontFamilies: sortByCount(fonts).slice(0, 10),
    fontSizes: sortByCount(fontSizes).slice(0, 15),
    fontWeights: sortByCount(fontWeights),
    borderRadii: sortByCount(borderRadii).slice(0, 10),
    boxShadows: [...shadows],
    cssCustomProperties: cssVars,
    viewport: { width: window.innerWidth, height: window.innerHeight }
  }, null, 2);
})();
```

## Section Structure Extraction

Identifies major page sections and extracts their layout properties.

```javascript
(() => {
  // Find major sections: direct children of body or main, plus header/footer/nav
  const candidates = [
    ...document.querySelectorAll('body > *'),
    ...document.querySelectorAll('main > *'),
    ...document.querySelectorAll('header, footer, nav, [role="banner"], [role="navigation"], [role="main"], [role="contentinfo"]')
  ];

  const seen = new Set();
  const sections = [];

  candidates.forEach(el => {
    if (seen.has(el) || el.offsetHeight < 20) return;
    seen.add(el);

    const s = getComputedStyle(el);
    const rect = el.getBoundingClientRect();

    sections.push({
      tag: el.tagName.toLowerCase(),
      id: el.id || null,
      classes: el.className ? el.className.toString().substring(0, 200) : null,
      role: el.getAttribute('role'),
      dimensions: {
        width: Math.round(rect.width),
        height: Math.round(rect.height),
        top: Math.round(rect.top + window.scrollY)
      },
      layout: {
        display: s.display,
        flexDirection: s.flexDirection,
        justifyContent: s.justifyContent,
        alignItems: s.alignItems,
        gridTemplateColumns: s.gridTemplateColumns !== 'none' ? s.gridTemplateColumns : null,
        gap: s.gap !== 'normal' ? s.gap : null,
        padding: s.padding,
        margin: s.margin
      },
      style: {
        backgroundColor: s.backgroundColor,
        color: s.color,
        maxWidth: s.maxWidth,
        overflow: s.overflow
      },
      childCount: el.children.length,
      textPreview: el.textContent.trim().substring(0, 100)
    });
  });

  return JSON.stringify(sections, null, 2);
})();
```

## Component Detail Extraction

For a specific selector, extracts detailed info about the element and its children. Call this on individual components you want to recreate precisely.

```javascript
// Replace SELECTOR with the target (e.g., 'header', '.hero-section', '#main-nav')
((selector) => {
  const el = document.querySelector(selector);
  if (!el) return JSON.stringify({ error: `No element found for: ${selector}` });

  const extractStyles = (element) => {
    const s = getComputedStyle(element);
    return {
      tag: element.tagName.toLowerCase(),
      classes: element.className ? element.className.toString().substring(0, 200) : null,
      text: element.childNodes.length === 1 && element.childNodes[0].nodeType === 3
        ? element.textContent.trim().substring(0, 100)
        : null,
      href: element.href || null,
      src: element.src || null,
      alt: element.alt || null,
      styles: {
        display: s.display,
        position: s.position,
        width: s.width,
        height: s.height,
        padding: s.padding,
        margin: s.margin,
        fontSize: s.fontSize,
        fontWeight: s.fontWeight,
        fontFamily: s.fontFamily,
        lineHeight: s.lineHeight,
        letterSpacing: s.letterSpacing,
        color: s.color,
        backgroundColor: s.backgroundColor,
        borderRadius: s.borderRadius,
        border: s.border,
        boxShadow: s.boxShadow !== 'none' ? s.boxShadow : null,
        textAlign: s.textAlign,
        textDecoration: s.textDecoration,
        textTransform: s.textTransform,
        opacity: s.opacity !== '1' ? s.opacity : null,
        transform: s.transform !== 'none' ? s.transform : null,
        transition: s.transition !== 'all 0s ease 0s' ? s.transition : null,
        flexDirection: s.display.includes('flex') ? s.flexDirection : null,
        justifyContent: s.display.includes('flex') ? s.justifyContent : null,
        alignItems: s.display.includes('flex') ? s.alignItems : null,
        gap: s.gap !== 'normal' ? s.gap : null
      },
      children: [...element.children].slice(0, 20).map(child => extractStyles(child))
    };
  };

  return JSON.stringify(extractStyles(el), null, 2);
})('SELECTOR');
```

## Responsive Breakpoints

Detects media query breakpoints defined in the page's stylesheets.

```javascript
(() => {
  const breakpoints = new Set();

  for (const sheet of document.styleSheets) {
    try {
      const scanRules = (rules) => {
        for (const rule of rules) {
          if (rule instanceof CSSMediaRule) {
            const match = rule.conditionText.match(/(\d+)px/g);
            if (match) match.forEach(bp => breakpoints.add(parseInt(bp)));
            scanRules(rule.cssRules);
          }
        }
      };
      scanRules(sheet.cssRules);
    } catch(e) {}
  }

  return JSON.stringify({
    breakpoints: [...breakpoints].sort((a, b) => a - b),
    currentWidth: window.innerWidth
  }, null, 2);
})();
```
