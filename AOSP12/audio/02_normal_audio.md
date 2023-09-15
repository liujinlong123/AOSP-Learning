<!-- # 普通 `AudioRecord` 开启流程 -->

## Application

1. 开启 `AudioRecord`

    ```kotlin
    val bufferSize = AudioRecord.getMinBufferSize(
                    44100,
                    AudioFormat.CHANNEL_IN_STEREO,
                    AudioFormat.ENCODING_PCM_16BIT
                )

    audioRecord = AudioRecord(MediaRecorder.AudioSource.MIC, 44100, AudioFormat.CHANNEL_IN_STEREO, AudioFormat.ENCODING_PCM_16BIT, bufferSize)
    ```

## Native

1. `AudioRecord::AudioRecord(const AttributionSourceState &client)`
    + 位于 `AudioRecord.cpp`
    + 位于 `libaudioclient.so`

    ```c++
    AudioRecord::AudioRecord(const AttributionSourceState &client)
    ```

    ```c++
    AudioRecord::AudioRecord(const AttributionSourceState &client)
        : mActive(false), mStatus(NO_INIT),
          mClientAttributionSource(client),
          mSessionId(AUDIO_SESSION_ALLOCATE), 
          mPreviousPriority(ANDROID_PRIORITY_NORMAL),
          mPreviousSchedulingGroup(SP_DEFAULT), 
          mSelectedDeviceId(AUDIO_PORT_HANDLE_NONE),
          mRoutedDeviceId(AUDIO_PORT_HANDLE_NONE), 
          mSelectedMicDirection(MIC_DIRECTION_UNSPECIFIED),
          mSelectedMicFieldDimension(MIC_FIELD_DIMENSION_DEFAULT)
    ```

    ```c++
    ...

    (void)set(inputSource, sampleRate, format, channelMask, frameCount, cbf, user,
            notificationFrames, false /*threadCanCallJava*/, sessionId, transferType, flags,
            uid, pid, pAttributes, selectedDeviceId, selectedMicDirection,
            microphoneFieldDimension);
    ```

2. `AudioRecord::set()`
    + 位于 `AudioRecord.cpp`
    + 位于 `libaudioclient.so`

    ```c++
    status_t AudioRecord::set(
        audio_source_t inputSource,
        uint32_t sampleRate,
        audio_format_t format,
        audio_channel_mask_t channelMask,
        size_t frameCount,
        callback_t cbf,
        void* user,
        uint32_t notificationFrames,
        bool threadCanCallJava,
        audio_session_t sessionId,
        transfer_type transferType,
        audio_input_flags_t flags,
        uid_t uid,
        pid_t pid,
        const audio_attributes_t* pAttributes,
        audio_port_handle_t selectedDeviceId,
        audio_microphone_direction_t selectedMicDirection,
        float microphoneFieldDimension,
        int32_t maxSharedAudioHistoryMs)
    ```

    ```c++
    ...

    {
        AutoMutex lock(mLock);
        status = createRecord_l(0 /*epoch*/);
    }
    ```

3. `AudioRecord::createRecord_l(const Modulo<uint32_t> &epoch)`
    + 位于 `AudioRecord.cpp`
    + 位于 `libaudioclient.so`

    ```c++
    const sp<IAudioFlinger>& audioFlinger = AudioSystem::get_audio_flinger();
    ```

    ```c++
    ...

    do {
        media::CreateRecordResponse response;
        status = audioFlinger->createRecord(VALUE_OR_FATAL(input.toAidl()), response);
        output = VALUE_OR_FATAL(IAudioFlinger::CreateRecordOutput::fromAidl(response));
        if (status == NO_ERROR) {
            break;
        }
        if (status != FAILED_TRANSACTION || --remainingAttempts <= 0) {
            ALOGE("%s(%d): AudioFlinger could not create record track, status: %d",
                  __func__, mPortId, status);
            goto exit;
        }
        
        usleep((20 + rand() % 30) * 10000);
    } while (1);

    ...

    mAudioRecord = output.audioRecord;
    ```

4. `const sp<IAudioFlinger> AudioSystem::get_audio_flinger()`
    + 位于 `AudioSystem.cpp`
    + 位于 `libaudioclient.so`

    ```c++
    // establish binder interface to AudioFlinger service
    const sp<IAudioFlinger> AudioSystem::get_audio_flinger() {
        sp<IAudioFlinger> af;
        sp<AudioFlingerClient> afc;
        bool reportNoError = false;
        {
            Mutex::Autolock _l(gLock);
            if (gAudioFlinger == 0) {
                sp<IBinder> binder;
                if (gAudioFlingerBinder != nullptr) {
                    binder = gAudioFlingerBinder;
                } else {
                    sp<IServiceManager> sm = defaultServiceManager();
                    do {
                        binder = sm->getService(String16(IAudioFlinger::DEFAULT_SERVICE_NAME));
                        if (binder != 0)
                            break;
                        ALOGW("AudioFlinger not published, waiting...");
                        usleep(500000); // 0.5 s
                    } while (true);
                }
                if (gAudioFlingerClient == NULL) {
                    gAudioFlingerClient = new AudioFlingerClient();
                } else {
                    reportNoError = true;
                }
                binder->linkToDeath(gAudioFlingerClient);
                gAudioFlinger = new AudioFlingerClientAdapter(
                        interface_cast<media::IAudioFlingerService>(binder));
                LOG_ALWAYS_FATAL_IF(gAudioFlinger == 0);
                afc = gAudioFlingerClient;
                // Make sure callbacks can be received by gAudioFlingerClient
                ProcessState::self()->startThreadPool();
            }
            af = gAudioFlinger;
        }
        if (afc != 0) {
            int64_t token = IPCThreadState::self()->clearCallingIdentity();
            af->registerClient(afc);
            IPCThreadState::self()->restoreCallingIdentity(token);
        }
        if (reportNoError) reportError(NO_ERROR);
        return af;
    }
    ```

    + `IAudioFlinger` 位于 `libaudioclient.so`
    + `AudioFlinger` 位于 `libaudiofligner.so`

5. `AudioFlinger::createRecord`
    + 位于 `AudioFlinger.cpp`
    + 位于 `libaudioflinger.so`

    ```c++
    status_t AudioFlinger::createRecord(const media::CreateRecordRequest& _input,
                                    media::CreateRecordResponse& _output)
    ```

    ```c++
    ```