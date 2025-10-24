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
//|                                           Copyright 2025, FaceND |
//|                             https://github.com/FaceND/Price_Line |
//+------------------------------------------------------------------+
#property copyright     "Copyright 2025, FaceND"
#property link          "https://github.com/FaceND/Price_Line"
#property version       "1.5"
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
 ENABLE  = 1, // Enable
 DISABLE = 0  // Disable
};

input group "PRICE"
input ENUM_STATUS           PStatus          = ENABLE;            // Price line
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

bool PriceStatus, HighLowStatus;
color priceColor, _ColorBull, _ColorBear;
//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
  {
   PriceStatus   = PStatus;
   HighLowStatus = HLStatus;
   SetPriceColor();
   
   UpdatePrice();
   UpdateHighLow();
   return INIT_SUCCEEDED;
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
   if(HighLowStatus)
     {
      ObjectDelete(0, HIGH_LINE_NAME);
      ObjectDelete(0, LOW_LINE_NAME);
      EventKillTimer();
     }
   ChartSetInteger(0, CHART_SHOW_BID_LINE, true);
   ChartRedraw();
  }
//+------------------------------------------------------------------+
//| Custom indicator Timer function                                  |
//+------------------------------------------------------------------+
void OnTimer()
  {
   UpdateHighLow();
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
   if(rates_total > 0)
     {
      UpdatePrice();
     }
   return rates_total;
  }
//+------------------------------------------------------------------+
//| Function to set the price color                                  |
//+------------------------------------------------------------------+
void SetPriceColor() 
  {
   if(HighLowStatus)
     {
      if(ColorHigh == clrNONE && ColorLow == clrNONE)
        {
         HighLowStatus = false;
        }
     }
   if(PriceStatus)
     {
      switch(ColorType)
        {
         //-----------------------[ Candle Type ]--------------------+
         case COLOR_CANDLE:
              {
               _ColorBull = (color)ChartGetInteger(0, CHART_COLOR_CANDLE_BULL);
               _ColorBear = (color)ChartGetInteger(0, CHART_COLOR_CANDLE_BEAR);
               break;
              }
         //------------------------[ Bar Type ]----------------------+
         case COLOR_BAR:
              {
               _ColorBull = (color)ChartGetInteger(0, CHART_COLOR_CHART_UP);
               _ColorBear = (color)ChartGetInteger(0, CHART_COLOR_CHART_DOWN);
               break;
              }
         //-----------------------[ Custom Type ]--------------------+
         case COLOR_CUSTOM:
              {
               if(ColorBull == clrNONE || ColorBear == clrNONE)
                 {
                  if(ColorBull != clrNONE && ColorBear == clrNONE)
                    {
                     priceColor = ColorBull;
                    }
                  else if(ColorBear != clrNONE && ColorBull == clrNONE)
                    {
                     priceColor = ColorBear;
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
         //----------------------------------------------------------+
         default:
           {
            _ColorBull = ColorBull;
            _ColorBear = ColorBear;
           break;
          }
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
   ObjectSetString (0, name, OBJPROP_TOOLTIP,     "\n");
  }
//+------------------------------------------------------------------+
//| Function to update the horizon line as price and line color      |
//+------------------------------------------------------------------+
bool UpdateLine(const string name, const double price, const color line_color)
  {
   if(ObjectFind(0, name) == 0)
     {
      ObjectMove(0, name, 0, 0, price);
      ObjectSetInteger(0, name, OBJPROP_COLOR, line_color);
      return true;
     }
   return false;
  }
//+------------------------------------------------------------------+
//| Function to update Price line as current price                   |
//+------------------------------------------------------------------+
void UpdatePrice(double price = NULL, const color line_color = NULL)
  {
   if(PriceStatus)
     {
      if(!price || !line_color)
        {
         double open;
         //+------------------------------------------------------------+
         open  = iOpen(_Symbol, _Period, 0);
         price = iClose(_Symbol, _Period, 0);
         //+------------------------------------------------------------+
         const double price_diff =  open - price;
         if(price_diff < _Point)
           {
            priceColor = _ColorBull;
           }
         else if(price_diff > _Point)
           {
            priceColor = _ColorBear;
           }
        }
      if(!UpdateLine(PRICE_LINE_NAME, price, priceColor))
        {
         CreateLine(PRICE_LINE_NAME, PriceStyle, PriceWidth);
         UpdatePrice(price, priceColor);

         ChartSetInteger(0, CHART_SHOW_BID_LINE, false);
        }
     }
  }
//+------------------------------------------------------------------+
//| Function to update high and low line as min and max price        |
//+------------------------------------------------------------------+
void UpdateHighLow(double high_price = NULL, double low_price = NULL)
  {
   if(HighLowStatus)
     {
      if(!high_price || !low_price)
        {
         int start = (int)ChartGetInteger(0, CHART_FIRST_VISIBLE_BAR);
         int count = (int)ChartGetInteger(0, CHART_VISIBLE_BARS);

         int highIndex = iHighest(_Symbol, _Period, MODE_HIGH, count, start - count + 1);
         int lowIndex  = iLowest(_Symbol, _Period, MODE_LOW, count, start - count + 1);
         //+---------------------------------------------------------+
         high_price = iHigh(_Symbol, _Period, highIndex);
         low_price  = iLow(_Symbol, _Period, lowIndex);
         //+---------------------------------------------------------+
        }
      if(!UpdateLine(HIGH_LINE_NAME, high_price, ColorHigh)||
         !UpdateLine(LOW_LINE_NAME,  low_price, ColorLow))
        {
         EventKillTimer();

         CreateLine(HIGH_LINE_NAME, HLStyle, HLWidth);
         CreateLine(LOW_LINE_NAME, HLStyle, HLWidth);
         UpdateHighLow(high_price, low_price);

         EventSetMillisecondTimer(250);
        }
      ChartRedraw();
     }
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
