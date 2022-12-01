# Pico-ADS131M04

A library for ADS131M04 analog digital converter, built upon [Lucas Etchezuri's work](https://github.com/LucasEtchezuri/Arduino-ADS131M04) on the Arduino version. This fork works with raspberry pi pico and has some additional functionalities and minor bug fixes. 

## ADS131M04
ADS131M04 is a 24 bit ADC that transmits data over SPI. The main advantage of ADS131M04 over other older ADCs are: 1) ADS131M04 uses simultaneous sampling of 4 input channels and therefore avoids MUX-related data-rate loss (e.g., as in ADS1256), 2)ADS131M04 supports higher speed SPI (max 15.625 MHz per datasheet), 3) it has 24 bit read depth, 4) it has a max sampling rate of 32KSPS for all 4 channels, 5) it is available as of 12/2022. The max voltage input swing of ADS131M04 is +/- 1.2V.

## Clock module
SIT2024BM-S3-33N-8.192000 is a 8.192 MHz oscillator with 3.3V logic voltage. In practice any 8.192 MHz MEMS oscillator (XO standard type) should work.

## PINOUT
#### Main module
| PI PICO  | ADS131M04 | 
| ------------- | ------------- |
| 3.3 V (OUT) | AVDD 1 |
| 3.3 V (OUT) | DVDD 20 |
| GND | GND 2 & 19 |
| SCK 18 | SCK14 |
| MISO/RX 16 | DOUT 15 |
| MOSI/TX 19 | DIN 16 |
| CS 17 | CS 12 |
| DRDY 20 | DRDY 13 |
| Reset 21 | Sync/Reset 11 |

#### Clock module
| Pico | ADS131M04 | SIT2024 |
| ------------- | ------------- | ------------- |
| 3.3 V (Out) | - | VDD 4 |
| GND | - | GND 1 |
| 3.3 V (Out) | - | OE 3 |
| - | CLKIN 17 | OUT 5 |

> Capacitors are placed between positive and negative rails because pico's switched-mode voltage regulator is known to be noisy (and so is the internal ADC). 

## Function list
### Common functions
1. Initializing ADC [Modified]
```C
void ADS131M04::begin(uint8_t clk_pin, uint8_t miso_pin, uint8_t mosi_pin, uint8_t cs_pin, uint8_t drdy_pin, uint8_t reset_pin)
```

2. Set sampling rate (default 2K SPS, max 32 SPS) [Etchezuri]
```C
bool ADS131M04::setOsr(uint16_t osr)
```

3. Channel gain (default 1, max 128) [Etchezuri]
```C
bool ADS131M04::setChannelPGA(uint8_t channel, uint16_t pga)
```

4. Input MUX selection (default differential pairs) [Etchezuri]
```C
bool ADS131M04::setInputChannelSelection(uint8_t channel, uint8_t input)
```

5. Offset calibration per channel (see datasheet) [Etchezuri]
```C
bool ADS131M04::setChannelOffsetCalibration(uint8_t channel, int32_t offset)
```

6. Gain calibration per channel (see datasheet) [Etchezuri]
```C
bool ADS131M04::setChannelGainCalibration(uint8_t channel, uint32_t gain)
```

7. Non-interrupt read of data ready (see testPanel for implementing an interrupt-based method) [Etchezuri]
```C
bool ADS131M04::isDataReady()
```

8. Reading adc (see testPanel for data formats) [Modified]
```C
adcOutput ADS131M04::readADC(void)
```

9. A fast two's complement implementation [New]
```C
int32_t ADS131M04::twoscom(int32_t datain)
```

10. Converting int32 data to floating voltage (assuming gain = 1) [New]
```C
float ADS131M04::convert(int32_t datain)
```

11. Issue a command to ADS131M04 [New]
```C
bool ADS131M04::command(uint16_t cmd)
```

12. Read register (16 bits per register address) [Etchezuri]
```C
uint16_t ADS131M04::readRegister(uint8_t address)
```

13. Reset ADC (lose all regsiter writes) [New]
```C
void ADS131M04::reset(void)
```

### Uncommon functions
14. Write ADC register (6 word per frame and default 24 bits per word) [Etchezuri]
```C
uint8_t ADS131M04::writeRegister(uint8_t address, uint16_t value)
```

15. A wrapper function for register writing [Etchezuri]
```C
void ADS131M04::writeRegisterMasked(uint8_t address, uint16_t value, uint16_t mask)
```

16. Get data ready from STATUS register [Etchezuri]
```C
int8_t ADS131M04::isDataReadySoft(byte channel)
```

17. Check reset status [Etchezuri]
```C
bool ADS131M04::isResetStatus(void)
```

18. Check SPI lock (see datasheet) [Etchezuri]
```C
bool ADS131M04::isLockSPI(void)
```
      
19. Set data ready format (see datasheet) [Etchezuri]
```C
bool ADS131M04::setDrdyFormat(uint8_t drdyFormat)
bool ADS131M04::setDrdyStateWhenUnavailable(uint8_t drdyState)
```

20. Set power mode (see datasheet for corresponding sampling and clock rates) [Etchezuri]
```C
bool ADS131M04::setPowerMode(uint8_t powerMode)
```

21. Enable/disble channels [Etchezuri]
```C
bool ADS131M04::setChannelEnable(uint8_t channel, uint16_t enable)
```

22. Global chops (see datasheet) [Etchezuri]
```C
void ADS131M04::setGlobalChop(uint16_t global_chop)
```

## Changes
0. Made everything work with raspberry pi pico (RP2040)
1. Added interrupt-based DRDY-pin reading
2. Added reset functions through writing Reset Pin and SPI bus
3. Added a function to assert command over SPI
4. Added a slightly faster two's complement function
5. Added a function to convert ADC reading to voltage (max +/- 1.2V)
6. Added a test panel example
7. Modified ADS131M04::begin function to add a reset step and more pin assertions. This function can be further modified to suppor other boards.

## Reference

The original library: https://github.com/LucasEtchezuri/Arduino-ADS131M04

Datasheet: https://www.ti.com/lit/ds/symlink/ads131m04.pdf

Clock datasheet: https://www.sitime.com/datasheet/SiT2024

ADC in test: https://www.digikey.com/en/products/detail/texas-instruments/ADS131M04IPWR/10448283

Clock in test: https://www.digikey.com/en/products/detail/sitime/SIT2024BM-S3-33N-8-192000/11289853
