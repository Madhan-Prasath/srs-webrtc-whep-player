## SRS WebRTC WHEP player
To set up the SRS WebRTC WHEP player, you can use the provided Docker Compose file (`docker-compose.yml`). 
This file specifies a service (`srs-player`) using the Nginx image to serve the HTML files. 

The player interface is contained in the `index.html` file located in the `./html` directory.

### Usage

Clone this repository.
```shell
git clone https://github.com/Madhan-Prasath/srs-webrtc-whep-player.git
```
Navigate to the root directory of the cloned repository:

```shell
cd srs-webrtc-whep-player
```

Use the provided Docker Compose file (docker-compose.yml) to set up the SRS WebRTC WHEP player:

```shell
docker-compose up -d
```
Access the player interface by opening http://localhost:8081 in your web browser.

Enter the WebRTC WHIP URL in the input field and click the "Play" button to start playing the video stream.

## Manual steps
### Docker Compose
```yml
version: '3'
services:
  srs-player:
    image: nginx:latest
    ports:
      - "8081:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
```

### HTML
create a index.html file in the html folder and add the following code to it

```html
<!DOCTYPE html>
<html>
<head>
    <title>Live Video Streaming Player</title>
    <meta charset="utf-8">
    <style>
        body{
            padding-top: 30px;
        }
    </style>
    <link rel="stylesheet" type="text/css" href="css/bootstrap.min.css"/>
    <script type="text/javascript" src="js/jquery-1.12.2.min.js"></script>
    <script type="text/javascript" src="js/adapter-7.4.0.min.js"></script>
    <script type="text/javascript" src="js/srs.sdk.js"></script>
    <script type="text/javascript" src="js/winlin.utility.js"></script>
    <script type="text/javascript" src="js/srs.page.js"></script>
</head>
<body>
<div class="container">
    <div id="main_info" class="alert alert-info fade in">
        <button type="button" class="close" data-dismiss="alert"></button>
        <strong><span>Usage:</span></strong> <span>Enter the WebRTC WHEP URL and click "Play" to start playing.</span>
    </div>
    <div class="form-inline">
        URL:
        <input type="text" id="txt_url" class="input-xxlarge" value="">
        <button class="btn btn-primary" id="btn_play">Play</button>
    </div>

    <label></label>

    <video id="rtc_media_player" controls autoplay></video>
    <div style="height: 100px;"></div>
</div>
<script type="text/javascript">
$(function(){
    var sdk = null; // Global handler to do cleanup when republishing.
    var startPlay = function() {
        $('#rtc_media_player').show();

        // Close PC when user replay.
        if (sdk) {
            sdk.close();
        }
        sdk = new SrsRtcWhipWhepAsync();

        // User should set the stream when publish is done, @see https://webrtc.org/getting-started/media-devices
        // However SRS SDK provides a consist API like https://webrtc.org/getting-started/remote-streams
        $('#rtc_media_player').prop('srcObject', sdk.stream);
        // Optional callback, SDK will add track to stream.
        // sdk.ontrack = function (event) { console.log('Got track', event); sdk.stream.addTrack(event.track); };

        // For example: webrtc://r.ossrs.net/live/livestream
        var url = $("#txt_url").val();
        sdk.play(url).then(function(session){
            $('#sessionid').html(session.sessionid);
            $('#simulator-drop').attr('href', session.simulator + '?drop=1&username=' + session.sessionid);
        }).catch(function (reason) {
            sdk.close();
            $('#rtc_media_player').hide();
            console.error(reason);
        });
    };

    $('#rtc_media_player').hide();
    var query = parse_query_string();
    srs_init_whep("#txt_url", query);

    $("#btn_play").click(startPlay);
    // Never play util windows loaded @see https://github.com/ossrs/srs/issues/2732
    if (query.autostart === 'true') {
        $('#rtc_media_player').prop('muted', true);
        console.warn('For autostart, we should mute it, see https://www.jianshu.com/p/c3c6944eed5a ' +
            'or https://developers.google.com/web/updates/2017/09/autoplay-policy-changes#audiovideo_elements');
        window.addEventListener("load", function(){ startPlay(); });
    }
});
</script>
</body>
</html>
```

### CSS and JS
Download the css and js files from the following links and add them to the html folder

- [bootstrap.min.css](https://ossrs.net/players/css/)
- [jquery-1.12.2.min.js](https://ossrs.net/players/js/)
- [adapter-7.4.0.min.js](https://ossrs.net/players/js/)
- [srs.sdk.js](https://ossrs.net/players/js/)
- [winlin.utility.js](https://ossrs.net/players/js/)
- [srs.page.js](https://ossrs.net/players/js/)


### Contributing
Feel free to contribute to this project by submitting issues or pull requests.

### License
This project is licensed under the MIT License

