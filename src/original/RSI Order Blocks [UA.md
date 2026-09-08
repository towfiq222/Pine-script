// This Pine Script™ code is subject to the terms of the Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) https://creativecommons.org/licenses/by-nc-sa/4.0/
// © UAlgo
//@version=6

indicator('RSI Order Blocks [UAlgo]', shorttitle = 'RSI Order Blocks [UAlgo]', overlay = true, max_bars_back = 500)
maxObStack = 8
rsiLength = input.int(14, title = 'RSI Length ', group = 'Relative Strength Index Order Block Settings')
length = input.int(10, title = 'RSI Order Block Sensitivity ', group = 'Relative Strength Index Order Block Settings')
showLastRsiOb = input.int(2, title = 'Show Last ( X ) Number of Order Blocks ', maxval = 6, minval = 1, group = 'Relative Strength Index Order Block Settings')
mitigationMethod = input.string('Close', title = 'Mitigation Method ', options = ['Close', 'Wick'], group = 'Relative Strength Index Order Block Settings')
orderBlockType = input.string('Candle Body', title = 'Order Block Style ', options = ['Candle Body', 'Full Candle'], group = 'Relative Strength Index Order Block Settings')
rsiFilter = input.bool(false, title = 'RSI OB / OS Filter ', tooltip = 'Do NOT Create Order Block if RSI is between 30 and 70', group = 'Relative Strength Index Order Block Settings')
colorBullishOb = input.color(color.new(#089981, 70), title = 'Order Block Colors       ', group = 'Visual Settings', inline = 'visual')
colorBearishOb = input.color(color.new(#f23645, 70), title = ' ', group = 'Visual Settings', inline = 'visual')

for bn in box.all
    bn.delete()

type orderblock
	float top
	float bottom
	int barStart
	bool broken
	float rsiVal

var array<orderblock> bullishObArray = array.new<orderblock>()
var array<orderblock> bearishObArray = array.new<orderblock>()
var int obCounter = na
var float mitigationVal = na
var bool overlap = false
var float top = na
var float bottom = na

rsi = ta.rsi(close, rsiLength)
rsiPh = ta.pivothigh(rsi, length, length)
rsiPl = ta.pivotlow(rsi, length, length)

if bool(rsiPh)
    top := orderBlockType == 'Candle Body' ? close[length] > open[length] ? close[length] : open[length] : high[length]
    bottom := orderBlockType == 'Candle Body' ? close[length] > open[length] ? open[length] : close[length] : low[length]
    if rsiFilter and rsi[length] > 70
        bearishObArray.push(orderblock.new(top, bottom, time[length], false, rsi[length]))
        if array.size(bearishObArray) > maxObStack
            array.shift(bearishObArray)
    else if rsiFilter == false
        bearishObArray.push(orderblock.new(top, bottom, time[length], false, rsi[length]))
        if array.size(bearishObArray) > maxObStack
            array.shift(bearishObArray)

if bool(rsiPl)
    top := orderBlockType == 'Candle Body' ? close[length] > open[length] ? close[length] : open[length] : high[length]
    bottom := orderBlockType == 'Candle Body' ? close[length] > open[length] ? open[length] : close[length] : low[length]
    if rsiFilter and rsi[length] < 30
        bullishObArray.push(orderblock.new(top, bottom, time[length], false, rsi[length]))
        if array.size(bullishObArray) > maxObStack
            array.shift(bullishObArray)
    else if rsiFilter == false
        bullishObArray.push(orderblock.new(top, bottom, time[length], false, rsi[length]))
        if array.size(bullishObArray) > maxObStack
            array.shift(bullishObArray)

// update, removing stuff
if array.size(bearishObArray) > 0
    mitigationVal := mitigationMethod == 'Close' ? close : high
    for i = array.size(bearishObArray) - 1 to 0 by 1
        ob = array.get(bearishObArray, i)
        if mitigationVal > ob.top
            ob.broken := true
            array.remove(bearishObArray, i)

if array.size(bullishObArray) > 0
    mitigationVal := mitigationMethod == 'Close' ? close : low
    for i = array.size(bullishObArray) - 1 to 0 by 1
        ob = array.get(bullishObArray, i)
        if mitigationVal < ob.bottom
            ob.broken := true
            array.remove(bullishObArray, i)

// Draw Blocks Part

if array.size(bearishObArray) > 0
    obCounter := 0
    for i = array.size(bearishObArray) - 1 to 0 by 1
        if obCounter < showLastRsiOb
            ob = array.get(bearishObArray, i)
            overlap := false
            for overlapBox in box.all
                if ob.top > overlapBox.get_top() and ob.bottom < overlapBox.get_bottom()
                    overlap := true
                    overlap
            if overlap == false
                box.new(left = ob.barStart, top = ob.top, right = time, bottom = ob.bottom, bgcolor = colorBearishOb, border_width = 1, text = 'Bearish RSI OB ' + ' - ' + str.tostring(ob.rsiVal, '#.#'), text_halign = text.align_right, text_size = size.tiny, text_color = color.new(color.white, 20), border_color = color.new(color.white, 85), xloc = xloc.bar_time)
                obCounter := obCounter + 1
                obCounter

if array.size(bullishObArray) > 0
    obCounter := 0
    for i = array.size(bullishObArray) - 1 to 0 by 1
        if obCounter < showLastRsiOb
            ob = array.get(bullishObArray, i)
            overlap := false
            for overlapBox in box.all
                if ob.top > overlapBox.get_top() and ob.bottom < overlapBox.get_bottom()
                    overlap := true
                    overlap
            if not overlap
                box.new(left = ob.barStart, top = ob.top, right = time, bottom = ob.bottom, bgcolor = colorBullishOb, border_width = 1, text = 'Bullish RSI OB ' + ' - ' + str.tostring(ob.rsiVal, '#.#'), text_halign = text.align_right, text_size = size.tiny, text_color = color.new(color.white, 20), border_color = color.new(color.white, 85), xloc = xloc.bar_time)
                obCounter := obCounter + 1
                obCounter
