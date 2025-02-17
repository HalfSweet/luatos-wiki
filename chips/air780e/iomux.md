# LuatOS固件下的IO复用配置

**本文档描述的是LuatOS视角**

1. 由于固件特性的存在, LuatOS的io复用是固定的.
2. 不同模块的外部管脚布局不一样, 但`PAD(paddr)`值是一致的, 要对应 "PIN/GPIO对应表格" 文档, 该文档可以在 https://air780e.cn 找到
3. 由于芯片的引脚少,存在大量复用的场景,有很多功能是会冲突的
4. 对于AT固件来说,本文档没有意义, 请无视
5. 对于CSDK来说, 相关复用都是可以修改的, 所以也请无视本文档
6. Air600E注定不适合二次开发,有些管脚在硬件设计手册里的描述会有差异,注意区分

## PWM说明

实际可用通道就4个(0/1/2/4), 但每个都有2种配置, PWM3/PWM5已经被底层使用.

例如 PWM1和PWM11都使用硬件通道1, **只能选其中一个使用**.

启用PWM1就不能启用PWM11, 调用pwm库的API时,填 `软件通道id`

|软件通道id|实际硬件通道|对应的GPIO|对应的PAD|备注|
|----------|------------|---------|---------|----|
|0         |    0       | GPIO23  |    43   | AGPIO3,驱动能力弱. **应避免使用本管脚**|
|1         |    1       | GPIO24  |    44   | MAIN_RI,实际为AGPIO4,驱动能力弱 |
|2         |    2       | GPIO25  |    45   | AGPIO5,驱动能力弱|
|4         |    4       | GPIO27  |    47   | NetLed,网络状态灯 |
|10        |    0       | GPIO1   |    16   | LCD_RST,实际为普通GPIO|
|11        |    1       | GPIO2   |    17   | MAIN_DCD,实际为普通GPIO |
|12        |    2       | GPIO16  |    31   | MAIN_CTS,实际为普通GPIO |
|14        |    4       | GPIO19  |    34   | UART1_TXD/MAIN_TXD |
|20        |    0       | 无      |    39   | I2S_MCLK|
|21        |    1       | GPI29   |    35   | I2S_LRCK |
|22        |    2       | GPI30   |    36   | I2S_LRCK |
|24        |    4       | GPI31   |    37   | I2S_DIN |

PS: 
1. 软件通道10/11/12/14需要V1002以上的固件, 20221219之后编译的版本
1. 软件通道20/21/22/24需要V1016以上的固件, 20230330之后编译的版本

## UART说明

物理uart有3个(0/1/2)
1. uart0是日志口(DBG_TX/DBG_RX),不推荐使用,启动时也有输出,LuatOS固件默认禁用uart0
2. uart1是主串口(MAIN_TX/MAIN_RX), 推荐使用
3. uart2是次串口(AUX_TX/AUX_RX), **带GNSS功能的模块会接GNSS芯片**,而且PAD不同,不可用作其他功能
4. 注意, UART2在Air780E与Air780EG用的PAD是不一样的,但软件会自动适配,不需要关注.

|功能    |软件含义  |对应的GPIO|对应的PAD|备注|
|--------|----------|---------|---------|----|
|DBG_RX  | UART0_RX | -       |    29   ||
|DBG_TX  | UART0_TX | -       |    30   ||
|MAIN_RX | UART1_RX | GPIO18  |    33   ||
|MAIN_TX | UART1_TX | GPIO19  |    34   ||
|AUX_RX  | UART2_RX | GPIO10  |    25   |Air780EG在PAD 27|
|AUX_TX  | UART2_TX | GPIO11  |    26   |Air780EG在PAD 28|

## I2C说明

物理i2c有2个(0/1)

