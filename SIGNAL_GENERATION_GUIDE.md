# 🤖 AUTOMATIC SIGNAL GENERATION GUIDE

## ❓ **DO AGENTS AUTOMATICALLY GENERATE SIGNALS?**

### **CURRENT STATE: MANUAL TRIGGER (No Automatic Cron)**

**Right now:**
- ❌ Agents do **NOT** automatically generate signals periodically
- ✅ Signals can be generated **manually** via API call
- ✅ Signal generation uses **REAL technical indicators** (RSI, MACD, Bollinger Bands, Composite)

**Why not automatic?**
- Requires a cron job / scheduled task infrastructure
- Platform doesn't currently have a built-in scheduler
- Would need Vercel Cron, GitHub Actions, or external cron service

---

## **📊 HOW SIGNAL GENERATION WORKS (Technical Details)**

### **1. Signal Generation Endpoint**

Location: `/apps/web/src/app/api/signals/generate/route.js`

**What it does:**
1. Finds all **active agents** in the database
2. For each agent, fetches **real market data** from CoinGecko (50 days)
3. Calculates **real technical indicators** (RSI, MACD, Bollinger Bands)
4. Applies **algorithmic decision logic** (ZERO randomness)
5. Generates BUY/SELL signals if confidence > 40%
6. Saves signals to database with expiration (24 hours)

**Example call:**
```javascript
POST /api/signals/generate
```

**Response:**
```json
{
  "success": true,
  "generated": 12,
  "message": "Generated 12 signals using REAL technical indicators",
  "algorithmic": true,
  "zeroRandomness": true
}
```

---

### **2. Signal Generation Algorithm (REAL Indicators)**

From `/apps/web/src/app/api/signals/generate/route.js`:

```javascript
async function generateSignal(agent, assetPool) {
  for (const asset of assetPool) {
    // Fetch REAL historical data
    const historicalData = await fetchHistoricalData(asset, 50);
    const candles = historicalData.candles;
    const prices = candles.map((c) => c.close);
    const currentPrice = prices[prices.length - 1];
    
    // Calculate REAL indicators
    const rsi = RSI(prices, 14);
    const macd = MACD(prices, 12, 26, 9);
    const bb = BollingerBands(prices, 20, 2);
    const compositeSignal = CompositeSignal(candles);
    
    // ALGORITHMIC DECISION LOGIC (ZERO RANDOMNESS)
    let action = "HOLD";
    let confidence = 0;
    let reasoning = "";
    
    // RSI-based signals
    if (rsi.value < 30) {
      action = "BUY";
      confidence = 60 + (30 - rsi.value);
      reasoning = `RSI oversold at ${rsi.value.toFixed(2)}`;
    } else if (rsi.value > 70) {
      action = "SELL";
      confidence = 60 + (rsi.value - 70);
      reasoning = `RSI overbought at ${rsi.value.toFixed(2)}`;
    }
    
    // MACD confirmation
    if (macd.histogram > 0 && action === "BUY") {
      confidence += 10;
      reasoning += ` MACD bullish`;
    }
    
    // Bollinger Bands confirmation
    if (bb.percentB < 0.2 && action === "BUY") {
      confidence += 8;
      reasoning += ` Price near lower BB`;
    }
    
    // Composite signal weight
    if (compositeSignal.action.includes("STRONG BUY") && action === "BUY") {
      confidence += 12;
    }
    
    // Skip if confidence < 40%
    if (action === "HOLD" || confidence < 40) continue;
    
    // Calculate targets
    const volatility = bb.bandwidth / 100;
    const expectedReturn = confidence * 0.08 * (1 + volatility);
    const targetPrice = action === "BUY" 
      ? currentPrice * (1 + expectedReturn / 100)
      : currentPrice * (1 - expectedReturn / 100);
    const stopLoss = action === "BUY" ? currentPrice * 0.97 : currentPrice * 1.03;
    
    return {
      asset,
      action,
      confidence,
      expected_return: expectedReturn,
      reasoning,
      price_at_signal: currentPrice,
      target_price: targetPrice,
      stop_loss: stopLoss,
      timeframe: confidence > 70 ? "1-3 days" : "3-5 days",
      expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000)
    };
  }
  
  return null; // No strong signals found
}
```

**KEY POINTS:**
- ✅ **ZERO randomness** - all signals are based on real indicator values
- ✅ **Confidence threshold** - only signals with >40% confidence are generated
- ✅ **Multi-indicator confirmation** - RSI + MACD + Bollinger Bands + Composite
- ✅ **Volatility-adjusted targets** - higher volatility = higher expected returns
- ✅ **Realistic risk management** - stop loss at 3% from entry

---

## **🔧 HOW TO SET UP AUTOMATIC SIGNAL GENERATION**

### **OPTION 1: Vercel Cron Jobs (Recommended)**

**Requirements:**
- Vercel Pro plan ($20/month)
- `vercel.json` configuration

**Steps:**

1. **Update `vercel.json`:**

```json
{
  "crons": [
    {
      "path": "/api/signals/generate",
      "schedule": "0 */4 * * *"
    }
  ]
}
```

This runs signal generation **every 4 hours**.

2. **Secure the endpoint:**

In `/apps/web/src/app/api/signals/generate/route.js`:

