# ULL High-Throughput FPGA-UART Bridge with Speculative BRAM FIFO

### Design Specifications
1) FPGA Board:     Arty-S7
2) UART Baud Rate: 6,000,000Baud
3) UART Protocol:  8P1 (1bit Start, 1bit Stop)
4) Clock Rate:     Use 100MHz On-board CLK to synthesis 288MHz CLK with MMCM
5) Sampling Rate:  16x Oversample
6) FIFO Buffer:    BRAM with 4096Addr, 3 Pointer Speculative Write with Rollback, FWFT with CDC

### Low Latency Optimization
1) Double-edge detection to cut down 2FF input buffer to 1.5ticks delay
2) ULL write-to-read FIFO using speculative write, FWFT and almostFull
  - Speculative write to send data to FIFO when last data bit is read, and immediately commit when stop bit is validated using combinational logic
  - FWFT to ensure data appears on read side of FIFO when commit is asserted
  - almostFull flag to allow for burst read/writes
3) Synthesised 288MHZ Clk to reduce tick cycle
4) Calculated parity bits concurently with each data bit read using XOR, to mitigate parity bit check delay
5) PBLOCK and usage of pipeline techniques to hit timing constraints
6) Lookahead registers to reduce logical negative slack in clocked blocks

### Error Mitigation
1) Parity check flag is appended to output data from UART RX
2) Stop bit detection either asserts commit or rollback to speculatively written FIFO
3) 2FF buffer on all CDC / Async inputs, between RX pin and UART RX, and between FIFO clock domains
4) 288MHz Clk specifically chosen to mitigate framing error at 6Mbaud
5) 4096addr deep FIFO to allow for >7ms accumulation at max throughput (6M Baud)

### Current Implementation Timings
Worst Negative Slack:   +0.045ns<br/>
Worst Hold Slack:       +0.130ns

### Updates
> Currently only implemented receiver HDL and met implementation time constraints, have not tested on Arty-S7
