# Go on Brillo

jbd's notes on Go development on Brillo devices.

## Goals

* Make GOOS=android working on Brillo boards by using the BDK toolchain.
* Publish libbinder bindings to enable services written in Go.
* Publish bindings for the HAL layer.
* Add GOOS=android/{arm,386} BDK toolchain builds to the Go build [dashboard](http://build.golang.org) and maintain CI devices.

## Guides

### 1. Build Go binaries targeting Brillo

Note: You need to have BDK installed before beginning.

Install the prebuilt standard library, targeting both ARM and 386 devices:

```
CGO_ENABLED=1 \
	CC=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-gcc \
	CXX=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-g++ \
	GOOS=android \
	GOARCH=386 \
	go install std && \
CGO_ENABLED=1 \
	CC=$BDK_HOME/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/arm-linux-androideabi-gcc \
	CXX=$BDK_HOME/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/arm-linux-androideabi-g++ \
	GOOS=android GOARCH=arm GOARM=7 \
	go install std
```

#### ARM targets

```
CGO_ENABLED=1 \
	CC=$BDK_HOME/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/arm-linux-androideabi-gcc \
	CXX=$BDK_HOME/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/arm-linux-androideabi-g++ \
	GOOS=android GOARCH=arm GOARM=7 \
	go build <pkg>
```

#### x86 targets

```
CGO_ENABLED=1 \
	CC=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-gcc \
	CXX=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-g++ \
	GOOS=android \
	GOARCH=386 \
	go build <pkg>
```

### 2. HAL calls

Currently, HAL layer doesn't have Go bindings and happened to be contributed in the future. In the meantime, you can use CGO to make HAL calls.

_Note: The final binary is targeting an Intel Edison board. Tweak GOARCH depending on your target device._

```
$ cat $GOPATH/src/github.com/rakyll/brillolights
package main

/*
#cgo LDFLAGS: -llog -lhardware

#import <hardware/lights.h>

func setLight() {
  light_state_t state = {};
  state.color = true;
  state.flashMode = LIGHT_FLASH_NONE;
  state.brightnessMode = BRIGHTNESS_MODE_USER;

  light_device_t* light_device = nullptr;
  int ret = hw_get_module(LIGHTS_HARDWARE_MODULE_ID, &light_device);
  if (ret) {
    LOG(ERROR) << "Failed to load the lights HAL.";
    return;
  }
  CHECK(light_device);
  rc = light_device->set_light(light_device, &state);
  if (rc) {
    LOG(ERROR) << "Unable to set " << hal_led_map_[index];
    return;
  }
  light_device->common.close(
      reinterpret_cast<hw_device_t*>(light_device));
  if (rc) {
    LOG(ERROR) << "Unable to close " << hal_led_map_[index];
    return;
  }
}
*/
import "C"

func main() {
	C.setLight()
}

$ CGO_ENABLED=1 \
	CC=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-gcc \
	CXX=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-g++ \
	GOOS=android \
	GOARCH=386 \
	go build github.com/rakyll/brillolights

# push the binary to an existing device
$ adb push brillolights /data/brillolights

# run the binary
$ adb shell /data/brillolights
```