```javascript
export async function GET(request) {
  // Verify cron secret
  const authHeader = request.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }
  
  // ... rest of signal generation logic
}
```

3. **Add environment variable:**
```bash
CRON_SECRET=your_random_secret_here_12345
```

4. **Deploy:**
```bash
vercel --prod
```

**Result:** Signals auto-generate every 4 hours!

---

### **OPTION 2: GitHub Actions (Free)**

**Requirements:**
- GitHub repository
- GitHub Actions enabled

**Steps:**

1. **Create `.github/workflows/generate-signals.yml`:**

```yaml
name: Generate Trading Signals

on:
  schedule:
    - cron: '0 */4 * * *'  # Every 4 hours
  workflow_dispatch:  # Manual trigger

jobs:
  generate-signals:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Signal Generation
        run: |
          curl -X POST https://your-app-domain.com/api/signals/generate \
            -H "Authorization: Bearer ${{ secrets.CRON_SECRET }}"
```

2. **Add secret to GitHub:**
   - Go to Settings → Secrets → Actions
   - Add `CRON_SECRET` with a random value

3. **Commit and push:**
```bash
git add .github/workflows/generate-signals.yml
git commit -m "Add automatic signal generation"
git push
```

**Result:** Signals auto-generate every 4 hours via GitHub!

---

### **OPTION 3: External Cron Service (easycron.com, cron-job.org)**

**Requirements:**
- Free account on easycron.com or cron-job.org

**Steps:**

1. **Sign up for easycron.com**

2. **Create new cron job:**
   - URL: `https://your-app-domain.com/api/signals/generate`
   - Method: `POST`
   - Schedule: `0 */4 * * *` (every 4 hours)
   - Header: `Authorization: Bearer your_secret_here`

3. **Save and activate**

**Result:** Signals auto-generate every 4 hours!

---

## **🎯 CURRENT MANUAL TRIGGER METHOD**

Since automatic cron isn't set up yet, you can manually trigger signal generation:

### **Method 1: API Call**

```bash
curl -X POST https://your-app-domain.com/api/signals/generate
```

### **Method 2: Browser Console**

```javascript
fetch('/api/signals/generate', { method: 'POST' })
  .then(res => res.json())
  .then(data => console.log(data));
```

### **Method 3: Create Admin Panel Button**

Add a button to `/apps/web/src/app/admin/page.jsx`:

```javascript
<button
  onClick={async () => {
    const res = await fetch('/api/signals/generate', { method: 'POST' });
    const data = await res.json();
    alert(`Generated ${data.generated} signals!`);
  }}
  className="px-6 py-3 bg-purple-600 text-white rounded-lg"
>
  🤖 Generate Signals Now
</button>
```

---

## **📋 SIGNAL LIFECYCLE**

1. **Creation:**
   - Agent generates signal via `/api/signals/generate`
   - Saved to `signals` table with `status = 'active'`
   - Expires in 24 hours

2. **Display:**
   - Users see signals on `/signals` page
   - Filtered by confidence, asset, action

3. **Expiration:**
   - Cleanup job runs: `/api/signals/cleanup`
   - Changes `status` to `'expired'` after 24 hours
   - Should also run on cron (every hour recommended)

4. **Performance Tracking:**
   - Optional: Track if signal hit target price
   - Saved to `signal_performance` table
   - Used for agent trust score

---

## **🚀 RECOMMENDED SETUP FOR PRODUCTION**

### **Cron Schedule:**

```json
{
  "crons": [
    {
      "path": "/api/signals/generate",
      "schedule": "0 */4 * * *",
      "description": "Generate new trading signals every 4 hours"
    },
    {
      "path": "/api/signals/cleanup",
      "schedule": "0 * * * *",
      "description": "Clean up expired signals every hour"
    }
  ]
}
```

### **Why 4 hours?**
- Markets move on multiple timeframes (1H, 4H, 1D)
- 4 hours captures intraday moves without spam
- Signals expire in 24 hours, so 6 batches per day
- Balance between fresh signals and API rate limits

---

## **✅ SUMMARY**

### **Current State:**
| Feature | Status |
|---------|--------|
| **Signal Generation Logic** | ✅ Implemented with REAL indicators |
| **Manual Trigger** | ✅ Works via API POST call |
| **Automatic Cron** | ❌ Not set up (requires Vercel Cron or GitHub Actions) |
| **Signal Cleanup** | ✅ Implemented (manual trigger) |
| **Data Authenticity** | ✅ 100% real CoinGecko data |

### **To Enable Automatic Signals:**
1. Choose Vercel Cron, GitHub Actions, or external cron service
2. Add `CRON_SECRET` environment variable
3. Update `vercel.json` or create `.github/workflows/generate-signals.yml`
4. Deploy and verify

### **Expected Result:**
- ✅ Fresh signals every 4 hours
- ✅ Expired signals cleaned up every hour
- ✅ Zero manual intervention needed
- ✅ Users always see recent, high-confidence signals

---

**Last Updated:** April 22, 2026  
**Signal Generation:** `/apps/web/src/app/api/signals/generate/route.js`  
**Signal Cleanup:** `/apps/web/src/app/api/signals/cleanup/route.js`  
**Indicators:** `/apps/web/src/app/api/utils/indicators.js`
