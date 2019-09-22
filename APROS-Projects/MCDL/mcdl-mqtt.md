# APROS MCDL(multi-channels data logger) MQTT Message Type

## Raw Data (MQTT Message)

- Transmission rate : 1 packet / second
- 9,611,200 bytes per second
- Data Format (1 packet)

```json
{
  "messageType":"Telemetry",
  "deviceName":"5GISMIANAMR0001",
  "timestamp":1564724476504,
  "frequency":10000,
  "channel1_shaft_x": [-0.19480411822979063,-0.17015989010150412,-0.1784128409165582,-0.14679945432222602,...9996 items],
  "channel2_shaft_x": [-1.6150223291837231,-1.666259398827184,-1.584623960348274,-1.607204950772797,...9996 items],
  "channel3_shaft_x": [-65.52844102566058,-65.60803615129912,-65.57855477699867,-65.62754521003136,...9996 items],
  "channel4_shaft_x": [-0.37208208670982745,-0.34282996104313573,-0.38492001019991157,-0.3772172561058611, ...9996 items]
}
```

## RMS Data (MQTT Message)

- Transmission rate : 1 packet / second
- 276 bytes per second
- Data Format (1 packet)

```json
{
  "messageType":"Telemetry",
  "deviceName":"5GISMIANAMR0001",
  "timestamp":1564723759570,
  "frequency":12800,
  "channel1_shaft_x_rms":0.0205518354785063,
  "channel2_shaft_x_rms":0.04406875489806863,
  "channel3_shaft_x_rms":0.05108736453245308,
  "channel4_shaft_x_rms":0.042653114466254125
}
```
