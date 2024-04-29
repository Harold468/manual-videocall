# 1. Manual webRTC connection using html and vanilla Javascript 

Hello, this is a simple setup of webrtc using stun servers and ice candidates to setup a webrtc connection between two peers.

![Reference Image](/vidimg.png)































# 2. It is an initial setup to foreshadow a bigger and more practical project using django and react to accomplish the same task accross a channel or socket connection


## 2.1 Steps

- Setup two video elements in your html
- Allow access to your webcam
- Setup and save the local stream 
- Setup an RTCPeerConnection
- Setup google stun servers and pass them as arguments in the peerconnection 
- Setup a remote stream 
- Create or generate an offer
- Set offer as the local description and send it over to the remote device
- Save the received offer on the remote device as the remote description 
- Generate an answer on the remote device and save the answer as its local description
- Send the answer to the initial device and save the answer received as the remote description

# 3. Steps with code

## 3.1 Setup localStream
```js
let localStream;
let remoteStream;
let peerConnection;

const servers={
    iceServers:[
        {
            urls:["stun:stun1.1.google.com:19302","stun:stun2.1.google.com:19302"]
        }
    ]
}
const init=async()=>{
    localStream = await navigator.mediaDevices.getUserMedia({video:true,audio:false})
}
```
## 3.2 Peer setup connection and remote stream
```js
const peer=async(sdpType)=>{
    peerConnection = new RTCPeerConnection(servers) 
    remoteStream = new MediaStream()
    document.getElementById('user-2').srcObject = remoteStream

    localStream.getTracks().forEach((track)=>{
        peerConnection.addTrack(track,localStream)
    })

    peerConnection.ontrack=async(event)=>{
        event.streams[0].getTracks().forEach((track)=>{
            remoteStream.addTrack(track)
        })
    }

    
    peerConnection.onicecandidate=async(event)=>{
        if(event.candidate){
            document.getElementById(sdpType).value = JSON.stringify(peerConnection.localDescription)
        }
    }

    

    
}
```

## 3.3 create and send offer
```js
const createOffer=async()=>{
    peer('offer-sdp')

    let offer = await peerConnection.createOffer()
    await peerConnection.setLocalDescription(offer)
    document.getElementById('offer-sdp').value = JSON.stringify(offer)
}
```

## 3.4 create and send answer
```js
const createAnswer=async()=>{
    peer('answer-sdp')

    let offer = document.getElementById('offer-sdp').value
    if(!offer)return alert('offer not found')
    offer = JSON.parse(offer)

    await peerConnection.setRemoteDescription(offer)
    let answer = await peerConnection.createAnswer()
    await peerConnection.setLocalDescription(answer)
    document.getElementById('answer-sdp').value = JSON.stringify(answer)
}
```

## 3.5 receive answer and set as the remote description
```js
const addAnswer=async()=>{

    let answer = document.getElementById('answer-sdp').value
    if(!answer)return alert('answer not found')
    answer = JSON.parse(answer)

    if(!peerConnection.currentRemoteDescription){
        peerConnection.setRemoteDescription(answer)
    }
 
}
```

# HTML code 
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="styles.css" />
    <title>WebRTC</title>
</head>
<body>

    <div id="video-container">
        <video id="user-1" playsInline="true" autoplay="true">

        </video>

        <video id="user-2" playsInline="true" autoplay="true">

        </video>

    </div>

    <div id="inputs-section">

        <button id="create-offer">Create offer</button>
        <textarea id="offer-sdp"></textarea>

        <button id="create-answer">Create Answer</button>
        <textarea id="answer-sdp"></textarea>

        <button id="add-answer">Answer Call</button>
        
    </div>
    
</body>
<script src="main.js"></script>
</html>

```
