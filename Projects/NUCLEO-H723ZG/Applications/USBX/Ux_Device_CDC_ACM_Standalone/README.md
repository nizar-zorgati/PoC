
## <b>Ux_Device_CDC_ACM_Standalone application description </b>

This application provides an example of Azure RTOS USBX stack usage on STM32H723ZG board, it shows how to develop USB Device communication Class "CDC_ACM" based application.
The application is designed to emulate an USB-to-UART bridge following the Virtual COM Port (VCP) implementation, the code provides all required device descriptors framework
and associated Class descriptor report to build a compliant USB CDC_ACM device.

  - usbx_app_thread_entry: used to initialize USB OTG HAL PCD driver and start the device.
  - CDC_ACM_Read_Task: used to Read the received data from Virtual COM Port.
  - CDC_ACM_Write_Task: used to send the received data over UART.

During enumeration phase, three communication pipes "endpoints" are declared in the CDC class implementation :

 - 1 x Bulk IN endpoint for receiving data from STM32 device to PC host:
   When data are received over UART they are saved in the buffer "UserTxBufferFS". Periodically, in a
   usbx_cdc_acm_write_thread_entry the state of the buffer "UserTxBufferFS" is checked. If there are available data, they
   are transmitted in response to IN token otherwise it is NAKed.

 - 1 x Bulk OUT endpoint for transmitting data from PC host to STM32 device:
   When data are received through this endpoint they are saved in the buffer "UserRxBufferFS" then they are transmitted
   over UART using DMA mode and in meanwhile the OUT endpoint is NAKed.
   Once the transmission is over, the OUT endpoint is prepared to receive next packet in HAL_UART_RxCpltCallback().
 - 1 x Interrupt IN endpoint for setting and getting serial-port parameters:
   When control setup is received, the corresponding request is executed in ux_app_parameters_change().

In this application, two requests are implemented:

    - Set line: Set the bit rate, number of Stop bits, parity, and number of data bits
    - Get line: Get the bit rate, number of Stop bits, parity, and number of data bits
   The other requests (send break, control line state) are not implemented.

<b>Notes</b>

- Receiving data over UART is handled by interrupt while transmitting is handled by DMA allowing hence the application to receive
data at the same time it is transmitting another data (full- duplex feature).

- The user has to check the list of the COM ports in Device Manager to find out the COM port number that have been assigned (by OS) to the VCP interface.

#### <b>Expected success behavior</b>

When plugged to PC host, the STM32H723ZG must be properly enumerated as an USB Serial device and an STlink Com port.
During the enumeration phase, the device must provide host with the requested descriptors (Device descriptor, configuration descriptor, string descriptors).
Those descriptors are used by host driver to identify the device capabilities. Once STM32H723ZG USB device successfully completed the enumeration phase,
Open two hyperterminals (USB com port and UART com port(USB STLink VCP)) to send/receive data to/from host from/to device.

#### <b>Error behaviors</b>

Host PC shows that USB device does not operate as designed (CDC Device enumeration failed, PC and Device can not communicate over VCP ports).

#### <b>Assumptions if any</b>

User is familiar with USB 2.0 "Universal Serial BUS" Specification and CDC_ACM class Specification.

#### <b> Known limitations</b>


### <b>Notes</b>

 1. Some code parts can be executed in the ITCM-RAM (64 KB up to 256kB) which decreases critical task execution time, compared to code execution from Flash memory. This feature can be activated using '#pragma location = ".itcmram"' to be placed above function declaration, or using the toolchain GUI (file options) to execute a whole source file in the ITCM-RAM.
 2.  If the application is using the DTCM/ITCM memories (@0x20000000/ 0x0000000: not cacheable and only accessible by the Cortex M7 and the MDMA), no need for cache maintenance when the Cortex M7 and the MDMA access these RAMs. If the application needs to use DMA (or other masters) based access or requires more RAM, then the user has to:
      - Use a non TCM SRAM. (example : D1 AXI-SRAM @ 0x24000000).
      - Add a cache maintenance mechanism to ensure the cache coherence between CPU and other masters (DMAs,DMA2D,LTDC,MDMA).
      - The addresses and the size of cacheable buffers (shared between CPU and other masters) must be properly defined to be aligned to L1-CACHE line size (32 bytes).
 3.  It is recommended to enable the cache and maintain its coherence:
      - Depending on the use case it is also possible to configure the cache attributes using the MPU.
      - Please refer to the **AN4838** "Managing memory protection unit (MPU) in STM32 MCUs".
      - Please refer to the **AN4839** "Level 1 cache on STM32F7 Series"


#### <b>USBX usage hints</b>

- The DTCM (0x20000000) memory region should not be used by application in case USB DMA is enabled
- Should make sure to configure the USB pool memory region with attribute "Non-Cacheable" to ensure coherency between CPU and USB DMA

### <b>Keywords</b>

RTOS, ThreadX, USBX, Device, USB_OTG, Full Speed, CDC, VCP, USART, DMA.

### <b>Hardware and Software environment</b>

  - This example runs on STM32H723xx devices
  - This example has been tested with STMicroelectronics STM32H723ZG boards Revision MB1364-H723ZG-E01 and can be easily tailored to any other supported device and development board.
  - STM32H723ZG Set-up
  - Connect the STM32H723ZG board CN13 to the PC through "MICRO-USB" to "Standard A" cable.
  - For VCP the configuration is dynamic for example it can be :
    - BaudRate = 115200 baud
    - Word Length = 8 Bits
    - Stop Bit = 1
    - Parity = None
    - Flow control = None

  - The USART3 interface available on PD8 and PD9 of the microcontroller are
  connected to ST-LINK MCU.
  By default the USART3 communication between the target MCU and ST-LINK MCU is enabled.
  It's configuration is as following:
    - BaudRate = 115200 baud
    - Word Length = 8 Bits
    - Stop Bit = 1
    - Parity = None
    - Flow control = None

<b>Note</b>

 - In case User configures USB VCP baudrate under 9600 the USART3 baudrate shall be set to 9600.

### <b>How to use it ?</b>

In order to make the program work, you must do the following :

 - Open your preferred toolchain
 - Rebuild all files and load your image into target memory
 - Run the application