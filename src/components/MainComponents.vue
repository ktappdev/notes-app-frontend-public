<template>
  <!--not sure if i still need this...-->
  <div class="div-fixed"></div>
  <!--If saving is true then flash the saving icon-->
  <div>
    <img v-if="saving" class="save-icon" alt="" src="/saving.png" />
  </div>
  <!-- on click toggle eyes will be the opposite and its binded to the class -->
  <div @click="() => (toggleEye = !toggleEye)" class="fa-2x">
    <i v-if="toggleEye" class="fa-solid fa-eye-slash fa-2xs eye" />
    <i v-else class="fa-solid fa-eye fa-2xs eye" />
  </div>
  <!-- if i knew better css i would remove this line break -->
  <br />
  <!-- Input field got alot going on
  some classes for styling, on input the loading icon is shown, the textarea is emptied, the date update section is emptied 
  the image source is cleared thinking maybe i should cancel any ongoing fetch but i haven't seen an issue yet.
  i created my own custom event "debounce" and it fires off based on a time treshold.
  the input type is binded to the togggleeye variable, and decides if to show the text or hidden password **** -->
  <div class="text-input form__group field">
    <input class="form__field" @input="
      () => [(loading = true), (this.textAreaInput = ''), (this.dateEdited = ''), (imageSrc = '')]
    " v-debounce:500ms="fetchNote" :type="[toggleEye ? 'password' : 'text']" placeholder="title" v-model="phraseInput"
      autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false" maxlength="40" />
  </div>
  <!-- Date component is just an area to display string from date edited variable(binded), 
  date edited gets set when we fetch new notes or update the textarea -->
  <div class="dateComp">
    <DateEdited v-if="[dateEdited !== '']" v-bind:date_="dateEdited" />
    <!-- not sure why these two are together but it's most likely me lacking propper css/html skills 
  and this is to tie them together 

  when a photo is taken the image data is sent to resizeImage_sm then _sm then send it to resizeImage_lg
  why not let the one function do both? well at first it was because it was easier to just copy paste and change some parameters,
  as of now i don't know if its a bad thing because the _sm image gets sent to the server first instead of both being sent.

THERE IS THE OPTION OF JUST MAKING THE LG AND SENDING THAT TO THE SERVER AND LET THE SERVER MAKE THE _SM  
sending the raw image isn't an option its too big
ok so imgSrc is a prop on the photo component and it's binded to imgSrc variable on this page, this imgSrc is set when
we pul images or just after converting a taken photo-->
    <div class="photo" id="photoId">
      <Photo @customChange="resizeImage_SM" v-if="phraseInput" :imgSrc=imageSrc />
    </div>
    <!-- thhis textarea is stress but it's currently working as best as i can get it. the main problem was getting it to grow as text 
is added to it but then stop at the bottom of the page and allow scrolling.
I couldn't make it the size of the screen because the size is set with rows... so i'm getting an idea as i type. what if i set each row to a certain height
so i basically interpret row as px... will investigate later.

