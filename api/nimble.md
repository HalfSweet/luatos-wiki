# nimble - 蓝牙BLE库(nimble版)

{bdg-success}`已适配` {bdg-primary}`Air101/Air103` {bdg-primary}`ESP32C3` {bdg-primary}`ESP32S3`

```{note}
本页文档由[这个文件](https://gitee.com/openLuat/LuatOS/tree/master/luat/../components/nimble/src/luat_lib_nimble.c)自动生成。如有错误，请提交issue或帮忙修改后pr，谢谢！
```

```{tip}
本库有专属demo，[点此链接查看nimble的demo例子](https://gitee.com/openLuat/LuatOS/tree/master/demo/nimble)
```

**示例**

```lua
-- 本库当前支持Air101/Air103/ESP32/ESP32C3/ESP32S3
-- 用法请查阅demo, API函数会归于指定的模式

```

## nimble.init(name)



初始化BLE上下文,开始对外广播/扫描

**参数**

|传入值类型|解释|
|-|-|
|string|蓝牙设备名称,可选,建议填写|

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功与否|

**例子**

```lua
-- 参考 demo/nimble
-- 本函数对所有模式都适用

```

---

## nimble.deinit()



关闭BLE上下文

**参数**

无

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功与否|

**例子**

```lua
-- 仅部分设备支持,当前可能都不支持
-- 本函数对所有模式都适用

```

---

## nimble.send_msg(conn, handle, data)



发送信息

**参数**

|传入值类型|解释|
|-|-|
|int|连接id, 当前固定填1|
|int|处理id, 当前固定填0|
|string|数据字符串,可包含不可见字符|

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功与否|

**例子**

```lua
-- 参考 demo/nimble
-- 本函数对peripheral/从机模式适用

```

---

## nimble.scan()



扫描从机

**参数**

无

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功与否|

**例子**

```lua
-- 参考 demo/nimble
-- 本函数对central/主机模式适用
-- 本函数会直接返回, 然后通过异步回调返回结果

```

---

## nimble.mode(tp)



设置模式

**参数**

|传入值类型|解释|
|-|-|
|int|模式, 默认server/peripheral, 可选 client/central模式 nimble.MODE_BLE_CLIENT|

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功与否|

**例子**

```lua
-- 参考 demo/nimble
-- 必须在nimble.init()之前调用
-- nimble.mode(nimble.MODE_BLE_CLIENT) -- 简称从机模式,未完善

```

---

## nimble.setUUID(tp, addr)



设置server/peripheral的UUID

**参数**

|传入值类型|解释|
|-|-|
|string|配置字符串,后面的示例有说明|
|string|地址字符串|

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功与否|

**例子**

```lua
-- 参考 demo/nimble, 2023-02-25之后编译的固件支持本API
-- 必须在nimble.init()之前调用
-- 本函数对peripheral/从机模式适用

-- 设置SERVER/Peripheral模式下的UUID, 支持设置3个
-- 地址支持 2/4/16字节, 需要二进制数据
-- 2字节地址示例: AABB, 写 string.fromHex("AABB") ,或者 string.char(0xAA, 0xBB)
-- 4字节地址示例: AABBCCDD , 写 string.fromHex("AABBCCDD") ,或者 string.char(0xAA, 0xBB, 0xCC, 0xDD)
nimble.setUUID("srv", string.fromHex("380D"))      -- 服务主UUID         ,  默认值 180D
nimble.setUUID("write", string.fromHex("FF31"))    -- 往本设备写数据的UUID,  默认值 FFF1
nimble.setUUID("indicate", string.fromHex("FF32")) -- 订阅本设备的数据的UUID,默认值 FFF2

```

---

## nimble.mac()



获取蓝牙MAC

**参数**

无

**返回值**

|返回值类型|解释|
|-|-|
|string|蓝牙MAC地址,6字节|

**例子**

```lua
-- 参考 demo/nimble, 2023-02-25之后编译的固件支持本API
-- 本函数对所有模式都适用
local mac = nimble.mac()
log.info("ble", "mac", mac and mac:toHex() or "Unknwn")

```

---

## nimble.ibeacon(data, major, minor, measured_power)



配置iBeacon的参数,仅iBeacon模式可用

**参数**

|传入值类型|解释|
|-|-|
|string|数据, 必须是16字节|
|int|主版本号,默认2, 可选, 范围 0 ~ 65536|
|int|次版本号,默认10,可选, 范围 0 ~ 65536|
|int|名义功率, 默认0, 范围 -126 到 20 |

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功返回true,否则返回false|

**例子**

```lua
-- 参考 demo/nimble, 2023-02-25之后编译的固件支持本API
-- 本函数对ibeacon模式适用
nimble.ibeacon(data, 2, 10, 0)
nimble.init()

```

---

## nimble.advData(data, flags)



配置广播数据,仅iBeacon模式可用

**参数**

|传入值类型|解释|
|-|-|
|string|广播数据, 当前最高128字节|
|int|广播标识, 可选, 默认值是 0x06,即 不支持传统蓝牙(0x04) + 普通发现模式(0x02)|

**返回值**

|返回值类型|解释|
|-|-|
|bool|成功返回true,否则返回false|

**例子**

```lua
-- 参考 demo/nimble/adv_free, 2023-03-18之后编译的固件支持本API
-- 本函数对ibeacon模式适用
-- 数据来源可以多种多样
local data = string.fromHex("123487651234876512348765123487651234876512348765")
-- local data = crypto.trng(25)
-- local data = string.char(0x11, 0x13, 0xA3, 0x5A, 0x11, 0x13, 0xA3, 0x5A, 0x11, 0x13, 0xA3, 0x5A, 0x11, 0x13, 0xA3, 0x5A)
nimble.advData(data)
nimble.init()

-- nimble支持在init之后的任意时刻再次调用, 以实现数据更新
-- 例如 1分钟变一次
while 1 do
    sys.wait(60000)
    local data = crypto.trng(25)
    nimble.advData(data)
end

```

---

## nimble.advParams(conn_mode, disc_mode, itvl_min, itvl_max, channel_map, filter_policy, high_duty_cycle)



设置广播参数

**参数**

|传入值类型|解释|
|-|-|
|int|广播模式, 0 - 不可连接, 1 - 定向连接, 2 - 未定向连接, 默认0|
|int|发现模式, 0 - 不可发现, 1 - 限制发现, 3 - 通用发现, 默认0|
|int|最小广播间隔, 0 - 使用默认值, 范围 1 - 65535, 单位0.625ms, 默认0|
|int|最大广播间隔, 0 - 使用默认值, 范围 1 - 65535, 单位0.625ms, 默认0|
|int|广播通道, 默认0, 一般不需要设置|
|int|过滤规则, 默认0, 一般不需要设置|
|int|当广播模式为"定向连接"时,是否使用高占空比模式, 默认0, 可选1|

**返回值**

|返回值类型|解释|
|-|-|
|nil|无返回值|

**例子**

```lua
-- 当前仅ibeacon模式/peripheral/从机可使用
-- 例如设置 不可连接 + 限制发现
-- 需要在nimble.init之前设置好
nimble.advParams(0, 1)
-- 注意peripheral模式下自动配置 conn_mode 和 disc_mode

```

---

