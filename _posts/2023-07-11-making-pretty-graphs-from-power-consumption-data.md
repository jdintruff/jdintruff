---
layout: default
title:  "Making Pretty Graphs From Power Consumption Data"
date:   2023-07-11 02:07:49 +0000
categories: jekyll update
---

# Making Pretty Graphs From Power Consumption Data

For some reason or another I wanted to measure how much power was being consumed by my desktop setup, as well as my router/modem combo. I'm regurgitating this from memory because I barely documented anything, but what follows is what I remember.

## Hardware

After buying a couple cheap smart outlets I ended up deciding to use the [Sonoff S31]([SONOFF S31/S31LITE - Power Usage Monitor Plug Wifi Smart Socket](https://sonoff.tech/product/smart-plugs/s31-s31lite/)) which goes for \~\$10 on [Amazon](https://www.amazon.com/Sonoff-Monitoring-Compatible-Assistant-Supporting/dp/B07YDC6D4D). The reasons for this were:

* It's based on the ESP32 microprocessor which I've used many times and am quite comfortable with

* There seemed to be others who had flashed Tasmota to it and had a lot of luck

* Tasmota has native support to push metrics to an MQTT broker and I happen to already run one of those on my local network. Ultimately I used Telegraf to forward that data to InfluxDB where I made pretty graphs, more on that in the software section

### Teardown

Taking apart these devices is pretty simple although at the time I had trouble finding anyone who had thoroughly documented that process. What follows is a step-by-step guide showing how I did it.

<img src="https://lh3.googleusercontent.com/pw/AIL4fc_qMISPIL3gQJ_Oog02E7ZKQhmbLY5C3tJUILE_fdEighDsDVCUr9HuN-cZm75tq7kNuhdPUUJsP7hOJ942BtoHVNq_RT5v3qhaQh1o_sRfj36Xk7fJF14lYrjGsGJDjhvRgcC-z-zjUdwgSiuNk8Cx4XRrVcqOcYgZzE5SKo5geyBLEOrya5L5pEZ2Aq6iAs2kn8B-CQQQnq0Syiv9uMISLYDEdcteGTHWoawGfCFGbXvOJLcih1Yay4wiXSEQcmyjG9x9cxQL-dx2oQ8_b0nMB2z2iPlcnwH2Gsok1cR9QWi00-LjduNQ4tq7lFyHv0asxw2wIOPuGic_TYRFJQoKK5_WvIBJkaeNGbGNWIswkkfMxDnW2SbePDmpBIjlcnqiaPNhzmAnAObxjxD1VObL1ypgKGlW_foVt8f7PFfCBleT4gtnUGTNHFZjAMSpwTuW5C0OTfV7svNOIcAs9AKfjyEjrrpxIIN1Wc3HHJKz4aFlqQbpKzOdYZvCEtavfEdm6HCgAqnp3DXjYx_go5BSZWs4kjSOBFzg9FzVqXKFuW0YuBg8GqmEbLboU1h-WAmO-a0qNwb9KhDTSWYNGcAaxJAZa3g77X1hgWtXnNmkfRCg6yRZwrEgnE7rWv0RjCo7To6LeDQtHjXv7EDzabExC7HATo8vCpmYlEW8pU-2tEugtaY9OE09cBWW-Tns0Ma530gKfTQDmDS8DzdCPsYQJPJCAku5tkaYt5_JNliKUFKrwbeWU5quLSezQmfNqoI-QgKzyTI54_a_N85dkaMVsZ5fAhXyddn6_Fkp1xKnicntZtilu-N6fX5rhM8WO3x_3mJiuKVAwgkN-eoBCd-b00vzOPhTMMdu2GOW7l0xTCiPdiFnpv5V_dsuWl5IoWwNpwhRVxKtaRH-_3fh6xVcflQ=w1545-h2052-s-no?authuser=0" title="" alt="" width="315">

1.) Carefully pry off the grey endcap with a flathead screwdriver. The plastic is pretty soft so be gentle.



<img src="https://lh3.googleusercontent.com/pw/AIL4fc9U7NmNmQcEAqd_tAVjlMZgxoFr0h3GfcCIh679s3qJ5h-jM3dzcbUGTo98LrT4R8RUwcm42yBjOCCZxocA8xdUZuo7NNVWhoMqG_kvH1LrNw2aLqcDK5voh4Op-TIEt9B0cS3cEsKMBYbe0nfRpTEtKPrs4BUIpykhvni1DLo0hKIGf__fPqohnmW3J9IdqRIBHm72A5EkKY1wCnbM6o3jADT_MvLi_CTi-1jXVXJP-svUpU2trbjhmoMDPsLw-S2GJ12Ld4P5fI3jAzX2kNn-SsjCtg0-1z8_8ycGaNRgNnvjaSrCalgrzIJiqpTkpXi7cSN3owTTRbUF3i4ZpnayPjsfGBs0u6xtIXVZPpFxMGT1i3e-FQYM0_WYB-P3qFNP9cz46WhAOkhrCGSdZ9UE-kKfG_CzMuwE3yHUaA6XLIhj9pyesCIef1yixSJXjVRyD_ZI-qQhNYeuXs0v_nc3A663ToWcQT0zURsfN6LuwuhE26pv9xvJe3QNJWWlLpK5rO5smUbnSWWpUMYH5GsQlAUDKBC8hmakh2A1LbKiczLGBqwro_42OC9stKZLJIWZciJqfJAHU3oKPuPHYBFMtA2MTN2vT991Ui9leAwQPFa3LBrJFSSNEsbg529F_HHbdgE5KRWsEO-xOHCClZSbExP_gaTN4Loxsj0Gf-asAZmA0fxi_sRZ0TGBqCTU-uAVPfrb3MmKIyK7P2sOzRCQbsV6ryrsQq2TRg13hWI_gN4wsHklNO5ssqXZNRVY3u5phUTziWZt5cZy8B8Nby_t-5LPc-JdkOKHBgF9asvXnYl-Nsa-KSdenjN8eThc5Jrpkdwm2kDgJYzEAgokGlDUGHqGbS8ZbLl_rIidXqT3x0MM6m-eMHz1fLv-VHWCR7QQ5Izr43P0xAoVVY0W4kYb7F8=w1545-h2052-s-no?authuser=0" title="" alt="" width="311"> <img src="https://lh3.googleusercontent.com/pw/AIL4fc9HEhDg5JBHnuN8ZxQJiK3q_zsHwUtXDNiooGX0RnR5H8qAhifpM2f80RNCHbAzAGKIpfIhE16ykCB_41r0R04ZffHsMm184EqOSZBOQpX9O00VrFzVKbCHHqrrKoLDSEv4zgQBoaiYPvactPU1jyMcDccMjr9_Ke6UGVzu_PgkT69h-uCQqxqAtSKmpQ6jafQhxDk8UrE6saJ5ZELnzyfs_AMIge0AzKT7eKYrGhq86kxTWN07W7WAnYh2fe9ZE4sB0fgvY4zi1RmBZSlidG6ZEx7z-Z4V7BR3AtFr_pDFuAMMUxJNTN0Q411d7YDOhxsovomkObsyX4uVDEbAB9OxWbL9P0VnLJX0RxW7ceLw--r4Z8qB0blWR2WiSOtaCJywsLoJWRYVrzZzd5_IVVN4rYOvAcDfceYNazf_Ne54EhaPRV9cyYmKm_Vncm8iYXtsgUdb1Y8mYK_tZfA7dibpK4IEzQzEVTkj3Zm3rc82t35bC3KWDGniDWY7NnJ8K9eQLf2-cVSdewIhdZThfcV6F13_elMbX2uj-eHR_0-lomWUfN1pQwfYqIQ88Pp0aiP8Gi1K_538LD5KCd2xA1J5-kX1Fk-olPvBWxvZMuuP4VxXSwmVfPGU87CB0UJw2QKc-b3AL19Rpk7XOI6lYkJsqQi3rAgcatowCXvYNTUgzEpAyNYFp-ZxqyMx5FITvAJLWmR1jkBDGMsnPZf56BUWmnOALmQ7waFDwCqIAOSb8g4xMx1WhfYBdQfRlRMFTnOMNnHxxqr05tpqm7cBOSTKbBOXbImAoZvxeI0pzaICGAig5qoOSXfDrVqEL62M4BAb5iqbDJ_NU9NQJ6gvgWZBAbfi9zjaaqE-OV9b5e-bzoyP5cWFoRtALGteWaaNd-xuJuUKOMynExPTfkVinABvuXo=w1545-h2052-s-no?authuser=0" title="" alt="" width="312">

2.) Slide off these two little bars, then plug part should just fall out from what's left of the plastic casing.



