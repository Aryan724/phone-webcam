
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Phone Webcam</title>
</head>
<body>
    <h1>Phone Webcam</h1>
    <video id="video" width="100%" autoplay></video>
    <script>
        // Get video stream from the phone's webcam
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(stream => {
                const videoElement = document.getElementById('video');
                videoElement.srcObject = stream;
                
                // Send stream to the remote peer (your PC)
                const peer = new RTCPeerConnection();
                peer.addStream(stream);
                
                // Replace 'YOUR_PC_IP' with your PC's IP address
                const ws = new WebSocket('ws://192.168.86.138:8080');
                
                ws.onmessage = (message) => {
                    peer.setRemoteDescription(new RTCSessionDescription(JSON.parse(message.data)));
                };

                peer.onicecandidate = (event) => {
                    if (event.candidate) {
                        ws.send(JSON.stringify(event.candidate));
                    }
                };

                peer.createOffer().then((offer) => {
                    peer.setLocalDescription(offer);
                    ws.send(JSON.stringify(offer));
                });
            })
            .catch(error => {
                console.error("Error accessing camera:", error);
            });
    </script>
</body>
</html>
