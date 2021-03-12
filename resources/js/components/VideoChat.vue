<template>
  <div class="container">
    <h1 class="text-center">Laravel Video Chat</h1>
    <div class="video-container" ref="video-container">
      <video class="video-here" ref="video-here" autoplay></video>  
      <video class="video-there" ref="video-there" autoplay></video>
      <div class="text-right" v-for="(name,userId) in others" :key="userId">
        <button @click="startVideoChat(userId)" v-text="`Talk with ${name}`"/>
      </div>
    </div>
  </div>
</template>
<script>
import Pusher from 'pusher-js';
import Peer from 'simple-peer';
export default {
  props: ['user', 'others', 'pusherKey', 'pusherCluster'],
  data() {
    return {
      channel: null,
      stream: null,
      peers: {}
    }
  },
  mounted() {
    this.setupVideoChat();
  },
  methods: {
    startVideoChat(userId) {
      console.log(userId);
      this.getPeer(userId, true);
    },
    getPeer(userId, initiator) {
      if(this.peers[userId] === undefined) {
        let peer = new Peer({
          initiator,
          stream: this.stream,
          trickle: false
        });
        peer.on('signal', (data) => {
          this.channel.trigger(`client-signal-${userId}`, {
            userId: this.user.id,
            data: data
          });
        })
        .on('stream', (stream) => {
          const videoThere = this.$refs['video-there'];
          videoThere.srcObject = stream;
        })
        .on('close', () => {
          const peer = this.peers[userId];
          if(peer !== undefined) {
            peer.destroy();
          }
          delete this.peers[userId];
        });
        this.peers[userId] = peer;
      } 
      return this.peers[userId];
    },
    async setupVideoChat() {
      // To show pusher errors
      // Pusher.logToConsole = true;
      
      const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
      // const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      const videoHere = this.$refs['video-here'];
      videoHere.srcObject = stream;
      this.stream = stream;
      const pusher = this.getPusherInstance();
      this.channel = pusher.subscribe('presence-video-chat');
      this.channel.bind(`client-signal-${this.user.id}`, (signal) => 
      {
        const peer = this.getPeer(signal.userId, false);
        peer.signal(signal.data);
      });
    },
    getPusherInstance() {
      return new Pusher(this.pusherKey, {
        authEndpoint: '/auth/video_chat',
        cluster: this.pusherCluster,
        auth: {
          headers: {
            'X-CSRF-Token': document.head.querySelector('meta[name="csrf-token"]').content
          }
        }
      });
    }
  }
};
</script>
<style>
    .video-container {
        width: 500px;
        height: 380px;
        margin: 8px auto;
        border: 3px solid #000;
        position: relative;
        box-shadow: 1px 1px 1px #9e9e9e;
    }
    .video-here {
        width: 130px;
        position: absolute;
        left: 10px;
        bottom: 16px;
        border: 1px solid #000;
        border-radius: 2px;
        z-index: 2;
    }
    .video-there {
        width: 100%;
        height: 100%;
        z-index: 1;
    }
    .text-right {
        text-align: right;
    }
</style>