# PWA Icon Generation Guide

## Required Icons for `/apps/web/public/icons/`

Your PWA needs app icons in multiple sizes. Generate them from your Quvanti logo:

### Option 1: Use PWA Asset Generator (Automated)
```bash
npx @vite-pwa/assets-generator --preset minimal public/logo.png
```

### Option 2: Manual Generation (Photoshop, Figma, Canva)
Create these exact sizes and save as PNG files:

#### Required Sizes:
- `icon-72x72.png` (72x72px)
- `icon-96x96.png` (96x96px)
- `icon-128x128.png` (128x128px)
- `icon-144x144.png` (144x144px)
- `icon-152x152.png` (152x152px)
- `icon-192x192.png` (192x192px) ⭐ Most important
- `icon-384x384.png` (384x384px)
- `icon-512x512.png` (512x512px) ⭐ Most important

### Design Guidelines:
1. **Background**: Use your brand color (#06B6D4 cyan) or dark (#0A0E14)
2. **Logo**: Center your "Q" logo or full wordmark
3. **Padding**: Leave 10% padding around edges (safe zone)
4. **Corners**: Keep corners sharp (not rounded) - OS will handle rounding
5. **Format**: PNG with transparency OR solid background

### Quick Canva Template:
1. Go to Canva.com
2. Create custom size: 512x512px
3. Add your logo centered
4. Background: Gradient from #06B6D4 to #7C3AED
5. Download as PNG
6. Use online resizer: https://imageresizer.com to create all sizes

### Shortcut Icons (Optional but Recommended):
Create these for app shortcuts:
- `dashboard-shortcut.png` (96x96px) - Chart icon
- `agent-shortcut.png` (96x96px) - Robot/AI icon  
- `signals-shortcut.png` (96x96px) - Lightning bolt icon

### Screenshots (Optional - Improves Install UX):
Save to `/apps/web/public/screenshots/`:
- `desktop-dashboard.png` (1920x1080px) - Wide screenshot
- `mobile-agents.png` (390x844px) - Narrow/mobile screenshot

Take actual screenshots of your app in action!

---

## Quick Icon Generation (5 minutes):

### Method 1: From Text Logo
If you don't have a logo, create a text-based icon:

1. Go to https://favicon.io/favicon-generator/
2. Text: "Q" (or "QV")
3. Background: #06B6D4 (cyan)
4. Font: Bold, modern (Inter/Roboto)
5. Download all sizes
6. Rename to match required filenames above

### Method 2: From Emoji 🚀
1. Screenshot a large emoji: 🤖 or 📈 or ⚡
2. Crop to square
3. Resize to all required sizes
4. Works surprisingly well for testing!

---

## Testing Your Icons

After generating, test locally:
```bash
# Start your dev server
npm run dev

# Open in Chrome
# Right-click → Inspect → Application tab → Manifest
# Check if all icons load correctly
```

### iOS Safari Test:
1. Open your app on iPhone Safari
2. Tap Share → Add to Home Screen
3. Check if icon looks good

### Android Chrome Test:
1. Open your app in Chrome
2. Menu → Install App
3. Check home screen icon

---

## Current Placeholder

Right now, your `/apps/web/public/icons/` folder is empty.

**TEMPORARY SOLUTION**: Your favicon from uploadcare is being used.
But for PWA to work properly, you MUST add proper icons.

**ACTION REQUIRED**: 
1. Generate icons using one of the methods above
2. Place in `/apps/web/public/icons/` folder
3. Restart your dev server
4. Test install prompt

---

## Pro Tip: Adaptive Icons (Android)

For best Android experience, create a maskable icon:
- Keep important content in center 80% of canvas
- Outer 20% can be cropped by OS into various shapes
- Background must be solid (no transparency)

---

You're 90% done with PWA setup! Just add these icons and you'll have a production-ready installable app! 🎉
