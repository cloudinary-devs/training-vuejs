<template>
  <div class="cl-upload">
    <!-- supply h2 heading -->
    <h2>Upload an Image to Cloudinary</h2>
    <div v-show="showProgress">
      <progress-bar :options="options" :value="progress" />
    </div>

    <!-- create a form that will not submit to a server but will prevent submit and
    use the upload function as a handle-->
    <form v-on:submit.prevent="upload">
      <!-- bind cloud-name to the input -->
      <label for="cloudname-input">Cloud Name:</label>
      <input id="cloudname-input" v-model="cloudName" placeholder="Enter cloud_name from dashboard" />
      <!-- bind preset to the input -->
      <label for="preset-input">Preset:</label>
      <input id="preset-input" v-model="preset" placeholder="Enter preset from upload settings" />
      <!-- allow the user to select an image file and when they have selected it call a function 
      to handle this event-->
      <label for="file-input">Upload:</label>
      <input
        id="file-input"
        type="file"
        accept="image/png, image/jpeg"
        @change="handleFileChange($event)"
      />
      <!-- submit button is disabled until a file is selected -->
      <button type="submit" :disabled="filesSelected < 1">Upload</button>
    </form>

    <!-- display uploaded image if successful -->
    <section v-if="results && results.secure_url">
      <img :src="results.secure_url" :alt="results.public_id" />
    </section>

    <!-- display errors if not successful -->
    <section>
      <ul v-if="errors.length > 0">
        <li v-for="(error,index) in errors" :key="index">{{error}}</li>
      </ul>
    </section>
  </div>
</template>

<script>
import axios from "axios";
import ProgressBar from "vuejs-progress-bar";
export default {
  name: "CloudinaryUpload",
  components: {
    ProgressBar
  },
  data() {
    const progressBarOptions = {
      text: {
        shadowColor: "black",
        fontSize: 14,
        fontFamily: "Helvetica",
        dynamicPosition: true
      },
      progress: {
        color: "#E8C401",
        backgroundColor: "#000000"
      },
      layout: {
        height: 35,
        width: 140,
        type: "line",
        progressPadding: 0,
        verticalTextAlign: 63
      }
    };
    return {
      results: null,
      errors: [],
      file: null,
      filesSelected: 0,
      cloudName: "",
      preset: "",
      tags: "browser-upload",
      progress: 0,
      showProgress: false,
      options: progressBarOptions,
      fileContents: null,
      formData: null
    };
  },
  methods: {
    // function to handle file info obtained from local
    // file system and set the file state
    handleFileChange: function(event) {
      console.log("handlefilechange", event.target.files);
      //returns an array of files even though multiple not used
      this.file = event.target.files[0];
      this.filesSelected = event.target.files.length;
    },
    prepareFormData: function() {
      this.formData = new FormData();
      this.formData.append("upload_preset", this.preset);
      this.formData.append("tags", this.tags); // Optional - add tag for image admin in Cloudinary
      this.formData.append("file", this.fileContents);
    },
    // function to handle form submit
    upload: function() {
      //no need to look at selected files if there is no cloudname or preset
      if (this.preset.length < 1 || this.cloudName.length < 1) {
        this.errors.push("You must enter a cloud name and preset to upload");
        return;
      }
      // clear errors
      else {
        this.errors = [];
      }
      console.log("upload", this.file.name);

      let reader = new FileReader();
      // attach listener to be called when data from file
      reader.addEventListener(
        "load",
        function() {
          this.fileContents = reader.result;
          this.prepareFormData();
          let cloudinaryUploadURL = `https://api.cloudinary.com/v1_1/${this.cloudName}/upload`;
          let requestObj = {
            url: cloudinaryUploadURL,
            method: "POST",
            data: this.formData,
            onUploadProgress: function(progressEvent) {
              console.log("progress", progressEvent);
              this.progress = Math.round(
                (progressEvent.loaded * 100.0) / progressEvent.total
              );
              console.log(this.progress);
            //bind "this" to access vue state during callback
            }.bind(this)
          };
          //show progress bar at beginning of post
          this.showProgress = true;
          axios(requestObj)
            .then(response => {
              this.results = response.data;
              console.log(this.results);
              console.log("public_id", this.results.public_id);
            })
            .catch(error => {
              this.errors.push(error);
              console.log(this.error);
            })
            .finally(() => {
              setTimeout(
                function() {
                  this.showProgress = false;
                }.bind(this),
                1000
              );
            });
        }.bind(this),
        false
      );
      // call for file read if there is a file
      if (this.file && this.file.name) {
        reader.readAsDataURL(this.file);
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
form {
  display: grid;
  padding: 1em;
  background: #f9f9f9;
  border: 1px solid #c1c1c1;
  margin: 2rem auto 0 auto;
  max-width: 500px;
  padding: 1em;
}
form input {
  background: #fff;
  border: 1px solid #9c9c9c;
}
form button {
  background-color: blue;
  color: white;
  font-size: 1em;
  font-weight: bold;
  padding: 0.7em;
  width: 100%;
  border: 0;
}
form button:hover {
  background: gold;
  color: black;
}

label {
  padding: 0.5em 0.5em 0.5em 0;
}

input {
  padding: 0.7em;
  margin-bottom: 0.5rem;
}
input:focus {
  outline: 3px solid gold;
}

@media (min-width: 400px) {
  form {
    grid-template-columns: 150px 1fr;
    grid-gap: 16px;
  }

  label {
    text-align: right;
    grid-column: 1 / 2;
  }

  input,
  button {
    grid-column: 2 / 3;
  }
}

button {
  background-color: blue;
  color: white;
  font-weight: bold;
  border-radius: 10px;
}
button:focus {
  outline: none;
}
form button:disabled,
form button[disabled] {
  border: 1px solid #999999;
  background-color: #cccccc;
  color: #666666;
}
section {
  margin: 10px 0;
}
img {
  max-width: 300px;
  height: auto;
}
</style>

