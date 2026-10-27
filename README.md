# mamba-fx-1min-trade
most profitable strategy for 1min 



 //@version=5
strategy("Mamba FX Trading Strategy", overlay=true, pyramiding=0, 
     initial_capital=10000)

// Settings
zoneTF = input.timeframe("5", "Support/Resistance Timeframe")
rr = input.float(2.0, "Risk Reward", minval=1.0)
pivotLen = input.int(3, "Pivot Strength", minval=1)
zonePct = input.float(0.05, "Zone Width (%)", minval=0.01, step=0.01)

// 5-minute support/resistance
ph = request.security(syminfo.tickerid, zoneTF,
     ta.pivothigh(high, pivotLen, pivotLen),
     lookahead=barmerge.lookahead_off)

pl = request.security(syminfo.tickerid, zoneTF,
     ta.pivotlow(low, pivotLen, pivotLen),
     lookahead=barmerge.lookahead_off)

var float resistance = na
var float support = na

if not na(ph)
    resistance := ph

if not na(pl)
    support := pl

// Zone boundaries
supTop = support * (1 + zonePct / 100)
supBot = support * (1 - zonePct / 100)

resTop = resistance * (1 + zonePct / 100)
resBot = resistance * (1 - zonePct / 100)

// 1-minute trigger candle
triggerHigh = high[1]
triggerLow  = low[1]

// Tap and breakout
supportTap = not na(support) and low <= supTop and high >= supBot
resistanceTap = not na(resistance) and high >= resBot and low <= resTop

var bool buyArmed = false
var bool sellArmed = false

if supportTap
    buyArmed := true

if resistanceTap
    sellArmed := true

buySignal = buyArmed and close > triggerHigh and close > open
sellSignal = sellArmed and close < triggerLow and close < open

// Stop loss and take profit
var float entryPrice = na
var float stopPrice = na
var float targetPrice = na

if buySignal and strategy.position_size == 0
    entryPrice := close
    stopPrice := ta.lowest(low, pivotLen + 1)
    risk = entryPrice - stopPrice
    if risk > syminfo.mintick
        targetPrice := entryPrice + risk * rr
        strategy.entry("BUY", strategy.long)
        strategy.exit("BUY EXIT", "BUY", stop=stopPrice, limit=targetPrice)
    buyArmed := false

if sellSignal and strategy.position_size == 0
    entryPrice := close
    stopPrice := ta.highest(high, pivotLen + 1)
    risk = stopPrice - entryPrice
    if risk > syminfo.mintick
        targetPrice := entryPrice - risk * rr
        strategy.entry("SELL", strategy.short)
        strategy.exit("SELL EXIT", "SELL", stop=stopPrice, limit=targetPrice)
    sellArmed := false

// Plot zones
plot(support, "5m Support", color=color.lime, linewidth=2,
     style=plot.style_stepline)
plot(resistance, "5m Resistance", color=color.red, linewidth=2,
     style=plot.style_stepline)

plotshape(buySignal, title="BUY", text="BUY",
     style=shape.labelup, location=location.belowbar,
     color=color.green, textcolor=color.white)

plotshape(sellSignal, title="SELL", text="SELL",
     style=shape.labeldown, location=location.abovebar,
     color=color.red, textcolor=color.white)

alertcondition(buySignal, "BUY Alert", "Mamba FX BUY breakout")
alertcondition(sellSignal, "SELL Alert", "Mamba FX SELL breakout")
