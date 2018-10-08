# Agile new added features
Practical
import matplotlib.pyplot as plt
from matplotlib import style
style.use('ggplot')
import numpy as np
import csv 
import pandas as pd
from sklearn import svm

#enddate='2018-02-16'
#startdate='2008-02-19'
#dates=pd.date_range(startdate,enddate)
#df2=pd.DataFrame(index=dates)

df=pd.read_csv('aapl.csv')
#1.Calculation of Moving Average
mov_avg=[]
    
for i in range(0,2508):
    if(df['Close'][i]>=df['Close'][i:i+10].sum()/10):
        mov_avg.append(1)
    else:
        mov_avg.append(-1)
for i in range(2508,2518):
    mov_avg.append(0)
    
    
df2=pd.DataFrame(mov_avg)  
df=df.join(df2)
df.columns=['Date','Open','High','Low','Close','Volume','Moving Average']

#2.Calculation of Momentum
mom=[]
for i in range(0,2508):
    #print(df['Close'][i]-df['Close'][i-10])
    if(df['Close'][i]-df['Close'][i+10]>=0):
        mom.append(1)
    else:
        mom.append(-1)

for i in range(2508,2518):
    mom.append(0)
    
df1=pd.DataFrame(mom)  
df=df.join(df1)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum']

#3.Calculation of CCI

#Calculation of Typical Prices((H+L+C)/3)
TP=[]
for i in range(0,2518):
    #print(df['Close'][i]-df['Close'][i-10])
    TP.append((df['Close'][i]+df['High'][i]+df['Low'][i])/3)

df4=pd.DataFrame(TP)
df4.columns=['TP']
#Calculation Of Moving Average for 20 day Period

mov_avg_20=[]

for i in range(0,2498):
    mov_avg_20.append(df4['TP'][i:i+20].sum()/20)

for i in range(2498,2518):
    mov_avg_20.append(0)
        
#Difference Array for absolute diff between Typical Price and Moving Average
    
diff=[]
for i in range(0,2518):
    diff.append(abs(df4['TP'][i]-mov_avg_20[i]))
    
df5=pd.DataFrame(diff)
df5.columns=['diff']
    
#Calculation of Mean Deviation Of 20 day back Window
mean_dev=[]

for i in range(0,2498):
    mean_dev.append(df5['diff'][i:i+20].sum()/20)
#Calculation For CCI

CCI=[]
for i in range(0,2497):
    x=(df4['TP'][i]-mov_avg_20[i])/(0.015*mean_dev[i])
    x2=(df4['TP'][i+1]-mov_avg_20[i+1])/(0.015*mean_dev[i+1])
    if(x>=200):
        CCI.append(-1)
    elif(x<=-200):
        CCI.append(1)
    elif(x-x2>=0):
        CCI.append(1)
    else:
        CCI.append(-1)
for i in range(2497,2518):
    CCI.append(0)
df6=pd.DataFrame(CCI)
df=df.join(df6)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI']

#print(df)

#4.Calculation Of LWR

lwr=[]
for i in range(0,2504):
    lwr.append(-100*(df['High'][i:i+14].max()-df['Close'][i])/(df['High'][i:i+14].max()-df['Low'][i:i+14].min()))

for i in range(2504,2518):
    lwr.append(0)
    
df2=pd.DataFrame(lwr)  
df=df.join(df2)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%']

#5.Calculation Of Accumulation Distribution Line

ADL=[]
    
for i in range(2517,0,-1):
    MFM=(((df['Close'][i]-df['Low'][i])-(df['High'][i]-df['Close'][i]))/(df['High'][i]-df['Low'][i])) #Money Flow Multiplier
    MFV=MFM*(df['Volume'][i]) #Money Flow Volume
    if(i==2517):
        ADL.append(MFV)
    else:
        ADL.append(ADL[2518-i-2]+MFV)
    
ADL.append(0)
      
