> This is a code study of the ODlib.java, although the following code is a study of odroidc2.c which makes contact with ODlib.java

## Defines and imports

```c
#include <jni.h>
#include <stdio.h>
#include <stdlib.h>
#include <android/log.h>
#include <time.h>
#include <fcntl.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
#include <errno.h>
#include <sys/ioctl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/wait.h>
#include <linux/input.h>
#include <linux/spi/spidev.h>
  
#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)
#define  LOGW(...)  __android_log_print(ANDROID_LOG_WARN,LOG_TAG,__VA_ARGS__)
#define  LOGE(...)  __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
#define LOG_TAG "ODROIDC2"

static volatile unsigned long   *gpio;
static volatile unsigned long   *gpio_ao;
static volatile unsigned long   *ao_in;

#define BLOCK_SIZE  (4*1024)
#define GPIO_REG_MAP        0xC8834000
#define GPIO_AO_REG_MAP     0xC8100000
#define GPIOAO_OFFSET       122
#define GPIOAO_PIN_START    (GPIOAO_OFFSET + 0)
#define GPIOAO_PIN_END      (GPIOAO_OFFSET + 13)
#define GPIOAO_FSEL_REG_OFFSET  0x09
#define GPIOAO_OUTP_REG_OFFSET  0x09
#define GPIOAO_INP_REG_OFFSET   0x0A
#define GPIOAO_PUPD_REG_OFFSET  0x0B
#define GPIOAO_PUEN_REG_OFFSET  0x0B
#define GPIO_BANKS_OFFSET   136
#define GPIODV_PIN_START    (GPIO_BANKS_OFFSET + 45)
#define GPIODV_PIN_END      (GPIO_BANKS_OFFSET + 74)
#define GPIODV_FSEL_REG_OFFSET  0x10C
#define GPIODV_OUTP_REG_OFFSET  0x10D
#define GPIODV_INP_REG_OFFSET   0x10E
#define GPIODV_PUPD_REG_OFFSET  0x13A
#define GPIODV_PUEN_REG_OFFSET  0x148
#define GPIOY_PIN_START     (GPIO_BANKS_OFFSET + 75)
#define GPIOY_PIN_END       (GPIO_BANKS_OFFSET + 91)
#define GPIOY_FSEL_REG_OFFSET   0x10F
#define GPIOY_OUTP_REG_OFFSET   0x110
#define GPIOY_INP_REG_OFFSET    0x111
#define GPIOY_PUPD_REG_OFFSET   0x13B
#define GPIOY_PUEN_REG_OFFSET   0x149
#define GPIOX_PIN_START     (GPIO_BANKS_OFFSET + 92)
#define GPIOX_PIN_END       (GPIO_BANKS_OFFSET + 114)
#define GPIOX_FSEL_REG_OFFSET   0x118
#define GPIOX_OUTP_REG_OFFSET   0x119
#define GPIOX_INP_REG_OFFSET    0x11A
#define GPIOX_PUPD_REG_OFFSET   0x13E
#define GPIOX_PUEN_REG_OFFSET   0x14C
#define AO_MUX_REG1     0x005
#define AO_MUX_REG2     0x006
#define BIT(x)          (1 << x)
```

## Sets & gets

