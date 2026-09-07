# Ancalentari Twitch Stream Recorder
This script allows you to record twitch streams live to .mp4 files.  
It is an improved version of [junian's twitch-recorder](https://gist.github.com/junian/b41dd8e544bf0e3980c971b0d015f5f6), migrated to [**helix**](https://dev.twitch.tv/docs/api) - the new twitch API. It uses OAuth2.
## Requirements
1. [python3.8](https://www.python.org/downloads/release/python-380/) or higher  
2. [streamlink](https://streamlink.github.io/)  
3. [ffmpeg](https://ffmpeg.org/)
4. optional: [streamlink-ttvlol](https://github.com/2bc4/streamlink-ttvlol), for proxying twitch ads

## Setting up
1) Check if you have latest version of streamlink:
    * `streamlink --version` shows current version
    * `streamlink --version-check` shows available upgrade
    * `sudo pip install --upgrade streamlink` do upgrade

2) Install `requests` module [if you don't have it](https://pypi.org/project/requests/)  
   * Windows:    ```python -m pip install requests```  
   * Linux:      ```python3.8 -m pip install requests```
3) Create `config.py` file in the same directory as `twitch-recorder.py` with:
```properties
root_path = "/home/abathur/Videos/twitch"
username = ["forsen", "cinna"]
client_id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
client_secret = "zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz"
streamlink_config = []
```
`root_path` - path to a folder where you want your VODs to be saved to  
`username` - name of the streamer or streamers you want to record by default, as an array  
`client_id` - you can grab this from [here](https://dev.twitch.tv/console/apps) once you register your application  
`client_secret` - you generate this [here](https://dev.twitch.tv/console/apps) as well, for your registered application  
`streamlink_config` - extra parameters for streamlink, see [ad blocking](#ad-blocking-with-streamlink-ttvlol-optional)  

> Older versions of this README suggested `--twitch-disable-ads` here. Streamlink now skips ad
> segments on its own, and the option has been reduced to a deprecated no-op that logs
> `The --twitch-disable-ads plugin argument has been disabled and will be removed in the
> future`. It is safe to drop.

## Ad blocking with streamlink-ttvlol (optional)
Streamlink skips ad segments by itself, which leaves a gap in the recording for as long as the
ads run. [streamlink-ttvlol](https://github.com/2bc4/streamlink-ttvlol) replaces streamlink's
built-in twitch plugin and can request the playlist through a proxy instead, avoiding the gap.

> **Match the plugin version to your streamlink version.** This is the one thing that reliably
> breaks. A plugin built against an older streamlink still loads, then fails the moment a
> channel goes live, with an error such as
> `TypeError: TwitchHLSStreamReader.__init__() got an unexpected keyword argument 'name'`.
> The recording exits after roughly a second and no file is written, so it looks like the
> stream was simply never there. Re-check the plugin after every streamlink upgrade.

1) Check which streamlink you are on:
```shell script
streamlink --version
```
2) On the [releases page](https://github.com/2bc4/streamlink-ttvlol/releases), pick the newest
   release whose stated minimum streamlink version is not higher than yours, and download its
   `twitch.py`.
3) Put that file in streamlink's plugin directory, creating the directory if it does not exist:
   * Windows: `%APPDATA%\streamlink\plugins`
   * macOS: `~/Library/Application Support/streamlink/plugins`
   * Linux: `~/.local/share/streamlink/plugins`

   On Linux, substituting the release tag you picked:
```shell script
mkdir -p ~/.local/share/streamlink/plugins
curl -fL -o ~/.local/share/streamlink/plugins/twitch.py \
  https://github.com/2bc4/streamlink-ttvlol/releases/download/<tag>/twitch.py
```
4) Confirm it loaded. This only lists the available qualities, it records nothing:
```shell script
streamlink --loglevel info twitch.tv/<channel>
```
   Streamlink reports both the override and the plugin version:
```
[session][info] Plugin twitch is being overridden by .../streamlink/plugins/twitch.py
[plugins.twitch][info] streamlink-ttvlol 8.3.0-20260701 (8.5.0)
```
5) To actually route the playlist through a proxy, add the option to `streamlink_config` in
   `config.py`:
```properties
streamlink_config = ["--twitch-proxy-playlist=https://your-proxy"]
```
   `--twitch-proxy-playlist-exclude` and `--twitch-proxy-playlist-fallback` are also available;
   run `streamlink --help` for details. Without `--twitch-proxy-playlist` the plugin is
   installed but behaves like the built-in one.

To remove the plugin, delete `twitch.py` from the plugin directory and streamlink falls back to
its built-in twitch plugin. To ignore it for a single run, pass `--no-plugin-sideloading`.

## Running script
The script will be logging to a console and to a file `twitch-recorder.log`
### On linux
Run the script
```shell script
python3.8 twitch-recorder.py
```
To record a specific streamer use `-u` or `--username`
```shell script
python3.8 twitch-recorder.py --username forsen
```
To specify quality use `-q` or `--quality`
```shell script
python3.8 twitch-recorder.py --quality 720p
```
To change default logging use `-l`, `--log` or `--logging`
```shell script
python3.8 twitch-recorder.py --log warn
```
To disable ffmpeg processing (fixing errors in recorded file) use `--disable-ffmpeg`
```shell script
python3.8 twitch-recorder.py --disable-ffmpeg
```
If you want to run the script as a job in the background and be able to close the terminal:
```shell script
nohup python3.8 twitch-recorder.py >/dev/null 2>&1 &
```
In order to kill the job, you first list them all:
```shell script
jobs
```
The output should show something like this:
```shell script
[1]+  Running                 nohup python3.8 twitch-recorder > /dev/null 2>&1 &
```
And now you can just kill the job:
```shell script
kill %1
```
### On Windows
You can run the scipt from `cmd` or [terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab), by simply going to the directory where the script is located at and using command:
```shell script
python twitch-recorder.py
```
The optional parameters should work exactly the same as on Linux.