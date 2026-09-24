# 🧪 COMPREHENSIVE TEST COMMANDS

## TESTING AGENT LIMITS

### Test 1: Free User - 1 Agent Lifetime Limit

```bash
# Scenario: Free user tries to create 2 agents
# Expected: First succeeds, second fails with "1 agent total (lifetime)"

# Create Agent 1
curl -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Agent 1",
    "naturalLanguagePrompt": "Buy Bitcoin when RSI is below 30"
  }'
# Expected:  Success

# Create Agent 2 (SHOULD FAIL)
curl -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Agent 2",
    "naturalLanguagePrompt": "Sell Ethereum when MACD crosses down"
  }'
# Expected:  Error: "Free users can have 1 agent total (lifetime). Upgrade to Pro for 10 agent slots!"
```

### Test 2: PRO User - 10 Agent Slots Limit

```bash
# Scenario: PRO user creates 11 agents
# Expected: First 10 succeed, 11th fails with "delete to make more"

# Loop to create 10 agents
for i in {1..10}; do
  curl -X POST http://localhost:3000/api/agents/create \
    -H "Content-Type: application/json" \
    -d "{
      \"name\": \"PRO Agent $i\",
      \"naturalLanguagePrompt\": \"Strategy $i\"
    }"
done
# Expected: All 10 succeed

# Create 11th agent (SHOULD FAIL)
curl -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "PRO Agent 11",
    "naturalLanguagePrompt": "Extra strategy"
  }'
# Expected:  Error: "You've reached the maximum of 10 agents. Delete an existing agent to create a new one."
```

### Test 3: PRO User Delete & Create Flow

```bash
# Scenario: PRO user at 10/10 agents, deletes 1, creates new one
# Expected: Delete succeeds, creation succeeds

# Delete Agent ID 5
curl -X POST http://localhost:3000/api/agents/delete \
  -H "Content-Type: application/json" \
  -d '{"agentId": 5}'
# Expected: ✅ Success

# Now create 11th agent (SHOULD SUCCEED)
curl -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "PRO Agent 11 (After Delete)",
    "naturalLanguagePrompt": "New strategy"
  }'
# Expected: ✅ Success (now at 10/10 again)
```

---

## TESTING STRATEGY GENERATION LIMITS

### Test 4: Free User - 1 Strategy Per Day

```bash
# Scenario: Free user tries to generate 2 strategies in same day
# Expected: First succeeds, second fails

# Strategy 1
curl -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a BTC momentum strategy"
  }'
# Expected: ✅ Success

# Strategy 2 (SAME DAY - SHOULD FAIL)
curl -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create an ETH scalping strategy"
  }'
# Expected: ❌ Error: "Free users get 1 strategy generation per day. Upgrade to Pro for 15 per day!"
```

### Test 5: PRO User - 15 Strategies Per Day

```bash
# Scenario: PRO user generates 16 strategies in same day
# Expected: First 15 succeed, 16th fails

# Loop to create 15 strategies
for i in {1..15}; do
  curl -X POST http://localhost:3000/api/strategy-lab/parse \
    -H "Content-Type: application/json" \
    -d "{
      \"prompt\": \"Strategy $i for PRO user\"
    }"
  echo "Strategy $i done"
done
# Expected: ✅ All 15 succeed

# Create 16th strategy (SHOULD FAIL)
curl -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Extra strategy (should fail)"
  }'
# Expected: ❌ Error: "You've reached your daily limit of 15 strategy generations. Try again tomorrow."
```

### Test 6: Strategy Limit Reset at Midnight

```bash
# Scenario: Free user generates 1 strategy, waits until next day, generates another
# Expected: Both succeed (different days)

# Day 1 - Strategy 1
curl -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Day 1 strategy"}'
# Expected: ✅ Success

# Wait until midnight or manually change date in database for testing

# Day 2 - Strategy 2
curl -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Day 2 strategy"}'
# Expected: ✅ Success (counter reset)
```

---

## TESTING BACKTEST VALIDATION (REAL ALGORITHMS)

