### Items:

```
Switch boiler "Boiler [%s]" //{ http=">[ON:POST:http://192.168.0.14/servo?value=1] >[OFF:POST:http://192.168.0.14/servo?value=0]" }

// heating setpoints
Number      Heating_Setpoint "Setpoint [%.1f °C]"    <degreesc>

// heating modes: 0 - Off, 1 - On, 2 - Auto
Number      Heating_Mode     "Heating Mode"          <heating>
```

### Rules: 

```
rule "automatic heating"
when
    Item Heating_Mode changed or
    Item Heating_Setpoint changed or
    Item Second_Temp changed 
then
    // 0="Off", 1="On", 2="Auto"
    if (Heating_Mode.state == 0) {
        // boiler off
        sendCommand(boiler,OFF)
    } else if (Heating_Mode.state == 1) {
        // boiler on
        sendCommand(boiler,ON)
    } else if (Heating_Mode.state == 2) {
        // get the current setpoint 
        var Number setpoint = Heating_Setpoint.state as DecimalType

        // calculate the turn on/off temperatures
        var Number turnOnTemp = setpoint - 0.5
        var Number turnOffTemp = setpoint + 0.5

        // get the current temperature
        var Number temp = Second_Temp.state as DecimalType
            
        // determine whether we need to turn on/off the boiler
        if (temp <= turnOnTemp) {
            // turn on temp has been reached so switch on the boiler
            sendCommand(boiler,ON)
        } else if (temp >= turnOffTemp) {
            // turn off temp has been reached so switch off the boiler
            sendCommand(boiler,OFF)
        }
        } else {
            // boiler off
            sendCommand(boiler,OFF)
        }
end
```

### Sitemap:

```
sitemap map1 label="Sitemap" {
      Frame label="Home" {
          Text item=boiler_confirm icon=fire
          Text item=BoilerESP_Online 
          //Switch item=boiler
          //Switch item=timer
          Text item=Second_Hum
          Text item=Second_Temp
          
          
          Switch item=Heating_Mode label="Heating Mode" mappings=[0="Off", 1="On", 2="Auto"]
          Setpoint item=Heating_Setpoint label="Target Temperature [%.1f °C]" minValue=16 maxValue=24 step=0.5 

          }
}
```
