// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © LonesomeTheBlue

indi1n = "Relative strength index",     indi1sn = "RSI"
indi2n ="Commodity Channel Index",      indi2sn ="CCI"
indi3n ="On Balance Volume",            indi3sn ="OBV"
indi4n ="Stochastic",                   indi4sn ="Stoch"
indi5n ="Money Flow Index",             indi5sn ="MFI"
indi6n ="Average True Range",           indi6sn ="ATR"
indi7n ="Chande Momentum Oscillator",   indi7sn ="CMO"
indi8n ="Chaikin Money Flow",           indi8sn ="CMF"
indi9n ="Stochastic RSI",               indi9sn ="StochRSI D"
indi10n ="DMI",                         indi10sn ="+DI -DI ADX"

getTN()=>
    int TN = 0
    tick__ = syminfo.mintick
    while tick__ < 1
        TN += 1
        tick__ *= 10

    TN
    
//@version=5
indicator("Indicators Overlay", overlay = true, max_lines_count = 500, max_bars_back = 900)

import LonesomeTheBlue/DrawIndicatorOnTheChart/5 as showindicator

indi1 = input.bool(defval = true, title = indi1n, group = "Indicators", inline = "indi1")
indi1s = input.source(defval = close, title = "", group = "Indicators", inline = "indi1")
indi1l = input.int(defval = 14, title = "", group = "Indicators", inline = "indi1")

indi2 = input.bool(defval = true, title = indi2n, group = "Indicators", inline = "indi2")
indi2s = input.source(defval = close, title = "", group = "Indicators", inline = "indi2")
indi2l = input.int(defval = 20, title = "", group = "Indicators", inline = "indi2")

indi3 = input.bool(defval = false, title = indi3n, group = "Indicators")

indi4 = input.bool(defval = false, title = indi4n, group = "Indicators", inline = "indi4")
indi4s = input.source(defval = close, title = "", group = "Indicators", inline = "indi4")
indi4l = input.int(defval = 14, title = "", group = "Indicators", inline = "indi4")

indi5 = input.bool(defval = true, title = indi5n, group = "Indicators", inline = "indi5")
indi5s = input.source(defval = hlc3, title = "", group = "Indicators", inline = "indi5")
indi5l = input.int(defval = 14, title = "", group = "Indicators", inline = "indi5")

indi6 = input.bool(defval = false, title = indi6n, group = "Indicators", inline = "indi6")
indi6l = input.int(defval = 14, title = "", group = "Indicators", inline = "indi6")

indi7 = input.bool(defval = false, title = indi7n, group = "Indicators", inline = "indi7")
indi7s = input.source(defval = close, title = "", group = "Indicators", inline = "indi7")
indi7l = input.int(defval = 9, title = "", group = "Indicators", inline = "indi7")

indi8 = input.bool(defval = true, title = indi8n, group = "Indicators", inline = "indi8")
indi8l = input.int(defval = 20, title = "", group = "Indicators", inline = "indi8")

indi9 = input.bool(defval = true, title = indi9n, group = "Indicators", inline = "indi9")
indi9l1 = input.int(defval = 14, title = "", group = "Indicators", inline = "indi9")
indi9l2 = input.int(defval = 3, title = "| D", group = "Indicators", inline = "indi9")

indi10 = input.bool(defval = false, title = indi10n, group = "Indicators", inline = "indi10")
indi10l1 = input.int(defval = 14, title = " | ADX Smoothing", minval=1, maxval=50, group = "Indicators", inline = "indi10")
indi10l2 = input.int(defval = 3, title = "DI Len", group = "Indicators", inline = "indi10")

period = input.int(defval = 50, title = "Length for Each Window", minval = 20, maxval = 100, inline ="Ext")
lnwidth = input.int(defval = 2, title = "Line Width", minval = 1, maxval = 3, inline ="Ext")

differentcolors = input.bool(defval = false, title = "Different Colors for Indicators?", inline = "colors")
color1 = input.color(defval = color.blue, title = " or These Colors", inline = "colors")
color2 = input.color(defval = color.red, title = "", inline = "colors")
color3 = input.color(defval = color.gray, title = "", inline = "colors")

