// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © mentalRock19315

//@version=6

//////////////////////////////////
// This indicator draw the orders blocks on your chart
//
// Order blocks are defined by the candle and its wicks.
// If the price goes into the order block, the size of the order block will change accordingly
// If the price cross the order block, the order block will disappear
//
// This way, you will have on your screen only the orders blocks never touched.
//
// Order block olders than 5000 bars are deleted
//
// You can choose to alway show the order block :
// - of the 1D (One day) timeframe
// - of the 1h (One hour) timeframe
// On those order block, you can choose the backgroung and border color
// you can also choose to write the order block name
//
// By default, the number of drawn 1D and 1h Order Block is limited to 1 above and 1 below the range of the chart price

// Modifications
// Way of storing and sorting the matrix more effective
// The script draw all the OB inside the lowest and highest value of the price 
// + will draw 1 OB which are outside the price range

MaxDrawn = 0

indicator('Orders Blocks [TK]', overlay = true, max_bars_back = 4999, max_boxes_count = 500)
import mentalRock19315/NumberOfVisibleBars/3
Color_border_order_bloc_bull = input.color(color.new(color.green, 50), 'Bull Order Block border color')
Color_background_order_bloc_bull = input.color(color.new(color.green, 90), 'Bull Order Block background color')
Color_border_order_bloc_bear = input.color(color.new(color.red, 50), 'Bear Order Block border color')
Color_background_order_bloc_bear = input.color(color.new(color.red, 90), 'Bear Order Block background color')

One_Day_Order_Block = input.bool(true, 'Always draw the Order Block from the 1 Day timeframe ?', group = 'One Day Timeframe')
Color_border_1D_order_bloc_bull = input.color(color.new(color.aqua, 50), '1D Bull Order Block border color', group = 'One Day Timeframe')
Color_background_1D_order_bloc_bull = input.color(color.new(color.aqua, 90), '1D Bull Order Block background color', group = 'One Day Timeframe')
Color_border_1D_order_bloc_bear = input.color(color.new(color.aqua, 50), '1D Bear Order Block border color', group = 'One Day Timeframe')
Color_background_1D_order_bloc_bear = input.color(color.new(color.aqua, 90), '1D Bear Order Block background color', group = 'One Day Timeframe')
Text_1D_Order_Block = input.bool(true, 'Write the name of the order block inside it ?', group = 'One Day Timeframe')

One_Hour_Order_Block = input.bool(true, 'Always draw the Order Block from the 1 Hour timeframe ?', group = 'One Hour Timeframe')
Color_border_1h_order_bloc_bull = input.color(color.new(color.teal, 50), '1h Bull Order Block border color', group = 'One Hour Timeframe')
Color_background_1h_order_bloc_bull = input.color(color.new(color.teal, 90), '1h Bull Order Block background color', group = 'One Hour Timeframe')
Color_border_1h_order_bloc_bear = input.color(color.new(color.teal, 50), '1h Bear Order Block border color', group = 'One Hour Timeframe')
Color_background_1h_order_bloc_bear = input.color(color.new(color.teal, 90), '1h Bear Order Block background color', group = 'One Hour Timeframe')
Text_1h_Order_Block = input.bool(true, 'Write the name of the order block inside it ?', group = 'One Hour Timeframe')

Max_bar_back = 4999
max_bars_back(time, 4999)

// Contains the OB formed by a Bear candle followed by a Bull candle
var Ob_Bull_Matrix = matrix.new<float>(0, 3, na) // Columns : bar_index_start / bottom_ob // top_ob
// Contains the OB formed by a Bull candle followed by a Bear candle
var Ob_Bear_Matrix = matrix.new<float>(0, 3, na) // Columns : bar_index_start / bottom_ob // top_ob
//
var Ob_1D_Bull_Matrix = matrix.new<float>(0, 3, na) // Columns : bar_index_start / bottom_ob // top_ob
var Ob_1D_Bear_Matrix = matrix.new<float>(0, 3, na) // Columns : bar_index_start / bottom_ob // top_ob
DailyLow = request.security(syminfo.tickerid, '1D', low)
DailyHigh = request.security(syminfo.tickerid, '1D', high)
DailyOpen = request.security(syminfo.tickerid, '1D', open)
DailyClose = request.security(syminfo.tickerid, '1D', close)
var DailyBarIndex = 0
var Last_Previous_Bar_Time_Daily_Test = 0
//
var Ob_1h_Bull_Matrix = matrix.new<float>(0, 3, na) // Columns : bar_index_start / bottom_ob // top_ob
var Ob_1h_Bear_Matrix = matrix.new<float>(0, 3, na) // Columns : bar_index_start / bottom_ob // top_ob
HourlyLow = request.security(syminfo.tickerid, '60', low)
HourlyHigh = request.security(syminfo.tickerid, '60', high)
HourlyOpen = request.security(syminfo.tickerid, '60', open)
HourlyClose = request.security(syminfo.tickerid, '60', close)
var HourlyBarIndex = 0
var Last_Previous_Bar_Time_Hourly_Test = 0