```c
void set_mode (int port, int mode) {
  
    if(port >= GPIOX_PIN_START && port <= GPIOX_PIN_END) {
        if(mode > 0)    *(gpio + GPIOX_FSEL_REG_OFFSET) |= BIT(port - GPIOX_PIN_START);
        else            *(gpio + GPIOX_FSEL_REG_OFFSET) &= ~BIT(port - GPIOX_PIN_START);
    }
    if(port >= GPIOY_PIN_START && port <= GPIOY_PIN_END) {
        if(mode > 0)    *(gpio + GPIOY_FSEL_REG_OFFSET) |= BIT(port - GPIOY_PIN_START);
        else            *(gpio + GPIOY_FSEL_REG_OFFSET) &= ~BIT(port - GPIOY_PIN_START);
    }
    if(port >= GPIODV_PIN_START && port <= GPIODV_PIN_END) {
        if(mode > 0)    *(gpio + GPIODV_FSEL_REG_OFFSET) |= BIT(port - GPIODV_PIN_START);
        else            *(gpio + GPIODV_FSEL_REG_OFFSET) &= ~BIT(port - GPIODV_PIN_START);
    }
    if(port >= GPIOAO_PIN_START && port <= GPIOAO_PIN_END) {
        if(mode > 0)    *(gpio + GPIOAO_FSEL_REG_OFFSET) |= BIT(port - GPIOAO_PIN_START);
        else            *(gpio + GPIOAO_FSEL_REG_OFFSET) &= ~BIT(port - GPIOAO_PIN_START);
    }
}

void set_pupd(int port, int mode) {

    if(port >= GPIOX_PIN_START && port <= GPIOX_PIN_END) {
        if(mode > 0) {
            *(gpio + GPIOX_PUEN_REG_OFFSET) |= BIT(port - GPIOX_PIN_START);
            if(mode == 2)   *(gpio + GPIOX_PUPD_REG_OFFSET) |= BIT(port - GPIOX_PIN_START);
            else            *(gpio + GPIOX_PUPD_REG_OFFSET) &= ~BIT(port - GPIOX_PIN_START);
        }
        else *(gpio + GPIOX_PUEN_REG_OFFSET) &= ~BIT(port - GPIOX_PIN_START);
    }
    if(port >= GPIOY_PIN_START && port <= GPIOY_PIN_END) {
        if(mode > 0) {
            *(gpio + GPIOY_PUEN_REG_OFFSET) |= BIT(port - GPIOY_PIN_START);
            if(mode == 2)   *(gpio + GPIOY_PUPD_REG_OFFSET) |= BIT(port - GPIOY_PIN_START);
            else            *(gpio + GPIOY_PUPD_REG_OFFSET) &= ~BIT(port - GPIOY_PIN_START);
        }
        else *(gpio + GPIOY_PUEN_REG_OFFSET) &= ~BIT(port - GPIOY_PIN_START);
    }
    if(port >= GPIODV_PIN_START && port <= GPIODV_PIN_END) {
        if(mode > 0) {
            *(gpio + GPIODV_PUEN_REG_OFFSET) |= BIT(port - GPIODV_PIN_START);
            if(mode == 2)   *(gpio + GPIODV_PUPD_REG_OFFSET) |= BIT(port - GPIODV_PIN_START);
            else            *(gpio + GPIODV_PUPD_REG_OFFSET) &= ~BIT(port - GPIODV_PIN_START);
        }
        else *(gpio + GPIODV_PUEN_REG_OFFSET) &= ~BIT(port - GPIODV_PIN_START);
    }
    if(port >= GPIOAO_PIN_START && port <= GPIOAO_PIN_END) {
        if(mode > 0) {

            *(gpio + GPIOAO_PUEN_REG_OFFSET) |= BIT(port - GPIOAO_PIN_START);

            if(mode == 2)   *(gpio + GPIOAO_PUPD_REG_OFFSET) |= BIT(port - GPIOAO_PIN_START);
            else            *(gpio + GPIOAO_PUPD_REG_OFFSET) &= ~BIT(port - GPIOAO_PIN_START);
        }
        else *(gpio + GPIOAO_PUEN_REG_OFFSET) &= ~BIT(port - GPIOAO_PIN_START);
    }
} 

int get_status(int port) {

    if(port >= GPIOX_PIN_START && port <= GPIOX_PIN_END) {
        return  *(gpio + GPIOX_INP_REG_OFFSET) & BIT(port - GPIOX_PIN_START) ? 1 : 0;
    }
    if(port >= GPIOY_PIN_START && port <= GPIOY_PIN_END) {
        return  *(gpio + GPIOY_INP_REG_OFFSET) & BIT(port - GPIOY_PIN_START) ? 1 : 0;
    }
    if(port >= GPIODV_PIN_START && port <= GPIODV_PIN_END) {
        return  *(gpio + GPIODV_INP_REG_OFFSET) & BIT(port - GPIODV_PIN_START) ? 1 : 0;
    }
    if(port >= GPIOAO_PIN_START && port <= GPIOAO_PIN_END) {
        return  *(gpio_ao + GPIOAO_INP_REG_OFFSET) & BIT(port - GPIOAO_PIN_START) ? 1 : 0;
    }
    return  0;
}

void set_status(int port, int state) {

    if(port >= GPIOX_PIN_START && port <= GPIOX_PIN_END) {
        if(state == 1)  *(gpio + GPIOX_OUTP_REG_OFFSET) |= BIT(port - GPIOX_PIN_START);
        else            *(gpio + GPIOX_OUTP_REG_OFFSET) &= ~BIT(port - GPIOX_PIN_START);
    }
    if(port >= GPIOY_PIN_START && port <= GPIOY_PIN_END) {
        if(state == 1)  *(gpio + GPIOY_OUTP_REG_OFFSET) |= BIT(port - GPIOY_PIN_START);
        else            *(gpio + GPIOY_OUTP_REG_OFFSET) &= ~BIT(port - GPIOY_PIN_START);
    }
    if(port >= GPIODV_PIN_START && port <= GPIODV_PIN_END) {
        if(state == 1)  *(gpio + GPIODV_OUTP_REG_OFFSET) |= BIT(port - GPIODV_PIN_START);
        else            *(gpio + GPIODV_OUTP_REG_OFFSET) &= ~BIT(port - GPIODV_PIN_START);
    }
    if(port >= GPIOAO_PIN_START && port <= GPIOAO_PIN_END) {
        if(state == 1)  *(gpio + GPIOAO_OUTP_REG_OFFSET) |= BIT(port - GPIOAO_PIN_START);
        else            *(gpio + GPIOAO_OUTP_REG_OFFSET) &= ~BIT(port - GPIOAO_PIN_START);
    }
}
```

