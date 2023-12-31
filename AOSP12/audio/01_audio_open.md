## 进程 audioServer

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
    // AudioFlinger 包含 MmapStreamInterface
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

7. `AAudioServiceEndpointMMap::open`
    + 位于 `AAudioServiceEndpointMMap.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    aaudio_result_t AAudioServiceEndpointMMAP::open()
    ```

    ```c++
    aaudio_result_t AAudioServiceEndpointMMAP::open(const aaudio::AAudioStreamRequest &request)
    ```

    ```c++
    result = openWithFormat(audioFormat);
    if (result == AAUDIO_OK) return result;
    ```

8. `AAudioEndpointManager::openExclusiveEndpoint`
    + 位于 `AAudioEndpointManager.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    sp<AAudioServiceEndpoint> AAudioEndpointManager::openExclusiveEndpoint()
    ```

    ```c++
    sp<AAudioServiceEndpoint> AAudioEndpointManager::openExclusiveEndpoint(
        AAudioService &aaudioService,
        const aaudio::AAudioStreamRequest &request,
        sp<AAudioServiceEndpoint> &endpointToSteal)
    ```

    ```c++
    sp<AAudioServiceEndpointMMAP> endpointMMap = new AAudioServiceEndpointMMAP(aaudioService);
    ALOGV("%s(), no match so try to open MMAP %p for dev %d",
          __func__, endpointMMap.get(), configuration.getDeviceId());
    endpoint = endpointMMap;

    // 在这里调用
    aaudio_result_t result = endpoint->open(request);
    if (result != AAUDIO_OK) {
        endpoint.clear();
    } else {
        mExclusiveStreams.push_back(endpointMMap);
        mExclusiveOpenCount++;
    }
    ```

9. `AAudioEndpointManager::openEndpoint`
    + 位于 `AAudioEndpointManager.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    sp<AAudioServiceEndpoint> AAudioEndpointManager::openEndpoint()
    ```

    ```c++
    sp<AAudioServiceEndpoint> AAudioEndpointManager::openEndpoint(AAudioService &audioService, const aaudio::AAudioStreamRequest &request)
    ```

    ```c++
    sp<AAudioServiceEndpoint> AAudioEndpointManager::openEndpoint(AAudioService &audioService, const aaudio::AAudioStreamRequest &request) {

        if (request.getConstantConfiguration().getSharingMode() ==  AAUDIO_SHARING_MODE_EXCLUSIVE) {
            sp<AAudioServiceEndpoint> endpointToSteal;

            // 在这里调用
            sp<AAudioServiceEndpoint> foundEndpoint =
                    openExclusiveEndpoint(audioService, request, endpointToSteal);
            if (endpointToSteal.get()) {
                endpointToSteal->releaseRegisteredStreams(); // free the MMAP resource
            }
            return foundEndpoint;
        } else {
            return openSharedEndpoint(audioService, request);
        }
    }
    ```

10. `AAudioServiceStreamBase::open`
    + 位于 `AAudioServiceStreamBase.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    aaudio_result_t AAudioServiceStreamBase::open()
    ```

    ```c++
    aaudio_result_t AAudioServiceStreamBase::open(const aaudio::AAudioStreamRequest &request)
    ```

    ```c++
    {
        std::lock_guard<std::mutex> lock(mUpMessageQueueLock);
        if (mUpMessageQueue != nullptr) {
            ALOGE("%s() called twice", __func__);
            return AAUDIO_ERROR_INVALID_STATE;
        }

        mUpMessageQueue = std::make_shared<SharedRingBuffer>();
        result = mUpMessageQueue->allocate(sizeof(AAudioServiceMessage),
                                           QUEUE_UP_CAPACITY_COMMANDS);
        if (result != AAUDIO_OK) {
            goto error;
        }

        // 在这里调用
        mServiceEndpoint = mEndpointManager.openEndpoint(mAudioService,
                                                         request);
        if (mServiceEndpoint == nullptr) {
            result = AAUDIO_ERROR_UNAVAILABLE;
            goto error;
        }
        // Save a weak pointer that we will use to access the endpoint.
        mServiceEndpointWeak = mServiceEndpoint;

        mFramesPerBurst = mServiceEndpoint->getFramesPerBurst();
        copyFrom(*mServiceEndpoint);
    }
    ```

11. `AAudioServiceStreamMMap::open`
    + 位于 `AAudioServiceStreamMMap.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    aaudio_result_t AAudioServiceStreamMMAP::open()
    ```

    ```c++
    aaudio_result_t AAudioServiceStreamMMAP::open(const aaudio::AAudioStreamRequest &request)
    ```

    ```c++
    {
        if (request.getConstantConfiguration().getSharingMode() != AAUDIO_SHARING_MODE_EXCLUSIVE) {
            ALOGE("%s() sharingMode mismatch %d", __func__,
                  request.getConstantConfiguration().getSharingMode());
            return AAUDIO_ERROR_INTERNAL;
        }

        // 在这里调用
        aaudio_result_t result = AAudioServiceStreamBase::open(request);
        if (result != AAUDIO_OK) {
            return result;
        }

        sp<AAudioServiceEndpoint> endpoint = mServiceEndpointWeak.promote();
        if (endpoint == nullptr) {
            ALOGE("%s() has no endpoint", __func__);
            return AAUDIO_ERROR_INVALID_STATE;
        }
    }
    ```