if timeframe.in_seconds() < 86400 // The actual timeframe is under 1 Day
    if dayofmonth(time, 'UTC+1') != dayofmonth(Last_Previous_Bar_Time_Daily_Test, 'UTC+1')
        // It a new day
        DailyBarIndex := bar_index
        DailyBarIndex

if timeframe.in_seconds() < 3600 // The actual timeframe is under 1 Hour
    if hour(time, 'UTC+1') != hour(Last_Previous_Bar_Time_Daily_Test, 'UTC+1')
        // It a new hour
        HourlyBarIndex := bar_index
        HourlyBarIndex

// chart.left_visible_bar_time 
// Functionto calculated the number of visible bar

// Function to draw the box
// Calcul the number of bar in the screen
NbBar = NumberOfVisibleBars.NumberOfVisibleBars()
HighestPrice = ta.highest(high, NbBar)
LowestPrice = ta.lowest(low, NbBar)
var Number_Of_Lines_To_Remove = 0
Draw(Matrix, BgColor, BorderColor, Max_bar_back, Text) =>
    // The Matrix should be sorted as the first line is the line the closest to the price (the last line is the farest to the price)
    if matrix.rows(Matrix) > 0
        Number_Of_OB_Drawn_Outside = 0
        for Line = 0 to matrix.rows(Matrix) - 1 by 1
            // Are we in the range of the price drawn on the user screen ?
            left_box = int(matrix.get(Matrix, Line, 0))
            if left_box < bar_index - Max_bar_back
                left_box := bar_index - Max_bar_back
                left_box
            top_box = matrix.get(Matrix, Line, 2)
            right_box = bar_index
            bottom_box = matrix.get(Matrix, Line, 1)
            if top_box < HighestPrice and bottom_box > LowestPrice
                box.new(left_box, top_box, right_box, bottom_box, border_color = BorderColor, bgcolor = BgColor, text = Text, text_halign = text.align_right, text_color = color.black, text_size = size.normal)
            else // the OB is outside the price range, we only draw MaxDrawn time
                if Number_Of_OB_Drawn_Outside <= MaxDrawn
                    box.new(left_box, top_box, right_box, bottom_box, border_color = BorderColor, bgcolor = BgColor, text = Text, text_halign = text.align_right, text_color = color.black, text_size = size.normal)
                else
                    break
                Number_Of_OB_Drawn_Outside := Number_Of_OB_Drawn_Outside + 1
                Number_Of_OB_Drawn_Outside
        result = true
        result
    result = true
    result
