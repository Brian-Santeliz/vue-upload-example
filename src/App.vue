<template>
  <div>
    <h1>Agrega imagenes</h1>
    <!-- <input type="file" name="" id="" @change="addChange" /> -->
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
  </div>
</template>

<script>
import uploadImage from "../graphql/uploadImage.graphql";
import uploadArrayImages from "../graphql/uploadArrayImages.graphql";
export default {
  name: "App",
  data() {
    return {
      file: null,
      disabled: false,
      files: [],
    };
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
        })
        .then(
          ({
            data: {
              singleUpload: { filename },
            },
          }) => {
            console.log(filename);
          }
        );
      this.disabled = false;
    },
    subirArray() {
      this.$apollo
        .mutate({
          mutation: uploadArrayImages,
          variables: {
            files: this.files,
          },
        })
        .then((res) => {
          console.log(res.data);
        });
    },
  },
};
</script>