### Test 7: Monte Carlo Simulation Integration

```bash
# Scenario: Run backtest and verify Monte Carlo results appear
# Expected: Backtest returns monteCarlo object with probabilityOfRuin

curl -X POST http://localhost:3000/api/backtest/run \
  -H "Content-Type: application/json" \
  -d '{
    "agentId": 1,
    "startDate": "2023-01-01",
    "endDate": "2024-01-01",
    "initialCapital": 10000
  }'

# Expected Response:
# {
#   "success": true,
#   "backtest": {
#     "total_return": 47.2,
#     "win_rate": 62.8,
#     "validation": {
#       "monteCarlo": {
#         "probabilityOfRuin": 2.3,
#         "expectedValue": 14720,
#         "confidenceIntervals": { ... }
#       },
#       "walkForward": { ... },
#       "overfitAnalysis": { ... }
#     }
#   }
# }
```

### Test 8: Overfit Detection Triggers

```bash
# Scenario: Backtest with unrealistic metrics triggers warnings
# Expected: overfitAnalysis.overfitDetected = true

# Create fake backtest data with Win Rate > 80%
curl -X POST http://localhost:3000/api/backtest/validate \
  -H "Content-Type: application/json" \
  -d '{
    "backtestResults": {
      "winRate": 85,
      "profitFactor": 4.5,
      "totalTrades": 100,
      "sharpeRatio": 3.5,
      "maxDrawdown": 5,
      "totalReturn": 200
    }
  }'

# Expected Response:
# {
#   "validation": {
#     "overfitAnalysis": {
#       "overfitDetected": true,
#       "overfitScore": 70,
#       "warnings": [
#         { "type": "HIGH_WIN_RATE", "message": "Win Rate of 85% exceeds historical norms..." },
#         { "type": "HIGH_PROFIT_FACTOR", "message": "Profit Factor of 4.5 is unrealistic..." }
#       ],
#       "recommendation": "CRITICAL: Run Monte Carlo and Walk-Forward before live deployment."
#     }
#   }
# }
```

---

## TESTING UI/UX

### Test 9: Mobile Dashboard (Visual Test)

1. Open `/dashboard` on mobile (375px width)
2. **Expected:**
   - No annoying banners at top
   - Stats grid stacks vertically
   - Text sizes are readable (10px-14px)
   - No horizontal scroll
   - Clean white space

### Test 10: Landing Page Algorithm Tags (Visual Test)

1. Open `/` (homepage)
2. **Expected:**
   - Algorithm tags visible at top (Monte Carlo, Walk-Forward, Volume Profile, etc.)
   - Tags wrap on mobile
   - No "No BS" text anywhere
   - No brand names (QuantConnect, Trade Ideas, etc.)

### Test 11: Billing Page Updates (Visual Test)

1. Open `/upgrade`
2. **Expected:**
   - ❌ No "Here's What PRO Users Are Making" section
   - ❌ No "Lock In Current Pricing" section
   - ❌ No "$216/day missed opportunities" text
   - ✅ Shows "1 agent lifetime" for FREE
   - ✅ Shows "10 agent slots" for PRO
   - ✅ Shows "1 strategy/day" for FREE
   - ✅ Shows "15 strategies/day" for PRO

### Test 12: "Try Now" Buttons Work (Functional Test)

1. Open `/` (homepage)
2. Scroll to UniqueFeatures section
3. Click "Try it now" on first 5 features
   - **Expected:** Redirects to `/agents/create`
4. Click "Try it now" on last 4 features
   - **Expected:** Redirects to `/strategy-lab`

### Test 13: PRO Dashboard Button (Functional Test)

1. Log in as FREE user
2. Check header navigation
   - **Expected:** No "PRO Dashboard" button

3. Log in as PRO user
4. Check header navigation
   - **Expected:** "PRO Dashboard" button visible between Analytics and Live Signals
   - **Expected:** Clicking redirects to `/pro-dashboard`

---

## TESTING DELETE AGENT BUTTON

### Test 14: Delete Agent Functionality

