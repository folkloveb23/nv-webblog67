<template>
  <div>
    <h1>เพิ่มกีฬา</h1>
    <form v-on:submit.prevent="createBlog">
      <p>
        กีฬา:
        <input type="text" v-model="blog.title" />
      </p>
      <transition name="fade">
        <div class="thumbnail-pic" v-if="blog.thumbnail !='null'">
          <img :src="BASE_URL + blog.thumbnail" alt="thumbnail" />      
        </div>
      </transition>
      <div enctype="multipart/form-data" novalidate>
        <div class="dropbox">
          <input
            type="file"
            multiple
            :name="uploadFieldName"
            :disabled="isSaving"
            @change="handleFileChange($event.target.files)"
            accept="image/*"
            class="input-file"
          />        
          <p v-if="isInitial">
            Drag your file(s) here to begin<br />
            or click to browse
          </p>
          <p v-if="isSaving">Uploading {{ fileCount }} files...</p>
          <p v-if="isSuccess">Upload Successful.</p>
        </div>
      </div>
      <div>
        <transition-group tag="ul" class="pictures">
          <li v-for="picture in pictures":key="picture.id">
            <img
              style="margin-bottom: 5px"
              :src="BASE_URL + picture.name"
              alt="picture image"
            />        
            <br />
            <button v-on:click.prevent="useThumbnail(picture.name)">
              Thumbnail
            </button>
            <button v-on:click.prevent="delFile(picture)">Delete</button>
          </li>
        </transition-group>
        <div class="clearfix"></div>
      </div>


      <p>
        ข้อมูล :
        <input type="text" v-model="blog.content" />
      </p>
      <vue-ckeditor
        v-model.lazy="blog.content"
        :config="config"
        @blur="onBlur($event)"
        @focus="onFocus($event)"
      />
      <p>
        ประเภท :
        <input type="text" v-model="blog.category" />
      </p>
      <p>
        จำนวนผู้เล่น :
        <input type="text" v-model="blog.status" />
      </p>
      <p>
        <button type="submit">เพิ่มกีฬา</button>
      </p>
    </form>
  </div>
</template>


<script>
import BlogsService from "@/services/BlogsService";
import VueCkeditor from "vue-ckeditor2";
import UploadService from "../../services/UploadService";

const STATUS_INITIAL = 0,
  STATUS_SAVING = 1,
  STATUS_SUCCESS = 2,
  STATUS_FAILED = 3;

export default {
  data() {
    return {
      BASE_URL: "http://localhost:8081/assets/uploads/",
      error: null,
      uploadError: null,
      currentStatus: null,
      uploadFieldName: "userPhoto",
      uploadedFileNames: [],
      pictures: [],
      pictureIndex: 0,
      blog: {
        title: "",
        thumbnail: "null",
        pictures: "null",
        content: "",
        category: "",
        status: "",
      },
      config: {
        toolbar: [
          ["Bold", "Italic", "Underline", "Strike", "Subscript", "Superscript"],
        ],
        height: 300,
      },
    };
  },
  methods: {
    async delFile(material) {
      let result = confirm("Want to delete?");
      if (result) {
        let dataJSON = {
          filename: material.name,
        };

        await UploadService.delete(dataJSON);
        this.pictures = this.pictures.filter(p => p.id !== material.id);
      }
    },
    async createBlog() {
      this.blog.pictures = JSON.stringify(this.pictures);
      console.log("JSON.stringify: ", this.blog);
      try {
        await BlogsService.post(this.blog);
        this.$router.push({ name: "blogs" });
      } catch (err) {
        console.log(err);
      }
    },
   
    handleFileChange(fileList) {
      const formData = new FormData();
      if (!fileList.length) return;
      Array.from(fileList).forEach(file => {
        formData.append(this.uploadFieldName, file, file.name);
        this.uploadedFileNames.push(file.name);
      });
      this.save(formData);
    },
    async save(formData) {

        this.updatePictures();
        this.clearUploadResult();
      } catch (error) {
        console.log(error);
        this.currentStatus = STATUS_FAILED;
      }
    },
    updatePictures() {
      this.uploadedFileNames.forEach((uploadFilename) => {
        if (!this.pictures.some(p => p.name === uploadFilename)) {
          this.pictureIndex++;
          this.pictures.push({
            id: this.pictureIndex,
            name: uploadFilename,
          });
        }
      });
    },
    clearUploadResult() {
      setTimeout(() => {
        this.currentStatus = STATUS_INITIAL;
        this.uploadedFileNames = [];
      }, 5000);
    },
    useThumbnail(filename) {
      this.blog.thumbnail = filename;
    },
  },
  computed: {
    isInitial() {
      return this.currentStatus === STATUS_INITIAL;
    },
    isSaving() {
      return this.currentStatus === STATUS_SAVING;
    },
    isSuccess() {
      return this.currentStatus === STATUS_SUCCESS;
    },
    isFailed() {
      return this.currentStatus === STATUS_FAILED;
    },
  },
  components: {
    VueCkeditor,
  },
  created() {
    this.currentStatus = STATUS_INITIAL;

  },
};
</script>

<style scoped>
.dropbox {
  outline: 2px dashed grey;

  outline-offset: -10px;

  background: lemonchiffon;
  color: dimgray;


  padding: 10px ;

  min-height: 200px; 

  position: relative;

  cursor: pointer;
}
.input-file {
  opacity: 0;

  width: 100%;

  height: 200px;

  position: absolute;

  cursor: pointer;
}

.dropbox:hover {
  background: khaki;
}

.dropbox p {
  font-size: 1.2em;
  text-align: center;
  padding: 50px 0;
}
ul.pictures {
  list-style: none;
  padding: 0;
  margin: 0;
  float: left;
  padding-top: 10px;
  padding-bottom: 10px;
}
ul.pictures li {
  float: left;
}
ul.pictures li img {
  max-width: 180px;
  margin-right: 20px;
}
.clearfix {
  clear: both;
}
.thumbnail-pic img {
  width: 200px;
}
</style>