12. `AAudioService::openStream`
    + 位于 `AAudioService.h/cpp`
    + 位于 `libaaudioservice.so`

    ```c++
    Status AAudioService::openStream()
    ```

    ```c++
    Status AAudioService::openStream(const StreamRequest &_request, StreamParameters* _paramsOut, int32_t *_aidl_return)
    ```

    ```c++
    if (sharingMode == AAUDIO_SHARING_MODE_EXCLUSIVE
        && AAudioClientTracker::getInstance().isExclusiveEnabled(pid)) {
        // only trust audioserver for in service indication
        bool inService = false;
        if (isCallerInService()) {
            inService = request.isInService();
        }

        // 在这里调用
        serviceStream = new AAudioServiceStreamMMAP(*this, inService);
        result = serviceStream->open(request);
        if (result != AAUDIO_OK) {
            // Clear it so we can possibly fall back to using a shared stream.
            ALOGW("openStream(), could not open in EXCLUSIVE mode");
            serviceStream.clear();
        }
    }
    ```



## 进程 App

1. `AudioStreamInternal::open`
    + 位于 `AudioStreamInternal.h/cpp`
    + 位于 `libaaudio_internal.so`

    ```c++
    aaudio_result_t AudioStreamInternal::open()
    ```

    ```c++
    aaudio_result_t AudioStreamInternal::open(const AudioStreamBuilder &builder)
    ```

    ```c++
    {
        // 在这里调用, AAudioServiceInterface &mServiceInterface;
        mServiceStreamHandle = mServiceInterface.openStream(request, configurationOutput);
    }
    ```

    实际上，`AAudioBinderClient` 中的 `openStream` 调用的是 `AAudioBinderClient::Adapter mAdapter` 中的 `openStream`; 其调用的是 `AAudioBinderAdapter` 中的 `openStream`, 其中 `AAudioBinderAdapter` 继承的是 `AAudioServiceInterface`。

2. `AudioStreamBuilder::build`
    + 位于 `AudioStreamBuilder.h/cpp`
    + 位于 `libaaudio_internal.so`

    ```c++
    aaudio_result_t AudioStreamBuilder::build()
    ```

    ```c++
    aaudio_result_t AudioStreamBuilder::build(AudioStream** streamPtr)
    ```

    ```c++
    android::sp<AudioStream> audioStream;

    // 在这里创建 audioStream
    result = builder_createStream(getDirection(), sharingMode, allowMMap, audioStream);
    if (result == AAUDIO_OK) {
        
        // 在这里调用
        result = audioStream->open(*this);
        if (result != AAUDIO_OK) {
            bool isMMap = audioStream->isMMap();
            if (isMMap && allowLegacy) {
                ALOGV("%s() MMAP stream did not open so try Legacy path", __func__);
                // If MMAP stream failed to open then TRY using a legacy stream.
                result = builder_createStream(getDirection(), sharingMode,
                                              false, audioStream);
                if (result == AAUDIO_OK) {
                    result = audioStream->open(*this);
                }
            }
        }
        if (result == AAUDIO_OK) {
            audioStream->registerPlayerBase();
            audioStream->logOpenActual();
            *streamPtr = startUsingStream(audioStream);
        } // else audioStream will go out of scope and be deleted
    }
    ```

    ```c++
    static aaudio_result_t builder_createStream(aaudio_direction_t direction,
                                         aaudio_sharing_mode_t sharingMode,
                                         bool tryMMap,
                                         android::sp<AudioStream> &stream) {
        aaudio_result_t result = AAUDIO_OK;

        switch (direction) {

            case AAUDIO_DIRECTION_INPUT:
                if (tryMMap) {

                    // AudioStreamInternalCapture 继承于 AudioStreamInternal
                    // AudioStreamInternal        继承于 AudioStream
                    stream = new AudioStreamInternalCapture (AAudioBinderClient::getInstance(), false);
                } else {
                    stream = new AudioStreamRecord();
                }
                break;

            case AAUDIO_DIRECTION_OUTPUT:
                if (tryMMap) {
                    stream = new AudioStreamInternalPlay    (AAudioBinderClient::getInstance(), false);
                } else {
                    stream = new AudioStreamTrack();
                }
                break;

            default:
                ALOGE("%s() bad direction = %d", __func__, direction);
                result = AAUDIO_ERROR_ILLEGAL_ARGUMENT;
        }
        return result;
    }
    ```

3. `AAudioStreamBuilder_openStream`
    + 位于 `AAudioStreamBuilder.cpp`
    + 位于 `libaaudio.so`

    ```c++
    AAUDIO_API aaudio_result_t  AAudioStreamBuilder_openStream()
    ```

    ```c++
    AAUDIO_API aaudio_result_t  AAudioStreamBuilder_openStream(AAudioStreamBuilder* builder, AAudioStream** streamPtr)
    ```

    ```c++
    {
        AudioStream *audioStream = nullptr;
        aaudio_stream_id_t id = 0;
        // Please leave these logs because they are very helpful when debugging.
        ALOGI("%s() called ----------------------------------------", __func__);
        AudioStreamBuilder *streamBuilder = COMMON_GET_FROM_BUILDER_OR_RETURN   (streamPtr);

        // 在这里调用
        aaudio_result_t result = streamBuilder->build(&audioStream);
        if (result == AAUDIO_OK) {
            *streamPtr = (AAudioStream*) audioStream;
            id = audioStream->getId();
        } else {
            *streamPtr = nullptr;
        }
        ALOGI("%s() returns %d = %s for s#%u ----------------",
            __func__, result, AAudio_convertResultToText(result), id);
        return result;
    }
    ```

4. 