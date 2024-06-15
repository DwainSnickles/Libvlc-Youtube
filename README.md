# Tutorial Libvlc videoview stream Youtube
I spent a great deal of time trying to figure how to stream youtube video to libvlc sharp videoview.
Videoview itself works awesome for actual video files eg .mp4 and others including audio .mp3.
But how can I stream a youtube link its not actually a video file eg https://youtu.be/lzm5llVmR2E
This tutorial will show you how to do it.
1) create a new winform project.
2) Add the following nuget packages.
3) 
   dotnet add package LibVLCSharp --version 3.8.5
   https://www.nuget.org/packages/LibVLCSharp/3.8.5#show-readme-container

   dotnet add package LibVLCSharp.WinForms --version 3.8.5
   https://www.nuget.org/packages/LibVLCSharp.WinForms/3.8.5#show-readme-container
   
   //this is used to parse the link to a stream and also get codec info if desired.
   dotnet add package YoutubeExplode --version 6.3.16
   https://www.nuget.org/packages/YoutubeExplode
   
5) Download the vlcfiles from here extract them to bin\Debug. eg ....bin\Debug\libvlc, these files are manditory to the success of the app. If you download the source here it should include them but please verify before continuing.
6) Add videoView from toolbox under LibVLCSharp.Winforms normally at the very top of the toolbar.
7) Open forms code in code view
8) add the following using statements at the top                                          
  using LibVLCSharp.Shared;
  using YoutubeExplode;
  using YoutubeExplode.Videos.Streams;
9) Under the InitializeComponent(); add the following code 
            //You must init libvlc core
            Core.Initialize();
           //the youtube link we are trying to stream
           //Dont worry about the red error we just need to create the method
           PlayYoutubeFile("https://youtu.be/lzm5llVmR2E");
10) Create the asyncronis method to stream the youtube link.
    ```C#
    public async void PlayYoutubeFile(string videoUrl)
    {
    try
    {
        //get youtube stream using youtube explode
        var youtube = new YoutubeClient();
        var video = await youtube.Videos.GetAsync(videoUrl);
        var title = video.Title;
        var author = video.Author.ChannelTitle;
        var duration = video.Duration;
        var streamManifest = await       
        youtube.Videos.Streams.GetManifestAsync("https://youtu.be/lzm5llVmR2E");
        var highestQualityVideoStream = streamManifest.GetMuxedStreams().OrderByDescending(s         => s.VideoQuality).First();

        // Get highest quality muxed stream
        var streamInfo = streamManifest.GetMuxedStreams().GetWithHighestVideoQuality();

        //// ...or highest bitrate audio-only stream
        //var streamInfo = streamManifest.GetAudioOnlyStreams().GetWithHighestBitrate();

        //// ...or highest quality MP4 video-only stream
        //var streamInfo = streamManifest
        //    .GetVideoOnlyStreams()
        //    .Where(s => s.Container == Container.Mp4)
        //    .GetWithHighestVideoQuality();

        // Now you can download the stream using the stream URL
        // Get the actual stream from youtube using youtube explode
        var stream = await youtube.Videos.Streams.GetAsync(streamInfo);

        // Download the stream to a file
        //await youtube.Videos.Streams.DownloadAsync(streamInfo, $"video.      
          {streamInfo.Container}");
        var libVLC = new LibVLC();
        var media = new Media(libVLC, new StreamMediaInput(stream));
        var mediaPlayer = new MediaPlayer(media);
        videoView1.MediaPlayer = mediaPlayer;
        mediaPlayer.Play();
       //its important to dispose of the media as I heard it can cause memory leaks.
        media.Dispose();
    }
    catch (Exception)
    {
        throw;
    }
}
```
   
