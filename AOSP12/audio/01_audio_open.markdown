1. `class AudioPolicyInterface`(virtual - 抽象类必须继承) 

    ```c++
    virtual status_t getInputForAttr(const audio_attributes_t *attr,
                                         audio_io_handle_t *input,
                                         audio_unique_id_t riid,
                                         audio_session_t session,
                                         const AttributionSourceState&  attributionSouce,
                                         const audio_config_base_t *config,
                                         audio_input_flags_t flags,
                                         audio_port_handle_t *selectedDeviceId,
                                         input_type_t *inputType,
                                         audio_port_handle_t *portId) = 0;
    ```


2. `class AudioPolicyManager` : public AudioPolicyInterface, public AudioPolicyManagerObserver
   + 位于 `libaudiopolicymanagerdefault.so`
   + 位于 `AudioPolicyManager.h/cpp`

    ```c++
    AudioPolicyManager::getInputForAttr()
    ```

    ```c++
    status_t AudioPolicyManager::getInputForAttr(const audio_attributes_t *attr,
                                                 audio_io_handle_t *input,
                                                 audio_unique_id_t riid,
                                                 audio_session_t session,
                                                 const AttributionSourceState&  attributionSource,
                                                 const audio_config_base_t *config,
                                                 audio_input_flags_t flags,
                                                 audio_port_handle_t *selectedDeviceId,
                                                 input_type_t *inputType,
                                                 audio_port_handle_t *portId)
    ```

3. `AudioPolicyService::getInputForAttr`
    + 位于 `AudioPolicyIntefaceImpl.cpp`
    + 位于 `libaudiopolicyservice.so`
    + 是 `AudioPolicyService` 的实现类

    ```c++
    Status AudioPolicyService::getInputForAttr()
    ```

    ```c++
    Status AudioPolicyService::getInputForAttr(const media::AudioAttributesInternal&    attrAidl,
                                               int32_t inputAidl,
                                               int32_t riidAidl,
                                               int32_t sessionAidl,
                                               const AttributionSourceState&    attributionSource,
                                               const media::AudioConfigBase&    configAidl,
                                               int32_t flagsAidl,
                                               int32_t selectedDeviceIdAidl,
                                               media::GetInputForAttrResponse*  _aidl_return)
    ```

    ```c++
    Mutex::Autolock _l(mLock);
        {
            AutoCallerClear acc;
            // the audio_in_acoustics_t parameter is ignored by get_input()
            status = mAudioPolicyManager->getInputForAttr(&attr, &input, riid, session,
                                                          adjAttributionSource, &config,
                                                          flags, &selectedDeviceId,
                                                          &inputType, &portId);

        }
    ```

4. `AudioSystem::getInputForAttr`
   + 位于 `AudioSystem.h/cpp`
   + 位于 `libaudioclient.so`

    ```c++
    status_t AudioSystem::getInputForAttr()
    ```

    ```c++
    status_t AudioSystem::getInputForAttr(const audio_attributes_t* attr,
                                          audio_io_handle_t* input,
                                          audio_unique_id_t riid,
                                          audio_session_t session,
                                          const AttributionSourceState &attributionSource,
                                          const audio_config_base_t* config,
                                          audio_input_flags_t flags,
                                          audio_port_handle_t* selectedDeviceId,
                                          audio_port_handle_t* portId)
    ```

    ```c++
    const sp<IAudioPolicyService>& aps = AudioSystem::get_audio_policy_service();
    if (aps == 0) return NO_INIT;

    RETURN_STATUS_IF_ERROR(statusTFromBinderStatus(
            aps->getInputForAttr(attrAidl, inputAidl, riidAidl, sessionAidl, attributionSource,
                configAidl, flagsAidl, selectedDeviceIdAidl, &response)));
    ```

5. `AudioFlinger::openMmapStream`
   + 位于 `AudioFlinger.h/cpp`
   + 位于 `libaudioclient.so`

    ```c++
    status_t AudioFlinger::openMmapStream()
    ```

    ```c++
    status_t AudioFlinger::openMmapStream(MmapStreamInterface::stream_direction_t direction,
                                      const audio_attributes_t *attr,
                                      audio_config_base_t *config,
                                      const AudioClient& client,
                                      audio_port_handle_t *deviceId,
                                      audio_session_t *sessionId,
                                      const sp<MmapStreamCallback>& callback,
                                      sp<MmapStreamInterface>& interface,
                                      audio_port_handle_t *handle)
    ```

    ```c++
    if (direction == MmapStreamInterface::DIRECTION_OUTPUT) {
        ...
    } else {
        ret = AudioSystem::getInputForAttr(&localAttr, &io,
                                              RECORD_RIID_INVALID,
                                              actualSessionId,
                                              client.attributionSource,
                                              config,
                                              AUDIO_INPUT_FLAG_MMAP_NOIRQ, deviceId, &portId);
    }
    ```

6. `AAudioServiceEndpointMMap::openWithFormat`
    + 位于 `AAudioServiceEndpointMMAP.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    aaudio_result_t AAudioServiceEndpointMMAP::openWithFormat(audio_format_t audioFormat)
    ```

    ```c++
    status_t status = MmapStreamInterface::openMmapStream(streamDirection,
                                                          &attributes,
                                                          &config,
                                                          mMmapClient,
                                                          &deviceId,
                                                          &sessionId,
                                                          this, // callback
                                                          mMmapStream,
                                                          &mPortHandle);
    ```