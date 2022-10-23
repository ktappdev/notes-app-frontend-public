<template>
  <!--    <button class="new_Btn " :onclick="clickFunc">Add</button>-->
  <div class="fa-3x new_Btn" v-if="mobileCheck()">
    <i class="fa-solid fa-camera fa-2xs camera" :onclick="clickFunc" />

  </div>

  <div>
    <!--v-show="imgSrc"-->
    <input id="capture" accept="image/*" type="file" :onchange="takePhoto" capture />
    <div class="photo">
      <img id="img" :src=imgSrc :onclick="viewImage" />
    </div>
  </div>
</template>


<script>
import 'photoviewer/dist/photoviewer.css';
import PhotoViewer from 'photoviewer';
// let b = 0;
export default {
  name: "PhotoComponent",
  components: {},
  emits: ['customChange'],
  props: {
    imgSrc: String
  },
  data() {
    return {
    };
  },
  methods: {
    mobileCheck() {
      const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
      if (isMobile) {
        return true;
      }
      return false
    },

    viewImage() {
      let img = document.getElementById("img");
      const items = [
        {
          src: img.src.replace('_sm', '_lg'), // image
          title: 'Image'
        }]
      const options = {
        draggable: false,
        title: false,
        headerToolbar: ['close'],
        footerToolbar: ['zoomIn', 'zoomOut']
      };
      new PhotoViewer(items, options);
    },


    clickFunc() {
      let input = document.getElementById("capture");
      input.click();
    },

    takePhoto() {
      let input = document.getElementById("capture");
      if (input.files[0].type.indexOf("image/") > -1) {
        this.$emit("customChange", input.files[0]);
      }
    },
  },
};
</script>

<style scoped>
#capture {
  display: none;
}

#img {
  height: 60px;
  border-radius: 5px;
  cursor: pointer;
  transition: 0.3s;
}

.photo {
  align-items: center;
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 300px;
  background: transparent;
}

.new_Btn {
  float: left;

}


/* Style the Image Used to Trigger the Modal */
#img {
  border-radius: 5px;
  cursor: pointer;
  transition: 0.3s;
}
</style>