|功能     |软件含义  |对应的GPIO|对应的PAD|备注|
|---------|---------|---------|---------|----|
|I2C0_SCL | I2C0时钟 | GPIO14  |    13   |GPIO功能看后面的说明|
|I2C0_SDA | I2C0数据 | GPIO15  |    14   |GPIO功能看后面的说明|
|I2C1_SCL | I2C1时钟 | GPIO9   |    24   |与SPI0冲突|
|I2C1_SDA | I2C1数据 | GPIO8   |    23   |与SPI0冲突|

## SPI说明

物理SPI有2个(0/1)

|功能     |软件含义     |对应的GPIO|对应的PAD|备注|
|---------|------------|---------|---------|----|
|SPI0_CS  | SPI0片选    | GPIO8   |    23   |与I2C1冲突|
|SPI0_MOSI| SPI0主机输出| GPIO9   |    24   |与I2C1冲突|
|SPI0_MISO| SPI0从机输出| GPIO10  |    25   ||
|SPI0_SCL | SPI0时钟    | GPIO11  |    26   ||
|SPI1_CS  | SPI1片选    | GPIO12  |    27   ||
|SPI1_MOSI| SPI1主机输出| GPIO13  |    28   ||
|SPI1_MISO| SPI1从机输出| -       |    29   |注意无GPIO功能|
|SPI1_SCL | SPI1时钟    | -       |    30   |注意无GPIO功能|

注意:
1. SPI0与UART2/I2C1是冲突的, 事实如此
2. SPI1的MISO和SCL虽然可复用为GPIO14/15,但这些GPIO实际映射到其他脚的,看`GPIO额外说明`

## GPIO额外说明

1. GPIO14/15在V1103有变动, 已正确映射到 `PAD 13/14`
2. 普通GPIO在深睡眠/SLEEP2, 会有周期性高电平脉冲, 务必注意
3. AONGPIO是休眠时仍可维持高电平的GPIO,但驱动能力很弱
4. GPIO12/GPIO13 有两种映射, 通过不同的API使用
5. 普通GPIO在配置成输入/中断模式时，上下拉无法设置，如果默认上下拉不能满足要求，可以设置成LUAT_GPIO_DEFAULT来取消默认上下拉，然后外部加上下拉
6. GPIO20,21,22配置成中断模式时，是wakeup功能，可以配置上下拉，也可以取消使用外部上下拉
7. **GPIO23** 上电后首先是输入+下拉,然后会设置成 **输出+上拉+高电平**, 建议避免使用该GPIO

|对应的GPIO|对应的PAD|使用的API示例|备注|
|---------|---------|---------|----|
| GPIO12   |    11   |pm.power(pm.CAMERA, true 或者 false)|LDO_CTL,在Air600E标的是GPIO12|
| GPIO13   |    12   |pm.power(pm.GPS, true 或者 false)|没有引出,在Air780EG控制GPS的电源|
| GPIO12   |    27   |gpio.setup(12, 0)|I2C0_SDA,也是复用的|
| GPIO13   |    28   |gpio.setup(13, 0)|I2C0_SCL,也是复用的|


## 虚拟GPIO

Air780E(EC618全系)支持多个虚拟的GPIO, 将非GPIO管脚通过软件模拟成GPIO来使用

|编号|名称|功能|备注|
|----|----|----|---|
|32| wakeup0|仅支持输入和中断| wakeup0休眠唤醒脚|
|33| vbus/wakeup1|仅支持输入和中断| USB的VBUS, 检测USB是否是插入状态|
|34| wakeup2|仅支持输入和中断| wakeup2休眠唤醒脚, USIM_DET|
|35| pwrkey |仅支持输入和中断| 即开机键, 开机之后当普通GPIO使用|

vbus的说明: 
1. 在CSDK/LuatOS固件中, vbus与USB功能是解耦的
2. 与常规认识不同, 在不接vbus的情况下, USB功能依然可用
3. 在进入休眠前, 将上述`wakeup0/wakeup1/wakeup2`设置成中断状态, 即可实现管脚唤醒功能

例如将`wakup0`设置为唤醒脚, 中断回调可以是空函数
```lua
gpio.setup(32, function() end, gpio.PULLUP)
```