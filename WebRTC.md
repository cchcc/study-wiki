### Android build

<https://webrtc.org/native-code/android/>  
그냥 위순서대로 하면됨. 무난하게 ubuntu 에서 빌드하고 WebRTC 최초 빌드하는 거면 컴파일 하기 전에 `build/install-build-deps-android.sh` 최소 한번은 해줄것.  

빌드는 풀빌드 할필요없이 demo app만 빌드  
```
gn gen out/android_arm64 --args='target_os="android" target_cpu="arm64"'
ninja -C out/android_arm64 AppRTCMobile
```  

```
gn gen out/and_arm --args='target_os="android" target_cpu="arm" is_component_build=false'
ninja -C out/and_arm AppRTCMobile
```