#print(df7)
ADLCompute=[]
for i in range(2517,1,-1):
    if(ADL[i]>=ADL[i-1]):
        ADLCompute.append(-1)
    else:
        ADLCompute.append(1)

df7=pd.DataFrame(ADLCompute)
df=df.join(df7)
#print(df7)
df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%','ADI']

#6.Stockastic K%

K=[]
for i in range(0,2504):
    K.append(100*(df['Close'][i]-df['Low'][i:i+14].min())/(df['High'][i:i+14].max()-df['Low'][i:i+14].min()))

for i in range(2504,2518):
    K.append(0)
    
df2=pd.DataFrame(K)  
df=df.join(df2)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%','ADI','Stochastic K%']

#7.Stochastic D%

D=[]
for i in range(0,2515):
    D.append(df['Stochastic K%'][i:i+3].sum()/3)

for i in range(2515,2518):
    D.append(0)
    
df2=pd.DataFrame(D)  
df=df.join(df2)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%','ADI','Stochastic K%','Stochastic D%']

#8.Calculation Of RSI
RSI=[]

AvgG=0
AvgL=0
for i in range(2504,2517):                      #Average Gain For First 14-day window
    change=df['Close'][i+1]-df['Close'][i]
    if change > 0:
        AvgG+=change
    else:
        AvgL+=abs(change)
        
RS=(AvgG/AvgL)
RSI.append(100-(100/(1+RS)))

for i in range(2504,0,-1):                #Average Gain for subsequent days
    Current_Gain=0
    Current_Loss=0
    change=df['Close'][i+1]-df['Close'][i]
    if change > 0:
        Current_Gain=change
    else:
        Current_Loss=abs(change)
    AvgG=((AvgG*13)+Current_Gain)/14
    AvgL=((AvgL*13)+Current_Loss)/14
    RS=(AvgG/AvgL)
    RSI.append(100-(100/(1+RS)))
for i in range(0,14):
    RSI.append(0)
RSI2=[]
for i in range(0,2517):   
    if(RSI[i]>70):
        RSI2.append(-1)
    elif(RSI[i]<30):
        RSI2.append(1)
    elif(RSI[i]-RSI[i+1]>=0):
        RSI2.append(1)
    else:
        RSI2.append(-1)
RSI2.append(0)
df2=pd.DataFrame(RSI2)  
df=df.join(df2)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%','ADI','Stochastic K%','Stochastic D%','RSI']

#9.Weighted Moving Average
Wmov_avg=[]

for i in range(0,2508):
    sum=0
    for j in range(0,10):
        sum+=(j+1)*df['Close'][i+j]
    Wmov_avg.append(sum/55)

for i in range(2508,2518):
    Wmov_avg.append(0)
    
WMA=[]
for i in range(0,2518):
    if(df['Close'][i]<=Wmov_avg[i]):
        WMA.append(-1)
    else:
        WMA.append(1)
df2=pd.DataFrame(WMA)  
df=df.join(df2)

df.columns=['Date','Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%','ADI','Stochastic K%','Stochastic D%','RSI','Weighted Moving Average']

#10.Calculation Of MACD

EMA_12=(df['Close'][0:12].sum()/12)
EMA_26=(df['Close'][0:26].sum()/26)

M1=(2/(12+1))
M2=(2/(26+1))

MACD=[]
for i in range(0,26):
    MACD.append(0)
    
for i in range(26,2518):
    EMA_12=((df['Close']-EMA_12)*M1) + EMA_12
    EMA_26=((df['Close']-EMA_26)*M2) + EMA_26
    MACD.append(EMA_12-EMA_26)
    
df2=pd.DataFrame(MACD)  
df=df.join(df2)
df=df.set_index('Date')

df.columns=['Open','High','Low','Close','Volume','Moving Average','Momentum','CCI','Larry Williams R%','ADI','Stochastic K%','Stochastic D%','RSI','Weighted Moving Average','MACD']

print(df)
df.to_csv('TI3.csv')
print(df)
