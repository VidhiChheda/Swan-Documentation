## BankNiftyWeeklyShortOptions(MinPremium)

    _SECTION_BEGIN("Price");   
    SetChartOptions(0,chartShowArrows|chartShowDates);   
    _N(Title = StrFormat("{{NAME}} - {{INTERVAL}} {{DATE}} Open %g, Hi %g, Lo %g, Close %g (%.1f%%) {{VALUES}}", O, H, L, C, SelectedValue( ROC( C, 1 ) ) ));   
    Plot( C, "Close", ParamColor("Color", colorDefault ), styleNoTitle | ParamStyle("Style") | GetPriceStyle() );    

    #include <BankNifty Weekly StrikePriceSelector(CO).afl>    

    SetOption("InitialEquity",10000000);    
    SetOption("AccountMargin",100);   
    MaxPos = 2;    
    SetOption("MaxOpenPositions",MaxPos);    
    SetTradeDelays(0,0,0,0);    
    SetOption("DisableRuinStop",True); 

    Cases = Optimize("EntryCases",0,0,14,2);   

    if(Cases == 0){Entrytime = 091500;}  
    if(Cases == 1){Entrytime = 093000;}  
    if(Cases == 2){Entrytime = 094500;}   
    if(Cases == 3){Entrytime = 100000;}   
    if(Cases == 4){Entrytime = 101500;}  
    if(Cases == 5){Entrytime = 103000;}   
    if(Cases == 6){Entrytime = 104500;}   
    if(Cases == 7){Entrytime = 110000;}  
    if(Cases == 8){Entrytime = 111500;}   
    if(Cases == 9){Entrytime = 113000;}   
    if(Cases == 10){Entrytime = 114500;}   
    if(Cases == 11){Entrytime = 120000;}   
    if(Cases == 12){Entrytime = 121500;}   
    if(Cases == 13){Entrytime = 123000;}   
    if(Cases == 14){Entrytime = 124500;}   
    if(Cases == 15){Entrytime = 130000;}   

    Exittime  = 152000;   

    bi = BarIndex();      
    exitlastbar = bi == LastValue(bi - 1);      

    DiwaliDates = DateNum()!=1111026 AND DateNum()!=1121113 AND DateNum()!=1131103 AND DateNum()!=1141023 AND DateNum()!=1151111 AND DateNum()!=1161030 AND DateNum()!=1171019 AND DateNum()!=1181107 AND DateNum()!=1191027 AND DateNum()!=1201114 AND Datenum()!=1211104;   

    PercentOfEq = Optimize("POE",2,1,5,1);  

    CallSymbol = StrLeft(Name(),20)+"CE";   
    PutSymbol = StrLeft(Name(),20)+"PE";   

    CallPremium = IIf(Type=="CE",Foreign(CallSymbol,"Close"),Null);  
    PutPremium = IIf(Type=="PE",Foreign(PutSymbol,"Close"),Null);  

    LotSize = IIf(Datenum()>=1110101 AND DateNum()<=1151029, 25 , IIf( DateNum()>=1151030 AND DateNum()<=1160630 , 30, IIf( DateNum()>=1160701 AND DateNum()<=1181025 , 40 , IIf( DateNum()>=1181026 AND DateNum()<=1200730 , 20 , 25))));   

    MaxEntries = 8;
    NSELeverage = 12;

    MISLeverage = 12;   
    Deposit = ((Foreign("$BANKNIFTY-NSE", "C")*LotSize*2)/MISLeverage);   
    MinPremium = (Deposit*(PercentOfEq/100))/LotSize;
    ActualMinPremium = (((Foreign("$BANKNIFTY-NSE", "C")*LotSize*2)/NSELeverage)*(PercentOfEq/100))/LotSize;
    DynamicCallSelection = IIF(Type == "CE",abs(CallPremium - MinPremium),Null);
    DynamicPutSelection = IIF(Type == "PE",abs(PutPremium - MinPremium),Null);
    CallFactor = IIf(CallPremium >= ActualMinPremium , 1 , CallPremium / ActualMinPremium);  
    PutFactor =  IIf(PutPremium >= ActualMinPremium , 1 , PutPremium / ActualMinPremium);  
    PositionScore = IIf(Type =="CE" AND StrikePriceSelector == 1 ,100000 - DynamicCallSelection,IIf(Type =="PE" AND StrikePriceSelector == 1 ,100000 - DynamicPutSelection,Null)); 
    PositionSize = IIf(Type == "CE" , -CallFactor*PercentOfEq , -PutFactor*PercentOfEq);  

    Short =  StrikePriceSelector == 1 AND TimeNum()==Entrytime AND DiwaliDates;   
    ShortPrice = Close;   

    Cover = TimeNum()==Exittime OR exitlastbar OR DateNum()!=Ref(DateNum(),1);    
    CoverPrice = Close;   

    Short = ExRem(Short,Cover);    
    Cover = ExRem(Cover,Short);   

    stop = IIf(DayOfWeek()==4,50,IIf(DayOfWeek()==3,40,IIf(DayOfWeek()==2,40,IIf(DayOfWeek()==1,30,IIf(DayOfWeek()==5,30,50))))); 
    ApplyStop(stopTypeLoss,stopModePercent,stop,1);

    PlotShapes(Short*shapeDownArrow , colorWhite , 0 , H , -10);    
    PlotShapes(Cover*shapeUpArrow , colorWhite , 0 , L , -10);   

    SetCustomBacktestProc("");   

    Limit = 1; 
    if(Status("action") == actionPortfolio)   
    {   

       bo = GetBacktesterObject();   
       bo.PreProcess();  
       BankNiftyClose = Foreign("$BANKNIFTY-NSE", "C" );     

       for( i = 0; i<BarCount; i++)   

       {     
        CurrentPortfolioEquity = 10000000;

        CallCount = 0; 
        PutCount = 0;                       

        for(sig = bo.GetFirstSignal(i) ; sig ; sig = bo.GetNextSignal(i) )   

        {   
          Type = StrRight(sig.Symbol,2); 

          if (sig.IsEntry()  AND Type == "PE")   
          {   

          if(PutCount < Limit ) 
              PutCount++; 
          else 
              sig.Price = -1;

          }  

          if (sig.IsEntry()  AND Type == "CE")   
          {   

           if(CallCount < Limit ) 
              CallCount++; 
           else 
              sig.Price = -1;

          }  

        }              

        bo.ProcessTradeSignals(i); 




        bo.UpdateStats(i, 1);   
        bo.UpdateStats(i, 2);  





        }   

       bo.PostProcess();   

    }    


    PortEquity = Foreign("~~~EQUITY", "C" );   

    Filter = 1; 
    AddColumn(PortEquity,"PortEquity",1);

    _SECTION_END();   