<img title="" src="https://lh3.googleusercontent.com/pw/AIL4fc_NO-sT0t4rn3VxgDMp5lUJMEtXk4LVPSbkzY2K1NuzwGEBWknuj6O8gYLQd4w5RAhhhEpxky7rNEtIhttymglnqHMZ2lyO1e75bqJjvFT6b_O5YeR2fC7Wa2fxa8_TUgxk7xeR6j9YgJHeu-zTHtMOcenotiV3AdMmI4IGi7cVaX87DZJDWWGdvPbxMPpqTesONLytvxIRgqs1Ohj9I53xvg7whR0BBLDjUSgQo5lnCeyUSYfmtBDlwqnJCxCbBRmdjcR5tbYRwy1dfQgDtGRRjqwlylgyBo-WIhAjpI6oWzcB7k8yrfwiAG7N2fOKxtqQf0TdJLs6z2HMAJxIAklwNH7d3cei_YBmYQ01B74CtLJ1Rfi-xLAQ0vNIHSaYYNgq4_cq14AXVsDl2JY9I-fX1ut10QznjyWbjq4sg5Tg656RjOQ4RvKb9sngXobJCY43uhpy58Kc75L4xDLTPdIa66dRqiVcecvtVkVoSFsVtlI5hToZdcT5-ljjlGYqBFHbi6qLEBCon_3wMb_6ejeaISO46zbuWaW4GudwW4KqMCjuJ-icsUQuwsuYXoixnAf2Grb1VV9sNfvKtCQi7lS8rYNxC8aPY0tTppdwY5V5yJoKfj41w1eKkuFG5UZUG_Syxqvrg83Aj47y2O-HK_JkHQNoVNRRn8HCsHHsn_ViRz1EHf8jNgP2vPr67P4L-KjIxNkezBb6UiHMCJnDm3OVX4yRMGYMySKiGDTlO_yXNblwpha8Ry1FpPAD0kB66lF-mNWzXQK90gPvAKfPNFQWM7VrvVNBTzH33wlSD5JB9nXWAVtmmeQ2wisROn3t64dDJqKraUsv_eQYIKlarl_qDJC0EIPOlJGCwNVmK89OPrX8dtp0rP65W5ZJ84bSu4P2rCExkkx0c6Vwbm0bldd9g5w=w1545-h2052-s-no?authuser=0" alt="" width="416">

