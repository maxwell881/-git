# git使用说明

//+------------------------------------------------------------------+
//|                                       ea-mytest2-xau-duokong.mq4 |
//|                        Copyright 2021, MetaQuotes Software Corp. |
//|                                             https://www.mql5.com |
//+------------------------------------------------------------------+
#property copyright "Copyright 2021, MetaQuotes Software Corp."
#property link      "https://www.mql5.com"
#property version   "1.00"
#property strict
//--- input parameters
input int      MagicNum=12;
input double   lot=0.01;
input int      MaxDanShu=15;
input int      止盈点数=1000;
input int      加仓点数=1000;
//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
  {
//---
   
//---
   Print("开始智能交易");
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Expert deinitialization function                                 |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
//---
  Print("结束智能交易"); 
  }
//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
  {
//---
  //仓位判断
  int buy_count=0;
  double last_buy_open_price=0;
  double last_buy_open_lot = 0;
  
  int sell_count=0;
  double last_sell_open_price=0;
  double last_sell_open_lot = 0;
  
  for(int i=0;i<OrdersTotal();i++)
   {
     if(OrderSelect(i,SELECT_BY_POS) && OrderSymbol()==Symbol() && OrderMagicNumber()==MagicNum)
      {
        if(OrderType()==OP_BUY)
         {
           buy_count = buy_count + 1;
           last_buy_open_price = OrderOpenPrice();
           last_buy_open_lot = OrderLots();
         }
         
        if(OrderType()==OP_SELL)
         {
           sell_count = sell_count + 1;
           last_sell_open_price = OrderOpenPrice();
           last_sell_open_lot = OrderLots();
         }
      }
   }
   
  
  
  //开仓
  if(buy_count==0)
   {
     //开第一单
     double stoploss = 0;
     double takeprofit = 0;
     //double takeprofit = NormalizeDouble(Ask+止盈点数*Point,Digits); 
     int ticket=OrderSend(Symbol(),OP_BUY,lot,Bid,3,stoploss,takeprofit,"buy-"+IntegerToString(buy_count+1),MagicNum,0,clrNONE); 
     
   }
   else
   {
     if(last_buy_open_price-Ask>=加仓点数*Point)
     { //加仓
       double stoploss = 0;
       double takeprofit = 0;
       int ticket = OrderSend(Symbol(),OP_BUY,last_buy_open_lot*2,Bid,3,stoploss,takeprofit,"buy-"+IntegerToString(buy_count+1),MagicNum,0,clrNONE);
     }
   }
  
  //平仓 倒数第二单 盈利为正 全部平仓
  
  
  if(buy_count==1)
  {
   //平买单
   if(Bid-last_buy_open_price>止盈点数*Point)
     {
       CloseAllBuy();
     }
  }
  else if(buy_count>=2)
   {
    
    if(GetProfitFromLastBuy2ticket()>=0)
      {//平仓全部
       CloseAllBuy();
      }
   } 
   
   
   //开卖单和平卖单
   //开仓
  if(sell_count==0)
   {
     //开第一单
     double stoploss = 0;
     double takeprofit = 0;
     //double takeprofit = NormalizeDouble(Ask+止盈点数*Point,Digits); 
     int ticket=OrderSend(Symbol(),OP_SELL,lot,Ask,3,stoploss,takeprofit,"sell-"+IntegerToString(sell_count+1),MagicNum,0,clrNONE); 
     
   }
   else
   {
     if(Bid-last_sell_open_price>=加仓点数*Point)
     { //加仓
       double stoploss = 0;
       double takeprofit = 0;
       int ticket = OrderSend(Symbol(),OP_SELL,last_sell_open_lot*2,Ask,3,stoploss,takeprofit,"sell-"+IntegerToString(sell_count+1),MagicNum,0,clrNONE);
     }
   }
  
  //平仓 倒数第二单 盈利为正 全部平仓
  
  
  if(sell_count==1)
  {
   //平买单
   if(last_sell_open_price-Ask>=止盈点数*Point)
     {
       CloseAllSell();
     }
  }
  else if(buy_count>=2)
   {
    
    if(GetProfitFromLastSell2ticket()>=0)
      {//平仓全部
       CloseAllSell();
      }
   } 
  
  
  
   
 }
//+------------------------------------------------------------------+

double GetProfitFromLastBuy2ticket()
 {
   int cc=0;
   //int last2ticket;
   for(int i = OrdersTotal()-1;i>=0;i--)
    {
     if(OrderSelect(i,SELECT_BY_POS) && OrderSymbol()==Symbol() && OrderMagicNumber()==MagicNum)
      {
        if(OrderType()==OP_BUY)
         {
          cc+=1;
          if(cc==2)
            { 
             return OrderProfit();            
            }
         }
       }   
    }
   return 0;
 }

void CloseAllBuy()
   {
    //平掉所有buy单  or sell 单
     for(int ii=0;ii<OrdersTotal();ii++)
      {
        if(OrderSelect(ii,SELECT_BY_POS) && OrderSymbol()==Symbol() && OrderMagicNumber()==MagicNum)
         {
           if(OrderType()==OP_BUY)
            {
              Print("订单号：",OrderTicket());
              bool ti = OrderClose(OrderTicket(),OrderLots(),Ask,3,clrNONE);
            }     
         }
      }
   }
   
double GetProfitFromLastSell2ticket()
 {
   int ccc=0;
   //int last2ticket;
   for(int i = OrdersTotal()-1;i>=0;i--)
    {
     if(OrderSelect(i,SELECT_BY_POS) && OrderSymbol()==Symbol() && OrderMagicNumber()==MagicNum)
      {
        if(OrderType()==OP_SELL)
         {
          ccc+=1;
          if(ccc==2)
            { 
             return OrderProfit();            
            }
         }
       }   
    }
   return 0;
 }

void CloseAllSell()
   {
    //平掉所有buy单  or sell 单
     for(int j=0;j<OrdersTotal();j++)
      {
        if(OrderSelect(j,SELECT_BY_POS) && OrderSymbol()==Symbol() && OrderMagicNumber()==MagicNum)
         {
           if(OrderType()==OP_SELL)
            {
              Print("订单号：",OrderTicket());
              bool ti = OrderClose(OrderTicket(),OrderLots(),Bid,3,clrNONE);
            }     
         }
      }
   }
