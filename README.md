# ExoPlayerPractice
[구글 코드랩](https://developer.android.com/codelabs/exoplayer-intro?hl=ko#6)을 통한 ExoPlayer 사용법을 연습한 프로젝트입니다.  

코드랩에 나오는 버전과 현재의 버전이 달라 수정된 부분들이 존재합니다.
<br>
<br>

## 230427 

### ExoPlayer 사용방법 정리

**1. 모듈 수준의 build.gradle 파일에 dependencies 추가**
``` kotlin
// 230427 기준 최신 버전 : 1.0.0-rc01 
def mediaVersion = "1.0.0-rc01"
implementation "androidx.media3:media3-exoplayer:$mediaVersion"
implementation "androidx.media3:media3-ui:$mediaVersion"
implementation "androidx.media3:media3-exoplayer-dash:$mediaVersion"
```
<br>

**2. 레이아웃 xml 파일에 androidx.media3.ui.PlayerView 추가**
- app:use_controller="true" <= 컨트롤러 보이게 할지 설정하는 코드
- app:show_timeout="10000" <= 컨트롤러가 사라지는데 걸리는 시간 설정하는 코드
- app:controller_layout_id="@layout/custom_player_control_view" <= 커스텀 컨트롤러(xml) 설정하는 코드
<br>

**3. ExoPlayer 객체 생성**
``` kotlin
// ExoPlayer 객체용 변수
private var player: ExoPlayer? = null

// ExoPlayer 객체 생성용 메소드
private fun initializePlayer() {
    val trackSelector = DefaultTrackSelector(this).apply {
        setParameters(buildUponParameters().setMaxVideoSizeSd())
    }

    player = ExoPlayer.Builder(this)
        .setTrackSelector(trackSelector)
        .build()
        .also { exoPlayer ->
            viewBinding.videoView.player = exoPlayer

            // player가 재생할 MediaItem 생성
            val mediaItem = MediaItem.fromUri(getString(R.string.media_url_mp4))
            exoPlayer.addMediaItem(mediaItem)

            // player가 재생할 재생 목록 추가
            val secondMediaItem = MediaItem.fromUri(getString(R.string.media_url_mp3))
            exoPlayer.addMediaItem(secondMediaItem)

            // 가변 품질 스트리밍 형식의 DASH 컨텐츠 스트리밍
            val thirdMediaItem = MediaItem.Builder()
                .setUri(getString(R.string.media_url_dash))
                .setMimeType(MimeTypes.APPLICATION_MPD)
                .build()
            exoPlayer.addMediaItem(thirdMediaItem)
            
            // 재생정보 복원
            exoPlayer.playWhenReady = playWhenReady
            exoPlayer.seekTo(currentItem, playbackPosition)
            
            // player에 이벤트 리스너 추가
            exoPlayer.addListener(playbackStateListener)
            exoPlayer.prepare()
        }
}
```
<br>

**4. 이벤트 리스너 추가**
``` kotlin
// player 이벤트 리스너
// activity 외부에서 선언
private fun playbackStateListener() = object : Player.Listener {
    override fun onPlaybackStateChanged(playbackState: Int) {
        val stateString: String = when (playbackState) {
            ExoPlayer.STATE_IDLE -> "ExoPlayer.STATE_IDLE      -"
            ExoPlayer.STATE_BUFFERING -> "ExoPlayer.STATE_BUFFERING -"
            ExoPlayer.STATE_READY -> "ExoPlayer.STATE_READY     -"
            ExoPlayer.STATE_ENDED -> "ExoPlayer.STATE_ENDED     -"
            else -> "UNKNOWN_STATE             -"
        }
        Log.d(TAG, "changed state to $stateString")
    }
}

// player 이벤트 리스너 객체 생성
// activity 내부에서 선언
private val playbackStateListener: Player.Listener = playbackStateListener()

// initializePlayer() 메소드에서 리스너 추가
exoPlayer.addListener(playbackStateListener)

// releasePlayer() 메소드에서 리스너 제거
removeListener(playbackStateListener)
```
<br>

**5. ExoPlayer 객체 해제용 메소드 작성**
```kotlin
// 재생 혹은 일시정지 정보 저장 변수
private var playWhenReady = true
// 현재 재생 중인 미디어의 인덱스를 저장 변수
private var currentItem = 0
// 현재 재생 중인 미디어의 재생 위치 저장 변수
private var playbackPosition = 0L

// ExoPlayer 객체 해제용 메소드
private fun releasePlayer() {
    player?.run {
        playbackPosition = this.currentPosition
        currentItem = this.currentMediaItemIndex
        playWhenReady = this.playWhenReady
        removeListener(playbackStateListener)
        release()
    }
    player = null
}
```
<br>

**6. 전체화면으로 변경하는 코드 추가**
``` kotlin 
@SuppressLint("InlinedApi")
private fun hideSystemUi() {
    // 시스템 윈도우 패딩 해제
    WindowCompat.setDecorFitsSystemWindows(window, false)

    // 전체 화면 모드로 변경
    WindowInsetsControllerCompat(window, viewBinding.videoView).apply {
        hide(WindowInsetsCompat.Type.systemBars())
        systemBarsBehavior = WindowInsetsControllerCompat.BEHAVIOR_SHOW_TRANSIENT_BARS_BY_SWIPE
    }
}
```
<br>

**7. 생명주기에 맞게 ExoPlayer 처리**
``` kotlin
override fun onStart() {
    super.onStart()
    if (Util.SDK_INT >= 24) {
        initializePlayer()
    }
}

override fun onResume() {
    super.onResume()
    hideSystemUi()
    if (Util.SDK_INT < 24 || playe
        initializePlayer()
    }
}

override fun onPause() {
    super.onPause()
    if (Util.SDK_INT < 24) {
        releasePlayer()
    }
}

override fun onStop() {
    super.onStop()
    if (Util.SDK_INT >= 24) {
        releasePlayer()
    }
}
```
