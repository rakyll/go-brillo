# Go on Brillo

jbd's notes on Go development on Brillo devices.

## Goals

* Make GOOS=android working on Brillo boards by using the BDK toolchain.
* Publish libbinder bindings to enable services written in Go.
* Publish bindings for the HAL layer.
* Add GOOS=android/{arm,386} BDK toolchain builds to the Go build [dashboard](http://build.golang.org) and maintain CI devices.

## Guides

### Build Go binaries targeting Brillo

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

#### 386 targets

```
CGO_ENABLED=1 \	CC=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-gcc \
CXX=$BDK_HOME/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin/x86_64-linux-android-g++ \
	GOOS=android \
	GOARCH=386 \
	go build <pkg>
```
