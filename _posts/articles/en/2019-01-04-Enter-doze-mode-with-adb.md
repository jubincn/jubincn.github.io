## Enter Doze mode
There are three stages for these steps:
1. Prepare: Mock unplug and enable deviceidle.
2. Go through doze modes step by step.
3. Reset: Disable deviceidle and reset battery.

### Prepare: Mock unplug and enable deviceidle
1. Check battery state

Enter ```adb shell dumpsys battery``` and ```USB powered: false``` will show if you are using a monitor or else ```USB powered: true``` will show for device.

2. Mock unplug for device

Enter ```adb shell dumpsys battery unplug``` to mock unplug state. After this, you can enter ```adb shell dumpsys battery``` to check the updated state.

3. Enter deviceidle

Enter ```adb shell dumpsys deviceidle enable``` and screen off.

### Go through doze modes step by step
Enter ```adb shell dumpsys deviceidle step``` and android will enter to these idle state on each round

1. Stepped to deep: IDLE_PENDING
2. Stepped to deep: SENSING
3. Stepped to deep: IDLE_MAINTENANCE
4. Stepped to deep: IDLE

If ```adb shell dumpsys deviceidle step``` is used again, idle state will switch between IDLE_MAINTENANCE and IDLE.

### Reset: Disable deviceidle and reset battery
1. Disable deviceidle

Enter ```adb shell dumpsys deviceidle disable```

2. Reset battery

Enter ```adb shell dumpsys battery reset```


## Reference
https://blog.csdn.net/zyp009/article/details/78456906
