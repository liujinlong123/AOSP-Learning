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

3. `Status AudioPolicyService::getInputForAttr`
    + 位于 `AudioPolicyIntefaceImpl.cpp`
    + 位于 `libaudiopolicyservice.so`
    + 是 `AudioPolicyService` 的实现类

    ```c++
    AudioPolicyService::getInputForAttr()
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