if barstate.isconfirmed // On each close of a bar
    // Adjustments of the OB in the Bull Matrix
    // The OB Bull Matrix are those UNDER the actual price
    // Sorted from the top_ob (column index 2) from the highest to the lowest
    if matrix.rows(Ob_Bull_Matrix) > 0
        Number_Of_Lines_To_Remove := 0
        matrix.sort(Ob_Bull_Matrix, 2, order.descending)
        for L = 0 to matrix.rows(Ob_Bull_Matrix) - 1 by 1
            // Columns : bar_index_start / bottom_ob // top_ob
            // The low of the price is lower than the top_ob
            if low < matrix.get(Ob_Bull_Matrix, L, 2)
                // The low of the price is higher than the bottom_ob 
                if low > matrix.get(Ob_Bull_Matrix, L, 1) // The price bites the OB
                    matrix.set(Ob_Bull_Matrix, L, 2, low) // Adjust the OB
                    break // End of the loop, 
                else // The low of the price is lower than the bottom_ob
                    Number_Of_Lines_To_Remove := Number_Of_Lines_To_Remove + 1
                    Number_Of_Lines_To_Remove
            else if low > matrix.get(Ob_Bull_Matrix, L, 2) // The price is now higher than the other OB
                break
        if Number_Of_Lines_To_Remove > 0
            for L = 0 to Number_Of_Lines_To_Remove - 1 by 1
                matrix.remove_row(Ob_Bull_Matrix, 0)

    // Adjustments of the OB in the 1D Bull Matrix
    if timeframe.in_seconds() < 86400 and One_Day_Order_Block // The actual timeframe is under 1 Day
        if matrix.rows(Ob_1D_Bull_Matrix) > 0
            Number_Of_Lines_To_Remove := 0
            matrix.sort(Ob_1D_Bull_Matrix, 2, order.descending)
            for L = 0 to matrix.rows(Ob_1D_Bull_Matrix) - 1 by 1
                // Columns : bar_index_start / bottom_ob // top_ob
                // The low of the price is lower than the top_ob
                if low < matrix.get(Ob_1D_Bull_Matrix, L, 2)
                    // The low of the price is higher than the bottom_ob 
                    if low > matrix.get(Ob_1D_Bull_Matrix, L, 1) // The price bites the OB
                        matrix.set(Ob_1D_Bull_Matrix, L, 2, low) // Adjust the OB
                        break // End of the loop, 
                    else // The low of the price is lower than the bottom_ob
                        Number_Of_Lines_To_Remove := Number_Of_Lines_To_Remove + 1
                        Number_Of_Lines_To_Remove
                else if low > matrix.get(Ob_1D_Bull_Matrix, L, 2) // The price is now higher than the other OB
                    break
            if Number_Of_Lines_To_Remove > 0
                for L = 0 to Number_Of_Lines_To_Remove - 1 by 1
                    matrix.remove_row(Ob_1D_Bull_Matrix, 0)

    // Adjustments of the OB in the 1h Bull Matrix
    if timeframe.in_seconds() < 3600 and One_Hour_Order_Block // The actual timeframe is under 1 Hour
        if matrix.rows(Ob_1h_Bull_Matrix) > 0
            Number_Of_Lines_To_Remove := 0
            matrix.sort(Ob_1h_Bull_Matrix, 2, order.descending)
            for L = 0 to matrix.rows(Ob_1h_Bull_Matrix) - 1 by 1
                // Columns : bar_index_start / bottom_ob // top_ob
                // The low of the price is lower than the top_ob
                if low < matrix.get(Ob_1h_Bull_Matrix, L, 2)
                    // The low of the price is higher than the bottom_ob 
                    if low > matrix.get(Ob_1h_Bull_Matrix, L, 1) // The price bites the OB
                        matrix.set(Ob_1h_Bull_Matrix, L, 2, low) // Adjust the OB
                        break // End of the loop, 
                    else // The low of the price is lower than the bottom_ob
                        Number_Of_Lines_To_Remove := Number_Of_Lines_To_Remove + 1
                        Number_Of_Lines_To_Remove
                else if low > matrix.get(Ob_1h_Bull_Matrix, L, 2) // The price is now higher than the other OB
                    break
            if Number_Of_Lines_To_Remove > 0
                for L = 0 to Number_Of_Lines_To_Remove - 1 by 1
                    matrix.remove_row(Ob_1h_Bull_Matrix, 0)

// Adjustments of the OB in the Bear Matrix