3.) Solder on four leads to the UART contacts. I found it was easier to glob a little solder on each pad before trying to attach the cables. 



![](C:\Users\jackd\AppData\Roaming\marktext\images\2023-07-10-19-25-46-image.png)

![](https://lh3.googleusercontent.com/pw/AIL4fc-FYACLODEYwudJoMGJ54QaZ31jjo096c8SQ_CQ3AK0iobV4cm5qR9Bsylx-UX8nWBl_4V5YMyjHiDRQ0w0A476tYppHsxta4f4oG6nPtxBp-e8dBGNPD-TY-OEERHaV5VlJvU253dtYPJDmvBIqlfEvJt7WndDtE67wol0SusaGMqas5mRl7PjvYScqsm8Eh_0b6y_975-05vZl46-WrevjdyLTtSSl0kzxShDYunWBs5R2izA12woA1mPNxxnmwKim0bjnUndcMhzUZmOaeS5CSO33bSn5LUEXjjOPyXGEJdrHrIlyXImXpm2Unucoqvh7SZWrGvfIeeiNnQPA6e27AeuykzTw7YCUMI3BMFqe-6JosRV_pWMQeYEDO4rLPKXFBGOTtMX5Rzdg53TJNxtOaRXHDdmmnGCZmb4qulpdQ3WUmj3sQ8sCopg7GiILJ3oedPNnIsj7uJXDy0h0bCyOuoFT1NADRL7ObnQUCf9qhWb8DCmBuDFolW_p0Ze6YrLidPf55lI4YyE2_z8hHzCWia_yE2H5elTgJD6KHvXsJxH0bAZRjQhX1H71hylGkxuSmZCfhyZk61WA4bK8hjOOHrOHrb45fjPB0Q_LJ1XvCm_H40ACKWbAq2mVXq5y_lGCFqL2xbq0vkKlpj-624-B1UgEAm3epkMIcQt-zyyWIp0bQzoaqcKGAb8tOZOl5yYizfzSNu41HsCV_gMR9ZoL8TGsCNj-81EaY63hMt-Kjd-bIsSp9YFh53yLfWvMZNeBi3K_95FNRGbsA2_RrIS1NbnoY-FFZ5SpFH93Ou9Wd7iTLgHKspTGh97SYw-Rcfky8JHNO0Gz081fMPbHj2HbaYHMyVBc__T2xOPMhapiCLxHMckat11hthFpLEB0mX0oJF7btXWTx2sAYqC0VFvCf0=w1545-h2052-s-no?authuser=0)

4.) I used a [USB-C to serial cable]([FTDI Serial TTL-232 USB Type C Cable [5V Power / 3.3V Logic] : ID 4364 : $19.95 : Adafruit Industries, Unique & fun DIY electronics and kits](https://www.adafruit.com/product/4364)) and the datasheet pictured above helped me figure out the wiring. RX on the device (blue wire) connected to TX on the serial cable (orange), TX on the device (green wire) connected to RX on the serial cable (yellow). Naturally, ground (brown wire) goes to ground (black) and VCC (red wire) goes to VCC (also red).



![](https://lh3.googleusercontent.com/pw/AIL4fc-Buvt6M7wtcG39UGiVNy5etPgr72lx-psxo0CSJMRuKagnPFkw51oGfRv7Ny9ThJVBh0Nh9KuE_qsMfTxq9tVRU9cJbGbp919ZlqssuHLtKI0k_3odc8aa5PeDE_yiFcuElsGM23DiR8XzWa8PqkvBU80cNtKbdoKDUJtl18aEyjrHXmov_HwrwB2j9aq1FpmLq6sShkX29hkdMsbEAAb78iUJ5tE4NhMks3pHDlPO87XXLQ-0_MEvvjtvTOeeMsFmaljp-aJnqNKw7qQ98CzRsOoyx9wzM-0g20HScdqKGVw8N2Ne-2Wnp0IH6KpuC83k56kDEONGTTPnMjuR28soTBcRATNAFQSSvEdsBcmD-4AQgwkF3xHmYceuUFd5o5CYa5LvNXNpp8g0YmCiv3KErzK-mxaNCafYfBoIZWBQXzi-hjJynD5n-avIT--YAyEig4ziRCl4wGhFCxr6NUr9L0pGdW-KcOsRGTCWyE8NsQh9e88X8151m1RPDn7iq69zgXrNfbrEAwAq2tanCAle4L7AAYklZCz9r89gt_m6nAROilnGOqbKo0QWepkZloMILy3McgymL_ydBPeFStMXuJSxAh9jiKzgzu9r6fLZ6kUU09Iqn7C_o21d7hrepZCMPv6ArUkQ9U-hcw8aBTbQUlsukCbRBqgtD1Z0uWKw51ETEUOVNEPiMDDbKDoWDAK-PI41xJrfewppV7DiWKGVSjtkfYQCIFJ6m5gc4UbnmYvzYUzdt5LPQlMf_uGC7ozrXdLsgZMQ93PZs3M9VaQ_ZFLwUedLREtFtLyZ_D2fz-jC5T4Xis4o1Z6VwkyxzFuXKUppKd9tgQZK0jJf2eOXHLrVWUqbZMf7nMRNQIYqc05QTZu4D_bcM7rBhcFVuKcSaflBr0k6_1mW0cPbFCdj9dA=w1545-h2052-s-no?authuser=0)

5.) Plug the cable into your computer.



### Software

1.) Install Tasmota from their [website]([Install Tasmota](https://tasmota.github.io/install/)). Makes it super simple and easy. Put in your SSID and password and then restart the device. Reassemble it, then plug it into an outlet.

![Screenshot 2023-07-10 194033.png](../../../../../images/Screenshot%202023-07-10%20194033.png)



2.) Once it powers up, locate it on your network. You should have a configuration page like the one above. Configure MQTT to point at your MQTT broker. Like I mentioned before, I also configured Telegraf to dump everything on my MQTT broker to InfluxDB but that's beyond the scope of this post.



![asdf](../../../../../images/Screenshot%202023-07-10%20194652.png)

3.) Pretty graphs!


