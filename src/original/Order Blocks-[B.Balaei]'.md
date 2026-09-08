//@version=6
indicator(title='Order Blocks-[B.Balaei]', shorttitle='Order Blocks-[B.Balaei]', overlay=true, max_boxes_count=20)

mtf = input.timeframe("5", title="Multi-Timeframe", group="Multi-Timeframe Settings")

mtf_open = request.security(syminfo.tickerid, mtf, open)
mtf_close = request.security(syminfo.tickerid, mtf, close)
mtf_high = request.security(syminfo.tickerid, mtf, high)
mtf_low = request.security(syminfo.tickerid, mtf, low)

_v = input.string("1.0.2", title="Version", options=["1.0.2"], group="Version")
sens = input.int(28, minval=1, title='Sensitivity', group='Order Block', tooltip='Lower the sensitivity to show more order blocks. A higher sensitivity will show less order blocks.')
sens /= 100

OBMitigationType = input.string("Close", title="OB Mitigation Type", options=["Close", "Wick"], group="Order Block", tooltip="Choose how Order Blocks are mitigated")
OBBullMitigation = OBMitigationType=="Close" ? mtf_close[1] : mtf_low
OBBearMitigation = OBMitigationType=="Close" ? mtf_close[1] : mtf_high

col_bullish = input.color(#5db49e, title="Bullish OB Border", inline="a", group="Order Block")
col_bullish_ob = input.color(color.new(#64C4AC, 85), title="Background", inline="a", group="Order Block")

col_bearish = input.color(#4760bb, title="Bearish OB Border", inline="b", group="Order Block")
col_bearish_ob = input.color(color.new(#506CD3, 85), title="Background", inline="b", group="Order Block")

buy_alert = input.bool(title='Buy Signal', defval=true, group='Alerts', tooltip='An alert will be sent when price goes below the top of a bullish order block.')
sell_alert = input.bool(title='Sell Signal', defval=true, group='Alerts', tooltip='An alert will be sent when price goes above the bottom of a bearish order block.')

bool ob_created = false
bool ob_created_bull = false
var int cross_index = na

var box drawlongBox = na
var longBoxes = array.new_box()
var box drawShortBox = na
var shortBoxes = array.new_box()

pc_mtf = (mtf_open - mtf_open[4]) / mtf_open[4] * 100

if ta.crossunder(pc_mtf, -sens)
    ob_created := true
    cross_index := bar_index
    cross_index

if ta.crossover(pc_mtf, sens)
    ob_created_bull := true
    cross_index := bar_index
    cross_index

if ob_created and cross_index - cross_index[1] > 5
    float last_green = 0
    float highest = 0
    for i = 4 to 15 by 1
        if mtf_close[i] > mtf_open[i]
            last_green := i
            break
    drawShortBox := box.new(left=bar_index[last_green], top=mtf_high[last_green], bottom=mtf_low[last_green], right=bar_index[last_green], bgcolor=col_bearish_ob, border_color=col_bearish, extend=extend.right)
    array.push(shortBoxes, drawShortBox)

if ob_created_bull and cross_index - cross_index[1] > 5
    float last_red = 0
    float highest = 0
    for i = 4 to 15 by 1
        if mtf_close[i] < mtf_open[i]
            last_red := i
            break
    drawlongBox := box.new(left=bar_index[last_red], top=mtf_high[last_red], bottom=mtf_low[last_red], right=bar_index[last_red], bgcolor=col_bullish_ob, border_color=col_bullish, extend=extend.right)
    array.push(longBoxes, drawlongBox)

if array.size(shortBoxes) > 0
    for i = array.size(shortBoxes) - 1 to 0 by 1
        sbox = array.get(shortBoxes, i)
        top = box.get_top(sbox)
        bot = box.get_bottom(sbox)
        if OBBearMitigation > top
            array.remove(shortBoxes, i)
            box.delete(sbox)
        if mtf_high > bot and sell_alert
            alert('Price inside Bearish OB (MTF)', alert.freq_once_per_bar)

if array.size(longBoxes) > 0
    for i = array.size(longBoxes) - 1 to 0 by 1
        sbox = array.get(longBoxes, i)
        bot = box.get_bottom(sbox)
        top = box.get_top(sbox)
        if OBBullMitigation < bot
            array.remove(longBoxes, i)
            box.delete(sbox)
        if mtf_low < top and buy_alert
            alert('Price inside Bullish OB (MTF)', alert.freq_once_per_bar)