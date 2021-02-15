<template>
  <div>
    <h1>Agrega imagenes</h1>
    <input type="file" name="" accept="image/*" id="" @change="addChange" />
    <input
      type="file"
      name=""
      id=""
      ref="inputFiles"
      @change="addMultiple"
      multiple
    />
    <button type="submit" @click="subir" :disabled="disabled">
      Submit Single
    </button>
    <button type="submit" @click="subirArray" :disabled="disabled">
      Submit ARRAY
    </button>
    <div v-for="(upload, index) in allUpload" :key="index">
      <img :src="upload.path" alt="" />
      <p>{{ upload.filename }}</p>
    </div>
  </div>
</template>

<script>
import uploadImage from "../graphql/uploadImage.graphql";
import uploadArrayImages from "../graphql/uploadArrayImages.graphql";
import getArrayImage from "../graphql/getArrayImage.graphql";
export default {
  name: "App",
  data() {
    return {
      file: null,
      disabled: false,
      files: [],
    };
  },
  apollo: {
    allUpload: getArrayImage,
  },
  methods: {
    addChange({ target }) {
      this.file = target.files[0];
    },
    addMultiple() {
      this.files = this.$refs.inputFiles.files;
      // e.target.files
    },
    subir() {
      if (!this.file) return;
      this.disabled = true;
      this.$apollo
        .mutate({
          mutation: uploadImage,
          variables: {
            file: this.file,
          },
          update: (store, { data: { singleUpload } }) => {
            const { allUpload } = store.readQuery({
              query: getArrayImage,
            });
            store.writeQuery({
              query: getArrayImage,
              data: {
                allUpload: [...allUpload, singleUpload],
              },
            });
          },
        })
        .then(
          ({
            data: {
              singleUpload: { filename, path },
            },
          }) => {
            console.log(filename, path);
            this.disabled = false;
          }
        );
    },
    subirArray() {
      this.disabled = true;

      this.$apollo
        .mutate({
          mutation: uploadArrayImages,
          variables: {
            files: this.files,
          },
          update: (store, { data: { arrayUpload } }) => {
            let { allUpload } = store.readQuery({
              query: getArrayImage,
            });
            allUpload = arrayUpload;
            store.writeQuery({
              query: getArrayImage,
              data: { allUpload },
            });
          },
        })
        .then((res) => {
          console.log(res.data);
          this.disabled = false;
        });
    },
  },
};
</script>
