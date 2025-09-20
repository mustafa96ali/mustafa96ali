## Hi there 👋

<!--
**mustafa96ali/mustafa96ali** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
//+------------------------------------------------------------------+
//|                                                   TBR Indicator  |
//|                                      Improved by Programming Guru|
//+------------------------------------------------------------------+
#property indicator_separate_window
#property indicator_buffers 4
#property indicator_plots   4
#property indicator_type1   DRAW_LINE
#property indicator_type2   DRAW_LINE
#property indicator_type3   DRAW_ARROW
#property indicator_type4   DRAW_ARROW
#property indicator_color1  clrBlue
#property indicator_color2  clrRed
#property indicator_color3  clrGreen
#property indicator_color4  clrOrange

// The inputs of the indicator
input int L = 20;               // TBR period
input int RSIPeriod = 14;       // RSI period
input int RSIThreshold = 50;    // RSI threshold (50 for trend)

// Indicator buffer variables
double TBR_R[]; // TBR-Negative buffer
double TBR_S[]; // TBR-Positive buffer
double BuyArrowBuffer[];  // Buy arrow buffer
double SellArrowBuffer[]; // Sell arrow buffer

//+------------------------------------------------------------------+
//| Initialization function of the custom indicator                  |
//+------------------------------------------------------------------+
int OnInit()
{
   // Set indicator buffers
   SetIndexBuffer(0, TBR_R, INDICATOR_DATA);
   SetIndexBuffer(1, TBR_S, INDICATOR_DATA);
   SetIndexBuffer(2, BuyArrowBuffer, INDICATOR_DATA);
   SetIndexBuffer(3, SellArrowBuffer, INDICATOR_DATA);
   
   // Set the arrow codes using PlotIndexSetInteger
   PlotIndexSetInteger(2, PLOT_ARROW, 233); // Up arrow (Buy)
   PlotIndexSetInteger(3, PLOT_ARROW, 234); // Down arrow (Sell)

   // Set a short name for the indicator buffers
   PlotIndexSetString(0, PLOT_LABEL, "TBR-R");
   PlotIndexSetString(1, PLOT_LABEL, "TBR-S");
   PlotIndexSetString(2, PLOT_LABEL, "Buy Signal");
   PlotIndexSetString(3, PLOT_LABEL, "Sell Signal");
   
   // We will not show the signal buffers in the data window
   PlotIndexSetInteger(2, PLOT_DRAW_BEGIN, L + RSIPeriod + 1);
   PlotIndexSetInteger(3, PLOT_DRAW_BEGIN, L + RSIPeriod + 1);

   // We need to set the shift for the arrows
   PlotIndexSetDouble(2, PLOT_ARROW_SHIFT, -10); // Shift the arrow down
   PlotIndexSetDouble(3, PLOT_ARROW_SHIFT, 10);  // Shift the arrow up

   // Return success
   return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Calculation function of the custom indicator                     |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
{
   // Make sure we have enough data to calculate
   if (rates_total < L + RSIPeriod) return 0;
   
   // Start calculation from the last calculated bar or from the start
   int start = prev_calculated;
   if (start < 1) start = 1;

   // Loop through all the bars to calculate the indicator values
   for (int i = start; i < rates_total; i++)
   {
      // Reset the arrow buffers for the current bar
      BuyArrowBuffer[i] = EMPTY_VALUE;
      SellArrowBuffer[i] = EMPTY_VALUE;
      
      // Calculate TBR-R and TBR-S
      TBR_R[i] = close[i] - close[i - L];
      TBR_S[i] = close[i] - close[i + L];
      
      // Get the RSI and MA values directly for the current bar
      double rsi_value = iRSI(_Symbol, _Period, RSIPeriod, ENUM_APPLIED_PRICE::PRICE_CLOSE, i);
      double ma_value = iMA(_Symbol, _Period, L, 0, MODE_SMA, ENUM_APPLIED_PRICE::PRICE_CLOSE, i);
      
      // Check for buy signal
      if (TBR_R[i] > 0 && TBR_R[i-1] <= 0 && close[i] > ma_value && rsi_value > RSIThreshold)
      {
         // Print a buy arrow below the candle
         BuyArrowBuffer[i] = low[i];
      }
      
      // Check for sell signal
      if (TBR_S[i] < 0 && TBR_S[i-1] >= 0 && close[i] < ma_value && rsi_value < (100 - RSIThreshold))
      {
         // Print a sell arrow above the candle
         SellArrowBuffer[i] = high[i];
      }
   }

   // Return the number of calculated bars
   return(rates_total);
}
