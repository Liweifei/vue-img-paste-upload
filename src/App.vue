<template>
  <div class="vue-img-paste-upload">
    <ul class="vipu-img-list">
      <li class="vipu-img-item" v-for="(file, index) in fileList" :key="index">
        <img :src="file.url" alt="" />
        <span class="vipu-hanlde-icon-box"  v-if="!readonly">
          <span class="iconBox" @click="handleViewFile(file)" v-if="zoom">
            <i class="vipufont vipu-icon-fangda"></i>
          </span>
          <span class="iconBox" @click="handleDelete(file, index)">
            <i class="vipufont vipu-icon-del"></i>
          </span>
        </span>
      </li>
    </ul>
    <div class="vipu-btn-box" @click="$refs.vipuInput.click()" v-if="!readonly && (fileList.length < max)">
      <input
        type="file"
        ref="vipuInput"
        class="vipu-input"
        :accept="accept"
        :multiple="multiple"
        @change="hanldeFileChange"
      />
      <i class="vipufont vipu-icon-add"></i>
    </div>
  </div>
</template>

<script>
export default {
  name: "vue-img-paste-upload",
  props: {
    active: {
      type: Boolean,
      default: false,
    },
    zoom: {
      type: Boolean,
      default: true,
    },
    readonly: {
      type: Boolean,
      default: false,
    },
    multiple: {
      type: Boolean,
      default: false,
    },
    fileList: {
      type: Array,
      default: () => {
        return [];
      },
    },
    max: {
      type: Number,
      default: 1995,
    },
    accept: {
      type: String,
      default: ".gif, .jpg, .jpeg, .png",
    },
    onDelete: {
      type: Function,
    },
    onUpload: {
      type: Function,
    },
  },
  data() {
    return {
      // fileList: [],
      realLength: 1995,
    };
  },
  watch: {
    active: {
      handler(val) {
        this.$nextTick(() => {
          this.init();
        });
      },
      immediate: true,
    },
    fileList: {
      handler(val) {
        this.fileList.forEach((item) => {
          item.hasOwnProperty("already") || (item.already = true);
        });
      },
      immediate: true,
    },
  },
  methods: {
    init() {
      const active = this.active;
      this.offEvent();
      active && !this.readonly && this.addEvent();
    },
    addEvent() {
      document.addEventListener("paste", this.handleFn, false);
    },
    offEvent() {
      document.removeEventListener("paste", this.handleFn);
    },
    handleFn(event) {
      //paste change
      event.preventDefault();
      if (event.clipboardData || event.originalEvent) {
        const clipboardData = event.clipboardData || event.originalEvent.clipboardData;
        const items = clipboardData.items;
        if (items) {
          let blob;
          //在items里找粘贴的image,据上面分析,需要循环
          for (let i = 0; i < items.length; i++) {
            if (items[i].type.indexOf("image") !== -1) {
              // console.log(items[i]);
              // console.log(typeof items[i]);
              blob = items[i].getAsFile();
            }
          }
          if (blob) {
            this.handleUpload(blob);
          }
        }
      }
    },
    getFileUrl(file) {
      //获取文件地址
      let url = null;
      if (window.createObjectURL != undefined) {
        // basic
        url = window.createObjectURL(file);
      } else if (window.URL != undefined) {
        // mozilla(firefox)
        url = window.URL.createObjectURL(file);
      } else if (window.webkitURL != undefined) {
        // webkit or chrome
        url = window.webkitURL.createObjectURL(file);
      }
      return url;
    },
    hanldeFileChange(event) {
      //input file change
      const files = Array.from(event.target.files);
      files.forEach((file) => {
        file.type.indexOf("image") !== -1 && this.handleUpload(file);
      });
    },
    handleViewFile(file) {
      //查看文件
      this.$emit("view", file);
    },
    checkExceed() {
      //add 1 everyTime
      return this.fileList.length + 1 > this.max ? true : false;
    },
    handleUpload(blob) {
      if (this.onUpload && typeof this.onUpload === "function") {
        const cb = this.onUpload(blob);
        if (cb && cb.then) {
          cb.then((param) => {
            this.doAdd(blob, param);
          });
        } else if (cb !== false) {
          this.doAdd(blob, cb);
        }
      } else {
        this.doAdd(blob);
      }
    },
    doAdd(blob, param) {
      let item = {
        already: false,
        b: blob,
        url: this.getFileUrl(blob),
      };
      typeof param === "object" && Object.assign(item, param);
      if (this.checkExceed()) {
        this.$emit("exceed", item);
        return;
      }
      this.fileList.push(item);
      this.$refs.vipuInput.value = null;
      this.$emit("after-upload", item);
    },
    handleDelete(file) {
      if (this.onDelete && typeof this.onDelete === "function") {
        const cb = this.onDelete(file);
        if (cb && cb.then) {
          cb.then(() => {
            this.doRemove(file);
          });
        } else if (cb !== false) {
          this.doRemove(file);
        }
      } else {
        this.doRemove(file);
      }
    },
    doRemove(file) {
      const index = this.fileList.findIndex((item) => file === item);
      if (index >= 0) {
        this.fileList.splice(index, 1);
        this.$emit("after-delete", file, index);
      }
    },
    getNotUpload() {
      return this.fileList.filter((item) => !item.already);
    },
    setUpload() {
      this.fileList.forEach((item) => {
        item.already = true;
      });
    },
  },
};
</script>

<style lang="scss">
@import "./assets/main.scss";
</style>
