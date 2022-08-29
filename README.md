# Sync

## Usage

Sync allows you to easily set a timer for a video and let it play automatically.

Since Sync is a web application, the video must be compatible with a HTML5 video player. You will need a tool to convert local unsupported files and it's easy. See [below](#file-preparation).

### Step 1: Set URL/File

Open [Sync](https://arthur-x.github.io/Sync/).

Input the video source URL in the textbox (not the website URL). You can preview the video in the player above.

![ScreenMode](./screenshots/input1.png)

Or, press `Alt-S` to select the video and subtitle (optional) from your local computer.  

![ScreenMode](./screenshots/input2.png)

You can use `Alt-S` to toggle between those two input modes.

When finished, click the **Confirm** button. 

### Step 2: Set Timer

![ScreenMode](./screenshots/timer.png)

Click on the left time-picker to choose a timestamp within the video's duration, this is where the video will begin to play.

Click on the right time-picker to choose a local time, this is when the video will start playing. If you choose a time before the current time, Sync will assume that it's for the next day.

Click the **Confirm** button when you are ready.

### Step 3: Begin!

Now you should see a big countdown for your video. This means you entered the *Screen Mode*, and the video will play itself all the way to the end when countdown goes to 0. In this mode, you cannot pause the video, and all controls (including cursor) are hided. 

![ScreenMode](./screenshots/screenmode.png)

However, you can exit the *Screen Mode* at any time by simply pressing `esc`. The video will be paused at the time you exit *Screen Mode* for you to set another timer.

### Step 4: Another Round

To switch to another video, simply refresh the page and you will go back to Step 1.

## File Preparation

Download [FFmpeg](https://ffmpeg.org), add its `bin` folder to your `PATH` environment variable.

Navigate to your source video, use the following command to convert a `.mkv` file to a `.mp4` file and extract the subtitle.
```
ffmpeg -i yourSource.mkv -c:v h264 -c:a aac output.mp4 -c:s webvtt subtitle.vtt
```
Depending on your source video's encodings and length, this could take several minutes.

Or, to do these things separately:
```
ffmpeg -i yourSource.mkv -c:v h264 -c:a aac output.mp4

ffmpeg -i yourSource.mkv -vn -an -c:s webvtt subtitle.vtt
```
Your can even specify which audio or subtitle to extract if there're multiple. To do this, use `ffprobe yourSource.mkv` to list all stream channels, find the one you want, and use the `-map` option. For example, if you want the second subtitle (usually it's in stream #0:3), use:
```
ffmpeg -i yourSource.mkv -map 0:3 -vn -an -c:s webvtt subtitle2.vtt
```
You can refer to the FFmpeg guide for more details.

## Credits

Used [Element Plus](https://element-plus.gitee.io/zh-CN/) for some UI components.