```bash
# 1. Create an agent
curl -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Delete Agent",
    "naturalLanguagePrompt": "Buy when RSI < 30"
  }'
# Note the returned agent ID

# 2. Delete the agent
curl -X POST http://localhost:3000/api/agents/delete \
  -H "Content-Type: application/json" \
  -d '{"agentId": 123}'  # Replace 123 with actual ID

# 3. Verify deletion - List agents
curl -X GET http://localhost:3000/api/agents/list

# Expected: Agent ID 123 should NOT appear in the list
```

### Test 15: Delete Button UI (Visual Test)

1. Open `/dashboard`
2. Look at agent cards
   - **Expected:** Each card has a red Trash icon button
   - **Expected:** Clicking shows confirmation dialog
   - **Expected:** After confirming, agent disappears
   - **Expected:** Dashboard auto-refreshes

---

## AUTOMATED TEST SCRIPT

Save this as `test_limits.sh` and run with `./test_limits.sh`:

```bash
#!/bin/bash

echo "🧪 Starting Quvanti Labs Limit Tests..."

# Test FREE user agent limit
echo "\n📋 Test 1: FREE user - 1 agent limit"
echo "Creating Agent 1..."
curl -s -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{"name":"Free Agent 1","naturalLanguagePrompt":"Test"}' | jq '.error // "✅ SUCCESS"'

echo "Creating Agent 2 (should fail)..."
curl -s -X POST http://localhost:3000/api/agents/create \
  -H "Content-Type: application/json" \
  -d '{"name":"Free Agent 2","naturalLanguagePrompt":"Test"}' | jq '.error // "❌ UNEXPECTED SUCCESS"'

# Test FREE user strategy limit
echo "\n📋 Test 2: FREE user - 1 strategy/day limit"
echo "Creating Strategy 1..."
curl -s -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Test strategy 1"}' | jq '.error // "✅ SUCCESS"'

echo "Creating Strategy 2 (should fail)..."
curl -s -X POST http://localhost:3000/api/strategy-lab/parse \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Test strategy 2"}' | jq '.error // "❌ UNEXPECTED SUCCESS"'

echo "\n✅ Tests complete!"
```

---

## EXPECTED ERROR MESSAGES

### Agent Limits

**FREE User (2nd agent):**
```json
{
  "error": "agent_limit_reached",
  "message": "Free users can have 1 agent total (lifetime). Upgrade to Pro for 10 agent slots!",
  "tier": "free"
}
```

**PRO User (11th agent):**
```json
{
  "error": "agent_limit_reached",
  "message": "You've reached the maximum of 10 agents. Delete an existing agent to create a new one.",
  "tier": "pro"
}
```

### Strategy Limits

**FREE User (2nd strategy same day):**
```json
{
  "error": "usage_limit_reached",
  "message": "Free users get 1 strategy generation per day. Upgrade to Pro for 15 per day!",
  "tier": "free",
  "remaining": 0
}
```

**PRO User (16th strategy same day):**
```json
{
  "error": "usage_limit_reached",
  "message": "You've reached your daily limit of 15 strategy generations. Try again tomorrow.",
  "tier": "pro",
  "remaining": 0
}
```

---

## VALIDATION CHECKLIST

- [ ] FREE user: Blocked at 2nd agent creation
- [ ] PRO user: Blocked at 11th agent creation
- [ ] PRO user: Can create after deleting agent
- [ ] FREE user: Blocked at 2nd strategy same day
- [ ] PRO user: Blocked at 16th strategy same day
- [ ] Backtest returns Monte Carlo results
- [ ] Backtest returns Walk-Forward results
- [ ] Backtest returns Overfit warnings
- [ ] Dashboard has no annoying banners
- [ ] Landing page shows algorithm tags
- [ ] Billing page shows correct limits
- [ ] "Try Now" buttons link correctly
- [ ] Delete button removes agents
- [ ] PRO Dashboard button only for PRO users
- [ ] Mobile view is clean and readable
- [ ] No brand names visible anywhere

**STATUS: ✅ ALL TESTS SHOULD PASS**