// Adjustments of the OB in the Bear Matrix
// The OB BuBearll Matrix are those ABOVE the actual price
// Sorted from the bottom_ob (column index 1) from the lowest to the highest
// Columns : bar_index_start / bottom_ob // top_ob

    if matrix.rows(Ob_Bear_Matrix) > 0
        Number_Of_Lines_To_Remove := 0
        matrix.sort(Ob_Bear_Matrix, 1, order.ascending)
        for L = 0 to matrix.rows(Ob_Bear_Matrix) - 1 by 1
            // The high of the price is higher than the bottom_ob
            if high > matrix.get(Ob_Bear_Matrix, L, 1)
                // The high of the price is lower than the top_ob 
                if high < matrix.get(Ob_Bear_Matrix, L, 2) // The price bites the OB
                    matrix.set(Ob_Bear_Matrix, L, 1, high) // Adjust the OB
                    break // End of the loop, 
                else // The high of the price is higher than the top_ob
                    Number_Of_Lines_To_Remove := Number_Of_Lines_To_Remove + 1
                    Number_Of_Lines_To_Remove
            else if high < matrix.get(Ob_Bear_Matrix, L, 1) // The price is now higher than the other OB
                break // End of the loop
        if Number_Of_Lines_To_Remove > 0
            for L = 0 to Number_Of_Lines_To_Remove - 1 by 1
                matrix.remove_row(Ob_Bear_Matrix, 0)

    // Adjustments of the OB in the 1D Bear Matrix
    if timeframe.in_seconds() < 86400 and One_Day_Order_Block // The actual timeframe is under 1 Day
        if matrix.rows(Ob_1D_Bear_Matrix) > 0
            Number_Of_Lines_To_Remove := 0
            matrix.sort(Ob_1D_Bear_Matrix, 1, order.ascending)
            for L = 0 to matrix.rows(Ob_1D_Bear_Matrix) - 1 by 1
                // The high of the price is higher than the bottom_ob
                if high > matrix.get(Ob_1D_Bear_Matrix, L, 1)
                    // The high of the price is lower than the top_ob 
                    if high < matrix.get(Ob_1D_Bear_Matrix, L, 2) // The price bites the OB
                        matrix.set(Ob_1D_Bear_Matrix, L, 1, high) // Adjust the OB
                        break // End of the loop, 
                    else // The high of the price is higher than the top_ob
                        Number_Of_Lines_To_Remove := Number_Of_Lines_To_Remove + 1
                        Number_Of_Lines_To_Remove
                else if high < matrix.get(Ob_1D_Bear_Matrix, L, 1) // The price is now higher than the other OB
                    break // End of the loop
            if Number_Of_Lines_To_Remove > 0
                for L = 0 to Number_Of_Lines_To_Remove - 1 by 1
                    matrix.remove_row(Ob_1D_Bear_Matrix, 0)

    // Adjustments of the OB in the 1h Bear Matrix
    if timeframe.in_seconds() < 3600 and One_Hour_Order_Block // The actual timeframe is under 1 Hour
        if matrix.rows(Ob_1h_Bear_Matrix) > 0
            Number_Of_Lines_To_Remove := 0
            matrix.sort(Ob_1h_Bear_Matrix, 1, order.ascending)
            for L = 0 to matrix.rows(Ob_1h_Bear_Matrix) - 1 by 1
                // The high of the price is higher than the bottom_ob
                if high > matrix.get(Ob_1h_Bear_Matrix, L, 1)
                    // The high of the price is lower than the top_ob 
                    if high < matrix.get(Ob_1h_Bear_Matrix, L, 2) // The price bites the OB
                        matrix.set(Ob_1h_Bear_Matrix, L, 1, high) // Adjust the OB
                        break // End of the loop, 
                    else // The high of the price is higher than the top_ob
                        Number_Of_Lines_To_Remove := Number_Of_Lines_To_Remove + 1
                        Number_Of_Lines_To_Remove
                else if high < matrix.get(Ob_1h_Bear_Matrix, L, 1) // The price is now higher than the other OB
                    break // End of the loop
            if Number_Of_Lines_To_Remove > 0
                for L = 0 to Number_Of_Lines_To_Remove - 1 by 1
                    matrix.remove_row(Ob_1h_Bear_Matrix, 0)

    // Add a Bull OB : Bear Candle followed by a Bull Candle
    if open[1] > close[1] and open < close and high > high[1]
        ArrayToAdd = array.from(bar_index, low[1], high[1])
        matrix.add_row(Ob_Bull_Matrix, matrix.rows(Ob_Bull_Matrix), ArrayToAdd)
        matrix.sort(Ob_Bull_Matrix, 2, order.descending)

    // Add a 1 Day Bull OB : Bear Candle followed by a Bull Candle
    if timeframe.in_seconds() < 86400 and One_Day_Order_Block // The actual timeframe is under 1 Day
        if DailyOpen[1] > DailyClose[1] and DailyOpen < DailyClose and DailyClose > DailyHigh[1]
            ArrayToAdd = array.from(DailyBarIndex, DailyLow[1], DailyHigh[1])
            matrix.add_row(Ob_1D_Bull_Matrix, matrix.rows(Ob_1D_Bull_Matrix), ArrayToAdd)
            matrix.sort(Ob_1D_Bull_Matrix, 2, order.descending)

    // Add a 1 Hour Bull OB : Bear Candle followed by a Bull Candle
    if timeframe.in_seconds() < 3600 and One_Hour_Order_Block // The actual timeframe is under 1 Hour
        if HourlyOpen[1] > HourlyClose[1] and HourlyOpen < HourlyClose and HourlyClose > HourlyHigh[1]
            ArrayToAdd = array.from(HourlyBarIndex, HourlyLow[1], HourlyHigh[1])
            matrix.add_row(Ob_1h_Bull_Matrix, matrix.rows(Ob_1h_Bull_Matrix), ArrayToAdd)
            // Inverted Sort of the Matrix with the top_ob key (columns index 2) => Line 0 = highest, Line max = lowest
            matrix.sort(Ob_1h_Bull_Matrix, 2, order.descending)

    // Add a Bear OB : Bull Candle followed by a Bear Candle
    if open[1] < close[1] and open > close and low < low[1]
        ArrayToAdd = array.from(bar_index, low[1], high[1])
        matrix.add_row(Ob_Bear_Matrix, matrix.rows(Ob_Bear_Matrix), ArrayToAdd)
        matrix.sort(Ob_Bear_Matrix, 1, order.ascending)

    // Add a 1 Day Bear OB : Bull Candle followed by a Bear Candle
    if timeframe.in_seconds() < 86400 and One_Day_Order_Block // The actual timeframe is under 1 Day
        if DailyOpen[1] < DailyClose[1] and DailyOpen > DailyClose and DailyClose < DailyLow[1]
            ArrayToAdd = array.from(DailyBarIndex, DailyLow[1], DailyHigh[1])
            matrix.add_row(Ob_1D_Bear_Matrix, matrix.rows(Ob_1D_Bear_Matrix), ArrayToAdd)
            matrix.sort(Ob_1D_Bear_Matrix, 1, order.ascending)

    // Add a 1 Hour Bear OB : Bull Candle followed by a Bear Candle
    if timeframe.in_seconds() < 3600 and One_Hour_Order_Block // The actual timeframe is under 1 Day
        if HourlyOpen[1] < HourlyClose[1] and HourlyOpen > HourlyClose and HourlyClose < HourlyLow[1]
            ArrayToAdd = array.from(HourlyBarIndex, HourlyLow[1], HourlyHigh[1])
            matrix.add_row(Ob_1h_Bear_Matrix, matrix.rows(Ob_1h_Bear_Matrix), ArrayToAdd)
            matrix.sort(Ob_1h_Bear_Matrix, 1, order.ascending)

