# Price Line
This MetaTrader 5 (MQL5) indicator displays a **real-time price line** and **visible High & Low lines** on the chart, with customizable styles and logic based on candle direction or bar color.


## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Installation](#installation)
- [Inputs](#inputs)
- [Inputs](#inputs)
- [Customization](#customization)
- [Script Code](#script-code)
- [Contributing](#contributing)
- [License](#license)


## Overview
This indicator enhances chart visuals by:
- Drawing a **price line** at the current price.
- Coloring the line based on candle or bar direction or using custom colors.
- Optionally showing the **highest** and **lowest** prices visible on the chart.

It helps traders instantly evaluate market momentum and range without complex indicators.


## Features
- Price line updates in real-time.
- Choose coloring logic: candle-based, bar-based, or fully custom.
- Automatically detects visible High & Low from chart view.
- Fully customizable line color, width, and style.


## Installation
1. Open MetaTrader 5.
2. Go to `File` → `Open Data Folder`.
3. Navigate to `MQL5/Indicators/`.
4. Copy the indicator `.mq5` file into the folder.
5. Restart MT5 or refresh the Navigator.
6. Drag the indicator onto any chart.


## Inputs

### 🔹 PRICE Group

| Input                    | Description                                                           |
|--------------------------|-----------------------------------------------------------------------|
| `Price colors setting`   | Determines color logic:                                               |
|                          | - **Candle colors**: Based on candle open vs close                    |
|                          | - **Bar colors**: Based on bar direction (up/down)                    |
|                          | - **Custom colors**: Uses custom colors below                         |
| `Bullish line color`     | Color when price is above open (Custom mode only)                     |
| `Bearish line color`     | Color when price is below open (Custom mode only)                     |
| `Line style`             | Style of price line (e.g., solid, dashed, dotted)                     |
| `Line width`             | Thickness of price line                                               |

### 🔹 HIGH & LOW Group

| Input                 | Description                                      |
|-----------------------|--------------------------------------------------|
| `High & Low line`     | Enable/Disable drawing High & Low lines          |
| `High line color`     | Line color for highest high                      |
| `Low line color`      | Line color for lowest low                        |
| `Line style`          | Style for High & Low lines                       |
| `Line width`          | Thickness of High & Low lines                    |


## Customization
You can customize the name of the object by modifying the following text in the script.
```mql5
#define PRICE_LINE_NAME "Price-Line"
#define HIGH_LINE_NAME  "Highest-Line"
#define LOW_LINE_NAME   "Lowest-Line"
```


## Script Code
Below is the MQL5 code used to create the "Price Line"
```mql5
//+------------------------------------------------------------------+
//|                                                   Price Line.mq5 |
//|                                                        Delta.mq5 |
//|                                           Copyright 2025, FaceND |
//|                             https://github.com/FaceND/Price_Line |
//+------------------------------------------------------------------+
#property copyright     "Copyright 2025, FaceND"
#property link          "https://github.com/FaceND/Price_Line"
#property version       "1.0"
#property description   "Displays a real-time price line at the current price."
#property description   "Colors the price line based on candle direction (bullish or bearish)."
#property description   "Also draws highest high and lowest low lines visible in the chart."
#property description   "Supports custom color settings and line styling."
#property strict
#property indicator_chart_window
#property indicator_plots 0

enum ENUM_TYPE_COLOR
{
 COLOR_CANDLE, // Candle colors
 COLOR_BAR,    // Bar colors
 COLOR_CUSTOM  // Custom colors
};

enum ENUM_STATUS
{
 ENABLE,  // Enable
 DISABLE  // Disable
};

input group "PRICE"
input ENUM_TYPE_COLOR       ColorType        = COLOR_CANDLE;      // Select Price line colors
input color                 ColorBull        = clrLime;           // Bullish line color (Custom)
input color                 ColorBear        = clrRed;            // Bearish line color (Custom)
input ENUM_LINE_STYLE       PriceStyle       = STYLE_SOLID;       // Line style
input int                   PriceWidth       = 1;                 // Line width

input group "HIGH & LOW"
input ENUM_STATUS           HLStatus         = ENABLE;            // High & Low line
input color                 ColorHigh        = clrDodgerBlue;     // High line color
input color                 ColorLow         = clrOrange;         // Low line color
input ENUM_LINE_STYLE       HLStyle          = STYLE_SOLID;       // Line style
input int                   HLWidth          = 1;                 // Line width

#define PRICE_LINE_NAME "Price-Line"
#define HIGH_LINE_NAME  "Highest-Line"
#define LOW_LINE_NAME   "Lowest-Line"

bool PriceStatus = true;
bool PriceSet = true;

color priceColor, _ColorBull, _ColorBear;
//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
  {
   SetPriceColor();
   if(PriceStatus)
     {
      CreateLine(PRICE_LINE_NAME, PriceStyle, PriceWidth);
     }
   if(HLStatus == ENABLE)
     {
      CreateLine(HIGH_LINE_NAME, HLStyle, HLWidth);
      CreateLine(LOW_LINE_NAME, HLStyle, HLWidth);
     }
   ChartSetInteger(0, CHART_SHOW_BID_LINE, false);
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Custom indicator deinitialization function                       |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
   if(PriceStatus)
     {
      ObjectDelete(0, PRICE_LINE_NAME);      
     }
   if(HLStatus == ENABLE)
     {
      ObjectDelete(0, HIGH_LINE_NAME);
      ObjectDelete(0, LOW_LINE_NAME);
     }
   ChartSetInteger(0, CHART_SHOW_BID_LINE, true);
   ChartRedraw();
  }
//+------------------------------------------------------------------+
//| Chart Event Handler                                              |
//+------------------------------------------------------------------+
void OnChartEvent(const int                 id,
                  const long           &lparam,
                  const double         &dparam,
                  const string         &sparam)
  {
   if(HLStatus == ENABLE)
     {
      switch(id)
        {
         case CHARTEVENT_CHART_CHANGE:
         case CHARTEVENT_MOUSE_WHEEL:
           {
            UpdateHighLow();
            break;
           }
        }
     }
   }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int           rates_total,
                const int       prev_calculated,
                const datetime          &time[],
                const double            &open[],
                const double            &high[],
                const double             &low[],
                const double           &close[],
                const long       &tick_volume[],
                const long            &volume[],
                const int             &spread[])
  {
   if(rates_total > 0 && PriceStatus)
     {
      //+------------------------------------------------------------+
      const double open  = open[rates_total-1];
      const double close = close[rates_total-1];
      //+------------------------------------------------------------+
      if(PriceSet)
        {
         const double price_diff =  open - close;
         if(price_diff < _Point * 0.1)
           {
            priceColor = _ColorBull;
           }
         else if(price_diff > _Point * 0.1)
           {
            priceColor = _ColorBear;
           }
        }
      UpdateLine(PRICE_LINE_NAME, close, priceColor);
      if(HLStatus == ENABLE)
        {
         UpdateHighLow();
        }
     }
   return(rates_total);
  }
//+------------------------------------------------------------------+
//| Function to set the price color                                  |
//+------------------------------------------------------------------+
void SetPriceColor() 
  {
   switch(ColorType)
     {
      //-------------------------------------------------------------+
      case COLOR_CANDLE:
           {
            _ColorBull = (color)ChartGetInteger(0, CHART_COLOR_CANDLE_BULL);
            _ColorBear = (color)ChartGetInteger(0, CHART_COLOR_CANDLE_BEAR);
            break;
           }
      //-------------------------------------------------------------+
      case COLOR_BAR:
           {
            _ColorBull = (color)ChartGetInteger(0, CHART_COLOR_CHART_UP);
            _ColorBear = (color)ChartGetInteger(0, CHART_COLOR_CHART_DOWN);
            break;
           }
      //-------------------------------------------------------------+
      case COLOR_CUSTOM:
           {
            if(ColorBull == clrNONE || ColorBear == clrNONE)
              {
               if(ColorBull != clrNONE && ColorBear == clrNONE)
                 {
                  priceColor = ColorBull;
                  PriceSet = false;
                 }
               else if(ColorBear != clrNONE && ColorBull == clrNONE)
                 {
                  priceColor = ColorBear;
                  PriceSet = false;
                 }
               else if(ColorBull == clrNONE && ColorBear == clrNONE)
                 {
                  PriceStatus = false;
                 }
              }
            else
              {
               _ColorBull = ColorBull;
               _ColorBear = ColorBear;
              }
            break;
           }
      //-------------------------------------------------------------+
      default:
        {
         _ColorBull = ColorBull;
         _ColorBear = ColorBear;
        break;
       }
     }
  }
//+------------------------------------------------------------------+
//| Function to create the horizon line                              |
//+------------------------------------------------------------------+
void CreateLine(const string name, const ENUM_LINE_STYLE line_style, const int line_width)
  {
   if(ObjectFind(0, name) < 0)
     {
      ObjectCreate(0, name, OBJ_HLINE, 0, 0, 0);
     }
   ObjectSetInteger(0, name, OBJPROP_STYLE, line_style);
   ObjectSetInteger(0, name, OBJPROP_WIDTH, line_width);
  }
//+------------------------------------------------------------------+
//| Function to update the horizon line as price and line color      |
//+------------------------------------------------------------------+
void UpdateLine(const string name, const double price, const color line_color)
  {
   if(ObjectFind(0, name) == 0)
     {
      ObjectSetDouble(0, name, OBJPROP_PRICE, price);
      ObjectSetInteger(0, name, OBJPROP_COLOR, line_color);
     }
  }
//+------------------------------------------------------------------+
//| Function to update high and low line as min and max price        |
//+------------------------------------------------------------------+
void UpdateHighLow()
  {
   int start = (int)ChartGetInteger(0, CHART_FIRST_VISIBLE_BAR);
   int count = (int)ChartGetInteger(0, CHART_VISIBLE_BARS);
   
   int highIndex = iHighest(_Symbol, _Period, MODE_HIGH, count, start - count + 1);
   int lowIndex  = iLowest(_Symbol, _Period, MODE_LOW, count, start - count + 1);
   
   double max_price = iHigh(_Symbol, _Period, highIndex);
   double min_price = iLow(_Symbol, _Period, lowIndex);
   
   UpdateLine(HIGH_LINE_NAME, max_price, ColorHigh);
   UpdateLine(LOW_LINE_NAME, min_price, ColorLow);
  }
//+------------------------------------------------------------------+
```


## Contributing

Want to improve this indicator?

1. Fork this repository.
2. Create your feature branch: `git checkout -b feature/YourFeature`.
3. Commit your changes.
4. Push to the branch: `git push origin feature/YourFeature`.
5. Open a Pull Request.


## License

This project is open-source under the [MIT License](LICENSE). Feel free to use, modify, and distribute.