couple things going on here
If imgSrc has content then make space for the image i'm doing it with classes, to get the textarea to work how i wanted i had to do
fixed position with a top of 20% or something like that. as you can imaging it's not working very well for PC. ok i just decided i will check if PC and move the text area where i want it.
let me go check the location on 1080 screen (it looks fine on 1080 screen, i'm on an Imac and there is a little more space than i would like but i will leave it)
if the title input is not empty then show the textarea...
ok so i just simplified it it can just check phraseInput directly instead of needing this function lol
showIntro() {
      return this.phraseInput === "";
    },
-->
    <div>
      <textarea :class="[(imageSrc)? 'yes-img':'no-img']" v-if="phraseInput" v-debounce:800ms="updateNote"
        v-model="textAreaInput" v-bind:rows="rowss" id="textArea" maxlength="10000"
        placeholder="Type something here..."></textarea>
    </div>
  </div>
  <!-- this is the loading icon, it spins. it's binded to loading. i just set loading to true when i need it onscreen -->
  <div class="fa-2x">
    <i v-if="loading" class="fa-solid fa-sync fa-spin fa-2xs"></i>
  </div>

  <!-- the intro is shown once phraseInput is empty -->
  <div v-if="!phraseInput">
    <Instructions />
  </div>

</template>


<script>
import axios from 'axios';
import { vue3Debounce } from "vue-debounce";
import Instructions from "./Instructions.vue";
import "animate.css"; // This is only here for one thing, ake the save icon blink. idk is i really need that, i could just use the spinning thing
import { nextTick } from "vue"; // my understand of this at this moment is that this waits until the next cycle of functions ar running, it lets animations finish and other things
import DateEdited from "@/components/DateEdited";
import moment from "moment";
import Photo from "./Photo.vue";
import * as blobUtil from "blob-util"; // I could send this functionality to the server (converts files to blobs)
const crypto = require("crypto"); //yes crypto is built into javascript

const api = "http://localhost:3000/notes/";


export default {
  directives: { //this took an entire night to get working propperly (hadd to make a directive kinda like your own event)
    debounce: vue3Debounce({
      lock: false,
      defaultTime: "1s",
      fireOnEmpty: false,
      trim: false,
    }),
  },
  name: "TextInput",
  data() {
    return {
      imageSrc: "",
      phraseInput: "",
      textAreaInput: "",
      dateEdited: "",
      saving: false,
      loading: false,
      toggleEye: true,
      rowss: 5000
    };
  },
  components: {
    DateEdited,
    Instructions,
    Photo
  },
  methods: {
    async setupPictures(item) {
      await this.resizeImage_SM(item)
      await this.resizeImage_LG(item)

    },

    async resizeImage_SM(item) {
      const tempNote = {
        phrase: this.phraseInput.trim().toLowerCase(),
        msg: ""
      }
      let fileName = this.encrypt(tempNote)
      const smallFileName = fileName.phrase + '_sm'
      var resize_width = 60; //keep asp ratio, i'm not sure if the image is smaller, what will happen
      var reader = new FileReader();
      reader.readAsDataURL(item);
      reader.name = item.name;
      reader.size = item.size;
      await this.resizeImage_LG(item)
      reader.onload = function (event) {
        var img = new Image(); //create a new image
        img.src = event.target.result; //result is base64-encoded Data URI
        img.name = event.target.name; //set name (optional)
        img.size = event.target.size; //set size (optional)
        img.onload = async function (el) {
          var elem = document.createElement("canvas");
          var scaleFactor = resize_width / el.target.width;
          elem.width = resize_width;
          elem.height = el.target.height * scaleFactor;
          //draw in canvas
          var ctx = elem.getContext("2d");
          ctx.drawImage(el.target, 0, 0, elem.width, elem.height);
          //get the base64-encoded Data URI from the resize image
          const srcEncoded = ctx.canvas.toDataURL("image/png", 1);
          // this.imageSrc = srcEncoded;
          //  document.querySelector("#img").src = srcEncoded
          // create the blob here for now
          const blobFile = await blobUtil.imgSrcToBlob(srcEncoded)
          // send file
          const formData = new FormData(); // axios makes sending files/formdata very easy, on the server there is a nice package that parses formdata and extracts the files
          formData.append('file', blobFile, smallFileName);
          await axios.post(api + 'file', formData);


        };

      };

    },


    async resizeImage_LG(item) {
      const tempNote = {
        phrase: this.phraseInput.trim().toLowerCase(),
        msg: ""
      }
      let fileName = this.encrypt(tempNote)
      const smallFileName = fileName.phrase + '_lg'
      var resize_width = 600; //keep asp ratio, i'm not sure if the image is smaller, what will happen
      var reader = new FileReader();
      reader.readAsDataURL(item);
      reader.name = item.name;
      reader.size = item.size;
      reader.onload = function (event) {
        var img = new Image(); //create a new image
        img.src = event.target.result; //result is base64-encoded Data URI
        img.name = event.target.name; //set name (optional)
        img.size = event.target.size; //set size (optional)
        img.onload = async function (el) {
          var elem = document.createElement("canvas");
          var scaleFactor = resize_width / el.target.width;
          elem.width = resize_width;
          elem.height = el.target.height * scaleFactor;
          //draw in canvas
          var ctx = elem.getContext("2d");
          ctx.drawImage(el.target, 0, 0, elem.width, elem.height);
          //get the base64-encoded Data URI from the resize image
          const srcEncoded = ctx.canvas.toDataURL("image/png", 1);
          this.imageSrc = srcEncoded;
          document.querySelector("#img").src = srcEncoded
          // create the blob here for now
          const blobFile = await blobUtil.imgSrcToBlob(srcEncoded)
          // send file
          const formData = new FormData();
          formData.append('file', blobFile, smallFileName);
          await axios.post(api + 'file', formData);
        };

      };

    },

    encrypt(note) {
      const trimmedPhrase = note.phrase.trim().toLowerCase();
      const phraseKey = crypto.createCipher("aes-128-cbc", trimmedPhrase); //key
      let encryptedPhrase = phraseKey.update(trimmedPhrase, "utf8", "hex");
      encryptedPhrase += phraseKey.final("hex");
      if (note.msg !== "" && note.msg !== undefined) {
        const msgKey = crypto.createCipher("aes-128-cbc", trimmedPhrase); //ke
        var encryptedMsg = msgKey.update(note.msg, "utf8", "hex");
        encryptedMsg += msgKey.final("hex");
        return { phrase: encryptedPhrase, msg: encryptedMsg };
      }
      return { phrase: encryptedPhrase, msg: "" };
    },

    decrypt(note, currentInput) {
      const trimmedPhrase = note.phrase.toLowerCase();
      const phraseKey = crypto.createDecipher("aes-128-cbc", currentInput); //key
      var decryptedPhrase = phraseKey.update(trimmedPhrase, "hex", "utf8");
      decryptedPhrase += phraseKey.final("utf8");
      if (note.msg !== "" && note.msg !== undefined) {
        const msgkey = crypto.createDecipher("aes-128-cbc", currentInput); //key
        var decryptedMsg = msgkey.update(note.msg, "hex", "utf8");
        decryptedMsg += msgkey.final("utf8");
        return { phrase: decryptedPhrase, msg: decryptedMsg };
      }
      return { phrase: decryptedPhrase, msg: "" };
    },



    async fetchNote() {
      if (this.phraseInput === "") {
        this.loading = false;
        this.textAreaInput = "";
        return;
      }
      const note = {
        phrase: this.phraseInput.toLowerCase(),
        msg: this.textAreaInput,
      };
      const encryptedNote = this.encrypt(note);
      const randomtimenow = new Date();
      try {
        const titleUrl = await axios.get(api + 'titleimage/' + encryptedNote.phrase + '_sm');
        // document.querySelector("#img").src = titleUrl.data.url
        if (titleUrl.data.url) {
          this.imageSrc = titleUrl.data.url + `?${randomtimenow}`;
        }

        const res = await axios.get(api + encryptedNote.phrase)
        if (res.status === 201) {
          this.textAreaInput = "";
          this.loading = false;
          this.dateEdited =
            this.phraseInput + " note created " + moment(Date.now()).fromNow();
          return;
        }
        if (res.status === 200) {
          const data = res.data;
          if (data.msg !== "" && data.msg !== undefined) {
            const decryptedData = this.decrypt(data, this.phraseInput.trim().toLowerCase());
            this.textAreaInput = decryptedData.msg;
            this.loading = false;
          }
          this.dateEdited =
            "Last edited " +
            moment(data.date).fromNow() +
            " || Created " +
            moment(data.created).calendar();
          this.loading = false;
        } else {
          this.textAreaInput = "";
          this.loading = false;
          this.dateEdited = "Something went wrong, refresh and try again";
        }
      } catch (e) {
        console.log(e)
      }
    },

    async updateNote() {
      this.saving = true;
      await nextTick();
      const element = document.querySelector(".save-icon");
      element.classList.add("animate__animated", "animate__flash");
      element.addEventListener("animationend", () => {
        element.classList.remove("animate__animated", "animate__flash");
        this.saving = false;
      });
      const updatedNote = {
        phrase: this.phraseInput.toLowerCase(),
        msg: this.textAreaInput,
      };
      const encryptedNote = this.encrypt(updatedNote);
      try {
        const res = await axios.patch(api + encryptedNote.phrase, encryptedNote)

        const response = res.data;
        // response.phrase = this.phraseInput.trim().toLowerCase()
        // const decryptedNote = this.decrypt(response, this.phraseInput.trim().toLowerCase());
        // this.textAreaInput = decryptedNote.msg; // do not reload the textarea
        // disabled reloading the text area to save resources and to make the experience smoother/snappier
        this.dateEdited =
          "Last edited " +
          moment(response.date).fromNow() +
          " || Created " +
          moment(response.created).calendar();

      } catch (e) {
        console.log(e)
      }

    },
  },



  emits: ["btn-click", "date_"], //
};
</script>

<style lang="scss" scoped>
instructions {
  position: relative;
}

textarea {
  font-family: Courier New, Courier, monospace;
  outline: 0;
  box-shadow: none;
  -webkit-overflow-scrolling: auto;
  scroll-behavior: auto;
  border: 0;
  overflow-scrolling: touch;
  overflow: auto;
  resize: none;
  font-size: 1.2rem;
  width: 97%;
  background-color: white;
  /*was transparent */
  flex-grow: 1;

  //z-index: -1000;

}


textarea::-webkit-scrollbar {
  display: none;
}

.no-img {
  position: fixed;
  left: 1%;
  right: 1%;
  bottom: 2%;
  top: 17.5%;

}

.yes-img {
  position: fixed;
  left: 1%;
  right: 1%;
  bottom: 2%;
  top: 25%;

}

input {
  font-family: Courier New, Courier, monospace;
  padding-left: 7px;
  width: auto;
  border: 0;
  border-bottom: 2px solid gray;
  outline: 0;
  font-size: 1.2rem;
  color: black;
  /*padding: 7px 0;*/
  background: white;
  position: fixed;
  top: 5%;
  left: 5%;
  right: 5%;
  text-transform: lowercase;
}

.dateComp {
  font-style: italic;
  font-size: 0.6rem;
  background: transparent;
  position: fixed;
  top: 9%;
  height: 3px;
  /*width: 100%;*/
  left: 5%;
  right: 5%;
}

.save-icon {
  background: white;
  height: 20px;
  width: 20px;
  position: fixed;
  top: 0;
  left: 48%;
}

.fa-spin {
  position: fixed;
  top: 6%;
  left: 90%;
}

.eye {
  position: fixed;
  top: 3%;
  left: 90%;
}

.logo {
  height: 60px;
  width: 60px;
  margin-left: auto;
  margin-right: auto;

}

.div-fixed {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 10%;
  background: transparent;
}


.cont.scrollbars {
  max-height: 100px;
  overflow: auto;
}




$primary: #11998e;
$secondary: #38ef7d;
$white: #fff;
$gray: #9b9b9b;


.form__field {
  transition: border-color 0.2s;

}

.form__field:focus {
  ~.form__label {

    display: block;
    transition: 0.2s;
    font-size: 1rem;
    color: $primary;
    //font-weight:700;
  }

  border-width: 4px;
  border-image: linear-gradient(to right, $primary, $secondary);
  border-image-slice: 1;
}
</style>
