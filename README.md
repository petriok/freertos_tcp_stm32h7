# freertos_tcp_stm32h7
Test and benchmark FreeRTOS TCP on STM32H7

## Purpose of the repository
This repository was created to test and benchmark the FreeRTOS+TCP on an STM32H7xx board running at 480MHz frequency.

[Lauterbach](https://lauterbach.com) uTrace for Cortex-M debugger was used for tracing. 

## Dependencies
FreeRTOSv10.3.1 was used as a basis and the [pull request #79](https://github.com/FreeRTOS/FreeRTOS/pull/79) was checked out on top of it. Minor PHY related modifications were made in the common phy handling.
The STM32H7xx HAL drivers can be acquired from the [STMicroelectronics](st.com) site. The driver package used in this repository is STM32Cube_FW_H7_V1.7.0. A custom BSP for the board used in testing the TCP stack is not made available in this repository.

[Iperf3](https://iperf.fr/) was used in benchmarking the TCP stack. Iperf3 server part as a FreeRTOS task by Hein Tibosch was obtained from a [FreeRTOS forum post](https://forums.freertos.org/t/freertos-tcp-iperf3-server/9574).

## Compiler options

Compiler used in this test was gcc-arm-none-eabi-8-2018-q4-major.
O3 optimization level was used to build the test firmware.

## Memory and caching

Both instruction and data caching was enabled. MPU was used to disable caching for the memory areas used by DMA. FreeRTOS core was running from the tightly coupled instruction memory ITCM. Except for the heap and DMA buffers, tightly coupled data memory DTCM was used for the the tasks.


## Results (client sends, server receives)

```

iperf3.exe -c 192.168.1.50 --port 5001 --bytes 100M
Connecting to host 192.168.1.50, port 5001
[  4] local 192.168.1.100 port 65331 connected to 192.168.1.50 port 5001
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-1.00   sec  9.50 MBytes  79.6 Mbits/sec
[  4]   1.00-2.00   sec  9.62 MBytes  80.7 Mbits/sec
[  4]   2.00-3.00   sec  9.50 MBytes  79.7 Mbits/sec
[  4]   3.00-4.00   sec  9.50 MBytes  79.7 Mbits/sec
[  4]   4.00-5.00   sec  9.50 MBytes  79.6 Mbits/sec
[  4]   5.00-6.00   sec  9.50 MBytes  79.8 Mbits/sec
[  4]   6.00-7.00   sec  9.62 MBytes  80.7 Mbits/sec
[  4]   7.00-8.00   sec  9.50 MBytes  79.6 Mbits/sec
[  4]   8.00-9.00   sec  9.62 MBytes  80.9 Mbits/sec
[  4]   9.00-10.00  sec  9.50 MBytes  79.5 Mbits/sec
[  4]  10.00-10.48  sec  4.62 MBytes  81.9 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-10.48  sec   100 MBytes  80.1 Mbits/sec                  sender
[  4]   0.00-10.48  sec  99.8 MBytes  79.9 Mbits/sec                  receiver

iperf Done.
```

### CPU usage

![Task statistics](/images/1/task_statistics.png)

### Task timing

![Task timing](/images/1/task_timing.png)

### Task stack coverage

![Stack coverage](/images/1/task_stack_coverage.png)

### Ethernet IRQ interval

![Ethernet IRQ distance](/images/1/eth_irq_distance)

### Task statistics tree

![Task tree 1](/images/1/task_tree_1.png)
![Task tree 2](/images/1/task_tree_2.png)

### Symbol chart

![Symbol chart 1](/images/1/trace_symbol_chart_1.png)
![Symbol chart 2](/images/1/trace_symbol_chart_2.png)

### Zoomed symbol chart

![Zoomed symbol chart 1](/images/1/trace_symbol_chart_zoomed_1.png)![Zoomed symbol chart 2](/images/1/trace_symbol_chart_zoomed_1.png)

Memcpy could definitely be optimized, in this test it is copying byte by byte.

## Results (server sends, client receives)

Here is something that goes wrong with retransmissions. I haven't investigated where the problem is. 

```
iperf3.exe -c 192.168.1.50 --port 5001 --bytes 100M -R
Connecting to host 192.168.1.50, port 5001
Reverse mode, remote host 192.168.1.50 is sending
[  4] local 192.168.1.100 port 65365 connected to 192.168.1.50 port 5001
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-1.01   sec   227 KBytes  1.84 Mbits/sec
[  4]   1.01-2.01   sec   214 KBytes  1.75 Mbits/sec
[  4]   2.01-3.00   sec   279 KBytes  2.32 Mbits/sec
```

From the console I see:
```
prvTCPWindowFastRetransmit: Requeue sequence number 20442 < 21902
prvTCPWindowFastRetransmit: Requeue sequence number 42342 < 48182
prvTCPWindowFastRetransmit: Requeue sequence number 46722 < 48182
prvTCPWindowFastRetransmit: Requeue sequence number 90525 < 91985
prvTCPWindowFastRetransmit: Requeue sequence number 112425 < 119725
prvTCPWindowFastRetransmit: Requeue sequence number 116805 < 125565
prvTCPWindowFastRetransmit: Requeue sequence number 118265 < 125565
prvTCPWindowFastRetransmit: Requeue sequence number 122645 < 125565
prvTCPWindowFastRetransmit: Requeue sequence number 124105 < 125565
prvTCPWindowFastRetransmit: Requeue sequence number 125568 < 127028
prvTCPWindowFastRetransmit: Requeue sequence number 131408 < 140168
prvTCPWindowFastRetransmit: Requeue sequence number 135788 < 140168
prvTCPWindowFastRetransmit: Requeue sequence number 137248 < 140168
prvTCPWindowFastRetransmit: Requeue sequence number 138708 < 140168
prvTCPWindowFastRetransmit: Requeue sequence number 143088 < 144548
....
....
````

### Task statistics and timing

![Task statistics iperf reversed](/images/1_reversed/task_statistics_reversed.png)
![Task timing iperf reversed](/images/1_reversed/task_timing_reversed)
