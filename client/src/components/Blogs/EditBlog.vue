<template>
  <div>
    <h1>แก้ไขข้อมูล</h1>
    <form v-on:submit.prevent="editBlog">
      <p>กีฬา: <input type="text" v-model="blog.title" /></p>

      <transition name="fade">
        <div class="thumbnail-pic" v-if="blog.thumbnail !== 'null'">
          <img :src="BASE_URL + blog.thumbnail" alt="thumbnail" />
        </div>
      </transition>

      <form enctype="multipart/form-data" novalidate>
        <div class="dropbox">
          <input
            type="file"
            multiple
            :name="uploadFieldName"
            :disabled="isSaving"
            @change="handleFileChange"
            accept="image/*"
            class="input-file"
          />
          <p v-if="isInitial">อัปโหลดรูปที่นี่</p>
          <p v-if="isSaving">กำลังอัปโหลด {{ fileCount }} files...</p>
          <p v-if="isSuccess">อัปโหลดสำเร็จ</p>
        </div>
      </form>

      <transition-group tag="ul" class="pictures">
        <li v-for="picture in pictures" :key="picture.id">
          <img :src="BASE_URL + picture.name" alt="picture image" />
          <br />
          <button v-on:click.prevent="useThumbnail(picture.name)">อัปโหลด</button>
          <button v-on:click.prevent="delFile(picture)">ลบ</button>
        </li>
      </transition-group>

      <div class="clearfix"></div>

      <p>ข้อมูล: <input type="text" v-model="blog.content" /></p>
      <p>ประเภท: <input type="text" v-model="blog.category" /></p>
      <p>จำนวนนักกีฬา: <input type="text" v-model="blog.status" /></p>

      <p>
        <button type="submit">ยืนยัน</button>
        <button v-on:click="navigateTo('/blogs')">กลับ</button>
      </p>
    </form>
  </div>
</template>

<script>
import BlogsService from "@/services/BlogsService";
import UploadService from "@/services/UploadService";
import VueCkeditor from "vue-ckeditor2";

const STATUS_INITIAL = 0,
  STATUS_SAVING = 1,
  STATUS_SUCCESS = 2,
  STATUS_FAILED = 3;

export default {
  components: { VueCkeditor },
  data() {
    return {
      BASE_URL: "http://localhost:8081/assets/uploads/",
      blog: {
        title: "",
        thumbnail: "null",
        pictures: "null",
        content: "",
        category: "",
        status: "",
      },
      pictures: [],
      pictureIndex: 0,
      fileCount: 0,
      currentStatus: STATUS_INITIAL,
      uploadFieldName: "userPhoto",
      uploadedFileNames: [],
      config: {
        toolbar: [
          ["Bold", "Italic", "Underline", "Strike", "Subscript", "Superscript"],
        ],
        height: 300,
      },
    };
  },
  methods: {
    async editBlog() {
      try {
        await BlogsService.put(this.blog);
        this.$router.push({ name: "blogs" });
      } catch (err) {
        console.error(err);
      }
    },
    handleFileChange(event) {
      const files = event.target.files;
      this.fileCount = files.length;
      this.filesChange(event.target.name, files);
    },
    async delFile(material) {
      if (confirm("ต้องการลบไฟล์นี้หรือไม่?")) {
        await UploadService.delete({ filename: material.name });
        this.pictures = this.pictures.filter(p => p.id !== material.id);
      }
    },
    async save(formData) {
      try {
        this.currentStatus = STATUS_SAVING;
        await UploadService.upload(formData);
        this.currentStatus = STATUS_SUCCESS;

        formData.forEach(uploadFilename => {
          if (!this.pictures.some(p => p.name === uploadFilename)) {
            this.pictureIndex++;
            this.pictures.push({ id: this.pictureIndex, name: uploadFilename });
          }
        });
        this.clearUploadResult();
      } catch (error) {
        console.error(error);
        this.currentStatus = STATUS_FAILED;
      }
    },
    filesChange(fieldName, fileList) {
      const formData = new FormData();
      if (!fileList.length) return;
      Array.from(fileList).forEach(file =>
        formData.append(fieldName, file, file.name)
      );
      this.save(formData);
    },
    clearUploadResult() {
      setTimeout(() => this.reset(), 5000);
    },
    reset() {
      this.currentStatus = STATUS_INITIAL;
      this.uploadedFileNames = [];
    },
    useThumbnail(filename) {
      this.blog.thumbnail = filename;
    },
    navigateTo(route) {
      this.$router.push(route);
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
  },
  async created() {
    const blogId = this.$route.params.blogId;
    try {
      const response = await BlogsService.show(blogId);
      this.blog = response.data;
      this.pictures = JSON.parse(this.blog.pictures);
      this.pictureIndex = this.pictures.length;
    } catch (error) {
      console.error(error);
    }
  },
};
</script>

<style scoped>
div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
}
form {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  max-width: 500px;
  width: 50%;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

input[type="text"], .dropbox {
  width: 100%; /* ให้กล่องข้อความและกล่องอัปโหลดมีขนาดเท่ากัน */
  padding: 5px;
  margin: 5px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
}

button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 1em;
  cursor: pointer;
  margin: 5px;
  transition: background-color 0.3s ease;
}
button:hover {
  background-color: #45a049;
}

.dropbox {
  outline: 2px dashed #999;
  background-color: #f7f7f7;
  padding: 20px;
  min-height: 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 5px;
  transition: background-color 0.3s ease;
  cursor: pointer;
  margin-top: 15px;
  position: relative; 
}

.dropbox:hover {
  background-color: #e0e0e0;
}

.input-file {
  opacity: 0;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  cursor: pointer;
  z-index: 1; /* z-index สำหรับ input file เพื่อไม่ให้ทับกล่องข้อความ */
}

.dropbox p {
  color: #666;
  font-size: 1.1em;
  text-align: center;
}

ul.pictures {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  padding-top: 20px;
}

ul.pictures li {
  margin: 10px;
  text-align: center;
}

ul.pictures li img {
  max-width: 150px;
  border-radius: 5px;
  transition: transform 0.3s ease;
}

ul.pictures li img:hover {
  transform: scale(1.05);
}

.clearfix {
  clear: both;
}

@media (max-width: 768px) {
  form {
    padding: 15px;
  }
  ul.pictures li img {
    max-width: 120px;
  }
  button {
    width: 100%;
  }
}
</style>
