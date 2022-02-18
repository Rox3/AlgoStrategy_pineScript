// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © TradeIITian

//@version=5

![](Images/Screenshot%20(172).png)

indicator('Relative Strength', shorttitle='RS+MACD+RSI', overlay=false)

////////////////////////////////////////////////////RSI(RELATIVE STRENGTH INDICATOR)///////////////////////////////

RSIinput=input(14, "RSI Input")
RSI=ta.rsi(close,RSIinput)
RSIvisi = input(title='RSI Visible', defval=true)
plot(RSIvisi?(RSI/20)-2.5:na,color = color.purple)
// Divided by 20 and then subtracted 2.5 cuz we know range of rsi is [0,100] which is far away with the rs range so to manage both ranges i did this.
// and this will not lead to change the characteristics of rsi plot








////////////////////////////////////////////////////(MACD)///////////////////////////////

//macd--- https://www.tradingview.com/pine-script-reference/v5/#fun_ta{dot}macd


fastInput = input(12, "Fast length")
slowInput = input(26, "Slow length")
[macdLine, signalLine, histLine] = ta.macd(close, fastInput, slowInput, 9)

MACDvisi = input(title='MACD Visible', defval=true)


plot(MACDvisi?macdLine/20:na, color = color.blue)
plot(MACDvisi?signalLine/20:na, color = color.orange)
    //plot(histLine, color=color.red, style=plot.style_histogram)
    
// Divided by 20 cuz we know range of MACD is far away with the rs range so to manage both ranges i did this.
// and this will not lead to change the characteristics of MACD plot












////////////////////////////////////////////////////RS(RELATIVE STRENGTH )///////////////////////////////

//Input
source = input(title='Source', defval=close)
comparativeTickerId = input.symbol('NSE:NIFTY', title='Comparative Symbol')
length = input.int(55, minval=1, title='Period')
showZeroLine = input(defval=true, title='Show Zero Line')
showRefDateLbl = input(defval=true, title='Show Reference Label')
toggleRSColor = input(defval=true, title='Toggle RS color on crossovers')
showRSTrend = input.bool(defval=false, title='RS Trend,', group='RS Trend', inline='RS Trend')
base = input.int(title='Range', minval=1, defval=5, group='RS Trend', inline='RS Trend')
showMA = input.bool(defval=false, title='Show Moving Average,', group='RS Mean', inline='RS Mean')
lengthMA = input.int(61, minval=1, title='Period', group='RS Mean', inline='RS Mean')

//Set up
baseSymbol = request.security(syminfo.tickerid, timeframe.period, source)
comparativeSymbol = request.security(comparativeTickerId, timeframe.period, source)

//Calculations
res = baseSymbol / baseSymbol[length] / (comparativeSymbol / comparativeSymbol[length]) - 1
resColor = toggleRSColor ? res > 0 ? color.green : color.red : color.blue
refDay = showRefDateLbl and barstate.islast ? dayofmonth(time[length]) : na
refMonth = showRefDateLbl and barstate.islast ? month(time[length]) : na
refYear = showRefDateLbl and barstate.islast ? year(time[length]) : na
refLabelStyle = res[length] > 0 ? label.style_label_up : label.style_label_down
refDateLabel = showRefDateLbl and barstate.islast ? label.new(bar_index - length, 0, text='RS-' + str.tostring(length) + ' reference, ' + str.tostring(refDay) + '-' + str.tostring(refMonth) + '-' + str.tostring(refYear), color=color.blue, style=refLabelStyle, yloc=yloc.price) : na
y0 = res - res[base]
angle0 = math.atan(y0 / base)  // radians
zeroLineColor = showRSTrend ? angle0 > 0.0 ? color.green : color.maroon : color.maroon

//Plot
plot(showZeroLine ? 0 : na, linewidth=2, color=zeroLineColor, title='Zero Line / RS Trend')
plot(res, title='RS', linewidth=2, color=resColor)
plot(showMA ? ta.sma(res, lengthMA) : na, color=color.new(color.gray, 0), title='MA')

