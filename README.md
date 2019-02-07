

```python
%matplotlib ipympl
%load_ext autoreload
%autoreload 2
%run ./simlib/init.py
#imp.reload(simlib)
#import os
cwd = os.getcwd()
#%aimport simlib.SensorSim
from simlib.SensorSim import gSimData, rSimData, doSim, gen_df_sim_var, fig_AD_plt,fig_SNR_plt, PrintSignalNoise
from simlib.buildtest import dict_overview, replace_all
```


```python
%aimport
```

    Modules to reload:
    all-except-skipped
    
    Modules to skip:
    
    

# Sensor Algo


```python
blen = 10e3
S1 = O200GP.c_sensor_h5(N = blen)
```


```python
for dc in S1.s_SimData:
    print(dc, S1.s_SimData[dc])
```

    c_RefFile 
    f32_RefDist_min 0.0
    f32_RefDist_max 0.0
    u32_SimSamples 100000
    f32_AlineNoiseFactor 0.0
    u32_NoiseSamples 0
    u32_NoiseOffset 0
    f32_TeachDist 0.0
    af32_AlineNoise [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
    f32_RefDist 0.0
    f32_Dist_xc 15.0
    f32_dt 9.999999747378752e-05
    f32_t 0.0
    as16_AlineSignal [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    u32_events 0
    u32_state_changes 0
    u32_updates 0
    

# Simulationsdaten


```python
S1_Data = SimData.SimData()
SimSensRefFile = '../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm.h5'
S1_Data.dRefFile(SimSensRefFile)
S1_Data.disable_print_f()
```

    
    => dRefFile()
    ['./../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm.h5']
    Info: _print_f() disabled!
    (Enable with enable_print_f()
    

## - Generieren


```python
gSimData(S1_Data,S1)
```

    
    => gSimFile(Filename = ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_wt_simdata, target = wt, Refx_arr = (...))
    Generation of simulation file ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_wt_simdata.h5 successful.
    
    => gSimFile(Filename = ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_gr_simdata, target = gr, Refx_arr = (...))
    Generation of simulation file ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_gr_simdata.h5 successful.
    
    => gSimFile(Filename = ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_bl_simdata, target = bl, Refx_arr = (...))
    Generation of simulation file ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_bl_simdata.h5 successful.
    
    => dMeasFile()
    ['./../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_bl_simdata.h5']
       
       Group raw:
       Dataset (len = 10000) raw/RefDist
       Dataset (len = 10000) raw/af32_PixelNoise
       Dataset (len = 10000) raw/as16_PixelSignal
       Dataset (len = 10000) raw/as16_PixelValues
       Dataset (len = 10000) raw/f32_Index
       Dataset (len = 10000) raw/s16_AlineAutoGain
       Dataset (len = 10000) raw/u16_EqualSurfaceLine
       Dataset (len = 10000) raw/u16_SumOfPixels
       Dataset (len = 10000) raw/u32_Index
    

## - Einlesen


```python
rSimData(S1_Data,S1)
```

    ../sensordata/180904_RawData_GP-SP_mitNeuemTubus_10-150mm_bl_simdata.h5
    

# Ausgabe


```python
plt.close()
plt.figure('AlineData'); plt.close()
fig_AD = plt.figure('AlineData',figsize = (12,6))
```


    FigureCanvasNbAgg()



    FigureCanvasNbAgg()



```python
fig2.clf()
for i in [0,1]:
    for line in fig_AD.axes[i].get_lines():
         xydata = line.get_xydata()
         plt.plot(xydata[:, 0], xydata[:, 1],marker = 'o')
ax = plt.gca()
#ax.grid(True)
fig2.show()
fig2.savefig('fig2.pdf')
```

## Simulation starten


```python
S1_Data.disable_print_f()
#S1.ResetProductionBlock()
s_SimData = S1.s_SimData;
s_SimData['f32_TeachDist'] = 80.
s_SimData['f32_AlineNoiseFactor'] = 1.
S1.s_SimData = s_SimData
doSim(S1_Data,S1, Signal_Gain = 1)
fig_AD_plt(fig_AD,S1)
PrintSignalNoise(S1)
```

    Info: _print_f() disabled!
    (Enable with enable_print_f()
     => simulate
    gen_df_sim_var(S2) ist momentan veraltet!
    Mean:
    s32_Signal_DR -70
    s32_Signal_sAB1 8
    s32_Signal_dAB1 -3
    
    Std:
    s32_Signal_DR  est: 275  sim: 168  est/sim:   1.64
    s32_Signal_sAB1  est: 261  sim: 249  est/sim:   1.05
    s32_Signal_dAB1  est: 253  sim: 242  est/sim:   1.05
    s32_SignalFiltered_DR  est: 136  sim: 0  est/sim:   0.00
    s32_SignalFiltered_sAB1  est: 129  sim: 0  est/sim:   0.00
    s32_SignalFiltered_dAB1  est: 125  sim: 110  est/sim:   1.14
    


```python
fig_AD_plt(fig_AD,S1)
#gen_df_sim_var(S1)
```


```python
S1.as16_CorrPixArrRaw = S1_Data.get_RefAlineOfDist(target = 'wt', dist = 80)
S1.u16_SignalTeach()
```




    654




```python
S1.s_Param_ProductionBlock_t['s_FitParameters']
```




    {'af32_FitParamsAlineXToDist': [-0.21336022019386292,
      4.534396171569824,
      8.2198486328125,
      -13.717443466186523,
      0.0],
     'af32_FitParamsDistToStdDev': [-1.794108129615779e-06,
      0.0005063748685643077,
      -0.03856576979160309,
      1.328181266784668],
     'af32_FitParamsDist': [10.0, 30.0, 50.0, 70.0, 90.0, 110.0, 130.0, 150.0],
     'af32_FitParamsStdDev': [0.9900000095367432,
      0.5799999833106995,
      0.4399999976158142,
      0.49000000953674316,
      0.6499999761581421,
      0.8299999833106995,
      0.9300000071525574,
      0.8799999952316284]}




```python
s_Param_ProductionBlock_t = S1.s_Param_ProductionBlock_t
for i in range(len(S1.s_Param_ProductionBlock_t['s_ALINE']['af32_Distance'])):
    d = s_Param_ProductionBlock_t['s_ALINE']['af32_Distance'][i]
    S1.as16_CorrPixArrRaw = S1_Data.get_RefAlineOfDist(target = 'wt', dist = d)
    u16_AlineX = S1.u16_Algo_GetAlineX()
    s_Param_ProductionBlock_t['s_ALINE']['au16_AlineX'][i] = u16_AlineX
    #print(u16_AlineX,s_Param_ProductionBlock_t['s_ALINE']['au16_AlineX'][i])

S1.s_Param_ProductionBlock_t = s_Param_ProductionBlock_t
```

    1914 1914
    1862 1862
    1790 1790
    1658 1658
    1589 1589
    1558 1558
    1519 1519
    1447 1447
    1368 1368
    1294 1294
    1230 1230
    1172 1172
    1119 1119
    1083 1083
    1022 1022
    977 977
    926 926
    890 890
    836 836
    813 813
    774 774
    738 738
    698 698
    673 673
    644 644
    612 612
    580 580
    563 563
    533 533
    499 499
    472 472
    446 446
    


```python
display_side_by_side({"s_Algo":dict_overview(S1.s_Algo),
                      "s_AlineData_b":dict_overview(S1.s_AlineData_b),
                      "s_AlgoState_b":dict_overview(S1.s_AlgoState_b),
                      "s_SimData_b":dict_overview(S1.s_SimData_b),
                      "s_SimEvents_b":dict_overview(S1.s_SimEvents_b),
                      #"ProductionBlock":dict_overview(S1.GetProductionBlock),
                     });
```


<style type="text/css" media="screen" style="width:100%">
        table style="display:inline", th, td {border: 0px solid black;  background-color: #eee; padding: 10px;}
        th {background-color: #C6E2FF; color:black; font-family: Tahoma;font-size : 11; text-align: center;}
        td {background-color: #fff; padding: 10px; font-family: Calibri; font-size : 11; text-align: center;}
      </style><style  type="text/css" >
    #T_99cce45c_25ff_11e9_b018_1002b5511cberow0_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow0_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow0_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow1_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow1_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow1_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow2_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow2_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow2_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow3_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow3_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow3_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow4_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow4_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow4_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow5_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow5_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow5_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow6_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow6_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99cce45c_25ff_11e9_b018_1002b5511cberow6_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }</style>  
<table style="display:inline" id="T_99cce45c_25ff_11e9_b018_1002b5511cbe" ><caption>s_Algo</caption> 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >name</th> 
        <th class="col_heading level0 col1" >shape</th> 
        <th class="col_heading level0 col2" >dtype</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow0_col0" class="data row0 col0" >s_AlineData</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow0_col1" class="data row0 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow0_col2" class="data row0 col2" > dict</td> 
    </tr>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row1" class="row_heading level0 row1" >1</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow1_col0" class="data row1 col0" >s_EnergyThresholds</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow1_col1" class="data row1 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow1_col2" class="data row1 col2" > dict</td> 
    </tr>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row2" class="row_heading level0 row2" >2</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow2_col0" class="data row2 col0" >s_AlgoState</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow2_col1" class="data row2 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow2_col2" class="data row2 col2" > dict</td> 
    </tr>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row3" class="row_heading level0 row3" >3</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow3_col0" class="data row3 col0" >s_AlgoTeach</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow3_col1" class="data row3 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow3_col2" class="data row3 col2" > dict</td> 
    </tr>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row4" class="row_heading level0 row4" >4</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow4_col0" class="data row4 col0" >s_AlgoMode</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow4_col1" class="data row4 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow4_col2" class="data row4 col2" > dict</td> 
    </tr>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row5" class="row_heading level0 row5" >5</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow5_col0" class="data row5 col0" >s_SimData</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow5_col1" class="data row5 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow5_col2" class="data row5 col2" > dict</td> 
    </tr>    <tr> 
        <th id="T_99cce45c_25ff_11e9_b018_1002b5511cbelevel0_row6" class="row_heading level0 row6" >6</th> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow6_col0" class="data row6 col0" >s_SimEvents</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow6_col1" class="data row6 col1" >(1,)</td> 
        <td id="T_99cce45c_25ff_11e9_b018_1002b5511cberow6_col2" class="data row6 col2" > dict</td> 
    </tr></tbody> 
</table style="display:inline"> <style  type="text/css" >
    #T_99d80836_25ff_11e9_944f_1002b5511cberow0_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow0_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow0_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow1_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow1_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow1_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow2_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow2_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow2_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow3_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow3_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow3_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow4_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow4_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow4_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow5_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow5_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow5_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow6_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow6_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow6_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow7_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow7_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow7_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow8_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow8_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow8_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow9_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow9_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow9_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow10_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow10_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow10_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow11_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow11_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow11_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow12_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow12_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow12_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow13_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow13_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow13_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow14_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow14_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow14_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow15_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow15_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow15_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow16_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow16_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow16_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow17_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow17_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow17_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow18_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow18_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow18_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow19_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow19_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow19_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow20_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow20_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow20_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow21_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow21_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow21_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow22_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow22_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow22_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow23_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow23_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow23_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow24_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow24_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow24_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow25_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow25_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow25_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow26_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow26_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99d80836_25ff_11e9_944f_1002b5511cberow26_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }</style>  
<table style="display:inline" id="T_99d80836_25ff_11e9_944f_1002b5511cbe" ><caption>s_AlineData_b</caption> 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >name</th> 
        <th class="col_heading level0 col1" >shape</th> 
        <th class="col_heading level0 col2" >dtype</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow0_col0" class="data row0 col0" >ps16_CorrPixArrRaw</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow0_col1" class="data row0 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow0_col2" class="data row0 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row1" class="row_heading level0 row1" >1</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow1_col0" class="data row1 col0" >ps16_RefPixArr_DR</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow1_col1" class="data row1 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow1_col2" class="data row1 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row2" class="row_heading level0 row2" >2</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow2_col0" class="data row2 col0" >ps16_RefPixArr_sAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow2_col1" class="data row2 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow2_col2" class="data row2 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row3" class="row_heading level0 row3" >3</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow3_col0" class="data row3 col0" >ps16_RefPixArr_sAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow3_col1" class="data row3 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow3_col2" class="data row3 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row4" class="row_heading level0 row4" >4</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow4_col0" class="data row4 col0" >ps16_RefPixArr_dAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow4_col1" class="data row4 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow4_col2" class="data row4 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row5" class="row_heading level0 row5" >5</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow5_col0" class="data row5 col0" >ps16_RefPixArr_dAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow5_col1" class="data row5 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow5_col2" class="data row5 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row6" class="row_heading level0 row6" >6</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow6_col0" class="data row6 col0" >ps32_AvgCorrPixTeach</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow6_col1" class="data row6 col1" >(10000, 16)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow6_col2" class="data row6 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row7" class="row_heading level0 row7" >7</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow7_col0" class="data row7 col0" >s32_Signal_DR</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow7_col1" class="data row7 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow7_col2" class="data row7 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row8" class="row_heading level0 row8" >8</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow8_col0" class="data row8 col0" >s32_Signal_sAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow8_col1" class="data row8 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow8_col2" class="data row8 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row9" class="row_heading level0 row9" >9</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow9_col0" class="data row9 col0" >s32_Signal_sAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow9_col1" class="data row9 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow9_col2" class="data row9 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row10" class="row_heading level0 row10" >10</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow10_col0" class="data row10 col0" >s32_Signal_dAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow10_col1" class="data row10 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow10_col2" class="data row10 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row11" class="row_heading level0 row11" >11</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow11_col0" class="data row11 col0" >s32_Signal_dAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow11_col1" class="data row11 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow11_col2" class="data row11 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row12" class="row_heading level0 row12" >12</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow12_col0" class="data row12 col0" >s32_SignalDark_DR</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow12_col1" class="data row12 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow12_col2" class="data row12 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row13" class="row_heading level0 row13" >13</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow13_col0" class="data row13 col0" >s32_SignalDark_sAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow13_col1" class="data row13 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow13_col2" class="data row13 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row14" class="row_heading level0 row14" >14</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow14_col0" class="data row14 col0" >s32_SignalDark_sAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow14_col1" class="data row14 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow14_col2" class="data row14 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row15" class="row_heading level0 row15" >15</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow15_col0" class="data row15 col0" >s32_SignalFiltered_dAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow15_col1" class="data row15 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow15_col2" class="data row15 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row16" class="row_heading level0 row16" >16</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow16_col0" class="data row16 col0" >s32_SignalFiltered_dAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow16_col1" class="data row16 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow16_col2" class="data row16 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row17" class="row_heading level0 row17" >17</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow17_col0" class="data row17 col0" >s32_SignalFiltered_sAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow17_col1" class="data row17 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow17_col2" class="data row17 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row18" class="row_heading level0 row18" >18</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow18_col0" class="data row18 col0" >s32_SignalFiltered_sAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow18_col1" class="data row18 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow18_col2" class="data row18 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row19" class="row_heading level0 row19" >19</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow19_col0" class="data row19 col0" >s32_SignalFiltered_DR</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow19_col1" class="data row19 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow19_col2" class="data row19 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row20" class="row_heading level0 row20" >20</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow20_col0" class="data row20 col0" >s32_SignalHighFiltered_sAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow20_col1" class="data row20 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow20_col2" class="data row20 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row21" class="row_heading level0 row21" >21</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow21_col0" class="data row21 col0" >s32_SignalHighFiltered_sAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow21_col1" class="data row21 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow21_col2" class="data row21 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row22" class="row_heading level0 row22" >22</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow22_col0" class="data row22 col0" >s32_SignalHighFiltered_DR</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow22_col1" class="data row22 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow22_col2" class="data row22 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row23" class="row_heading level0 row23" >23</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow23_col0" class="data row23 col0" >s32_SignalWeakFiltered_sAB1</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow23_col1" class="data row23 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow23_col2" class="data row23 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row24" class="row_heading level0 row24" >24</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow24_col0" class="data row24 col0" >s32_SignalWeakFiltered_sAB2</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow24_col1" class="data row24 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow24_col2" class="data row24 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row25" class="row_heading level0 row25" >25</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow25_col0" class="data row25 col0" >s32_SignalWeakFiltered_DR</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow25_col1" class="data row25 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow25_col2" class="data row25 col2" > np.int32</td> 
    </tr>    <tr> 
        <th id="T_99d80836_25ff_11e9_944f_1002b5511cbelevel0_row26" class="row_heading level0 row26" >26</th> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow26_col0" class="data row26 col0" >f32_Distance</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow26_col1" class="data row26 col1" >(10000,)</td> 
        <td id="T_99d80836_25ff_11e9_944f_1002b5511cberow26_col2" class="data row26 col2" > np.float32</td> 
    </tr></tbody> 
</table style="display:inline"> <style  type="text/css" >
    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow0_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow0_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow0_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow1_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow1_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow1_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow2_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow2_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow2_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow3_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow3_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow3_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow4_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow4_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow4_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow5_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow5_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow5_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow6_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow6_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow6_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow7_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow7_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99dd1166_25ff_11e9_9d95_1002b5511cberow7_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }</style>  
<table style="display:inline" id="T_99dd1166_25ff_11e9_9d95_1002b5511cbe" ><caption>s_AlgoState_b</caption> 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >name</th> 
        <th class="col_heading level0 col1" >shape</th> 
        <th class="col_heading level0 col2" >dtype</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow0_col0" class="data row0 col0" >b_SignalGood_DR</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow0_col1" class="data row0 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow0_col2" class="data row0 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row1" class="row_heading level0 row1" >1</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow1_col0" class="data row1 col0" >b_SignalGood_sAB1</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow1_col1" class="data row1 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow1_col2" class="data row1 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row2" class="row_heading level0 row2" >2</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow2_col0" class="data row2 col0" >b_SignalGood_sAB2</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow2_col1" class="data row2 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow2_col2" class="data row2 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row3" class="row_heading level0 row3" >3</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow3_col0" class="data row3 col0" >b_SensorState_dAB1</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow3_col1" class="data row3 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow3_col2" class="data row3 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row4" class="row_heading level0 row4" >4</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow4_col0" class="data row4 col0" >b_SensorState_dAB2</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow4_col1" class="data row4 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow4_col2" class="data row4 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row5" class="row_heading level0 row5" >5</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow5_col0" class="data row5 col0" >b_SensorState</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow5_col1" class="data row5 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow5_col2" class="data row5 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row6" class="row_heading level0 row6" >6</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow6_col0" class="data row6 col0" >b_ObjectDetected</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow6_col1" class="data row6 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow6_col2" class="data row6 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99dd1166_25ff_11e9_9d95_1002b5511cbelevel0_row7" class="row_heading level0 row7" >7</th> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow7_col0" class="data row7 col0" >b_WeakSignal</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow7_col1" class="data row7 col1" >(10000,)</td> 
        <td id="T_99dd1166_25ff_11e9_9d95_1002b5511cberow7_col2" class="data row7 col2" > np.uint8</td> 
    </tr></tbody> 
</table style="display:inline"> <style  type="text/css" >
    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow0_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow0_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow0_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow1_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow1_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow1_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow2_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow2_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow2_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow3_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow3_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow3_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow4_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow4_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow4_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow5_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow5_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow5_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow6_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow6_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow6_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow7_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow7_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e1091e_25ff_11e9_89cf_1002b5511cberow7_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }</style>  
<table style="display:inline" id="T_99e1091e_25ff_11e9_89cf_1002b5511cbe" ><caption>s_SimData_b</caption> 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >name</th> 
        <th class="col_heading level0 col1" >shape</th> 
        <th class="col_heading level0 col2" >dtype</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow0_col0" class="data row0 col0" >af32_AlineNoise</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow0_col1" class="data row0 col1" >(10000, 16)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow0_col2" class="data row0 col2" > np.float32</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row1" class="row_heading level0 row1" >1</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow1_col0" class="data row1 col0" >f32_RefDist</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow1_col1" class="data row1 col1" >(10000,)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow1_col2" class="data row1 col2" > np.float32</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row2" class="row_heading level0 row2" >2</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow2_col0" class="data row2 col0" >f32_dt</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow2_col1" class="data row2 col1" >(10000,)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow2_col2" class="data row2 col2" > np.float32</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row3" class="row_heading level0 row3" >3</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow3_col0" class="data row3 col0" >f32_t</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow3_col1" class="data row3 col1" >(10000,)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow3_col2" class="data row3 col2" > np.float32</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row4" class="row_heading level0 row4" >4</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow4_col0" class="data row4 col0" >as16_AlineSignal</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow4_col1" class="data row4 col1" >(10000, 16)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow4_col2" class="data row4 col2" > np.int16</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row5" class="row_heading level0 row5" >5</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow5_col0" class="data row5 col0" >u32_events</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow5_col1" class="data row5 col1" >(10000,)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow5_col2" class="data row5 col2" > np.uint32</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row6" class="row_heading level0 row6" >6</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow6_col0" class="data row6 col0" >u32_state_changes</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow6_col1" class="data row6 col1" >(10000,)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow6_col2" class="data row6 col2" > np.uint32</td> 
    </tr>    <tr> 
        <th id="T_99e1091e_25ff_11e9_89cf_1002b5511cbelevel0_row7" class="row_heading level0 row7" >7</th> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow7_col0" class="data row7 col0" >u32_updates</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow7_col1" class="data row7 col1" >(10000,)</td> 
        <td id="T_99e1091e_25ff_11e9_89cf_1002b5511cberow7_col2" class="data row7 col2" > np.uint32</td> 
    </tr></tbody> 
</table style="display:inline"> <style  type="text/css" >
    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow0_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow0_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow0_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow1_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow1_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow1_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow2_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow2_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow2_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow3_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow3_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow3_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow4_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow4_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow4_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow5_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow5_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow5_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow6_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow6_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow6_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow7_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow7_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow7_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow8_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow8_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow8_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow9_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow9_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow9_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow10_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow10_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow10_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow11_col0 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow11_col1 {
            font-size:  10pt;
            font-family:  Calibri;
        }    #T_99e527ec_25ff_11e9_bf3a_1002b5511cberow11_col2 {
            font-size:  10pt;
            font-family:  Calibri;
        }</style>  
<table style="display:inline" id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbe" ><caption>s_SimEvents_b</caption> 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >name</th> 
        <th class="col_heading level0 col1" >shape</th> 
        <th class="col_heading level0 col2" >dtype</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow0_col0" class="data row0 col0" >b_SignalGood_DR</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow0_col1" class="data row0 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow0_col2" class="data row0 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row1" class="row_heading level0 row1" >1</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow1_col0" class="data row1 col0" >b_SignalGood_sAB1</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow1_col1" class="data row1 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow1_col2" class="data row1 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row2" class="row_heading level0 row2" >2</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow2_col0" class="data row2 col0" >b_SignalGood_sAB2</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow2_col1" class="data row2 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow2_col2" class="data row2 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row3" class="row_heading level0 row3" >3</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow3_col0" class="data row3 col0" >b_SensorState_dAB1</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow3_col1" class="data row3 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow3_col2" class="data row3 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row4" class="row_heading level0 row4" >4</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow4_col0" class="data row4 col0" >b_SensorState_dAB2</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow4_col1" class="data row4 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow4_col2" class="data row4 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row5" class="row_heading level0 row5" >5</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow5_col0" class="data row5 col0" >b_SensorState</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow5_col1" class="data row5 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow5_col2" class="data row5 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row6" class="row_heading level0 row6" >6</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow6_col0" class="data row6 col0" >b_ObjectDetected</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow6_col1" class="data row6 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow6_col2" class="data row6 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row7" class="row_heading level0 row7" >7</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow7_col0" class="data row7 col0" >b_WeakSignal</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow7_col1" class="data row7 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow7_col2" class="data row7 col2" > np.uint8</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row8" class="row_heading level0 row8" >8</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow8_col0" class="data row8 col0" >f32_RefDist</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow8_col1" class="data row8 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow8_col2" class="data row8 col2" > np.float32</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row9" class="row_heading level0 row9" >9</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow9_col0" class="data row9 col0" >f32_t</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow9_col1" class="data row9 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow9_col2" class="data row9 col2" > np.float32</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row10" class="row_heading level0 row10" >10</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow10_col0" class="data row10 col0" >u32_events</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow10_col1" class="data row10 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow10_col2" class="data row10 col2" > np.uint32</td> 
    </tr>    <tr> 
        <th id="T_99e527ec_25ff_11e9_bf3a_1002b5511cbelevel0_row11" class="row_heading level0 row11" >11</th> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow11_col0" class="data row11 col0" >u32_updates</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow11_col1" class="data row11 col1" >(11,)</td> 
        <td id="T_99e527ec_25ff_11e9_bf3a_1002b5511cberow11_col2" class="data row11 col2" > np.uint32</td> 
    </tr></tbody> 
</table style="display:inline"> 



```python

```