These functions seem to be related to the port being manipulated, asking both values in order for it to work... the `port` variable being self-explanatory and the variables `mode` and `state` being used respective for being for the input and activation ("out" and "in")

these functions contain a check for their activation in case they are higher than 0
```c
if (mode > 0) return "example";
```

the `get_status()` seem to give us an output of either 'out' or 'in' which determines weather they are both **writable and readable** or just **readable**.

---


```c
int mem_init(void) {

    int	fd;
    if ((fd = open ("/dev/mem", O_RDWR | O_SYNC | O_CLOEXEC) ) < 0) {
        LOGI("/dev/mem open error!\n");	fflush(stdout);
        return -1;
    }

    gpio =    (unsigned long *)mmap64(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_REG_MAP);
    gpio_ao = (unsigned long *)mmap64(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_AO_REG_MAP);

    if((unsigned long)gpio == -1 || (unsigned long)gpio_ao == -1) {
        LOGI("mmap error!\n");
        return	-1;
    }
    return	0;
}
```

> This function shouldn't be touched.

## Pulse Width Modulation

```c
static int PWMPortCreated[500];
static int PWMPortFreq[500];
static int PWMPulseWidth[500];
static pthread_t PWMthreads[500];

#define PWM_LOOP_THREAD(X) void *X (void *dummy)

static int SERVOPortCreated[500];
static int SERVOPortFreq[500];
static int SERVOPulseWidth[500];
static pthread_t SERVOthreads[500];

#define SERVO_LOOP_THREAD(X) void *X (void *dummy)


static int INTERRUPTPortCreated[500];
static int INTERRUPTMode[500];
static pthread_t INTERRUPTthreads[500];

#define INTERRUPT_LOOP_THREAD(X) void *X (void *dummy)



static int analogCreated = 0;

int init_port;
int interrupt_port;

int fdI2C[3]			= {0,-1,-1};  	// We need only array pos: 1 and 2 (I2C Port 1 and 2)
int8_t i2cAddrOLD[3] 	= {0,-1,-1};	// We need only array pos: 1 and 2 (I2C Port 1 and 2)



static int realkeyboardfd 			= -1;
static int realmousefd 				= -1;
pthread_t realMouseEventThread;
pthread_t realKeyboardEventThread;
pthread_t wheelThread;


int memInit = 0;
int spiBitBangInit = 0;

int destroying = 0;
```

These variables seem to be related to [[Pulse Width Modulation|PWM]].

# ODlib.java


