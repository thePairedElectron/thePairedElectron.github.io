---
layout: post
title:  "My Progress Report of GSoC'19 with Libre Space Foundation"
date:   2019-10-10 18:34:10 +0700
categories: [gsoc, machine learning, python]
---

***
![Google Summer of Code](https://i.imgur.com/LTW3Tgl.png)
___
***
![Libre Space Foundation](https://i.imgur.com/7pwtR15.png)

***

Hey there! We are in the endgame now. Yes, Google Summer of Code 2019 is coming to an end. I couldn’t say how 3 months passed by, but this is one of my most memorable experiences I will always cherish forever. I worked on [Machine Learning on Health- Keeping Telemetry for Cubesat Awareness and Diagnostics](https://summerofcode.withgoogle.com/projects/#5419414323724288) as a part of GSoC'19. So let me wrap GSOC with this final report. This post is part of my final submission. **Buckle up tight and bring your beer because it’s going to be a long post.**:beer:

![](https://i.imgur.com/jhVoEX6.jpg)

Introduction
===
The project aims to analyze a satellite set of telemetry to understand links and dependencies among different subsystems. The project should be able to demonstrate an understanding of the links between the different behaviour changes of each telemetry within a satellite or within a set of external sources of information (mission plan, solar aspect angles, ephemerides, etc.) in order to rapidly characterize future debris events to support risk analysis, close approach analysis, collision avoidance maneuvering, forensic analysis and other decision making. Machine learning can be used to learn the different link models and storage of acquired knowledge should be stored in a graph (Bayesian network). The intermediate and final output should be represented as data interpretable by a visualization interfaces, preferably in JSON.

The [polaris](https://gitlab.com/crespum/polaris) project is divided in three phases:

- [ ] [polaris](https://gitlab.com/crespum/polaris/wikis/Home)
  - [x] [polaris_fetch](https://gitlab.com/crespum/polaris/wikis/Polaris-Fetch)
  - [x] [polaris_learn](https://gitlab.com/crespum/polaris/wikis/Polaris-Learn)
  - [ ] [polaris_viz](https://gitlab.com/crespum/polaris/wikis/Polaris-Viz)

Organization 
===
[Libre Space Foundation](https://libre.space/)

Mentors
===
[Xabi Crespo](https://gitlab.com/crespum)
[Red Boumghar](https://gitlab.com/redsharpbyte)
[Hugh Brown](https://gitlab.com/saintaardvark)
[Patrick Dohmen](https://gitlab.com/DL4PD)

Buddy
===
[Jan-Peter Ceglarek](https://gitlab.com/jan-peter)- Student for [ESA SOCIS](https://socis.esa.int/) Program 

Rewinding Polaris ⏪
===
I’ll go through what has been done till now on Polaris. I know that you guys would have read the same stuffs in my previous posts, please don’t get bored, I promise it’s going to be so interesting this time.


## Community Bonding
By this time, I have talked with lot of open source developers mostly chitchat using riot or jit.si. I have never felt so welcomed anywhere but in open source. They answered to me even if I asked any stupid or silly question. 

You're always welcome at [Polaris](https://riot.im/app/#/room/#polaris:matrix.org) Riot channel.

## Proposal
In case you still haven't got chance to look at it:
<iframe src="https://drive.google.com/file/d/1gIaWHfrn3mDIKMD5exRLyZprilA90nV5/preview" width="640" height="480"></iframe>

## Polaris Deliverables
The main deliverable is a working python module that can take any set of telemetry from the SatNOGS network, merge it with spacecraft context, and perform an analysis to diagnose the health of the satellite subsystems. The final output should be represented as data which is interpretable by a visualization interfaces, preferably in JSON which can be integrated with the SatNOGS dashboard (Grafana).

## Polaris Architecture
![](https://i.imgur.com/nvrjB7z.png)

Work Progress
===
## Merged PR

**Added data fetching and decoding utility**
[Merge Request !15](https://gitlab.com/crespum/polaris/merge_requests/15)
I started the polaris_fetch module with fetching the dataframes for the ELFIN-A satellite. So as to check whether the polaris_fetch is working properly or not.  

I used subprocess module to build up the commands to call glouton and the satnogs decoders. 

The glouton downloads the frames for a particular satellite (having unique NORAD ID) in csv format between the start period and end period provided by the user. So for one timestamp, we got one csv file. So after downloading all the csv files in one directory, we have to merge the all csv files into one. 

The next task was to call the satnogs decoders command, which will take input satellite decoder (Elfin is the decoder for satellite ELFIN-A) and the merged CSV file. The output of the satnogs decoder is array of JSON like objects. So there is a object per timestamp.

The json like objects that we're getting as an output of satnogs decoders needs to be normalized. 

So summarizing this merge request: I developed a python module which can be used to fetch and decode the dataframes from the SatNOGS network and store it as a JSON object. [Glouton](https://github.com/deckbsd/glouton-satnogs-data-downloader) is used for fetching dataframes for a satellite within specified timeframes from SatNOGS network and [Satnogs Decoders](https://gitlab.com/acinonyx/satnogs-decoders) is used to decode the downloaded data and store it as a JSON object.

Command Line Output of polaris_fetch:
```
(.venv) aditya@aditya-Lenovo-Z51-70:~/polaris$ polaris fetch ELFIN-A
2019-08-25 12:41:46,823 - polaris.polaris - INFO - output dir: /tmp
INFO: Satellite: id=43617 name=ELFIN-A decoder=Elfin
INFO: selected decoder=Elfin
INFO: Fetch start date: 2019-08-25 06:11:46.823920
INFO: Fetch end date: 2019-08-25 07:11:46.823920
Demoddata module(s) loading :
module : CSV loaded
scanning page...1

downloading started (Ctrl + C to stop)...	~(  ^o^)~
Saving the dataframes in directory: /tmp
Merging all the csv files into one CSV file.
sed: can't read /tmp/data_1566693706_82392/demod*/*.csv: No such file or directory
Merge Completed
Storing merged CSV file: /tmp/merged_frames.csv
Starting decoding of the data
Decoding of data finished.
Stored the decoded data JSON file in root directory: /tmp/decoded_frames.json

```

This is how decoded telemetry looks like:
```javascipt
{
    "hskp_pwr1_bv_mon": 8, 
    "errors_error4_hour": 1, 
    "fc_counters_fcpkts_fm_radio": 174, 
    "errors_error3_error": 11, 
    "radio_tlm_bytes_rx": 1892771585, s
    "acb_pc_data2_tmps_tmp3": 0, 
    "errors_error3_hour": 1, 
    "hskp_pwr1_bat_mon_1_acc_curr_reg": 0, 
    "hskp_pwr2_adc_data_power_bus_current_1": 0, 
    "hskp_pwr2_adc_data_power_bus_current_2": 0, 
    "fc_counters_intrnl_wdttmout": 0, 
    "hskp_pwr2_accumulated_curr_bat2_rsrc": 0, 
    "errors_error1_minute": 16, 
    "hskp_pwr1_accumulated_curr_bat2_rarc": 0, 
    "fc_counters_badcmds_recv": 0, 
    "hskp_pwr1_bat_mon_2_temperature_register": 0, 
    "acb_pc_data2_ipdu_mrm_x": 270, 
    "hskp_pwr2_bat_mon_2_avg_cur_reg": 0, 
    "fc_counters_vu1_off": 15, 
    "radio_cfg_read_radio_palvl": 145, 
    "status_2_htr_alert": true, 
    "hskp_pwr2_rtcc_minute": 6, 
    "hskp_pwr2_bat_mon_1_volt_reg": 0, 
    "errors_error6_error": 11, 
    "hskp_pwr2_bat_mon_2_temperature_register": 0, 
    "acb_pc_data1_rtcc_day": 9, 
    "errors_error7_day": 9, 
    "acb_pc_data1_tmps_tmp1": 0, 
    "acb_pc_data1_tmps_tmp3": 0, 
    "hskp_pwr2_adc_data_adc_sa_volt_12": 0, 
    "acb_pc_data1_tmps_tmp4": 0, 
    "hskp_pwr1_bat_mon_2_avg_cur_reg": 0, 
    "errors_error5_minute": 72, 
    "hskp_pwr1_accumulated_curr_bat2_rsrc": 0, 
    "hskp_pwr2_bat_mon_1_avg_cur_reg": 0, 
    "acb_sense_adc_data_voltage": 13107, 
    "errors_error6_second": 36, 
    "status_1_reserved": 3, 
    "radio_tlm_bytes_tx": 6047757, 
    "acb_pc_data1_rtcc_month": 5, 
    "hskp_pwr1_adc_data_adc_sa_volt_12": 0, 
    "fc_counters_vu2_on": 17, 
    "radio_tlm_rssi": 0, 
    "hskp_pwr2_rtcc_year": 25, 
    "hskp_pwr2_adc_data_bat_2_volt": 0, 
    "errors_error5_error": 11, 
    "status_2_9v_boost": false, 
    "fc_counters_badpkts_fm_radio": 84, 
    "hskp_pwr1_rtcc_minute": 22, 
    "hskp_pwr1_accumulated_curr_bat1_rarc": 0, 
    "errors_error2_second": 82, 
    "acb_pc_data2_ipdu_mrm_z": -17408, 
    "acb_pc_data2_ipdu_mrm_y": 373, 
    "acb_pc_data1_tmps_tmp2": 0, 
    "acb_pc_data2_rtcc_month": 5, 
    "fc_counters_wdpicrst": 60, 
    "hskp_pwr2_bat_mon_2_acc_curr_reg": 0, 
    "hskp_pwr1_adc_data_bat_1_volt": 0, 
    "hskp_pwr2_tmps_tmp1": 3888, 
    "hskp_pwr2_tmps_tmp2": 3824, 
    "hskp_pwr2_tmps_tmp3": 3792, 
    "hskp_pwr2_tmps_tmp4": 3728, 
    "acb_pc_data1_ipdu_mrm_x": 0, 
    "acb_pc_data1_ipdu_mrm_y": 0, 
    "acb_pc_data1_rtcc_hour": 0, 
    "acb_pc_data2_rtcc_day": 9, 
    "hskp_pwr1_tmps_tmp3": 4032, 
    "hskp_pwr1_tmps_tmp2": 4128, 
    "hskp_pwr2_bat_mon_1_temperature_register": 0, 
    "acb_pc_data2_tmps_tmp1": 0, 
    "hskp_pwr1_bat_mon_1_cur_reg": 0, 
    "hskp_pwr1_tmps_tmp4": 4048, 
    "hskp_pwr1_bat_mon_1_volt_reg": 0, 
    "acb_pc_data1_ipdu_mrm_z": 256, 
    "errors_error6_day": 9, 
    "fc_counters_errors": 248, 
    "status_1_safe_mode": true, 
    "hskp_pwr2_adc_data_adc_sa_volt_34": 0, 
    "ctl": 3, 
    "fc_counters_sips_ovcur_evts": 0, 
    "errors_error7_hour": 2, 
    "hskp_pwr2_bat_mon_1_acc_curr_reg": 0, 
    "hskp_pwr2_accumulated_curr_bat1_rarc": 0, 
    "status_2_htr_force": false, 
    "errors_error1_hour": 1, 
    "hskp_pwr2_rtcc_hour": 2, 
    "hskp_pwr2_bv_mon": 8, 
    "dest_ssid": 0, 
    "fc_counters_vu1_on": 9, 
    "acb_pc_data2_rtcc_minute": 51, 
    "hskp_pwr2_rtcc_month": 5, 
    "errors_error4_day": 9, 
    "hskp_pwr1_tmps_tmp1": 4304, 
    "status_1_early_orbit": 8, 
    "errors_error1_error": 11, 
    "fc_counters_brwnouts": 59, 
    "dest_callsign": "W6YRA9", 
    "acb_pc_data2_rtcc_year": 25, 
    "hskp_pwr2_bat_mon_2_volt_reg": 0, 
    "hskp_pwr1_bat_mon_2_cur_reg": 0, 
    "hskp_pwr1_accumulated_curr_bat1_rsrc": 0, 
    "hskp_pwr1_adc_data_adc_sa_volt_34": 0, 
    "errors_error1_day": 9, 
    "hskp_pwr1_bat_mon_2_acc_curr_reg": 0, 
    "errors_error6_minute": 87, 
    "errors_error5_day": 9, 
    "errors_error7_second": 72, 
    "hskp_pwr2_accumulated_curr_bat1_rsrc": 0, 
    "hskp_pwr2_bat_mon_2_cur_reg": 0, 
    "acb_pc_data1_acb_mrm_y": 0, 
    "errors_error5_second": 1, 
    "acb_pc_data2_rtcc_second": 68, 
    "fc_counters_cmds_recv": 118, 
    "errors_error5_hour": 1, 
    "errors_error7_error": 11, 
    "errors_error4_minute": 56, 
    "errors_error1_second": 41, 
    "errors_error3_minute": 41, 
    "errors_error3_day": 9, 
    "fc_crc": 12, 
    "hskp_pwr1_adc_data_sa_short_circuit_current": 0, 
    "errors_error2_hour": 1, 
    "hskp_pwr1_rtcc_second": 18, 
    "hskp_pwr1_rtcc_hour": 2, 
    "hskp_pwr2_adc_data_adc_sa_volt_56": 0, 
    "status_2_reserved": 7, 
    "errors_error6_hour": 1, 
    "acb_pc_data2_tmps_tmp2": 0, 
    "hskp_pwr1_adc_data_reg_sa_volt_1": 0, 
    "hskp_pwr1_adc_data_reg_sa_volt_2": 0, 
    "hskp_pwr1_adc_data_reg_sa_volt_3": 0, 
    "reserved": 0, 
    "acb_pc_data2_tmps_tmp4": 0, 
    "hskp_pwr2_adc_data_sa_short_circuit_current": 0, 
    "pid": 240, 
    "hskp_pwr1_rtcc_month": 5, 
    "fc_counters_reboots": 98, 
    "hskp_pwr2_rtcc_day": 9, 
    "errors_error7_minute": 6, 
    "acb_pc_data2_acb_mrm_z": 30201, 
    "acb_pc_data2_acb_mrm_x": 12289, 
    "acb_pc_data2_acb_mrm_y": 4865, 
    "fc_counters_uart1_parseerrs": 249, 
    "errors_error2_error": 11, 
    "errors_error4_error": 11, 
    "src_callsign": "WJ2XNX", 
    "hskp_pwr2_accumulated_curr_bat2_rarc": 0, 
    "hskp_pwr1_adc_data_adc_sa_volt_56": 0, 
    "errors_error2_minute": 25, 
    "fc_counters_uart1_recvpkts": 156, 
    "status_2_payload_power": false, 
    "acb_pc_data2_rtcc_hour": 0, 
    "hskp_pwr1_rtcc_day": 9, 
    "hskp_pwr1_adc_data_power_bus_current_1": 0, 
    "hskp_pwr1_adc_data_power_bus_current_2": 0, 
    "acb_pc_data1_rtcc_year": 25, 
    "hskp_pwr2_adc_data_reg_sa_volt_1": 0, 
    "hskp_pwr2_adc_data_reg_sa_volt_3": 0, 
    "hskp_pwr2_adc_data_reg_sa_volt_2": 0, 
    "acb_pc_data1_rtcc_second": 16, 
    "hskp_pwr1_adc_data_bat_2_volt": 0, 
    "fc_salt": "elfa", 
    "fc_counters_porst": 2, 
    "errors_error3_second": 21, 
    "hskp_pwr2_adc_data_bat_1_volt": 0, 
    "src_ssid": 0, 
    "acb_pc_data1_acb_mrm_x": 0, 
    "hskp_pwr2_bat_mon_1_cur_reg": 0, 
    "acb_pc_data1_acb_mrm_z": 0, 
    "beacon_setting": 14, 
    "acb_pc_data1_rtcc_minute": 51, 
    "fc_counters_vu2_off": 23, 
    "hskp_pwr2_rtcc_second": 72, 
    "hskp_pwr1_rtcc_year": 25, 
    "hskp_pwr1_bat_mon_1_avg_cur_reg": 0, 
    "errors_error4_second": 55, 
    "errors_error2_day": 9, 
    "hskp_pwr1_bat_mon_1_temperature_register": 0, 
    "acb_sense_adc_data_current": 13081
}
{
    "hskp_pwr1_bv_mon": 8, 
    "errors_error4_hour": 1, 
    "fc_counters_fcpkts_fm_radio": 176, 
    "errors_error3_error": 11, 
    "radio_tlm_bytes_rx": 1892771585, 
    "acb_pc_data2_tmps_tmp3": 0, 
    "errors_error3_hour": 1, 
    "hskp_pwr1_bat_mon_1_acc_curr_reg": 0, 
    "hskp_pwr2_adc_data_power_bus_current_1": 0, 
    "hskp_pwr2_adc_data_power_bus_current_2": 0, 
    "fc_counters_intrnl_wdttmout": 0, 
    "hskp_pwr2_accumulated_curr_bat2_rsrc": 0, 
    "errors_error1_minute": 16, 
    "hskp_pwr1_accumulated_curr_bat2_rarc": 0, 
    "fc_counters_badcmds_recv": 0, 
    "hskp_pwr1_bat_mon_2_temperature_register": 0, 
    "acb_pc_data2_ipdu_mrm_x": 270, 
    "hskp_pwr2_bat_mon_2_avg_cur_reg": 0, 
    "fc_counters_vu1_off": 15, 
    "radio_cfg_read_radio_palvl": 145, 
    "status_2_htr_alert": true, 
    "hskp_pwr2_rtcc_minute": 6, 
    "hskp_pwr2_bat_mon_1_volt_reg": 0, 
    "errors_error6_error": 11, 
    "hskp_pwr2_bat_mon_2_temperature_register": 0, 
    "acb_pc_data1_rtcc_day": 9, 
    "errors_error7_day": 9, 
    "acb_pc_data1_tmps_tmp1": 0, 
    "acb_pc_data1_tmps_tmp3": 0, 
    "hskp_pwr2_adc_data_adc_sa_volt_12": 0, 
    "acb_pc_data1_tmps_tmp4": 0, 
    "hskp_pwr1_bat_mon_2_avg_cur_reg": 0, 
    "errors_error5_minute": 72, 
    "hskp_pwr1_accumulated_curr_bat2_rsrc": 0, 
    "hskp_pwr2_bat_mon_1_avg_cur_reg": 0, 
    "acb_sense_adc_data_voltage": 13107, 
    "errors_error6_second": 36, 
    "status_1_reserved": 3, 
    "radio_tlm_bytes_tx": 6047757, 
    "acb_pc_data1_rtcc_month": 5, 
    "hskp_pwr1_adc_data_adc_sa_volt_12": 0, 
    "fc_counters_vu2_on": 17, 
    "radio_tlm_rssi": 0, 
    "hskp_pwr2_rtcc_year": 25, 
    "hskp_pwr2_adc_data_bat_2_volt": 0, 
    "errors_error5_error": 11, 
    "status_2_9v_boost": false, 
    "fc_counters_badpkts_fm_radio": 84, 
    "hskp_pwr1_rtcc_minute": 22, 
    "hskp_pwr1_accumulated_curr_bat1_rarc": 0, 
    "errors_error2_second": 82, 
    "acb_pc_data2_ipdu_mrm_z": -17408, 
    "acb_pc_data2_ipdu_mrm_y": 373, 
    "acb_pc_data1_tmps_tmp2": 0, 
    "acb_pc_data2_rtcc_month": 5, 
    "fc_counters_wdpicrst": 60, 
    "hskp_pwr2_bat_mon_2_acc_curr_reg": 0, 
    "hskp_pwr1_adc_data_bat_1_volt": 0, 
    "hskp_pwr2_tmps_tmp1": 3888, 
    "hskp_pwr2_tmps_tmp2": 3824, 
    "hskp_pwr2_tmps_tmp3": 3792, 
    "hskp_pwr2_tmps_tmp4": 3728, 
    "acb_pc_data1_ipdu_mrm_x": 0, 
    "acb_pc_data1_ipdu_mrm_y": 0, 
    "acb_pc_data1_rtcc_hour": 0, 
    "acb_pc_data2_rtcc_day": 9, 
    "hskp_pwr1_tmps_tmp3": 4032, 
    "hskp_pwr1_tmps_tmp2": 4128, 
    "hskp_pwr2_bat_mon_1_temperature_register": 0, 
    "acb_pc_data2_tmps_tmp1": 0, 
    "hskp_pwr1_bat_mon_1_cur_reg": 0, 
    "hskp_pwr1_tmps_tmp4": 4048, 
    "hskp_pwr1_bat_mon_1_volt_reg": 0, 
    "acb_pc_data1_ipdu_mrm_z": 256, 
    "errors_error6_day": 9, 
    "fc_counters_errors": 249, 
    "status_1_safe_mode": true, 
    "hskp_pwr2_adc_data_adc_sa_volt_34": 0, 
    "ctl": 3, 
    "fc_counters_sips_ovcur_evts": 0, 
    "errors_error7_hour": 2, 
    "hskp_pwr2_bat_mon_1_acc_curr_reg": 0, 
    "hskp_pwr2_accumulated_curr_bat1_rarc": 0, 
    "status_2_htr_force": false, 
    "errors_error1_hour": 1, 
    "hskp_pwr2_rtcc_hour": 2, 
    "hskp_pwr2_bv_mon": 8, 
    "dest_ssid": 0, 
    "fc_counters_vu1_on": 9, 
    "acb_pc_data2_rtcc_minute": 51, 
    "hskp_pwr2_rtcc_month": 5, 
    "errors_error4_day": 9, 
    "hskp_pwr1_tmps_tmp1": 4304, 
    "status_1_early_orbit": 8, 
    "errors_error1_error": 11, 
    "fc_counters_brwnouts": 59, 
    "dest_callsign": "W6YRA9", 
    "acb_pc_data2_rtcc_year": 25, 
    "hskp_pwr2_bat_mon_2_volt_reg": 0, 
    "hskp_pwr1_bat_mon_2_cur_reg": 0, 
    "hskp_pwr1_accumulated_curr_bat1_rsrc": 0, 
    "hskp_pwr1_adc_data_adc_sa_volt_34": 0, 
    "errors_error1_day": 9, 
    "hskp_pwr1_bat_mon_2_acc_curr_reg": 0, 
    "errors_error6_minute": 87, 
    "errors_error5_day": 9, 
    "errors_error7_second": 72, 
    "hskp_pwr2_accumulated_curr_bat1_rsrc": 0, 
    "hskp_pwr2_bat_mon_2_cur_reg": 0, 
    "acb_pc_data1_acb_mrm_y": 0, 
    "errors_error5_second": 1, 
    "acb_pc_data2_rtcc_second": 68, 
    "fc_counters_cmds_recv": 118, 
    "errors_error5_hour": 1, 
    "errors_error7_error": 11, 
    "errors_error4_minute": 56, 
    "errors_error1_second": 41, 
    "errors_error3_minute": 41, 
    "errors_error3_day": 9, 
    "fc_crc": 195, 
    "hskp_pwr1_adc_data_sa_short_circuit_current": 0, 
    "errors_error2_hour": 1, 
    "hskp_pwr1_rtcc_second": 18, 
    "hskp_pwr1_rtcc_hour": 2, 
    "hskp_pwr2_adc_data_adc_sa_volt_56": 0, 
    "status_2_reserved": 7, 
    "errors_error6_hour": 1, 
    "acb_pc_data2_tmps_tmp2": 0, 
    "hskp_pwr1_adc_data_reg_sa_volt_1": 0, 
    "hskp_pwr1_adc_data_reg_sa_volt_2": 0, 
    "hskp_pwr1_adc_data_reg_sa_volt_3": 0, 
    "reserved": 0, 
    "acb_pc_data2_tmps_tmp4": 0, 
    "hskp_pwr2_adc_data_sa_short_circuit_current": 0, 
    "pid": 240, 
    "hskp_pwr1_rtcc_month": 5, 
    "fc_counters_reboots": 98, 
    "hskp_pwr2_rtcc_day": 9, 
    "errors_error7_minute": 6, 
    "acb_pc_data2_acb_mrm_z": 30201, 
    "acb_pc_data2_acb_mrm_x": 12289, 
    "acb_pc_data2_acb_mrm_y": 4865, 
    "fc_counters_uart1_parseerrs": 251, 
    "errors_error2_error": 11, 
    "errors_error4_error": 11, 
    "src_callsign": "WJ2XNX", 
    "hskp_pwr2_accumulated_curr_bat2_rarc": 0, 
    "hskp_pwr1_adc_data_adc_sa_volt_56": 0, 
    "errors_error2_minute": 25, 
    "fc_counters_uart1_recvpkts": 156, 
    "status_2_payload_power": false, 
    "acb_pc_data2_rtcc_hour": 0, 
    "hskp_pwr1_rtcc_day": 9, 
    "hskp_pwr1_adc_data_power_bus_current_1": 0, 
    "hskp_pwr1_adc_data_power_bus_current_2": 0, 
    "acb_pc_data1_rtcc_year": 25, 
    "hskp_pwr2_adc_data_reg_sa_volt_1": 0, 
    "hskp_pwr2_adc_data_reg_sa_volt_3": 0, 
    "hskp_pwr2_adc_data_reg_sa_volt_2": 0, 
    "acb_pc_data1_rtcc_second": 16, 
    "hskp_pwr1_adc_data_bat_2_volt": 0, 
    "fc_salt": "elfa", 
    "fc_counters_porst": 2, 
    "errors_error3_second": 21, 
    "hskp_pwr2_adc_data_bat_1_volt": 0, 
    "src_ssid": 0, 
    "acb_pc_data1_acb_mrm_x": 0, 
    "hskp_pwr2_bat_mon_1_cur_reg": 0, 
    "acb_pc_data1_acb_mrm_z": 0, 
    "beacon_setting": 14, 
    "acb_pc_data1_rtcc_minute": 51, 
    "fc_counters_vu2_off": 23, 
    "hskp_pwr2_rtcc_second": 72, 
    "hskp_pwr1_rtcc_year": 25, 
    "hskp_pwr1_bat_mon_1_avg_cur_reg": 0, 
    "errors_error4_second": 55, 
    "errors_error2_day": 9, 
    "hskp_pwr1_bat_mon_1_temperature_register": 0, 
    "acb_sense_adc_data_current": 13081
}
{
    "hskp_pwr1_bv_mon": 8, 
    "errors_error4_hour": 23, 
    "fc_counters_fcpkts_fm_radio": 192, 
    "errors_error3_error": 11, 
    "radio_tlm_bytes_rx": 1798217472, 
    "acb_pc_data2_tmps_tmp3": 4992, 
    "errors_error3_hour": 23, 
    "hskp_pwr1_bat_mon_1_acc_curr_reg": 313, 
    "hskp_pwr2_adc_data_power_bus_current_1": 32, 
    "hskp_pwr2_adc_data_power_bus_current_2": 0, 
    "fc_counters_intrnl_wdttmout": 0, 
    "hskp_pwr2_accumulated_curr_bat2_rsrc": 3, 
    "errors_error1_minute": 35, 
    "hskp_pwr1_accumulated_curr_bat2_rarc": 100, 
    "fc_counters_badcmds_recv": 0, 
    "hskp_pwr1_bat_mon_2_temperature_register": 5376, 
    "acb_pc_data2_ipdu_mrm_x": -67, 
    "hskp_pwr2_bat_mon_2_avg_cur_reg": 182, 
    "fc_counters_vu1_off": 15, 
    "radio_cfg_read_radio_palvl": 145, 
    "status_2_htr_alert": true, 
    "hskp_pwr2_rtcc_minute": 87, 
    "hskp_pwr2_bat_mon_1_volt_reg": 26144, 
    "errors_error6_error": 11, 
    "hskp_pwr2_bat_mon_2_temperature_register": 5568, 
    "acb_pc_data1_rtcc_day": 8, 
    "errors_error7_day": 8, 
    "acb_pc_data1_tmps_tmp1": 3840, 
    "acb_pc_data1_tmps_tmp3": 4992, 
    "hskp_pwr2_adc_data_adc_sa_volt_12": 397, 
    "acb_pc_data1_tmps_tmp4": -128, 
    "hskp_pwr1_bat_mon_2_avg_cur_reg": 63, 
    "errors_error5_minute": 1, 
    "hskp_pwr1_accumulated_curr_bat2_rsrc": 100, 
    "hskp_pwr2_bat_mon_1_avg_cur_reg": 550, 
    "acb_sense_adc_data_voltage": 994, 
    "errors_error6_second": 38, 
    "status_1_reserved": 3, 
    "radio_tlm_bytes_tx": 2899201, 
    "acb_pc_data1_rtcc_month": 5, 
    "hskp_pwr1_adc_data_adc_sa_volt_12": 473, 
    "fc_counters_vu2_on": 17, 
    "radio_tlm_rssi": 0, 
    "hskp_pwr2_rtcc_year": 25, 
    "hskp_pwr2_adc_data_bat_2_volt": 701, 
    "errors_error5_error": 11, 
    "status_2_9v_boost": false, 
    "fc_counters_badpkts_fm_radio": 196, 
    "hskp_pwr1_rtcc_minute": 86, 
    "hskp_pwr1_accumulated_curr_bat1_rarc": 1, 
    "errors_error2_second": 83, 
    "acb_pc_data2_ipdu_mrm_z": -159, 
    "acb_pc_data2_ipdu_mrm_y": 48, 
    "acb_pc_data1_tmps_tmp2": 10752, 
    "acb_pc_data2_rtcc_month": 5, 
    "fc_counters_wdpicrst": 60, 
    "hskp_pwr2_bat_mon_2_acc_curr_reg": 946, 
    "hskp_pwr1_adc_data_bat_1_volt": 783, 
    "hskp_pwr2_tmps_tmp1": 3728, 
    "hskp_pwr2_tmps_tmp2": 3408, 
    "hskp_pwr2_tmps_tmp3": 5328, 
    "hskp_pwr2_tmps_tmp4": 0, 
    "acb_pc_data1_ipdu_mrm_x": -160, 
    "acb_pc_data1_ipdu_mrm_y": 36, 
    "acb_pc_data1_rtcc_hour": 25, 
    "acb_pc_data2_rtcc_day": 8, 
    "hskp_pwr1_tmps_tmp3": 3760, 
    "hskp_pwr1_tmps_tmp2": 3856, 
    "hskp_pwr2_bat_mon_1_temperature_register": 5536, 
    "acb_pc_data2_tmps_tmp1": 3840, 
    "hskp_pwr1_bat_mon_1_cur_reg": 633, 
    "hskp_pwr1_tmps_tmp4": 3776, 
    "hskp_pwr1_bat_mon_1_volt_reg": 26176, 
    "acb_pc_data1_ipdu_mrm_z": -26, 
    "errors_error6_day": 8, 
    "fc_counters_errors": 235, 
    "status_1_safe_mode": true, 
    "hskp_pwr2_adc_data_adc_sa_volt_34": 411, 
    "ctl": 3, 
    "fc_counters_sips_ovcur_evts": 0, 
    "errors_error7_hour": 24, 
    "hskp_pwr2_bat_mon_1_acc_curr_reg": 275, 
    "hskp_pwr2_accumulated_curr_bat1_rarc": 1, 
    "status_2_htr_force": false, 
    "errors_error1_hour": 23, 
    "hskp_pwr2_rtcc_hour": 25, 
    "hskp_pwr2_bv_mon": 8, 
    "dest_ssid": 0, 
    "fc_counters_vu1_on": 9, 
    "acb_pc_data2_rtcc_minute": 88, 
    "hskp_pwr2_rtcc_month": 5, 
    "errors_error4_day": 8, 
    "hskp_pwr1_tmps_tmp1": 2992, 
    "status_1_early_orbit": 8, 
    "errors_error1_error": 11, 
    "fc_counters_brwnouts": 59, 
    "dest_callsign": "W6YRA9", 
    "acb_pc_data2_rtcc_year": 25, 
    "hskp_pwr2_bat_mon_2_volt_reg": 26240, 
    "hskp_pwr1_bat_mon_2_cur_reg": 91, 
    "hskp_pwr1_accumulated_curr_bat1_rsrc": 1, 
    "hskp_pwr1_adc_data_adc_sa_volt_34": 529, 
    "errors_error1_day": 8, 
    "hskp_pwr1_bat_mon_2_acc_curr_reg": 7137, 
    "errors_error6_minute": 16, 
    "errors_error5_day": 8, 
    "errors_error7_second": 80, 
    "hskp_pwr2_accumulated_curr_bat1_rsrc": 1, 
    "hskp_pwr2_bat_mon_2_cur_reg": 231, 
    "acb_pc_data1_acb_mrm_y": 40, 
    "errors_error5_second": 3, 
    "acb_pc_data2_rtcc_second": 67, 
    "fc_counters_cmds_recv": 33, 
    "errors_error5_hour": 24, 
    "errors_error7_error": 11, 
    "errors_error4_minute": 81, 
    "errors_error1_second": 48, 
    "errors_error3_minute": 66, 
    "errors_error3_day": 8, 
    "fc_crc": 130, 
    "hskp_pwr1_adc_data_sa_short_circuit_current": 2, 
    "errors_error2_hour": 23, 
    "hskp_pwr1_rtcc_second": 54, 
    "hskp_pwr1_rtcc_hour": 25, 
    "hskp_pwr2_adc_data_adc_sa_volt_56": 444, 
    "status_2_reserved": 3, 
    "errors_error6_hour": 24, 
    "acb_pc_data2_tmps_tmp2": 10752, 
    "hskp_pwr1_adc_data_reg_sa_volt_1": 783, 
    "hskp_pwr1_adc_data_reg_sa_volt_2": 790, 
    "hskp_pwr1_adc_data_reg_sa_volt_3": 790, 
    "reserved": 0, 
    "acb_pc_data2_tmps_tmp4": -256, 
    "hskp_pwr2_adc_data_sa_short_circuit_current": 2, 
    "pid": 240, 
    "hskp_pwr1_rtcc_month": 5, 
    "fc_counters_reboots": 96, 
    "hskp_pwr2_rtcc_day": 8, 
    "errors_error7_minute": 25, 
    "acb_pc_data2_acb_mrm_z": -45, 
    "acb_pc_data2_acb_mrm_x": -31, 
    "acb_pc_data2_acb_mrm_y": -79, 
    "fc_counters_uart1_parseerrs": 241, 
    "errors_error2_error": 11, 
    "errors_error4_error": 11, 
    "src_callsign": "WJ2XNX", 
    "hskp_pwr2_accumulated_curr_bat2_rarc": 3, 
    "hskp_pwr1_adc_data_adc_sa_volt_56": 480, 
    "errors_error2_minute": 50, 
    "fc_counters_uart1_recvpkts": 119, 
    "status_2_payload_power": false, 
    "acb_pc_data2_rtcc_hour": 25, 
    "hskp_pwr1_rtcc_day": 8, 
    "hskp_pwr1_adc_data_power_bus_current_1": 5, 
    "hskp_pwr1_adc_data_power_bus_current_2": 0, 
    "acb_pc_data1_rtcc_year": 25, 
    "hskp_pwr2_adc_data_reg_sa_volt_1": 782, 
    "hskp_pwr2_adc_data_reg_sa_volt_3": 783, 
    "hskp_pwr2_adc_data_reg_sa_volt_2": 783, 
    "acb_pc_data1_rtcc_second": 7, 
    "hskp_pwr1_adc_data_bat_2_volt": 713, 
    "fc_salt": "elfa", 
    "fc_counters_porst": 0, 
    "errors_error3_second": 23, 
    "hskp_pwr2_adc_data_bat_1_volt": 766, 
    "src_ssid": 0, 
    "acb_pc_data1_acb_mrm_x": -121, 
    "hskp_pwr2_bat_mon_1_cur_reg": 588, 
    "acb_pc_data1_acb_mrm_z": -34, 
    "beacon_setting": 14, 
    "acb_pc_data1_rtcc_minute": 88, 
    "fc_counters_vu2_off": 23, 
    "hskp_pwr2_rtcc_second": 86, 
    "hskp_pwr1_rtcc_year": 25, 
    "hskp_pwr1_bat_mon_1_avg_cur_reg": 559, 
    "errors_error4_second": 64, 
    "errors_error2_day": 8, 
    "hskp_pwr1_bat_mon_1_temperature_register": 5440, 
    "acb_sense_adc_data_current": 25
}
```

## Work in Progress:
**Feature importances iterative distribution flatening**
[Merge Request !30](https://gitlab.com/crespum/polaris/merge_requests/30)

In this merge request, we're working on augmenting the dataset with new features. The new features are created using the transformers that are fed as an input to the polaris learn module.

Using this ready to use transformers developed by Red ([fets](https://gitlab.com/redsharpbyte/fets)), we are augmenting the dataset. With the help of transformers we're creating a feature engineering pipelines in sklearn and transforming the original dataset using them.

The objective of this merge request is to create a process to iteratively obtain the feature importances of an augmented dataset along with the previous most important features. 

As iterations grow, the more relevant features are selected, stored and the features importance distribution will be flatened gradually.

The flow of polaris_learn will be something like this:

* Input telemetry data along with the context data
* So we have N parameters which will be our target variables.
* We have to find out the best optimized pipeline for the given dataset.
* Ensembling the models
  We can use stacking ensemble method where we can have different regression models like XGBoost, ADABoost or neural network.
The idea of stacking is to learn several different weak learners and combine them by training a meta-model to output predictions based on the multiple predictions returned by these weak models. So, we need to define two things in order to build our stacking model: the L learners we want to fit and the meta-model that combines them.

* Managing models and Packaging them for reuse.

## Playground Full of Jupyter Notebooks:
I played with a lot of machine learning libraries and the models [here](https://gitlab.com/crespum/polaris/tree/17-preprocessing-of-output-json-data-for-polaris-learning/playground/notebooks).

Worked on over sampling the Mars Express Power Dataset with following methods:


| Sampling Method |Random State | Mean Squared Error |
|:-------------:|:-----:|:-------:|
| SMOTE (Synthetic Minority Oversampling Technique) | 0 | 0.55
| SMOTE | 2 |0.58
| Random Over Sampling  | 0 | 0.49
| SMOTETomek | 42 | 0.588
| ADASYN (Adaptive Synthetic) | 0 | 0.56


## What's left to do?
* The next goal is to visualize the feature importance distribution using heatmap, finding out the dependencies between the features using the graphs.

* A python module for polaris viz which will help to output data in a JSON format to feed into data visualization tools like Grafana. It can also have have containers to visualize the data, highligting the outliers and generating alerts for it.

* In future, we can have customized Grafana widgets for health keeping and diagnostics of the satellites.

* Merging the TLE data with the telemetry data of the satellites 

* Machine learning lifecycle for polaris

* Automating the learning part of polaris

* Testing module and proper documentation

Learnings
===
I acquired several skills during the Google Summer of Code including:
* Much more comfortable at reading, writing and debugging valid python code.
* Learned important differences between python2 and python3. Improved proficiency in the migration process and setting up the virtual environments for the projects.
* Improved my git skills
* Got acquainted with Test Driven Development (TDD) and how it’s useful in pragmatic programming.
* Deepened my knowledge about machine learning models and libraries
* Learned a lot about the space technologies (SatNOGS)
* Learned about importance of communication while working in a team.
* Maintaining journal for tracking my work


Credits
===
It is a very pleasant experience for me in polaris and that was made possible again by very awesome developers of Polaris, SatNOGS and Libre Space Foundation for making me feel welcomed and always helping me out when things were not going as expected.

A special thanks to all my mentors Xabi, Hugh and Red for patiently listening to me, teaching me even smallest of things and also having some chats and video calls over jitsi with me despite their busy schedule. I got to learn lot of things in python, version control and machine learning. I'm so lucky to have all of them as my mentors😀.

I would also like to thank a developer from SatNOGS, Patrick, for helping me with the version control and python environment setups. I'm really very grateful for this. Thank you so much.

I would also like to thank people from SatNOGS and Libre Space Foundation for giving us solutions whenever we were stuck.


Apologies
===
English is my pretty weak spot. I’m sorry if there are any spelling mistakes or grammatical errors or any comprehension problems.(It's just my 3rd blog post, so I know, I have a lot of scope for improvement).

I had lot in my mind and I didn't make any plans on what to put on blog. I just went with the flow, apologies if I missed something important.(Please let me know in the comments and I'll surely work on it.)

And sincere apologies to all my mentors and thanks for bearing with me and responding and explaining to my silly and stupid doubts.


Future Work
===

We have only setup a base for us to work upon there is a lot of pending things that needs to be done before it can be put into use.

In the future, we will develop the whole machine learning lifecycle for the polaris, which can be used to track the parameters of the various machine learning models along with the metrics like mean squared error, mean absolute error and root mean squared error that we can use to assess the performance of the machine learning model and experiment with it. And packaging the given machine learning model so as to make it reproducible and experimentable, packaging machine learning models that can be used in a variety of downstream tools — for example, real-time serving through a REST API or batch inference on Apache Spark.

We're also planning for automating the whole process of  fetching the data, decoding it, storing it in reliable storage system, the feature preprocessing, feature selection, new feature creation and selection of the best machine learning pipeline for the given dataset. 



My Next Plans
===
* Contribute to open source as much as possible
* Continue working on Polaris
* Attend tech events around and abroad to gain maximum experience
* Mentor Google Code-in 2019
* Mentor Google Summer of Code 2020

Connect with Me
===
Email: [malshikhareaditya@gmail.com](mailto:malshikhareaditya@gmail.com)

Riot: [Aditya Malshikhare](https://riot.im/app/#/user/@thepairedelectron:matrix.org)

LinkedIn: [Aditya Malshikhare](https://in.linkedin.com/in/aditya-malshikhare)

GitHub: [thepairedelectron](https://in.linkedin.com/in/aditya-malshikhare)

GitLab: [malshikhareaditya](https://gitlab.com/malshikhareaditya)

Portfolio: thepairedelectron.github.io

Conclusion
===
It was really a great summer, full of learning ("machine" learning :smile:) for me. I learnt a lot throughout the timeline of the GSoC especially how to use communication platforms efficiently, work on a open source project, collaborate with other developers, maintain a journal and most importantly how to not loose my hopes anytime (seriously I doubted myself many times because I messed up my working tree, my workstation and lot of commits that I pushed into the project repo). But I managed it till the end with the support of mentors.


In the future, I would like to continue my work on the polaris, and I also have a few crazy ideas to implement. Stay tuned, and see you in the ether! Peace till then.:v:

