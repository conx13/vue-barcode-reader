<template>
  <div class="scanner-container">
    {{ deviceID }}: device id nimi
    <div v-show="!isLoading">
      <video poster="data:image/gif,AAAA" ref="scanner"></video>
      <div class="overlay-element"></div>
      <div class="laser"></div>
    </div>
  </div>
</template>

<script >
import { BrowserMultiFormatReader, Exception, DecodeHintType, BarcodeFormat } from "@zxing/library";

export default {
  name: "stream-barcode-reader",
  // props:['deviceID'],

  
  props: {
    deviceID: {
      type: String,
      default: undefined,
    },},
    data() {
      return {
        hints:new Map(),
        codeReader:null,
        isLoading: true,
        isMediaStreamAPISupported: navigator && navigator.mediaDevices && "enumerateDevices" in navigator.mediaDevices,
      };
    },
    
    mounted() {
      
      const formats = [BarcodeFormat.CODE_39]
      this.hints.set(DecodeHintType.POSSIBLE_FORMATS, formats)
      this.codeReader= new BrowserMultiFormatReader(this.hints)
      
      if (!this.isMediaStreamAPISupported) {
        throw new Exception("Media Stream API is not supported");
        return;
      }
      this.start();
      this.$refs.scanner.oncanplay = (event) => {
        this.isLoading = false;
        this.$emit("loaded");
      };
    },
    
    beforeUnmount() {
      this.codeReader.reset();
    },
    
    methods: {
      start() {
      this.codeReader.decodeFromVideoDevice(this.deviceID, this.$refs.scanner, (result, err) => {
        if (result) {
          this.$emit("decode", result.text);
          this.$emit("result", result);
        }
      });
    },
  },
};
</script>

<style scoped>
video {
  max-width: 100%;
  max-height: 100%;
}
.scanner-container {
  position: relative;
}

.overlay-element {
  position: absolute;
  top: 0;
  width: 100%;
  height: 99%;
  background: rgba(30, 30, 30, 0.5);

  -webkit-clip-path: polygon(0% 0%, 0% 100%, 20% 100%, 20% 20%, 80% 20%, 80% 80%, 20% 80%, 20% 100%, 100% 100%, 100% 0%);
  clip-path: polygon(0% 0%, 0% 100%, 20% 100%, 20% 20%, 80% 20%, 80% 80%, 20% 80%, 20% 100%, 100% 100%, 100% 0%);
}

.laser {
  width: 60%;
  margin-left: 20%;
  background-color: tomato;
  height: 1px;
  position: absolute;
  top: 40%;
  z-index: 2;
  box-shadow: 0 0 4px red;
  -webkit-animation: scanning 2s infinite;
  animation: scanning 2s infinite;
}
@-webkit-keyframes scanning {
  50% {
    -webkit-transform: translateY(75px);
    transform: translateY(75px);
  }
}
@keyframes scanning {
  50% {
    -webkit-transform: translateY(75px);
    transform: translateY(75px);
  }
}
</style>
