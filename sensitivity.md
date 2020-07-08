# Change sensivity on a sensor
This is the code that I use to change the sensivity of a sensor via terminal using homebridge-hue and in this case, an Aqara vibration sensor.

`curl -H 'Content-Type: application/json' -X PUT -d '{"sensitivity": 2}' http://192.168.7.240:8080/api/394B3BBAD1/sensors/9/config`
