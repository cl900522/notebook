# Youtube-dl

## 安装

```sh
pip install --upgrade youtube-dl
```

## 下载视频

```sh
youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio' --embed-thumbnail --add-metadata --merge-output-format mp4  https://www.youtube.com/watch?v=Qpm2l017qDE

youtube-dl --proxy socks5://127.0.0.1:1080 -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio' --embed-thumbnail --add-metadata --merge-output-format mp4  https://www.youtube.com/watch?v=Qpm2l017qDE

youtube-dl --proxy http://127.0.0.1:1081 -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio' --embed-thumbnail --add-metadata --merge-output-format mp4  https://www.youtube.com/watch?v=Qpm2l017qDE

```

## 下载音乐

```sh
youtube-dl -x --audio-format mp3 https://www.youtube.com/watch?v=BaW_jenozKc

youtube-dl --proxy socks5://127.0.0.1:1080 -x --audio-format mp3 https://www.youtube.com/watch?v=BaW_jenozKc


youtube-dl --proxy http://127.0.0.1:1081 -x --audio-format mp3 https://www.youtube.com/watch?v=BaW_jenozKc

```