if barstate.islast // On the last bar on the screen
    // Drawing
    // Delete all the box on screen
    a_allBoxes = box.all
    if array.size(a_allBoxes) > 0
        for i = 0 to array.size(a_allBoxes) - 1 by 1
            box.delete(array.get(a_allBoxes, i))
    // Sort 
    // Draw all the box of the Ob_Bull_Matrix
    Draw(Ob_Bull_Matrix, Color_background_order_bloc_bull, Color_border_order_bloc_bull, Max_bar_back, '')
    // Draw all the box of the Ob_Bear_Matrix
    Draw(Ob_Bear_Matrix, Color_background_order_bloc_bear, Color_border_order_bloc_bear, Max_bar_back, '')

    if timeframe.in_seconds() < 86400 and One_Day_Order_Block
        // Draw all the box of the Ob_Bull_Matrix
        Draw(Ob_1D_Bull_Matrix, Color_background_1D_order_bloc_bull, Color_border_1D_order_bloc_bull, Max_bar_back, 'Order Block [1D]')
        // Draw all the box of the Ob_Bear_Matrix
        Draw(Ob_1D_Bear_Matrix, Color_background_1D_order_bloc_bear, Color_border_1D_order_bloc_bear, Max_bar_back, 'Order Block [1D]')

    if timeframe.in_seconds() < 3600 and One_Hour_Order_Block
        // Draw all the box of the Ob_Bull_Matrix
        Draw(Ob_1h_Bull_Matrix, Color_background_1h_order_bloc_bull, Color_border_1h_order_bloc_bull, Max_bar_back, 'Order Block [1h]')
        // Draw all the box of the Ob_Bear_Matrix
        Draw(Ob_1h_Bear_Matrix, Color_background_1h_order_bloc_bear, Color_border_1h_order_bloc_bear, Max_bar_back, 'Order Block [1h]')