// get indicator values
rsi = ta.rsi(indi1s, indi1l)
cci = ta.cci(indi2s, indi2l)
obv = ta.obv
stoch = ta.stoch(indi4s, high, low, indi4l)
mfi = ta.mfi(indi5s, indi5l)
atr = ta.atr(indi6l)
cmo = ta.cmo(indi7s, indi7l)
ad = close==high and close==low or high==low ? 0 : ((2*close-low-high)/(high-low))*volume
cmf = math.sum(ad, indi8l) / math.sum(volume, indi8l)
srsi = ta.sma(ta.stoch(rsi, rsi, rsi, indi9l1), 3)
srsid = ta.sma(srsi, indi10l2)
[diplus, diminus, adx] = ta.dmi(indi10l2, indi10l1)

// initialize variables/arrays
var indicolors = differentcolors ? array.from(color.blue, color.olive, color.lime, color.teal, color.silver, color.green, color.yellow, color.orange, color.blue, color.olive, color.lime, color.teal, color.silver) : 
                                   array.from(color1, color2, color3)
var indinames = array.from(indi1sn, indi2sn, indi3sn, indi4sn, indi5sn, indi6sn, indi7sn, indi8sn, indi9sn, indi10sn)
var indienabled = array.from(indi1, indi2, indi3, indi4, indi5, indi6, indi7, indi8, indi9, indi10)
var precision = array.from(2, 2, 2, 2, 2, getTN(), 2, 2, 2, 2)
var float naval = na
var indimaxmin =               array.from(100.0, 0.0,         naval, naval,          naval, naval,  100.0, 0.0,    100.0, 0.0,    naval, naval,     100.0, -100.0,    naval, naval,     102.0, -2.0,    70.0, -5.0)
var indihorizontallevels =     array.from(30.0, 50.0, 70.0,   0.0, 100.0, -100.0,    naval,         20.0, 80.0,    20.0, 80.0,    naval,            0.0,              0.0,              20.0, 80.0,     0.0, 20.0, 40.0, 60.0)
var indihorizontallevelindex = array.from(0, 3,               3, 3,                  6, 1,          7, 2,          9, 2,          11, 1,            12, 1,            13, 1,            14, 2,          16, 4)

// get indicator values
get_indi(x)=>
    [ret1, ret2, ret3] = switch x
        0 => [rsi, naval, naval]
        1 => [cci, naval, naval]
        2 => [obv, naval, naval]
        3 => [stoch, naval, naval]
        4 => [mfi, naval, naval]
        5 => [atr, naval, naval]
        6 => [cmo, naval, naval]
        7 => [cmf, naval, naval]
        8 => [srsi, srsid, naval]
        9 => [diplus, diminus, adx]
    [ret1, ret2, ret3]

// get and show the indicator in a separate window
get_the_indicator(numofindi, colnum)=>
    [indic1, indic2, indic3] = get_indi(numofindi)
    showindicator.drawIndicator(array.get(indienabled, numofindi), array.get(indinames, numofindi), 
                                 indic1, indic2, indic3,
                                 (differentcolors ? array.from(array.get(indicolors, numofindi), array.get(indicolors, numofindi + 1), array.get(indicolors, numofindi + 2)) :indicolors), 
                                 period, 
                                 array.get(indimaxmin, numofindi * 2), array.get(indimaxmin, numofindi * 2 + 1),
                                 array.slice(indihorizontallevels, array.get(indihorizontallevelindex, numofindi * 2), array.get(indihorizontallevelindex, numofindi * 2) + array.get(indihorizontallevelindex, numofindi * 2 + 1)),
                                 array.get(precision, numofindi), 
                                 colnum * (period + 2),
                                 lnwidth)
    (array.get(indienabled, numofindi) ? 1 : 0)

// draw each indicator in separate window
int colnum = 0
int numofindi = 0
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
colnum += (get_the_indicator(numofindi, colnum)), numofindi += 1
get_the_indicator(numofindi, colnum)
