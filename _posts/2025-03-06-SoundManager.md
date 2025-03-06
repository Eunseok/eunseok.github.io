---
layout: single
title: "SoundManager"
categories: [Framework]
tags: [C#, Framework]
typora-root-url: ../

---

- ### 개요

  `SoundManager` 클래스는 Unity 프로젝트에서 사운드 관리(배경음악 및 효과음)를 총괄하는 유틸리티 클래스입니다. 이 클래스는 게임 사운드를 유연하게 제어하고, 플레이어 경험을 강화하기 위해 설계되었습니다. 주요 기능으로는 배경음악(BGM)의 재생 및 전환, 사운드 이펙트(SFX) 재생, 볼륨 및 뮤트 설정, 페이드 인/아웃 등 다양한 사운드 관리 작업을 제공합니다.

  ------

  ### 주요 기능

  1. **BGM 관리**
     - 배경음악(BGM)을 재생, 정지, 전환하며 페이드 인/아웃을 지원합니다.
     - 상황 변화에 따라 배경음악을 자연스럽게 전환할 수 있습니다.
  2. **SFX 재생**
     - 효과음을 재생하여 게임 내 이벤트의 임팩트를 전달합니다.
  3. **볼륨 및 뮤트 설정**
     - BGM 및 SFX 볼륨을 개별적으로 조정할 수 있으며, 전체 뮤트 기능을 제공합니다.
     - 플레이어가 선호하는 사운드 환경을 설정 가능.
  4. **페이드 처리**
     - 페이드 인/아웃 효과를 통해 음악 전환이 부드럽게 느껴지도록 지원합니다.

  ------

  ### 코드 구성 요소

  #### 1. **변수**

  ```csharp
  [Header("Audio Sources")]
  [SerializeField] private AudioSource bgmSource;
  [SerializeField] private AudioSource sfxSource;
  ```

  - **`bgmSource`**: 배경음악(BGM)을 재생하는 오디오 소스.
  - **`sfxSource`**: 사운드 이펙트(SFX)를 재생하는 오디오 소스.

  ```csharp
  [Header("Audio Settings")]
  [SerializeField] private float defaultBGMVolume = 1f;
  [SerializeField] private float defaultSFXVolume = 1f;
  ```

  - BGM/SFX의 기본 볼륨을 지정.

  ```csharp
  [SerializeField] private bool isMuted;
  ```

  - 현재 사운드 뮤트 상태를 저장.

  #### 2. **Awake 메서드**

  ```csharp
  private void Awake()
  {
      DontDestroyOnLoad(gameObject);
      ApplyVolumeSettings();
  }
  ```

  - `DontDestroyOnLoad`를 통해 씬 전환 시에도 오브젝트를 유지하도록 설정.
  - 초기에 볼륨 설정을 적용.

  #### 3. **BGM 관리 메서드**

  ##### **`PlayBGM(string clipName)`**

  ```csharp
  public void PlayBGM(string clipName)
  {
      AudioClip clip = Resources.Load<AudioClip>($"Audio/BGM/{clipName}");
      if (clip != null)
      {
          bgmSource.clip = clip;
          bgmSource.Play();
      }
  }
  ```

  - 지정된 이름의 오디오 클립을 로드하여 BGM으로 설정.
  - 로드된 클립을 재생.

  ##### **`StopBGM()`**

  ```csharp
  public void StopBGM()
  {
      bgmSource.Stop();
  }
  ```

  - 현재 재생 중인 BGM을 중지.

  ##### **`ChangeBGMWithFade(string clipName, float fadeDuration)`**

  ```csharp
  public void ChangeBGMWithFade(string clipName, float fadeDuration)
  {
      StartCoroutine(ChangeBGMCoroutine(clipName, fadeDuration));
  }
  ```

  - 페이드아웃/페이드인 과정을 통해 자연스럽게 BGM을 전환.

  ------

  #### 4. **SFX 관리 메서드**

  ##### **`PlaySFX(AudioClip clip)`**

  ```csharp
  public void PlaySFX(AudioClip clip)
  {
      sfxSource.PlayOneShot(clip);
  }
  ```

  - 제공된 오디오 클립을 SFX로 재생.

  ------

  #### 5. **볼륨 및 뮤트 설정**

  ##### **`SetMute(bool mute)`**

  ```csharp
  public void SetMute(bool mute)
  {
      isMuted = mute;
      bgmSource.mute = mute;
      sfxSource.mute = mute;
  }
  ```

  - 전체 게임 사운드를 뮤트하거나 뮤트 해제.

  ##### **`SetBGMVolume(float volume)`**

  ```csharp
  public void SetBGMVolume(float volume)
  {
      bgmSource.volume = volume;
  }
  ```

  - 배경음악의 볼륨을 설정.

  ##### **`SetSFXVolume(float volume)`**

  ```csharp
  public void SetSFXVolume(float volume)
  {
      sfxSource.volume = volume;
  }
  ```

  - 사운드 이펙트의 볼륨을 설정.

  ------

  #### 6. **페이드 메서드**

  ##### **`StartFadeOut(float fadeDuration)`**

  ```csharp
  public void StartFadeOut(float fadeDuration)
  {
      StartCoroutine(FadeOutBGM(fadeDuration));
  }
  ```

  - 오디오를 점진적으로 낮춰 페이드아웃을 진행.

  ##### **`StartFadeIn(string clipName, float fadeDuration)`**

  ```csharp
  public void StartFadeIn(string clipName, float fadeDuration)
  {
      StartCoroutine(FadeInBGM(clipName, fadeDuration));
  }
  ```

  - 지정된 BGM을 점진적으로 올려 페이드인을 진행.

  ------

  ### 클래스 전체 요약 및 활용

  #### **특징**

  1. **BGM/SFX 분리 관리**: 배경음악과 효과음을 독립적으로 처리하여 더 세밀한 제어가 가능.
  2. **페이드 처리 지원**: 페이드 인/아웃 효과를 통해 사운드의 자연스러운 전환 제공.
  3. **유연한 볼륨 조절**: BGM과 SFX의 볼륨을 개별적으로 조정 가능.
  4. **뮤트 기능 제공**: 모든 사운드를 빠르게 뮤트하거나 해제.

  ------

  #### **사용법**

  1. **BGM 재생**

  ```csharp
  SoundManager.Instance.PlayBGM("MainTheme");
  ```

  1. **SFX 재생**

  ```csharp
  AudioClip jumpSound = Resources.Load<AudioClip>("Audio/SFX/Jump");
  SoundManager.Instance.PlaySFX(jumpSound);
  ```

  1. **볼륨 조절**

  ```csharp
  SoundManager.Instance.SetBGMVolume(0.5f); // BGM 볼륨 50%
  SoundManager.Instance.SetSFXVolume(1f);   // SFX 볼륨 최대
  ```

  1. **사운드 전환**

  ```csharp
  SoundManager.Instance.ChangeBGMWithFade("BattleTheme", 1f); // 1초 페이드 전환
  ```

  ------

  이 클래스는 게임에서의 사운드를 제어하고 전환하며, 플레이어에게 더 몰입감 있는 게임 경험을 제공하기 위한 도구입니다. **유연한 설정**과 **부드러운 전환** 덕분에, 다양한 상황에서 효과적으로 활용될 수 있습니다.

<details markdown="1"> <summary>코드스니펫</summary>

  ```csharp
public class SoundManager : Singleton<SoundManager>
    {
        [Header("BGM Settings")] [SerializeField]
        private AudioSource bgmSource; // 배경 음악용 오디오 소스
        [Range(0f, 1f)]public float bgmVolume = 1f; // 배경음악(BGM)의 볼륨

        [SerializeField] private List<AudioClip> bgmClips; // 배경 음악 클립 리스트

        [Header("SFX Settings")]
        [SerializeField] [Range(0f, 1f)] private float sfxVolume = 1f; // 효과음 볼륨
        [SerializeField][Range(0f, 1f)] private float sfxPitchVariance; // 효과음 피치 변동 범위
        
        public SoundSource soundSourcePrefab; // 효과음 재생을 위한 사운드 소스 프리팹
        
        [SerializeField] private bool isMuted; // 음소거 설정 여부
    
        private Coroutine _fadeCoroutine; // 현재 활성화된 페이드 코루틴

        
        
        private void Awake()
        {
            ApplyVolumeSettings(); // 초기 볼륨 설정
            bgmSource = gameObject.GetOrAddComponent<AudioSource>();
        }
        
        /// <summary>
        /// 볼륨 설정 적용
        /// </summary>
        private void ApplyVolumeSettings()
        {
            bgmSource.volume = isMuted ? 0f : bgmVolume;
            bgmSource.loop = true;
            //isMuted | isSfxMuted? 0f : sfxVolume;
        }


        /// <summary>
        /// BGM 재생
        /// </summary>
        /// <param name="clipName">재생할 BGM 클립 이름</param>
        public void PlayBGM(string clipName)
        {
            if (isMuted) return;

            // 이름으로 BGM 찾기
            AudioClip clip = bgmClips.Find(c => c.name == clipName);

            if (clip != null && bgmSource.clip != clip)
            {
                bgmSource.Stop(); // 현재 배경 음악 정지
                bgmSource.clip = clip;// 새로운 배경 음악 설정
                bgmSource.Play(); // 배경 음악 재생
            }
        }

        /// <summary>
        /// BGM 중지
        /// </summary>
        public void StopBGM()
        {
            bgmSource.Stop();
        }

        /// <summary>
        /// SFX 재생
        /// </summary>
        /// <param name="clip">재생할 효과음 클립</param>
        public static void PlaySFX(AudioClip clip)
        {
            if (Instance.isMuted || !clip) return;

            SoundSource obj = Instantiate(Instance.soundSourcePrefab); // 새로운 사운드 소스 오브젝트 생성
            SoundSource soundSource = obj.GetComponent<SoundSource>(); // 사운드 소스 가져오기
            soundSource.Play(clip, Instance.sfxVolume, Instance.sfxPitchVariance); // 효과음 재생
        }
        
        /// <summary>
        /// 오디오 음소거 설정
        /// </summary>
        /// <param name="mute">음소거 여부</param>
        public void SetMute(bool mute)
        {
            isMuted = mute;
            ApplyVolumeSettings();
        }
        
        
        /// <summary>
        /// 배경음악 볼륨 설정
        /// </summary>
        /// <param name="volume">볼륨 크기</param>
        public void SetBGMVolume(float volume)
        {
            bgmVolume = Mathf.Clamp(volume, 0f, 1f);
            ApplyVolumeSettings();
        }
        
        
        /// <summary>
        /// 효과음 볼륨 설정
        /// </summary>
        /// <param name="volume">볼륨 크기</param>
        public void SetSFXVolume(float volume)
        {
            sfxVolume = Mathf.Clamp(volume, 0f, 1f);
            ApplyVolumeSettings();
        }
        
        
    /// <summary>
    /// 배경음악 페이드아웃
    /// </summary>
    /// <param name="fadeDuration">페이드 아웃 시간</param>
    private IEnumerator FadeOutBGM(float fadeDuration)
    {
        // 현재 실행 중인 페이드 코루틴 중단 (중복 실행 방지)
        if (_fadeCoroutine != null)
            StopCoroutine(_fadeCoroutine);

        float startVolume = bgmSource.volume;
        
        while (bgmSource.volume > 0)
        {
            bgmSource.volume = Mathf.MoveTowards(bgmSource.volume, 0f, (startVolume / fadeDuration) * Time.deltaTime);
            yield return null;
        }

        bgmSource.Stop();
        bgmSource.volume = 0; // 완전히 0으로 설정
    }

    /// <summary>
    /// 배경음악 페이드인
    /// </summary>
    /// <param name="clipName">재생할 클립 이름</param>
    /// <param name="fadeDuration">페이드 인 시간</param>
    private IEnumerator FadeInBGM(string clipName, float fadeDuration)
    {
        // 새로운 클립 가져오기
        AudioClip newClip = bgmClips.Find(c => c.name == clipName);

        if (!newClip)
        {
            Debug.LogError($"SoundManager: BGM '{clipName}' not found.");
            yield break;
        }
        
        if (bgmSource.clip != newClip | !bgmSource.isPlaying)
        {
            bgmSource.clip = newClip;
            bgmSource.volume = 0;
            bgmSource.Play();
        }


        while (bgmSource.volume < bgmVolume)
        {
            bgmSource.volume = Mathf.MoveTowards(bgmSource.volume, bgmVolume, (bgmVolume / fadeDuration) * Time.deltaTime);
            yield return null;
        }

        bgmSource.volume = bgmVolume; // 최종 볼륨 고정
    }

    /// <summary>
    /// BGM 변경 및 페이드 처리 (외부 메서드 호출)
    /// </summary>
    /// <param name="clipName">변경할 BGM 이름</param>
    /// <param name="fadeDuration">페이드 시간</param>
    public void ChangeBGMWithFade(string clipName, float fadeDuration)
    {
        // 기존 코루틴 중단 후 새로운 코루틴 시작
        if (_fadeCoroutine != null)
            StopCoroutine(_fadeCoroutine);

        _fadeCoroutine = StartCoroutine(ChangeBGMCoroutine(clipName, fadeDuration));
    }

    /// <summary>
    /// 페이드 아웃 후 페이드 인
    /// </summary>
    /// <param name="clipName">재생할 클립 이름</param>
    /// <param name="fadeDuration">페이드 시간</param>
    private IEnumerator ChangeBGMCoroutine(string clipName, float fadeDuration)
    {
        float initialVolume = bgmSource.volume;

        // 클립이 같고 이미 재생 중이라면 페이드아웃 생략
        if (bgmSource.isPlaying && bgmSource.clip && (bgmSource.clip.name == clipName))
        {
            bgmSource.volume = initialVolume; // 현재 볼륨 유지
        }
        else
        {
            yield return FadeOutBGM(fadeDuration); // 현재 재생 중인 음악 페이드아웃
        }

        yield return FadeInBGM(clipName, fadeDuration); // 새로운 음악 페이드인
    }

    /// <summary>
    /// 독립적으로 페이드 아웃 시작
    /// </summary>
    /// <param name="fadeDuration">페이드 아웃 시간</param>
    public void StartFadeOut(float fadeDuration)
    {
        if (_fadeCoroutine != null)
            StopCoroutine(_fadeCoroutine);

        _fadeCoroutine = StartCoroutine(FadeOutBGM(fadeDuration));
    }

    /// <summary>
    /// 독립적으로 페이드 인 시작
    /// </summary>
    /// <param name="clipName">재생할 클립 이름</param>
    /// <param name="fadeDuration">페이드 인 시간</param>
    public void StartFadeIn(string clipName, float fadeDuration)
    {
        if (_fadeCoroutine != null)
            StopCoroutine(_fadeCoroutine);

        _fadeCoroutine = StartCoroutine(FadeInBGM(clipName, fadeDuration));
    }

        
    }
  ```

</details>
